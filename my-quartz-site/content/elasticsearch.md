تعداد داکیومنت های ایندکس

http://localhost:9200/exploit/_count

ساخت ایندکس جدید

PUT  http://192.168.13.248:9200/exploit

حذف ایندکس

DELETE  http://192.168.13.248:9200/exploit

گرفتن داکیومنت

http://192.168.13.248:9200/exploit/_doc/5

گرفتن داکیومنت بدون متا دیتا

http://192.168.13.248:9200/exploit/_source/5

گرفتن  مالتی داکیومنت ( مطالعه شود mget )

http://192.168.13.248:9200/exploit/_mget

لیست داکیومنت

http://192.168.13.196:9200/exploit/_search/?size=1000&pretty=true

لیست ایندکس ها

http://localhost:9200/_cat/indices?v=true

### idioms 
term - اگر مقادیر فیلد ها در الستیک را بهقسمت های کوچک بشکنیم و برای هر کدام ایندکس بگذاریم ، به هر یونیت یا قسمت از مقدار یک ترم گفته می شود ، مثلا یک فیلد با تایپ text به چندین ترم کوچک می شکند و هر کدام ایندکس گذاری می شود و یک فیلد با تایپ keyword ، کلش یک term است

### mapping

توجه خیلی مهم : 

در صورتی که بخواهیم مپینگ رو ثابت بزاریم ، یعنی داینامیک  به صورت دیفالت نباشه ، خیلی از ساختار هایی که ابتدا تو مپ تعریف می کنیم ، قابل تغییر نیست و هر بار نیازه reindex  کنیم

بهتره قبل از مپ دستی ، تمامی فیلد ها  و ساختار ها رو بررسی کنیم چون هر بار reindex کلی دردسر داره


در صورتی که مپ رو به ایندکس ندهیم ، جنریک ترین تایپ دیفالت رو خودش متناسب به داده در نظر می گیره و بدیش اینه دیگه امکان تغییر در مپ وجود نداره ( می تونیم اضافه کنیم ولی نمی توانیم مپ موجود رو تغییر بدیم ) همچنین در صورتی که بخواهیم مپ رو تغییر بدیم تنها روش reindex کردن هست .

+ روش اول اینه موقع ساخت ایندکس ، همون جا مپ رو بدیم

                  PUT /my-index-000001
                {
                  "mappings": {
                    "properties": {
                      "age":    { "type": "integer" },  
                      "email":  { "type": "keyword"  }, 
                      "name":   { "type": "text"  }     
                    }
                  }
                }
  
 روش دوم اینه بعد ساخت ایندکس ، مپ کنیم

          PUT /my-index-000001/_mapping
        {
          "properties": {
            "employee-id": {
              "type": "keyword",
              "index": false
            }
          }
        }

بهتر است مپ کردن با توجه به ساختار داده رو غیرفعال کنیم , ۲ روش می توان تنظیم کرد که اگر داده شامل فیلدی بود که خارج از ساختار است ، آن را نادیده بگیرد(false) و حالت دوم اینکه تمامی داده را با خطا برگرداند و چیزی ذخیره نکند (strict) :

    {
      "mappings": {
        "dynamic": "strict",  // در این حالت اگر ورودی تایپی متفاوت داشته باشد خطا می خورد و سطر ذخیره نمی شود
        "properties": {
          // Define your properties explicitly here
        }
      }
    }

+ توجه شود در صورتی که داینامیک فالس باشد ، فیلد های خارج از مپ در جیسون _source  ذخیره میشوند ولی ایندکس نمیشوند
و به این  روش می توانیم مشخص کنیم در سورس چه فیلد هایی بشیند :

