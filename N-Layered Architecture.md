# N-Layered Architecture ဆိုတာဘာလဲ?

**N-Layered Architecture** ဆိုတာ Application ကို တာဝန်အလိုက် Layer တွေခွဲပြီး ရေးသားတဲ့ Architecture Pattern ဖြစ်ပါတယ်။

ဒီ Project မှာ Layer သုံးခု သုံးပါမယ်။

| Layer | တာဝန် |
|---|---|
| Presentation | Controller က Request လက်ခံပြီး Response ပြန်ပေးတယ် |
| Business | Service က Business Logic လုပ်ပြီး Database Layer ကို ခေါ်တယ် |
| Database | EF Core Database First နဲ့ Dapper ကို ထားတယ် |

## Request Flow

```text
Client
  │
  ▼
Controller
  │
  ▼
Service
  │
  ├── EF Core DbContext
  └── Dapper
         │
         ▼
      Database
```

Repository Layer မထည့်ပါဘူး။ Service က `DbContext` သို့မဟုတ် Dapper ကို တိုက်ရိုက်သုံးတာကြောင့် မလိုအပ်တဲ့ Pass-through Code နည်းပြီး Flow က ပိုရှင်းပါတယ်။

## Project Structure

```text
MyApp.Api
└── Controllers

MyApp.Business
└── Services

MyApp.Database
├── Models
├── AppDbContext.cs
└── DapperContext.cs
```

Project Dependency က အောက်ပါအတိုင်း ဖြစ်ပါတယ်။

```text
MyApp.Api → MyApp.Business → MyApp.Database
```

## Database Layer

### EF Core Database First

ရှိပြီးသား Database ကနေ `DbContext` နဲ့ Model တွေ Generate လုပ်ပါတယ်။

```bash
dotnet ef dbcontext scaffold "Name=ConnectionStrings:DefaultConnection" Microsoft.EntityFrameworkCore.SqlServer --context AppDbContext --context-dir . --output-dir Models
```

Generate ဖြစ်လာတဲ့ `AppDbContext` နဲ့ Model တွေကို `MyApp.Database` Project ထဲမှာ ထားပါမယ်။

### Dapper Context

```csharp
public class DapperContext
{
    private readonly string _connectionString;

    public DapperContext(IConfiguration configuration)
    {
        _connectionString = configuration.GetConnectionString(
            "DefaultConnection");
    }

    public IDbConnection CreateConnection()
    {
        return new SqlConnection(_connectionString);
    }
}
```

## Business Layer

Service က Business Logic နဲ့ Database Call ကို တစ်နေရာတည်းမှာ ရှင်းရှင်းလင်းလင်း ရေးနိုင်ပါတယ်။

```csharp
public class ProductService : IProductService
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

    public async Task<IEnumerable<ProductDto>> GetActiveAsync()
    {
        const string sql = @"
            SELECT Id, Name, Price
            FROM Products
            WHERE IsActive = 1";

        using (IDbConnection connection =
            _dapperContext.CreateConnection())
        {
            return await connection.QueryAsync<ProductDto>(sql);
        }
    }
}
```

EF Core ကို Insert, Update နဲ့ ရိုးရှင်းတဲ့ Query တွေအတွက် သုံးနိုင်ပါတယ်။ Dapper ကို Complex Query နဲ့ Report တွေအတွက် သုံးနိုင်ပါတယ်။

## Presentation Layer

Controller က Service ကိုပဲ ခေါ်ပါတယ်။ Database Code နဲ့ Business Logic မရေးပါဘူး။

```csharp
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
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

## Dependency Injection

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"));
});

builder.Services.AddScoped<DapperContext>();
builder.Services.AddScoped<IProductService, ProductService>();
```

## သတိထားရန်

- Controller ထဲမှာ Database Code မရေးပါနဲ့
- Business Rule တွေကို Service ထဲမှာပဲ ထားပါ
- Generated EF Core Model တွေကို Controller ထဲမှာ တိုက်ရိုက်မပြင်ပါနဲ့
- Service ကြီးလာရင် Feature အလိုက် Service အသေးတွေ ခွဲပါ

## အကျဉ်းချုပ်

ဒီ Architecture မှာ **Controller → Service → Database** ဆိုတဲ့ ရိုးရှင်းတဲ့ Flow ကို သုံးပါတယ်။ Repository မထည့်ဘဲ Service က EF Core Database First နဲ့ Dapper ကို တိုက်ရိုက်သုံးတာကြောင့် Code Layer အဆင့်ဆင့် ထပ်မနေဘဲ ဖတ်ရလွယ်ပါတယ်။
