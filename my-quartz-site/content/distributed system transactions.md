# دیستریبیوت ترنس‌اکشن‌ها (Distributed Transactions)



## دیتابیس به ازای هر سرویس (Database per Service)

در معماری میکروسرویس اگر برای هر سرویس یک Storage جداگانه داشته باشیم و از یک دیتابیس مرکزی استفاده نکنیم، از الگوی "دیتابیس‌-پر-سرویس" پیروی کرده‌ایم.

* ویژگی‌ها: ایزولیشن، انتخاب دیتابیس متناسب با نیاز سرویس، و قابلیت اسکیل‌پذیری مستقل.
* نکته: در این معماری باید مکانیزم‌هایی برای هماهنگ‌سازی تراکنش‌های بین سرویس‌ها در نظر گرفته شود.

---

## الگوهای هماهنگی تراکنش‌ها

### Two-Phase Commit (2PC)

در این روش پارت‌ها باید مطمئن شوند که تراکنش با موفقیت انجام می‌شود و سپس کامیت کنند. این الگو شامل دو نقش اصلی است:

* **Coordinator** 

 سرویس مدیر که مدیریت تراکنش را برعهده دارد.

* **Participants** 

 نودهایی که داده را نگهداری یا پردازش می‌کنند.


الگوریتم کار به این صورت است که در فاز اول Coordinator از همهٔ Participants می‌پرسد که آیا می‌توانند کامیت انجام دهند؛ اگر همه پاسخ مثبت بدهند، در فاز دوم به همه دستور کامیت ارسال می‌کند، در غیر این صورت دستور رول‌بک صادر می‌شود.

* مزایا: تضمین سازگاری ACID در صورت پشتیبانی کامل از تراکنش‌ها.
* معایب: پیچیدگی، بن‌بست (blocking) در زمان قطعی، و ناسازگاری با برخی دیتابیس‌های NoSQL که قابلیت prepare/commit ندارند.

---

### Saga

در این الگو تراکنش بلندِ توزیع‌شده به مجموعه‌ای از تراکنش‌های محلی تقسیم می‌شود که هر کدام در سرویس مربوطه به‌صورت محلی کامیت می‌شوند.

* نکته: ساگا برای حفظ ACID به‌صورت سنتی مناسب نیست اما الگوی مناسبی برای مقیاس‌پذیری و تحمل خطا است.
* مسئله: دیباگ و تست ساگا دشوار است و نیاز به طراحی برای idempotency و تشخیص توالی تراکنش‌ها دارد.

#### روش‌های پیاده‌سازی Saga

* **Orchestration (متمرکز):** یک سرویس ارکستریتور مسئول ترتیب اجرای مراحل است و در صورت خطا مراحل جبران‌کننده (compensating actions) را اجرا می‌کند.

  * مزایا: مناسب برای workflowهای پیچیده؛ مشارکت‌کننده‌ها نیازی به شناخت یکدیگر ندارند.
  * معایب: ارکستریتور تایت‌کاپل با همه تغییرات می‌شود و تک‌نقطهٔ شکست بالقوه است.

* **Choreography (رقص / توزیعی):** هر سرویس با فرستادن و شنیدن ایونت‌ها تصمیم به ادامه یا جبران می‌گیرد؛ هیچ ارکستریتور مرکزی وجود ندارد.

  * مزایا: سبک و مناسب برای سرویس‌های کوچک؛ وابستگی مستقیم به سرویس مرکزی حذف می‌شود.
  * معایب: پیچیدگی نگهداری، دشواری در تست انتها‌به‌انتها و افزایش پیچیدگی با مقیاس سیستم.

---

## مقایسهٔ Saga و 2PC

| معیار      |                                           Saga |                                             2PC |
| ---------- | ---------------------------------------------: | ----------------------------------------------: |
| زمان کامیت |               هر جزء به‌صورت محلی کامیت می‌کند |           کل کامیت پس از توافق همه انجام می‌شود |
| پیچیدگی    | نیاز به طراحی برای idempotency و compensations | نیاز به پشتیبانی prepare/commit در participants |
| تحمل خطا   |          با compensating actions مدیریت می‌شود |        ممکن است به blocking و deadlock منجر شود |

---

## Event Sourcing

در این روش تغییرات به‌صورت سری از ایونت‌ها ذخیره می‌شوند و به‌جای آپدیت/دیلیت مستقیم، ایونت‌ها append می‌شوند. ترتیب ایونت‌ها اهمیت بسیار زیادی دارد.

* نکته: Event Sourcing معمولاً همراه با CQRS استفاده می‌شود تا خواندن و نوشتن از هم جدا شوند.

---

## Distributed Transactions در دیتابیس‌های مدرن

در برخی دیتابیس‌های جدید امکاناتی برای پشتیبانی از تراکنش توزیع‌شده یا شبه-تراکنش‌های توزیع‌شده فراهم شده است؛ این رویکرد پیچیدگی هماهنگی را از لایهٔ معماری به خود دیتابیس منتقل می‌کند.

