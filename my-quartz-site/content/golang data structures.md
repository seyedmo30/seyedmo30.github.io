## Map

Map یک ساختار داده کلید-مقدار است که درون خود آرایه‌ای دارد و ایندکس‌ها خروجی‌های تابع Hash هستند و مقادیر در آن ذخیره می‌شوند.

- **Hash Map / Hash Table**  
  ساختاری که با سرعت بالا کلیدهای map را به یک جدول نگاشت می‌کند.

- **Hash Function**  
  با استفاده از این تابع، کلید ورودی به یک عدد تبدیل می‌شود که همان ایندکس آرایه است و مقدار مرتبط با آن بازیابی می‌شود.

- **Handling Collisions**  
  وقتی دو کلید مختلف به یک ایندکس اشاره کنند، با الگوریتم‌هایی مثل chaining حل می‌شود.

- **Time Complexity:** O(1)  
  به همین دلیل سرعت بسیار بالایی دارد.

- **Initial Capacity**  
  اگر map ایجاد شود اما مقدار اولیه ندهیم، پیش‌فرض ظرفیت ۸ است.

[توضیح فارسی ساختار داده Map](https://blog.faradars.org/%D8%B3%D8%A7%D8%AE%D8%AA%D9%85%D8%A7%D9%86-%D8%AF%D8%A7%D8%AF%D9%87-data-structure-%D8%B1%D8%A7%D9%87%D9%86%D9%85%D8%A7%DB%8C-%D8%AC%D8%A7%D9%85%D8%B9-%D9%88-%DA%A9%D8%A7%D8%B1%D8%A8%D8%B1%D8%AF/)

## Linked List

لیست پیوندی آرایه‌ای از گره‌هاست که هر گره شامل داده و اشاره‌گری به گره بعدی است.  
یک اشاره‌گر Head به اولین گره اشاره می‌کند و اگر لیست خالی باشد، مقدار آن null است.

**نکته:**  
در حلقه‌ها برای پیمایش و به‌روزرسانی، بهتر است مقدار را در یک متغیر موقت بریزیم و روی آن کار کنیم.

[مثال Linked List در Go](https://www.developer.com/languages/linked-list-go/)  
[مثال دیگر Linked List](https://www.golangprograms.com/golang-program-for-implementation-of-linked-list.html)

## Stack (LIFO)

داده‌ها به صورتی ذخیره می‌شوند که آخرین داده ذخیره شده، اولین داده بازخوانی شده باشد.

- **Push:** افزودن عنصر به بالای پشته  
- **Pop:** حذف و بازگرداندن عنصر بالای پشته  
- **isEmpty:** بررسی خالی بودن پشته  
- **Top (Peek):** بازگرداندن عنصر بالای پشته بدون حذف آن

[مثال Stack در Go](https://www.tutorialspoint.com/golang-program-to-implement-stack-data-structure)

## Queue (FIFO)

صف، ساختاری است که داده‌هایی که زودتر ذخیره شوند، زودتر خارج می‌شوند.

- **Enqueue():** افزودن عنصر به انتهای صف  
- **Dequeue():** حذف و بازگرداندن اولین عنصر صف  
- **isEmpty():** بررسی خالی بودن صف  
- **Top():** مشاهده اولین عنصر بدون حذف

[مثال Queue در Go](https://www.tutorialspoint.com/golang-program-to-implement-the-queue-data-structure)
