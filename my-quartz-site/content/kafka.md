# interview

به دلیل اینکه شاید سریع بخوایم برای مصاحبه آماده شیم ، این اول اودم که حتما برای مصاحبه خونده شه

## چجوری اطمینان کنیم که پیام هامون از کاکا پاک نمی شن ؟


+ replication

اگه یکی از بروکر ها کرش کنه ، میشه قبلش کافکا رو جوری کانفیگ کرد که چندین رپلیکا داشته باشه و پرودوسر و کانسیومر ها باید جوری کانفیگ بشن که سوییچ کنن رو رپلیکای جدید

+ offset commit - consumer

به صورت پیش فرض هر پیام که توسط **کانسیومر** خوانده میشود کامیت میشود ولی برای بالا بردن اطمینان می شه **manual offset commit** رو فعال کرد



+ acknowledgment - producer

این برای اینه که **پرودیوسر** مطمعن شه که پیام به بروکر رسیده ، ربطی به **کانسیومر** هنوز نداره و ۳ سطح داره :

+ +  acks=0: هیچ تأییدی دریافت نمی‌کند
+ +  acks=1: فقط از رهبر پیام را دریافت می‌کند
+ +  acks=all: پیام را فقط دریافت می‌کند اگر همه رهبرها به آن پاسخ دهند

+ Log Retention
مدت  زمانی که پیام ها در سیستم ذخیره می شوند و پیش فرض ۷ روز است



## راه های تضمین ارسال پیام

### Delivery semantics

most-once, at-least-once, or exactly-once

این ها تنظیمات ندارن بلکه با تغییر کامیت یا اکنالج ، به این ها برسیم

+ At-most-once Semantics - حداکثر یکبار


پیام یا می‌رسه یا ممکنه گم بشه، ولی دوباره تحویل داده نمی‌شه.

Producer configs:

acks=0 یا acks=1 (یعنی producer منتظر تأیید کامل از همه replica ها نیست)

retries=0 (بدون retry)

enable.idempotence=false

Consumer configs:

enable.auto.commit=true (consumer اتوماتیک offset رو commit می‌کنه)

commit قبل از پردازش پیام → اگر consumer crash کنه، پیام از دست میره.



+ At-least-once Semantics

پیام از دست نمی‌ره، ولی ممکنه تکراری تحویل بشه.

Producer configs:

acks=all (باید leader + همه replica ها تأیید بدن)

retries=Integer.MAX_VALUE (retry نامحدود برای جلوگیری از گم‌شدن پیام)

enable.idempotence=false (هنوز idempotence فعال نیست، پس ممکنه پیام تکراری بشه)

Consumer configs:

enable.auto.commit=false (خودمون offset رو commit می‌کنیم)

commit بعد از پردازش پیام → تضمین می‌کنه چیزی گم نشه، ولی ممکنه یک پیام دوبار پردازش بشه.

✅ نتیجه: هیچ پیامی گم نمی‌شه، ولی باید پردازش idempotent باشه (مثلاً آپدیت بانکی یا تراکنش تکراری نشه).


+ Exactly-once Semantics (EOS)

پیام نه گم می‌شه، نه تکرار می‌شه.

این از Kafka 0.11+ اضافه شد.

Producer configs:

enable.idempotence=true (تضمین می‌کنه producer پیام تکراری نده حتی با retry)

acks=all (نیازمنده)

retries=Integer.MAX_VALUE (نیازمنده)

max.in.flight.requests.per.connection=1 (در نسخه‌های قدیمی‌تر برای جلوگیری از reorder)

Transactional Producer:

transactional.id=some-unique-id (لازم برای Transactional Producer)

از API های initTransactions(), beginTransaction(), commitTransaction() استفاده بشه.

Consumer configs:

isolation.level=read_committed (فقط پیام‌هایی رو می‌خونه که commit شدن، نه aborted)

commit offset باید داخل همون transaction با write باشه (با Kafka Streams یا transactional producer API).

✅ نتیجه: دقیقا-once delivery end-to-end → نه پیام از دست می‌ره، نه تکراری میاد.


| Semantics     | Producer Configs                                                       | Consumer Configs                                        | رفتار                             |
| ------------- | ---------------------------------------------------------------------- | ------------------------------------------------------- | --------------------------------- |
| At-most-once  | `acks=0/1`, `retries=0`, `enable.idempotence=false`                    | `enable.auto.commit=true` (commit قبل از پردازش)        | ممکنه پیام گم شه، تکراری نداره    |
| At-least-once | `acks=all`, `retries=∞`, `enable.idempotence=false`                    | `enable.auto.commit=false`, commit بعد از پردازش        | پیام گم نمی‌شه، ممکنه تکراری باشه |
| Exactly-once  | `acks=all`, `retries=∞`, `enable.idempotence=true`, `transactional.id` | `isolation.level=read_committed`, transactional offsets | نه گم می‌شه، نه تکراری            |


