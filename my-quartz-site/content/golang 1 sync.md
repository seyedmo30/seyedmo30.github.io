# Mutex

برای جلوگیری از **race condition**، یعنی زمانی که چند گوروتین به‌صورت همزمان به داده‌ی مشترک دسترسی یا آن را تغییر می‌دهند، از `mutex` استفاده می‌کنیم.

در بیشتر مثال‌ها داده درون یک struct همراه با mutex قرار می‌گیرد:

```go
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}
```

همچنین getter و setter این struct، متدهایی هستند که به داده دسترسی دارند.  
برای حالتی که بخواهیم چند گوروتین بتوانند داده را همزمان **بخوانند** ولی فقط یکی بتواند آن را **تغییر دهد**، از `RWMutex` استفاده می‌کنیم.

```go
package main

import (
	"fmt"
	"sync"
)

var counter int
var mu sync.Mutex // Mutex برای همگام‌سازی

func increment() {
	mu.Lock()   // قفل کردن mutex
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

## RWMutex

زمانی که خواندن یک متغیر نیازی به قفل کردن کامل ندارد، می‌توان از `sync.RWMutex` استفاده کرد.  
در این حالت چند گوروتین می‌توانند همزمان داده را بخوانند، ولی در هنگام نوشتن، تنها یک گوروتین اجازه دارد وارد بخش بحرانی شود.

---

# WaitGroup

بهترین روش استفاده از `WaitGroup` این است که به جای ارسال آن به‌عنوان پارامتر به تابع، مستقیماً در همان محدوده مدیریت شود:

```go
var wg sync.WaitGroup

for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(i int) {
        defer wg.Done()
        writeData(i, i*i)
    }(i)
}

wg.Wait()
```

---

# sync.Cond

این پکیج درون خود یک `mutex` دارد و برای هماهنگ‌سازی گوروتین‌ها از طریق **سیگنال‌دهی و انتظار (wait/signal)** استفاده می‌شود.

کاربرد آن زمانی است که مثلاً یک گوروتین باید منتظر بماند تا کار گوروتین دیگری تمام شود و سپس ادامه دهد.  
در غیر این صورت مجبور می‌شویم در یک حلقه شرط را بررسی کنیم و در صورت آماده نبودن، از `sleep` استفاده کنیم.

نکته: خروجی گوروتین ابتدایی درون یک channel ارسال نمی‌شود تا گوروتین بعدی آن را دریافت کند.

مثال واقعی : فرض کنیم یه گوروتین وظیفش اینه که توی دیتابیس سطر جدید  اضافه کنه و دیگری باید سطر های موجود تو دیتابیس رو جمع بزنه ، حال اگر گوروتین اول به دومی اطلاع ندهد ، گوروتین دوم یا باید تیریگر از دیتابیس بگیره یا حلقه بزنه و پولینگ کنه 

اما راه حل اینه که هر موقع گوروتین اول دیتا ذخیره کرد ، یه سیگنال بده به دومی تا نیاز به پولینگ دیتابیس نباشه


مهمترین نکته اینه که به دو روش unlock  داریم ، یا به روش مستقیم یا با استفاده از cond.Wait() 

هر بار wait  کال بشه ، unlock  رخ می ده تا بتونه سیگنال بفرسته


```go

package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    mu := &sync.Mutex{}
    cond := sync.NewCond(mu)
    ready := false

    // --- waiting goroutine ---
    go func() {
        mu.Lock()
        for !ready {
            cond.Wait()  // releases mu; waits to be signaled
        }
        fmt.Println("Ready! Proceeding...")
        mu.Unlock()
    }()

    // simulate some work, then signal readiness
    time.Sleep(2 * time.Second)

    mu.Lock()
    ready = true
    cond.Signal()  // wake one waiting goroutine
    mu.Unlock()

    // give some time for goroutine to print
    time.Sleep(1 * time.Second)
}

```
---

# sync.Atomic

پکیج `sync/atomic` برای انجام عملیات اتمی روی انواع داده‌ای مانند `int` و `uint64` استفاده می‌شود.  
این پکیج به‌صورت داخلی متغیرها را قفل می‌کند و مقدارشان را تغییر می‌دهد.  
در واقع همان کاری است که می‌توان با mutex نیز انجام داد، اما به‌شکل سبک‌تر و مخصوص عملیات ساده‌تر.

---

# sync.Once

این پکیج اجازه می‌دهد یک تابع فقط **یک بار** اجرا شود.  
بیشتر در پیاده‌سازی الگوهایی مانند **singleton** یا **init** استفاده می‌شود.  
درون آن یک `mutex` و یک flag وجود دارد که بعد از اجرای تابع، آن را قفل می‌کند تا دوباره اجرا نشود.

---

# sync.Pool

پکیج بسیار مفیدی است که به جای ساخت و حذف مکرر objectها، یک **استخر (pool)** از آن‌ها نگه می‌دارد.  
هر بار که نیاز داریم، یک نمونه از استخر برمی‌داریم و پس از اتمام کار، آن را برمی‌گردانیم.

اگر استخر خالی باشد، خود پکیج نمونه‌ی جدیدی می‌سازد.  
یوزکیس این پکیج شبیه به singleton است، اما در اینجا چند نمونه همزمان می‌تواند وجود داشته باشد.

موارد استفاده معمول شامل:
- اتصال‌های پایگاه داده (DB connections)
- اتصال‌های شبکه (Network connections)
- مدیریت حافظه‌ی I/O

---

# singleflight

گاهی درخواست‌های تکراری و همزمانی به دیتابیس، Redis یا سرویس‌های third-party ارسال می‌شود تا داده‌ای تکراری را فچ کنند.  
وقتی مطمئنیم پاسخ‌ها در آن بازه زمانی یکسان هستند، می‌توانیم به جای کش، از پکیج زیر استفاده کنیم:

`golang.org/x/sync/singleflight`

این پکیج از انجام همزمان درخواست‌های تکراری جلوگیری می‌کند و فقط یک بار تابع را اجرا می‌کند.  
متدهای اصلی آن عبارت‌اند از:

- **Do:** اجرای تابع و دریافت نتیجه  
- **DoChan:** دریافت نتیجه در channel  
- **Forget:** حذف دستی نتیجه‌ی کش‌شده (چون TTL ندارد)

نمونه کد:

```go
import "golang.org/x/sync/singleflight"

var group singleflight.Group

func getFromDatabase(userID int) (string, error) {
    // شبیه‌سازی یک درخواست کند به دیتابیس
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
