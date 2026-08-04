# Dependency Injection (DI) ဆိုတာဘာလဲ?

**Dependency Injection (DI)** ဆိုတာ Class တစ်ခုက အလုပ်လုပ်ဖို့လိုအပ်တဲ့ Object ကို ကိုယ်တိုင် `new` နဲ့ မဆောက်ဘဲ အပြင်ကနေ ထည့်ပေးတဲ့ နည်းလမ်းဖြစ်ပါတယ်။

ရိုးရိုးပြောရရင်

```csharp
var emailService = new EmailService();
```

လို့ Class ထဲမှာ ကိုယ်တိုင်ဆောက်မယ့်အစား

```csharp
public UsersController(EmailService emailService)
{
    _emailService = emailService;
}
```

လို့ လိုအပ်တဲ့ Object ကို Constructor ကနေ တောင်းလိုက်တာဖြစ်ပါတယ်။

ဘယ် Object ကို ဘယ်လိုဆောက်ပေးမလဲဆိုတာ ASP.NET Core ရဲ့ Built-in **DI Container** က တာဝန်ယူပါတယ်။

DI က Class နဲ့ သူ့ Dependency ကို ဖန်တီးတဲ့တာဝန် ပြောင်းပြန်လွှဲပေးထားတာကြောင့် **Inversion of Control (IoC)** ကို အကောင်အထည်ဖော်တဲ့ နည်းလမ်းတစ်ခုလည်း ဖြစ်ပါတယ်။

---

# Dependency ဆိုတာဘာလဲ?

Dependency ဆိုတာ Class တစ်ခု အလုပ်လုပ်ဖို့ လိုအပ်တဲ့ အခြား Object တစ်ခု ဖြစ်ပါတယ်။

```text
UsersController
        │
        ▼
EmailService
```

ဒီမှာ `UsersController` က Email ပို့ဖို့ `EmailService` ကို လိုအပ်ပါတယ်။

ဒါကြောင့် `EmailService` ဟာ `UsersController` ရဲ့ Dependency ဖြစ်ပါတယ်။

Dependency က Service တစ်ခုတည်း မဟုတ်ပါဘူး။

* Repository
* `DbContext`
* `ILogger`
* `IHttpClientFactory`
* Configuration

စတာတွေလည်း Dependency ဖြစ်နိုင်ပါတယ်။

---

# Without Dependency Injection

User Register လုပ်တဲ့ API တစ်ခု ရေးမယ်ဆိုပါစို့။

```csharp
public sealed class UsersController
{
    private readonly EmailService _emailService;

    public UsersController()
    {
        _emailService = new EmailService();
    }

    public void Register()
    {
        _emailService.Send("Welcome");
    }
}
```

ဒီ Code မှာ Controller က

```csharp
new EmailService();
```

ဆိုပြီး Dependency ကို ကိုယ်တိုင်ဆောက်နေပါတယ်။

ဒါကြောင့် Controller က `EmailService` ကို ဘယ်လိုဆောက်ရမလဲ သိနေရပါတယ်။

နောက်ပိုင်း `EmailService` မှာ `ILogger` နဲ့ Configuration လိုလာရင် Controller ကပါ ထပ်ဆောက်ပေးရပါမယ်။

```text
UsersController
    │
    ├── new EmailService(...)
    │       ├── new Logger(...)
    │       └── new Configuration(...)
    │
    └── Business Logic
```

Class တစ်ခုက Dependency ဖန်တီးတဲ့အသေးစိတ်အထိ သိနေရတာကို **Tight Coupling** လို့ ခေါ်ပါတယ်။

Project ကြီးလာတာနဲ့ Object ဆောက်တဲ့ Code တွေ နေရာအနှံ့ ပျံ့သွားပြီး ပြင်ဆင်ရခက်လာပါတယ်။

---

# With Dependency Injection

DI သုံးရင် `UsersController` က လိုအပ်တာကို Constructor ကနေ တောင်းရုံပါပဲ။

```csharp
public sealed class UsersController
{
    private readonly EmailService _emailService;

    public UsersController(EmailService emailService)
    {
        _emailService = emailService;
    }

    public void Register()
    {
        _emailService.Send("Welcome");
    }
}
```

`Program.cs` မှာ `EmailService` ကို Register လုပ်ပေးရပါတယ်။

