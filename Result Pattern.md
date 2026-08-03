# Result Pattern ဆိုတာဘာလဲ?

အခုနောက်ပိုင်း ကျွန်တော်တို့ ရေးရတဲ့ **ASP.NET Core Web API Project** တွေတိုင်းမှာ မဖြစ်မနေ ထည့်သုံးဖြစ်တဲ့ Pattern တစ်ခုရှိပါတယ်။

ဒါကတော့ **Result Pattern** ဖြစ်ပါတယ်။

**Result Pattern** ဆိုတာကတော့ **Method** တစ်ခုကနေ ထွက်လာတဲ့ **ရလဒ် (Outcome)** ကို **Object** တစ်ခုတည်းနဲ့ ပြန်ပေးတဲ့ **Programming Design Pattern** တစ်ခု ဖြစ်ပါတယ်။

ပုံမှန်အားဖြင့် Method တစ်ခုက

* Data ကိုပဲ Return ပြန်ပေးတာ
* `null` Return ပြန်ပေးတာ
* Exception Throw လုပ်တာ

လိုမျိုးလုပ်လေ့ရှိပါတယ်။

ဒါပေမယ့် Result Pattern ကို သုံးတဲ့အခါ

Method က

* အောင်မြင်လား
* မအောင်မြင်ဘူးလား
* Data ရှိလား
* Error Message ဘာလဲ
* Error Type ဘာလဲ

ဆိုတာတွေကို **Result Object** တစ်ခုထဲမှာ စုစည်းပြီး Return ပြန်ပေးပါတယ်။

---

# ASP.NET Core Web API မှာ ဘယ်နေရာမှာ သုံးလဲ?

ဥပမာ...

Product API တစ်ခုရှိတယ်ဆိုပါစို့။

```
GET /api/products/100
```

User က Product တစ်ခုကို Request လုပ်လိုက်တယ်။

ဒီ Request က

```text
Client
   │
   ▼
ProductsController
   │
   ▼
ProductService
   │
   ▼
ProductRepository
   │
   ▼
SQL Server
```

ဒီ Flow အတိုင်း သွားပါတယ်။

ဒီမှာ **Result Pattern** ကို အများအားဖြင့်

✅ Service Layer (Business Layer)

မှာ Return ပြန်ပေးလေ့ရှိပါတယ်။

---

# N-Layered Architecture မှာ Result Pattern

```text
Presentation Layer
        │
        ▼
 ProductsController
        │
        ▼
 Business Layer
 ProductService
        │
        ▼
 Result<ProductDto>
        │
        ▼
 Data Access Layer
 ProductRepository
        │
        ▼
 SQL Server
```

ဒီ Architecture မှာ

Controller က Database ကို မသွားပါဘူး။

Repository က HTTP Response ကို မပြန်ပါဘူး။

Service က Business Logic တွက်ပြီး

Result Object ကို ပြန်ပေးပါတယ်။

---

# Real World Example

ဥပမာ

Customer က

```
GET /api/products/10
```

ကို Request လုပ်လိုက်တယ်။

---

## Step 1

Controller က

```csharp
var result = await _productService.GetByIdAsync(id);
```

လို့ Service ကိုခေါ်ပါတယ်။

Controller က Database မသွားပါဘူး။

---

## Step 2

Service က

Repository ကိုခေါ်ပြီး

Database ထဲမှာ Product ရှိမရှိ သွားစစ်ပါတယ်။

```text
Controller
      │
      ▼
ProductService
      │
      ▼
ProductRepository
      │
      ▼
Database
```

---

## Step 3

Database မှာ

Product မရှိဘူးဆိုရင်

Service က

Exception Throw မလုပ်ပါဘူး။

`null` လည်း Return မလုပ်ပါဘူး။

အဲ့ဒီအစား

```csharp
return Result<ProductDto>.NotFoundError(
    "Product not found."
);
```

ကို Return ပြန်ပေးပါတယ်။

---

## Step 4

Controller က Result ကိုကြည့်ပြီး

```csharp
if(result.IsNotFound)
{
    return NotFound(result);
}
```

HTTP 404 ကို ပြန်ပေးလိုက်ပါတယ်။

---

# Result Pattern မသုံးရင်

ဥပမာ

```csharp
public Product GetById(int id)
```

ဒီ Method ဆိုရင်

Product မရှိတဲ့အခါ

ဘာပြန်လာမလဲ?

