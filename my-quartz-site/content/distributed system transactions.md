# دیستریبیوت ترنس اکشن ها

 دیتابیس پر سرویس:در معماری میکرو سرویس اگر به ازای هر سرویس یک استوریج درونش باشد و از یک استوریج مرکزی استفاده نکنیم از این مفهوم پیروی کردیم - ویژگی : ایزوله ، با توجه به نیاز دیتابیس مورد نظرو انتخاب میکنیم ، اسکیلبل

در معماری دیتابیس پر سرویس باید از یکی از الگو های زیر استفاده کنیم 

### **2pc**  two phase commit 

در این روش ، پارت ها باید مطمعن شوند که تراکنش با موفقیت انجام می شود و سپس کامیت کنند . همچنین دپ بخش دارد : 
+ کوردینیتور  - این سرویس وظیفه ی مدیریت ترنساکشن ها را برعهده دارد .
+ پارتیسپیت ها - نود های ما هستند
در این روش ، برای هر تراکنش ، ترنساکشنی باز می شود ولی کامیت نمی کنند . در فاز اول کوردینیتور از همه ی پارت ها می پرسد که می توانید کامیت کنید ، اگر تمام پارت ها مثبت بود ، در فاز دوم به همه می گه کامیت کنید و در صورتی که یک پارت نتواند ، به همه دستور می ده رول بک بزنن .

یک روش ترساکشن در دیتابیس ها است ، در این روش پارتیسیپنت (دیتابیس ها) باید توانایی کامیت کردن و رولبک زدن رو داشته باشند ، این روش خوب نیست چون شاید تو اکوسیستم از nosql استفاده شود و این قابلیت ها را ندتشته باشد 


### Saga
Saga اگر بخوهیم اسید را در استوریج خود ، در اسکیل چندین میکرسرویس پیاده سازی کنیم ، از دیزاین پترن ساگا استفاده میکنیم

نکات:
ساگا در ابتدا خیلی پیچیده است ،  دیباگ در ساگا خیلی دشوار است ، دیتا ها نمی تونن رول بک کنن ، چون همواره در لوکال کامیت میکنن ، طراحی باید به گونه ای باشد که توانایی شناخت ترانس اکشن های تکراری (idempotent)  و شناسایی توالی ترنس اکشن ها را داشته باشد،

و دو روش برای پیاده سازی این الگو هست:

+ Orchestration - متمرکز
  
یک ابزار یا سرویس در مرکز میکروسرویس وجود دارد و تمام سرویس ها باید با این هسته مرکزی سینک باشند و ایونت هاشون رو اونجا بفرستن - 

خیلی خیلی شبیه به یوزکیس در تعریف معماری کلین است به این معنی  که توالی انجام یک کار را پیگیری می کند و در صورت موفقیت آمیز بودن مانند استیت ماشین جلو میره و در صورت خرابی میبایست مراحلی که تغییر داده را یکی یکی compensate  کنهمثلن اگر در مرحله اول خطا رخ داد تنها خطا رو برگردونه ، اما اگر در مرحله ی سوم خطا رخ داد باید ۲ مرحله ی قبلی رو compensate کنه و بعد خطا رو بگه

مثلن اگر سرویسی به نام پرداخت داریم و سرویسی به نام انبار داریم

اگر بخواهیم از این الگو استفاده کنیم باید یه سرویس داشته باشیم که این ها رو مدیریت کنه و اسمش سفارش باشه و الگو ی orchestrator  رو پیاده کنه

همچنین در این الگو باید سرویس های زیر دست یا داون استریم ها هم اینترفیسی به آپ استریم بدن به نام conpensate و پیش بینی کنن هر ایونت شاید بخواد کنسل بشه

**مزایا**

:برای ورکفلو های بزرگ خوبه ، participate ها نیازی ندارند down-stream هارو بشناسن ، تنها باید ایونت به کوردینیتور بفرستن ، 

**معایب**

شاید سرویس های داون استریم ه لوس کاپل باشن نسبت به هم اما به ارکستریتور تایت کاپل هستند و با هر تغییر باید اورکستریتور هم تغییر کند

+ choreography - رقص
  
در این حالت هر تغییر باید به سرویس هایی که وابسته به تراکنش هست گفته شود و در صورت ارور بفهمن

در حقیقت با ارسال ایونت یا مسیج با تریگر، به پارتیسپت ها میگن که موفق بوده یا با خطا مواجه شده

می تونیم بگیم این event driven  هست 
**مزایا**:

 برای سرویسای کوچیک خوبه ، نیازی به یه سرویس دیگه نیست

**مشکلات**:

شاید خیلی مدیریت و نگه داریش سخت باشه ، تست کردنش خیلی سخته چون همه باید آپ باشن ، به مرور پیچیده تر میشه


شاید دیگه نیازی به سرویس مرکزی وجود نداشته باشه اما 

#### تفاوت saga و 2pc
در ساگا ترنساکشن در هر پارتیس انجام می شه ، ایونت یا مسیج با تریگر به پاریسی بعدی می ده ، در حالی که در 2pc  کامیت نمی کنه و منتظر پارتیس می مونه و در صورت تایید کامیت میکنه

علاوه بر saga , 2pc دو مورد زیر هم هست

### **event sourcing** 