```csharp
builder.Services.AddScoped<EmailService>();
```

ASP.NET Core က `UsersController` ဆောက်တဲ့အခါ Constructor ကိုကြည့်ပါတယ်။

`EmailService` လိုအပ်နေတာကိုတွေ့ရင် DI Container ထဲက Registration ကိုရှာပြီး Object ဆောက်ကာ Inject လုပ်ပေးပါတယ်။

```text
UsersController က EmailService တောင်းတယ်
                 │
                 ▼
          DI Container ရှာတယ်
                 │
                 ▼
          EmailService ဆောက်တယ်
                 │
                 ▼
      UsersController ထဲ Inject လုပ်တယ်
```

ဒီနည်းကို **Constructor Injection** လို့ ခေါ်ပါတယ်။ ASP.NET Core မှာ အသုံးအများဆုံးနဲ့ အကြံပြုထားတဲ့နည်း ဖြစ်ပါတယ်။

---

# DI Container က ဘာလုပ်ပေးလဲ?

ASP.NET Core ရဲ့ DI Container မှာ အဓိကအလုပ် နှစ်ခုရှိပါတယ်။

## 1. Registration

ဘယ် Service ကို ဘယ် Lifetime နဲ့ ဆောက်ရမလဲဆိုတာ `IServiceCollection` ထဲ Register လုပ်ပါတယ်။

```csharp
builder.Services.AddScoped<ProductService>();
```

## 2. Resolution

Class တစ်ခုက Dependency တောင်းလာတဲ့အခါ Container က Registration ကိုကြည့်ပြီး Object Graph တစ်ခုလုံး ဆောက်ပေးပါတယ်။

```text
ProductsController
        │ needs
        ▼
ProductService
        │ needs
        ▼
ProductRepository
        │ needs
        ▼
AppDbContext
```

Container က `ProductsController` တစ်ခုတည်း ဆောက်တာမဟုတ်ဘဲ သူလိုအပ်တဲ့ `ProductService`, `ProductRepository`, `AppDbContext` တွေကိုပါ အဆင့်ဆင့် ဖြည့်ပေးပါတယ်။

ဒါကို **Dependency Graph** သို့မဟုတ် **Object Graph** လို့ ခေါ်ပါတယ်။

---

# ASP.NET Core Web API Example

Product API တစ်ခု ရေးမယ်ဆိုပါစို့။

```text
Client
  │
  ▼
ProductsController
  │
  ▼
ProductService
```

Interface မလိုသေးတဲ့ ရိုးရှင်းတဲ့ Example နဲ့ စကြည့်ရအောင်။

## Step 1 - Model

```csharp
public sealed record ProductDto(int Id, string Name);
```

## Step 2 - Service

```csharp
public sealed class ProductService
{
    public IReadOnlyList<ProductDto> GetAll()
    {
        return
        [
            new ProductDto(1, "Keyboard"),
            new ProductDto(2, "Mouse")
        ];
    }
}
```

## Step 3 - Program.cs မှာ Register လုပ်မယ်

```csharp
builder.Services.AddScoped<ProductService>();
```

ဒီ Code ရဲ့ အဓိပ္ပာယ်က

```text
ProductService တောင်းလာရင်

Scoped Lifetime နဲ့ ဆောက်ပြီး

Inject လုပ်ပေးပါ။
```

လို့ ASP.NET Core ကို ပြောလိုက်တာဖြစ်ပါတယ်။

## Step 4 - Controller မှာ Inject လုပ်မယ်

```csharp
[ApiController]
[Route("api/[controller]")]
public sealed class ProductsController : ControllerBase
{
    private readonly ProductService _productService;

    public ProductsController(ProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public ActionResult<IReadOnlyList<ProductDto>> GetAll()
    {
        return Ok(_productService.GetAll());
    }
}
```

Request ဝင်လာတဲ့အခါ ASP.NET Core က Controller ကိုဆောက်ပြီး `ProductService` ကို အလိုအလျောက် Inject လုပ်ပေးပါတယ်။

---

# DI သုံးရင် Interface မဖြစ်မနေလိုလား?

မလိုပါဘူး။

```csharp
builder.Services.AddScoped<ProductService>();
```

လို Concrete Class ကို တိုက်ရိုက် Register လုပ်ပြီး Inject လုပ်တာလည်း Dependency Injection ပါပဲ။

