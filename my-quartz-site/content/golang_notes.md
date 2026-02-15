







## private git repository  - 403 - 443  - forbidden

### common
اگر بخواهیم یه پکیبج درون گیت شرکت توسغه بدیم و تنها اونجا استفاده کنیم ، بادی مقدار زیر رو اکسپورت کنیم

یه نمونه آدرسی که درون گروه بک اند است
```
https://git.maani.app/maani/backend/kafka-wrapper
```
حالا می توان این رو به فایل /etc/environment اضافه کرد 
```
GOPRIVATE=*.example.app
```

همچنین اگر پروژه ذیل یه لایبری باشد مانند آدرس زیر ، باید درون gitconfig مقداری رو اضافه کنیم


nano ~/.gitconfig


```
[url "ssh://git@git.maani.app/"]
        insteadOf = https://git.maani.app/
```

### پیشرفته

یه سری وقتا یه پکیج نصب نمی شه ، شاید بشه با یه سری فلگ نصب کرد

`GOPROXY=https://goproxy.io,direct GOPRIVATE=git.srxx.org  GONOSUMDB=off  go mod tidy`

`GOPROXY=https://proxy.golang.org,direct go get github.com/gin-gonic/gin`

`GODEBUG=netdns=go GOHOST=localhost GOINSECURE=nullprogram.com GONOSUMDB=nullprogram.com GO111MODULE=on go get -v github.com/gin-gonic/gin`

اگر نشد این هم امتحان کنیم :

```
export GOPROXY=direct  
export GOSUMDB=off

go mod tidy
```

### جنگی - ssh

گاهی وقتا که جنگ می شه یا اینترنتا رو میبندن ، شاید این به کار بیاد ولی بعدش دوباره تغییرات رو پاک کنید

گاهی با ssh نمی شه کار کرد بجاش باید http go get - (git fetch) زد در این صورت باید مانند کانفیگ گیت ،  درون کانفیگ go get  هم تغییر ایجاد کرد ، 

باید دقیقا مثل کانفیگ گیت ، اکسس توکن توی آدرس بزاریم

```

➜   git:(master) pwd
/home/mo30/go/pkg/mod/cache/vcs/d126bcdab4b653fdb2a5bc2e92d0fab809494e1e63e6535f0187add1c34a399d




[core]
	repositoryformatversion = 0
	filemode = true
	bare = true
[remote "origin"]
	url = https://jang:glpat-xxxxxxxxxx@git.srxx.org/clients/exhub-client.git  // اینجا مهم است
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	gk-last-accessed = 2026-01-19T07:34:41.164Z

```

حالا همچنین نیاز داریم پکیج های داخلی رو موقع نصب با http  نصب کنیم به جای ssh  در این صورت ابتدا باید user , pass  گیت رو گلوبال بزارم تا موقغ خودکار نصب دیپندنسی ها توسط go mod tidt --- go get  بره از اونجا بگیره : مراحل زیر رو باید انجام بدیم :



```sh
cat > ~/.netrc <<EOF
machine git.srxx.org
login jang
password glpat-xxxxxx
EOF
chmod 600 ~/.netrc
```

و بعد از این

`git config --global url."https://git.srxx.org/".insteadOf "git@git.srxx.org:"`
 و 

`git config --global url."https://git.srxx.org/".insteadOf "ssh://git@git.srxx.org/"`




### no required module provides package  - to add it

وقتی توی یه workspace  یه پروژه میزاری ولی یوز نمیکنی ، این خطا رو میده

main.go:9:2: no required module provides package git.srxx.org/core/depowith/di; to add it:
        go get git.srxx.org/core/depowith/di

## clear cache internal gitlab repo

مشاهده

`grep -R "git.srxx.org" $(go env GOPATH)/pkg/mod/cache/vcs 2>/dev/null | head -n 20`

پاک کردن

`find "$(go env GOPATH)/pkg/mod/cache/vcs" -type f -exec grep -l "git.srxx.org" {} \; | xargs -r rm -v`

### unrecognized import path 404 Not Found

سر این من دهنم سرویس شد - وقتی یه ریپازیتوری داخلی رو go get  میزنیم میگه نیست - در این صورت احتمال داره گیت ssh  نداشته باشه یا خراب باشه در این صورت باید replace کنیم به صورت زیر :

دلیلش اینه که احتمالا گیت آدرسی با **.git** نداره



> module declares its path as: git.srxx.org/clients/exhub-client
> but was required as: git.srxx.org/clients/exhub-client.git


من سر این موضوع ۲ روز کامل درگیر بودم ، با اینکه ست کرده بودم به جای **http** از **ssh** استفاده کن اما بازم میگفت 404 Not Found http پس غیر مستقیم تلاش میکرد از http  بگیره اگر نمی تونست از ssh  میرفت می گرفت - هر کار کردم تجواب نداد - اول سعی کردم replace کنم و جواب داد

