
# Data Analysis in PostgreSQL

برای آنالیز داده‌ها، می‌توان از کوئری‌های پیچیده استفاده کرد.  
نکته‌ها:  

- این کوئری‌ها معمولاً ایندکس نشده‌اند.
- هدف استفاده توسط **ادمین‌ها** یا در موارد بسیار نادر توسط کاربران است.
- داده‌ها فقط از دیتابیس گرفته می‌شوند و نباید در کد تغییر داده شوند.

شاید PostgreSQL بهترین ابزار آنالیز نباشد، اما با تکنیک‌ها و فانکشن‌های مختلف می‌توان تحلیل‌های مفیدی انجام داد.

### تکنیک‌ها و فانکشن‌ها

#### 1. کار با رشته‌ها
- `lower()`, `upper()`, `trim()`, `substring()`

#### 2. کار با تاریخ و زمان
- `to_char()`, `trunc()`  
- توجه: نیاز به راه‌حل مناسب برای تاریخ شمسی است.

+ date_trunc('month',...) تاریخ را به اول ماه تبدیل می‌کند.

+ تنها راه و اکستنشن روی پستگرس : https://github.com/teamappir/jalali_utils

#### 3. مپ کردن مقادیر
- `CASE` برای تبدیل عدد یا مقدار به رشته‌ی مشخص.

#### 4. گرد کردن اعداد
- `round()`, `floor()`, `ceil()`

#### 5. تبدیل نوع داده
- `CAST()` برای تبدیل عدد به رشته یا تاریخ به عدد

#### 6. مدیریت مقادیر NULL
- `COALESCE()` برای جایگزینی مقدار نال با مقدار دلخواه.

#### 7. تحلیل داده‌ها با ستون مشترک
- به جای `GROUP BY` می‌توان از **Window Functions** استفاده کرد.  
- مثال: برای یوزرها در یک شهر، رتبه‌بندی حقوق آن‌ها بدون ترکیب در یک سطر، اما با ردیف‌بندی دقیق:




```sql
SELECT
    city,
    user_id,
    salary,
    RANK() OVER(PARTITION BY city ORDER BY salary DESC) AS city_rank
FROM users;
```


### ETL / ELT

ETL = Extract → Transform → Load

یعنی داده از منبع گرفته می‌شود → تبدیل می‌شود → وارد warehouse می‌شود.

ELT = Extract → Load → Transform

داده خام لود می‌شود و تبدیل در دیتابیس‌هایی مثل BigQuery/Snowflake انجام می‌شود.

### CDC (Change Data Capture)

راهکار استخراج فقط تغییرات روی دیتابیس، نه کل جدول.

+ مبتنی بر WAL (مثلاً PostgreSQL logical decoding)

+ مبتنی بر Trigger

+ مبتنی بر Query-based polling

### چرا warehouseها معمولاً denormalized هستند؟

+ Star Schema 
 
 اسکیما هایی که دینورمالایز هستند یعنی حجم بیشتر میگیرند ولی جویین ندارن

+ Snowflake Schema

 اسکیما نورمالایز هستن جویین ها خیلی زیاد و برای آنالیزور  خیلی سخت اما فضا کمتر می خواد و کانسیستنسی بهتر

 


در OLTP (سیستم‌های عملیاتی مثل بانک، فروشگاه، سفارشات) داده‌ها معمولاً Normalized هستند:

کاهش redundancy

جلوگیری از anomalyها

تمرکز بر write performance و consistency

اما در Data Warehouse (DW) هدف کاملاً برعکس است:

 سرعت Query بالا - (Joinهای زیاد کند هستند → پس جدول‌ها تا جای ممکن ساده و یکپارچه نگه داشته می‌شوند).

 تحلیل‌گر بدون فکر کردن به ۱۲ تا join پشت‌سرهم


# سوالات تمرینی PostgreSQL برای Data Analysis


### نمونه اسکیمای داده (نمونه)

**users**

* id (int)
* name (text)
* city (text)
* signup_date (timestamp)

**products**

