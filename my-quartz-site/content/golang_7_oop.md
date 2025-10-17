## inveriance vs embed

توی گولنگ وراثت نداریم 

یه مثال که می تونیم نیازمندی رو جایزین کنیم :

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
در این مثال کلاس فرزند ، کلاس پدر رو `overwrite` کردند

```go
package main  

import (  
    "fmt"  
)  

// Base struct  
type Animal struct{}  

func (a Animal) Speak() string {  
    return "Animal speaks"  
}  

// Dog struct embedding Animal  
type Dog struct {  
    Animal // embedding the Animal struct  
}  

func (d Dog) Speak() string {  
    return "Woof!"  
}  

// Cat struct embedding Animal  
type Cat struct {  
    Animal // embedding the Animal struct  
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


اما موارد زیر که هیچ کاری نمی تونیم بکنیم :

+ **Multiple Inheritance**

```python

# Base class 1  
class Flyer:  
    def fly(self):  
        return "Flying"  

# Base class 2  
class Swimmer:  
    def swim(self):  
        return "Swimming"  

# Derived class inheriting from both Flyer and Swimmer  
class Duck(Flyer, Swimmer):  
    def quack(self):  
        return "Quacking"  

# Usage  
duck = Duck()  
print(duck.fly())   # Output: Flying  
print(duck.swim())  # Output: Swimming  
print(duck.quack()) # Output: Quacking
```

گولنگ نمی تونه یه استراکت بسازه که چندین ساتراکت والد داشته باشه ، اما به ناچار باید درون خود اونها رو امبد کنه

‍‍
```go
package main  

import (  
    "fmt"  
)  

// Base struct 1  
type Flyer struct{}  

func (f Flyer) Fly() string {  
    return "Flying"  
}  

// Method with potential conflict  
func (f Flyer) Action() string {  
    return "Flying Action"  
}  

// Base struct 2  
type Swimmer struct{}  

func (s Swimmer) Swim() string {  
    return "Swimming"  
}  

// Method with potential conflict  
func (s Swimmer) Action() string {  
    return "Swimming Action"  
}  

// Duck struct embedding Flyer and Swimmer  
type Duck struct {  
    Flyer   // Embedding Flyer  
    Swimmer // Embedding Swimmer  
}  

// No direct 'inheritance' behavior; Duck can implement its own methods  
func (d Duck) Quack() string {  
    return "Quacking"  
}  

func main() {  
    duck := Duck{}  
    
    // Call Fly method  
    fmt.Println(duck.Fly())   // Output: Flying  
  
    // Call Swim method  
    fmt.Println(duck.Swim())  // Output: Swimming  
  
    // Calling Quack method  
    fmt.Println(duck.Quack()) // Output: Quacking  
    
    // Now, let's try to call Action() method from Duck  
    // fmt.Println(duck.Action()) // Uncommenting this line will cause a compilation error  ---------------this
}
```


اگر هر دو استراکت پدر متدی یکسان داشته باشند آنگاه نمی توان به صورت مستقیم آنها را استفاده کرد

اگر متدی یکسان داشته باشند و به صورت مستقیم صدا کنیم این خطا رخ می ده

`error : ambiguous selector duck.Action`



## Polymorphism

چند ریختی میگه شاید چند کلاس متد هایی یکسان و هم نام داشته باشن اما رفتار اونها یکسان نیست


در گولنگ با استفاده از اینترفیس می توان پولیمورفیسم رو پیاده کرد :

```go
package main

import "fmt"

// Shape is an interface that defines a method named `Area`
type Shape interface {
	Area() float64
}

// Rectangle is a struct that represents a rectangle
type Rectangle struct {
	width  float64
	height float64
}

// Area implements the Shape interface for Rectangle
func (r Rectangle) Area() float64 {
	return r.width * r.height
}

// Circle is a struct that represents a circle
type Circle struct {
	radius float64
}

// Area implements the Shape interface for Circle
func (c Circle) Area() float64 {
	return 3.14 * c.radius * c.radius
}

