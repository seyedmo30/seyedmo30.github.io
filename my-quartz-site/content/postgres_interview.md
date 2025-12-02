# سوالات مصاحبه: PostgreSQL، تراکنش‌ها، لاک‌ها و `pgx`

یک مجموعهٔ کامل سؤال و پاسخ (به فارسی) برای آماده‌شدن در مصاحبه‌های فنی که روی PostgreSQL، تراکنش‌ها، locking و کدنویسی با `pgx` تمرکز دارند. شامل مثال‌های کد و الگوهای عملی.

---

# بخش ۱ — مفاهیم پایه (Basic)

## 1) سوال: PostgreSQL چطوری قفل‌ها (locks) را مدیریت می‌کنه؟

**پاسخ نمونه:** PostgreSQL از MVCC (Multi-Version Concurrency Control) استفاده می‌کند؛ خواندن معمولاً نسخه‌ای از داده می‌گیرد بدون قفلِ نوشتن. قفل‌های واقعی (row-level locks مثل `FOR UPDATE`, table locks, advisory locks) زمانی اعمال می‌شوند که تراکنش‌ها بخواهند عملیات نویسندگی انجام دهند. این باعث می‌شود خواندن بدون بلوکه شدن در بسیاری از موارد انجام شود و همزمان نوشتن با قفل سطح ردیف هماهنگ شود.

## 2) سوال: فرق بین `SELECT ... FOR UPDATE` و `SELECT ... FOR SHARE` چیه؟

**پاسخ:** `FOR UPDATE` ردیف‌ها را برای نوشتن قفل می‌کند (exclusive lock) و مانع از اینکه تراکنش‌های دیگر آنها را برای نوشتن تغییر دهند. `FOR SHARE` قفل اشتراکی می‌گیرد (چند تراکنش می‌توانند FOR SHARE بگیرند) و مناسبِ مواردی است که می‌خواهی از حذف/تغییر ردیف جلوگیری کنی اما اجازهٔ خواندن همزمان را بدهی.

## 3) سوال: isolation levels در Postgres کدام‌اند و default کدام است؟

**پاسخ:** Postgres از `Read Committed` (پیش‌فرض)، `Repeatable Read` و `Serializable` پشتیبانی می‌کند. `Read Committed` آخرین committed view را در هر statement می‌بیند. `Serializable` سختگیرانه‌ترین است و ممکن است خطاهای serialization (SQLSTATE = `40001`) تولید کند که نیاز به retry دارد.

## 4) سوال: deadlock چیست و PostgreSQL چگونه آن را تشخیص می‌دهد؟

**پاسخ:** deadlock زمانی رخ می‌دهد که دو یا چند تراکنش به‌صورت چرخشی منتظر منابعی باشند که دیگری نگه داشته. Postgres یک مکانیزم تشخیص deadlock دارد؛ وقتی deadlock پیدا شد یکی از تراکنش‌ها با خطای `deadlock detected` (SQLSTATE `40P01`) rollback می‌شود؛ معمولاً باید آن تراکنش را retry کرد.

---

# بخش ۲ — عملیاتی و best practices

## 5) سوال: بهترین‌ روش برای نوشتن تراکنش‌ها در اپلیکیشن چیست؟

**پاسخ:** تراکنش‌ها کوتاه نگه داشته شوند، هیچ IO خارجی (HTTP، long CPU) داخل تراکنش انجام نشود، از timeout/`lock_timeout` و `idle_in_transaction_session_timeout` استفاده شود، و منطق تراکنش در لایهٔ سرویس یا UnitOfWork مدیریت شود نه در هر repository.

## 6) سوال: Outbox pattern چیست و چرا توصیه می‌شود؟

**پاسخ:** Outbox pattern یعنی در همان تراکنش DB که state را تغییر می‌دهی، یک ردیف event در جدول outbox بنویسی؛ پس از commit یک worker آن ردیف را می‌فرستد (مثلاً به Kafka). این روش تضمین می‌کند state و event هم‌زمان هستند و از پیام‌های گم‌شده جلوگیری می‌کند.

## 7) سوال: برای workerهای پردازش queue چطور rows را به صورت امن claim کنیم؟