* id (int)
* name (text)
* category (text)

**orders**

* id (int)
* user_id (int)
* product_id (int)
* quantity (int)
* price numeric(12,2) -- قیمت واحد در لحظه سفارش
* status text -- مثال: 'completed', 'cancelled'
* created_at timestamp

**page_views**

* id (int)
* user_id (int)
* page text
* duration_seconds int
* viewed_at timestamp

---

# users (نمونه)

| id | name       | city    | signup_date         |
| -: | ---------- | ------- | ------------------- |
|  1 | علی رضایی  | Tehran  | 2024-11-02 09:15:00 |
|  2 | سارا محمدی | Shiraz  | 2025-01-10 14:05:12 |
|  3 | مهدی حسینی | Mashhad | 2025-02-01 08:00:00 |
|  4 | نرگس کریمی | Isfahan | 2024-12-20 19:30:45 |
|  5 | Amir K.    | Tabriz  | 2025-03-03 11:11:11 |
|  6 | لیلا جعفری | Karaj   | 2025-06-07 07:45:00 |

---

# products (نمونه)

| id | name             | category    |
| -: | ---------------- | ----------- |
|  1 | کتاب «SQL ساده»  | books       |
|  2 | هدفون X100       | electronics |
|  3 | ماوس بی‌سیم M5   | electronics |
|  4 | کفش اسپرت Z      | fashion     |
|  5 | گلدان سرامیکی    | home-decor  |
|  6 | بسته لوازم‌تحریر | stationery  |

---

# orders (نمونه)

| id | user_id | product_id | quantity |  price | status    | created_at          |
| -: | ------: | ---------: | -------: | -----: | --------- | ------------------- |
|  1 |       1 |          1 |        1 | 120.00 | completed | 2025-01-05 10:23:45 |
|  2 |       2 |          2 |        2 |  75.50 | completed | 2025-01-15 16:45:00 |
|  3 |       3 |          3 |        1 |  25.00 | cancelled | 2024-12-28 09:00:00 |
|  4 |       1 |          2 |        1 |  75.50 | completed | 2025-02-03 12:00:00 |
|  5 |       4 |          5 |        3 |  40.00 | completed | 2025-03-21 20:10:10 |
|  6 |       5 |          6 |        5 |   8.99 | completed | 2025-03-25 13:05:05 |
|  7 |       6 |          4 |        1 |  55.00 | cancelled | 2025-04-01 09:09:09 |
|  8 |       2 |          1 |        2 | 120.00 | completed | 2025-04-15 21:00:00 |
|  9 |       3 |          2 |        1 |  75.50 | completed | 2025-06-05 17:30:00 |
| 10 |       5 |          3 |        2 |  25.00 | completed | 2025-11-10 08:00:00 |

(مثال: revenue هر ردیف = quantity * price — برای محاسبهٔ گزارش‌ها قابل استفاده است)

---

# page_views (نمونه)

| id | user_id | page            | duration_seconds | viewed_at           |
| -: | ------: | --------------- | ---------------: | ------------------- |
|  1 |       1 | /home           |               35 | 2025-01-05 10:25:00 |
|  2 |       2 | /product/2      |              120 | 2025-01-15 16:46:00 |
|  3 |       3 | /product/3      |               45 | 2024-12-28 09:02:00 |
|  4 |       1 | /checkout       |               90 | 2025-02-03 12:02:00 |
|  5 |       4 | /product/5      |              210 | 2025-03-21 20:12:00 |
|  6 |       5 | /category/books |               60 | 2025-03-25 13:10:00 |
|  7 |       6 | /home           |               15 | 2025-04-01 09:10:00 |
|  8 |       2 | /product/1      |              300 | 2025-04-15 21:02:00 |
|  9 |       3 | /search?q=sql   |               25 | 2025-06-05 17:35:00 |
| 10 |       5 | /blog/post-1    |              180 | 2025-11-10 08:05:00 |


---

## سوال 1 — تعداد کل سفارشات

**مسئله:** تعداد کل ردیف‌های جدول `orders` را محاسبه کن.

