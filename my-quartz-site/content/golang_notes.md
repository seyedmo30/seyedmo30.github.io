







#### private git repository

اگر بخواهیم یه پکیبج درون گیت شرکت توسغه بدیم و تنها اونجا استفاده کنیم ، بادی مقدار زیر رو اکسپورت کنیم

یه نمونه آدرسی که درون گروه بک اند است
```
https://git.maani.app/maani/backend/kafka-wrapper
```
حالا می توان این رو به فایل /etc/environment اضافه کرد 
```
GOPRIVATE=*.example.app
```

همچنین اگر پروژه ذیل یه لایبری باشد مانند آدرس زیر ، باید درون gitconfig مقداری رو اضافه کنیم


nano ~/.gitconfig


```
[url "ssh://git@git.maani.app/"]
        insteadOf = https://git.maani.app/
```

#### 403 - 443 

یه سری وقتا یه پکیج نصب نمی شه ، شاید بشه با یه سری فلگ نصب کرد

`GOPROXY=https://proxy.golang.org,direct go get github.com/gin-gonic/gin`

`GODEBUG=netdns=go GOHOST=localhost GOINSECURE=nullprogram.com GONOSUMDB=nullprogram.com GO111MODULE=on go get -v github.com/gin-gonic/gin`

#### cobra vs multiple cmd folders

در صورتی که برنامه ی ما بیش از یک حالت ران شود یعنی همزمان هم rest api http  داره هم چند کرون جاب  ، می تونیم ۲ کار کنیم :

حالت اول این که چند cmd  و دایرکتوری داشته باشیم و درون هر کدوم ، main جدا ، خوبیش اینه خیلی مستقل هستن و نیاز به پکیج بیرونی نداره مانند کبرا همچنین به ازای هر کدوم ، یه داکر فایل باید داشته باشیم

حالت دوم استفاده از کبرا هست ، کلن یه بار بیلد میشه و یه فایل باینری داریم ولی در عوض می تونه با پارامتر هایی که میگیره ، 



#### time.Time
این تایپ برای زمانه و اگر استرینگ با فرمت زیر بدیم ، می توینم unmarshal کنیم
```
type Event struct {
    Name      string
    Timestamp time.Time
}
...
    jsonStr := `{"Name":"Birthday","Timestamp":"2024-04-27T12:00:00Z"}`
    var event Event
    err := json.Unmarshal([]byte(jsonStr), &event)
```

#### json unmarshal

وجه شود در صورتی که فیلد های استراکت پرایویت باشد ، اگر اینستنسی از آن مارشال شود ، فیلد های پرایویت برگردانده نمی شود ، پس با دقت فیلد ها را پرایوت کن . ( کلا پرایوت جالب نیست )

redis - json ------- اینجوری می تونیم دیکد کنیم :



	result := new(interface{})
	json.Unmarshal([]byte(cacheResponse.(string)), result)
 

#### json.Decoder

+ یکی از ابزار های خوب برای کار با جیسون آنمارشال است اما گاهی جیسون ما خیلی بزگ است و یا گاهی می خواهیم بخشی از جیسون رو استفاده کنیم در این صورت نیازی نیست تمامی جیسون را در مموری نگه داریم و راه حل  استفاده از json.Decoder




### golang conventional naming package

متاسفانه مرسومه که نام پکیج ها `lowercase` باشه و از `-` or `_` استفاده نکنیم

و این شد که من باید اسم پکیجم رو این بزارم :

`internalrequestservice`
