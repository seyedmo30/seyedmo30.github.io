# مفاهیم پایه‌ای Go

در زبان Go، درک تفاوت بین مقدار، متغیر، نوع داده و ساختار داده اهمیت زیادی دارد.  
**value** یعنی خود مقدار، مثل `6` یا `"علی"`.  
**variable** ظرفی است که مقدار درون آن قرار می‌گیرد (مثل `const`, `let`, یا `var`).  
**data type** نوع ظرف را مشخص می‌کند، مثل `string` یا `int`.  
**data structure** مجموعه‌ای از چند نوع داده است، مثل `list`, `map`, `tree` یا `dictionary`.

---

## Mutable و Immutable Types

**Mutable data types** داده‌هایی هستند که با تغییر مقدارشان، مکانشان در حافظه تغییر نمی‌کند.  
مانند:  
`Slice`, `Array`, `Map`, `Channel`  

**Immutable data types** داده‌هایی هستند که در صورت تغییر، حافظه‌ی جدیدی به آن‌ها اختصاص داده می‌شود:  
`Boolean`, `Int`, `Float`, `Pointers`, `String`, `Interfaces`  

اگر در مورد immutability اعداد شک دارید، می‌توانید این لینک را مطالعه کنید:  
[Are ints and strings immutable in Go?](https://stackoverflow.com/questions/71589811/go-ints-and-strings-are-immutable-or-mutable/71590289#71590289)

---

## Type Casting و Conversion

**Type Casting** در Go وجود ندارد، اما می‌توان با **Conversion** نوع مقدار را هنگام قرار دادن در یک متغیر متفاوت تغییر داد.  
به‌عبارتی، اگر مقدار از یک نوع باشد ولی متغیری که می‌خواهیم در آن بریزیم نوع متفاوتی داشته باشد، از conversion استفاده می‌کنیم.

---

## Race Condition

اگر دو دستور به‌صورت همزمان بخواهند مقدار یک متغیر را تغییر دهند، **Race Condition** رخ می‌دهد.  
برای جلوگیری از آن از `mutex` استفاده می‌کنیم.

---

## Type Embedding

به این معناست که می‌توان یک نوع (مثلاً struct یا interface) را درون struct دیگر قرار داد.  
در این حالت فیلد embedded بخشی از struct بیرونی محسوب می‌شود و برای دسترسی به آن نیازی به نام‌گذاری مجدد نیست.  

تفاوت بین **struct embedding** و **field containment** در این است که در embedding، struct داخلی بخشی از struct بیرونی است و به‌صورت مستقیم قابل دسترسی است.  
در کار با JSON، structهای embedded کلید جداگانه‌ای در خروجی JSON ندارند.

```go
type Outer struct {
    Inner // Embedded struct
}

type GetOrderInfoBodyResponse struct {
    Message      string               `json:"message"`
    ResultCode   int                  `json:"resultCode"`
    ResponseBody GetOrderInfoResponse `json:"responseBody"` // This is a field, not embedding
}
```

---
## file embeding  

### embed.FS

یکی از کار هایی خفنی که هنگام خوندن فایل های استاتیک می شه انجام داد امبد کردن اون به یه وریبل هست . ۲ تا وبی داره ، یکی این که هر بار از دیسک خونده نمی شه ، دوم اینکه مشکل گولنگ اینه که آدرس های نسبی زمان توسعه و کامپایل متفاوته ، با این تکنیک تو هر صورتی فایل خونده می شه ، اگرم موقع توسعه آدرس درست نباشه ide  خطا میده

```go
//go:embed swagger-ui/*
var embedded embed.FS

// SwaggerUIFS returns an http.FileSystem rooted at the swagger-ui directory
func SwaggerUIFS() (http.FileSystem, error) {
	// swagger-ui/* files are embedded under "swagger-ui" (relative to this file)
	fsys, err := fs.Sub(embedded, "swagger-ui")
	if err != nil {
		return nil, err
	}
	return http.FS(fsys), nil
}
```


## Deadlock vs Blocking

زمانی که channel پر باشد و goroutine‌ها منتظر خالی شدن آن بمانند اما هیچ‌گاه این اتفاق نیفتد، **deadlock** رخ می‌دهد.  
همچنین اگر همه‌ی goroutine‌ها منتظر داده باشند و هیچ‌کدام نتوانند داده‌ای تولید کنند، باز هم deadlock ایجاد می‌شود.  

در صورتی که mutex قفل شود ولی هیچ goroutineی آن را آزاد نکند، باز هم deadlock است.  

اما **blocking** حالتی طبیعی است؛ مثلاً goroutine منتظر داده‌ی channel یا آزاد شدن mutex می‌ماند.  
تفاوت اصلی این است که اگر همه‌ی goroutine‌ها در حالت block باشند و هیچ‌کدام نتوانند دیگری را آزاد کنند، deadlock اتفاق می‌افتد.

---

## Function / Method Overloading

در Go قابلیت **Overloading** وجود ندارد؛ یعنی نمی‌توان چند تابع با نام یکسان ولی پارامترهای مختلف داشت.  
اما روش‌های جایگزین وجود دارد:

+ استفاده از **Variadic Functions**  
+ تعریف توابع مشابه با نام‌های کمی متفاوت (مثلاً `addInt`، `addFloat`)  
+ استفاده از **Receiverها** و پیاده‌سازی در **Interfaceها**

---

## Type Assertion

اگر بخواهیم نوع دقیق متغیری از نوع interface را مشخص کنیم:

```go
t, ok := i.(string)
```

در اینجا اگر `i` از نوع `string` باشد، `ok` برابر true خواهد بود.

---

## Function Prototype / Signature

ترکیب نام تابع، لیست پارامترها و نوع خروجی را **Function Signature** می‌گویند.

---

## Interface Compliance Assertion

برای اطمینان از اینکه یک struct واقعاً یک interface را پیاده‌سازی کرده است (در زمان کامپایل):

```go
var _ Shape = (*Rectangle)(nil)
```

---

## Blank Identifier (_)

اگر تابعی چند خروجی داشته باشد و بخواهیم بعضی از آن‌ها را نادیده بگیریم، از `_` استفاده می‌کنیم:

```go
_, res := Double(8)
```

---

## String Formatting with Placeholders

می‌توان داخل رشته‌ها با علامت `%` مقادیر را جایگزین کرد (مثل `fmt.Sprintf("%d", num)`).

---

## Variadic Functions

توابعی که تعداد نامحدودی ورودی می‌گیرند:

```go
func functionName(params ...Type)
```

این نوع پارامتر باید آخرین پارامتر تابع باشد. معمولاً زمانی استفاده می‌شود که پارامتر اختیاری باشد.

---

## Composite Literals

زمانی که یک data structure را ایجاد و هم‌زمان مقداردهی اولیه کنیم، از composite literal استفاده کرده‌ایم:

```go
arr := [3]int{1, 2, 3}
slice := []int{1, 2, 3}
bt := []byte("salam")
m := map[string]int{"Alice": 30, "Bob": 25}
gg := struct{ name string }{name: "ali"}
gg2 := struct{}{}
```

---

### Empty Struct

هیچ حافظه‌ای اشغال نمی‌کند.  
کاربردها:

+ به‌جای آرایه یا اسلایس برای ساخت مجموعه‌ای از داده‌ها با دسترسی سریع:
  ```go
  map_obj := make(map[string]struct{})
  ```
+ ارسال سیگنال در channelها (البته context روش بهتر است).

---

## GoPATH و GoROOT

`GoROOT` مسیر نصب زبان Go و ابزارهای داخلی آن است.  
`GoPATH` مسیر workspace کاربر و محل ذخیره‌ی پکیج‌ها و باینری‌هاست.

```bash
export GOPATH=$HOME/go
export GOROOT=/usr/local/go
```

نمونه مسیرها:

```
/home/seyed/go/pkg/mod/github.com/gin-gonic/        // Packages
/home/seyed/go/bin/swag                             // Compiled binaries
/usr/local/go/bin/go                                // Go compiler binary
/usr/local/go/src/fmt/                              // Built-in packages
```

---

## Go Modules Commands

+ **go install**  
  برای نصب ابزارهای CLI (بدون نیاز به go mod):
  ```
  go install github.com/rakyll/hey@latest
  ```

+ **go get**  
  برای نصب و به‌روزرسانی پکیج‌های ماژول:
  ```
  go get github.com/segmentio/kafka-go
  ```

+ **go mod tidy**  
  هماهنگ‌سازی پکیج‌های استفاده‌شده با `go.mod`:
  ```
  go mod tidy
  ```

+ **go mod download**  
  دانلود پکیج‌ها از `go.mod` (بدون تغییر آن، برای build آفلاین):
  ```
  go mod download
  ```

---

## Use Named Return Values

در Go می‌توان نام خروجی‌ها را در تعریف تابع مشخص کرد:

```go
func calculate(x, y int) (sum int, product int) {
    sum = x + y
    product = x * y
    return
}
```

---

## Variable Types

### Visibility Across Packages

+ **Exported variables** — با حرف بزرگ شروع می‌شوند و در پکیج‌های دیگر قابل مشاهده‌اند.  
+ **Unexported variables** — با حرف کوچک شروع می‌شوند و فقط در همان پکیج قابل مشاهده‌اند.

### Accessibility Throughout the Package

+ **Global variables** — بیرون از تابع‌ها تعریف می‌شوند و در تمام توابع پکیج قابل دسترسی‌اند.  
   در goroutineها استفاده از global variableها خطرناک است و ممکن است race condition ایجاد کند.  
+ **Local variables** — درون تابع تعریف می‌شوند و فقط همان تابع به آن‌ها دسترسی دارد.

---

## Functional Options Pattern

زمانی که می‌خواهیم سازنده‌ی (constructor) یک struct را بنویسیم و برخی فیلدها را اختیاری کنیم، برای هر فیلد یک متد setter تعریف می‌کنیم و در نهایت همه‌ی آن‌ها را روی struct اعمال می‌کنیم.

---

## Variable Scope و Lifetime

**Scope** مشخص می‌کند که متغیر در چه محدوده‌ای معتبر است (در سطح پکیج، تابع یا بلوک).  
**Lifetime** مدت زمانی است که متغیر در حافظه زنده می‌ماند.  
وقتی متغیر از scope خارج شود و هیچ referenceی به آن نباشد، garbage collector آن را آزاد می‌کند.  
متغیرهای global تا پایان اجرای برنامه باقی می‌مانند.

---

## Type Comparable

structها در Go قابل مقایسه‌اند اگر تمام فیلدهایشان قابل مقایسه باشند.  
اما **function typeها قابل مقایسه نیستند**.

---

## Functional Programming در Go

هرچند Go یک زبان تمام‌عیار functional نیست، ولی از مفاهیم مهم آن پشتیبانی می‌کند:

+ **First-Class Functions** — تابع‌ها می‌توانند در متغیرها ذخیره شوند یا به‌عنوان پارامتر ارسال شوند.  
در زبان گو ، فانکشن ها first class   هستند

بعضی زبان های برنامه نویسی قابلیتی دارند که می توان، فانکشن را درون یک متغییر ریخت ، به قابلیت فرس کلاس فانکشن می گوند

و در کل بشه رفتار هایی که با وریبل های معمول مانند عدد یا استرینگ می توان انجام داد را با فانکشن ها نیز می توان انجام داد

  [مثال و توضیح](https://golangbot.com/first-class-functions/)

+ **Higher Order Functions** — تابعی که تابع دیگری را می‌گیرد یا برمی‌گرداند.  
 تابعی که بتوان  یک تابع به عنوان پارامتر بگیرد ، یا بتوان خروجی اش یک تابع باشد، از این قابلیت برخوردار است
 
 توجه : این به این معنی نیست که تابع مقدار دهی شده ( کانکریت) در ورودی یا خروجی استفاده شود ، این یعنی تایپ آن فانکشن است
 
همچنین توجه شود که نام تابع به عنوان ورودی یا خروجی استفاده نمی شود و تنها تعداد ورودی و تعداد خروجی آن تابع به عنوان متغییر مشخص است
  [مثال](https://www.golangprograms.com/higher-order-functions-in-golang.html)

```go
package main

import (
    "fmt"
    "net/http"
)

func hello(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintf(w, "hello\n")
}

func main() {
    http.HandleFunc("/hello", hello)
    http.ListenAndServe(":8090", nil)
}
```

در این مثال، تایپ `hello` به عنوان ورودی به `HandleFunc` داده شده است.

---

### Closure

تابعی که خروجی آن تابع دیگری است:

```go
func makeAdder(x int) func(int) int {
    return func(y int) int {
        return x + y
    }
}

func main() {
    addFive := makeAdder(5)
    result := addFive(3)  // 5 + 3 = 8
    fmt.Println(result)
}
```
## go tool

یه فیچر جدیده که از 1.24 اومده و مزایای زیر رو داره

+ قبلا مجبور بودیم گلوبالی نصب کنیم بدیشم این بود که ورژن بندی نبود و مجبور بودیم تمام برنامه ها از یه ورژن استفاده کنن ، تقریبا شبیه استفاده نکردن ازenv  در پایتون

+ تویci/cd هر بار باید دانلود کنیم و و هر بار احتمالا جدید ترین نسخه رو دانلود می کرد

+ نیاز مندی به تولز ها قابل مشاهده و مستند نبود

## work (workspace)

گاهی داریم ۲ یا چند ریپازیتوری را در یک پروژه استفاده می کنیم و می خواهیم که در یک ریپازیتوری تغییرات را بدهیم و در ریپازیتوری دیگر هم تغییرات را ببینیم بدون اینکه ریپوی پیش نیاز رو پوش کنیم ، ریلیز بگیریم و در نهایت تست کنیم ، در این صورت فیچر جدید که از 1.24 اومده رو استفاده می کنیم 

`go work init ./go-clean-template ./go-clean-template-client/api`

هر دو رو توی یک فولدر میزاریم و دستور بالا رو می زنیم ، دیگه از gopath نمیره پکیج ها رو بخونه 