func CalcArea(shapes ...Shape) {
	for _, shape := range shapes {
		fmt.Println(shape.Area())
	}
}

func main() {
	r := Rectangle{width: 10, height: 5}
	c := Circle{radius: 5}

	CalcArea(r, c)
}
```


## encapsulation (public private)

می دونیم که این مفهوم هم در گولنگ ساپورت میشه با `type export unexport` میشه پیاده سازی کرد تا از بیرون پکیج دسترسی کسی نداشته باشه

توجه شود هدف این نیست که کار بر به هیچ وجه دسترسی نداشته باشد بلکه هدف این است که کاربر بداند سازمان رهی و ساختار پکیج طوری است که به صورت مستقیم نباید به داده دسترسی داشته باشد


## Response Envelope

فرض کنید یه **DTO** داریم و به صورت کلی توی پاسخ مثلا **status** , **code**  داریم و با توجه به هر فانکشن ، جواب خاص خودشو داره ، مطمعنیم که فلان سرویس تمام پاسخ هاش یه استاندارد  داره و تو جزییات ، جواب منحصر به فرد همون api  هست
###  interface
اینو پروتوباف خیلی زیبا پیاده کرده ، اون استراکت **wrapper** چندین فیلد ثابت داره که تایپ استرینگ یا اینت هستن اما فیلدی که متغییره ، اینترفیس هست که باید سیگنیچر **body**  داشته باشه و به ازای هر فانکشن باید یه ریسیور داشته باشه که اون رو تایپ اسرشن کنه ،

```go


func (x *PushDataV3ApiWrapper) GetPublicLimitDepths() *PublicLimitDepthsV3Api {
	if x != nil {
		if x, ok := x.Body.(*PushDataV3ApiWrapper_PublicLimitDepths); ok {
			return x.PublicLimitDepths
		}
	}
	return nil
}

func (x *PushDataV3ApiWrapper) GetPrivateOrders() *PrivateOrdersV3Api {
	if x != nil {
		if x, ok := x.Body.(*PushDataV3ApiWrapper_PrivateOrders); ok {
			return x.PrivateOrders
		}
	}
	return nil
}

```


### generics

این روش هم عالیه ، و راحت تر می تونیم unmarshal کنیم

```go


func ParseDepositHistory(jsonBytes []byte) (*APIResponse[DepositHistoryData], error) {
    var resp APIResponse[DepositHistoryData]
    err := json.Unmarshal(jsonBytes, &resp)
    if err != nil {
        return nil, err
    }
    return &resp, nil
}
type APIResponse[T any] struct {
    Code string `json:"code"`
    Data T      `json:"data"`
}
// Similarly, for another endpoint:
type SomeOtherData struct {
    FieldA string `json:"fieldA"`
    FieldB int    `json:"fieldB"`
}

func ParseOther(jsonBytes []byte) (*APIResponse[SomeOtherData], error) {
    var resp APIResponse[SomeOtherData]
    err := json.Unmarshal(jsonBytes, &resp)
    if err != nil {
        return nil, err
    }
    return &resp, nil
}
```


## overriding (Shadow Method)

میشه با استفاده از `type embedding` و پیاده سازی متد یک `اینترفیس` این تکنیک را انجام دهید


به متدی که `override` انجام داده `Shadow Method` می گویند


```python
from abc import ABC, abstractmethod  

# Abstract base class  
class Shape(ABC):  
    
    @abstractmethod  
    def area(self):  
        pass  
    
    @abstractmethod  
    def perimeter(self):  
        pass  

# Circle class inheriting from Shape  
class Circle(Shape):  
    def __init__(self, radius):  
        self.radius = radius  
        
    def area(self):  
        return 3.14 * self.radius * self.radius  
    
    def perimeter(self):  
        return 2 * 3.14 * self.radius  

# Rectangle class inheriting from Shape  
class Rectangle(Shape):  
    def __init__(self, width, height):  
        self.width = width  
        self.height = height  
        
    def area(self):  
        return self.width * self.height  
    
    def perimeter(self):  
        return 2 * (self.width + self.height)  

