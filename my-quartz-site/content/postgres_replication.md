
### wal


####  WAL physical (فیزیکی)

+ نیاز به همان نسخهٔ major PostgreSQL و تقریباً همان ساختار فایل‌ها دارد.
+ لاگ در سطح صفحات/بایت‌های داخلی دیتابیس نوشته می‌شود (صفحات 8KB مثلاً).
+ وقتی لاگ فیزیکی است  تغییرات schema هم اعمال می‌شود

#### WAL logical

+ لاگ روی تغییرات منطقی‌تر (insert/update/delete در سطح ردیف/tuple) تولید می‌کند — نه بایت به بایت صفحه.

+ با logical decoding و output plugins (مثل wal2json, pgoutput) استخراج می‌شود.

+ برای CDC / logical replication مفید است

#### می‌تونیم بدون استفاده از WAL، ETL پیاده‌سازی کنیم؟

بله — اما یا باید پول کنی هر چند ثانیه داده رو بگیری یا باید تیریگر بنویسی و خب خیلی کارای سخت  و احتمال از دست دادن داده یا تکراری خیلی هست ، تویآبدیت و دیلیک هم خیلی دشوار



### slot

یک نگهدارنده در primary است که تضمین می‌کند WAL تا وقتی یک مصرف‌کننده (standby یا logical decoder) آن را نخوانده، از بین نرود.

+ Physical replication slot:

 برای streaming standby‌ها استفاده می‌شود؛ جلو رفتن WAL توسط نشانگر مصرف‌کننده کنترل می‌شود.

+ Logical replication slot:

 برای logical decoding (CDC)؛ خروجی با plugin (wal2json, test_decoding و ...) خوانده می‌شود.

#### disk/slot bloat

اگر consumer دیر کند یا قطع شود و slot حذف نشود، WAL روی primary انباشته شده و دیسک پر می‌شود

برای جلوگیری از این کار عملا نمی شه کاری کرد ، فقط باید مایتور و آلارم خیلی خفن ماستفاده کرد




---

## پیکربندی نمونه و دستورات مفید

> این بخش شامل snippetهای عملی برای `postgresql.conf`, `pg_hba.conf`, ایجاد publication/subscription، و کار با replication slots است.

### 1) نمونه‌های پیکربندی در `postgresql.conf`

```conf
# فعال‌سازی logical replication
wal_level = logical
# تعداد اسلات‌های replication
max_replication_slots = 5
# تعداد senders برای streaming
max_wal_senders = 10
# (اختیاری) کاهش latency در sync replication
synchronous_commit = remote_write
```

### 2) نمونهٔ ورودی در `pg_hba.conf`

```conf
# اجازه به کاربر replicauser برای اتصال replication از IP مشخص
host    replication    replicauser     192.168.10.10/32   scram-sha-256
# یا اگر می‌خواهی publish/subscription از راه دور به DB عادی
host    sampledb       replicauser     192.168.10.10/32   scram-sha-256
```

### 3) ساخت Publication / Subscription (logical replication)

```sql
-- ایجاد publication برای یک جدول
CREATE PUBLICATION bookpub FOR TABLE book;

-- ایجاد publication برای همهٔ جداول
CREATE PUBLICATION my_publication FOR ALL TABLES;

-- در subscriber (روی سرور مقصد)
CREATE SUBSCRIPTION booksub
  CONNECTION 'dbname=sampledb host=192.168.10.5 user=replicauser password=secret port=5432'
  PUBLICATION bookpub;

-- مدیریت
ALTER SUBSCRIPTION booksub DISABLE;
ALTER SUBSCRIPTION booksub SET (slot_name = NONE);
DROP SUBSCRIPTION booksub;
DROP PUBLICATION bookpub;
```

### 4) کار با replication slots

```sql
-- نمایش اسلات‌ها
SELECT * FROM pg_replication_slots;

-- ایجاد logical slot با pgoutput
SELECT * FROM pg_create_logical_replication_slot('replication_slot', 'pgoutput');

-- ایجاد physical slot (مثال)
SELECT * FROM pg_create_physical_replication_slot('phys_slot_1');

-- حذف اسلات
SELECT pg_drop_replication_slot('slot_name');
```

### 5) ابزارهای جانبی و نمونهٔ connection string برای logical replication

```text
# connection string برای logical replication (pg_recvlogical / client)
postgres://pglogrepl:secret@127.0.0.1/pglogrepl?replication=database

# اجرای دستورات psql برای replication
psql "dbname=postgres replication=database" -U postgres -h 192.168.13.248 -c "CREATE_REPLICATION_SLOT my_slot3 LOGICAL pgoutput;"
psql "dbname=phonebook replication=database" -U postgres -h 192.168.13.248 -c "IDENTIFY_SYSTEM;"

# ابزارهای رسمی
pg_receivewal  (برای physical replication)
pg_recvlogical (برای logical replication)
# مستندات:
# https://www.postgresql.org/docs/current/app-pgreceivewal.html
# https://www.postgresql.org/docs/current/app-pgrecvlogical.html
```

