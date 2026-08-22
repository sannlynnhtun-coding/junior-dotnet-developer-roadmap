# Result Pattern ဆိုတာဘာလဲ?

**Result Pattern** ဆိုတာ Method တစ်ခုရဲ့ အောင်မြင်မှု၊ Data နဲ့ Error ကို Object တစ်ခုတည်းနဲ့ ပြန်ပေးတဲ့ Pattern ဖြစ်ပါတယ်။

```csharp
Result<ProductDto>
```

ဒီ Return Type ကိုကြည့်တာနဲ့ Method က Product Data သို့မဟုတ် Error ပြန်ပေးနိုင်တယ်ဆိုတာ သိနိုင်ပါတယ်။

## ဘာကြောင့်သုံးတာလဲ?

- `null` ပြန်လာတဲ့ အကြောင်းရင်းကို ရှင်းရှင်းလင်းလင်း သိနိုင်တယ်
- Validation နဲ့ Not Found လို Business Error တွေအတွက် Exception မလိုဘူး
- Service က Business Result ပြန်ပြီး Controller က HTTP Status Code သတ်မှတ်နိုင်တယ်

## Result Class

```csharp
public enum ErrorType
{
    None,
    Validation,
    NotFound,
    Conflict
}

public class Result<T>
{
    public bool IsSuccess { get; private set; }
    public T Data { get; private set; }
    public string ErrorMessage { get; private set; }
    public ErrorType ErrorType { get; private set; }

    private Result(
        bool isSuccess,
        T data,
        string errorMessage,
        ErrorType errorType)
    {
        IsSuccess = isSuccess;
        Data = data;
        ErrorMessage = errorMessage;
        ErrorType = errorType;
    }

    public static Result<T> Success(T data)
    {
        return new Result<T>(
            true,
            data,
            string.Empty,
            ErrorType.None);
    }

    public static Result<T> Failure(
        string errorMessage,
        ErrorType errorType)
    {
        return new Result<T>(
            false,
            default(T),
            errorMessage,
            errorType);
    }
}
```

## Service မှာသုံးမယ်

```csharp
public class ProductService
{
    private readonly IProductRepository _repository;

    public ProductService(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<Result<ProductDto>> GetByIdAsync(int id)
    {
        var product = await _repository.GetByIdAsync(id);

        if (product == null)
        {
            return Result<ProductDto>.Failure(
                "Product not found.",
                ErrorType.NotFound);
        }

        var productDto = new ProductDto
        {
            Id = product.Id,
            Name = product.Name
        };

        return Result<ProductDto>.Success(productDto);
    }
}
```

Service က HTTP Status Code မပြန်ဘဲ Business Result ကိုပဲ ပြန်ပေးပါတယ်။

## Controller မှာသုံးမယ်

```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var result = await _productService.GetByIdAsync(id);

    if (result.IsSuccess)
    {
        return Ok(result.Data);
    }

    if (result.ErrorType == ErrorType.NotFound)
    {
        return NotFound(result.ErrorMessage);
    }

    return BadRequest(result.ErrorMessage);
}
```

## ဘယ်အချိန်သုံးမလဲ?

Result Pattern ကို ခန့်မှန်းနိုင်တဲ့ Business Error တွေအတွက် သုံးပါ။

- Validation Error
- Record မတွေ့ခြင်း
- Duplicate Record
- Business Rule မကိုက်ညီခြင်း

Database Connection ပျက်ခြင်း၊ Network Error နဲ့ Programming Bug လို မမျှော်လင့်ထားတဲ့ System Error တွေအတွက်တော့ Exception ကို သုံးပါ။

## အကျဉ်းချုပ်

`Result<T>` က Success Data သို့မဟုတ် Error Detail ကို တိတိကျကျ ပြန်ပေးပါတယ်။ Service က `Result<T>` ပြန်ပြီး Controller က အဲဒီ Result ကို `200`, `400`, `404` စတဲ့ HTTP Status Code အဖြစ် ပြောင်းပေးပါတယ်။
