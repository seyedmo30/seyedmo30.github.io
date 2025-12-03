# Inheritance vs Embedding in Go

در Go مفهومی به نام وراثت کلاس‌ها وجود ندارد. به جای آن از **struct embedding** استفاده می‌کنیم.

---

## مثال ساده وراثت در Python

```python
# Base class  
class Animal:  
    def speak(self):  
        return "Animal speaks"  

# Child class inheriting from Animal  
class Dog(Animal):  
    def speak(self):  
        return "Woof!"  

class Cat(Animal):  
    def speak(self):  
        return "Meow!"  

# Usage  
my_dog = Dog()  
print(my_dog.speak())  # Output: Woof!  

my_cat = Cat()  
print(my_cat.speak())  # Output: Meow!
```

در این مثال کلاس‌های فرزند، متد کلاس پدر را `overwrite` کردند.

---

## Embedding در Go

```go
package main  

import "fmt"

// Base struct  
type Animal struct{}  

func (a Animal) Speak() string {  
    return "Animal speaks"  
}  

// Dog struct embedding Animal  
type Dog struct {  
    Animal  
}  

func (d Dog) Speak() string {  
    return "Woof!"  
}  

// Cat struct embedding Animal  
type Cat struct {  
    Animal  
}  

func (c Cat) Speak() string {  
    return "Meow!"  
}  

func main() {  
    myDog := Dog{}  
    fmt.Println(myDog.Speak()) // Output: Woof!  

    myCat := Cat{}  
    fmt.Println(myCat.Speak()) // Output: Meow!  
}  
```

---

## محدودیت‌ها: Multiple Inheritance

Go از وراثت چندگانه پشتیبانی نمی‌کند.  
برای شبیه‌سازی، چند struct را در یک struct جدید embedding می‌کنیم:

```go
package main  

import "fmt"

// Base struct 1  
type Flyer struct{}  
func (f Flyer) Fly() string { return "Flying" }  
func (f Flyer) Action() string { return "Flying Action" }

// Base struct 2  
type Swimmer struct{}  
func (s Swimmer) Swim() string { return "Swimming" }  
func (s Swimmer) Action() string { return "Swimming Action" }

// Duck embedding Flyer and Swimmer
type Duck struct {  
    Flyer  
    Swimmer  
}  

func (d Duck) Quack() string { return "Quacking" }

func main() {  
    duck := Duck{}  
    fmt.Println(duck.Fly())   // Output: Flying  
    fmt.Println(duck.Swim())  // Output: Swimming  
    fmt.Println(duck.Quack()) // Output: Quacking  

    // Ambiguous method call:
    // fmt.Println(duck.Action()) // ERROR: ambiguous selector
}
```

> اگر متدی با نام یکسان در دو struct والد وجود داشته باشد، دسترسی مستقیم به آن در struct فرزند امکان‌پذیر نیست (`ambiguous selector`).

---

## Polymorphism در Go

چندریختی (Polymorphism) یعنی چند کلاس می‌توانند متدهای هم‌نام داشته باشند اما رفتار متفاوتی ارائه دهند.  
در Go با استفاده از **Interface** قابل پیاده‌سازی است.

```go
package main

import "fmt"

type Shape interface {
    Area() float64
}

type Rectangle struct {
    width, height float64
}

func (r Rectangle) Area() float64 { return r.width * r.height }

type Circle struct {
    radius float64
}

func (c Circle) Area() float64 { return 3.14 * c.radius * c.radius }

func CalcArea(shapes ...Shape) {
    for _, shape := range shapes {
        fmt.Println(shape.Area())
    }
}

func main() {
    r := Rectangle{10, 5}
    c := Circle{5}
    CalcArea(r, c)
}
```

---

## Encapsulation (Public vs Private)

در Go می‌توان با **capitalization** دسترسی به داده‌ها را محدود کرد:

