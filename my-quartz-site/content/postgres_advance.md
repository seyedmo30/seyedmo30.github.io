# interview postgres

همیشه ۲ تا مسعله پرسیده میشه و من نمی تونم جواب بدم

+ اگر سرعت کوییری ها پایین بیاد باید چیکار کنیم؟

+ advance query (select , groupby,having,join,subquery)

## اگر سرعت کوییری ها پایین بیاد باید چیکار کنیم؟

یه اصطلاحی هست تحت عنوان  tuning هست که می تونه به سرعت کوییری ها کمک کنه


## نوشتن کوییری های پیشرفته

به نظرم به هوش مصنوعی بگیم سوال بپرسه بهتره ولی حتما grouup by ها رو بگیم , join , having 

مثال

```sql

SELECT
    c.name,
    SUM(o.total_amount) AS total_spend           // این جا اگریگیت ها مهم هستن
FROM
    customers c
JOIN
    orders o ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id, c.name                        // همیشه جدولی که ایتماش کمه و اونی که کلید خارجی داده  تو گروپ آیدی میاد
HAVING
    SUM(o.total_amount) > 500                    // دقیقا مثل where هست برای گروپ
ORDER BY
    total_spend DESC;                            // نزولی یعنی بیشترن تعداد یا جدید ترین تاریخ

```

### Customers Table
| `customer_id` | `name`          | `email`               |
|--------------|-----------------|-----------------------|
| 1            | Alice Smith     | alice@example.com     |
| 2            | Bob Johnson     | bob@example.com       |
| 3            | Charlie Brown   | charlie@example.com   |

### Orders Table
| `order_id` | `customer_id` | `order_date` | `total_amount` |
|------------|---------------|--------------|----------------|
| 101        | 1             | 2023-10-01   | 200.00         |
| 102        | 1             | 2023-10-05   | 350.00         |
| 103        | 2             | 2023-10-10   | 600.00         |
| 104        | 3             | 2023-10-12   | 100.00         |
| 105        | 3             | 2023-10-15   | 150.00         |
| 106        | 3             | 2023-10-20   | 300.00         |

### Query Result
| `name`          | `total_spend` |
|-----------------|---------------|
| Bob Johnson     | 600.00        |
| Alice Smith     | 550.00        |





### explain analyze


اگر یه کوییری ثابت اذییت کرد و سنگین شد از explain analize  استفاده میکنیم به این صورت که اول کوییری این دستور رو مینویسیم


مثلن می تونیم با استفاده از ایندکس ها و دوباره بررسی از طریق explain analyze سرعت کوییری رو محاسبه کنیم و با استفاده از تست ها ، tuning کنیم


```sql

EXPLAIN SELECT * FROM employees
WHERE department_id = 10
ORDER BY salary DESC;

```


و پاسخ

```sql
                         Query Plan                         
-------------------------------------------------------------
 Seq Scan on employees  (cost=0.00..45.25 rows=10 width=220)
   Filter: (department_id = 10)
   ->  Sort  (cost=0.00..0.00 rows=0 width=220)
         Sort Key: salary DESC

```

در این صورت تنها cost رو داریم اما اگر بخواهیم علاوه بر پلن ، زمان کلی رو بدست بیاریم ، باید آنالیز هم کنیم 

```sql
EXPLAIN ANALYZE SELECT * FROM employees
WHERE department_id = 10
ORDER BY salary DESC;

```

باید آنایز افزوده شود

```sql
                                                               Query Plan                                                                
----------------------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=160.00..165.00 rows=10 width=220) (actual time=3.345..3.737 rows=8 loops=1)
   Sort Key: salary DESC
   Sort Method: quicksort  Memory: 25kB
   ->  Seq Scan on employees  (cost=0.00..45.25 rows=10 width=220) (actual time=0.018..3.061 rows=8 loops=1)
         Filter: (department_id = 10)
         Rows Removed by Filter: 992
 Planning Time: 0.364 ms
 Execution Time: 3.789 ms
(7 rows)
```

