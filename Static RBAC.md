# Static RBAC ဆိုတာဘာလဲ?

**Static RBAC (Static Role-Based Access Control)** ဆိုတာ User တစ်ယောက် ဘာလုပ်ခွင့်ရှိသလဲဆိုတာကို သူ့ရဲ့ **Role** နဲ့ ဆုံးဖြတ်ပေးတဲ့ Authorization ပုံစံတစ်ခုဖြစ်ပါတယ်။

ဒီပုံစံမှာ Role တွေ၊ Permission တွေ၊ Endpoint တစ်ခုချင်းစီအတွက် လိုအပ်တဲ့ Policy တွေကို Code သို့မဟုတ် သတ်မှတ်ထားတဲ့ User Data ထဲမှာ ကြိုတင်ရေးထားပါတယ်။

ဥပမာ...

* `Admin` က Product ကြည့်၊ အသစ်ထည့်၊ ပြင်၊ ဖျက် လုပ်နိုင်တယ်
* `Staff` က Product ကြည့်ရုံပဲ လုပ်နိုင်တယ်

ဆိုတဲ့ Rule တွေကို Application Run မလုပ်ခင်ကတည်းက သတ်မှတ်ထားတာကို **Static RBAC** လို့ခေါ်ပါတယ်။

```text
User
  │
  ▼
Role
  │
  ▼
Permissions
  │
  ▼
Protected Resource
```

---

# RBAC ဆိုတာဘာလဲ?

RBAC ရဲ့ အရှည်ကောက်က

```text
Role-Based Access Control
```

ဖြစ်ပါတယ်။

User တစ်ယောက်ချင်းစီကို Permission တစ်ခုချင်းစီ တိုက်ရိုက်လိုက်ပေးမယ့်အစား Role တစ်ခု သတ်မှတ်ပြီး အဲ့ဒီ Role ကတစ်ဆင့် Permission တွေ ရရှိစေပါတယ်။

ဥပမာ...

```text
Mg Mg ──► Admin ──► Product.View
                  ├─► Product.Create
                  ├─► Product.Update
                  └─► Product.Delete

Aye Aye ─► Staff ──► Product.View
```

User အရေအတွက်များလာတဲ့အခါ Permission တစ်ခုချင်းစီ လိုက်ပေးစရာမလိုတော့ဘဲ Role အလိုက် စီမံနိုင်ပါတယ်။

---

# Authentication နဲ့ Authorization မတူပါဘူး

RBAC က **Authorization** အပိုင်းဖြစ်ပါတယ်။

Authentication နဲ့ Authorization ကို မရောသင့်ပါဘူး။

## Authentication

```text
ဒီ User က ဘယ်သူလဲ?
```

ဆိုတာ စစ်တာဖြစ်ပါတယ်။

ဥပမာ...

* Username/Password စစ်ခြင်း
* JWT Token စစ်ခြင်း
* Authentication Cookie စစ်ခြင်း

## Authorization

```text
ဒီ User က ဒီအလုပ်ကို လုပ်ခွင့်ရှိလား?
```

ဆိုတာ စစ်တာဖြစ်ပါတယ်။

ဥပမာ...

* Product ကြည့်ခွင့်ရှိလား?
* Product ဖျက်ခွင့်ရှိလား?
* Admin Page ဝင်ခွင့်ရှိလား?

Flow အပြည့်အစုံကတော့

```text
Request
   │
   ▼
Authentication
User ဘယ်သူလဲ?
   │
   ▼
Authorization
ဘာလုပ်ခွင့်ရှိလဲ?
   │
   ▼
Controller Action
```

ဖြစ်ပါတယ်။

---

# Static ဆိုတာ ဘာကိုဆိုလိုတာလဲ?

Static ဆိုတာ Application ထဲက Authorization Rule တွေကို ကြိုတင်သတ်မှတ်ထားတာကို ဆိုလိုပါတယ်။

ဥပမာ...

```csharp
[Authorize(Roles = "Admin")]
```

