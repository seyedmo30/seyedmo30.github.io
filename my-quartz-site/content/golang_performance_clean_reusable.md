
## سوالی که توی مصاحبه می پرسن زمانی که کد سنگین میشه یا مموری لیک داریم و می خوایم مشکل رو رفع کنیم 

+ pprof

+ Logging

+ Monitoring

+ Benchmarking

+ Concurrency Issues


## tips

### comment

برای پارامترو های ورودی می توان کامت هم گذاشت

### struct

 اگر ترتیب فیلد ها در استراکت بر اساس سایز باشند ، می توان مموری کمتری استفاده کرد


### defer close Resource Leakage

در مواردی که یک body یا connection یا به طور کل موارد زیر باز می شود باید بسته شود تا منابع آزاد شود :

+ file
+ network connections
+ database handles

در صورتی که کلوز نشن ، تا زمانی که برنامه خاتمه یابد یا منابع پر شود در حافظه می مونن



### ambient context ( Service Locator anti-pattern )  

به این معنی که به جای این که ورودی ها و ومتغییر هایی که فانکشن با اون سر و کار داره رو مستقیم از ورودی بگیریم ، از کانتکس یا از گلوبال بگیریم . اصطلاحا تو کیف میریزی و برمی داریم - key/value bag - و نتایجش اینه که :
+ حوانایی به شدت پایین میاد و کد پیچیده می شه
+ تاید کاپل اتفاق می افته و همه جا وابسته به کانتکس جین هستیم
+ تست نویسی سخت می شه چون معلوم نیست از کجا میان
+ خطا در ران تایم می گیریم
+ درIDE  خیلی سخته تریس کردن کد ( البته راه حل اینه به جای کلید استرینگ از تایپ استفاده کنیم )

### validator

+ سعی بشه تا جای امکان با استفاده از playground  از تگ های ولید کردن استفاده شه 
+ گاهی از DTO  مشترک استفاده میشه ، تو یکی ولیده و توی دیگری آپشناله ، بهتره پلید توی کد با استفاده از متد ولید دستی صورت بگیره
+ در کل بهتره اگر ولیدیشن خیلی پیچیده بود ، توی یه فانکشن صورت بگیره و کد لاجیک کوچیک باشه.


### Domain Primitives (domain-specific types)

به نظرم یه کار خفن اینه که از تایپ های خاص به جای استرینگ  یا اینت در **dto**  ها استفاده کنیم و مزیتاش اینه که اگر از یه مفهوم در چندین dto  استفاده می کنیم و اونا از **entity-domain** استفاده کنیم و بخواهیم اون رو حذف کنیم ، بتونیم تایپ رو حذف کنیم و همه جا که استفاده میشده ، خطا گرفت

توجه شود مشکلاتی مانند بیشتر شدن حجم کد و اینکه شاید چندین تایپ نیاز داشته باشند نام یکسان داشته باشند اما میبایست نام یونیک انتخاب کنیم 

```go

type (
	BankType string
	NationalCodeType string
	SourceAccountType string
 )

type BaseGetAccountBalanceRequest struct {
	Bank          BankType         `json:"bank" validate:"required"`
	NationalCode  NationalCodeType `json:"nationalCode" validate:"required"`
	SourceAccount SourceAccountType `json:"sourceAccount" validate:"required"`
}
```
 
### golang_memory_leak
توجه شود مهم ترین کار استفاده از  pprof  است

همچنین برای مشاهده مموری لیک در گوروتین ها میتونیم از -race هم استفاده کنیم


همچنین میتونیم از تست هایی که نوشتیم ، لود تست هم بگیریم

Go test -bench

از کتابخونه خوب برای تولید لاگ مثل zap , logrus باید استفاده بشه

لاگ هایی شامل info , warning , error  باید برن توی سنترالایز لاگ مثل الستیک


همچنین مموری متریک هایی باید نوشت و اون ها رو به پرومتیوس و گرافانا داد  

```
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promhttp
```

آموزش داره ، ولی یه api  میدیم و اون تو هر با کال بشه ، memalloc رو میفرستیم ، اینجوری metric  رو هم دادیم


### Enum with a Map vs Direct Constant String


```go

// Enum with a Map
const (
	sadadGetAccountBalance routeSadadName = iota
	sadadTransfer
	sadadTransactionInquiry
	sadadTransferValidation
	sadadGetIbanInfo
)

var routesSadad = map[routeSadadName]string{
	sadadGetAccountBalance:  "/v0.3/obh/api/aisp/get-account-balance",
	sadadTransfer:           "/v0.3/obh/api/pisp/transfer",
	sadadTransactionInquiry: "/v0.3/obh/api/pisp/transaction-inquiry",
	sadadTransferValidation: "/v0.3/obh/api/pisp/transfer-validation",
	sadadGetIbanInfo:        "/v0.3/obh/api/aisp/get-iban-info",
}

// Direct Constant String

const (
	sadadGetAccountBalance  = "/v0.3/obh/api/aisp/get-account-balance"
	sadadTransfer           = "/v0.3/obh/api/pisp/transfer"
	sadadTransactionInquiry = "/v0.3/obh/api/pisp/transaction-inquiry"
	sadadTransferValidation = "/v0.3/obh/api/pisp/transfer-validation"
	sadadGetIbanInfo        = "/v0.3/obh/api/aisp/get-iban-info"
)

```


