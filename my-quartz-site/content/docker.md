




# Docker Cheat Sheet

## دانلود و اجرای ایمیج

```bash
sudo docker run یا pull نام ایمیج
docker run -d nginx
```

## مدیریت ایمیج‌ها و کانتینرها

### لیست ایمیج‌ها

```bash
sudo docker images
```

### لیست کانتینرهای در حال اجرا

```bash
sudo docker ps
```

### لیست تمام کانتینرها (حتی استاپ شده)

```bash
sudo docker ps -a
```

### پاک کردن کانتینرها

```bash
sudo docker rm id
docker rm $(docker ps -a -q)
```

### ایستاپ و استارت کردن کانتینرها

```bash
docker stop id
docker start id
```

### مشاهده لاگ‌های کانتینر

```bash
sudo docker logs -f id_container
```

### اتصال به یک کانتینر در حال اجرا

```bash
sudo docker attach id
```

## پاکسازی و مدیریت حافظه

### حذف ایمیج‌های دنگلینگ

```bash
docker image prune
```

### حذف کانتینرهای اگزیت شده

```bash
docker container prune
```

### حذف والیوم‌های بدون استفاده

```bash
sudo docker volume prune
```

### مشاهده وضعیت دیسک

```bash
sudo docker system df
```

## مدیریت والیوم‌ها

### تعریف والیوم در `docker-compose.yml`

```yaml
volumes:
  - postgres-db-airflow:/var/lib/postgresql/data
  - ./init-database.sh:/docker-entrypoint-initdb.d/init-database.sh
  - ./init-sql.sql:/init-sql.sql
```

### نکته مهم در والیوم‌ها

+ در صورتی که داکر به‌صورت عادی ران شود، دایرکتوری درون کانتینر پاک شده و دایرکتوری تعریف شده سیستم جای آن را می‌گیرد.

+  volume mount

یه name میدیم و داکر خودش هندل میکنه


+ bind mount

آدرس دستی میدیم


## اجرای دستورات درون کانتینر

```bash
sudo docker exec -it id bash
```

## Troubleshooting

### دیباگ کردن کانتینری که اجرا نمی‌شود

```bash
docker run -ti image_id sh
```

### مشاهده لاگ‌های یک کانتینر کرش‌شده

```bash
sudo docker logs -f id_container
```

### تغییر `entrypoint` هنگام اجرای کانتینر

```bash
docker run -it --entrypoint /bin/sh echo-app
```

## ورود به داکر هاب

```bash
docker login -u username -p password
```

## بیلد کردن و ذخیره ایمیج‌ها

### بیلد کردن ایمیج از `Dockerfile`

```bash
docker build -t name:version directory
docker build -t food:1.0 ~/test/
```

### ذخیره و لود کردن ایمیج‌ها

```bash
sudo docker save -o ~/ngin nginx:1.0.0
sudo docker load < ngin
```

## استفاده از PostgreSQL در داکر

### اجرای PostgreSQL

```bash
docker run -itd -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=salam -p 5432:5432 -v /data:/var/lib/postgresql/data --name postgresql postgres:15-alpine
```

### ورود به دیتابیس PostgreSQL

```bash
PGPASSWORD=salam psql -U postgres
```

### اجرای اسکریپت SQL در دیتابیس

```bash
psql -U postgres -d news_fetcher -h localhost -p 5432 -a -f init-sql.sql
```

## Dockerfile نمونه

```dockerfile
FROM alpine:3.17
WORKDIR /app
COPY . /app
CMD /app/salam
```

## مدیریت شبکه‌ها در داکر

### ساخت شبکه

```bash
sudo docker network create mynetwork
```

### مشاهده شبکه‌ها

```bash
docker network ls
```

### مشاهده اطلاعات شبکه

```bash
docker network inspect <network-name>
```

### پینگ گرفتن از کانتینرها درون شبکه

```bash
nslookup <container-name>
ping <container-name>
```

more

