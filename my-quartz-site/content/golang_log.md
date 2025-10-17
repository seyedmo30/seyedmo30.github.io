لاگ باید جوری باشه که وقتی پروژه رفت بالا یا روی سیستم کسی ، به هیچ وجه نیاز به تفسیر و دیباگ نباشه

حتی صرفا نباید خطا رخ بده تا لاگ رو بخونیم

به گونه ای که طرف اگه لاگ رو خوند ، باید کامل بدونیم جنسش چی بوده ، کجا رخ داده ،  داده هایی که طرف داده و ترکیده چی بوده و ....

کدام api  کال شده 

https://www.kelche.co/blog/go/zap/
      
      
      package logger
      
      import (
      	"go.uber.org/zap/zapcore"
      	"os"
      	"sync"
      	"time"
      
      	"go.uber.org/zap"
      )
      
      var once sync.Once
      
      var instance *zap.Logger
      
      func Connect() *zap.Logger {
      
      	once.Do(func() {
      
      		config := zap.NewProductionEncoderConfig()
      		config.EncodeLevel = zapcore.CapitalColorLevelEncoder
      		config.EncodeTime = zapcore.TimeEncoderOfLayout(time.DateTime)
      		consoleEncoder := zapcore.NewConsoleEncoder(config)
      		defaultLogLevel := zapcore.InfoLevel
      
      		deepLevel := zapcore.FatalLevel
      		core := zapcore.NewTee(
      			zapcore.NewCore(consoleEncoder, zapcore.AddSync(os.Stdout), defaultLogLevel),
      		)
      
      		instance = zap.New(core, zap.AddCaller(), zap.AddStacktrace(deepLevel))
      
      		//logger, _ := zap.NewProduction()
      		//defer logger.Sync() // flushes buffer, if any
      		//logger.Sugar()
      		//instance1 := logger.Sugar()
      		//instance1.Info()
      		//var filename string
      		//err := os.MkdirAll("/var/logger/exploit/", 0555)
      		//if err != nil {
      		//	fmt.Println("MkdirAll : ", err)
      		//	filename = "./logs.logger"
      		//
      		//} else {
      		//	filename = "/var/logger/exploit/logs.logger"
      		//}
      		//filename = "/var/logger/exploit/logs.logger"
      		//
      		//config := zap.NewProductionEncoderConfig()
      		//config.EncodeTime = zapcore.ISO8601TimeEncoder
      		//fileEncoder := zapcore.NewJSONEncoder(config)
      		//logFile, err := os.OpenFile(filename, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0777)
      		//if err != nil {
      		//	fmt.Println("os.OpenFile var/logger/exploit/logs.logger ", err)
      		//	filename = "./logs.logger"
      		//
      		//	logFile, err = os.OpenFile(filename, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0777)
      		//	if err != nil {
      		//		fmt.Println("os.OpenFile logs.logger ", err)
      		//
      		//		os.Exit(1)
      		//	}
      		//}
      		//writer := zapcore.AddSync(logFile)
      		//defaultLogLevel := zapcore.DebugLevel
      		//core := zapcore.NewTee(
      		//	zapcore.NewCore(fileEncoder, writer, defaultLogLevel),
      		//)
      		//instance = zap.New(core, zap.AddCaller(), zap.AddStacktrace(zapcore.ErrorLevel))
      
      	})
      
      	return instance
      }

### tips
توجه شود ارور زیر مجموعه لاگ است و می توان لاگ لول اینفو باشد ، می توان ارور باشد

بهتر است در محیط توسعه لاگ بین لایه ها هم باشد اما در محیط پروداکت ، لاگ در ابتدا و انتهای سرویس باشد مثلن در مانی در لایه ی کافکا لاگ زده میشه ، همون ابتدا می گیم در خواست اومده و اطلاعات در خواست  ، و در نهایتهنگام پاسخ ، لاگ زده میشه و اگر ارور بود ، باید لاگ از لایه های زیری wrap شده باشه و لاگ شود
یکی از ساختار های سادهلاگ :

```
{
  "timestamp": "2024-07-11T15:30:00Z",
  "level": "INFO",
  "service": "example-service",
  "message": "User successfully logged in",
  "logger": "com.example.service.UserService",
  "thread": "main",
  "context": {
    "user_id": "123456",
    "request_id": "abc-123-def-456",
    "ip_address": "192.168.1.100",
    "request_link": "https://your-service.com/requests/abc-123-def-456",
    "request_payload": {
      "username": "example_user",
      "password": "*****"
      // Add other relevant request payload fields as needed
    }
  }
}
```

 و در نهایت میشه خروجی برنامه رو با logstash توی الستیک دامپ کرد

 
نکته : اگر در اجرای داکری سرویس از دستور زیر استفاده کنیم ، تمام لاگ ها مستقیم درون فایل لاگ ریخته میشود  و هیچ لاگی در صفحه سیستم عامل پرینت نمی شود ، و در نهایت نتیجه می گیریم لاگ های داکری همیشه خالی است .

CMD /app/postgres_tailer >> /app/logs/log_tailer.log 2>>/app/logs/error.log

با توجه به فرض بالا ، لاگ های داکر همیشه خالی است : 

docker logs -f id --------> empty

اگر هم سرویس را بدون ریختن لاگ در جایی طراحی کنیم ، لاگ ها درون docker logs ریخته میشود ، اما اگر در خارج از محیط داکری اجرا کنیم . لاگ ها را از دست می دهیم :) مطالعه شود بست پرکتیس سنتری !