Interface ကို

* Implementation အမှန်တကယ် နှစ်မျိုးထက်ပိုရှိတဲ့အခါ
* External System နဲ့ Layer Boundary ကို Contract ခွဲချင်တဲ့အခါ
* Test မှာ Fake Implementation ထည့်ဖို့လိုတဲ့အခါ

သုံးနိုင်ပါတယ်။

Service တစ်ခု၊ Implementation တစ်ခုတည်းရှိပြီး ပြောင်းဖို့မရှိသေးရင် Interface အပိုမဆောက်လည်း ရပါတယ်။

---

# Interface နဲ့ Register လုပ်ခြင်း

Email နဲ့ SMS Notification ကို ပြောင်းသုံးနိုင်မယ့် Requirement တကယ်ရှိတယ်ဆိုပါစို့။

## Contract

```csharp
public interface INotificationService
{
    Task SendAsync(
        string message,
        CancellationToken cancellationToken);
}
```

## Email Implementation

```csharp
public sealed class EmailNotificationService
    : INotificationService
{
    public Task SendAsync(
        string message,
        CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }
}
```

## SMS Implementation

```csharp
public sealed class SmsNotificationService
    : INotificationService
{
    public Task SendAsync(
        string message,
        CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }
}
```

## Registration

```csharp
builder.Services.AddScoped<
    INotificationService,
    EmailNotificationService>();
```

ဒီ Registration က

```text
INotificationService တောင်းလာရင်

EmailNotificationService ကို

Inject လုပ်ပေးပါ။
```

လို့ဆိုလိုပါတယ်။

## Injection

```csharp
public sealed class RegistrationService
{
    private readonly INotificationService _notification;

    public RegistrationService(
        INotificationService notification)
    {
        _notification = notification;
    }

    public Task RegisterAsync(
        CancellationToken cancellationToken)
    {
        return _notification.SendAsync(
            "Welcome",
            cancellationToken);
    }
}
```

နောက်ပိုင်း SMS ကို ပြောင်းသုံးဖို့ လိုလာရင် Consumer Code ကို မပြင်ဘဲ Registration ကို ပြောင်းနိုင်ပါတယ်။

```csharp
builder.Services.AddScoped<
    INotificationService,
    SmsNotificationService>();
```

---

# Service Lifetime

Service Lifetime ဆိုတာ Container က Service Instance တစ်ခုကို ဘယ်လောက်ကြာကြာ ထားသုံးမလဲဆိုတာ ဖြစ်ပါတယ်။

ASP.NET Core မှာ အဓိက Lifetime သုံးမျိုးရှိပါတယ်။

| Lifetime | Instance အသစ်ဆောက်ချိန် | အသုံးများတဲ့နေရာ |
|---|---|---|
| Transient | Resolve လုပ်တိုင်း | ပေါ့ပါးပြီး State မရှိတဲ့ Service |
| Scoped | HTTP Request တစ်ခုလျှင် တစ်ခု | Business Service, Repository, `DbContext` |
| Singleton | Application တစ်ခုလုံးအတွက် တစ်ခု | Thread-safe ဖြစ်တဲ့ Shared Service |

---

# 1. AddTransient

```csharp
builder.Services.AddTransient<EmailFormatter>();
```

Transient Service ကို Container က တောင်းတိုင်း Instance အသစ် ဆောက်ပေးပါတယ်။

```text
Resolve 1 ──► EmailFormatter Instance A
Resolve 2 ──► EmailFormatter Instance B
Resolve 3 ──► EmailFormatter Instance C
```

အသုံးပြုရန် သင့်တော်တဲ့နေရာ

* ပေါ့ပါးတဲ့ Formatter
* State မသိမ်းတဲ့ Calculator
* သက်တမ်းတိုတဲ့ Helper Service

Transient ကို မလိုဘဲ Resolve များများလုပ်ရင် Object Allocation များနိုင်ပါတယ်။

---

# 2. AddScoped

```csharp
builder.Services.AddScoped<ProductService>();
```

ASP.NET Core Web Application မှာ Scoped Service ကို HTTP Request တစ်ခုလျှင် Instance တစ်ခု ဆောက်ပေးပါတယ်။

