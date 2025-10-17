سالید مجموعه اصولی است که اسکوپ آن پایین تر از معماری و در سطح ماژول است

# Single Responsibility

این اصل میگه هر کلاس باید یک کار را انجام دهد به عبارت دیگر یک کلاس فقط باید به یک دلیل تغییر کنه

برای رعایت این اصل ، یکی از نکات این است که ریسیور های استراکچر را با دقت بنویسیم ، زیرا شاید یه استراکت بیش از چند ریسیور داشته باشد و در نتیجه بیش از یک کار انجام دهد

نکات: خوبیه این اصل اینه که کار ها رو میشکونیم و به بخش های ساده و قابل فهم تقسیم میکنیم و نتیجه نگه داری هم آسون تره ولی بدیش اینه که داپلیکیت زیاد میشه و کلی چیز داریم که تنها یه وظیفه دارن .


+ تشخیص مسعولیت های مستقل و جدا

+ در گو می توان با ساختن پکیج ها ، انکپسوله کرد اون فانکشنالیتی رو و می توان پکیج ها رو مستقل و مسعول یک کار در نظر گرفت

+ باید در ایجاد ریسیور های یک استراکت دقت کرد و نباید تعداد زیادی ریسیور بی ربط در یک استراکت ساخت

+ اگر یه اینترفیس سیگنیچر های زیاد داشت می توان شکوند اون رو و به تعداد **اینترفیس** های کوچیک تغییر داد

+ جدا سازی لاجیک از دیتاترنسفورم و ذخیره دیتا

+ اگر پیاده کنیم در نتیجه راحت تر میتونیم تست کنیم ، ریفکتور کنیم ، نگهداری و توسعه بدیم

```go
type UserService struct{}

func (s *UserService) CreateUser(name string) {
    // Create user logic
}

func (s *UserService) SendEmailNotification(userID int, message string) {
    // Send email logic
}
```

**srp**

```go
type UserService struct{}

func (s *UserService) CreateUser(name string) {
    // Create user logic
}

type NotificationService struct{}

func (n *NotificationService) SendEmailNotification(userID int, message string) {
    // Send email logic
}
```

# Open/close

برای رعایت این اصل ، باید برنامه جوری نوشته شده باشه که در صورتی که بخوایم تغییری در آینده ایجاد کنیم ، تنها چیزی به برنامه اضافه کنیم و کد های قبلی دچار تغییر زیاد یا حذف نشن

بهترین راه برای رعایت این اصل ، ارث بری است


+ استفاده از اینترفیس به این صورت که هر کی بخواد یه اینستنس بیفزاد ، باید اینترفیس رو پیاده سازی کنه

+ مثال می خوایم یه درگاه پرداخت اضافی بزاریم کافیه درگاه پرداخت جدید ، اینترفیس قبلی رو پیاده کنه
```go
type DiscountCalculator struct{}

func (d *DiscountCalculator) CalculateDiscount(orderType string, amount float64) float64 {
    if orderType == "regular" {
        return amount * 0.1 // 10% discount for regular orders
    }
    if orderType == "special" {
        return amount * 0.2 // 20% discount for special orders
    }
    return 0
}

```
باید تبدیل شه به این

```go
type Discount interface {
    Calculate(amount float64) float64
}

type RegularDiscount struct{}
func (r *RegularDiscount) Calculate(amount float64) float64 {
    return amount * 0.1 // 10% discount for regular orders
}

type SpecialDiscount struct{}
func (s *SpecialDiscount) Calculate(amount float64) float64 {
    return amount * 0.2 // 20% discount for special orders
}

type DiscountCalculator struct{
    discount Discount
}

func (d *DiscountCalculator) CalculateDiscount(amount float64) float64 {
    return d.discount.Calculate(amount)
}

```


# Liskov

شیاء یک سوپرکلاس باید با اشیاء یک زیر کلاس بدون تأثیر بر صحت برنامه قابل تعویض باشند

میگه که در صورتی که از یک شی ، چند نمونه بسازیم ، باید برنامه جوری باشه که اگر نمونه ها را جا بجا کنیم ، برنامه به خطا نخوره


برای رعایت این اصل ، باید در ایجاد وراثت دقت کنیم ، مثلا اگر کلاس پدر یک متد دارد ، نباید کلاس فرزند همان متد را داشته باشد 


در حقیقت جوری باید بنویسیم که اگر فرزندی ساختیم ، مانند والد بتوان کار ها رو انجام بده و راه حل ، استفاده از پلی مورفیم است . 
    

یه جورایی چون وراثت نداریم و نمی تونیم از طریق کلاس فرزند ، متد والد رو overwrite کنیم زیاد این دقدقه رو نداریم