**پاسخ:** از `SELECT ... FOR UPDATE SKIP LOCKED` یا از CTE با `FOR UPDATE SKIP LOCKED` استفاده کن تا ردیف‌هایی که در حال پردازش‌اند به بقیه ندهی؛ این الگو برای مصرف‌کننده‌های موازی عالیه.

---

# بخش ۳ — `pgx` و الگوهای Go (متداول در مصاحبه)

## 8) سوال: چطور تراکنش با `pgx` باز/کامیت/برک‌بک می‌کنیم؟ (مثال کوتاه)

**پاسخ و کد نمونه:**

```go
tx, err := pool.Begin(ctx)
if err != nil { return err }
defer func() { _ = tx.Rollback(ctx) }()
if _, err := tx.Exec(ctx, "UPDATE accounts SET balance = balance - $1 WHERE id=$2", amt, id); err != nil { return err }
if err := tx.Commit(ctx); err != nil { return err }
```

## 9) سوال: چطور خطای serialization (`40001`) و deadlock را هندل و retry کنیم؟

**پاسخ:** خطاها را چک کن و در صورت SQLSTATE `40001` یا `40P01`، با backoff محدود retry کن. در `pgx` می‌توانی خطا را به `*pgconn.PgError` تبدیل کنی و `pgErr.Code` را بررسی کنی.

نمونه:

```go
if perr, ok := err.(*pgconn.PgError); ok {
    if perr.Code == "40001" || perr.Code == "40P01" {
        // retry logic
    }
}
```

## 10) سوال: connection-level vs session-level locks (advisory locks) چگونه است؟

**پاسخ:** Advisory locks در Postgres قفل‌های برنامه‌ای هستند که توسط `pg_try_advisory_lock`/`pg_advisory_unlock` کار می‌کنند و روی سشن (کانکشن) هستند. اگر از connection pool استفاده می‌کنید، حتما مطمئن شوید قفل را روی همان کانکشنی می‌گیرید که قرار است تا آزاد شود (معمولاً درون تراکنش/قاب connection-specific).

مثال گرفتن advisory lock:

```go
var got bool
err := conn.QueryRow(ctx, "SELECT pg_try_advisory_lock($1)", key).Scan(&got)
```

---

# بخش ۴ — سوالات میانی (Intermediate)

## 11) سوال: `SKIP LOCKED` چه وقتی مفیده و چه موقع خطرناک است؟

**پاسخ:** مفید برای worker‌هایی که می‌خواهند سریع ردیف‌های آزاد را بگیرند و نخواهند برای قفل شدن معطل شوند (concurrent consumers). خطرش اینه که بعضی ردیف‌ها ممکنه برای مدت طولانی پردازش نشوند اگر consumerها سریع بخش‌های دیگری را بگیرند؛ همچنین ترتیب پردازش تضمین نمی‌شود.

## 12) سوال: savepoint چیست و چه وقت ازش استفاده می‌کنیم؟

**پاسخ:** savepoint به‌عنوان sub-transaction عمل می‌کند؛ می‌توان بخشی از تراکنش را rollback کرد بدون rollback کل تراکنش. مفید است وقتی نیاز به rollback محلی داری اما تراکنش کلی را ادامه دهی.

مثال:

```sql
SAVEPOINT sp1;
-- do stuff
ROLLBACK TO SAVEPOINT sp1; -- on error
RELEASE SAVEPOINT sp1; -- on success
```

## 13) سوال: چگونه از `SELECT ... FOR UPDATE` برای پیاده‌سازی lock-based concurrency control استفاده می‌کنی؟

**پاسخ:** برای مثال برای کاهش موجودی انبار، ابتدا ردیف محصول را با `FOR UPDATE` می‌گیری تا دیگر تراکنش‌ها نتوانند هم‌زمان آن ردیف را تغییر دهند، سپس مقدار را کاهش داده و commit کنی. این تضمین می‌کند تغییرات سریالی اجرا شوند.

## 14) سوال: `Serializable` چه مشکلاتی ایجاد می‌کند و چرا باید برایش retry نوشت؟

**پاسخ:** `Serializable` اجازهٔ اجرای تراکنش‌ها را می‌دهد به‌صورتی که نتیجه معادل یک سریال (یک‌در-یک) باشد. Postgres ممکن است برای حفظ این شرط، یکی از تراکنش‌ها را با خطای serialization failure قطع کند؛ بنابراین کد باید retry داشته باشد. این isolation برای correctness قویه اما throughput کمتری می‌دهد و نیاز به retry دارد.