در این روش تغییرات به صورت سری در دیتا بیس تنها ایجاد میشود ، به این صورت هیچ دستور آپدیت یا دیلیت در دیتابیس وجود ندارد بلکه ایونت ها ذخیره میشوند ، توجه داشته باشید ترتیب بسیار مهم است

### **distributed transactions**

دیتابیس های جدید ، این قابلیت رو به خودشون اضافه میکنند ، در حقیقت پیچیدگی معماری رو حذف میکنیم و پیچیدگی رو به کار با این قابلیت دیتابیس ها میبریم


### database per service

در معماری database per service باید از یکی از الگو های زیر استفاده کنیم

database per service benefit : ایزوله ، با توجه به نیاز دیتابیس مورد نظرو انتخاب میکنیم ، اسکیلبل




### یه مثال شبیه سازی شده از saga Orchestration ولی با chatgpt

ساگا در ساختار تیبل ها تغییری ایجاد نمی کند ، یا فیلدی اضافه نمی کند ، بلکه یک رویکرد است و سرویس ساگا کوردینیتور باید با ارسال دستور به میکروسرویس ها ، آن ها را به مرحله ی بعد ببرد یا رول بک بزنند

در حقیقت باید هر میکرو سرویس ،قابلیت رول بک داشته باشه مثلن اگر داشته باشیم ثبت سفارش ، باید همچنین داشته باشیم کامپنسیت سفارش 

در یک مثال chatgpt

```go

package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

type OrderRequest struct {
    OrderID    int     `json:"order_id"`
    UserID     int     `json:"user_id"`
    ProductID  int     `json:"product_id"`
    Quantity   int     `json:"quantity"`
    TotalPrice float64 `json:"total_price"`
}

type InventoryRequest struct {
    ProductID int `json:"product_id"`
    Quantity  int `json:"quantity"`
}

type PaymentRequest struct {
    OrderID int     `json:"order_id"`
    Amount  float64 `json:"amount"`
}

func main() {
    order := OrderRequest{
        OrderID:    1,
        UserID:     1,
        ProductID:  1,
        Quantity:   2,
        TotalPrice: 200.00,
    }

    err := createOrderSaga(order)
    if err != nil {
        fmt.Printf("Saga failed: %v\n", err)
    } else {
        fmt.Println("Saga completed successfully")
    }
}

func createOrderSaga(order OrderRequest) error {
    // Step 1: Create Order
    if err := createOrder(order); err != nil {
        return err
    }

    // Step 2: Reserve Inventory
    if err := reserveInventory(order.ProductID, order.Quantity); err != nil {
        compensateOrder(order.OrderID)
        return err
    }

    // Step 3: Process Payment
    if err := createPayment(order.OrderID, order.TotalPrice); err != nil {
        compensateInventory(order.ProductID, order.Quantity)
        compensateOrder(order.OrderID)
        return err
    }

    // All steps succeeded
    return nil
}

func createOrder(order OrderRequest) error {
    orderURL := "http://order-service/orders"
    return sendRequest("POST", orderURL, order)
}

func reserveInventory(productID, quantity int) error {
    inventoryURL := "http://inventory-service/inventory"
    req := InventoryRequest{ProductID: productID, Quantity: quantity}
    return sendRequest("POST", inventoryURL, req)
}

func createPayment(orderID int, amount float64) error {
    paymentURL := "http://payment-service/payments"
    req := PaymentRequest{OrderID: orderID, Amount: amount}
    return sendRequest("POST", paymentURL, req)
}

func compensateOrder(orderID int) {
    orderURL := fmt.Sprintf("http://order-service/orders/%d", orderID)
    sendRequest("DELETE", orderURL, nil)
}

func compensateInventory(productID, quantity int) {
    inventoryURL := "http://inventory-service/inventory/compensate"
    req := InventoryRequest{ProductID: productID, Quantity: quantity}
    sendRequest("POST", inventoryURL, req)
}

func sendRequest(method, url string, body interface{}) error {
    jsonBody, err := json.Marshal(body)
    if err != nil {
        return err
    }

    req, err := http.NewRequest(method

```





# لاجیک شناسه درخواست تو بیزنس

چرا کد رهگیری؟؟ به چه دردی میخوره؟؟  فرقش با شناسه درخواست چیه؟ یکی از دعوا های تیم open banking با loan سر همین بود  ، وقتی یه کلاینت داریم ، یه سرور  ، یکیشون درخواست میده ، یکی شاید پاسخ تو همان زمان نده ، مثلا پاسخ ۲ دقیقه دیگه مشخص میشه ، تو این موارد سرور از کلاینت یه شناسه درخواست میده که چک کنه درخواست تکراری نباشه ، و به کاربر بگه درخواستت تکراریه ( البته بانک 

ها چون نمیخوان ایندکس کردن سمتشون باشه ، به کاربر میگن ، شناسه یونیک خودت جنریت کن و بر عهده خودته  تکراری بودن . ولی چون مطمعنن یه یوزر ۲ بار پرداخت نمیکنه مشکلی پیش نمیاد و هزینه بر نیست یه کلاینت ۲ بار درخواست توکن پرداخت بگیره، توی بانک کد رهگیری مهمه چون به این معنیه که یونیکه این و با کد رهگیری میشه رهگیری کرد  )   


