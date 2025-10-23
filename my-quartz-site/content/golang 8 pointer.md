# Escape Analysis in Go

**Escape Analysis** یک تکنیک در Go است که تعیین می‌کند داده‌ها در **stack** ذخیره شوند یا **heap**.

---

## چه زمانی یک متغیر اسکیپ می‌کند؟

- زمانی که از فانکشن return شود
- زمانی که به یک **global variable** انتساب داده شود

در غیر این صورت:

- داده در پایان scope ری‌کلیم می‌شود
- اگر پوینتر باشد، فیلدهایش ری‌کلیم می‌شوند و توسط garbage collector پاک می‌شود.

---

## Data Allocation (Stack vs Heap)

- تصمیم‌گیری توسط **escape analysis**
- بعد از اتمام scope، داده‌های استک به صورت خودکار reclaim می‌شوند
- داده‌های heap توسط GC مدیریت می‌شوند

---

## Type of the Variable: Reference Types vs Value Types

### Value Types

- مقدار واقعی در خود متغیر ذخیره می‌شود
- هر بار پاس دادن به فانکشن → یک کپی از داده ایجاد می‌شود
- پس از پایان کار فانکشن، داده پاک می‌شود (stack)

**Examples:**  
`int, bool, string, struct, arrays`

> اگر بخواهیم pointer بسازیم و مقدار روی heap باشد، از `new()` استفاده می‌کنیم.

---

### Reference Types

- شامل **آدرس داده** هستند، نه داده واقعی
- تغییر در تابع اصلی اثر مستقیم روی داده اصلی دارد
- اگر با `make()` ساخته شوند → روی heap ذخیره می‌شوند
- `new()` نیز می‌تواند ایجاد کند

**Examples:**  
`interfaces, slices, maps, channels, pointers, functions`

```go
var s1 = []int{1, 2, 3}
s2 := s1 // s2 points to the same underlying array

m1 := make(map[string]int)
m1["one"] = 1
m2 := m1 // same map

ch1 := make(chan int)
ch2 := ch1 // same channel

var x int = 42
var p *int = &x // pointer to x
```

---

### مثال تغییر Reference Types در تابع

```go
func modifySlice(s []int) {  
    s[0] = 100  
}

func modifyValue(n *int) {  
    *n = 100  
}

func main() {  
    original := []int{1, 2, 3}  
    modifySlice(original)  
    fmt.Println(original) // Output: [100 2 3]  

    originalValue := 42  
    modifyValue(&originalValue)  
    fmt.Println(originalValue) // Output: 100  
}
```

> توجه: Reference Types بودن به معنای ذخیره شدن حتماً روی heap نیست.

---

## Mutable vs Immutable Types

- **Immutable Types:** تغییر داده → ایجاد کپی جدید در حافظه  
  مثال: `int`, `string`
- **Mutable Types:** تغییر داده → تغییر همان خانه حافظه  

---

## Tips

- **Dereferencing a pointer:** رسیدن از آدرس به مقدار واقعی  
- **Pointer variables:** اشاره به آدرس  
- **Value variables:** نگهداری مقدار واقعی  

### Value vs Reference

| Type | Example | Storage |
|------|---------|---------|
| Value | int, struct, array, string | Stack (usually) |
| Reference | slice, map, chan, func, pointer | Heap (usually) |

> اگر Reference به تابع پاس داده شود، داده کپی نمی‌شود اما روی داده‌های scope بالاتر اثر دارد.  

---

### Slices

- متغیر slice: روی stack  
- محتوا: روی heap (خصوصاً اگر بزرگ باشد یا با make ایجاد شده باشد)

### Maps

- همیشه داده‌ها روی heap هستند  
- متغیر نگهدارنده ممکن است روی stack باشد

### Pointers

- بستگی به محل تعریف دارد  
- اگر روی stack باشد → stack  
- اگر با `new()` یا `make()` ساخته شود → heap  

---

## Garbage Collection (GC)

### الگوریتم‌ها

- **Mark-and-Sweep:** متغیرهای دسترسی‌ناپذیر حذف می‌شوند  
- Heap → توسط GC مدیریت می‌شود  
- Stack → توسط runtime و pattern LIFO بدون GC reclaim می‌شود  

**تنظیم GOGC:** میزان درصد heap برای فعال شدن GC

- هر گوروتین استک مخصوص خود دارد (شروع با 2KB، در صورت نیاز رشد یا shrink می‌کند)  
- Heap بین گوروتین‌ها مشترک است

---

## Nil and Zero Initialization

```go
var A *int
fmt.Println(A) // nil, 0x0 == nil

B := new(int)
fmt.Println(B) // non-nil, zero-initialized
```

> توجه: استفاده بیش از حد از pointer ممکن است هزینه‌بر باشد.

---

## Pointer in Go

```go
var x *int
*x = 20 // ERROR

px := new(int)
*px = 30 // OK
x = px
```

> هرگز از `*interface{}` استفاده نکنید؛ در حقیقت مقدار داخل interface pointer است.

---

## Pointer in For-Range Loop Variable

- المنت حلقه روی یک خانه حافظه ذخیره می‌شود
- استفاده از آدرس المنت → تمامی آدرس‌ها یکسان → خروجی یکسان

**راه حل:**

```go
for k := range src {
    key := k
    dst = append(dst, &key)
}
```

---

## Method Receiver: Pointer vs Value

- **Value Receiver:** کپی از struct → تغییرات روی اصلی اثر ندارد  
- **Pointer Receiver:** تغییرات روی struct اصلی اعمال می‌شود  
- استفاده از pointer برای struct بزرگ یا mutable fields توصیه می‌شود  
- برای map, func, chan → نیازی به pointer نیست

**مثال:**

```go
type Person struct {
	Name string
	Age  int
}

p1 := Person{}
p2 := Person{Name: "Salam", Age: 20}
p3 := new(Person)

fmt.Printf("p1: %+v\n", p1)
fmt.Printf("p2: %+v\n", p2)
fmt.Printf("p3: %+v\n", p3) // Pointer
```

> p3 → Pointer Variable  
> p1, p2 → Value Variables  

---

## Summary

- **Escape Analysis:** تعیین محل ذخیره داده (stack vs heap)  
- **Value vs Reference Types:** کنترل کپی و تغییرات داده  
- **Mutable vs Immutable:** مدیریت تغییر داده‌ها  
- **Pointer Receivers:** کارآمد و mutable  
- **GC:** heap توسط garbage collector، stack توسط runtime مدیریت می‌شود

