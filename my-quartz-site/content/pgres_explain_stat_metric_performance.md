# pgbench

یکی از ابزار های خوب برای seed - dummy data - و نمایش تعداد و سرعت همین ابزار هست


### insert

بعد از طراحی دیتابیس می تونیم با استفاده از ai دیتای رندوم بسازیم و با یه دیتابیس پر دیتا ، اون رو آنالیز کنیم و بهبود بدیم

فرض کنیم که میخواهیم ۲ جدول زیر رو پر کنیم :

```sql

-- public.contract definition

CREATE TABLE public.contract ( id uuid DEFAULT gen_random_uuid() NOT NULL, user_id varchar(60) NOT NULL, external_id varchar(60) DEFAULT gen_random_uuid() NOT NULL, max_daily_transaction int2 NULL, max_monthly_transaction int2 NULL, max_transaction_amount int8 NULL, withdrawal_permission bool DEFAULT false NOT NULL, bank varchar(60) NULL, bill_payment_permission bool DEFAULT false NOT NULL, provider varchar(60) NULL, status varchar(60) NULL, provider_code varchar(60) NULL, provider_reference varchar(60) NULL, start_time timestamp NULL, expire_time timestamp NULL, create_time timestamp NULL, update_time timestamp DEFAULT CURRENT_TIMESTAMP NULL, deposit_number varchar(60) NULL, CONSTRAINT contract_pkey PRIMARY KEY (id));
CREATE INDEX bank_index ON public.contract USING btree (bank);
CREATE INDEX idx_external_id ON public.contract USING btree (external_id);
CREATE INDEX idx_provider_reference ON public.contract USING btree (provider_reference);
CREATE INDEX provider_code_index ON public.contract USING btree (provider_code);
CREATE UNIQUE INDEX provider_code_unique ON public.contract USING btree (provider_code);
CREATE INDEX provider_index ON public.contract USING btree (provider);
CREATE INDEX status_index ON public.contract USING btree (status);
CREATE UNIQUE INDEX unique_external_id ON public.contract USING btree (external_id);
CREATE INDEX user_id_index ON public.contract USING btree (user_id);


-- public.pay definition

CREATE TABLE public.pay ( id uuid DEFAULT gen_random_uuid() NOT NULL, external_id varchar(60) DEFAULT gen_random_uuid() NOT NULL, contract_id uuid NOT NULL, reference_id varchar(60) NULL, amount int8 NOT NULL, status varchar(60) NULL, create_time timestamp NULL, transaction_time timestamp NULL, peyman_batch varchar(60) NULL, update_time timestamp DEFAULT CURRENT_TIMESTAMP NOT NULL, destination_bank varchar(100) NULL, destination_deposit varchar(100) NULL, source_bank varchar(100) NULL, source_deposit varchar(100) NULL, CONSTRAINT pay_pkey PRIMARY KEY (id), CONSTRAINT fk_contract_id FOREIGN KEY (contract_id) REFERENCES public.contract(id));
CREATE INDEX contract_id_index ON public.pay USING btree (contract_id);
CREATE INDEX status_index_pay ON public.pay USING btree (status);
CREATE UNIQUE INDEX unique_external_id_pay ON public.pay USING btree (external_id);

```



میتونیم با استفاده از این ۲ تا فایل ، داده رو پر کنیم :


برای contract:
```sql

-- file: seed_contracts.sql

\set user_id random(1, 1000000)
\set max_daily_transaction random(1, 100)
\set max_monthly_transaction random(1, 3000)
\set max_transaction_amount random(1000, 1000000)
\set withdrawal_permission random(0, 1)
\set bill_payment_permission random(0, 1)

INSERT INTO public.contract
(user_id, external_id, max_daily_transaction, max_monthly_transaction, max_transaction_amount,
 withdrawal_permission, bank, bill_payment_permission, provider, status, provider_code,
 provider_reference, start_time, expire_time, create_time, deposit_number)
VALUES
(:user_id,
 md5(random()::text), -- random external_id
 :max_daily_transaction,
 :max_monthly_transaction,
 :max_transaction_amount,
 (:withdrawal_permission)::boolean,
 'bank_' || md5(random()::text),
 (:bill_payment_permission)::boolean,
 'provider_' || md5(random()::text),
 'active',
 'provcode_' || md5(random()::text),
 'provref_' || md5(random()::text),
 now(),
 '2026-01-01 00:00:00',
 now(),
 'dep_' || md5(random()::text)
);
```


برای pay :
```sql

-- file: seed_pay.sql

\set amount random(1000, 1000000)

INSERT INTO public.pay
(external_id, contract_id, reference_id, amount, status, create_time, transaction_time,
 peyman_batch, destination_bank, destination_deposit, source_bank, source_deposit)
VALUES
(
 md5(random()::text), -- external_id
 (SELECT id FROM public.contract ORDER BY random() LIMIT 1), -- random contract_id
 md5(random()::text), -- reference_id
 :amount,
 'pending',
 now(),
 now(),
 'batch_' || md5(random()::text),
 'dest_bank_' || md5(random()::text),
 'dest_dep_' || md5(random()::text),
 'src_bank_' || md5(random()::text),
 'src_dep_' || md5(random()::text)
);
```

و در نهایت با دستور زیر اعمال میکنیم :


`pgbench -n -f pgbench_pay.sql -T 30  -d database_name`




# pg_stats

###  pg_stat_database

اطلاعات جنریک دیتابیس ها رو میده 

`SELECT * FROM pg_stat_database;`

اگر نسیت زیر کمتر از 95 درصد بود باید به شیرد بافر اضافه کنیم
```sql
SELECT datname,
       blks_hit::float / (blks_hit + blks_read) AS cache_hit_ratio,
       blks_hit, blks_read
FROM pg_stat_database
WHERE blks_hit + blks_read > 0;
‍‍‍
```


### pg_stat_statements

آمار کوییری هایی که زدیم رو میگه ، مثلا بیشتری کوییری های اینسرت یا سلکت و یا ...

چرا مفیده؟ می تونیم ببینیم کدوم کوییری بیشتر استفاده شده ، کوییری هایی که تو ۶ ماه بیشترین استفاده شده ، میشه با مدیریت ایندکس بهینش کرد ، قطعا کوییری که خیلی سرعتش پایینه اما به ندرت استفاده میشه مهم نیست ، مهم کوییری هست که شاید سرعتش معمولی باشه ولی اونقدر استفاده میشه اگر از معمولی اون رو بکنیم سریع ، کلی جلو می افتیم .

CREATE EXTENSION IF NOT EXISTS pg_stat_statements;


```sql
SELECT *
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```
#### ستون های مهم :

+ query --- متن کوییری
+ calls --- تعداد بار های فراخوانی
+ total_exec_time --- زمانی که صرف شده
+ rows --- تعداد سطر هایی که میاره ، میشه با calls  مقایسه کرد
+ shared_blks_hit, shared_blks_read, shared_blks_written --- چقد از کش استفاده می کنیم و یا از io





