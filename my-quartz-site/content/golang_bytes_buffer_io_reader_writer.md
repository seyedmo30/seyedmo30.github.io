# byte

گاهی می خواهیم روی یک متغییر استرینگ تغییرات در دفعات زیادی اعمال کنیم ، در این صورت هر بار آدرس و مکان متغییر در مموری تغییر می کند زیرا تایپ استرینگ  ، ایمیوتبل هست . همچنین گاهی رشته ی ما آنقدر بزرگه که بهتر با اسلایس بایت اون رو مدیریت کرد .  

می توان به صورت کلی گفت بهتر است همیشه برای رشته هایی که امکان تغییر در آن ها زیاد است از اسلایس بایت استفاده کنیم، و بدانیم استرینگ بعد از هر بار تغییر ، مکانش در مموری تغییر می کند ، استرینگ برای خوانایی انسان و یا مقدار ثابت استفاده میشود .

```
bt := []byte("salam")              لیترال اینیت

x := []int{10, 20, 30, 40, 50}     لیترال اینیت

byt := make([]byte, 0, 50)                     اینیت


```

# Reader

ریدر در مفهوم گولنگ ، یک اینترفیس است که باید پروتوتایپ زیر را پیاده کند . اصولا در کانکریت ، یک استراکتی وجود دارد که متدی به نام رید دارد که از فیلد های استراکت برمی دارد و بر روی اسلایس ورودی میریزد .

به زبان دیگر از استراکت کم می کنه و به اسلایس بایت ورودی اضافه می کند ، توجه شود این را تا سقف اسلایس می ریزد و اتوماتیک طول اسلایس رو افزایش نمی ده

```
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

# Writer

رایتر یک انترفیس است که متدی به نام رایت باید پیاده سازی کند که از اسلایس ورودی بخواند و به زیر لایه بریزد ، ( در استراکت خود اصولا فیلدی هست که بر روی آن بیفزاید ) 

به زبان دیگر از اسلایس ورودی کم می کند و به اتربیوت درون استراکت میریزد

```
type Writer interface {
	Write(p []byte) (n int, err error)
}
```
# دلیل

چرا خودمونو اذییت می کنیم و الکی سعی می کنیم برای یه استرینگ ساده یا یه اسلایس ساده این متد هارو بسازیم؟ 

چون بعضی پکیجا یا متد هایی وجود دارن سطح پایین مثل تغییر روی فایل یا شبکه و ... 

اینا ورودی یا خروجی ، تایپ نمی گیرن ، بلکه اینترفیس رایتر یا ریدر می گیرن ، حالا شما می تونید هر چی که رید یا رایت داره رو بدید به اینا مثل :


مثلا فانکشن **Fprintf** می گه هر **تایپی** به من بدی من روش یه استرینگ چاپ میکنم فقط باید اون **تایپ** متد **write** داشته باشه
```go
stdout := os.Stdout

fmt.Fprintf(stdout, "$$$$")

bt := bytes.NewBuffer([]byte("123456789"))

fmt.Fprintf(bt, "####")

f := new(os.File)

fmt.Fprintf(f, "@@@@")


```
## Common Usage Writer/Reader

+ **File I/O**

```go
 file, err := os.Create("example.txt")
 defer file.Close()

// Write
_, err = file.Write([]byte("Hello, Go!"))


// Read
buf := make([]byte, 100)
n, err := file.Read(buf)
fmt.Println("Read", n, "bytes:", string(buf[:n]))
```
+ **Custom Implementations**

```go
type CustomWriter struct{}

func (cw CustomWriter) Write(p []byte) (n int, err error) {
    return fmt.Print(string(p)) // Write data to standard output
}

func main() {
    cw := CustomWriter{}
    cw.Write([]byte("Hello, CustomWriter!"))
}

```
+ **Streams and Pipes**

```go
srcFile, err := os.Open("source.txt")
defer srcFile.Close()

dstFile, err := os.Create("destination.txt")
defer dstFile.Close()

 _, err = io.Copy(dstFile, srcFile)
```
+ **HTTP Requests and Responses**
```go
resp, err := http.Get("http://example.com")
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)

```
در مثال های زیر ریدر و رایتر پیاده سازی شدند

# Buffer

 کتاب خونه bytes ، برای دستکاری ( manipulation ) روی اسلایس بایت ها متد های زیادی داره . 

 در حقیقت بافر یک استراکت شامل اسلایس بایت ، طول اسلایس و تعداد متد برای راحتی کار با اسلایس فراهم کرده است ، استراکت بافر :


```go
type Buffer struct {
    buf      []byte // contents are the bytes buf[off : len(buf)]
    off      int    // read at &buf[off], write at &buf[len(buf)]
    lastRead readOp // last read operation, so that Unread* can work correctly.
}

