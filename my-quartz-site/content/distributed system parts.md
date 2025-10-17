
# system design

به تعریف و طراحی معماری سیستم ، کامپوننت ها ، اینترفیس ها - همان API  - ، دیتابیس می گن و با توجه به نیاز ها و بیزینس می شه تردآف هایی در نظر گرفت



### **Authentication and Authorization**

مدیریت یوزر ها و دسترسی به کامپوننت ها


این جا با یه سری مفاهیم مانند **sso** می تونیم از طریق اتورایزر مرکزی  دسترسی ها رو مدیریت کنیم

یکی از 

+ RBAC - role base access control

+ ABAC - Attrebute base access control

###  PEP  - policy enforcment point

مشخص می کنیم که در چه سطحی چک شود که دسترسی امکان پذیر است

+ لایه اول دسترسی به **api**  است ، خیلی سطحی است و مشکل این است که مثلا برای کاربر عادی باید یک api  باشه و برای کاربر طلایی و یا ادمین  **api**  جدا


+ سطح بعدی متد http  یا header  ها هستند . مثلا  - post , put , get , - header = scope

در این روش ها هم میتوان محدودیت ها را پیشرفته تر کرد

+ بررسی context request   در این حالت درخواست بررسی می شود و با توجه به محتوی بررسی می شود ، در این مرحله درخواست در لایه کد هندل میشود و کد نیاز به داده های سطح دسترسی از rbak  یا سرویس authorizer  دارد




tools keyclock



### **API Gateway**

بهتر است تنها یک entry point داشته باشیم برای مدیریت موارد زیر : 

Handles routing, authentication, authorization, rate limiting, load balancing, logging, and monitoring.

tools kong 

**tips**

 کد 429 در http به معنی rate limiting است

می شه توی ردیس ذخیره کرد

**algorithm**

### **token bucket**

هر ریکوست یک توکن داره و به ازای هر کاربر یک باکت در نظر میگیریم این باکت میتونه مثلن سقف ۱۰ توکن داشته باشه و هر در خواست یه دونه توش میندازه ، اگر پر شده اید صبر کنیم تا توکن ها اکسپایر شن و باکت خالی شه 




### **Service Registry and Discovery**

این سرویس مانند یک دیتابیس است که هر سرویس که بالا میاد اطلاعاتش رو نگه میداره ، مانند آدرسش توی شبکه  اینکه اینستنس از چه سرویسیه ، هلت چک و اطلاعات متا دیتا از اینستنس

کمتر مورد استفاده است و وظیفش اینه که بتونیم به سرویس ها دسترسی داشته باشیم و اونها رو با یه نام مشخص کنیم

در حقیقت این یه سرویس هست که بتونیم به جای یه آدرس ، یه نام برای سرویس داشته باشیم و با یه نام مشخص کنیم و در صورتی که بخوایم این سرویس رو به جای یه سرویس دیگه قرار دهیم می تونیم اون رو به جای یه سرویس دیگه قرار دهیم 

کوبرنتیز این روی فراهم می کنه اما ابزار های دیگه هم هستند مثل tools zookeeper


ابزار هایی مانند Zookeeper - Consul - Eureka

### **Strangler Fig**

یه استراتجی برای مهاجرت از مونولیتیک به معماری جدید و به معنی انجیر خفه کننده است و یعنی آرام آرام درخت اصلی رو می کشه

### **CQRS(Command Query Responsibility Segregation)** میگه بیایم دستوراتی که چیزی به استوریج اضافه میکند یا تغییری ایجاد می کند را جدا کنیم از دستوراتی که داده را میخوانند 
 - - **command side** postgres - Cassandra - mongodb
 - - **query side** mongo - elk - redis


#### **Infrastructure Services**

یکم زیر ساختی تر :

### **Configuration Management**

یه کانفیگ مرکزی مثل والت

### **Monitoring and Logging**

Examples: Prometheus, Grafana, ELK

### **Message Brokers/Event Buses/Message queue**

asynchronous communication and event-driven architectures.

Examples: Apache Kafka, RabbitMQ

### **Load Balancers**

Examples: NGINX, HAProxy

### **Health Checking**

Examples: Kubernetes Liveness

### **Container Orchestration**

مدیریت کامپوننت ها / کانتینر ها و سرویس ها مقیاس پذیری ، اتوماتیک دیپلوی

Examples: Kubernetes, Docker Swarm

### **Service Mesh**
  
example tools :consul
مدیریت  ارتباط سرویس ها با هم  مانند سیکیوریتی ، قابلیت مشاهده ی یک دیگر سرویس ها ،  و ترافیک بین آنها