- نام فیلد یا متد با حرف بزرگ → **Public**
- نام فیلد یا متد با حرف کوچک → **Private**

> هدف این است که کاربران از بیرون پکیج به داده‌ها دسترسی مستقیم نداشته باشند، نه اینکه دسترسی کاملاً قطع شود.

---

## Response Envelope و Wrapper

برای پاسخ‌دهی استاندارد (مثلاً `status`, `code`) می‌توان از **wrapper struct** با فیلد interface برای body استفاده کرد:

```go
func (x *PushDataV3ApiWrapper) GetPublicLimitDepths() *PublicLimitDepthsV3Api {
	if x != nil {
		if x, ok := x.Body.(*PushDataV3ApiWrapper_PublicLimitDepths); ok {
			return x.PublicLimitDepths
		}
	}
	return nil
}
```

> روش مشابه برای دیگر پاسخ‌ها نیز قابل استفاده است.

---

## Generics

استفاده از Generics باعث می‌شود Unmarshal JSON راحت‌تر انجام شود:

```go
type APIResponse[T any] struct {
    Code string `json:"code"`
    Data T      `json:"data"`
}

func ParseDepositHistory(jsonBytes []byte) (*APIResponse[DepositHistoryData], error) {
    var resp APIResponse[DepositHistoryData]
    err := json.Unmarshal(jsonBytes, &resp)
    if err != nil { return nil, err }
    return &resp, nil
}
```

خیلی مسخرست اما جنریک ها نمی تونن تا پارامتر های  ریسیور باشن اما می تونن پارامتر متد باشن 

```go
func ListContains[T comparable](needle T, haystack []T) bool {
}

// compiler error: method must have no type parameters
func (u Utils) ListContains[T comparable](needle T, haystack []T) bool {
}
```
---

## Overriding (Shadow Method)

با **embedding** و پیاده‌سازی متد یک اینترفیس، می‌توان تکنیک Overriding را انجام داد:

```go
package main

import "fmt"

type Shape interface {
    Area() float64
    Perimeter() float64
}

type Circle struct{ Radius float64 }

func (c Circle) Area() float64 { return 3.14 * c.Radius * c.Radius }
func (c Circle) Perimeter() float64 { return 2 * 3.14 * c.Radius }
func (c Circle) Action() string { return "Circle Action" }

func main() {
	c := Circle{Radius: 5}
	fmt.Println(c.Action()) // Output: Circle Action
}
```

> اگر متد موردنظر در Interface تعریف نشده باشد، دسترسی به آن با استفاده از نوع Interface امکان‌پذیر نیست.

---

## Abstraction

Abstraction در Go از طریق Interface پیاده‌سازی می‌شود و امکان جداسازی رفتار و پیاده‌سازی فراهم است.

---

## Decorator

Go از **runtime function modification** یا **first-class decorator support** پشتیبانی نمی‌کند، ولی می‌توان الگوی **Decorator** را با توابع پیاده کرد:

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Println("Request received")
        next.ServeHTTP(w, r)
    })
}
```

یا برای توابع ساده:

```go
type Operation func(int, int) int

func logDecorator(fn Operation) Operation {
	return func(a, b int) int {
		start := time.Now()
		defer fmt.Printf("Execution time: %v\n", time.Since(start))
		fmt.Printf("Calling function with (%d, %d)\n", a, b)
		return fn(a, b)
	}
}
```

---

## سایر محدودیت‌ها در Go

- **Type Inheritance:** وراثت مستقیم وجود ندارد، باید از **composition** و اینترفیس‌ها استفاده کرد.
- **Operator Overloading:** تغییر عملگرهای محاسباتی (`+`, `-`, `*`, ...) پشتیبانی نمی‌شود.
- **Pointer Arithmetic:** عملیات ریاضی مستقیم روی pointer‌ها امکان‌پذیر نیست.
- **Struct Type in consts:** نمی‌توان struct را به‌صورت ثابت تعریف کرد (برخلاف basic types).

