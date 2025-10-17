
## runtime

سیستم زمان اجرا Go نقش مهمی در مدیریت حافظه، زمان‌بندی برنامه‌ها، پشتیبانی از همزمانی، ادغام با سیستم عامل و ارائه خدمات ضروری دارد که برنامه‌های Go را کارآمد، ایمن و قابل حمل در پلتفرم‌های مختلف می‌کند. 

توجه شود با اینکه گولینگ در دسته ی زبان های کامپایلی قرار میگیرد اما زمانی که از رانتایم استفاده می کنیم جنبه ی interpretation دارد
 
- **Memory Management**

با استفاده از گاربج کالکتور مدیریت می کنه حافظه های اشغال شده 

توجه شود هر گوروتین استک خودش را دارد و با ۲ KB شروع میشود اگر حجم استک بیشتری بخواد ، runtime این بخش رو مدیریت کرده و به گوروتین اختصاص میده ، الگوریتمش به این روشه که دو برابر می ده ، محتوای قبلی رو کپی می کنه تو استک جدید ، همچنین GC وظیفه ی حذف استک را دارد همچنین گاهی runtime  می تونه حجم استک رو **Stack Shrinking** کن تا اپتیمایز کنه

اگر این حجم بیش از حد بزرگ شه  استک اور فلو میشه و پنیک میزنه
- **Goroutine Management**

با استفاده از اسکجولر ، ترد هایی **M** که سیستم عامل به برنامه داده را به پراسسور **P** های مجازی که چندین گوروتین را مدیریت کرده و اجرا می کند ، میدهد

- **Go Type System**

تایپ های داینامیکی را مدیریت می کند مانند type assertions, reflection, and interfaces.


- **Channel Communication**

ساخت ، ارسال و دریافت داده در چنل ها یکی دیگر از وظایف رانتایم است

- **Package Management**

اینیت کردن پکیج ها منظور کد های یک فولدر ، همچنین مدیریت کردن پیش نیاز ها با استفاده از ایمپورت  وظیفه ران تایم است

- **Error Handling and Panic Recovery**

مکانیزم ریکاوری پنیک و هندل کردن ارور

- **Integration with OS**

بیشتر مرتبط با درخواست ترد ها از سیستم عامل است

- **Ahead Of Time compilation(AOT  vs JIT )**

 کامپایلر گو آن را به زبان ماشین کامپایل می کند اما `JIT` بعضی زبان ها (مانند جاوا) ابتدا به زبان میانی کامپایل کرده و برای اجرا توسط `runtime engine` اجرا میشه


## reflection

 
زبان گو بیست استاتیک تتیپ هست و بهتره compile time  کداش اجرا شه ، اما با استفاده از reflect می شه ایجاد تایپ یا تعریف تایپ در runtime  اجرا بشه و چند تا بدی داره 

+ سرعت و پرفورمنس گو رو میگیره
+ تایپ سیف نیست و نمیشه روی تایپاش اطمینان کرد
+ خوانایی کد رو خیلی پایین میاره و پیچیده میکنه کد رو
+ توی تایپ های پریویت _ کمل کیس _ نمیتونه کار کنه

**اما مجبوریم جاهای زیر ازش استفاده کنیم**

+ مارشال و آنمارشال
+ جنریک پروگرم
+ جنریک دیتا استراکچر
+ خوانایی





## tips


+ زمانی که یه استک ساخته میشه و مقدارش کمه ، توسط runtime بزرگ میشه

+ اگر یه جایی باید یه اینترفیس رو ایمپلمنت می کردیم ، حال نیاز به فیلد های آن استراکت داشتیم ، مثال یه error custom  در نظر بگیرید
درون استراکت علاوه بر مسیج ارور میخواهیم فیلد status code هم بزاریم برای ارور، در این صورت تنها راه رسیدن به این فیلد، استفاده از تایپ اسرشن 
هست



### fmt

توجه شوداین پکیج از رفلکشن استفاده میکند ، به این معنی که برای همه ی ورودی هایی که بهش میدیم ، سوییچ کیس تایپش رو در میاره پس در جاهایی از کد که زیاد استفاده میشه ، بهتره جایگزینش رو استفاده کنیم
مثلن زمانی که می خوایم عدد رو تبدیل به اسرینگ کنیم بهتره ازstrconv استفاده شه



با استفاده از این قابلیت در runtime  می توانیم نوع و مقدار و تمامی اطلاعات یک interface{}  را مشخص کنیم


    v := reflect.ValueOf(data)   // مقدار data را دریافت می کنیم


    t := v.Type()   // نوع data را مشخص می کنیم


    if v.Kind() != reflect.Func   // اگر نوع data یک function باشد

    // با استفاده از kind می توانیم می توانیم تایپ مورد نظر رو در بیاریم

    if v.Type().NumIn() != 1  // در صورتی که تایپ داده ، فانکشن باشد ، تعداد ورودی را چک می کنیم

    if v.Type().NumOut() != 2 // در صورتی که تایپ داده ، فانکشن باشد ، تعداد خروجی را چک می کنیم

    reqParamType := v.Type().In(0) // در صورتی که تایپ داده ، فانکشن باشد ، نوع ورودی را مشخص می کنیم ، در اینجا اولین ورودی را مشخص می کنیم

    reqParam := reflect.New(reqParamType) // در اینجا با توجه به نوع که بالا مشخص شد ، یک اینستنس از آن نوع ایجاد می کنیم

    reqParamElem = reqParam.Elem() //  

    reqParamInterface = reqParam.Interface() //  


    result := v.Call(args)  // در صورتی که تایپ داده ، فانکشن باشد ، در اینجا این فانکشن را فراخوانی می کنیم

    result[0] // خروجی اولین فانکشن

### tip

+ **Type safety**

گاهی مشخص میشود ساختار ورودی و تایپ آن ، در این صورت  برنامه strict می کند داده رو به تایپ مورد نظر و از قبل از اجرای برنامه و زمان **compile time**  تایپ را مشخص می کند این باعث افزایش سرعت برنامه می شود ولی داینامیک بودن رو خیلی دشوار می کند



در صورتی که ورودی جیسون است اما ساختار آن را نمی دانیم ، به طور معمول آن را درون مپ می ریزیم:


    	var reqParamMap map[string]interface{}
	    err := json.Unmarshal([]byte(jsonData), &reqParamMap)







```


func GenericValidateAndUnmarshalFunc(data interface{}, jsonData string) {

	v := reflect.ValueOf(data)
	if v.Kind() != reflect.Func {
		fmt.Println("Input is not a function")
		return
	}
	if v.Type().NumIn() != 2 {
		fmt.Println("Function should have exactly 1 parameters")
		return
	}

	if v.Type().NumOut() != 2 {
		fmt.Println("Function should have NumOut exactly 2 parameters")
		return
	}

	reqParamType := v.Type().In(1)

	reqParam := reflect.New(reqParamType)

	reqParamInterface := reqParam.Interface()
	json.Unmarshal([]byte(jsonData), reqParamInterface)

	reqParamElem := reqParam.Elem()

	args := make([]reflect.Value, v.Type().NumIn())

	args[1] = reqParamElem
	ctx := context.Background()

	valueCtx := reflect.TypeOf(ctx)

	args[0] = reflect.New(valueCtx)

	result := v.Call(args)

	for _, v := range result {

		fmt.Printf("Result: %v, Type: %v\n", v, v.Type())

	}
}


```