သို့မဟုတ်

```csharp
options.AddPolicy("ProductDelete", policy =>
    policy.RequireClaim("permission", "Product.Delete"));
```

ဒီ Code တွေထဲမှာ Role Name, Policy Name နဲ့ Permission Name တွေကို ကြိုတင်ရေးထားပါတယ်။

Rule အသစ်ထည့်ချင်ရင် သို့မဟုတ် Endpoint တစ်ခုအတွက် လိုအပ်တဲ့ Permission ကိုပြောင်းချင်ရင် Code ပြင်ပြီး Application ကို ပြန် Deploy လုပ်ရနိုင်ပါတယ်။

ဒါက Dynamic RBAC နဲ့ အဓိကကွာခြားချက်ဖြစ်ပါတယ်။

---

# Static RBAC ရဲ့ အဓိကအစိတ်အပိုင်းများ

Static RBAC မှာ အများအားဖြင့် အောက်ပါအစိတ်အပိုင်းတွေ ပါဝင်ပါတယ်။

## 1. User

System ကို Login ဝင်မယ့်သူဖြစ်ပါတယ်။

ဥပမာ...

```text
admin
staff
```

## 2. Role

User ရဲ့ အလုပ်တာဝန် သို့မဟုတ် Access Level ဖြစ်ပါတယ်။

ဥပမာ...

```text
Admin
Manager
Staff
```

## 3. Permission

System ထဲမှာ လုပ်နိုင်တဲ့ Action တစ်ခုကို ကိုယ်စားပြုပါတယ်။

ဥပမာ...

```text
Product.View
Product.Create
Product.Update
Product.Delete
```

## 4. Claim

Authenticated User အကြောင်းကို JWT သို့မဟုတ် Cookie ထဲမှာ ထည့်သိမ်းထားတဲ့ အချက်အလက်ဖြစ်ပါတယ်။

ဥပမာ...

```text
name = admin
role = Admin
permission = Product.View
permission = Product.Create
```

## 5. Policy

Endpoint တစ်ခုကို ဝင်ခွင့်ရဖို့ ဖြည့်ဆည်းရမယ့် Rule ဖြစ်ပါတယ်။

ဥပမာ...

```text
ProductDelete Policy
    requires
Product.Delete Claim
```

---

# Permission Matrix ဥပမာ

Product Management System တစ်ခုမှာ Permission Matrix ကို ဒီလိုသတ်မှတ်ထားတယ်ဆိုပါစို့။

| Action | Admin | Staff |
|---|---:|---:|
| Product View | ✅ | ✅ |
| Product Create | ✅ | ❌ |
| Product Update | ✅ | ❌ |
| Product Delete | ✅ | ❌ |

ဒီ Matrix က Static Rule ဖြစ်ပါတယ်။

Application ထဲမှာ Admin နဲ့ Staff အတွက် Permission တွေကို ကြိုသတ်မှတ်ထားပါတယ်။

---

# ASP.NET Core မှာ Static RBAC တည်ဆောက်ပုံ

Repository ထဲက JWT နဲ့ MVC Static RBAC Example နှစ်ခုလုံးမှာ အခြေခံ Flow တူပါတယ်။

```text
Login
  │
  ▼
User Data ထဲက Role နဲ့ Permissions ယူ
  │
  ▼
Claims တည်ဆောက်
  │
  ├─► JWT ထဲထည့်
  │
  └─► Cookie ထဲထည့်
  │
  ▼
[Authorize] / Policy နဲ့ စစ်
```

---

# Step 1 - User Model ထဲမှာ Role နဲ့ Permissions သိမ်းခြင်း

သင်ခန်းစာ Example မှာ User တစ်ယောက်ရဲ့ Role နဲ့ Permissions ကို User Record ထဲမှာ တိုက်ရိုက်သိမ်းထားပါတယ်။

```csharp
public class AppUser
{
    public int Id { get; set; }
    public string Username { get; set; } = "";
    public string PasswordHash { get; set; } = "";
    public string Role { get; set; } = "";
    public List<string> Permissions { get; set; } = new();
}
```

