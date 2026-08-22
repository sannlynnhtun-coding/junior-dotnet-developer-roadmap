# Dependency Injection (DI) ဆိုတာဘာလဲ?

**Dependency Injection (DI)** ဆိုတာ Class တစ်ခုက လိုအပ်တဲ့ Object ကို ကိုယ်တိုင် `new` နဲ့ မဆောက်ဘဲ Constructor ကနေ လက်ခံအသုံးပြုတဲ့ နည်းလမ်းဖြစ်ပါတယ်။

လိုအပ်တဲ့ Object တွေကို ASP.NET Core ရဲ့ **DI Container** က ဖန်တီးပြီး ထည့်ပေးပါတယ်။

## Dependency ဆိုတာဘာလဲ?

Class တစ်ခု အလုပ်လုပ်ဖို့ လိုအပ်တဲ့ အခြား Object ကို Dependency လို့ခေါ်ပါတယ်။

ဥပမာ `ProductsController` က `ProductService` ကို လိုအပ်တဲ့အတွက် `ProductService` က သူ့ရဲ့ Dependency ဖြစ်ပါတယ်။

```text
ProductsController → ProductService → AppDbContext
```

## DI မသုံးရင်

```csharp
public class UsersController
{
    private readonly EmailService _emailService;

    public UsersController()
    {
        _emailService = new EmailService();
    }
}
```

Controller က `EmailService` ကို ဘယ်လိုဆောက်ရမလဲ သိနေရပါတယ်။ `EmailService` ရဲ့ Constructor ပြောင်းရင် Controller ကိုပါ ပြင်ရပါမယ်။ ဒါကို **Tight Coupling** လို့ခေါ်ပါတယ်။

## Constructor Injection (Recommended)

```csharp
public class UsersController : ControllerBase
{
    private readonly EmailService _emailService;

    public UsersController(EmailService emailService)
    {
        _emailService = emailService;
    }
}
```

Controller က လိုအပ်တဲ့ `EmailService` ကို Constructor ကနေ တောင်းရုံပါပဲ။ ဒီနည်းကို **Constructor Injection** လို့ခေါ်ပါတယ်။

Class အလုပ်လုပ်ဖို့ Dependency မဖြစ်မနေလိုအပ်တဲ့အခါ သုံးပါတယ်။ Dependency ကို Constructor မှာ ရှင်းရှင်းလင်းလင်း မြင်ရပြီး Object ဆောက်ပြီးတာနဲ့ အသုံးပြုဖို့ အဆင်သင့်ဖြစ်ပါတယ်။

## Method Injection

Dependency ကို လိုအပ်တဲ့ Method ရဲ့ Parameter ကနေ ထည့်ပေးတာ ဖြစ်ပါတယ်။ ASP.NET Core Controller Action မှာ `[FromServices]` သုံးနိုင်ပါတယ်။

```csharp
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult Register(
        [FromServices] EmailService emailService)
    {
        emailService.Send("Welcome");
        return Ok();
    }
}
```

Dependency ကို Method တစ်ခုတည်းမှာပဲ လိုအပ်တဲ့အခါ သင့်တော်ပါတယ်။ Method အများကြီးမှာ လိုအပ်ရင် Constructor Injection သုံးပါ။

## Property Injection

Dependency ကို Public Property ကနေ ထည့်ပေးတာ ဖြစ်ပါတယ်။

```csharp
public class UserService
{
    public EmailService EmailService { get; set; }

    public void Register()
    {
        if (EmailService == null)
        {
            throw new InvalidOperationException(
                "EmailService is required.");
        }

        EmailService.Send("Welcome");
    }
}
```

Property ကို အပြင်ကနေ သတ်မှတ်မပေးရင် `null` ဖြစ်နိုင်ပြီး Object က မပြည့်စုံတဲ့အခြေအနေမှာ ရှိနိုင်ပါတယ်။ ASP.NET Core Built-in DI Container က ပုံမှန် Property Injection ကို အလိုအလျောက် မလုပ်ပေးပါဘူး။ ဒါကြောင့် အများအားဖြင့် Constructor Injection ကို သုံးသင့်ပါတယ်။

| နည်းလမ်း | ဘယ်အချိန်သုံးမလဲ |
|---|---|
| Constructor Injection | Class အတွက် Dependency မဖြစ်မနေလိုအပ်ရင် |
| Method Injection | Method တစ်ခုတည်းမှာသာ လိုအပ်ရင် |
| Property Injection | Optional Dependency ဖြစ်ပြီး Container က Support လုပ်ရင် |

## Service Registration

