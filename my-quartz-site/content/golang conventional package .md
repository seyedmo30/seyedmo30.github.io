# Repository - DB

با این که ORM در Go همیشه کامل‌ترین راه نیست، اما در برخی موارد کاربرد دارد.

## GORM

برای کوئری‌های داینامیک، فیلترها و سلکت‌های اختیاری، استفاده از Query Builder توصیه می‌شود. GORM علاوه بر میگریشن و فانکشن‌هایی مثل `First`، امکان استفاده از Query Builder را نیز فراهم می‌کند.

### Best Practices

- **Use Structs for Filters**

```go
// Define the UserFilter Struct
type UserFilter struct {
	Name  *string
	Age   *int
	City  *string
	Email *string
}

// Add filters dynamically
if filter.Name != nil {
    query = query.Where("name LIKE ?", "%"+*filter.Name+"%")
}
if filter.Age != nil {
    query = query.Where("age = ?", *filter.Age)
}
```

- **Use Slices for Select Fields**

```go
// Select specified fields
if len(fields) > 0 {
    query = query.Select(fields)
}
```

- **Single Database Connection**

می‌توان به جای کانکشن‌های مختلف برای Prepared Statement و Dynamic Query، از یک کانکشن یکتا استفاده کرد.

```go
func connectDB() (*sql.DB, *gorm.DB, error) {
	dsn := "username:password@tcp(localhost:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
	sqlDB, err := sql.Open("mysql", dsn)
	if err != nil {
		return nil, nil, err
	}

	gormDB, err := gorm.Open(mysql.New(mysql.Config{Conn: sqlDB}), &gorm.Config{})
	if err != nil {
		return nil, nil, err
	}

	return sqlDB, gormDB, nil
}
```

## Swagger - Echo

برای مستندسازی API با Echo و Swagger، ابتدا باید داکیومنت‌های زیر به تابع `main` اضافه شود:

```go
import (
    "github.com/swaggo/echo-swagger"
    _ "aggrigation/docs"
)

// @title          title API
// @version        1.0
// @description    description.com
// @query.collection.format multi
func main() {
	e.GET("", func(c echo.Context) error { 
        return c.Redirect(http.StatusSeeOther, "/swagger/index.html") 
    })

	e.GET("/swagger/*", echoSwagger.WrapHandler)
}
```

# Auth - RBAC

فرض کنیم سرویس مونولیتیک باشد یا بخواهیم سرویس Authorizer خود را پیاده کنیم. در این صورت نیاز به احراز هویت، تولید و اعتبارسنجی توکن و مدیریت رول‌های مختلف برای کاربران و گروه‌ها وجود دارد.

## Casbin

برای مدیریت دسترسی و نقش‌ها می‌توان از Casbin استفاده کرد.

## Provider

گاهی نیاز است از Provider استفاده کنیم. برای مدیریت صحیح آن نکات زیر مهم هستند:

- **Source of Truth**  
همیشه باید مشخص شود که منبع اصلی داده چیست؛ اغلب Provider است.

- **Synchronization Strategy**  
راه‌های هماهنگ‌سازی:
    - **Schedule:** بررسی دوره‌ای، مثلاً شب‌ها.
    - **Event-driven:** بدون نیاز به زمان‌بندی، با اطمینان از همگام بودن داده‌ها.
    - **State-less:** تا جای ممکن اطلاعات ذخیره نشود و مستقیماً با API کار شود.

- **Graceful Degradation**  
حتی اگر Providerها در دسترس نباشند، بهتر است نیازهایی که بدون آن‌ها قابل پاسخ هستند را فراهم کنیم.  
مثلاً در صورت عدم دسترسی به Provider برای ایجاد سفته جدید، می‌توان لیست سفته‌های موجود در Repository داخلی را ارائه کرد.
