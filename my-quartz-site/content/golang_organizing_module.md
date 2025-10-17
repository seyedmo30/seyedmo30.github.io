## Go API Design

### Accept Interfaces, Return Structs
بهتره به جای گرفتن یک استراکت یا یک فانکشن ، از یه اینترفیس استفاده کنیم و متد هایی که می خواییم رو اینجوری بگیریم ، انعطاف پذیری خیلی بیشتر میشه

```

// Define an interface
type Reader interface {
    Read(p []byte) (n int, err error)
}

// A function that accepts an interface and returns a struct
func NewContent(reader Reader) (*ContentResult, error) {

```


### Parameter and Result Objects

به جای گرفتن چند ورودی ، یک استراکت تعریف کنیم و آن را بگیریم


### dto embeding

هر لایه باید  dto  خودش رو داشته باشه ، به هیچ  وجه نباید dto  شیر بشه یا dto  generic  داشته باشیم و برای جلو گیری از پیچیدگی یا readundancy  میتونیم از موارد زیر استفاده کنیم

+ mapper package : go-napper , mapstructure

+ embeding : Base dto

```go
type BaseDigitalSignature struct {
    ID          string
    CreatedAt   time.Time
    UpdatedAt   time.Time
}
```


## Designing Go Packages

### Keeping the File Structure Flat
برای دسترسی آسان به متد ها و ساختار ها ، بهتره تمامی بخش های برنامه که باید در دسترس کلاینت باشد ، در روت پروژه باشد

#### Organizing Supplementary Features
بهتره بخش های فرعی  رو در فولدر ها گذاشت

### Directory Structure

مطالعه شود لینک زیر ، راه کار های جدید  گوانگ برای ساختار دایرکتوری و اورگنایز پروژه

    https://go.dev/doc/modules/layout

#### Discoverability and Clarity in Naming

مهم ترین چیز نام گذاری متد ها و ساختار برنامه است

#### Robustness and Potential for Misuse

برنامه جوری نباید باشه که به ارور بخورده 

#### Flexibility and Future-Proofing

باید در نظر داشته باشیم برنامه در آینده گسترش پیدا می کنه و توانایی ورژن زدن و یا گسترش داشته باشه