# kafka

data pipelines - distributed event store -  Kafka is not a traditional message queue -  Kafka is a distributed messaging system -  RabbitMQ is a message broker, while Kafka is a distributed streaming platform. One of the primary differences between the two is that Kafka is pull-based, while RabbitMQ is push-based -Kafka offers much higher performance than message brokers like RabbitMQ - bus using a pub-sub model of stream-processing


### kafka vs rabbit

RabbitMQ cannot replay events ربیت زمانی می تواند مفید باشد که برای پخش مجدد پیام ها در یک موضوع به این ویژگی نیاز ندارید
RabbitMQ برای برنامه هایی که معماری برنامه ناشناخته است بهتر است و با بیان مشکل و راه حل توسعه و تکامل می یابد. RabbitMQ در این شرایط در مقایسه با کافکا بسیار انعطاف پذیرتر و آسان تر است. اما زمانی که برنامه بالغ شد و نیاز به مقیاس‌پذیری، توان عملیاتی زیاد، قابلیت اطمینان، استحکام و قابلیت پخش مجدد پیام‌ها وجود داشت، RabbitMQ تبدیل به یک گلوگاه می‌شود و بهتر است به کافکا بروید.

messaging system :

یک سیستم پیام رسان وظیفه انتقال دیتا ازبرنامه ای به برنامه دیگر را دارد  , دو نوع الگوی پیام رسان در دسترس است point-to-point و دیگری publish-subscribe

Point-to-point message system :

پیام ها در صف هایی قرار می گیرند و یک یا چند مصرف کننده می تواند از پیام ها استفاده کنند اما تنها یک پیام خاص را یک مصرف کننده می تواند مصرف کند هنگامی که یک مصرف کننده یک پیام را در صف می خواند پیام از آن صف می تواند خارج شود

Publish-subscribe message system :

در این سیستم پیام ها بر اساس topic یا موضوعات هستند، برخلاف point-to-point system زمانی که مصرف کنندگان از پیام ها استفاده می کنند پیام ها باقی می مانند پیام ها در یک یا چند topic هستند و مصرف کنندگان به همه ی پیام هایی که به topicهای مختلف است دسترسی دارند


## خلاصه 

اگر نیاز داریدپیام بعد استفاده حذف شود ، می توان از rabbit استفاده کرد ، ولی اگر پیام در یک بازه ی زمانی بماند ، از کافکا استفاده کنید ولی در کل کافکا بهتر است

# terms

#### topic :

stream پیام هایی که متعلق به یک دسته خاصی هستند topic نامیده می شوند و دیتاها در topicها ذخیره می شود.Topicها به پارتیشن هایی تقسیم می شوند برای هر topic Kafka حداقل یک پارتیشن را در نظر می گیرد

پاتیشن : تاپیک  ممکن است پارتیشن های زیادی داشته باشد، بنابراین می تواند مقدار دلخواه داده را مدیریت کند

#### partitions :

برای اینکه بتوانیم دیستریبیوتد کار کنیم ، و این که تاپیک ها رو بشکونیم و در نود ها یا ماشین های زیادی تقسیم کنیم ، می توانیم از پارتیشن استفاده کنیم

**آیا منطقی است در یک ماشین ، چند پارتیشن داشته باشیم ؟**

اما از بابت اینکه روی یک ماشین چند پارتیشن بالا بیاریم و روی هر ترد یک کانسیومر بالا بیاریم ، می توانیم به تعداد پارتیشن ها کانسیومر جدا از هم بالا بیاریم

ایده ال اینه که تعداد پارتیشن ها برابر یا بیشتر با تعداد کانسیومر ها در یک گروپ آیدی  باشد

**اگر در یک ماشین یک پارتیشن داشته باشیم و ۱۰ کانسیومر داشته باشیم چه اتفاقی می افتد ؟**

در این حالت یک کانسیومر داده کانسوم می کند و ۹ کانسومر دیگه بی کار میشینن



#### Partition offset : 
هر  پیام، یک شناسه منحصر به فردی دارد که آفست نامیده می شود

#### Lag consumer :
 فاصله ی بین آخرین پیامی که منتشر شده و آخرین پیامی که مصرف شده

#### Broker :

بروکر ها  سیستم ساده ای هستند که مسئول نگهداری داده های منتشر شده هستند. هر کارگزار ممکن است صفر یا بیشتر پارتیشن در هر تاپیک داشته باشد

#### Producers :

تولیدکنندگان داده ها را برای کارگزاران کافکا ارسال می کنند
 . هر بار که یک تولید کننده پیامی را برای یک کارگزار منتشر می کند، کارگزار به سادگی پیام را به آخرین فایل بخش اضافه می کند .  در واقع، پیام به یک پارتیشن اضافه می شود. سازنده همچنین می تواند به پارتیشن مورد نظر خود پیام ارسال کند    .