# Usage  
shapes = [Circle(5), Rectangle(4, 6)]  

for shape in shapes:  
    print(f"Area: {shape.area()}, Perimeter: {shape.perimeter()}")
    
```

حال ما نیاز داریم کلاس های ما در گو زمانی که این ها رو پیاده سازی می کنن ، به متد های خودشون هم دسترسی داشته باشن اما در مثال زیر میبینیم که دسرتسی به اکشن ندارن 


```go

package main

import (
	"fmt"
	"math"
)

// Shape interface defining methods for Area and Perimeter
type Shape interface {
	Area() float64
	Perimeter() float64
	
}

// Circle struct implementing Shape
type Circle struct {
	Radius float64
}

// Area method for Circle
func (c Circle) Area() float64 {
	return math.Pi * c.Radius * c.Radius
}

// Perimeter method for Circle
func (c Circle) Perimeter() float64 {
	return 2 * math.Pi * c.Radius
}

// Method with potential conflict
func (c Circle) Action() string {
	return "Circle Action"
}

// Rectangle struct implementing Shape
type Rectangle struct {
	Width, Height float64
}

// Area method for Rectangle
func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}

// Perimeter method for Rectangle
func (r Rectangle) Perimeter() float64 {
	return 2 * (r.Width + r.Height)
}

// Method with potential conflict
func (r Rectangle) Action() string {
	return "Rectangle Action"
}

// Function to perform actions on Shape
func performShapeActions(shape Shape) {
	fmt.Printf("Area: %.2f, Perimeter: %.2f\n", shape.Area(), shape.Perimeter())
	// Uncommenting the next line will lead to ambiguity
	fmt.Println(shape.Action()) // Compiler error: ambiguous call
}

func main() {
	shapes := []Shape{Circle{Radius: 5}, Rectangle{Width: 4, Height: 6}}

	// Iterate through shapes and perform actions
	for _, shape := range shapes {
		performShapeActions(shape)
	}

	// If you want to call Action, do it explicitly for each type
	circle := Circle{Radius: 5}
	rectangle := Rectangle{Width: 4, Height: 6}

	fmt.Println(circle.Action())    // Output: Circle Action
	fmt.Println(rectangle.Action()) // Output: Rectangle Action
}

```

چون توی اینترفیس اکشن نداریم دسترسی به اون هم نداریم 



## Abstraction

این هم داریم و به خوبی بلدیم



## decorator


در گولنگ این قابلیت رو مثل پایتون نداریم زیرا

+ توی رانتایم فانکشن ها تغییر نمی کنن - Runtime function modification

 + از دکوریتور ساپورت نمیشه - First-class decorator support


ولی شاید بتونیم نیاز مندی مون رو با استفاده از دیزاین پترن **decorator** برطرف کنیم

example 1 :
```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Println("Request received")
        next.ServeHTTP(w, r)
    })
}
```


example 2 :

```go
// ===== Decorator Pattern for Functions =====
type Operation func(int, int) int

// Decorator: Adds logging to any function
func logDecorator(fn Operation) Operation {
	return func(a, b int) int {
		start := time.Now()
		defer fmt.Printf("Execution time: %v\n", time.Since(start))
		fmt.Printf("Calling function with (%d, %d)\n", a, b)
		return fn(a, b)
	}
}

```




+ **type inheritance**

وراثت ساپورت نمی کنه ولی بجاش از `composition` همچنین اینترفیس ها می تونیم استفاده کنیم


+ **operator overloading**

تغییر علامت های محاسبات مانند موارد زیر در گو امکان پذیر نیست

`like +, -, *,`


+ **pointer arithmetic**

مثلن میشه عملیات روی پوینتر ها زد فرض کنید می گیم خونه ی بعدی یک پوینتر رو بده یا پلاس پلاس کن و در خونه بعدی یه چیز رو ذخیره کن و گولنگ اجازه نمی ده

+ **struct type in consts**

نمیشه یه استراکت ثابت ساخت برحلاف تایپ های بیسیک `like integers, floats, strings, and booleans` 
