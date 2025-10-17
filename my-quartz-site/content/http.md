## Content-Type
یکی از مهمترین هدر ها ، مشخص می کند فرمت بادی چیست همچنین دیکد را هم می توانیم آپشنال بیفزاییم . با توجه به اینکه HTTP ی ما  request یا response است ، موارد استفاده متفاوت است ( تمام موارد هم برای درخواست هم برای پاسخ می توان استفاده کرد ) .
### request
application/json - application/x-www-form-urlencoded - multipart/form-data - text/plain - 

+ application/x-www-form-urlencoded : مانند کوییری پارام در متد گت می ماند ، بعد از افزودن کلید و مقدار ، آنرا در بادی شبیه به کوییری کرده
+ multipart/form-data : با یک ریکوست هم می توان فایل باینری فرستاد ، هم تکست بادی

### response
application/json - text/html - text/plain - image/jpeg, image/png - audio/mpeg - video/mp4 - multipart/mixed - application/octet-stream

در صورتی که ریکویست حاوی چند فرم دیتا و چند فایل باینریی باید ، می بایست از multipart/form-data استفاده شود ، در این صورت فرم دیتا ها را تنها می توان مانند x-www-form-urlencoded در url  ارسال کرد و نمی توان json فرستاد زیرا در  body باید فایل های باینری قرار داده شود.


### http client

در پایتون یا گو ، وقتی ریکویستی می سازیم ، می توانیم اطلاعات علاوه بر متد یا url یا header به درخواست بدهیم . 
+ httptrace
 ـ مثلا می توانیم درخواست را با DNS ست کنیم

        req, _ := http.NewRequest("GET", "http://example.com", nil)
        trace := &httptrace.ClientTrace{
        DNSDone: func(dnsInfo httptrace.DNSDoneInfo) {
            fmt.Printf("DNS Info: %+v\n", dnsInfo)
        },
        req = req.WithContext(httptrace.WithClientTrace(req.Context(), trace))