#### Leader :

لیدر نودی است که مسئول تمام خواندن و نوشتن برای پارتیشن داده شده است. هر پارتیشن یک سرور دارد که به عنوان رهبر عمل می کند.

#### Follower :

گره‌ای که از دستورالعمل‌های رهبر پیروی می‌کند، فالوور نامیده می‌شود. اگر رهبر شکست بخورد، یکی از پیروان به طور خودکار رهبر جدید می شود. یک فالوور به عنوان مصرف کننده عادی عمل می کند، پیام ها را کانسوم می کند و  فروشگاه داده خود را به روز می کند

#### groupID :

کافکا استیت لس است و اطلاعات اینکه هر کانسیومر تا کدام افست از تاپیک را دریافت کرده ، درون زوکیپر ذخیره می شود . اگر هنگام تعریف کانسیومر، گروپ آیدی مشخص نشود ، در هر درخواست ، تمام مسیج های تاپیک را دریافت می کند . همچنین اگر همزمان چند کانسیومر با گروپ آیدی یکسان از تاپیک کانسیوم کنند ، بسته به تعداد پارتیشن ها ، تعداد مسیج ها بین آنها تقسیم میشود .

یک گروپ آیدی اگر ۷ روز استفاده نشود حذف میشود پس ترسی از ساخت گروپ آیدی های زیاد و بی استفاده نداشته باشیم

همچنین اگر بخواهیم با شروع کانسوم کردن ‍‍‍‍‍**‍LastOffset** رو ست کنیم ، امکان نداره ، تنا راه اینه که گروپ آیدی جدید بسازیم و **LastOffset** رو بزاریم -۲


**سوال** آیا میشه از گروپ آیدی استفاده نکرد؟

جواب : بله هم سرعت بالا تری داره چون نیاز نیست محاسبه کنیم تا کجا خونده اما ۲ بدی داره اول اینکه هر چی توی تاپیک باشه می خونه و دوم اینکه تنها از یه پارتیشن می تونه بخونه ، و کانسومر بدون گروه نمیتونه تمام پارتیشن ها رو بخونه




#### standalon (single ) consumer :

در صورتی که تنها یک کانسیومر داشته باشیم ، نیازی به گروپ کانسیومر نیست ( حتی ریبالانس ) 


#### First Offset:
 کافکا داده های قدیمی را پاک می کند ، داده هایی که نگه می دارد ، شروعش با این عبارت مشخص می شود ، پس اگر عدد فرست آفست و لست آفست یکی باشد یعنی داده های قبلی منقضی شدند  . 


در مثال زیر تنها ۳ مسیج موجود است

First Offset: 271905  Last Offset: 271908  Size: 3

#### flush :
 یکی از ابزار هایی برای پابلیشر هست برای این که مطمعن شود تمام پیام ها ارسال شده ، زیرا پابلیشر در کافکا یک صف دارد و همیشه داده را تک تک نمی فرستد ، در صورتی که بخواهیم برنامه را اگزیت کنیم این دستور به کار می آید

#### key message (id) :

یکی از متا دیتا های مسیج است اما کافکا بررسی نمی کند که یونیک باشد ، درح قیقت یونیک نیست


#### commit mewssage :

می توان بعد از خوانده پیام آن را کامیت کرد ، در این صورت کافکا متوجه میشود پیام خوانده شده

واقعن نفهمیدم به چه دردی می خوره


#### peek :

کافکا به صورت پیش فرض همچین چیزی نداره ، اما میشه مکان آفست رو تغییر داد .

پس ابتدا آفست رو ذخیره می کنیم ، سپس مسیج رو کانسیوم می کنیم و در انتها آفست رو به مکان قبلی انتقال می دیم

# tips
هر تاپیک حداقل یک پارتیشن دارد ، اما اگر بخواهیم حجم داده از سمت پابلیشر را افزایش دهیم ، برای این کار پارتیشن های تاپیک را به نصبت پالیشر ها افزایش می دهیم تا راحت تر پابلیشر ها داده را در کافکا قرار دهند

اگر در یک تاپیک، چند پارتیشن داشته باشیم و سرعت پابلیش بالا باشه ، کانسیومر باید از تمام پارتیشن ها داده بخونه و این یک باتلنک هست ، برای حل این می توانیم تعداد سابسکرایبر هامون رو افزایش بدیم به طوری که هر سایبسکرایبر یک پارتیشن رو بخونه ، اما به شرطی که همه یک گروپ آی دی داشته باشن  ، اگر تعداد سابسکرایبرای یک گروپ بیشتر از پارتیشن ها باشه ، اصطلاحات سابسکرایبر بی کار (idle ) می شه .


