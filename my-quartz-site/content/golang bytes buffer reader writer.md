# Strings vs Byte Slices

- **Strings** در Go **immutable** هستند؛ هر بار که تغییر کنند، مکانشان در حافظه تغییر می‌کند.  
- برای تغییرات مکرر یا رشته‌های بزرگ، بهتر است از **byte slice (`[]byte`)** استفاده کنیم.

bt := []byte("salam")              // Literal initialization
x := []int{10, 20, 30, 40, 50}     // Literal initialization
byt := make([]byte, 0, 50)         // Using make

> **Rule of Thumb:** برای رشته‌های با تغییرات زیاد، همیشه `[]byte` استفاده کنید.  
> Strings → مناسب برای مقادیر ثابت و خوانایی انسان.

---

# Reader Interface


ریدر در مفهوم گولنگ ، یک اینترفیس است که باید پروتوتایپ زیر را پیاده کند . اصولا در کانکریت ، یک استراکتی وجود دارد که متدی به نام رید دارد که از فیلد های استراکت برمی دارد و بر روی اسلایس ورودی میریزد .

به زبان دیگر از استراکت کم می کنه و به اسلایس بایت ورودی اضافه می کند ، توجه شود این را تا سقف اسلایس می ریزد و اتوماتیک طول اسلایس رو افزایش نمی ده

type Reader interface {
	Read(p []byte) (n int, err error)
}

- عملکرد: داده‌ها را از استراکت می‌خواند و در **slice ورودی** قرار می‌دهد.  
- توجه: طول slice تغییر نمی‌کند، فقط تا سقف آن می‌ریزد.

---

# Writer Interface

**Writer** یک interface است که متد زیر را پیاده‌سازی می‌کند:

type Writer interface {
	Write(p []byte) (n int, err error)
}

چرا خودمونو اذییت می کنیم و الکی سعی می کنیم برای یه استرینگ ساده یا یه اسلایس ساده این متد هارو بسازیم؟ 

چون بعضی پکیجا یا متد هایی وجود دارن سطح پایین مثل تغییر روی فایل یا شبکه و ... 

اینا ورودی یا خروجی ، تایپ نمی گیرن ، بلکه اینترفیس رایتر یا ریدر می گیرن ، حالا شما می تونید هر چی که رید یا رایت داره رو بدید به اینا مثل :



# کاربرد Practical

- بسیاری از پکیج‌ها سطح پایین مانند فایل یا شبکه، **Reader** و **Writer** می‌گیرند، نه string یا slice.  
- مثال: `fmt.Fprintf` می‌تواند هر تایپی که `Write` دارد دریافت کند.

stdout := os.Stdout
fmt.Fprintf(stdout, "$$$$")

bt := bytes.NewBuffer([]byte("123456789"))
fmt.Fprintf(bt, "####")

f := new(os.File)
fmt.Fprintf(f, "@@@@")

---

# Common Usage of Writer/Reader

## File I/O

file, _ := os.Create("example.txt")
defer file.Close()

// Write
file.Write([]byte("Hello, Go!"))

// Read
buf := make([]byte, 100)
n, _ := file.Read(buf)
fmt.Println("Read", n, "bytes:", string(buf[:n]))

---

## Custom Implementation

type CustomWriter struct{}

func (cw CustomWriter) Write(p []byte) (n int, err error) {
    return fmt.Print(string(p)) // write to stdout
}

func main() {
    cw := CustomWriter{}
    cw.Write([]byte("Hello, CustomWriter!"))
}

---

## Streams and Pipes

srcFile, _ := os.Open("source.txt")
defer srcFile.Close()

dstFile, _ := os.Create("destination.txt")
defer dstFile.Close()

io.Copy(dstFile, srcFile)

---

## HTTP Requests

resp, _ := http.Get("http://example.com")
defer resp.Body.Close()
body, _ := ioutil.ReadAll(resp.Body)

# Buffer

کتاب‌خانه bytes برای دستکاری (manipulation) روی اسلایس بایت‌ها متدهای زیادی دارد.  
Buffer یک struct شامل اسلایس بایت، طول اسلایس و متدهای کمکی است:

type Buffer struct {
    buf      []byte // contents are the bytes buf[off : len(buf)]
    off      int    // read at &buf[off], write at &buf[len(buf)]
    lastRead readOp // last read operation, so that Unread* can work correctly
}

متدهای مهم:

Bytes() []byte             // خروجی بایت می‌دهد  
Cap() int                  // ظرفیت  
Grow(n int)                // افزایش ظرفیت  
Len() int                  // طول  
Next(n int) []byte         // خروجی n بایت و افزایش offset  
Read(p []byte) (n int, err error)  // می‌خواند و روی slice ورودی می‌ریزد  
ReadByte() (byte, error)  
ReadBytes(delim byte) ([]byte, error)  
ReadFrom(r io.Reader) (n int64, err error)  
ReadRune() (r rune, size int, err error)  
ReadString(delim byte) (string, error)  
Reset()                     // خالی کردن buffer  
String() string             // خروجی string  
Truncate(n int)             // کاهش طول  
UnreadByte() error  
UnreadRune() error  
Write(p []byte) (n int, err error)  
WriteByte(c byte) error  
WriteRune(r rune) (n int, err error)  
WriteString(s string) (n int, err error)  
WriteTo(w io.Writer) (n int64, err error)  

---

# NewReader (strings)

Reader یک struct شامل string و اندیس خواندن است که interface Reader را پیاده می‌کند:

type Reader struct {
	s        string
	i        int64 // current reading index
	prevRune int   // index of previous rune; or < 0
}

متدهای مهم:

Len() int  
Read(b []byte) (n int, err error)  
ReadAt(b []byte, off int64) (n int, err error)  
ReadByte() (byte, error)  
ReadRune() (ch rune, size int, err error)  
Reset(s string)  
Seek(offset int64, whence int) (int64, error)  
Size() int64  
UnreadByte() error  
UnreadRune() error  
WriteTo(w io.Writer) (n int64, err error)  

---

# Example

package main

import (
	"fmt"
	"bytes"
	"os"
)

func main() {
	var strBuffer bytes.Buffer     // تعریف متغیر
	strBuffer.Grow(64)             // ظرفیت اولیه ۶۴
	fmt.Fprintf(&strBuffer, "It is number: %d, This is a string: %v\n", 10, "Bridge")
	strBuffer.Write([]byte("Hello "))
	strBuffer.WriteString("Ranjan")
	strBuffer.WriteString("Kumar")
	bugstr := strBuffer.String()
	fmt.Println("The string buffer output is", bugstr)
	strBuffer.WriteTo(os.Stdout)
}

---

# Reset Slice Efficiently

bucket := make([]byte, 0, 100) // پاک می‌کند و GC آزاد می‌کند، ظرفیت پیش‌فرض تغییر می‌کند  
bucket = nil                   // پاک نمی‌شود، اما capacity ثابت است  
bucket = bucket[:0]            // طول صفر، ظرفیت ثابت، بدون GC

---

# Stream

- باز کردن فایل به معنی بارگذاری در حافظه نیست:

file, err := os.Open("largefile.txt")

### روش‌های متداول:

for {
	n, err := file.Read(buffer)
}

scanner := bufio.NewScanner(file)
for scanner.Scan() {
	// پردازش
}

writer.Flush() // اطمینان از نوشتن همه داده‌ها

### Memory-Mapped Files

import "golang.org/x/exp/mmap"

r, err := mmap.Open("largefile.txt")
