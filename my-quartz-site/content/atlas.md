# schema

این روش خیلی راحته و برای زمانیه که تنها می خواییم تغییرات رو فورس بر روی دیزایر  -- پیاده سازی کنیم

## inspect

برای کپی کردن ساختار از دیتابیس دولوپ

```
atlas schema inspect -u "mysql://hp:1qaz@WSX@localhost:3306/salam" > schema.hcl
```

## apply
برای پیاده سازی در پروداکت

```
atlas schema apply -u "mysql://root:1qaz@WSX@localhost:3307/salam" --to file://schema.hcl
```

## نکته
توجه شود همه چی رو پاک می کنه و نگاه به ترتیب نداره

اگر دیتا بیس سینک نباشه بازم ارور نمی ده 

به تعریف دیگه اینسپکت تنها ساختار رو کپی می کنه و بعد اپلای اون رو پیاده می کنه ، بدون توجه بهاین که سینک از قبل بودن یا نه

# migrate

این روش خیلی حرفه ای تر و دقیق تر است و مراحل زیر رو می توان پیاده کرد

Develop _ diff  ⇨ Check (CI) _ lint  ⇨ Push (CD) ⇨ Deploy _ apply

## diff

به چند روش می توان ساختار داده را تعریف کرد :
+ file (sql) فایل بدیم و بگیم از رو اون فایل بخون
+ socket database آدرس دیتابیس دولوپ رو بدیم
+ gorm - فکر کنم از استراکت gorm میخونه
### file

schema.sql
```
CREATE TABLE `users` (
  `id` bigint,
  `name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);

```
cli :
```
atlas migrate diff initial \                                                                         
  --to file://schema.sql \                                                                                   
  --dev-url "mysql://hp:1qaz@WSX@localhost:3306/dev" \                                                  
  --format '{{ sql . "  " }}'
```

### socket 
```
atlas migrate diff test1  --dir "file:///home/mostafa/test/migratetest/migrationsasd"  --to "mysql://root:1qaz@WSX@:3307/salam"  --dev-url "mysql://root:1qaz@WSX@localhost:3307/dev"   --format '{{ sql . "  " }}' 
```
## status

```
atlas migrate status -u "mysql://hp:1qaz@WSX@:3306/salam"

atlas migrate status --env local
```

## lint (optianal)

بعد از لاگین کردن میتوان مرحله ی چک یا CI را به صورت web interface  مشاهده کرد

در صورتی که این مرحله مشکلی نداشته باشد می توان اپلای کرد

بعد از لاگین باید دستور زیر رو زد و یه لینک جنریت میشه

```
atlas migrate lint  --dev-url "mysql://root:1qaz@WSX@localhost:3307/dev" -w
```


## apply

```

atlas migrate apply -u "mysql://hp:1qaz@WSX@localhost:3306/salam"

atlas migrate apply --env local


```

## roll back 

برای رول بک زدن باید ۲ مرحله انجام دهیم
+ تغییرات را از روی دیتابیس پاک کنیم

+ فایل ورژن شده را از migrations پاک کنیم


### down 
این مرحله باید با دقت شود زیرا روی دیتابیس تغییرات اعمال میشود ، برای اینکه فقط ببینیم چه تغییراتی صورت می پذیرد می توان با فلگ زیر تنها تغییرات رو خواند
```
--dry-run

atlas migrate down -u "mysql://hp:1qaz@WSX@localhost:3306/salam" --dev-url "mysql://root:1qaz@WSX@localhost:3307/dev" --dry-run
```

### rm

بعد از اعمال تغییرات روی دیتابیس می توان فایل migration هم پاک کرد با دستور زیر :

```
atlas migrate rm 20240413153126
```

### set 
می دانیم که یک جدول در دیتا بیس پروداکت داریم که اطلاعات اطلس را نگه می دارد

می توانیم با این دستور بروز کنیم آخرین آپدیت کدام ورژن باشد اما در دایرکتوری مایگریشن تغییری ایجاد نمی شود

هنوز نمی دونم به چه دردی می خوره

```
atlas migrate set 20240413120206 --url "mysql://hp:1qaz@WSX@localhost:3306/salam"
```


## نکته
به صورت پیش فرض هر فایل در یک ترنساکشن انجام می شود و بهتر است چند فایل را در صورت مایگریت کردن ، به صورت یک ترنساکشن ببینیم ،  برای این منظور از فلگ زیر استفاده می کنیم
```
--tx-mode all
```

## concepts
+ Dev Database
اطلس برای بعضی دستورات نیاز به یک محیط برای تست خودش دارد ، یک دیتا بیس برای تست و در نهایت خود اطلس تغییرات رو پاک کرده و  تمییز می کند آ ن را ، و به صورت پیش فرض می توان کانتینر با نسخه ای که در پروداکت استفاده می کنیم ، استفاده کرد - پیشنهاد اطلس در غیر این صورت یک دسترسی به هر دیتابیسی داد تا اطلس  بر روی آن تست مند

+ url
یه فرمت استاندارد هست و سوکت رو می گیره
```
mysql://user:pass@localhost:3306/schema
```

+ file
برای گرفتن فایل هست درون فایل سیستم
```
file://path/to/schema.sql
```
+ env
می تونیم توی یه فایل بریزیم ، فایل پیش فرض :
```
atlas.hcl
```
مقادیر به این صورت با فرمت hcl میریزیم
```
env "local" {
	url = "mysql://root:1qaz@WSX@localhost:3307/salam"
	dev = "mysql://root:1qaz@WSX@localhost:3307/dev"
}
```

+ login

برای بعضی قابلیت های web interface 
```
atlas login parnian
```