

+ اگر کنسل کنیم سیگنال میده به همه ی کانتکست های زیریش و دست آوردش میتونه reclaim  کردن ریسورس باشه ، همچنین متوقف کردن پراسس و جلو گیری از انجام پراسسی که نمی خواهیم


+ می شه داده ها و metadata   های درخواست رو به لایه های داخلی برد و مدیریت کرد


+ توجه شود چون هر کانتکست که کنسل شه ، تنها کانتکست های فرزند کنسل میشن ، پس تایپ ولیو هست و هر بار کپی ارسال میشه


### Common Usages

+ **Managing Goroutines**


زمانی که یک حلقه ی بی نهایت قرار است به صورت گریس فول خاتمه یابد یا فرمان دللاین یا کنسل بیاد

```go


func longRunningTask(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Task canceled")
            return
        default:
            // Do work
        }
    }
}

```
+ **Timeouts**

در جاهایی از برنامه که احتمال تاخیر در جواب باشد مانند database , network در این موارد پاسخ ممکن است با تاخیر باشد پس می توان تایمر گذاشت :

```go
func timeoutExample() {
    // Create a context with a timeout of 2 seconds
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel() // Ensure the context is canceled to avoid resource leaks

    // Start a goroutine that does some work
    done := make(chan struct{})
    go func() {
        defer close(done)
        select {
        case <-time.After(3 * time.Second):
            fmt.Println("Work completed")
        case <-ctx.Done():
            fmt.Println("Work timed out:", ctx.Err())
        }
    }()

    // Wait for goroutine to exit
    <-done
}
```


+ **graceful shutdown**

زمانی که برنامه به صورت طبیعی بخواد شات داون شه بهتره به تمامی فانکشن هایی که در حال اجرا هستن بگیم که با کمترین خسارت بسته بشن


چگونه بفهمیم graseful shutdown پیاده کردیم تو کدمون؟

تو تمام i,o  ها و جاهایی که ریسیو چنل داریم ، باید سیگنال ترمینیت هم بزاریم

همچنین در آخرین خط main.go  باید تعداد گوروتین های فعال رو پرینت کنیم ، اگر ۱ بود درسته ، اما اگر بیشتر از آن بود یعنی گوروتینی هنوز داره کار میکنه




```go
func main() {
    // Create a cancellable context
    ctx, cancel := context.WithCancel(context.Background())

    // Set up a channel to listen for OS signals
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)

    // مثلن کدی که در مثال اول آورده شده
    go longRunningTask(ctx)

    // Block until a signal is received
    sig := <-sigChan
    fmt.Printf("Received signal: %s\n", sig)

    // Cancel the context to notify goroutines to stop
    cancel()

    // Give goroutines time to clean up before exiting
    time.Sleep(5 * time.Second)
    fmt.Println("Shutting down gracefully")
}
```


![alt text](https://github.com/seyedmo30/Tips/blob/main/static/iii6.png)

### context.withcancel

یوز کیس این برای زمانی هست که می خواهیم به صورت دستی ، پراسسی رو متوقف کنیم

وقتی کنسل کال بشه ، تمام فرزند ها می تونن سیگنال دریافت کنن و گوروتین هایی که هستن رو ببندن ،در حقیقت اگر از کنسل استفاده نشه ، هیچ خاصیتی نداره ، اگر هم کنسل کال بشه ولی فرزندان از select ctx.Done  استفاده نکنن ، هیچ تاثیری نداره

### context.withdeadline

ورودی یه زمان میگیره

یک wrapper  هست روی withcancel
درون خود یک cancel  و یک گوروتیندارد ،  که بعد از رسیدن به  زمان ست شده ،  cancel  کال میشود


### context.withtimeout

یه wrapper  هست روی deadline به این صورت که زمانی که میدیم ، با time.now  جمع میشه و بعدش cancel  کال میشه
 


### context.withvalue

توجه داشته باشید کلید حتما باید compatible type باشه، پس کلید هر چیزی نمیتونه باشه


   میشه به جای استفاده از متغییر سراری - گلوبال وریبل - از این استاده کرد ، و از اونجا به پایین نمی تونن روی گلوبال تاثیر بزارن


ولی در کل بدیش اینه که **Side Effects** داریم یعنی داده از جای نا معلوم در دسترسه و میشه گفت **dependency injection** تولید می کنه


+ **cancel()**

توجه شود حتما بعد از همه چی کلوز بشه وگرنه مموری لیک رخ میده


### نکات کلی context

+ هر جا که i/o  داشتیم بهتره که کانتکت رو چک کنیم و تایم اوت بزاریم

+ فانکشن های درون درونی باید در برابر cancel defer  واکنش نشان دهند

+ todo برای زمانی هست که نمیدونیم از کدوم ctx  استفاده کنیم

+ نباید از value برای ارسال داده آپشنال استفاده کرد

### نکات مهم استفاده از ctx  در http server

برای جلو گیری از ddos  و یا مدیریت ریسورس و یا file descriptor بهتره از ctx  استفاده کنیم


+ read timeout 



گاهی درخواست مخرب میده و خواندن درخواست سنگین، منابع رو درگیر میکنه ، البته خود خواندن به بخش های کوچیک تقسیم میشه و تو تصویر مشخصه