Admin User အတွက်

```csharp
new AppUser
{
    Id = 1,
    Username = "admin",
    Role = "Admin",
    Permissions = new List<string>
    {
        "Product.View",
        "Product.Create",
        "Product.Update",
        "Product.Delete"
    }
}
```

Staff User အတွက်

```csharp
new AppUser
{
    Id = 2,
    Username = "staff",
    Role = "Staff",
    Permissions = new List<string>
    {
        "Product.View"
    }
}
```

ဒီပုံစံက Project အသေးနဲ့ Permission Rule မကြာခဏမပြောင်းတဲ့ System တွေအတွက် နားလည်ရလွယ်ပါတယ်။

---

# Step 2 - Authorization Policies သတ်မှတ်ခြင်း

`Program.cs` ထဲမှာ Permission တစ်ခုချင်းစီအတွက် Policy ကို ကြိုတင် Register လုပ်ပါတယ်။

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ProductView", policy =>
        policy.RequireClaim("permission", "Product.View"));

    options.AddPolicy("ProductCreate", policy =>
        policy.RequireClaim("permission", "Product.Create"));

    options.AddPolicy("ProductUpdate", policy =>
        policy.RequireClaim("permission", "Product.Update"));

    options.AddPolicy("ProductDelete", policy =>
        policy.RequireClaim("permission", "Product.Delete"));
});
```

ဥပမာ `ProductDelete` Policy က လက်ရှိ User မှာ

```text
permission = Product.Delete
```

Claim ရှိမှ Pass ဖြစ်ပါတယ်။

Policy Name နဲ့ လိုအပ်တဲ့ Permission ကို Code ထဲမှာ ကြိုတင်သတ်မှတ်ထားတဲ့အတွက် Static ဖြစ်ပါတယ်။

---

# Step 3 - Login အချိန်မှာ Claims တည်ဆောက်ခြင်း

User ရဲ့ Username, Role နဲ့ Permissions ကို Claims အဖြစ် ပြောင်းပါတယ်။

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Name, user.Username),
    new Claim(ClaimTypes.Role, user.Role)
};

foreach (var permission in user.Permissions)
{
    claims.Add(new Claim("permission", permission));
}
```

Admin အတွက် Claims က အကြမ်းဖျဉ်း ဒီလိုဖြစ်ပါတယ်။

```text
NameIdentifier = 1
Name           = admin
Role           = Admin
Permission     = Product.View
Permission     = Product.Create
Permission     = Product.Update
Permission     = Product.Delete
```

---

# Step 4A - JWT Web API မှာ Claims ထည့်ခြင်း

Web API မှာ Claims တွေကို JWT Access Token ထဲ ထည့်ပေးပါတယ်။

```text
Client Login
   │
   ▼
Server validates credentials
   │
   ▼
Server creates JWT with role and permission claims
   │
   ▼
Client receives access token
```

နောက် Request တွေမှာ Client က Token ကို Header ထဲကနေ ပို့ပါတယ်။

```http
Authorization: Bearer <access-token>
```

JWT Middleware က

* Signature မှန်လား
* Issuer မှန်လား
* Audience မှန်လား
* Token Expire ဖြစ်ပြီးပြီလား

ဆိုတာတွေ စစ်ပြီး `HttpContext.User` ကို Claims တွေနဲ့ တည်ဆောက်ပေးပါတယ်။

> JWT က ပုံမှန်အားဖြင့် Encrypt လုပ်ထားတာမဟုတ်ပါဘူး။ Payload ကို ဖတ်နိုင်တာကြောင့် Password နဲ့ Secret Data တွေ မထည့်သင့်ပါဘူး။ Signature က Token ကို ပြင်ထားခြင်းရှိမရှိ စစ်ပေးတာဖြစ်ပါတယ်။

---

# Step 4B - MVC Cookie Authentication မှာ Claims ထည့်ခြင်း