---

# بخش ۵ — سوالات پیشرفته (Advanced)

## 15) سوال: explain کنید `MVCC` چگونه phantom read و non-repeatable read را هندل می‌کند.

**پاسخ:** MVCC به هر transaction یک snapshot از داده‌ها می‌دهد. در `Read Committed` هر statement snapshot جدید می‌گیرد، بنابراین non-repeatable reads امکان‌پذیرند. در `Repeatable Read` و `Serializable` snapshot برای کل تراکنش ثابت می‌ماند، بنابراین non-repeatable reads و phantom reads بسته به رفتار isolation حذف می‌شوند.

## 16) سوال: چطور deadlockهای پیچیده را debug می‌کنی در production؟

**پاسخ:** از لاگ‌های Postgres (`log_lock_waits`, `deadlock_timeout`) و trace queryها استفاده کن، لاگ queries مربوط را جمع‌آوری کن، از `pg_stat_activity` و `pg_locks` برای مشاهدهٔ وضعیت قفل‌ها استفاده کن، و الگوها (مثلاً order of updates) را اصلاح کن تا از قفل‌گذاری به ترتیب جلوگیری شود.

## 17) سوال: چرا نباید کارهای شبکه‌ای (HTTP) داخل تراکنش انجام داد؟ چه الگوهای جایگزین موجودند؟

**پاسخ:** چون تراکنش طولانی می‌شود و قفل‌ها را برای مدت طولانی نگه می‌دارد که باعث contention و deadlock می‌شود. الگوهای جایگزین: نوشتن state در DB + outbox و ارسال پیام پس از commit، یا جدا کردن کار طولانی به یک job async که پس از commit اجرا می‌شود.

## 18) سوال: explain `FOR UPDATE SKIP LOCKED` + CTE pattern برای claim‌کردن batch از ردیف‌ها.

**پاسخ و مثال:**

```sql
WITH cte AS (
  SELECT id FROM jobs WHERE state='pending' FOR UPDATE SKIP LOCKED LIMIT 10
)
UPDATE jobs
SET state='processing', locked_at = now()
WHERE id IN (SELECT id FROM cte)
RETURNING id, payload;
```

این الگو atomically ردیف‌ها را claim می‌کند و مناسب workerهای موازی است.

---

# بخش ۶ — سوالات طراحی و معماری

## 19) سوال: وقتی همه سرویس‌ها روی یک DB مرکزی هستند آیا باید همه‌جا تراکنش باز شود یا UnitOfWork؟

**پاسخ:** بهترین‌عمل این است که تراکنش‌ها در لایهٔ سرویس/UseCase یا یک UnitOfWork مدیریت شوند و `tx` را به repositoryها پاس بدهی. این رویکرد composability، تست‌پذیری و شفافیت را حفظ می‌کند.

## 20) سوال: وقتی از connection pool استفاده می‌کنی، چه نکات قفلی باید بدانی؟

**پاسخ:** advisory lockها session/connection-level هستند — اگر connection pool یک کانکشن آزاد بده و بعد lock در connection دیگری گرفته شود، رفتار unexpected است؛ همچنین تراکنش‌های طولانی کانکشن را نگه داشته و pool exhaustion ایجاد می‌کنند؛ بنابراین timeout و محدودیت‌ها را تنظیم کن.

---

# بخش ۷ — سوالات کدنویسی/تمرینی (Coding)

## 21) سوال تمرینی: بنویس کدی که با pgx یک تراکنش باز کند، دو update اجرا کند و اگر deadlock یا serialization failure آمد با retry انجام بده (حداکثر 3 بار).

**پاسخ نمونه (ساختاری):**

```go
func doTx(ctx context.Context, pool *pgxpool.Pool) error {
    maxRetries := 3
    for i := 0; i < maxRetries; i++ {
        tx, err := pool.Begin(ctx)
        if err != nil { return err }
        // ensure rollback if anything wrong
        defer tx.Rollback(ctx)

        // do updates...
        if _, err := tx.Exec(ctx, "UPDATE ..."); err != nil {
            if isRetryable(err) {
                tx.Rollback(ctx)
                // backoff
                time.Sleep(time.Duration(i+1) * 100 * time.Millisecond)
                continue
            }
            return err
        }

        if err := tx.Commit(ctx); err != nil {
            if isRetryable(err) {
                // retry
                time.Sleep(time.Duration(i+1) * 100 * time.Millisecond)
                continue
            }
            return err
        }
        return nil
    }
    return fmt.Errorf("max retries exceeded")
}
```