میشه متوجه شد که میره ۱۰۰۰ تا سطر رو میبینه ، چون ایندکس idx_department_id   نداره پس این ایندکس رو اضافه میکنیم تا مستقیم تنها رو ۸ تا مورد سرچ بزنه


```sql
CREATE INDEX idx_department_id ON employees(department_id); 
```

```sql
                                                               Query Plan                                                                
----------------------------------------------------------------------------------------------------------------------------------------
 Sort  (cost=160.00..165.00 rows=10 width=220) (actual time=3.345..3.737 rows=8 loops=1)
   Sort Key: salary DESC
   Sort Method: quicksort  Memory: 25kB
   ->  Seq Scan on employees  (cost=0.00..45.25 rows=10 width=220) (actual time=0.018..3.061 rows=8 loops=1)
         Filter: (department_id = 10)
         Rows Removed by Filter: 992
 Planning Time: 0.364 ms
 Execution Time: 3.789 ms
(7 rows)
```



یک مثال از تجربه عملی که تونستم یک کوییری رو اوپتیمایز کنم :

```json

 

{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "2.75"   //   هزینه بالاست
    },
    "table": {
      "table_name": "incoming_payment_requests",
      "access_type": "ALL",
      "rows_examined_per_scan": 25,  // ۲۵ تا رو اسکن کرده
      "rows_produced_per_join": 0,
      "filtered": "4.00",  //  درصد فیلترا به دردش خورده 
      "cost_info": {
        "read_cost": "2.65",
        "eval_cost": "0.10",
        "prefix_cost": "2.75",
        "data_read_per_join": "5K"
      },
      "used_columns": [
        "amount",
        "created_at",
        "status",
        "bank_id"
      ],
      "attached_condition": "((`open_banking`.`incoming_payment_requests`.`bank_id` = 'sadad') and (`open_banking`.`incoming_payment_requests`.`status` in ('SUCCESS','READY_FOR_EXECUTE')))"
    }
  }
}

```

حالا بعد از ایندکس کردن

```json


{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "0.72"  // هزینه
    },
    "table": {
      "table_name": "incoming_payment_requests",
      "access_type": "range",
      "possible_keys": [
        "idx_metrics_cover"
      ],
      "key": "idx_metrics_cover",
      "used_key_parts": [
        "bank_id",
        "status",
        "created_at"
      ],
      "key_length": "2050",
      "rows_examined_per_scan": 2,   // کلا ۲ تا اسکن کرده
      "rows_produced_per_join": 2,   // ۲ تا نیازش بوده
      "filtered": "100.00",   ‍// کامل مفید بوده
      "using_index": true,   // **عالی**
      "cost_info": {
        "read_cost": "0.52",
        "eval_cost": "0.20",
        "prefix_cost": "0.72",
        "data_read_per_join": "11K"
      },
      "used_columns": [
        "amount",
        "created_at",
        "status",
        "bank_id"
      ],
      "attached_condition": "((`open_banking`.`incoming_payment_requests`.`bank_id` = '11') and (`open_banking`.`incoming_payment_requests`.`created_at` >= TIMESTAMP'2025-05-22 00:00:00') and (`open_banking`.`incoming_payment_requests`.`status` in ('SUCCESS','READY_FOR_EXECUTE')))"
    }
  }
}


```

### vacuum

در پستگرس (PostgreSQL)، دستور VACUUM به منظور پاکسازی و بهینه‌سازی دیتابیس استفاده می‌شود. وقتی داده‌ها در پایگاه‌داده حذف یا به‌روزرسانی می‌شوند، فضای حافظه‌ای که اشغال کرده‌اند به‌طور خودکار آزاد نمی‌شود. این موضوع باعث می‌شود که پایگاه‌داده به مرور زمان حجیم‌تر شود و کارایی آن کاهش یابد. دستور VACUUM به این کمک می‌کند که این فضای آزاد شده را دوباره بازیابی کرده و عملکرد سیستم را بهبود بخشد.