ASP.NET Core MVC မှာ Claims တွေကို `ClaimsPrincipal` အဖြစ်တည်ဆောက်ပြီး Sign In လုပ်ပါတယ်။

```csharp
var identity = new ClaimsIdentity(
    claims,
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(identity));
```

ASP.NET Core က Authentication Ticket ကို Data Protection နဲ့ ကာကွယ်ပြီး Cookie အဖြစ် Browser ကို ပြန်ပေးပါတယ်။

နောက် Request တွေမှာ Browser က Cookie ကို အလိုအလျောက် ပို့ပေးပါတယ်။

```text
Browser Request
   │
   ▼
Authentication Cookie
   │
   ▼
Cookie Middleware
   │
   ▼
HttpContext.User
```

---

# Step 5 - Controller မှာ Role နဲ့ စစ်ခြင်း

Admin Role ပဲ ဝင်ခွင့်ပေးချင်ရင်

```csharp
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly()
{
    return Ok();
}
```

Admin သို့မဟုတ် Manager တစ်ခုခုဖြစ်ရင် ဝင်ခွင့်ပေးချင်ရင် Role တွေကို Comma နဲ့ ခွဲရေးနိုင်ပါတယ်။

```csharp
[Authorize(Roles = "Admin,Manager")]
```

ဒီ Code ရဲ့ အဓိပ္ပာယ်က

```text
Admin OR Manager
```

ဖြစ်ပါတယ်။

`[Authorize]` Attribute အများကြီး သီးခြားတပ်ထားရင်တော့ Rule အားလုံး Pass ဖြစ်ရပါတယ်။

---

# Step 6 - Controller မှာ Permission Policy နဲ့ စစ်ခြင်း

Role တစ်ခုလုံးကို စစ်တာထက် Action တစ်ခုအတွက် လိုအပ်တဲ့ Permission ကို တိတိကျကျစစ်နိုင်ပါတယ်။

```csharp
[HttpGet]
[Authorize(Policy = "ProductView")]
public IActionResult GetProducts()
{
    return Ok();
}
```

```csharp
[HttpPost]
[Authorize(Policy = "ProductCreate")]
public IActionResult CreateProduct(ProductCreateRequest request)
{
    return Ok();
}
```

```csharp
[HttpDelete("{id}")]
[Authorize(Policy = "ProductDelete")]
public IActionResult DeleteProduct(int id)
{
    return Ok();
}
```

ဒီလိုရေးထားရင် Endpoint ကိုခေါ်တဲ့ User မှာ သက်ဆိုင်ရာ Permission Claim ရှိမှ Action ထဲရောက်ပါတယ်။

---

# Role Check နဲ့ Permission Check ဘယ်ဟာသုံးသင့်လဲ?

Role Check က ရိုးရှင်းပါတယ်။

```csharp
[Authorize(Roles = "Admin")]
```

ဒါပေမယ့် Business Feature များလာတဲ့အခါ Role Name နဲ့ Controller Action တွေ တင်းတင်းကျပ်ကျပ် ချိတ်မိသွားနိုင်ပါတယ်။

Permission Check က ပိုတိကျပါတယ်။

```csharp
[Authorize(Policy = "ProductDelete")]
```

ဒီ Action က ဘယ် Role ဖြစ်ရမယ်ဆိုတာထက် ဘာ Permission လိုတယ်ဆိုတာကို ဖော်ပြပါတယ်။

အများအားဖြင့်

* Admin-only Page လို Role ကိုယ်တိုင် အဓိပ္ပာယ်ရှိတဲ့နေရာမှာ Role Check
* CRUD Action လို လုပ်ပိုင်ခွင့်ကို တိတိကျကျခွဲချင်တဲ့နေရာမှာ Permission Policy

သုံးနိုင်ပါတယ်။

---

# MVC View မှာ Menu နဲ့ Button ဖျောက်ခြင်း

MVC View မှာ Role ကိုစစ်ပြီး Menu ဖော်ပြနိုင်ပါတယ်။