DI Container က ဘယ် Object ကို ဆောက်ပေးရမလဲ သိနိုင်ဖို့ `Program.cs` မှာ Register လုပ်ရပါတယ်။

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"));
});

builder.Services.AddScoped<DapperContext>();
builder.Services.AddScoped<EmailService>();
builder.Services.AddScoped<ProductService>();
```

Request ဝင်လာရင် DI Container က Object တွေကို အောက်ပါအတိုင်း ဆောက်ပေးပါတယ်။

```text
ProductsController
       │
       ▼
ProductService
       │
       ├── AppDbContext
       └── DapperContext
```

Repository မလိုဘဲ Service က EF Core သို့မဟုတ် Dapper ကို တိုက်ရိုက်သုံးနိုင်ပါတယ်။

## Service Example

```csharp
public class ProductService
{
    private readonly AppDbContext _dbContext;
    private readonly DapperContext _dapperContext;

    public ProductService(
        AppDbContext dbContext,
        DapperContext dapperContext)
    {
        _dbContext = dbContext;
        _dapperContext = dapperContext;
    }

    public async Task<Product> GetByIdAsync(int id)
    {
        return await _dbContext.Products
            .FirstOrDefaultAsync(x => x.Id == id);
    }
}
```

## Controller Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ProductService _productService;

    public ProductsController(ProductService productService)
    {
        _productService = productService;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        Product product = await _productService.GetByIdAsync(id);

        if (product == null)
        {
            return NotFound();
        }

        return Ok(product);
    }
}
```

## Service Lifetime

Lifetime ဆိုတာ DI Container က Object တစ်ခုကို ဘယ်လောက်ကြာကြာ အသုံးပြုမလဲဆိုတာ ဖြစ်ပါတယ်။

| Lifetime | Instance အသစ်ဆောက်ချိန် | အသုံးများတဲ့နေရာ |
|---|---|---|
| Transient | တောင်းတိုင်း | ပေါ့ပါးပြီး State မရှိတဲ့ Service |
| Scoped | HTTP Request တစ်ခုလျှင် တစ်ခု | Business Service, `DbContext` |
| Singleton | Application တစ်ခုလုံး တစ်ခု | Thread-safe Shared Service |

```csharp
builder.Services.AddTransient<EmailFormatter>();
builder.Services.AddScoped<ProductService>();
builder.Services.AddSingleton<ExchangeRateCache>();
```

Web API မှာ Business Service နဲ့ `DbContext` အတွက် `Scoped` ကို အများဆုံးသုံးပါတယ်။

> Singleton Service ထဲကို Scoped Service သို့မဟုတ် `DbContext` မထည့်ပါနဲ့။ Singleton က Scoped Object ကို Application တစ်လျှောက် ဖမ်းထားနိုင်ပါတယ်။

## Interface လိုသလား?

DI သုံးဖို့ Interface မဖြစ်မနေ မလိုပါဘူး။

```csharp
builder.Services.AddScoped<ProductService>();
```

Implementation အမျိုးမျိုးရှိတဲ့အခါ Interface သုံးနိုင်ပါတယ်။

```csharp
builder.Services.AddScoped<INotificationService, EmailService>();
```

Implementation တစ်ခုတည်းရှိရင် Concrete Class နဲ့ စတင်တာ ပိုရိုးရှင်းပါတယ်။

## သတိထားရန်

- Constructor မှာ Inject လုပ်ထားတဲ့ Service ကို `Program.cs` မှာ Register လုပ်ပါ
- Inject လုပ်ပေးထားတဲ့ `DbContext` ကို ကိုယ်တိုင် Dispose မလုပ်ပါနဲ့
- Class ထဲမှာ `IServiceProvider.GetRequiredService()` သုံးပြီး Dependency မဖျောက်ပါနဲ့
- Constructor Dependency အရမ်းများရင် Class ရဲ့ တာဝန်တွေကို ခွဲပါ

## အကျဉ်းချုပ်

Dependency Injection က Class တွေကို `new` နဲ့ တိုက်ရိုက်ချိတ်ဆက်မထားဘဲ လိုအပ်တဲ့ Object ကို Constructor ကနေ ထည့်ပေးပါတယ်။ ASP.NET Core မှာ Service တွေကို `Program.cs` မှာ Register လုပ်ပြီး Web Request နဲ့သက်ဆိုင်တဲ့ Service တွေအတွက် `Scoped` Lifetime ကို အသုံးများပါတယ်။

## ဆက်လက်လေ့လာရန်

- [Microsoft Learn - Dependency injection in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection)
- [Microsoft Learn - Service lifetimes](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection/service-lifetimes)