```csharp
null
```

လား

```csharp
throw Exception();
```

လား

Developer က Code ထဲဝင်ကြည့်မှ သိရပါတယ်။

---

# Result Pattern သုံးရင်

Method Signature ကိုကြည့်တာနဲ့

```csharp
public Result<ProductDto> GetByIdAsync(int id)
```

ဒီ Method က

Product ပြန်ပေးနိုင်တယ်။

ဒါပေမယ့်

Error လည်း ဖြစ်နိုင်တယ်ဆိုတာ

Signature ကိုကြည့်တာနဲ့ သိနိုင်ပါတယ်။

---

# Example 1 - Success

Database မှာ Product ရှိတယ်။

Service က

```csharp
return Result<ProductDto>.Success(product);
```

Controller က

```csharp
return Ok(result);
```

Client ရမယ့် Response

```json
{
  "isSuccess": true,
  "message": "Success",
  "data": {
    "id": 1,
    "name": "iPhone 17"
  }
}
```

---

# Example 2 - Not Found

Database ထဲမှာ Product မရှိဘူး။

Service

```csharp
return Result<ProductDto>.NotFoundError(
    "Product not found."
);
```

Controller

```csharp
return NotFound(result);
```

Client

```json
{
  "isSuccess": false,
  "message": "Product not found."
}
```

---

# Example 3 - Validation Error

User က

```json
{
    "price": -100
}
```

ပို့လိုက်တယ်။

Service က

```csharp
return Result<ProductDto>.ValidationError(
    "Price must be greater than zero."
);
```

Controller

```csharp
return BadRequest(result);
```

Client

```json
{
  "isSuccess": false,
  "message": "Price must be greater than zero."
}
```

---

# Result Pattern ရဲ့ အားသာချက်များ

### 1. Error Handling ပိုရှင်းတယ်

Exception Throw လုပ်တာထက်

```csharp
if(result.IsSuccess)
```

နဲ့စစ်ရုံပဲ။

---

### 2. Method Signature ကြည့်တာနဲ့ နားလည်တယ်

```csharp
Result<ProductDto>
```

ဆိုတာနဲ့

Success လည်း ဖြစ်နိုင်တယ်

Error လည်း ဖြစ်နိုင်တယ်ဆိုတာ သိတယ်။

---

### 3. Null Return မလုပ်တော့ဘူး

```csharp
return null;
```

မရှိတော့ဘူး။

အဲ့အစား

```csharp
return Result<Product>.NotFoundError();
```

ကို ပြန်ပေးတယ်။

---

### 4. Exception နည်းသွားတယ်

Business Error တွေအတွက်

Exception Throw မလုပ်တော့ဘူး။

Exception ကို

* SQL Connection Fail
* File Not Found
* Network Error

လိုမျိုး **System Error** တွေအတွက်ပဲ အသုံးပြုသင့်ပါတယ်။

Business Rule Error တွေ (ဥပမာ `NotFound`, `DuplicateRecord`, `ValidationError`) ကိုတော့ `Result<T>` နဲ့ ပြန်ပေးတာက ပိုသင့်တော်ပါတယ်။

---

# Summary

**Result Pattern** ဆိုတာ Method တစ်ခုရဲ့ **အောင်မြင်ခြင်း/မအောင်မြင်ခြင်း**, **Data**, **Error Message**, **Error Type** တွေကို `Result<T>` Object တစ်ခုတည်းနဲ့ ပြန်ပေးတဲ့ Design Pattern တစ်ခုဖြစ်ပါတယ်။

**ASP.NET Core Web API + N-Layered Architecture** မှာတော့ `Result<T>` ကို **Business Layer (Service Layer)** က Return ပြန်ပေးပြီး၊ **Presentation Layer (Controller)** က အဲ့ဒီ Result ကိုကြည့်ပြီး သင့်တော်တဲ့ **HTTP Status Code (200, 400, 404, 500)** အဖြစ် Client ကို ပြန်ပေးလေ့ရှိပါတယ်။ ဒီလိုခွဲထားခြင်းကြောင့် Business Logic နဲ့ HTTP Response Logic တို့ကို သီးခြားစီ ထိန်းသိမ်းနိုင်ပြီး Code က ပိုမိုရှင်းလင်း၊ စနစ်ကျပြီး Maintain လုပ်ရလည်း လွယ်ကူလာပါတယ်။