Request တစ်ခုအတွင်း နေရာအများကြီးက တောင်းရင် Instance တစ်ခုတည်း ပြန်ရပါတယ်။ နောက် Request မှာ Instance အသစ်ရပါတယ်။

```text
Request A
├── Controller ──► RequestContext A
└── Service ─────► RequestContext A

Request B
└── Controller ──► RequestContext B
```

အသုံးပြုရန် သင့်တော်တဲ့နေရာ

* Business Service
* Repository
* Request နဲ့သက်ဆိုင်တဲ့ State
* EF Core `DbContext`

`AddDbContext<TContext>()` က `DbContext` ကို Default အနေနဲ့ Scoped Register လုပ်ပေးပါတယ်။

Web API Project တွေမှာ Business Service နဲ့ Repository အတွက် Scoped ကို စတင်ရွေးချယ်လို့ရပါတယ်။ တကယ့် Lifetime လိုအပ်ချက်ရှိမှ ပြောင်းသင့်ပါတယ်။

---

# 3. AddSingleton

```csharp
builder.Services.AddSingleton<ExchangeRateCache>();
```

Singleton Service ကို ပထမဆုံး တောင်းတဲ့အချိန်မှာ Container က တစ်ကြိမ်ပဲ ဆောက်ပြီး နောက်ပိုင်း Request အားလုံးကို Instance တစ်ခုတည်း ပြန်ပေးပါတယ်။

```text
Request A ──► ExchangeRateCache A
Request B ──► ExchangeRateCache A
Request C ──► ExchangeRateCache A
```

Application ပိတ်တဲ့အထိ အဲ့ဒီ Instance ကို ထားသုံးပါတယ်။

Singleton က Request အများကြီးက တစ်ပြိုင်နက် သုံးနိုင်တာကြောင့်

* Thread-safe ဖြစ်ရမယ်
* Request-specific Data မသိမ်းရဘူး
* User တစ်ယောက်ရဲ့ Data ကို Field ထဲမထားရဘူး
* Memory အများကြီး အမြဲမဖမ်းထားသင့်ဘူး

ဆိုတာတွေ သတိထားရပါတယ်။

Shared Instance တကယ်လိုတဲ့အခါမှ Singleton ကို သုံးသင့်ပါတယ်။

---

# Lifetime ရွေးချယ်ပုံ

```text
Request တစ်ခုလျှင် Instance တစ်ခုလိုလား?
        │
        ├── Yes ──► Scoped
        │
        └── No
             │
             ├── Resolve တိုင်း အသစ်လိုလား? ──► Transient
             │
             └── App တစ်ခုလုံး Share ရမလား? ──► Singleton
```

မသေချာရင် Web API Business Service နဲ့ Repository ကို Scoped နဲ့စတာက ရိုးရှင်းပါတယ်။

Lifetime ကို Performance ခန့်မှန်းပြီး မပြောင်းဘဲ State နဲ့ Ownership လိုအပ်ချက်အပေါ် မူတည်ပြီး ရွေးသင့်ပါတယ်။

---

# အရေးကြီးဆုံး Lifetime Rule

သက်တမ်းရှည်တဲ့ Service က သက်တမ်းတိုတဲ့ Scoped Service ကို ဖမ်းမထားရပါဘူး။

အထူးသဖြင့် **Singleton ထဲကို Scoped Service Inject မလုပ်ရပါဘူး**။

```csharp
builder.Services.AddDbContext<AppDbContext>();
builder.Services.AddSingleton<ReportCache>();

public sealed class ReportCache
{
    // Wrong: Singleton က Scoped DbContext ကို ဖမ်းထားတယ်
    public ReportCache(AppDbContext dbContext)
    {
    }
}
```

ဒီလိုရေးရင် Request တစ်ခုစာပဲ သက်တမ်းရှိသင့်တဲ့ `AppDbContext` ကို Singleton က Application တစ်လျှောက် ဖမ်းထားနိုင်ပါတယ်။

ရိုးရှင်းတဲ့ Fix က Consumer ကို Scoped ပြောင်းတာဖြစ်ပါတယ်။

```csharp
builder.Services.AddScoped<ReportService>();
```

Background Service လို Singleton က Scoped Service ကို တကယ်ခေါ်ရမယ်ဆိုရင် `IServiceScopeFactory` နဲ့ အလုပ်တစ်ကြိမ်စီအတွက် Scope အသစ်ဆောက်ရပါတယ်။

