# Runtime

سیستم زمان اجرای Go نقش بسیار مهمی در مدیریت حافظه، زمان‌بندی، پشتیبانی از هم‌زمانی، تعامل با سیستم‌عامل و ارائه‌ی سرویس‌های ضروری دارد. در واقع، `runtime` باعث می‌شود برنامه‌های Go سریع، ایمن و قابل حمل در پلتفرم‌های مختلف باشند.  

نکته‌ی مهم این است که با وجود اینکه Go یک زبان **کامپایلری** است، استفاده از `runtime` جنبه‌هایی از **interpretation** را هم به برنامه اضافه می‌کند.

## Memory Management

مدیریت حافظه در Go توسط **Garbage Collector** انجام می‌شود. هر goroutine دارای یک استک مستقل است که با حدود **2KB** آغاز می‌شود. اگر گوروتین به حافظه‌ی بیشتری نیاز داشته باشد، `runtime` استک را **دو برابر** کرده و داده‌ها را در استک جدید کپی می‌کند.  
این مکانیزم به کمک الگوریتم **Stack Growth** و **Stack Shrinking** انجام می‌شود تا عملکرد بهینه‌ای داشته باشد. اگر حجم استک بیش از حد شود، برنامه دچار **Stack Overflow** شده و پنیک می‌کند.

## Goroutine Management

زمان‌بندی و اجرای گوروتین‌ها توسط **Scheduler** در runtime انجام می‌شود. این زمان‌بند، تردهای سیستم‌عامل (**M**) را به پردازنده‌های مجازی (**P**) اختصاص می‌دهد و هر پردازنده مجموعه‌ای از گوروتین‌ها را مدیریت و اجرا می‌کند. این ساختار باعث مدیریت بهینه‌ی هم‌زمانی در Go می‌شود.

## Go Type System

Runtime وظیفه‌ی مدیریت تایپ‌های **داینامیک** را دارد؛ مانند `type assertions`، `reflection` و `interfaces` که در زمان اجرا تفسیر و بررسی می‌شوند.

## Channel Communication

ساخت، ارسال و دریافت داده در **کانال‌ها** نیز توسط runtime مدیریت می‌شود تا هماهنگی بین گوروتین‌ها حفظ شود.

## Package Management

مدیریت و **initialize** کردن پکیج‌ها (کدهای داخل یک فولدر) و بررسی وابستگی‌های آنها در هنگام **import** بر عهده‌ی runtime است.

## Error Handling and Panic Recovery

Runtime مکانیزم **panic** و **recover** را برای مدیریت خطاها فراهم می‌کند تا برنامه بتواند پس از وقوع خطا به‌صورت کنترل‌شده ادامه یابد.

## Integration with OS

بخش دیگری از وظایف runtime ارتباط با سیستم‌عامل است؛ مانند درخواست ترد، مدیریت سیگنال‌ها و ارتباط با منابع سطح پایین.

## Ahead Of Time Compilation (AOT) vs Just In Time (JIT)

در Go، کدها به‌صورت **AOT** مستقیماً به زبان ماشین کامپایل می‌شوند. در مقابل، زبان‌هایی مثل Java از **JIT** استفاده می‌کنند؛ یعنی ابتدا کد را به زبان میانی کامپایل کرده و سپس در زمان اجرا آن را توسط `runtime engine` تفسیر و اجرا می‌کنند.

---

# Reflection

Go زبانی **استاتیک تایپ** است، یعنی بیشتر بررسی‌ها در **compile time** انجام می‌شوند. با این حال، با استفاده از **reflect** می‌توان در زمان اجرا (`runtime`) نوع داده‌ها را بررسی یا ایجاد کرد.  
این قابلیت قدرتمند است اما چند ضعف دارد:

- سرعت و پرفورمنس برنامه را پایین می‌آورد  
- تایپ سیف نیست (امکان خطاهای زمان اجرا وجود دارد)  
- خوانایی کد را کاهش می‌دهد و دیباگ را سخت می‌کند  
- روی فیلدهای **private** (با نام کوچک یا camelCase) کار نمی‌کند  

### کاربردهای اصلی reflect

- **marshal / unmarshal** داده‌ها  
- **generic programming**  
- ساختارهای داده‌ی عمومی (generic data structures)  
- خوانایی و تحلیل داده‌های داینامیک  

---

# Tips

- زمانی که استک کوچک است، `runtime` به‌صورت خودکار آن را بزرگ‌تر می‌کند.  
- وقتی نیاز به دسترسی به فیلدهای یک struct از طریق interface داریم (مثلاً در custom errorها)، باید از **type assertion** استفاده کنیم.

---

## fmt Package

پکیج `fmt` از **reflection** استفاده می‌کند. یعنی برای هر ورودی نوع آن را بررسی می‌کند و بر اساس تایپ، آن را چاپ می‌کند. به همین دلیل در جاهایی که پرفورمنس مهم است، بهتر است از جایگزین‌هایی مثل `strconv` استفاده شود.  
مثلاً برای تبدیل عدد به رشته به‌جای `fmt.Sprintf` می‌توان از `strconv.Itoa` استفاده کرد.

با استفاده از قابلیت reflect می‌توان نوع و مقدار یک `interface{}` را مشخص کرد:

```go
v := reflect.ValueOf(data)   // مقدار را می‌گیرد
t := v.Type()                // نوع داده را مشخص می‌کند

if v.Kind() != reflect.Func {
    // بررسی می‌کند که داده از نوع function نباشد
}

if v.Type().NumIn() != 1 {
    // بررسی تعداد پارامترهای ورودی
}

if v.Type().NumOut() != 2 {
    // بررسی تعداد خروجی‌ها
}

reqParamType := v.Type().In(0)           // نوع اولین ورودی
reqParam := reflect.New(reqParamType)    // ایجاد نمونه‌ای از نوع مورد نظر
reqParamElem := reqParam.Elem()
reqParamInterface := reqParam.Interface()
result := v.Call(args)                   // فراخوانی فانکشن
fmt.Println(result[0])                   // نمایش خروجی اول
```

---

### Type Safety

در Go نوع داده‌ها در زمان **compile time** مشخص می‌شود، بنابراین برنامه سریع‌تر و امن‌تر اجرا می‌شود، اما انعطاف‌پذیری کمتری دارد.  
اگر ورودی JSON باشد و ساختار آن را ندانیم، معمولاً آن را درون یک `map[string]interface{}` قرار می‌دهیم:

```go
var reqParamMap map[string]interface{}
err := json.Unmarshal([]byte(jsonData), &reqParamMap)
```

---

## نمونه کد کامل Reflect

```go
func GenericValidateAndUnmarshalFunc(data interface{}, jsonData string) {
	v := reflect.ValueOf(data)
	if v.Kind() != reflect.Func {
		fmt.Println("Input is not a function")
		return
	}
	if v.Type().NumIn() != 2 {
		fmt.Println("Function should have exactly 2 parameters")
		return
	}
	if v.Type().NumOut() != 2 {
		fmt.Println("Function should have exactly 2 return values")
		return
	}

	reqParamType := v.Type().In(1)
	reqParam := reflect.New(reqParamType)
	reqParamInterface := reqParam.Interface()
	json.Unmarshal([]byte(jsonData), reqParamInterface)

	reqParamElem := reqParam.Elem()
	args := make([]reflect.Value, v.Type().NumIn())

	ctx := context.Background()
	args[0] = reflect.ValueOf(ctx)
	args[1] = reqParamElem

	results := v.Call(args)
	for _, r := range results {
		fmt.Printf("Result: %v, Type: %v\n", r, r.Type())
	}
}
```