**جواب (SQL):**

```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```

**توضیح خط‌به‌خط:**

* `SELECT COUNT(*) AS total_orders` — تابع تجمعی `COUNT(*)` تعداد کل ردیف‌ها را می‌شمارد. از `AS` برای نامگذاری خروجی استفاده شده.
* `FROM orders` — داده‌ها از جدول `orders` خوانده می‌شوند.

---

## سوال 2 — تعداد سفارشات تکمیل‌شده در یک ماه خاص

**مسئله:** تعداد سفارشاتی که وضعیتشان `completed` است و در ماه ژانویهٔ 2025 انجام شده‌اند را پیدا کن.

**جواب (SQL):**

```sql
SELECT COUNT(*) AS completed_jan_2025
FROM orders
WHERE status = 'completed'
  AND created_at >= '2025-01-01'::date
  AND created_at <  '2025-02-01'::date;
```

**توضیح خط‌به‌خط:**

* `WHERE status = 'completed'` — فقط سفارش‌های تکمیل‌شده را انتخاب می‌کنیم.
* `created_at >= '2025-01-01'::date` — تاریخ شروع محدوده (شامل)؛ `::date` رشته را به نوع تاریخ تبدیل می‌کند.
* `created_at < '2025-02-01'::date` — مرز بالایی باز است تا تمام تاریخ‌های ژانویهِ 2025 پوشش داده شوند.

---

## سوال 3 — مجموع درآمد کل (Revenue)

**مسئله:** مجموع درآمد حاصل از سفارشات تکمیل‌شده را محاسبه کن. درآمد هر سفارش `quantity * price` است.

**جواب (SQL):**

```sql
SELECT SUM(quantity * price) AS total_revenue
FROM orders
WHERE status = 'completed';
```

**توضیح خط‌به‌خط:**

* `SUM(quantity * price)` — برای هر ردیف مقدار `quantity * price` محاسبه و سپس همهٔ آن‌ها جمع می‌شود.
* `WHERE status = 'completed'` — فقط سفارش‌های تکمیل‌شده وارد محاسبه می‌شوند.

---

## سوال 4 — درآمد ماهانه (گروه‌بندی بر اساس ماه)

**مسئله:** درآمد کل را برای هر ماه محاسبه کن و بر اساس ماه صعودی مرتب کن.

**جواب (SQL):**

```sql
SELECT
  date_trunc('month', created_at) AS month,
  SUM(quantity * price) AS revenue
FROM orders
WHERE status = 'completed'
GROUP BY date_trunc('month', created_at)
ORDER BY month;
```

**توضیح خط‌به‌خط:**

* `date_trunc('month', created_at)` — ستون `created_at` را به ابتدای ماه مربوطه گرد می‌کند (مثلاً '2025-01-01 00:00:00').
* `GROUP BY date_trunc('month', created_at)` — گروه‌بندی بر اساس ماه انجام می‌شود.
* `SUM(quantity * price)` — درآمد هر گروه ماهیانه محاسبه می‌شود.
* `ORDER BY month` — نتیجه براساس ماه به صورت صعودی مرتب می‌شود.

---

## سوال 5 — پربازدیدترین صفحات (Top pages)

**مسئله:** پنج صفحه‌ای که بیشترین تعداد بازدید (`page_views`) را دارند بیاب.

**جواب (SQL):**

```sql
SELECT page, COUNT(*) AS views
FROM page_views
GROUP BY page
ORDER BY views DESC
LIMIT 5;
```

**توضیح خط‌به‌خط:**

* `GROUP BY page` — گروه‌بندی بر اساس آدرس / نام صفحه.
* `COUNT(*) AS views` — شمارش بازدیدها در هر صفحه.
* `ORDER BY views DESC` — مرتب‌سازی نزولی تا صفحات با بیشترین بازدید بالاتر باشند.
* `LIMIT 5` — فقط ۵ ردیف برتر را نشان می‌دهد.

---

## سوال 6 — میانگین مدت بازدید هر صفحه