Bytes() []byte                                                     یک خروجی بایت می ده
Cap() int                                                ظرفیت
Grow(n int)                                         می تونیم به ظرفیت بیفزاییم
Len() int                                             طول 
Next(n int) []byte           اون تعداد که مشخص می کنیم از بافر ور می داره  در یه اسلایس می ریزه و خروجی می ده  
Read(p []byte) (n int, err error)      خیلی شبیه نکست هست ، از بافر می خونه و روی اسلایس ورودی می ریزه ، دقت کنید اضافه نمی کنه                 
ReadByte() (byte, error)
ReadBytes(delim byte) (line []byte, err error)
ReadFrom(r io.Reader) (n int64, err error)
ReadRune() (r rune, size int, err error)
ReadString(delim byte) (line string, err error)
Reset()               بافر رو خالی می کنه
String() string                            خروجی استرینگ می ده
Truncate(n int)                   یک عدد می گیره و به همون تعداد نگه می دار بقیه رو پاک می کنه
UnreadByte() error
UnreadRune() error
Write(p []byte) (n int, err error)            ورودی بایت گرفته و به سلایس می افزاید
WriteByte(c byte) error
WriteRune(r rune) (n int, err error)
WriteString(s string) (n int, err error)             ورودی استرینگ گرفته و به سلایس می افزاید
WriteTo(w io.Writer) (n int64, err error)                     
```





# NewReader (strings)

توجه شود اینترفیس ریدر و استراکت ریدر در پکیج استرینگ قاطی نشود :)



در بافر ، رید وجود دارد ، در زیر یک مثال از strings می بینیم که ریدر را پیاده سازی کرده . و متد رید دارد : 



ریدر یک استراکت شامل یک استرینگ ، طول رشته است که چند متد دارد ، در حقیقت شبیه با بافر است ، با این فرق که بافر درون خود اسلایس بایت دارد و ریدر استرینگ .

```go
type Reader struct {
	s        string
	i        int64 // current reading index
	prevRune int   // index of previous rune; or < 0
}



Len() int                 طول استرینگ
Read(b []byte) (n int, err error)        از استرینگ می خونه و روی سلایس ورودی میریزه        
ReadAt(b []byte, off int64) (n int, err error)
ReadByte() (byte, error)
ReadRune() (ch rune, size int, err error)
Reset(s string)
Seek(offset int64, whence int) (int64, error)
Size() int64
UnreadByte() error
UnreadRune() error
WriteTo(w io.Writer) (n int64, err error)

```


# example 

 
```go
   package main
   import (
   	"fmt"
   	"bytes"
   )
   func main() {
   //Creating buffer variable to hold and manage the string data
   	var strBuffer bytes.Buffer          تعریف متغیر
    strBuffer.Grow(64)           به صورت دیفالت ظرفیت بافر ۶۴ است ، بعد اون ۸۰ ولی می تونیم با این همون اول ظرفیتشو مشخص کنیم
    fmt.Fprintf(&strBuffer, "It is number: %d, This is a string: %v\n", 10, "Bridge")           افزودن به بافر
    strBuffer.Write([]byte("Hello "))         افزودن به بافر
   	strBuffer.WriteString("Ranjan")              افزودن به بافر
   	strBuffer.WriteString("Kumar")
    bugstr := strBuffer.String()                  گرفتن خروجی استرینگ
   	fmt.Println("The string buffer output is",bugstr)

    strBuffer.WriteTo(os.Stdout)            مثل پرینت
   }
```

print :

```
The string buffer output is It is number: 10, This is a string: Bridge
Hello RanjanKumar
It is number: 10, This is a string: Bridge
Hello RanjanKumar% 
```

### نکات به دو روش بهینه می توان یک اسلایس را ریست یا خالی کنیم :
		bucket := make([]byte, 0, 100)
در این روش گاربج کالکتور این مریبل رو پاک می کنه و مموری منیجمنت بهتره اما اگر دوباره بخوایم توش چیزی بریزیم ، Capacity آن دیگر 100 نیست و مقدار پیشفرض گو می شود 

    		bucket = nil
در حالت دوم اندازه Capacity ثابت است و مکان اسلایس در حافظه تغییری نمی کند اما گاربج کالکتور دیگر به سرغت آن را پاک نمی کند

		bucket = bucket[:0] 

  

# stream

توجه شود زمانی که فایلی اوپن میشود ، به این معنی نیست که در مموری ریخته میشود ، بلکه تنها اطلاعات  فایل از os  گرفته میشود

`file, err := os.Open("largefile.txt")`

چند راه متداول برای استریم کردن هست : 
+ معمول ترین راه

```
for {
       n, err := file.Read(buffer)
```

+ همچنین میشه از bufio استفاده کرد
تقریبا همون read byte  هست با یه سری قابلیت ها

```
scanner := bufio.NewScanner(file)
   for scanner.Scan() {
```
توجه شود برای write file  باید آخرش فلاش کرد

`writer.Flush() // Ensure all`

+ **Memory-Mapped Files**

این روش بهینه تر از موارد قبلی هست و در مموری مپ ایجاد میکنه ، همچنین پکیج خوب این هست :

```
import "golang.org/x/exp/mmap"

   r, err := mmap.Open("largefile.txt")

```