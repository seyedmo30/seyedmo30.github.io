یه ابزار خوب برای استوریج دیتا
### concepts
+ Bucket - شبیه با فولدر است و برای دسته بندی فایل ها  است
+ Object - هر فایل در این ابزار ، یک آبجکت است

### methods
+ GetObject - می توان بدون اینکه فایل در دایرکتوری یا فولدر تمپ ، ریخته شود ، به برنامه داده شود
+ FGetObject - آدرس دایرکتوری را میگیرد و در آن جا فایل را میریزد
+ FGetObject - داده ی باینری را گرفته و ذخیره می کند .
+ FPutObject - بجای گرفتن باینری فایل ، آدرس فایل را گرفته و ذخیره می کند
+ presigned_get_object - به سرور درخواست میزنیم که لینک با تاریخ انقضا و متا دیتا بسازد و یه لینک میگیریم
+ get_object_retention - برای ارتباط s3 هست 
  ### notes
  در پایتون :
  ```
  response = client.get_object('newsimage','3.jpg')
  print(response.data)
  ////////////////////
  client.put_object(bucket_name='newsimage',object_name=record[0]+".jpg",data=io.BytesIO(file_img.content),length=len(file_img.content), )

  ```
      