```
PUT /your_index
{
  "mappings": {
    "dynamic": "false",
    "_source": {
      "includes": ["field1", "field2", "field3"] // اینجا میگیم تنها این ۳ فیلد اجازه دارن
    },
    "properties": {
      "field1": { "type": "text" },
      "field2": { "type": "keyword" },
      "field3": { "type": "date", "format": "yyyy-MM-dd" }
    }
  }
}


```
### text vs keyword
اگر بخواهیم یک فیلد استرینگ را آنالیز کرده و فول تکست سرچ کنیم باید فیلد را text بزاریم ، برای متن ها و دیسکریپشن و ...

اما اگر استرینگ یک کلید هست و نیاز به aggs یا exact matching هست برای فیلتر یا مرتب کردن ، باید از keyword استفاده کنیم . توجه کنیم کل keyword یک term  هست و آنالیز یا شکسته نمی شود مانند ID , tag , lable , username 

و به صورت دیفالت هم می توان یک فیلد ، هر دو باشد به صورت زیر

اگر یک فیلد تکست را ‌خیره کنیم ، احتمالا الستیک به شکل زیر ، آن را ذخیره کند (dynamic mapping ):
    
    "description": {
        "type": "text",
        "fields": {
            "keyword": {
                "type": "keyword",
                "ignore_above": 256
            }
        }
    }

و در نهایت اگر می خواهیم یک فیلد تنها ذخیره شود ، بدون آنالیز و ایندکس باید index: false و تایپ  type : keyword باشد

      "field_without_index": {
        "type": "keyword",
        "index": false
      }
      
### bulk create
    
    POST /_bulk
    { "create": {"_id": "7", "_index": "index1"} }
    { "my_field": "foo" }
    { "create": {"_index": "index1"} }
    { "my_field": "foo2" } 
    