`replace git.srxx.org/clients/internalwallet-client => git.srxx.org/clients/internalwallet-client.git v0.0.3`

ولی نمی شد go.mod , و go.sum رو پوش کرد 

و در آخر تماامی اطلاعات http  رو پاک کردم جوری که دیگه نمیشد هیچ چیز رو آپدیت کرد یا گرفت ، در این صورت کامل از ssh  میگرفت


#### cobra vs multiple cmd folders

در صورتی که برنامه ی ما بیش از یک حالت ران شود یعنی همزمان هم rest api http  داره هم چند کرون جاب  ، می تونیم ۲ کار کنیم :

حالت اول این که چند cmd  و دایرکتوری داشته باشیم و درون هر کدوم ، main جدا ، خوبیش اینه خیلی مستقل هستن و نیاز به پکیج بیرونی نداره مانند کبرا همچنین به ازای هر کدوم ، یه داکر فایل باید داشته باشیم

حالت دوم استفاده از کبرا هست ، کلن یه بار بیلد میشه و یه فایل باینری داریم ولی در عوض می تونه با پارامتر هایی که میگیره ، 


#### unmarshal 

+ invalid character 'u003c' looking for beginning of value
+ invalid character '<' looking for beginning of value

خیلی وقتا می خوایم بادی یه ریسپانس رو آنمارشال کنیم ولی اینا رو میبینیم ، اینا به این معنیه که جواب جیسون نبوده بلکه تگ html tag  بوده ، احتمالا nginx , cloudeflare  بوده

+ در کل بهتر هر جا آنمارشال نشد ، علاوه بر متن خطای آنمارشال ، 200 کارکتر اول رو هم بدیم بالا


+ سعی کنیم همیشه در صورتی که پروایدر چندین حالت استایل پاسخ می ده ، اون رو مانند سوییچ کیس هندل کنیم و پاسخ یکی باشه مثلا در صورتی که جواب 200 و لیست خالی هست ، استایلش متفاوت میشه

+ سعی کنیم در صورتی که هیچ جوره نتونستیم آنمارشال کنیم ، اون رو لاگ کنیم ، مثلا خطای آنمارشال و ۱۰۰ کارکتر اول اون بادی یا اسلایس بایت ها

#### time.Time
این تایپ برای زمانه و اگر استرینگ با فرمت زیر بدیم ، می توینم unmarshal کنیم
```
type Event struct {
    Name      string
    Timestamp time.Time
}
...
    jsonStr := `{"Name":"Birthday","Timestamp":"2024-04-27T12:00:00Z"}`
    var event Event
    err := json.Unmarshal([]byte(jsonStr), &event)
```

#### json unmarshal

وجه شود در صورتی که فیلد های استراکت پرایویت باشد ، اگر اینستنسی از آن مارشال شود ، فیلد های پرایویت برگردانده نمی شود ، پس با دقت فیلد ها را پرایوت کن . ( کلا پرایوت جالب نیست )

redis - json ------- اینجوری می تونیم دیکد کنیم :



	result := new(interface{})
	json.Unmarshal([]byte(cacheResponse.(string)), result)
 

#### json.Decoder

+ یکی از ابزار های خوب برای کار با جیسون آنمارشال است اما گاهی جیسون ما خیلی بزگ است و یا گاهی می خواهیم بخشی از جیسون رو استفاده کنیم در این صورت نیازی نیست تمامی جیسون را در مموری نگه داریم و راه حل  استفاده از json.Decoder




### golang conventional naming package

متاسفانه مرسومه که نام پکیج ها `lowercase` باشه و از `-` or `_` استفاده نکنیم

و این شد که من باید اسم پکیجم رو این بزارم :

`internalrequestservice`


### go work(workspace)

گاهی ۲ تا پروژّ داریم که تو ۲ تا ریپازیتوری جدا هستن ولی خیلی کاپل هم هستن ، مثلا سرویس و SDK اون یا همون کلاینش ، در این صورت حالا می تونیم اینا رو توی یه ورک اسپیس بزاریم 

مزایا :

 اینا رو نیازی نیست **replace** و یا پابلیش کنیم ، سپس استفاده کنیم ، همچنین به محض تغییر استفاده میشه و حتی اگه نیاز مندی نیاز به آپدیت داره ، توی محیط توسعه نیازی به پابلیش کردن نیست همچنین دستورات  go build, go test, go run رو بزنیم ، همه ی ورک اسپیس رو شامل میشه

 یادمه kafkka-wrapper  همیشه نیاز بود ابتدا ریلیز بدیم سپس تست واقعی بگیریم ، اما این کمک می کنه راحت تر وابستگی ها رو توسعه بدیم
 