```go
package main

import "fmt"

// Shape interface defines methods for any shape
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Circle struct that implements the Shape interface
type Circle struct {
    radius float64
}

func (c *Circle) Area() float64 {
    return 3.14 * c.radius * c.radius
}

func (c *Circle) Perimeter() float64 {
    return 2 * 3.14 * c.radius
}

// Square struct that implements the Shape interface
type Square struct {
    sideLength float64
}

func (s *Square) Area() float64 {
    return s.sideLength * s.sideLength
}

// LSP Violation: Square does not support Perimeter method correctly.
func (s *Square) Perimeter() float64 {
    return 4 * s.sideLength
}

func printShapeInfo(shape Shape) {
    fmt.Println("Area:", shape.Area())
    fmt.Println("Perimeter:", shape.Perimeter())
}

func main() {
    c := &Circle{radius: 5}
    s := &Square{sideLength: 4}

    // Both Circle and Square conform to the Shape interface
    printShapeInfo(c)
    printShapeInfo(s)
}
```

**lsp**

```go
package main

import "fmt"

// Shape interface defines methods for any shape
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Circle struct that implements the Shape interface
type Circle struct {
    radius float64
}

func (c *Circle) Area() float64 {
    return 3.14 * c.radius * c.radius
}

func (c *Circle) Perimeter() float64 {
    return 2 * 3.14 * c.radius
}

// Square struct that implements the Shape interface
type Square struct {
    sideLength float64
}

func (s *Square) Area() float64 {
    return s.sideLength * s.sideLength
}

func (s *Square) Perimeter() float64 {
    return 4 * s.sideLength
}

func printShapeInfo(shape Shape) {
    fmt.Println("Area:", shape.Area())
    fmt.Println("Perimeter:", shape.Perimeter())
}

func main() {
    c := &Circle{radius: 5}
    s := &Square{sideLength: 4}

    // Now both Circle and Square correctly implement the Shape interface
    printShapeInfo(c)
    printShapeInfo(s)
}
```
# Interface Segregation
    
کلاس‌ها نباید مجبور باشن متدهایی که به اونها احتیاجی ندارن رو پیاده‌سازی کنن ، اینترفیس (Interface) ها رو جوری بنویسیم که وقتی یک کلاس از اون استفاده میکنه، مجبور نباشه متدهایی که لازم نداره رو پیاده‌سازی کنه. یعنی متدهای بی‌ربط نباید توی یک اینترفیس کنار هم باشن 
    
یا اینکه بخاطر اینکه یک اینترفیس را پیاده سازی کنیم ، مجبور باشیم متد های بی خاصیتی بسازیم که فقط چارچوب اینترفیس را امپلمنت کرده باشیم

یکی از دیزاین هایی که می توان با استفاده از آن ، به این رسید ، facade است - البته مطمعن نیستم -

باید اینترفیس هایی که چندین سیگنیچر دارن رو بشکونیم و به اینترفیس های کوچیک در بیاریم ، حالا از هر کدوم نیاز شد استفاده کنیم

مثلن اینترفیس ریدر یا رایتر هر کدوم جدا تنها یه سیگنیچر دارن

```go
type Worker interface {
    Work()
    Eat()
}

type Robot struct{}
func (r *Robot) Work() {
    fmt.Println("Robot is working")
}
func (r *Robot) Eat() {
    fmt.Println("Robot does not eat")
}

type Human struct{}
func (h *Human) Work() {
    fmt.Println("Human is working")
}
func (h *Human) Eat() {
    fmt.Println("Human is eating")
}

```
**isp**
```go
type Workable interface {
    Work()
}

type Eatable interface {
    Eat()
}

type Robot struct{}
func (r *Robot) Work() {
    fmt.Println("Robot is working")
}

type Human struct{}
func (h *Human) Work() {
    fmt.Println("Human is working")
}
func (h *Human) Eat() {
    fmt.Println("Human is eating")
}
```
# Dependency Inversion
 
 کلاس‌های سطح بالا نباید به کلاس‌های سطح پایین وابسته باشن؛ هر دو باید وابسته به انتزاع (Abstractions) باشن. موارد انتزاعی نباید وابسته به جزییات باشن. جزییات باید وابسته به انتزاع باشن

اگر این اصل را رعایت نکنیم ، برای یونیک تست، باید تمام نیاز های Use case را پیاده سازی کینم.

از کاربرد های این اصل ، می توان به رفع dependency injection اشاره کرد . یکی از رایج ترین راه های decoupl کردن کد :مثلا  فرض کنید یک Use case ( زیر سرویس ) داریم و می خواهیم وابستگی آن به کانفیگ دیتا بیس رو با اینترفیس حل کنیم . 





```go
type EmailService struct{}
func (e *EmailService) SendEmail(message string) {
    fmt.Println("Sending email:", message)
}

type UserService struct {
    emailService *EmailService
}

func (u *UserService) NotifyUser(message string) {
    u.emailService.SendEmail(message)
}
```
**dip**
```go

type NotificationService interface {
    Send(message string)
}

type EmailService struct{}
func (e *EmailService) Send(message string) {
    fmt.Println("Sending email:", message)
}

type UserService struct {
    notificationService NotificationService
}

func (u *UserService) NotifyUser(message string) {
    u.notificationService.Send(message)
}
```