### bulk update
    POST _bulk
    { "update" : {"_id" : "1", "_index" : "index1", "retry_on_conflict" : 3} }
    { "doc" : {"field" : "value"} }
    { "update" : { "_id" : "0", "_index" : "index1", "retry_on_conflict" : 3} }
    { "script" : { "source": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}

### update without id
در این صورت چون ایدی برای آپدیت مهم نیست باید کوییری بزنیم و سپس فیلد مورد نظر را آپدیت کنیم

    POST feeds/_update_by_query
    {"script": {
          "source":"ctx._source.title_fa = 'salam1243';ctx._source.sentiment.negative = '0.47';ctx._source.summary_fa ='salam1234';",
          "lang": "painless"
        },
        "query": {
          "term": {
                "common_key.keyword": "b9e0c6de-9613-404d-9377-bdc43175d3b1"
          }
        }
        }



    
### mget - multi get
یکی از روش ها برای گرفتن داکیومنت ها با استفاده از ID ، این روش زمانی استفاده می شود که ما شناسه داکیومنت ها را داشته باشیم .

اگر بخوایم مشخص کنیم چه فیلد هایی می خواییم و چه فیلد هایی نمی خواییم :

GET /_mget

        {
          "docs": [
            {
              "_index": "my-index-000001",               مشخص می کنیم بر روی کدام ایندکس
              "_id": "1",                                 مشخص می کنیم بر روی کدام ایدی
              "_source": [ "field3", "field4" ],           مشخص می کنیم  کدام فیلد ها رو می خوایم
            },
            {
              "_index": "my-index-000001",
              "_id": "2",
                        "_source": {
                        "include": [ "user" ],                       مشخص می کنیم  کدام فیلد ها رو می خوایم
                        "exclude": [ "user.location" ]                مشخص می کنیم  کدام فیلد ها رو نمی خوایم
                        "stored_fields": [ "field1", "field2" ]         ترتیب
                        "routing": "key2"                               انتخاب شارد
                      }
            }
          ]
        }



# query

خیلی قوی تر از مالتی گت هست و داخل سرچ استفاده میشود

شامل دو دسته میشود : لیف کويری ها که ساده هستند و بدون بول یا نستد استفاده می شوند - کامپوند کوییری ها که ترکیبی اند و پر کاربرد ترینش بول هست 

درون api search قرار دارد و بعد از کوییری ، می بایست مشخص کینم

___ leaf query





match -------
  در مچ باید کلمات دقیق باشند .

prefix -------
اگر ابتدای کلمه را می دانیم باید از prefix استفاده کنیم .

match_phrase -----
ترتیب عبارت هم مهم است .

fuzzy -----
غلط های املایی رو درست می کنه .

term ------ 
زمانی که می خوهیم سرچ دقیق بر روی عدد تاریخ یا استیرینگ بزنیم ، توجه شود روی تکست کار نمی کند .

range -------- 
ترم لول است و بازه ای از نوع مشخص یگیرد






        GET /_search
        {
          "_source" : ["title" , "age " ] ,         مشخص می کنه چه فیلد هایی و ترتیب آنها
          "query": {                                باید درون کوییری مشخص کنیم که match , term , range ,....
            "match": {                                   در متچ ، باید کله دقیق برابر مقدار باشد تا پیدا شود ، مقدار اول کلید و مقدار دوم متن مورد نظر هست
              "field1": {                                                        فیلدی که در مچ باید جستوجو درون آن انجام شود
                "query":
                      {
                        "query": "Programs Rating",                متن جستوجو
                        "operator": "and" ,                        باید تمام متن در خروجی باشد
                        "fuzziness": "AUTO",                        اجازه می دهد که به صورت فازی سرچ کنیم
                      }

              }
            }
          }
        }


___ compound query

ابتدا یک آبجکت بول میسازیم و ۴ روش برای ایجاد کامپوند کویری داریم : درون موارد زیر می توانیم شرایط لیف کویری رو بنویسیم .توجه شود در هر کدام از موارد زیر میتوان چند شرط لیف کوییری گذاشت .

must -------
که یعنی باید درون کویری شرط های آن باشد

should ----- 
اگر شرط های این باشه امتیاز آن بالا تر می ره . به عبارت دیگر اگر ریزالت ما ۴۰ تا باشه ، شولد هیچ تاثیری توی تعداد نتایج نداره ، اما تو نمره و ترتیب آنها نقش داره .

filter ----- 
در نهایت می توان با فیلتر ، خروجی را فیلتر کرد . همچنین می توان گفت ترم لول ها درون فیلتر قرار می گیرند ، مثل term , range

must_not ---- 
نباید این شرط ها درون نتیجه باشد .


        POST exploit/_search
        {
          "size": 10,
          "query": {
            "bool": {
              "must": [
                {
                  "match": {
                    "type": {
                      "query": "Linux Kernel",
                      "operator": "and"
                    }
                  }
                },
                {
                  "match": {
                    "platform": {
                      "query": "Linux mac",
                      "operator": "or"
                    }
                  }
                }
              ],
              "filter": [
                {
                  "range": {
                    "date": { "gte" : "2007-12-29T00:00:00Z", "lte" : "2023-12-29T00:00:00Z" } 
                  }
                }
              ]
            }
          }
        }





اگر بخواهیم درون یک کوییری چند شرط بزاریم ، باید از bool استفاده کنیم ، خود بول باید یکی از موارد must و filter و should و must_not را بگیرد .

اینسرت تکی

curl -POST 'localhost:9200/exploit/_doc' -H 'Content-Type: application/json' -d'
{
        "timestamp": "2018-01-24 12:34:56",
        "message": "User logged in",
        "user_id": 4,
        "admin": false
} '

curl -X PUT "localhost:9200/exploit/_bulk?pretty" -H 'Content-Type: application/json' -d'
{ "create": { } }
{ "@timestamp": "2099-05-07T16:24:32.000Z", "event": { "original": "192.0.2.242 - - [07/May/2020:16:24:32 -0500] \"GET /images/hm_nbg.jpg HTTP/1.0\" 304 0" } }
{ "create": { } }
{ "@timestamp": "2099-05-08T16:25:42.000Z", "event": { "original": "192.0.2.255 - - [08/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638" } }
'

مرتب بر اساس آیدی

http://192.168.13.248:9200/exploit/_search/?size=1000&pretty=true&sort=_id