```cshtml
@if (User.IsInRole("Admin"))
{
    <a asp-controller="Product" asp-action="AdminOnly">
        Admin Only
    </a>
}
```

Permission Claim ကိုလည်း စစ်နိုင်ပါတယ်။

```cshtml
@if (User.HasClaim("permission", "Product.Create"))
{
    <a asp-controller="Product" asp-action="Create">
        Create Product
    </a>
}
```

ဒါပေမယ့် Button ဖျောက်ထားတာက Security မဟုတ်ပါဘူး။

User က URL သို့မဟုတ် HTTP Request ကို တိုက်ရိုက်ပို့နိုင်တဲ့အတွက် Controller သို့မဟုတ် Endpoint မှာ `[Authorize]` Rule မဖြစ်မနေထားရပါတယ်။

```text
UI Check       = User Experience
Endpoint Check = Security Boundary
```

---

# Product Delete Request Flow

Admin က Product တစ်ခုဖျက်တယ်ဆိုပါစို့။

```text
DELETE /api/product/10
          │
          ▼
JWT/Cookie Authentication
          │
          ▼
HttpContext.User created
          │
          ▼
ProductDelete Policy
          │
          ▼
Has Product.Delete claim?
       │          │
      Yes         No
       │          │
       ▼          ▼
Controller      403 Forbidden
       │
       ▼
Product deleted
```

Staff မှာ `Product.Delete` Claim မရှိတဲ့အတွက် Controller Action ထဲ မရောက်ဘဲ `403 Forbidden` ဖြစ်ပါတယ်။

---

# 401 Unauthorized နဲ့ 403 Forbidden

ဒီ HTTP Status Code နှစ်ခုကို ခွဲနားလည်ဖို့ အရေးကြီးပါတယ်။

## 401 Unauthorized

User ကို Authentication မလုပ်နိုင်တဲ့အခါ ဖြစ်ပါတယ်။

ဥပမာ...

* JWT မပါခြင်း
* JWT Expire ဖြစ်ခြင်း
* JWT Signature မမှန်ခြင်း
* Login မဝင်ထားခြင်း

အဓိပ္ပာယ်က

```text
ဒီ Request ကို ဘယ်သူပို့တာလဲ မသိဘူး
```

ဖြစ်ပါတယ်။

## 403 Forbidden

User ကို သိတယ်၊ Login လည်းဝင်ထားတယ်၊ ဒါပေမယ့် လိုအပ်တဲ့ Role သို့မဟုတ် Permission မရှိတဲ့အခါ ဖြစ်ပါတယ်။

အဓိပ္ပာယ်က

```text
ဒီ User ကို သိတယ်၊ ဒါပေမယ့် ဒီအလုပ်လုပ်ခွင့်မရှိဘူး
```

ဖြစ်ပါတယ်။

MVC Cookie Authentication မှာတော့ 401/403 ကို Login Page နဲ့ Access Denied Page ဆီ Redirect လုပ်ပေးလေ့ရှိပါတယ်။

---

# Static RBAC မှာ Permission ပြောင်းရင် ဘယ်အချိန် Effect ဖြစ်မလဲ?

Role နဲ့ Permission Claims တွေကို Login အချိန်မှာ JWT သို့မဟုတ် Cookie ထဲ ထည့်ထားပါတယ်။

ဒါကြောင့် Database ထဲက User Permission ကို ပြောင်းလိုက်ရုံနဲ့ လက်ရှိ Token သို့မဟုတ် Cookie ထဲက Claim က ချက်ချင်းမပြောင်းပါဘူး။

User က

* ပြန် Login ဝင်ခြင်း
* Token Refresh လုပ်ခြင်း
* Token သို့မဟုတ် Cookie Expire ဖြစ်ပြီး အသစ်ရယူခြင်း

လုပ်မှ Claim အသစ်ရနိုင်ပါတယ်။

Authorization Policy ကို Code ထဲမှာ ပြောင်းရင်တော့ Application ကို ပြန် Build/Deploy လုပ်ရပါတယ်။