وقتی که رکوردی در جدول حذف یا به‌روزرسانی می‌شود، پستگرس به جای حذف فیزیکی رکورد از دیسک، آن را علامت‌گذاری به عنوان "حذف شده" می‌کند

همچنین میشه پلنر رو آپدیت کرد ، شاید داده ها قدیمی شدن یا پاک شدن پس نیازه که با وکییوم ، بروز بشن


**tip** 

با این که **autovacuum** به صورت خود کار اجرا میشه ، ولی چرا باز این دستور دستی میشه زد ؟

شاید جداول خیلی بزرگ رو نتونه autovacuum به صورت کامل اجرا کنه
 ، همچنین گاهی چون این درخواست زمان زیادی می برد خود پستگرس اتوماتیک انجام نمی دهد


**Tuple**

در حقیقت همان سطر های موجود در تیبل هستند و چند نوع دارند

+ **Dead tuple**

تاپل هایی که اکتیو نیستن و توی کوییری ها نمایش داده نمیشه اما هنوز پاک نشدن ، اینا یا  سطر قبل update بودن یا delete شدن اما به هر ترتیب هستند و نیاز به پاک سازی دارن

+ **Live tuple**

اینا واقعن سطر های موجود و درست هستند



`VACUUM ANALYZE employees;`

**ANALYZE**: آمار و اطلاعات مربوط به جدول employees را به‌روزرسانی می‌کند.


می تونیم بعد از این درخواست نتیجه رو ببینیم

`SELECT * FROM pg_stat_user_tables WHERE relname = 'employees';`

با به‌روزرسانی آمار، بهینه‌ساز (Query Planner) قادر خواهد بود که بهترین برنامه‌های اجرایی را برای کوئری‌ها انتخاب کند.

نتیجه وکیووم


 schemaname |   relname   | seq_scan | seq_tup_read | idx_scan | idx_tup_fetch | n_tup_ins | n_tup_upd | n_tup_del | n_tup_hot | n_live_tup | n_dead_tup | last_vacuum | last_autovacuum | last_analyze | last_autoanalyze | vacuum_count | analyze_count 
-------------+-------------+----------+---------------+----------+----------------+-----------+-----------+-----------+-----------+-------------+-------------+--------------+-----------------+--------------+-------------------+--------------+----------------
 public      | employees   | 50       |        50000 |     1000 |            800 |      2000 |      1000 |      500  |      300  |       48000 |        2000 | 2023-11-01   |                 | 2023-11-01   |                   |            2 |              3



**query planner**

یه سری اطلاعات مانند تعداد سطر ها ، تعدادایندکس ها  ، با استفاده از این آمار تصمیم می گیره برای مثال اگر تعداد جدول کم باشد تصمیم میگیرد به جای رفتن سراغ جدول ایندکس و جویین زدن ، داخل همون جدول سرچ کند (نادیده گرفتن ایندکس) اما اگر آمار رو بروز کنیم شاید بره از ایندکس داده رو بیاره شاید تعداد تاپل ها زیاد شده اما در آمار کمه


### Enabling Logging

میتونیم با تغییر این فایل لاگ رو فعال کنیم : `postgresql.conf`

```
log_min_duration_statement = 1000  # Log queries taking longer than 1000 ms (1 second)
log_statement = 'all'               # Log all statements (optional)
log_directory = 'pg_log'            # Directory for log files
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'  # Log file naming pattern

```
و لاگ رو ببینیم 

`tail -f /path/to/your/pg_log/postgresql-*.log`

بعد از فعال کردن لاگ ، این طور میبینیم :


+ همچنین میشه لاگ ها رو آپشنال کرد و مشاهده کرد ، در مثال زیر کوییری هایی که خیلی سرعت کمی دارند رو میشه مشاهده کرد

SET log_min_duration_statement = 1000; -- log queries taking longer than 1000 ms



