### config

 گاهی از کانفیگ در تست استفاده میکنیم ، گاهی در پروداکت ، گاهی بعضی از فیلد هاآپشنال است و میتوان دیفالت هم مقدار دهی کرد ، گاهی الزامی است و در صورت پر نکردن باید خطا برگرداند ، همچنین توجه شود در یونیت تست نیاز است تنها کانفیگ همان لایه کامل گردد و نیاز به تمامی env  نیست

در صروتی که کانفیگ nested  هست ، بهتر است از جیسون یا یمل استفاده کرد ولی به صورت کلی بهتر است از env  استفاده کرد

با توجه به موارد بالا best practice باید به صورت زیر ببینیم

+ از استراکت هماره استفاده کنیم

کانفیگ بهتر است env  باشد اما برای valid کردن یا نگه داری و پاس دادن بهتر است در struct ریخته شود

```go
import "github.com/caarlos0/env/v11"

type DatabaseConfig struct {
    URL  string `env:"DATABASE_URL,required"`
    User string `env:"DATABASE_USER" envDefault:"3000"`
    Pass string `env:"DATABASE_PASS"`
}

// Define a struct for server settings
type ServerConfig struct {
    Port string `env:"SERVER_PORT" envDefault:"3000"`
    Host string `env:"SERVER_HOST" envDefault:"3000"`
}

// Define a struct for the main configuration with embedded structs
type Config struct {
    Database DatabaseConfig
    Server   ServerConfig
    LogLevel string `env:"LOG_LEVEL"`
}
```
+ Default and required Values

میتونیم جاهایی که الزامیه با خطا نشان دهیم و در صورتی که مقادیر پیش فرض می شه گذاشت ، ست کنیم

توجه شود مقدار دیفالت در env ست نمیشود بلکه در استراکت پر می شود
```
    cfg := &Config{}
    if err := env.Parse(cfg); err != nil {
        return err
    }
```
+ Integration with Dependency Injection

حتی اگر از env  استفاده می کنیم ، بهتره در برنامه به صورت مستقیم از os.get استفاده نکنیم و اون رو پاس بدیم ، به این صورت مکی فهمیم هر لایه چه کانفیگ هایی احتیاج داره

bad :
```go
...
os.get("SAMPLE_ENV")
...
```

ok :
```go
    cfg := config.LoadConfig()
    // Pass cfg to your application components
    app := NewApp(cfg)
```

+ Environment-Specific

می تونیم دو محیط در نظر بگیریم ، توسعه و پروداکت  ، می تونیم شرط بزاریم در صورتی که محیط توسعه بود ، یه سری از env هارو بگیر

مثلن یه گوروتین که منابع رو مانیتور میکنه یا یه بخشی از کد که تنها در  توسعه قراره خونده بشه

+ test in maani
در مانی ، از env والت برای تست استفاده نمیشه ، بجاش یه جیسون برای تست استفاده کنه 

باید ابتدا فایل جیسون رو بخونیم ، سپس به ترتیب اون رو os.setENV بکنیم و طبق روال عادی با کتابخونه اون رو پارس کنیم

### maani

توی حصین والت داریم ، و برای هر سرویس چند تا مجموعه env  دارم ، مثلا openbank_dev , openbank_local , openbank_production


و یه متغییر به نام app_env  دارم که نشون میده محیط دولوپ هست یا پروداکشن ، چون توی کد هر جا تستی باشه ، حتی بر روی لاجیک هم تاثیر میزاره
