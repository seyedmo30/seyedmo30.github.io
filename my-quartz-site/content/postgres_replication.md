
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



\c db

wal_level = logical

listen_addressess = "*" '*'


pg_hba.conf

host    sampledb    replicauser     192.168.10.10/32   scram-sha-256


CREATE PUBLICATION bookpub FOR TABLE book;

CREATE PUBLICATION my_publication FOR ALL TABLES;



CREATE SUBSCRIPTION booksub CONNECTION 'dbname=sampledb host=192.168.10.5 user=replicauser password=12345 port=5432' PUBLICATION bookpub



select * from pg_catalog.pg_publication;  

\dRp+




select * from pg_catalog.pg_subscription;

\dRs+



select * from pg_replication_slots ;

select pg_drop_replication_slot('booksub');


ALTER SUBSCRIPTION booksub DISABLE;
ALTER SUBSCRIPTION booksub SET (slot_name=NONE);
DROP SUBSCRIPTION booksub;


DROP publication bookpub ;


---------------------------------------------------------

postgresql.conf

wal_level = logical
max_replication_slots = 5
max_wal_senders = 10


SELECT * FROM pg_create_logical_replication_slot('replication_slot', 'pgoutput');


SELECT * FROM pg_replication_slots;


psql "dbname=postgres replication=database" -U postgres -h 192.168.13.248 -c "CREATE_REPLICATION_SLOT my_slot3 LOGICAL pgoutput;"

psql "dbname=phonebook replication=database" -U postgres -h 192.168.13.248 -c "IDENTIFY_SYSTEM;"


postgres://pglogrepl:secret@127.0.0.1/pglogrepl?replication=database


pg_receivewal (for physical replication)
https://www.postgresql.org/docs/current/app-pgreceivewal.html


 pg_recvlogical (for logical replication).
 https://www.postgresql.org/docs/current/app-pgrecvlogical.html
