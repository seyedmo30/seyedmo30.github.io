### Lazy loading

توی جنگو این مفهوم هست و کاربردش اینه تا زمانی که به داده نیاز نیست ، درخواست سمت دیتابیس نمی ره  ، یکی از نتایج این مفهوم اینه زمانی که با **orm** می خواهیم **join** بزنیم ، میشه با این ۲ متد زیر تمامی دادهد هایی که در چند تیبل وجود دارد را یک جا فچ کرد

+ select_related
+ `prefetch_related`

اما در GORM راه حل استفاده از **Preload** هست

اگر از **Preload** استفاده نکنیم تنها id  های تیبل سفارش رو داریم

```go
type User struct {
  gorm.Model
  Orders []Order
}
type Order struct {
  gorm.Model
  UserID uint
}

var users []User

// Lazy: no orders fetched
db.Find(&users)


// Eager load orders via separate query
// توی این حالت ۲ بار کوییری زده میشه

db.Preload("Orders").Find(&users)


SELECT * FROM `users`;

SELECT * 
FROM `orders` 
WHERE `orders`.`user_id` IN (1,2,3, …);



var users []User
if err := db.Preload("Orders").Find(&users).Error; err != nil {
    log.Fatal(err)
}

// Now each users[i].Orders is filled:
for _, u := range users {
    fmt.Printf("User %d (%s) has %d orders:\n", u.ID, u.Name, len(u.Orders))
    for _, o := range u.Orders {
        fmt.Printf("  • Order %d: total=%.2f\n", o.ID, o.Total)
    }
}


// Or via JOIN
// زمانی که از جویین استفاده می کنیم تنها یک کوییری زده میشه

db.Joins("Orders").Find(&users)


SELECT
  `users`.`id`,
  `users`.`name`,
  `orders`.`id`   AS `orders__id`,
  `orders`.`user_id` AS `orders__user_id`,
  `orders`.`total` AS `orders__total`
FROM `users`
LEFT JOIN `orders` ON `orders`.`user_id` = `users`.`id`;

// اما بدیش اینه که برای سلکت جویین باید خودمون بشینیم  کلی استرینگ بنویسیم و کد بشدت کثیف میشه


// helper struct to capture flattened columns
type UserOrderRow struct {
    UserID    uint
    UserName  string
    OrderID   uint
    OrderTotal float64
}

var rows []UserOrderRow
err := db.
    Model(&User{}).
    Select("users.id   AS user_id, " +
           "users.name AS user_name, " +
           "orders.id  AS order_id, " +
           "orders.total AS order_total").
    Joins("LEFT JOIN orders ON orders.user_id = users.id").
    Scan(&rows).
    Error
if err != nil {
    log.Fatal(err)
}

// Now rows is a flat list; group them in Go if you want per-user slices
grouped := make(map[uint][]Order)
for _, r := range rows {
    o := Order{ID: r.OrderID, UserID: r.UserID, Total: r.OrderTotal}
    grouped[r.UserID] = append(grouped[r.UserID], o)
}

// iterate users and their orders
for userID, orders := range grouped {
    fmt.Printf("User %d has %d orders:\n", userID, len(orders))
    for _, o := range orders {
        fmt.Printf("  • Order %d: total=%.2f\n", o.ID, o.Total)
    }
}


```



**جمع بندی :**

برای تمییزی حاضرم ۲ کوییری بزنیم اما کد تمییز تر هندل شه و در نهایت از  **Preload** استفاده می کنم

