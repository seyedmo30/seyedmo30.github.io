# Config

در برنامه‌ها گاهی از کانفیگ در تست استفاده می‌کنیم، گاهی در محیط پروداکشن و گاهی بعضی از فیلدها آپشنال هستند و می‌توان مقدار پیش‌فرض هم داد. در صورت پر نکردن فیلدهای الزامی باید خطا برگردانده شود. در یونیت تست، معمولاً تنها کانفیگ همان لایه کامل می‌شود و نیازی به تمامی ENV ها نیست.

در صورتی که کانفیگ Nested باشد، بهتر است از JSON یا YAML استفاده شود، اما به طور کلی استفاده از ENV ترجیح داده می‌شود.

بدونیم که هر بار getenv  میکنیم هر بار (io cost)  نداره و خیلی هزینش ناچیزه نسبت به وریبل توی گولنگ اما خیلی تمییز تره از کانفیگ استفاده کنیم


## Best Practice

- **استفاده از Struct:**  
کانفیگ بهتر است از ENV خوانده شود، اما برای اعتبارسنجی، نگهداری و پاس دادن به لایه‌ها بهتر است در Struct ریخته شود.

```go
import "github.com/caarlos0/env/v11"

type DatabaseConfig struct {
    URL  string `env:"DATABASE_URL,required"`
    User string `env:"DATABASE_USER" envDefault:"3000"`
    Pass string `env:"DATABASE_PASS"`
}

// Server settings
type ServerConfig struct {
    Port string `env:"SERVER_PORT" envDefault:"3000"`
    Host string `env:"SERVER_HOST" envDefault:"3000"`
}

// Main configuration with embedded structs
type Config struct {
    Database DatabaseConfig
    Server   ServerConfig
    LogLevel string `env:"LOG_LEVEL"`
}
```

- **Default and Required Values:**  
مقادیر الزامی با خطا نشان داده می‌شوند و مقادیر پیش‌فرض در Struct پر می‌شوند، نه در ENV.

```go
cfg := &Config{}
if err := env.Parse(cfg); err != nil {
    return err
}
```

- **Integration with Dependency Injection:**  
بهتر است در برنامه به صورت مستقیم از `os.Get` استفاده نشود و کانفیگ به لایه‌ها پاس داده شود.

**Bad:**
```go
os.Get("SAMPLE_ENV")
```

**OK:**
```go
cfg := config.LoadConfig()
// Pass cfg to your application components
app := NewApp(cfg)
```

- **Environment-Specific:**  
می‌توان محیط‌های توسعه و پروداکشن را در نظر گرفت و شرط گذاشت که برخی ENVها تنها در توسعه استفاده شوند. مثال: گوروتینی که منابع را مانیتور می‌کند یا بخشی از کد که فقط در توسعه اجرا می‌شود.

- **Test in Maani:**  
در محیط مانی، برای تست از ENV استفاده نمی‌شود. بجای آن، JSON برای تست خوانده شده و به ترتیب با `os.SetEnv` تنظیم می‌شود و طبق روال عادی با کتابخانه پارس می‌شود.

## Maani Example

در حصین والت، برای هر سرویس چند مجموعه ENV داریم، مثل:  
`openbank_dev`, `openbank_local`, `openbank_production`  

و متغیری به نام `app_env` داریم که مشخص می‌کند محیط توسعه است یا پروداکشن. این متغیر حتی بر روی لاجیک تست‌ها نیز تاثیر دارد.
