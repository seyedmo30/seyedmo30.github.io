# Interfaces in Go

### Interface Basics
وقتی گفته می‌شود یک struct یک interface را ستیسفای می‌کند، یعنی تمام متدهای آن اینترفیس را ایمپلمنت کرده است.

```go
type assertion interface conversion

str, ok := interfaceVar.(string)

if msg, ok := rawMsg.(*pgproto3.CopyData); ok {
    // type assertion
}
```

### Embedding Interfaces in Structs
این روش برای وابسته کردن یک struct به یک interface است. در کانستراکتور struct باید شی‌ای ساخته شود که تمام receiver های اینترفیس را ایمپلمنت کرده باشد.

```go
type controller struct {
	dataStore   protocol.DataStore
	downloader  protocol.Downloader
}

func NewController(
	dataStore protocol.DataStore,
	downloader protocol.Downloader,
) *controller {
	return &controller{
		downloader:  downloader,
		dataStore:   dataStore,
	}
}
```

مثال DataStore interface:
```go
type DataStore interface {
	GetLastIdList() (uint32, error)
	List(ctx context.Context) chan types.SeedLink
	Migration(ctx context.Context) error
	Store(ctx context.Context, ggg chan types.ggg) error
}
```
کلاینت باید با توجه به سیگنیچرهای اینترفیس آن را پیاده‌سازی کند.

### Fun Tip
به جای تعریف فانکشن‌های مستقل برای یک struct، بهتر است آن‌ها را روی receiver struct تعریف کنیم:

```go
// ❌ قدیمی
func ProcessSingleOrder(ctx context.Context,u *orderUseCase) error { ... }

// ✅ صحیح
func (u *orderUseCase) processSingleOrder(ctx context.Context) error { ... }
```

### Passing Interfaces as Function Arguments

#### ۱ - کانستراکتور یک struct نیاز به interface داشته باشد
- رایج‌ترین روش برای decoupling
- وابستگی‌ها در کانستراکتور مشخص می‌شوند و قابلیت تست یونیت فراهم است (mock کردن)

#### نکته منفی
- با پاس دادن struct مستقیم، هم به receiver و هم به فیلدها دسترسی داریم.
- با پاس دادن interface، تنها به receiverها دسترسی داریم.
- استفاده بیش از حد می‌تواند محدودیت ایجاد کند، خصوصاً وقتی struct nested باشد.

#### ۲ - ورودی یک فانکشن باشد
- برای گرفتن ورودی‌های جنریک
- مثال: io.Reader  
هر struct که پروتوتایپ read داشته باشد می‌تواند ورودی باشد.

### Returning Interface
- خیلی کم استفاده می‌شود و توصیه نمی‌شود.
- وقتی می‌خواهیم متدهای خروجی محدود شوند، از این روش استفاده می‌کنیم.
- کلاینت فقط به متدهای interface دسترسی دارد، نه به فیلدهای public struct.

### Notes
- استفاده از reflect.Type برای مشخص کردن نوع پارامتر تابع امکان‌پذیر است ولی مشکلات زیادی دارد.
- پاس دادن تابع به عنوان پارامتر راحت است، اما پاس دادن متدها پیچیده است و دردسر زیادی دارد.