```sql
2024-10-19 09:02:30.789 UTC [12347] LOG:  duration: 10 ms  statement: INSERT INTO employees (name, department, salary) VALUES ('John Doe', 'Sales', 50000);

2024-10-19 09:01:10.456 UTC [12346] LOG:  duration: 2000 ms  statement: UPDATE employees SET salary = salary * 1.1 WHERE department = 'Marketing';

2024-10-19 09:00:00.123 UTC [12345] LOG:  duration: 1500 ms  statement: SELECT * FROM employees WHERE department = 'Sales';

2024-10-19 09:06:30.123 UTC [12351] LOG:  statement: BEGIN;

2024-10-19 09:06:35.456 UTC [12351] LOG:  statement: COMMIT;

2024-10-19 09:07:00.789 UTC [12352] ERROR:  relation "non_existent_table" does not exist
2024-10-19 09:07:00.789 UTC [12352] STATEMENT:  SELECT * FROM non_existent_table;


```
### pg_stat_activity

با استفاده از این میتونیم ببینیم کدام کلاینت کانکت شده و چه کاری انجام میده و چه کوییری هایی زده و چقد زمان برده اطلاعاتی مانند ip و یوزرنیم و ... کار بر توش هست

```sql

SELECT pid, usename, state, query, now() - query_start AS duration
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;
```

SELECT * FROM orders WHERE status = 'pending';      ------- 00:00:45
UPDATE products SET stock = stock - 1 WHERE id = 1001;   -- 00:00:30


### Monitoring Logs
می تونیم لاگ هایی که مورد نظرمونه رو نمایش بدیم و نگه داریم

با استفاده از تولز هایی این کار رو انجام میدیم
`postgres_exporter`

### pg_stat_statements
ابتدا باید فعالش کنیم و ببینیم هر کوییری چقدر هزینه بر بوده :

`CREATE EXTENSION pg_stat_statements;`


```sql
SELECT
    query,
    calls,
    total_time,
    rows,
    mean_time,
    stddev_time
FROM
    pg_stat_statements
ORDER BY
    total_time DESC
LIMIT 10;
```

پاسخ این کوییری این است که بیشترین کوییری ها که تکرار شدن ، چقد زمان بردن


### work_mem

ظرفیت حافظه کاری که برای هر کوییری استفاده می شود و داخل آن ریخته میشود

در حقیقت هر بار که کوییری میزنیم کلی داده تمییز نشده که بر اساس sort , where , groupby  جدا جدا در مموری میاد و میتونیم سقف ظرفیت رو بالا ببریم

`SET work_mem = '16MB';`

### shared_buffers

این حافظه ی `cache` است نتایج اخیر رو ذخیره میکنه  به صورت پیش فرض 128 MB است و میتونیم آن را بالا ببریم

`SHOW shared_buffers;`

SET shared_buffers = '128MB';`
### pg_locks

با استفاده از این دستور میتونیم ببینیم که کدام لاک رو استفاده میکنه و چه حالتی داره

همچنین با تنظیم انواع ترن اکشن ها می تونیم با توجه به بیزینس ، سرعتو بهبود ببخشیم :



**Access Share Lock**
SELECT

**Row Exclusive Lock**

INSERT, UPDATE, or DELETE

**Share Lock**

SELECT FOR UPDATE


```sql

SELECT
    pid,
    mode,
    relation::regclass AS relation,
    transactionid,
    virtualtransaction,
    state,
    granted
FROM
    pg_locks
WHERE
    NOT granted;


