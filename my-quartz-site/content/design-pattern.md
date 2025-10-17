راه کار هایی اثبات شده برای بهتر کردن و خوانا کد زدن 


از کجا میشه فهمید کدام دیزاین پترن مال کدوم دسته است 

+ **creatiural**

برای ساخت پکیج ، ساخت پلتفرم و ایمپورت و استفاده از پکیج

+ **structural**

درگیر ساختار و dto ها میشیم ، نحوه ی ورودی و   خروجی برنامه یا فانکشن (io)

+ **behavioral**

زمانی که ورودی یا ساختار رو گرفتیم ، حالا می خوایم تصمیم بگیریم چجوری باهاش کار کنیم ، مثلن توی runtime  چجوریکار کنیم


# Creational Patterns
بر مکانیسم های ایجاد شی تمرکز می کنند و سعی می کنند اشیا را به شیوه ای مناسب با موقعیت ایجاد کنند


## Abstract Factory

 یک رابط برای ایجاد خانواده های اشیاء مرتبط یا وابسته بدون مشخص کردن کلاس های مشخص آنها فراهم می کند.

در حقیقت زمانی استفاده میشه که پیاده سازی کلاس برامون مهم نیست و مستقله



در این دیزاین پترن ، نیازی نیست تمام اجزای برنامه را ( کلاس ها را ) پیاده سازی کنیم ، بلکه با تعریف اینترفیس ها تنها رابطه ی بین کلاس ها را تعریف می کنیم و پیاده سازی کلاس ها را مستقل از رابطه آنها در نظر می گیریم. 


چهار بخش اصلی :

+ **Abstract Factory**

+ **Concrete Factory**

+ **Abstract Products**

+ **Concrete Products**

**usage**

زمانی استفاده میشه که مانند Builder بخوایم یه پکیج بسازیم ولی ورودی هامون اینترفیس باشه و با رفتار هایی که تعریف کردیم توی اینترفیس کار کنیم

مثلن شما orm جنگو رو دارید ولی می خوایید از mysql  برید روی postgres  پس نیاز دارید یه پکیج نصب کنید ، اون پکیج بدون این که ببینیدتمامی اینترفیس های فکتوری رو پیاده کرده

یا مثلن کسی که یه پلتفورم رو طراحی کرده و بقیه می تونن extention روش بنویسن از این استفاده می کنن

باید interaface در خروجی داد

برای نوشتن پلتفرمی که اکستنشن ها بتونن تو اون پلتفرم کار کنن از این الگو باید استفاده کرد

```go
// ProductA is the interface for the first product type
type ProductA interface {
	Use() string
}

// ProductB is the interface for the second product type
type ProductB interface {
	Use() string
}
type AbstractFactory interface {
	CreateProductA() ProductA
	CreateProductB() ProductB
}


////////////////////


// ConcreteFactory1 is a concrete implementation of AbstractFactory
type ConcreteFactory1 struct{}

func (f *ConcreteFactory1) CreateProductA() ProductA {
	return &ConcreteProductA1{}
}

func (f *ConcreteFactory1) CreateProductB() ProductB {
	return &ConcreteProductB1{}
}

// ConcreteFactory2 is another concrete implementation of AbstractFactory
type ConcreteFactory2 struct{}

func (f *ConcreteFactory2) CreateProductA() ProductA {
	return &ConcreteProductA2{}
}

func (f *ConcreteFactory2) CreateProductB() ProductB {
	return &ConcreteProductB2{}
}

//////////////
func main() {
	var factory AbstractFactory

	// Use ConcreteFactory1
	factory = &ConcreteFactory1{}
	productA1 := factory.CreateProductA()
	productB1 := factory.CreateProductB()
	
	println(productA1.Use()) // Output: Using Product A1
	println(productB1.Use()) // Output: Using Product B1

	// Use ConcreteFactory2
	factory = &ConcreteFactory2{}
	productA2 := factory.CreateProductA()
	productB2 := factory.CreateProductB()

	println(productA2.Use()) // Output: Using Product A2
	println(productB2.Use()) // Output: Using Product B2
}
```