---

# N-Layered Architecture မှာ DI ကို ဘယ်လိုသုံးလဲ?

```text
Presentation Layer
------------------
ProductsController
        │
        ▼
Business Layer
--------------
ProductService
        │
        ▼
Data Access Layer
-----------------
ProductRepository
        │
        ▼
AppDbContext
        │
        ▼
SQL Server
```

`Program.cs`

```csharp
builder.Services.AddScoped<ProductRepository>();
builder.Services.AddScoped<ProductService>();
```

ဒီ Architecture မှာ

* Controller က Service ကို Constructor ကနေ တောင်းတယ်
* Service က Repository ကို Constructor ကနေ တောင်းတယ်
* Repository က `AppDbContext` ကို Constructor ကနေ တောင်းတယ်
* DI Container က Object Graph တစ်ခုလုံး ဆောက်ပေးတယ်

Layer တိုင်းမှာ `new` နဲ့ နောက် Layer ကို ကိုယ်တိုင်မဆောက်တော့တာကြောင့် Object Creation Code ကို `Program.cs` တစ်နေရာတည်းက စီမံနိုင်ပါတယ်။

Layer Boundary အတွက် Contract တကယ်လိုရင် `IProductRepository` လို Interface ထည့်နိုင်ပါတယ်။ မလိုသေးရင် Concrete Class နဲ့တင် စနိုင်ပါတယ်။

---

# Real World Example - Internet Banking

Customer က ငွေလွှဲတယ်ဆိုပါစို့။

```text
Client
  │
  ▼
TransferController
  │
  ▼
TransferService
  │
  ▼
AccountRepository
  │
  ▼
BankDbContext
  │
  ▼
SQL Server
```

Registration

```csharp
builder.Services.AddScoped<AccountRepository>();
builder.Services.AddScoped<TransferService>();
```

`TransferService`

```csharp
public sealed class TransferService
{
    private readonly AccountRepository _accounts;

    public TransferService(AccountRepository accounts)
    {
        _accounts = accounts;
    }

    public Task TransferAsync(
        decimal amount,
        CancellationToken cancellationToken)
    {
        return _accounts.TransferAsync(
            amount,
            cancellationToken);
    }
}
```

Service က Repository ကို ဘယ်လိုဆောက်ရမလဲ မသိပါဘူး။ Repository ကို လိုတယ်လို့ Constructor ကနေ ပြောရုံပါပဲ။

DI Container က `AccountRepository` နဲ့ သူလိုအပ်တဲ့ `BankDbContext` ကို အလိုအလျောက် ဖြည့်ပေးပါတယ်။

---

# DI Container က Dispose လုပ်ပေးလား?

လုပ်ပေးပါတယ်။

Container က ဆောက်ပေးထားတဲ့ `IDisposable` နဲ့ `IAsyncDisposable` Service တွေကို သက်ဆိုင်ရာ Lifetime ပြီးတဲ့အခါ Dispose လုပ်ပေးပါတယ်။

* Scoped Service ကို Request ပြီးတဲ့အခါ Dispose လုပ်တယ်
* Singleton Service ကို Application ပိတ်တဲ့အခါ Dispose လုပ်တယ်

Container က Inject လုပ်ပေးထားတဲ့ Service ကို Consumer က ကိုယ်တိုင် Dispose မလုပ်သင့်ပါဘူး။

```csharp
public ProductsController(AppDbContext dbContext)
{
    _dbContext = dbContext;
}

// _dbContext.Dispose() မခေါ်ရပါဘူး။
```

Object ကို ကိုယ်တိုင် `new` နဲ့ဆောက်ထားရင်တော့ သူ့ Lifetime ကို ကိုယ်တိုင် စီမံရပါတယ်။

---

# Common Mistakes

## 1. Service ကို Register မလုပ်ခြင်း

Constructor မှာ Service တောင်းထားပေမယ့် `Program.cs` မှာ Register မလုပ်ထားရင် Runtime မှာ Service Resolve မလုပ်နိုင်တဲ့ Error တက်ပါတယ်။

```text
Unable to resolve service for type 'ProductService'
```

## 2. Singleton ထဲ Scoped Service Inject လုပ်ခြင်း

