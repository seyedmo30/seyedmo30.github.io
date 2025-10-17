

### **mutex**

برای اینکه race condition رخ ندهد ، یعنی چند گوروتین دسترسی یا تغییر shared data همزمان نداشته باشند از mutex استفاده می کنیم


در بیشتر مثال ها داده درون یک استراکت ، همراه mutex است 


type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

همچنین getter setter آن ، ریسیور هایی هست که دسترسی به داده دارند . همچنین می توان از ... برای این قابلیت که همزمان بتوان داده را چند گوروتین خواند ولی تنها یک گوروتین بتوان آن را تغییر داد ، استفاده کرد .

```go
package main

import (
	"fmt"
	"sync"
)

var counter int
var mu sync.Mutex // Mutex برای همگام‌سازی

func increment() {
	mu.Lock() // قفل کردن mutex
	counter++
	mu.Unlock() // باز کردن mutex
}

func main() {
	var wg sync.WaitGroup

	// راه‌اندازی چند گوروتین
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			increment()
		}()
	}

	wg.Wait() // منتظر می‌مانیم تا همه گوروتین‌ها تمام شوند

	fmt.Println("Final counter value:", counter) // خروجی نهایی
}
```
+ **rwmutex**

زمانی که خواندن یک متغییر نیازی به لاک کردن ندارد میشه از این استفاده کرد


### waitgroup

بهترین روش برای استفاده اینه که به جای فرستادن به عنوان پارامتر به فانکشن ، و داخل فانکشن هندل کردن ، بهتره از روش زیر استفاده کرد ، و دیگه پارامتر فانکشن نیست

```go

var wg sync.WaitGroup

        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            writeData(i, i*i)
        }(i)
    }
    wg.Wait()

```



### sync.cond

این پکیج درون خودش mutex   داره 

با استفاده از این پکیج میشه wait  کرد و سیگنال فرستاد 

کاربردش تو اینه که مثلا منتظریم یه گوروتین کار دیگه انجام شد و به پایان رسید ، بعدش گوروتین دیگه سیگنال بگیره و یه کار دیگه بکنه ، اگر از این استفاده نکنیم ، باید ابتدا یه حلقه داشته باشیم و شرط رو چک کنیم و در صورتی که شرط هنوز اتفاق نیفتاده ، sleep  بزاریم و بعدش شرطی رو چک کنیم که می تونیم یا خیر 


توجه شود خروجی گوروتین ابتدایی درون چنلی 

ارسال نمیشود و گوروتین بعدی بردارد

### sync.atomic

این پکیج ابتدا وریبل هایی مانند int, uint64  و لاک میکند و مقدار آن رو تغییر میده  ، خودمون هم میتونیم راحت با mutex  پیاده کنیم


### sync.once

اجازه میده یه فانکشن یه بار اجرا بشه ، بیشتر برای اینیت کردن و یا سینگلتون استفاده میشه

این هم درونش یه mutex  داره و یه فلگ ، و یه بار اجازه میده فانکشنالیتی اجرا شه و بعد از اجرا لاک میکنه  


### sync.pool

پکیج مهمیه ، به جای این که برای کار های تکراری ، اینستنس بسازیم و بعد پایان کار اینستنس رو حذف کنیم ، یه استخر ایستنس داشته باشیم و از اون برداریم ، و بعد پایان کارمون بزاریم اون تو

نحوه کارش هم اینجوریه که سعی میکنیم از استخر یه اینستنس بر داریم اگر بود بر میداریم و اگر نبود میسازیم و در پایان میزاریم تو استخر

یوزکیس خیلی شبیه به سینگلتون هست ، اما سینگلتون یکی است و همه از اون یه دونه استفاده میکنن این میتونه چند تا باشه

موارد استفاده مثل db connections , network connection , io memory

### singleflight

گاهی کلی درخواست تکراری و در زمان خیلی محدود به دیتابیس و یا حتی ردیس و یا تردپارتی میره که داده ی تکراری رو فچ کنه و مطمعنیم تو اون بازه پاسخ ها تکراری هستند ، در این صورت می تونیم به جای استفاده از کش و مدیریت کردن آن از پکیج زیر استفاده کنیم :

`golang.org/x/sync/singleflight`

همچنین ۲ تا متد داره :

+ Do

برای انجام و دریافت نتیجه

+ DoChan

دریافت در چنل

+ Forget

نمی تونیم **ttl** یا پالیسی برای عمر موقت جواب بدیم و باید دستی خودمون بعد از مدت مورد نظر اون رو حذف کنیم تا  پاسخ ها بروز باشن


‍
```go

import     "golang.org/x/sync/singleflight"

var group singleflight.Group

func getFromDatabase(userID int) (string, error) {
    // Simulate a slow database call
    log.Printf("DEBUG: Querying database for user %d... This only happens once!", userID)
    time.Sleep(1 * time.Second)
    return fmt.Sprintf("Data for user %d", userID), nil
}

func getUserData(userID int) (string, error) {
    key := fmt.Sprintf("user_%d", userID)

    value, err, _ := group.Do(key, func() (interface{}, error) {
        return getFromDatabase(userID)
    })

    return value.(string), err
}

func main() {
    var wg sync.WaitGroup
    userID := 123

    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(requestID int) {
            defer wg.Done()
            start := time.Now()
            data, err := getUserData(userID)
            elapsed := time.Since(start)

            log.Printf("Request %d: Got '%s' (took %v)", requestID, data, elapsed)
        }(i)
    }

    wg.Wait()
}
```