## factory method

 این الگو به ما اجازه می دهد تا با استفاده از interface ها ، شی مورد نظر خود را بسازیم اما زیر کلاس ها بتونن و اجازه داشته باشن کلاس ساخته شده را تغییر دهند 
 
 معمولا زمانی استفاده میشود که یک شی فیلد هاش قابل پیش بینی نباشد 

 بخوایم به زبون ساده بگیم به فانکشنی که اینترفیس ریترن می کنه میگن فکتوری متود.

 به دلیل این که گولنگ از وراثت پشتیبانی نمی کند ، نمی توان مثالی آورد ،  اما میشه به روش زیر تلاش کنیم نیاز مندی هامون رو رفع کنیم

```go
package main

import "fmt"

// Product Interface
type Vehicle interface {
	Drive() string
}

// Concrete Products
type Car struct{}
func (c Car) Drive() string { return "Driving a car 🚗" }

type Bike struct{}
func (b Bike) Drive() string { return "Riding a bike 🚴" }

// Factory Method
func GetVehicle(vehicleType string) Vehicle {
	switch vehicleType {
	case "car":
		return Car{}
	case "bike":
		return Bike{}
	default:
		panic("unknown vehicle")
	}
}

func main() {
	car := GetVehicle("car")
	fmt.Println(car.Drive()) // Driving a car 🚗
}
```

اما اونجا که سابکلاس ها می تونن سوپر کلاس رو اور رایت کنن رو ئایتون داره و گولنگ نداره و گولنگ باید با امبد و کامپوزیشن ئیاده سازی کنه :


```python

from abc import ABC, abstractmethod

class VehicleFactory(ABC):  # Base Creator
    def __init__(self, base_color="red"):
        self.base_color = base_color  # Shared state for all factories
    
    def prepare_vehicle(self):  # Shared logic for all factories
        print(f"Preparing {self.base_color} vehicle")
    
    @abstractmethod
    def create_vehicle(self) -> "Vehicle":  # Factory Method
        pass

# Concrete Creators (inherit shared logic)
class CarFactory(VehicleFactory):
    def create_vehicle(self):  # Override factory method
        self.prepare_vehicle()
        return Car()

class BikeFactory(VehicleFactory):
    def create_vehicle(self):
        self.prepare_vehicle()
        return Bike()
```
به لاجیک شیرد ها توجه شود

```go
type VehicleFactory struct {
    baseColor string
}

func (f *VehicleFactory) prepareVehicle() {
    fmt.Printf("Preparing %s vehicle\n", f.baseColor)
}

// No way to force implementation in "subclasses"
type Creator interface {
    CreateVehicle() Vehicle
}

// Must reimplement shared logic per factory
type CarFactory struct {
    VehicleFactory // Embedding (composition)
}

func (f *CarFactory) CreateVehicle() Vehicle {
    f.prepareVehicle() // Manual call to shared logic
    return Car{}
}

type BikeFactory struct {
    VehicleFactory
}

func (f *BikeFactory) CreateVehicle() Vehicle {
    f.prepareVehicle() // Duplicated boilerplate
    return Bike{}
}
```


## factory method vs Abstract Factory

**شباهت**

ساختار هر دو بسیار شبیه است

و باید interface در خروجی باشد

**نکته** جز معدود جاهایی است که در سیگنیچر این ها ، اینترفیس باید ریترن کرد

بیش از حد شبیه **Abstract Factory** هست تنها تفاوتشون نحوه ی استفادشون هست ولی ساختارشون یکیه 

توی Abstract Factory می تونیم چندین شی شبیه به هم کانکریت کنیم

```go
type AbstractFactory interface {
	CreateProductA() ProductA
	CreateProductB() ProductB
}
```
اما توی factory method نیازه تنها یک شی کانکریت بشه
```go
type Creator interface {
	FactoryMethod() Product
}
```