کانسیومر ها متوانند یا تاپیک به آنها اساین شود ، یا می توان پارتیشن به آنها اساین شود ، اما هردو نمی شود .


توی حصین یه ماه درگیر یه نکته بودم ، اگر دیدید بعد از هر بار کانکشن ، مثل آدم کانسوم نمی کنه ، احتمال خیلی زیاد کانسومر شاید به دلیل  کانکارنسی روی کار نمیاد ،

مشکل من این بود بعضی وقتا که تعداد کانسیومر ها زیاد می شد ، کانسیوم دیر می کرد ، من فکر می کردم مشکل از کافکاست ولی دیدم گروپ آی دی جدید ساخته میشه ولبی کانسیومر لگ حتی ۰ هم نیست ( توی کافکا یو آی ) و بعد یه ماه فهمیدم گروپ آیدی ساخته می شه ولی نمی دونم چرا ، هیچ کی اونو کانسوم نمی کنه


# commands

مشاهده لیست تاپیک ها  
    
    sudo docker exec  -it 69078a9693f0 kafka-topics  --bootstrap-server localhost:9092 --list

نوشتن پیام در تاپیک            

    
    sudo docker exec  -it 69078a9693f0 kafka-console-producer  --topic hafizium_178_exploit --bootstrap-server localhost:9092       

خواندن پیام در تاپیک

    
    sudo docker exec  -it 69078a9693f0 kafka-console-consumer  --topic hafizium_178_exploit --bootstrap-server localhost:9092

ساخت تاپیک جدید                
    
    sudo docker exec  -it 69078a9693f0 kafka-topics --create --topic quickstart-events --bootstrap-server localhost:9092         


reset groupID

    
    sudo docker exec -it kafka-docker-kafka-1-1 kafka-consumer-groups  --bootstrap-server localhost:9092 --group news_information_extraction_112  --reset-offsets --to-earliest --execute --topic ai_tagger_topic 


حذف تاپیک

    
    sudo docker exec  -it 69078a9693f0 kafka-topics --delete --topic hafizium_178_exploit --bootstrap-server localhost:9092

تعداد مسیج در تاپیک

    
    sudo docker exec -it 69078a9693f0 kafka-run-class kafka.tools.GetOffsetShell --bootstrap-server localhost:9092 --topic hafizium_178_exploit | awk -F  ":" '{sum += $3} END {print sum}'
  
  جزییات ، تعداد پارتیشن ، رپلیکا و لیدر :

    
    sudo docker exec -it 69078a9693f0 kafka-topics  --describe --topic hafizium_178_exploit --bootstrap-server localhost:9092

می توان گروپ آیدی را داد و جزییاتی مانند اطلاعات کانسیومر ها CONSUMER-ID , HOST ,CLIENT-ID را مشاهده کرد

    
    sudo docker exec -it hafizium-kafka-1-1 kafka-consumer-groups  --bootstrap-server localhost:9092 --describe --group feeds_group

مشاهده ی آفست های یک تاپیگ در زوکیپر :

    
    sudo docker exec -it 69078a9693f0 kafka-run-class kafka.tools.GetOffsetShell  --bootstrap-server localhost:9092 -topic hafizium_178_exploit --time -1


گاهی اوقات از kafka به عنوان request , response غیر همزمان استفاده می کنیم .
در این صورت نیاز است تشخیص دهیم کدام پاسخ برای کدام جواب است و امکان دارد دو درخواست کاملن یکسان باشند اما جواب ها متفاوت باشند . 

همچنین در صورتی که سرویس درخواست کننده منتظر باشد باید یک شناسه برای جواب خود داشته باشد .

بهترین راه قرار دادن unix timestamp micro است . 

همچنین درخواست کنند منتظر می ماند که جوابی با شناسه همان تایم استمپ ارسال شود.

### confluent-kafka-go

موارد زیر نیاز است


        RUN apk update && apk add --no-cache gcc musl-dev make librdkafka

برای اجرای کانفلونس در اوبونتو با apt

        apt-get install librdkafka-dev

ولی در کل پکیج زیر تمامی نیاز ها را نصب می کند

        sudo apt-get install build-essential


### docker file

خیلی ساده با یه پارتیشن

```

version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    restart: always
    network_mode: "host"

  kafka:
    container_name: kafka
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_LISTENERS: PLAINTEXT://:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_ZOOKEEPER_CONNECT: localhost:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
    depends_on:
      - zookeeper
    network_mode: "host"
    
  kafdrop:
    container_name: kafdrop
    image: obsidiandynamics/kafdrop
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: localhost:9092
    restart: always
    depends_on:
      - kafka
    network_mode: "host"

```


#### **Confluent Schema Registry**:

اگر بخواهیم api  هایی تو کافکا رو مانند http rest  که با swagger  مستند و قابل تست کرده ، انجام بدیم از این ابزار استفاده میکنیم
