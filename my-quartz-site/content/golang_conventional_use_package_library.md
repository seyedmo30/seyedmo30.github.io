# repository - db

با این که به نظرم orm  در golang  زیاد خوب نیست اما جاهایی باید استفاده شه

### gorm

زمانی که بخواییم کوییری های داینامیک ، قیلتر ها و سلکت های داینامیک بزنیم مانند ادمین  یا آپشنال ، بهتره از یه کوییری بیلدر استفاده کنیم و به نظرم gorm  علاوه بر میگریشن و فانکشن هایی مانند first میشه کوییری بیلدرش هم استفاده کرد

در این کوییری های داینامیک بهتره ۲ تا نکته رو رعایت کنیم

+ Use Structs for Filters

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
+ Use Slices for Select Fields

``` go

// Select specified fields
if len(fields) > 0 {
    query = query.Select(fields)
}


```

+ single database connection
میشه به جای کانکشن های مختلف برای prepared statement و dynamic query ... یه کانکشن یکتا استفاده کرد 

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





### **swagger-echo**

این مثال بر اساس اکو نوشته شده است

ابتدا باید برای تابع مین داکیومنت های زیر را اضافه کنیم

```


    import (
    "github.com/swaggo/echo-swagger"
    _ "aggrigation/docs"
    )

    // @title          title API
    // @version        1.0
    // @description    description.com
    // @query.collection.format multi
    func main() {
    	e.GET("", func(c echo.Context) error { return c.Redirect(http.StatusSeeOther, "/swagger/index.html") })

	    e.GET("/swagger/*", echoSwagger.WrapHandler)
    
    }
```






# auth - rbac

فرض کنیم سرویس مونولیتیک باشه و یا قرار سرویس authorizer  رو ما بنویسیم ، حال نیاز به احراز هویت و تولید و اعتبار سنجی توکن و رول های مختلف به یوزر ها و گروه ها باشه.


### casbin






## provider

گاهی قراره از پروایدر استفاده کنیم حال چندین راه برای راحتی و رفع مشکل باید در نظر گرفت

+ **source of truth**

همیشه باید مشخص کرد اینو ، در اکثر موارد پروایدر هست

+ **Synchronization Strategy**

می تونیم به ۲ روش این رو بر قرار کنیم

+ + **schedule**

در بازه های زمانی مانند شب ها چک شود و مقاییرت ها گرفته شود 

+ + **event-driven**

بر اساس منطق  درست و دید همه ی جوانب ، نیازی به اسکجولر نیست اما اطمینان داریم که سینک است 

+ + **state-less**

تا جای ممکن اطلاعات ذخیره نکنیم و با api call  ها کار کنیم


+ **Graceful Degradation**

حتی در صورتی که پروایدر ها پایین بیایند ، بهتره به جای درخواست از آن نیاز هایی که بدون آن هم می توان پاسخ داد را داشته باشیم

مثلن شاید نتوان سفته ی جدید را در صورت در دسترس نبودن پروایدر انجام داد ، اما می شه لیست سفته های موجود رو از repo  داخلی پاسخ داد