همچنین client یا کاربر تنها در main.go از اشیا استفده می کنند و خود آن ها را کانکریت نمی کنند

کانکریت کردن وظیفه ی کاربر نیست و subclass باید superclass را کانکریت کند

## Builder

 ساخت یک شی پیچیده را از نمایش آن جدا می کند و به فرآیند ساخت یکسان اجازه می دهد تا نمایش های متفاوتی ایجاد کند.


این دیزاین پترن زمانی استفاده می شه که بخوایم یک استراکت که تعداد زیادی متغیر داره رو مقدار دهی کنیم

بجای اینکه بخوایم تمام مقادیر رو یگ جا بدیم، می تونیم از الگوی بیلدر استفاده کنیم و تیکه تیکه به اون مقدار بدیم

و یکی از بهترین مثال هاش ، او آر ام جنگو هست که می شه تیکه تیکه آبجکت رو ساخت و در نهایت اون رو اگزکیوت کرد

## singleton

 اطمینان می دهد که یک کلاس فقط یک نمونه دارد و یک نقطه دسترسی جهانی به آن فراهم می کند.


این الگو می گوید بهتر است در صورتی که شی داریم که چند جا استفاده می شود با استفاده از الگوی سینگل تن یک بار آن شی را بسازیم و بار ها از آن شی استفاده کنیم .
همچنین دیگر کاربر نمی تواند داده را تغییر دهد و نیاز به تخصیص حافظه بیش از حد به این شی نیست ( در صورتی که از این الگو استفاده نکنیم )

برای این کار باید ابتدا تایپ مورد نظر را تعریف کنیم ( کلاس را بنویسیم ) سپس یک اینستنس از آن بسازیم و با متد گیرنده ، آن را در اختیار تمام پروژه قرار دهیم
    
    
نکته : به هیچ وجه در ساختار سینگلتون نباید اشاره گر استفاده شود زیرا با استفاده از آن می توان اینستنس را تغییر داد  
    

    
نکته : دقت شود بعضی از آموزش ها ، در کنار اینستنس ، تایپ هم تعریف می کنند . در صورتی که کلاس سینگلتن را درون تایپ تعریفی خود بریزیم و آن تایپ را ریترن کنیم ، ریسیور های آن منتقل نخواهند شد . باید خروجی خود استراکت پکیج باشد


    

## Prototype

این الگو ما را ملزم می کند که هر شی که ایجاد می کنیم ، بتوان آن را با تمام ویژگی ها ( پراپرتی ها و متد ها ) بتوان کپی کرد.

درحقیق یک اینترفیس وجود دارد که باید متد  clone را پیاده سازی کرد که خروجی این متد ، تایپ استراکت است .


# Structural Patterns

"چگونه اشیاء/کلاس‌ها ترکیب یا سازماندهی می‌شوند؟"

اگر مشکل ساختار اشیاء است (مثلاً تبدیل رابط‌ها یا ترکیب اشیاء) → ساختاری.




## Proxy

این دیزاین پترن میگه فرض کنیم سرویس یا کلاسی داریم که شاید نتونه همه ی درخواست ها رو هندل کنه ، شاید حجم درخواست ها زیاد باشه ، شاید نیاز باشه درخواست ها رو چک یا کنترل کرده 
به همین دلیل سرویس میانی میسازیم که خیلی شبیه سرویس اصلی است و کلاینت دچار مشکل نمیشه اگه بجای ارسال به سرویس اصلی ، به این بفرسته .

و برای پیاده سازیش نیاز داریم یه اینترفیس با سیگنیچر های سرویس اصلی بسازیم ، و در نهایت سرویس میانی رو مینویسیم و بعد از کنترل در خواست ها رو به سرویس اصلی میفرستیم 

میشه گفت پروکسی خیلی شبیه یک رپر است ، کلاینت نباید متوجه بشود که با کلاس اصلی در تعامل است یا پروکسی 