**مسئله:** برای هر صفحه میانگین `duration_seconds` را محاسبه کن؛ نتیجه را به صورت نزولی بر اساس میانگین مرتب کن.

**جواب (SQL):**

```sql
SELECT page, AVG(duration_seconds) AS avg_duration
FROM page_views
GROUP BY page
ORDER BY avg_duration DESC;
```

**توضیح خط‌به‌خط:**

* `AVG(duration_seconds)` — میانگین زمان بازدید برای هر گروه محاسبه می‌شود.
* `GROUP BY page` — گروه‌بندی بر اساس صفحه.
* `ORDER BY avg_duration DESC` — مرتب‌سازی نزولی بر اساس میانگین.

---

## سوال 7 — تعداد کاربران جدید در هر ماه

**مسئله:** برای هر ماه، تعداد کاربران جدیدی که ثبت‌نام کرده‌اند را محاسبه کن.

**جواب (SQL):**

```sql
SELECT date_trunc('month', signup_date) AS month, COUNT(*) AS new_users
FROM users
GROUP BY date_trunc('month', signup_date)
ORDER BY month;
```

**توضیح خط‌به‌خط:**

* `date_trunc('month', signup_date)` — ثبت‌نام‌ها را به ماه خلاصه می‌کنیم.
* `COUNT(*)` — تعداد کاربران در هر ماه.
* `GROUP BY` و `ORDER BY` مشابه مثال‌های قبل.

---

## سوال 8 — فروش به تفکیک دستهٔ محصولات (JOIN)

**مسئله:** برای هر `category` مجموع درآمد (`quantity * price`) را محاسبه کن.

**جواب (SQL):**

```sql
SELECT p.category, SUM(o.quantity * o.price) AS revenue
FROM orders o
JOIN products p ON o.product_id = p.id
WHERE o.status = 'completed'
GROUP BY p.category
ORDER BY revenue DESC;
```

**توضیح خط‌به‌خط:**

* `FROM orders o` — از جدول `orders` با نام مستعار `o` شروع می‌کنیم.
* `JOIN products p ON o.product_id = p.id` — با جدول `products` وصل می‌شویم تا دستهٔ محصول را بگیریم.
* `WHERE o.status = 'completed'` — فقط سفارش‌های تکمیل‌شده.
* `SUM(o.quantity * o.price)` — محاسبهٔ درآمد برای هر دسته.
* `GROUP BY p.category` — گروه‌بندی بر اساس دسته.
* `ORDER BY revenue DESC` — دسته‌ها را بر اساس درآمد نزولی مرتب می‌کنیم.

---

## سوال 9 — ۱۰ کاربر برتر بر اساس هزینهٔ کل

**مسئله:** ده کاربری که بیشترین هزینهٔ کل (جمع `quantity * price`) را داشته‌اند پیدا کن.

**جواب (SQL):**

```sql
SELECT u.id, u.name, SUM(o.quantity * o.price) AS total_spent
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'
GROUP BY u.id, u.name
ORDER BY total_spent DESC
LIMIT 10;
```

**توضیح خط‌به‌خط:**

* `JOIN orders o ON u.id = o.user_id` — وصل کردن کاربران به سفارش‌هایشان.
* `SUM(o.quantity * o.price)` — مجموع هزینهٔ هر کاربر.
* `GROUP BY u.id, u.name` — چون `u.name` هم در SELECT آمده، باید در GROUP BY باشد یا از تابع تجمعی استفاده کنیم.
* `ORDER BY total_spent DESC` و `LIMIT 10` — ۱۰ کاربر برتر را برمی‌گردانیم.

---

برای هر سفارشِ تکمیل‌شده، به‌ازای هر user_id شمارهٔ ترتیبی سفارش را مشخص کنیم (یعنی «این، اولین/دومین/سومین سفارش کاربر X است»). برای این‌کار از ROW_NUMBER() به‌همراه PARTITION BY استفاده می‌کنیم.

