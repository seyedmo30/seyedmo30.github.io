یکی از راه های مستند سازی ، ساخت میک فایل است

به جای اینکه کاربر ، بیاد از هیستوری نگاه کنه ما چی کارا کردیم ، می تونه بیاد این فایل رو بخونه


```


# Go parameters
GOCMD=go
GOBUILD=$(GOCMD) build
GOCLEAN=$(GOCMD) clean
GOTEST=$(GOCMD) test
GOGET=$(GOCMD) get

# Name of the binary
BINARY_NAME=myapp

# Entry point/main package
MAIN_PACKAGE=./cmd/main.go

# Default target
all: test build

build:
	$(GOBUILD) -o $(BINARY_NAME) $(MAIN_PACKAGE)  -a -v -tags musl

test:
	$(GOTEST) ./...

clean:
	$(GOCLEAN)
	rm -f $(BINARY_NAME)

```

+ tags ـــ به صورت پیش فرض از glibc استفاده می کند اما musl سبک تر و گاهی مناسب تر است اما باید در apline  آن را نصب کنیم