### refactor Switch Statement to map

 اگر دیدیم چندین متغییر شبیه به هم با نام های متفاوت داریم ، شاید بشه  یه مپ گذاشت

+ bad

```go

type sadadService struct {
	httpService                 interfaces.HttpService
	config                      config.App
	bank                        string
	scopeTokenMoneyTransfer     SadadAccessToken
	scopeTokenAccount           SadadAccessToken
	scopeTokenSvcMgmtMqStmtInfo SadadAccessToken
	mu                          sync.Mutex // mu is a mutex to protect concurrent access to the AccessToken.
}
```

+ good
```go
type sadadService struct {
	httpService interfaces.HttpService
	config      config.App
	bank        string
	scopeToken  map[string]SadadAccessToken
	mu sync.Mutex // mu is a mutex to protect concurrent access to the AccessToken.
}


```
## نکات کتاب ۱00 اشتباه

### ۱ با احتیاط از مقدار دهی سریع استفاده کنیم

گاهی ممکن است یک داده چند بار مفدار دهی شود، بهتر است در این شرایط از مقدار دهی سریع استفاده نکینم


```go
var client *http.Client ❶
if tracing {
 client, err := createClientWithTracing() ❷
 if err != nil {
 return err
 }
 log.Println(client)
} else {
 client, err := createDefaultClient() ❸
 if err != nil {
 return err
 }
 log.Println(client)
}
```

کد زیر بهتر است

```go
var client *http.Client
var err error ❶
if tracing {
 client, err = createClientWithTracing() ❷
 if err != nil {
 return err
 }
} else {
 // Same logic
}

```






### ۲ برای خوانایی کد، بهتر است از کد های تو در تو پرهیز کنیم

در صورتی که امکان دارد می توانیم بجای استفاده از ایف و الس ، تنها از ایف استفاده کنیم


```go
func join(s1, s2 string, max int) (string, error) {
 if s1 == "" {
 return "", errors.New("s1 is empty")
 } else {
 if s2 == "" {
 return "", errors.New("s2 is empty")
 } else {
 concat, err := concatenate(s1, s2) 
 if err != nil {
 return "", err
 } else {
 if len(concat) > max {
 return concat[:max], nil
 } else {
 return concat, nil
 }
 }
 }
 }
}
```
به شکل زیربنویسیم

```go
func join(s1, s2 string, max int) (string, error) {
 if s1 == "" {
 return "", errors.New("s1 is empty")
 }
 if s2 == "" {
 return "", errors.New("s2 is empty")
 }
 concat, err := concatenate(s1, s2)
 if err != nil {
 return "", err
 }
 if len(concat) > max {
 return concat[:max], nil
 }
 return concat, nil
}
```



### ۳ باید با دقت از تابع اینیت استفاده کنیم ، زیرا چند مشکل احتمال دارد به وجود بیاید

الف ـ اینیت همیشه اجرا می شود و شاید کد ما در جایی که انتظار نداشته باشیم یا نیاز نباشد ، فرا خوانی شود و نا خواسته اینیت اجرا شود

ب ـ در اینیت نمی توانیم ارورهندلینگ را به خوبی پیاده سازی کنیم ، تنها می توانیم پنیک کنیم که شاید خیلی خطرناک باشد

ج ـ اینیت حتی در تست هم اجرا می شود ، در حالی که شاید ما می خواهیم با استفده از یونیت تست بخش کوچکی رو تست کنیم و اینیت رو نیاز نداریم اما اجرا می شود و شاید پنیک دهد

د ـ تابع اینیت چون نمی توان چیزی ریترن کرد ، می بایست در یک گلوبال وریبل داده خود را بریزد ، و این خود مشکل ساز می توان باشد زیرا هم توابع دیگر می توانند این وریبل را تغییر دهند ، هم در صورتی که یک تست بخواهد از این وریبل استفاده کند ، دیگر اوزوله نیست( یونیت نیست)

کدی که از اینیت بد استفاده کرده

```go
var db *sql.DB
func init() {
 dataSourceName :=
 os.Getenv("MYSQL_DATA_SOURCE_NAME") 
 d, err := sql.Open("mysql", dataSourceName)
 if err != nil {
 log.Panic(err)
 }
 err = d.Ping()
 if err != nil {
 log.Panic(err)
 }
 db = d 
}
```

و اصلاح آن

```go
func createClient(dsn string) (*sql.DB, error) { 
 db, err := sql.Open("mysql", dsn)
 if err != nil {
 return nil, err 
 }
 if err = db.Ping(); err != nil {
 return nil, err
 }
 return db, nil
}
```

### تمییزی در ورودی و خروجی های فانکشن ها

بهتره در مواردی که تعداد پارامتر های ورودی و خروجی در اکثر سیگینچر ها برابر است ، ورودی رو با نام **req** و خروجی رو با نام **res** نامگذاری کنیم

خوبیش اینه که می دونیم در تمامی سیگنیچر های مشابه اسم ورودی چیه و در صورت توسعه یا تغییر خیلی سرعت بیشتر میشه و دقدقه برای تغییر نام گذاری و انتخاب نام جدید نداریم

همچنین اگر برایاستراکت خروجی اسم نزاریم ، مجبوریم اون رو تعریف کنیم ، پس چه بهتره که تو امضای فانکشن اسم داشته باشه و تنها **return** رو برای خروجی بنویسیم