```sql
SELECT
  id,
  user_id,
  created_at,
  quantity * price AS revenue,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at) AS order_seq
FROM orders
WHERE status = 'completed'
ORDER BY user_id, order_seq;
```

ROW_NUMBER()

یک تابع پنجره‌ای (window function) است که به هر ردیف در پنجره یک شمارهٔ یکتا می‌دهد.

OVER (PARTITION BY user_id ORDER BY created_at)

این بخش مشخص می‌کند پنجره چگونه تعریف شود:

+ PARTITION BY user_id 

 برای هر کاربر جداگانه شروع به شماره‌گذاری می‌کند (یعنی شمارش از ۱ برای هر user_id جدا می‌شود).

+ ORDER BY created_at 

 ترتیب شماره‌گذاری بر اساس زمان ایجاد سفارش است (سفارش‌های قدیمی‌تر شمارهٔ کوچک‌تری می‌گیرند).

---
## سوال 10 — رتبه‌بندی کاربران بر اساس هزینه (Window function)

**مسئله:** هر کاربر را بر اساس مجموع هزینه‌شان رتبه‌بندی کن و تعداد کل کاربران و رتبهٔ هر کاربر را نشان بده.

**جواب (SQL):**

```sql
SELECT
  id,
  name,
  total_spent,
  RANK() OVER (ORDER BY total_spent DESC) AS spend_rank
FROM (
  SELECT u.id, u.name, SUM(o.quantity * o.price) AS total_spent
  FROM users u
  JOIN orders o ON u.id = o.user_id
  WHERE o.status = 'completed'
  GROUP BY u.id, u.name
) AS user_spend
ORDER BY spend_rank;
```

**توضیح خط‌به‌خط:**

* بخش داخلی (subquery):

  * `SELECT u.id, u.name, SUM(...) AS total_spent` — برای هر کاربر مجموع هزینه را محاسبه می‌کند.
  * `GROUP BY u.id, u.name` — برای تجمیع بر اساس کاربر.
* بخش بیرونی:

  * `RANK() OVER (ORDER BY total_spent DESC)` — یک تابع پنجره‌ای که به هر ردیف رتبه بر اساس `total_spent` می‌دهد؛ اگر دو کاربر هزینهٔ برابر داشته باشند، رتبهٔ یکسان خواهند گرفت.
  * `ORDER BY spend_rank` — نتایج را بر اساس رتبه مرتب می‌کند.

---

## سوال 11 — تعداد کاربران فعال در ۳۰ روز گذشته (Distinct)

**مسئله:** تعداد کاربران یکتا که در ۳۰ روز گذشته حداقل یک `page_view` داشته‌اند را بدست بیار.

**جواب (SQL):**

```sql
SELECT COUNT(DISTINCT user_id) AS active_users_30d
FROM page_views
WHERE viewed_at >= now() - interval '30 days';
```

**توضیح خط‌به‌خط:**

* `COUNT(DISTINCT user_id)` — تعداد کاربران یکتا را می‌شمارد (هر کاربر یک‌بار حساب می‌شود).
* `viewed_at >= now() - interval '30 days'` — فقط بازدیدهای ۳۰ روز گذشته را در نظر می‌گیریم.

---

### پیشنهادها برای ادامهٔ تمرین

* این کوئری‌ها را با داده‌های واقعی (CSV یا dump) اجرا کن و نتایج را بررسی کن.
* اندیس‌ها (indexes) را برای ستون‌های `created_at`, `user_id`, `product_id`, `viewed_at` بسنج؛ ببین کدام کوئری‌ها بیشتر از همه سود می‌برند.
* پرس و جوهای مشابه را با `EXPLAIN ANALYZE` اجرا کن تا عملکرد را ببینی.

---

اگر دوست داری من می‌تونم:

* همین تمرین‌ها را با دیتاست تستی (SQL inserts یا CSV) آماده کنم، یا
* سوالات را سخت‌تر (CTE، subqueries پیچیده، تحلیل سری زمانی، sampling) بکنم.

به من بگو کدوم سطح رو می‌خوای ادامه بدیم.