---

# Real World Example - Inventory System

Inventory System တစ်ခုမှာ Role နှစ်ခုရှိတယ်ဆိုပါစို့။

## Admin

* Stock ကြည့်နိုင်တယ်
* Item အသစ်ထည့်နိုင်တယ်
* Item ပြင်နိုင်တယ်
* Item ဖျက်နိုင်တယ်

## Staff

* Stock ကြည့်နိုင်တယ်
* Item အသစ်ထည့်မရဘူး
* Item ပြင်မရဘူး
* Item ဖျက်မရဘူး

Staff က Delete Button ကို မမြင်ရပေမယ့် API ကို တိုက်ရိုက်ခေါ်တယ်ဆိုပါစို့။

```http
DELETE /api/products/10
```

Endpoint မှာ

```csharp
[Authorize(Policy = "ProductDelete")]
```

ရှိတဲ့အတွက် Staff ရဲ့ Request ကို `403 Forbidden` နဲ့ တားပေးပါတယ်။

ဒါကြောင့် RBAC ကို UI မှာတင်မဟုတ်ဘဲ Server-side Endpoint မှာ စစ်ရတာဖြစ်ပါတယ်။

---

# Static RBAC ရဲ့ အားသာချက်များ

* **နားလည်ရလွယ်တယ်** – Role နဲ့ Policy Rule တွေကို Code ထဲမှာ တိုက်ရိုက်မြင်နိုင်ပါတယ်။
* **တည်ဆောက်ရမြန်တယ်** – Role/Permission Management Tables နဲ့ Admin UI အများကြီး မလိုပါဘူး။
* **Request တိုင်း Database မစစ်ရဘူး** – JWT သို့မဟုတ် Cookie ထဲက Claims ကို စစ်ရုံဖြစ်ပါတယ်။
* **Project အသေးအတွက် လုံလောက်တယ်** – Role နည်းပြီး Rule မပြောင်းတဲ့ Internal System တွေအတွက် သင့်တော်ပါတယ်။
* **စမ်းသပ်ရလွယ်တယ်** – Role နဲ့ Permission Matrix က တည်ငြိမ်တာကြောင့် Test Case ရေးရလွယ်ပါတယ်။

---

# Static RBAC ရဲ့ အားနည်းချက်များ

* Permission Rule ပြောင်းရင် Code ပြင်ပြီး Deploy ပြန်လုပ်ရနိုင်ပါတယ်။
* Role နဲ့ Permission များလာရင် `Program.cs` နဲ့ Controller Attributes တွေ များလာနိုင်ပါတယ်။
* User တစ်ယောက်ချင်းစီရဲ့ Permission List ထပ်နေတာကြောင့် Data ထိန်းသိမ်းရခက်နိုင်ပါတယ်။
* လက်ရှိ JWT/Cookie ထဲက Permission ကို ချက်ချင်းရုပ်သိမ်းဖို့ မလွယ်ပါဘူး။
* Admin User က UI ကနေ Role-Permission Mapping ကို လွယ်လွယ်ပြောင်းနိုင်တဲ့စနစ် မဟုတ်ပါဘူး။

---

# ဘယ်လို Project တွေမှာ Static RBAC သုံးသင့်လဲ?

Static RBAC ကို အောက်ပါအခြေအနေတွေမှာ သုံးနိုင်ပါတယ်။

* Role နှစ်ခု၊ သုံးခုလောက်ပဲရှိတဲ့ Project
* Permission Rule မကြာခဏမပြောင်းတဲ့ Internal Application
* Prototype သို့မဟုတ် Training Project
* Admin UI ကနေ Permission စီမံဖို့ မလိုသေးတဲ့ System
* Request Performance အတွက် Database Authorization Query မလိုချင်တဲ့ System

Permission ကို Admin က Runtime မှာ ပြောင်းချင်တယ်၊ Role များတယ်၊ Module များတယ်ဆိုရင်တော့ [Dynamic RBAC](Dynamic%20RBAC.md) ကို စဉ်းစားသင့်ပါတယ်။