```



## سرعت نوشتن (write) کند شده ، چه کا کنیم ؟



سوال : آیا **insert** هم میشه explain analyze کرد؟



#### بررسی ایندکس ها 

همون طور که سرعت خوندن رو زیاد می کنه اگر داده ها زیاد شه و ایندکس ها  زیاد شه ، شاید برای هر سطر باید چندین جدول ایندکس ، بهش سطر اضافه بشه

#### Batch Insert

#### WAL level

 از WAL level مناسب استفاده کنید و تنظیمات checkpoint را بهینه کنید.


#### sharding

#### transaction isolation level


#### synchronous_commit = OFF
 میتونیم با  fsync and synchronous_commit   بک آپ برای کرش یا سوختگی رو خاموش کنیم ، اما خطرناکه

Set synchronous_commit = OFF;  -- Not recommended without understanding the implications

### همچنین اگه سرعت نوشتن اومده باشه پایین باید چی کار کرد؟

+ بررسی کنیم i/o سخت افزاری رو ابتدا ببینیم هارد دیسک کند نشده ؟ و اینکه شاید از سمت سیستم عامل و در کل ربطی به پایگاه داده نیست

+ شاید WAL رو خوب مدیریت نکردیم و همه چیز رو داره ارشیو می کنه و پاک نمی کنه و بررسی کانفیگ archive_mode and archive_command

+ شاید  **Table Bloat**  رخ داده ، یعنی بر اثر تغییر های زیاد مانند آپدیت و دیلیت مدیریت حافظه از بهینه در اومده و با استفاده **VACUUM FULL** میشه بهبود دد البته توجه کنید این کار تیبل رو لاک می کنه .  توضیح در باره ی **Table bloat** این پدیده زمانی می گیم که حافظه ی تخصیص داده شده برای ذخیره بیشتر از مقدار مورد نیاز برای ذخیره سازی داده ها استفاده می شود. و زمانی رخ میدهد که تعداد تغییرات تاپل ها زیاد است ، دلایل زیر می توان باعث این دیده شه : 
 MVCC (Multi-Version Concurrency Control) و Frequent Updates and Deletes 


+ بررسی **Lock Contention** با استفاده از `pg_stat_activity`  و همچنین با استفاده از `pg_locks` میتونیم ببینیم که کدام لاک رو استفاده میکنه و چه حالتی داره

+ بررسی ایندکس ها اینکه همه چیز رو الکی ایندکس کردیم همچنین نگاه به کانفیگ shared_buffers, work_mem, maintenance_work_mem

+ اگه همه چیز اوکی بود باید با افزایش مموری یا ssd و تکنیک های شاردینگ درستش کنیم


### Optimistic vs	Pessimistic locking
قبلش توجه شه این دو به ترنس اکشن ارتباطی ندارن ولی می تونن تو ترنساکشن باشن

+ Optimistic

در مورد خوشبینانه می تونیم از ترسن اکشن استفاده نکنیم (می تونیم استفاده بکنیم) و با استفاده از فیلد ورژن بندی مانند `updated_at` چک کنیم هنوز فیلد این مقدار رو داره یا نه ، اگر `affected_rows` درست بود یعنی اوکیه ، این مورد بهتره تو موارد استفاده شه که ریس کاندیشن نادر صورت میگیره ، زیرا به جای این که کلاینت منتظر لاک بمونه باید خطا دریافت کنه

+ Pessimistic

در این حالت با استفاده از `update for` اون کوییری رو قفل می کنیم  تا کسی نتونه ریتریو کنه و بعد از اتمام ترسن اکشن  باز می کنیم

توجه شه که همه تیبل لاک نمی شه بلکه اونایی که تو شرط هستند ، اگر جواب `select` یک سطر باشه ، همون لاک میشه ، باید شرط لاک رو خیلی خیلی محدود کرد




### سوال ،

یک سطر داریم و خیلی از کلاینت ها میخوان یک مقدار مشخص رو تغییر بدن ، مثلا اعتبار یک شرکت ، شاید چندین یوزر بخوان اون رو تغییر بدن ، چگونه هم زیس کاندیشن رخ نده هم پرفورمنس خوبی داشته باشیم .جواب ، چندین راه وجود دارد:

+ atomic operation

در این روش به جای این که مقدار فعلی رو بگیریم ، خودمون مبلقی کم کنیم یا اضافه کنیم ، اون رو توی دیتابیس انجام میدیم ، مثلا به خود دیتابیس میگیم به مقدار فعلی ۲۰۰ هزار تومن اضافه کن

```sql

UPDATE counters
SET value = value + 1
WHERE id = 123;
```



###### tips
توجه شود این دو روش هایی هستند که جدای از ایجاد `tranaction ` هستند برای مثال اگر بخواهیم تغییرات در سطح چندین تیبل باشد نیاز داریم `tranaction` ایجاد کنیم

 