`DbContext` နဲ့ Scoped Service တွေကို Singleton ထဲ မဖမ်းထားရပါဘူး။

## 3. Service Locator သုံးခြင်း

Class ထဲမှာ `IServiceProvider.GetRequiredService()` ခေါ်ပြီး Dependency ကို ဖျောက်မထားသင့်ပါဘူး။

```csharp
// Avoid
var service = serviceProvider
    .GetRequiredService<ProductService>();
```

Constructor မှာ တိုက်ရိုက်တောင်းတာ ပိုရှင်းပြီး Test လုပ်ရလွယ်ပါတယ်။

## 4. Constructor Dependency အရမ်းများခြင်း

Class တစ်ခုမှာ Inject လုပ်ထားတဲ့ Dependency အရမ်းများနေရင် DI ပြဿနာမဟုတ်ဘဲ Class က တာဝန်အရမ်းများနေတဲ့ လက္ခဏာ ဖြစ်နိုင်ပါတယ်။

## 5. Interface အလွတ်များ ဆောက်ခြင်း

Implementation တစ်ခုတည်းရှိပြီး Contract ခွဲဖို့ အကြောင်းမရှိသေးရင် Interface ဆောက်စရာ မလိုပါဘူး။ လိုအပ်လာမှ ထည့်နိုင်ပါတယ်။

---

# Dependency Injection ရဲ့ အားသာချက်များ

### 1. Object Creation တစ်နေရာတည်း စီမံနိုင်တယ်

Service Registration နဲ့ Lifetime ကို `Program.cs` မှာ စုထားနိုင်ပါတယ်။

### 2. Dependency ရှင်းရှင်းလင်းလင်း မြင်ရတယ်

Constructor ကိုကြည့်တာနဲ့ Class တစ်ခု အလုပ်လုပ်ဖို့ ဘာတွေလိုသလဲ သိနိုင်ပါတယ်။

### 3. Test လုပ်ရလွယ်တယ်

Test မှာ လိုအပ်တဲ့ Dependency ကို Fake သို့မဟုတ် Test Implementation နဲ့ ထည့်ပေးနိုင်ပါတယ်။

### 4. Lifetime ကို Container က စီမံပေးတယ်

Transient, Scoped, Singleton Lifetime နဲ့ Disposal ကို စနစ်တကျ စီမံပေးပါတယ်။

### 5. Implementation ပြောင်းရလွယ်တယ်

Interface Contract သုံးထားတဲ့နေရာမှာ Consumer ကို မပြင်ဘဲ Registration ကနေ Implementation ပြောင်းနိုင်ပါတယ်။

---

# Summary

**Dependency Injection** ဆိုတာ Class တစ်ခုက လိုအပ်တဲ့ Dependency ကို ကိုယ်တိုင် `new` နဲ့ မဆောက်ဘဲ Constructor ကနေ တောင်းပြီး DI Container က ထည့်ပေးတဲ့ နည်းလမ်းဖြစ်ပါတယ်။

ASP.NET Core မှာ Service ကို `Program.cs` ရဲ့ `IServiceCollection` ထဲ Register လုပ်ပြီး Constructor Injection နဲ့ အသုံးပြုပါတယ်။ Container က Dependency Graph ဆောက်ပေးသလို Service Lifetime နဲ့ Disposal ကိုပါ စီမံပေးပါတယ်။

Web API Business Service နဲ့ Repository တွေကို Scoped နဲ့ စနိုင်ပါတယ်။ Transient ကို Resolve တိုင်း Instance အသစ်လိုတဲ့ ပေါ့ပါးတဲ့ Service အတွက်၊ Singleton ကို Application တစ်ခုလုံး Share လုပ်ရမယ့် Thread-safe Service အတွက်သာ သုံးသင့်ပါတယ်။

DI သုံးတိုင်း Interface မဖြစ်မနေ မလိုပါဘူး။ Concrete Class နဲ့ စပြီး Multiple Implementation သို့မဟုတ် Layer Contract တကယ်လိုလာတဲ့အခါမှ Interface ထည့်တာက ပိုရိုးရှင်းပါတယ်။

---

# ဆက်လက်လေ့လာရန်

* [Microsoft Learn - Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
* [Microsoft Learn - Service lifetimes](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes)
* [Microsoft Learn - Dependency injection guidelines](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines)