---

# Security Best Practices

Static RBAC ရိုးရှင်းတယ်ဆိုပေမယ့် Security အပိုင်းကို မလျှော့သင့်ပါဘူး။

* Password ကို Plain Text မသိမ်းဘဲ Password Hasher သုံးပါ။
* JWT Signing Key ကို Source Code ထဲ မရေးဘဲ Secret Store သို့မဟုတ် Environment Variable ထဲထားပါ။
* JWT မှာ Password, Secret, Sensitive Data မထည့်ပါနဲ့။
* Production မှာ HTTPS မဖြစ်မနေသုံးပါ။
* Cookie မှာ `HttpOnly`, `Secure`, `SameSite` Setting တွေ မှန်ကန်စွာထားပါ။
* Token/Cookie Expiration ကို အလွန်မရှည်စေပါနဲ့။
* Permission မရှိရင် Default အနေနဲ့ Deny လုပ်ပါ။
* User ကို လိုအပ်သလောက် Permission ပဲပေးတဲ့ **Least Privilege** Principle ကိုလိုက်နာပါ။
* UI Button ဖျောက်ထားတာကို Authorization အဖြစ် မယုံပါနဲ့။
* Authorization Failure တွေကို Audit Log မှာ မှတ်တမ်းတင်ပါ။

Repository ထဲက Plain Password နဲ့ In-Memory Database တွေက သင်ခန်းစာ Flow ကို ရှင်းပြဖို့သာ ဖြစ်ပြီး Production Pattern မဟုတ်ပါဘူး။

---

# စမ်းသပ်သင့်တဲ့ Test Cases

Static RBAC အတွက် အနည်းဆုံး အောက်ပါ Test Cases တွေ စမ်းသင့်ပါတယ်။

| User State | Required Permission | Expected Result |
|---|---|---|
| Login မဝင်ထား | Product.View | 401 / Login Redirect |
| Admin | Product.View | Allowed |
| Admin | Product.Delete | Allowed |
| Staff | Product.View | Allowed |
| Staff | Product.Delete | 403 / Access Denied |
| Expired Token | Product.View | 401 |
| Invalid Token | Product.View | 401 |

Role-only Endpoint ကိုလည်း သီးခြားစမ်းသင့်ပါတယ်။

```text
Admin  → AdminOnly → Allowed
Staff  → AdminOnly → Forbidden
```

---

# Summary

**Static RBAC** ဆိုတာ User တစ်ယောက်ရဲ့ Access ကို ကြိုတင်သတ်မှတ်ထားတဲ့ **Role**, **Permission**, **Claim** နဲ့ **Policy** တွေကို အသုံးပြုပြီး ထိန်းချုပ်တဲ့ Authorization ပုံစံဖြစ်ပါတယ်။

ASP.NET Core မှာ Role ကို `ClaimTypes.Role`, Permission ကို Custom Claim အဖြစ် ထည့်ပြီး `[Authorize(Roles = "...")]` သို့မဟုတ် `[Authorize(Policy = "...")]` နဲ့ Endpoint တွေကို ကာကွယ်နိုင်ပါတယ်။ JWT Web API မှာ Claims တွေ Token ထဲပါသွားပြီး MVC Cookie Authentication မှာတော့ Protected Authentication Cookie ထဲပါသွားပါတယ်။

Static RBAC က ရိုးရှင်းပြီး မြန်ဆန်ပေမယ့် Permission Rule မကြာခဏပြောင်းတဲ့ System၊ Admin UI ကနေ Access Control စီမံရတဲ့ System နဲ့ Permission အများကြီးရှိတဲ့ Enterprise Project တွေအတွက်တော့ Dynamic RBAC က ပိုသင့်တော်နိုင်ပါတယ်။

---

# References

* [Introduction to authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authorization/introduction)
* [Role-based authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/security/authorization/roles)
* [Policy-based authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authorization/policies)