* نکته: استفاده از چنین قابلیت‌هایی ممکن است ساده‌سازی توسعه را بیاورد ولی وابستگی به ویژگی‌های خاص دیتابیس را افزایش می‌دهد.

---

## مزایای Database per Service — جمع‌بندی

* ایزولیشن داده و مالکیت سرویس روی دیتای خودش.
* انتخاب تکنولوژی دیتابیس مناسب برای نیاز هر سرویس.
* مقیاس‌پذیری مستقل هر سرویس.

---

## مثال: پیاده‌سازی Saga (Orchestration) — شبیه‌سازی با Go

در این مثال یک Orchestrator ساده داریم که مراحل ثبت سفارش، رزرو موجودی و پردازش پرداخت را پشت سر هم اجرا می‌کند و در صورت خطا مراحل جبران‌کننده را فراخوانی می‌کند.

```go
package main

import (
    "bytes"
    "encoding/json"
    "errors"
    "fmt"
    "io"
    "net/http"
    "time"
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
    order := OrderRequest{OrderID: 1, UserID: 1, ProductID: 1, Quantity: 2, TotalPrice: 200.00}
    if err := createOrderSaga(order); err != nil {
        fmt.Printf("Saga failed: %v\n", err)
    } else {
        fmt.Println("Saga completed successfully")
    }
}

func createOrderSaga(order OrderRequest) error {
    if err := createOrder(order); err != nil {
        return err
    }

    if err := reserveInventory(order.ProductID, order.Quantity); err != nil {
        if derr := compensateOrder(order.OrderID); derr != nil {
            return fmt.Errorf("reserve failed: %v, compensate failed: %v", err, derr)
        }
        return err
    }

    if err := createPayment(order.OrderID, order.TotalPrice); err != nil {
        if derr := compensateInventory(order.ProductID, order.Quantity); derr != nil {
            _ = compensateOrder(order.OrderID)
            return fmt.Errorf("payment failed: %v, compensate inventory failed: %v", err, derr)
        }
        if derr := compensateOrder(order.OrderID); derr != nil {
            return fmt.Errorf("payment failed: %v, compensate order failed: %v", err, derr)
        }
        return err
    }

    return nil
}

func createOrder(order OrderRequest) error {
    return sendRequest("POST", "http://order-service/orders", order)
}

func reserveInventory(productID, quantity int) error {
    req := InventoryRequest{ProductID: productID, Quantity: quantity}
    return sendRequest("POST", "http://inventory-service/inventory", req)
}

func createPayment(orderID int, amount float64) error {
    req := PaymentRequest{OrderID: orderID, Amount: amount}
    return sendRequest("POST", "http://payment-service/payments", req)
}

func compensateOrder(orderID int) error {
    url := fmt.Sprintf("http://order-service/orders/%d", orderID)
    return sendRequest("DELETE", url, nil)
}

func compensateInventory(productID, quantity int) error {
    req := InventoryRequest{ProductID: productID, Quantity: quantity}
    return sendRequest("POST", "http://inventory-service/inventory/compensate", req)
}

func sendRequest(method, url string, body interface{}) error {
    var bodyReader io.Reader
    if body != nil {
        b, err := json.Marshal(body)
        if err != nil {
            return err
        }
        bodyReader = bytes.NewReader(b)
    }

    req, err := http.NewRequest(method, url, bodyReader)
    if err != nil {
        return err
    }
    req.Header.Set("Content-Type", "application/json")

    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    if resp.StatusCode >= 200 && resp.StatusCode < 300 {
        return nil
    }

    // خواندن بادی جهت لاگ یا خطای معنی‌دار
    respBody, _ := io.ReadAll(resp.Body)
    return errors.New(fmt.Sprintf("request to %s failed: status=%d body=%s", url, resp.StatusCode, string(respBody)))
}
```

* نکته: هر میکروسرویس باید endpointهای مناسب برای عملیات جبران‌کننده (compensate) فراهم کند.

---

## منطق شناسهٔ درخواست (Request ID) در بیزینس

در سیستم‌های پراکند، استفاده از یک شناسهٔ یکتا برای هر درخواست به دلایل زیر مهم است:

* جلوگیری از پردازش دوبارهٔ درخواست‌ها (idempotency).
* امکان ردیابی جریان درخواست در سراسر سرویس‌ها (tracing).
* تسهیل پاسخ‌دهی آسنکرون: سرور می‌تواند یک کد رهگیری به کلاینت بدهد تا وضعیت را پیگیری کند.

نکته: در برخی سرویس‌ها (مثل سرویس‌های بانکی) ممکن است از کاربر خواسته شود که `request_id` خودش را تولید کند تا از بار ذخیره‌سازی سمت سرور کاسته شود، اما در بسیاری موارد Gateway یا کلاینت مسئول تولید شناسه است.

---

اگر بخواهید می‌توانم همین سند را به‌صورت یک فایل Markdown یا PDF آمادهٔ دانلود کنم یا نسخهٔ بدون تیتر (یکپارچه) براتون بسازم.