## facade

الگو بسیار ساده است و ما از آن استفاده می کینم ، زمانی که ما نیاز به چند متد از چندین کلاس متفاوت داریم ، می توان در یک قطعه کد بزرگ ، تمامی آن ها را فراخوانی کرد و استفاده کرد ، نیاز داریم تمامی کلاس ها رو ایمپورت کنیم ، اما راه بهتر این است که یک کلاس ایجاد کنیم که آن متد ها و کلاس ها رو فرا خوانی کنه ، و نیازی نباشه در کد اصلی همه ی جزییات متد ها رو بشناسیم

همچنین زمانی که یک قطعه کد قدیمی که کار با آن دشوار بوده و بهبود آن ناممکن است ، می توان از این دیزاین پترن استفاده کرد
به عبارت دیگر میدونیم یه کد درست کار میکنه ولی کار با اون خیلی سخت است ، یه لایه روی این کد ها مینویسیم که کلاینت هم توانایی کار با متد های قدیمی و دشوار رو داشته باشه ، هم از متد های راحت ما استفاده کنه

و اولین الگو هست دیدم که نیاز به اینترفیس نداره

فرض کنید به جای اینکه کاربر از تک تک ریسیور های یه کلاس باید بلد باشه استفاده کنه و باید ترتیب اجرای چندین ریسیور از چندین کلاس رو توی برنامه اش بچینه ، ما بیایم این ها رو به ترتیب بچینیم براش و کلاینت فقط اون رو کال کنه


## adaptor

در صورتی که بخواهیم دو ابجکت که اینترفیس جدا دارن رو اینتگریت کنیم
همچنین در صورتی که از ترد پارتی استفاده بخوایم بکنیم و نیازه از ورژن های جدید استفاده کنیم

**bridge between two incompatible interfaces**


## composite

این دیزاین پترن برای دیتا استراکچر درختی استفاده میشه و میاد داده رو به صورت سلسله مراتبی تو آبجکت ها میزاره و راحت تر میشه اطلاعات واکشی کرد


## bridge

دقیقا همون مفهوم dependency inversion  یا جدا سازی نیاز مندی با استفاده از اینترفیس هست . میگه وابستگی هاتو با اینترفیس بر طرف کن

## decorator

افزودن قابلیت به کد با کمترین هزینه و بدون تغییر کلاینت

# Behavioral Patterns

الگوهای رفتاری با الگوریتم ها و تخصیص مسئولیت ها بین اشیا سروکار دارند. آنها به مدیریت الگوریتم ها، روابط و مسئولیت ها در بین اشیا کمک می کنند.


## command

این الگو میگه به جای این که پارامتر به فانکشن ها بدیم ، آبجکت بدیم   ، به این صورت ، هر درخواست علاوه بر مقدار ، چندین متد هم داره

یک درخواست را به عنوان یک شی کپسوله می کند و در نتیجه امکان پارامترسازی کلاینت ها با صف ها، درخواست ها و عملیات را فراهم می کند.

همچنین برای پیاده سازی باید فانکشن ها ، اینترفیس بگیرند و ما درخواست ها را ابتدا باید با توجه به پروتوتایپ اینترفیس پیاده کنیم ، فانکشن هم به جای دریافت کلاس یا پارامتر عادی، اینترفیس گرفته و با سیگنیچر های اینترفیس کار میکند

این مفهوم رو توی گولنگ زمانی می بینیم که از `io.Reader/Writer` استفاغده می کنیم & ورودی مهم نیست چیه ، مهم اینه که چند تا سیگنیچر داشته باشه

## mediator

مورد علاقه سپهر ، میاد لوس کاپل میکنه کامپوننت ها رو با استفاده از ارتباط مرکزی ، به جای این که کامپوننت ها به صورت مستقیم با هم در ارتباط باشن ، از طریق یک ابجکت مرکزی با هم در ارتباط هستند