`isRetryable` باید خطاهای `pgconn.PgError` را برای کد `40001` یا `40P01` بررسی کند.

## 22) سوال تمرینی: چگونه idempotency برای endpointی که پرداخت را انجام می‌دهد پیاده‌سازی می‌کنی؟

**پاسخ:** از یک جدول `idempotency_keys` یا unique constraint بر روی `request_id` استفاده کن. هنگام درخواست، داخل تراکنش بررسی و درج کن؛ اگر موجود است، پاسخ قبلی را برگردان یا عملیات را نادیده بگیر.

نمونه SQL:

```sql
INSERT INTO idempotency_keys (request_id, response, created_at)
VALUES ($1, $2, now())
ON CONFLICT (request_id) DO NOTHING;
```

---

# بخش ۸ — سوالات tricky / edge cases

## 23) سوال: چه تفاوتی بین `LOCK TABLE ... IN SHARE MODE` و `FOR UPDATE` هست؟

**پاسخ:** `LOCK TABLE` قفل در سطح جدول می‌گذارد و تأثیر بزرگ‌تری دارد؛ `FOR UPDATE` قفل در سطح ردیف می‌گیرد. معمولاً قفل ردیفی (ریزدانه‌تر) بهتر است مگر این‌که نیاز به حفظ consistency سراسری داشته باشی.

## 24) سوال: اگر یک تراکنش commit کند و بعد worker نتواند پیام outbox را ارسال کند چه کار می‌کنی؟

**پاسخ:** outbox worker باید retry و backoff داشته باشد. بهتر است `attempt_count` و `last_error` را در outbox نگه داری و از backoff/alerting و DLQ (dead letter queue) برای مواردی که تکرارها به خطا می‌خورند استفاده کنی.

## 25) سوال: چگونه queryهای طولانی و تراکنش‌های طولانی را از هم جدا می‌کنی تا قفل‌ها روی داده‌های مهم نگیرند؟

**پاسخ:** queryهای گزارش‌گیری/تحلیلی را روی replica اجرا کن، تراکنش‌های کوتاه بنویس، از indexها و batching استفاده کن و از `lock_timeout` یا `statement_timeout` برای جلوگیری از اثرات بد استفاده کن.

---

# بخش ۹ — سوالات رفتاری/Operational

## 26) سوال: یک بار در production با lock contention مواجه شدی — چه کار کردی؟

**پاسخ نمونه:** ابتدا `pg_stat_activity` و `pg_locks` را چک کردم، queries درگیر را کشف و ترتیب قفل‌گذاری را اصلاح کردم (مثلاً همگی یک ترتیب lock بگیرند). در بعضی موارد logic را refactor کردم تا تراکنش‌ها کوتاه شوند و از outbox استفاده کردم. سپس monitoring اضافه کردم تا به محض رشد wait eventها هشدار دریافت کنیم.

---

# بخش ۱۰ — نکات نهایی و checklist برای مصاحبه‌گر

* همیشه روی مفاهیم: MVCC، isolation levels، deadlocks، locking primitives (`FOR UPDATE`, `SKIP LOCKED`, advisory locks) تسلط داشته باش.
* بلد باش نمونه‌کد تراکنش در Go/pgx بنویسی و خطاهای قابل retry را تشخیص بدی.
* بحث در مورد trade-offs: `Serializable` ایمنی بالا ولی هزینهٔ کارایی و retry دارد. `Read Committed` کاراتر ولی anomalies ممکنه رخ دهد.
* آماده باش دربارهٔ الگوهای عملیاتی: Outbox, Saga, UnitOfWork, Savepoints، و `SKIP LOCKED`.

---

*اگر بخواهی این سند را به PDF یا فلش‌کارت تبدیل کنم یا یک شبیه‌ساز ۲۰ سواله برای تمرین بسازم، بگو تا برات مرتبش کنم.*