خیلی خیلی شبیه مفهوم ایونت دیریون هست ، در اسکوپ فانکشن اگر بخواهیم فانکشن ها رو لوز کاپل کنیم ، از این مفهوم استفاده میکنیم


## observer

اگر بخواهیم فانکشن یا کلاسی از تغییرات یا آپدیت شدن کلاسی دیگر مطلع شود از این دیزاین پترن استفاده می کنیم

به تعریف دیگر اگر بخواهیم بعد از آپدیت شدن یک تابع، پیام یا داده ای به دیگر کلاس ها داده شود از ابزرور استفاده می کنیم

مثلا سیگنال در جنگو از این دیزاین پترن استفاده می کند

در اینجا ۲ مفهوم داریم  :

Subject یا پابلیشر

Observer یا سابسکرایبر

در سابجکت لیستی وجود دارد که هر ابزرور که بخواهد تغییرات را مشاهده کند ، در آن اضافه می کنیم

همچنیندر در ابزرور ها باید متدی مانند آپدیت باشد تا هر بار در سابجکت تغییر صورت گرفت ، این متد را صدا بزند و آبزرور متوجه تغییر بشود

## strategy 

زمانی که برای انجام کاری ، چند الگوریتم متفاوت داریم و با توجه به ورودی ، باید یکی از الگوریتم ها را انتخاب کنیم ، از دیزاین پترن استراتجی استفاده می کنیم . 

فرض کنیم برای پرداخت الکترونیکی چند درگاه با متد های مختلف داریم و کلاینت می تواند یکی را انتخاب کند . شاید الگوریتم ها متفاوت باشد ولی روند کلی همه یکسان است.

ساختار :

یک اینترفس استراتجی داریم که امضای متد مشترک تمام الگوریتم ها را دارد

تمام متد ها باید اینترفیس را پیاده سازی کنند

کانتکست ، گرفتن اسم متد مشخص از کلاینت و اجرای متد های پیاده سازی شده

کلاینت یا متد اصلی

نکته ی مهم دیگر استراتجی پترن هم این است که به جای استفاده از **if/else** یا **switch** از **map** استفاده می کنیم به این صورت که کلید مپ نام آن استراتجی است و مقدارش **FACTORY FUNCTION** است

فرض کنیم به ازای هر استراتجی یک کانستراکتور باید داشته باشیم مانند کانستراکتور زیر :

`NewShahinService(config config.App, bank string) (*shahinService, error) `
`NewSadadService(config config.App, bank string) (*sadadService, error) `

و وظیفه ی این است که استراکتی که اینترفیس را ایمپلمنت کرده بسازیم

اما مشکلی که وجود دارد این است که **هارد کدی** باید بگیم به ازای این استراتجی چه کانستراکتوری باید داشته باشیم
 همچنین تنها یک بار ایجاد میشود و هر بار از اون مجددا استفالده میشود

اما روش بهینه این است که تایپ اون فانکشن رو بسازیم و اون رو داخل **MAP** قرار بدیم

```go


type BankServiceFactory func(config.App, string) (interfaces.BankService, error)

var registry = make(map[string]BankServiceFactory)
func RegisterBankServiceFactory(providerName string, factory BankServiceFactory) {
	registry[providerName] = factory
}

```

اگر توجه شود سیگنیچر **BankServiceFactory** شبیه کانستراکتور است


## iterator

در صورتی که تعدادی داده داشته باشیم و بخواهیم آن را پیمایش کنیم از این دیزاین پترن استفاده می کنیم .

گاهی ساختار کالکشن داده ی ما به سادگی لیست نیست ، مثلا درختی است ، در این صورت می توانیم نحوه ی پیمایش درختی دلخواه خود را پیاده کنیم ( مثلا بر اساس عمق درخت پیمایش کند یا فرزند خوشه )

مثلا مجموعه ای سطر از دیتابیس گرفته ( به صورت پیور) و حال می خواهیم آن را پیمایش کنیم .
