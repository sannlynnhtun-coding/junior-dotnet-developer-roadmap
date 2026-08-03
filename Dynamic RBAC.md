# Dynamic RBAC ဆိုတာဘာလဲ?

**Dynamic RBAC (Dynamic Role-Based Access Control)** ဆိုတာ User, Role နဲ့ Permission တွေရဲ့ ဆက်နွယ်မှုကို Database ထဲမှာ စီမံပြီး Application ကို Code ပြင်စရာ၊ Deploy ပြန်လုပ်စရာမလိုဘဲ Role တစ်ခုရဲ့ လုပ်ပိုင်ခွင့်တွေကို ပြောင်းနိုင်တဲ့ Authorization ပုံစံဖြစ်ပါတယ်။

ဥပမာ...

`Staff` Role က အစမှာ

```text
Product.View
```

Permission ပဲရှိတယ်ဆိုပါစို့။

နောက်ပိုင်း `Staff` ကို Product အသစ်ထည့်ခွင့် ပေးချင်ရင် Database ထဲက Role-Permission Mapping မှာ

```text
Staff ──► Product.Create
```

ကို ထပ်ထည့်နိုင်ပါတယ်။

Controller Code ကို ပြင်စရာမလိုပါဘူး။

```text
User
  │
  ▼
Role
  │
  ▼
RolePermission
  │
  ▼
Permission
  │
  ▼
Protected Resource
```

---

# Dynamic ဆိုတာ ဘာကိုဆိုလိုတာလဲ?

Dynamic ဆိုတာ Endpoint မှာ လိုအပ်တဲ့ Permission Name လုံးဝမရေးရတော့တာကို မဆိုလိုပါဘူး။

ဥပမာ Product Delete Endpoint မှာ

```csharp
[Permission("Product.Delete")]
```

လို့ ရေးထားနိုင်ပါတယ်။

Dynamic ဖြစ်တဲ့အပိုင်းက

```text
ဘယ် Role မှာ Product.Delete Permission ရှိသလဲ?
```

ဆိုတဲ့ Mapping ကို Code ထဲမှာ Hard-code မလုပ်ဘဲ Database ထဲကနေ ဆုံးဖြတ်တာဖြစ်ပါတယ်။

```text
Endpoint requirement = Product.Delete

Admin has Product.Delete?   Database ကဆုံးဖြတ်
Staff has Product.Delete?   Database ကဆုံးဖြတ်
Editor has Product.Delete?  Database ကဆုံးဖြတ်
```

---

# Static RBAC နဲ့ Dynamic RBAC ကွာခြားချက်

| အချက် | Static RBAC | Dynamic RBAC |
|---|---|---|
| Role-Permission Mapping | Code/User Record ထဲမှာ ကြိုသတ်မှတ် | Database Table ထဲမှာ စီမံ |
| Permission ပြောင်းခြင်း | Code/Data ပြင်ပြီး Re-login သို့မဟုတ် Deploy လိုနိုင် | Mapping ပြင်ပြီး Claim Refresh သို့မဟုတ် Live Check ဖြင့် အသုံးပြုနိုင် |
| Database Structure | ရိုးရှင်း | Roles, Permissions, Mapping Tables လို |
| Admin UI | မလိုအပ်လေ့ရှိ | Role/Permission Management UI ရေးနိုင် |
| Request Cost | Claims စစ်ရုံ | Live Check ဆို Database Query လိုနိုင် |
| သင့်တော်တဲ့ Project | Role နည်းပြီး Rule တည်ငြိမ် | Permission များပြီး Rule မကြာခဏပြောင်း |

[Static RBAC](Static%20RBAC.md) က စတင်တည်ဆောက်ရလွယ်ပါတယ်။ Dynamic RBAC က စီမံခန့်ခွဲရပိုလွယ်ပေမယ့် Database Design နဲ့ Authorization Flow ပိုများပါတယ်။

---

# Authentication နဲ့ Authorization

Dynamic RBAC က **Authorization** အပိုင်းဖြစ်ပါတယ်။

## Authentication

User ဘယ်သူလဲဆိုတာ စစ်ပါတယ်။

```text
Username + Password
JWT
Cookie
```

## Authorization

Authenticated User က လက်ရှိ Request ကို လုပ်ခွင့်ရှိမရှိ စစ်ပါတယ်။

```text
User ရဲ့ Role ကဘာလဲ?
Role မှာ ဘာ Permissions ရှိလဲ?
Endpoint က ဘာ Permission လိုလဲ?
```

Dynamic RBAC Authorization Flow ကတော့

```text
Authenticated User
       │
       ▼
Find current Role
       │
       ▼
Find Role Permissions
       │
       ▼
Compare required Permission
       │
   ┌───┴───┐
   ▼       ▼
 Allow    Deny
```

ဖြစ်ပါတယ်။

---

# Dynamic RBAC Database Design

Repository ထဲက Dynamic RBAC Example တွေမှာ Table လေးခုသုံးထားပါတယ်။

1. `TblUsers`
2. `TblRoles`
3. `TblPermissions`
4. `TblRolePermissions`

Relationship ကတော့

```text
TblUsers
   │ RoleId
   ▼
TblRoles
   │ RoleId
   ▼
TblRolePermissions
   │ PermissionId
   ▼
TblPermissions
```

ဖြစ်ပါတယ်။

---

# 1. Users Table

User ရဲ့ Identity နဲ့ Role ကို သိမ်းပါတယ်။

| Id | Username | RoleId |
|---:|---|---:|
| 1 | admin | 1 |
| 2 | staff | 2 |

Entity Model က

```csharp
public class AppUser
{
    public int Id { get; set; }
    public string Username { get; set; } = "";
    public string PasswordHash { get; set; } = "";
    public int RoleId { get; set; }
}
```

ဖြစ်ပါတယ်။

ဒီ Example မှာ User တစ်ယောက်ကို Role တစ်ခုပဲပေးထားပါတယ်။ User တစ်ယောက်မှာ Role အများကြီးလိုရင် `UserRoles` Mapping Table ထပ်ခွဲသင့်ပါတယ်။

---

# 2. Roles Table

System ထဲက Role တွေကို သိမ်းပါတယ်။

| Id | RoleName |
|---:|---|
| 1 | Admin |
| 2 | Staff |

```csharp
public class AppRole
{
    public int Id { get; set; }
    public string RoleName { get; set; } = "";
}
```

Role Name ကို User Record တိုင်းမှာ ထပ်သိမ်းမယ့်အစား Role Table တစ်ခုထဲမှာ စုစည်းထားပါတယ်။

---

# 3. Permissions Table

Application ထဲမှာ လုပ်နိုင်တဲ့ Action တွေကို သိမ်းပါတယ်။

| Id | PermissionName |
|---:|---|
| 1 | Product.View |
| 2 | Product.Create |
| 3 | Product.Update |
| 4 | Product.Delete |

```csharp
public class AppPermission
{
    public int Id { get; set; }
    public string PermissionName { get; set; } = "";
}
```

Permission Name ကို

```text
Resource.Action
```

ပုံစံနဲ့ပေးထားရင် နားလည်ရလွယ်ပါတယ်။

ဥပမာ...

```text
Product.View
Product.Create
Order.Approve
User.Disable
Report.Export
```

---

# 4. RolePermissions Table

Role တစ်ခုနဲ့ Permission တစ်ခုကို ချိတ်ပေးတဲ့ Mapping Table ဖြစ်ပါတယ်။

| Id | RoleId | PermissionId |
|---:|---:|---:|
| 1 | 1 | 1 |
| 2 | 1 | 2 |
| 3 | 1 | 3 |
| 4 | 1 | 4 |
| 5 | 2 | 1 |

အဓိပ္ပာယ်က

```text
Admin ──► Product.View
      ├─► Product.Create
      ├─► Product.Update
      └─► Product.Delete

Staff ──► Product.View
```

ဖြစ်ပါတယ်။

```csharp
public class AppRolePermission
{
    public int Id { get; set; }
    public int RoleId { get; set; }
    public int PermissionId { get; set; }
}
```

Production Database မှာ `(RoleId, PermissionId)` ကို Unique ဖြစ်အောင် Constraint ထားသင့်ပါတယ်။ ဒါမှ Permission Mapping တူတာ နှစ်ကြိမ်ထည့်မိခြင်းကို Database Level မှာ တားနိုင်ပါတယ်။

---

# DbContext

Entity Framework Core သုံးရင် `DbContext` ထဲမှာ Table လေးခု Register လုပ်ပါတယ်။

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<AppUser> TblUsers { get; set; }
    public DbSet<AppRole> TblRoles { get; set; }
    public DbSet<AppPermission> TblPermissions { get; set; }
    public DbSet<AppRolePermission> TblRolePermissions { get; set; }
}
```

Database ထဲမှာ Foreign Key, Unique Index နဲ့ Required Column တွေကိုလည်း သတ်မှတ်သင့်ပါတယ်။

---

# Dynamic RBAC Flow နှစ်မျိုး

Dynamic RBAC မှာ Permission ပြောင်းလဲမှု ဘယ်လောက်မြန်မြန် Effect ဖြစ်စေချင်သလဲဆိုတာပေါ်မူတည်ပြီး Flow နှစ်မျိုးအသုံးများပါတယ်။

## 1. Login-time Claim Snapshot

Login အချိန်မှာ Database က Role နဲ့ Permissions ယူပြီး JWT သို့မဟုတ် Cookie ထဲ ထည့်ပါတယ်။

နောက် Request တွေမှာ Claim ကိုပဲ စစ်ပါတယ်။

```text
Login
  │
  ▼
Load permissions from database
  │
  ▼
Put permissions into JWT/Cookie
  │
  ▼
Later requests check claims only
```

အားသာချက်က Request တိုင်း Database Query မလိုတာဖြစ်ပါတယ်။

အားနည်းချက်က Database ထဲမှာ Permission ပြောင်းလိုက်ရင် လက်ရှိ JWT/Cookie ထဲက Claim က မပြောင်းသေးတာဖြစ်ပါတယ်။ User က Re-login, Token Refresh သို့မဟုတ် Session Renewal လုပ်မှ Permission အသစ်ရပါတယ်။

## 2. Live Database Check

Protected Request တစ်ခုဝင်လာတိုင်း လက်ရှိ User Role နဲ့ Permission Mapping ကို Database မှာ စစ်ပါတယ်။

```text
Every protected request
       │
       ▼
Read current user role
       │
       ▼
Read current role permissions
       │
       ▼
Allow or deny immediately
```

အားသာချက်က Admin က Permission ပြောင်းလိုက်တာနဲ့ နောက် Request ကစပြီး Effect ဖြစ်နိုင်တာပါ။

အားနည်းချက်က Protected Request တိုင်း Database Query ပိုဝင်နိုင်တာပါ။

---

# ဘယ် Flow ကို သုံးသင့်လဲ?

လိုအပ်ချက်ပေါ်မူတည်ပါတယ်။

| လိုအပ်ချက် | သင့်တော်တဲ့ Flow |
|---|---|
| Request Performance အရေးကြီး | Login-time Claim Snapshot |
| Permission ချက်ချင်းရုပ်သိမ်းရမယ် | Live Database Check |
| JWT Access Token သက်တမ်းတို | Claim Snapshot လုံလောက်နိုင် |
| Banking/Admin Security မြင့် | Live Check သို့မဟုတ် Revocation Strategy |
| Traffic များပြီး Permission အပြောင်းအလဲနည်း | Claim Snapshot သို့မဟုတ် Cache |

တချို့ System တွေမှာ Hybrid ပုံစံသုံးပါတယ်။

```text
Database is source of truth
          │
          ▼
Short-lived cache
          │
          ▼
Authorization check
```

Permission ပြောင်းတဲ့အခါ Cache ကို Invalidate လုပ်နိုင်ပါတယ်။

---

# Step 1 - Login အချိန်မှာ User နဲ့ Role ရှာခြင်း

User Login ဝင်လာတဲ့အခါ Credentials စစ်ပြီး `RoleId` ကတစ်ဆင့် Role Name ကိုယူပါတယ်။

```csharp
var user = await _context.TblUsers.FirstOrDefaultAsync(x =>
    x.Username == request.Username);

if (user is null)
    return null;

var role = await _context.TblRoles
    .FirstOrDefaultAsync(x => x.Id == user.RoleId);
```

Production Code မှာ Password Plain Text မနှိုင်းဘဲ Password Hasher နဲ့ Verify လုပ်ရပါတယ်။

---

# Step 2 - Role ရဲ့ Permissions ကို Database က ယူခြင်း

`TblRolePermissions` နဲ့ `TblPermissions` ကို Join ပြီး User Role မှာရှိတဲ့ Permission Name တွေယူပါတယ်။

```csharp
var permissions = await (
    from rolePermission in _context.TblRolePermissions
    join permission in _context.TblPermissions
        on rolePermission.PermissionId equals permission.Id
    where rolePermission.RoleId == user.RoleId
    select permission.PermissionName
).ToListAsync();
```

Admin အတွက် Result က

```text
Product.View
Product.Create
Product.Update
Product.Delete
```

ဖြစ်နိုင်ပြီး Staff အတွက်

```text
Product.View
```

ပဲဖြစ်နိုင်ပါတယ်။

---

# Step 3 - Claims တည်ဆောက်ခြင်း

User Identity, Role နဲ့ Permission တွေကို Claims အဖြစ် တည်ဆောက်ပါတယ်။

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Name, user.Username),
    new Claim(ClaimTypes.Role, role.RoleName)
};

foreach (var permission in permissions)
{
    claims.Add(new Claim("permission", permission));
}
```

JWT Web API မှာ ဒီ Claims တွေကို Access Token ထဲထည့်ပါတယ်။

MVC Cookie Authentication မှာတော့ ဒီ Claims တွေကို `ClaimsPrincipal` ထဲထည့်ပြီး Authentication Cookie ထုတ်ပေးပါတယ်။

---

# Step 4 - Permission Attribute တည်ဆောက်ခြင်း

Repository ထဲက Dynamic RBAC Example က Endpoint တစ်ခုအတွက် လိုအပ်တဲ့ Permission ကို Custom Attribute နဲ့ သတ်မှတ်ထားပါတယ်။

```csharp
public class PermissionAttribute : TypeFilterAttribute
{
    public PermissionAttribute(string permission)
        : base(typeof(PermissionFilter))
    {
        Arguments = new object[] { permission };
    }
}
```

ဒါဆို Controller မှာ

```csharp
[Permission("Product.Delete")]
```

လို့ ရိုးရှင်းစွာ အသုံးပြုနိုင်ပါတယ်။

---

# Step 5 - Database က Permission စစ်ခြင်း

`PermissionFilter` က လက်ရှိ User ID ကို Claim ကနေယူပြီး Database ထဲက လက်ရှိ Role-Permission Mapping ကို စစ်ပါတယ်။

```csharp
public class PermissionFilter : IAsyncAuthorizationFilter
{
    private readonly string _permission;
    private readonly AppDbContext _context;

    public PermissionFilter(
        string permission,
        AppDbContext context)
    {
        _permission = permission;
        _context = context;
    }

    public async Task OnAuthorizationAsync(
        AuthorizationFilterContext context)
    {
        if (context.HttpContext.User.Identity?.IsAuthenticated != true)
        {
            context.Result = new ChallengeResult();
            return;
        }

        var userIdValue = context.HttpContext.User
            .FindFirstValue(ClaimTypes.NameIdentifier);

        if (!int.TryParse(userIdValue, out var userId))
        {
            context.Result = new ForbidResult();
            return;
        }

        var roleId = await _context.TblUsers
            .Where(x => x.Id == userId)
            .Select(x => (int?)x.RoleId)
            .FirstOrDefaultAsync();

        var hasPermission = roleId.HasValue &&
            await (
                from rolePermission in _context.TblRolePermissions
                join permission in _context.TblPermissions
                    on rolePermission.PermissionId equals permission.Id
                where rolePermission.RoleId == roleId.Value
                    && permission.PermissionName == _permission
                select rolePermission.Id
            ).AnyAsync();

        if (!hasPermission)
        {
            context.Result = new ForbidResult();
        }
    }
}
```

ဒီ Filter က Claim ထဲက Permission List ကို မယုံဘဲ Database ကို Live Check လုပ်တာကြောင့် Permission ပြောင်းလဲမှု ချက်ချင်း Effect ဖြစ်နိုင်ပါတယ်။

---

# Step 6 - Controller Actions ကို ကာကွယ်ခြင်း

Product Controller မှာ Action တစ်ခုချင်းစီအတွက် လိုအပ်တဲ့ Permission ကို သတ်မှတ်နိုင်ပါတယ်။

```csharp
[Authorize]
public class ProductController : Controller
{
    [HttpGet]
    [Permission("Product.View")]
    public IActionResult Index()
    {
        return View();
    }

    [HttpPost]
    [Permission("Product.Create")]
    public IActionResult Create(ProductCreateRequest request)
    {
        return RedirectToAction("Index");
    }

    [HttpPost]
    [Permission("Product.Delete")]
    public IActionResult Delete(int id)
    {
        return RedirectToAction("Index");
    }
}
```

Web API Controller မှာလည်း ဒီ Attribute ကို အတူတူသုံးနိုင်ပါတယ်။

```csharp
[HttpDelete("{id}")]
[Permission("Product.Delete")]
public IActionResult DeleteProduct(int id)
{
    return Ok();
}
```

---

# JWT Dynamic RBAC Flow

JWT Web API မှာ Flow ကို နှစ်မျိုးစီမံနိုင်ပါတယ်။

## Claim Snapshot နဲ့ စစ်ခြင်း

```text
Login
  │
  ▼
Database permissions loaded
  │
  ▼
JWT created with permission claims
  │
  ▼
Request checks JWT claims
```

Code က

```csharp
var hasPermission = context.HttpContext.User.HasClaim(
    "permission",
    _permission);
```

လိုမျိုးဖြစ်ပါတယ်။

ဒီပုံစံက မြန်ပေမယ့် Permission ပြောင်းလဲမှုက လက်ရှိ Token သက်တမ်းကုန်သည်အထိ မသက်ရောက်နိုင်ပါဘူး။

## Database Live Check နဲ့ စစ်ခြင်း

```text
JWT identifies user
       │
       ▼
Current user role loaded from DB
       │
       ▼
Current permission mapping checked
       │
       ▼
Allow or deny
```

ဒီပုံစံက Permission ရုပ်သိမ်းမှု မြန်ပေမယ့် Request တိုင်း Query ဝင်နိုင်ပါတယ်။

---

# MVC Cookie Dynamic RBAC Flow

MVC မှာ Login အချိန် Role နဲ့ Permissions ကို Cookie ထဲ ထည့်ထားနိုင်ပါတယ်။

```text
Login Form
   │
   ▼
Database role and permissions
   │
   ▼
ClaimsPrincipal
   │
   ▼
Protected authentication cookie
```

UI Button ဖော်ပြဖို့ Cookie Claim ကိုသုံးနိုင်ပါတယ်။

```cshtml
@if (User.HasClaim("permission", "Product.Create"))
{
    <a asp-action="Create">Create Product</a>
}
```

Security Decision ကိုတော့ Live Database Filter နဲ့ ပြန်စစ်နိုင်ပါတယ်။

```text
Cookie Claim = UI rendering အတွက်မြန်
Database Check = Endpoint authorization အတွက်လက်ရှိအခြေအနေ
```

Admin က Permission ရုပ်သိမ်းပြီးနောက် Button က Cookie Expire/Re-login မဖြစ်မချင်း မြင်နေရနိုင်ပေမယ့် Endpoint က Live Check လုပ်ထားရင် Action ကိုတော့ ဝင်လို့မရပါဘူး။

---

# Role-only Check ရဲ့ သတိပြုရန်အချက်

Dynamic Permission Filter သုံးထားပေမယ့် Endpoint တစ်ခုမှာ

```csharp
[Authorize(Roles = "Admin")]
```

ကို သုံးထားရင် Role Check က JWT သို့မဟုတ် Cookie ထဲက Role Claim ကို စစ်တာဖြစ်ပါတယ်။

Database ထဲမှာ User Role ကို `Admin` ကနေ `Staff` ပြောင်းလိုက်ပေမယ့် လက်ရှိ JWT/Cookie ထဲမှာ `Admin` Claim ရှိနေသေးရင် Role-only Endpoint က ချက်ချင်းပိတ်မသွားနိုင်ပါဘူး။

ချက်ချင်း Effect ဖြစ်ရမယ့် System ဆိုရင်

* Role ကိုပါ Database က Live Check လုပ်ခြင်း
* Token/Cookie ကို Revoke လုပ်ခြင်း
* Session Version သို့မဟုတ် Security Stamp စစ်ခြင်း
* Access Token သက်တမ်းတိုထားခြင်း

စတဲ့ Strategy တစ်ခုလိုပါတယ်။

---

# Admin က Permission ပြောင်းတဲ့ Real World Flow

Staff ကို Product Create Permission ပေးတယ်ဆိုပါစို့။

## Step 1

Admin UI မှာ

```text
Role: Staff
Permission: Product.Create
```

ကိုရွေးပါတယ်။

## Step 2

Application က `TblRolePermissions` ထဲ Record ထည့်ပါတယ်။

```text
RoleId = 2
PermissionId = 2
```

## Step 3A - Claim Snapshot သုံးရင်

Staff User က Re-login သို့မဟုတ် Token Refresh လုပ်တဲ့အခါ Permission အသစ်ကို Claim ထဲရပါတယ်။

```text
Old token/cookie → Product.Create မရှိသေး
New token/cookie → Product.Create ပါလာပြီ
```

## Step 3B - Live Check သုံးရင်

Staff ရဲ့ နောက် Request မှာ Database Mapping အသစ်ကို တွေ့ပြီး ချက်ချင်း Access ရနိုင်ပါတယ်။

Application Restart လုပ်စရာမလိုပါဘူး။

---

# Permission ရုပ်သိမ်းခြင်း

Permission ပေးတာထက် ရုပ်သိမ်းတာက Security အတွက် ပိုအရေးကြီးပါတယ်။

Admin Role ကနေ `Product.Delete` ကို ဖယ်လိုက်တယ်ဆိုပါစို့။

## Claim Snapshot

လက်ရှိ JWT/Cookie ထဲမှာ `Product.Delete` Claim ရှိနေသေးနိုင်ပါတယ်။ Token/Cookie အသစ်မရမချင်း Delete လုပ်နိုင်နေသေးနိုင်ပါတယ်။

## Live Database Check

နောက် Delete Request မှာ Mapping မရှိတော့တာကို Database က တွေ့ပြီး `403 Forbidden` ပြန်ပေးနိုင်ပါတယ်။

ဒါကြောင့် Permission Revocation ချက်ချင်းဖြစ်ရမယ့် Requirement ရှိမရှိကို Design မလုပ်ခင် ဆုံးဖြတ်သင့်ပါတယ်။

---

# N-Layered Architecture မှာ Dynamic RBAC

Dynamic RBAC ကို N-Layered Architecture မှာ ဒီလိုခွဲနိုင်ပါတယ်။

```text
Presentation Layer
Controller / Permission Attribute
            │
            ▼
Application or Business Layer
Permission evaluation / Role management
            │
            ▼
Data Access Layer
User, Role, Permission queries
            │
            ▼
Database
```

## Presentation Layer

* Required Permission ကို ဖော်ပြတယ်
* Unauthorized/Forbidden Response ပြန်တယ်
* Admin Role-Permission Management UI ပြတယ်

## Business Layer

* Permission Assign/Remove Rule စစ်တယ်
* Default Deny Policy ကို ထိန်းတယ်
* Admin က ကိုယ့်ကိုယ်ကို Critical Permission အကုန်မဖယ်မိအောင် Business Rule စစ်နိုင်တယ်

## Data Access Layer

* Roles ယူတယ်
* Permissions ယူတယ်
* Role-Permission Mapping Insert/Delete လုပ်တယ်
* Current User Permission Query လုပ်တယ်

Controller က SQL တိုက်ရိုက်မရေးသင့်သလို Repository က HTTP Response မပြန်သင့်ပါဘူး။

---

# 401 Unauthorized နဲ့ 403 Forbidden

## 401 Unauthorized

Authentication မအောင်မြင်တဲ့အခါ ဖြစ်ပါတယ်။

ဥပမာ...

* JWT မပါခြင်း
* JWT Expire ဖြစ်ခြင်း
* Invalid Cookie
* Login မဝင်ထားခြင်း

## 403 Forbidden

User ကို Authentication လုပ်ပြီးပြီ၊ ဒါပေမယ့် Required Permission မရှိတဲ့အခါ ဖြစ်ပါတယ်။

ဥပမာ...

```text
Staff is authenticated
Staff does not have Product.Delete
Result = 403 Forbidden
```

Permission Filter မှာ User Login မဝင်ထားရင် `ChallengeResult`, Permission မရှိရင် `ForbidResult` ပြန်ပေးတာက ဒီကွာခြားချက်ကို ထိန်းထားတာဖြစ်ပါတယ်။

---

# Performance Considerations

Live Database Check က Dynamic ဖြစ်ပေမယ့် Protected Request တိုင်း Query ဝင်ရင် Traffic များတဲ့ System မှာ Database Load တက်နိုင်ပါတယ်။

လိုအပ်မှသာ အောက်ပါ Optimization တွေ စဉ်းစားသင့်ပါတယ်။

* `(RoleId, PermissionId)` Index ထားခြင်း
* Permission Name ကို Unique Index ထားခြင်း
* Read-only Query မှာ `AsNoTracking()` သုံးခြင်း
* Role-Permission Set ကို Short-lived Cache လုပ်ခြင်း
* Permission ပြောင်းတဲ့အခါ Cache Invalidate လုပ်ခြင်း
* JWT Access Token ကို သက်တမ်းတိုထားပြီး Claim Snapshot သုံးခြင်း

Project အသေးမှာ Cache System ကြီး စတင်တည်ဆောက်စရာမလိုပါဘူး။ Query Performance ကို တိုင်းပြီးမှ လိုအပ်တဲ့ Optimization ကို ထည့်သင့်ပါတယ်။

---

# Data Integrity Considerations

Dynamic RBAC မှာ Database က Authorization Source of Truth ဖြစ်တဲ့အတွက် Data မှန်ကန်မှုအရေးကြီးပါတယ်။

Production Database မှာ အနည်းဆုံး

* Role Name ကို Unique ထားခြင်း
* Permission Name ကို Unique ထားခြင်း
* `(RoleId, PermissionId)` ကို Unique ထားခြင်း
* Foreign Keys သတ်မှတ်ခြင်း
* Role ဖျက်တဲ့အခါ Mapping တွေကို ဘယ်လိုလုပ်မလဲ သတ်မှတ်ခြင်း
* Permission Assign/Remove ကို Transaction ထဲမှာလုပ်ခြင်း

စတာတွေ ထားသင့်ပါတယ်။

---

# Admin UI မှာ ပါသင့်တဲ့ Features

Dynamic RBAC ကို Admin UI နဲ့ စီမံမယ်ဆိုရင် အများအားဖြင့်

* Role List
* Create/Edit Role
* Permission List
* Role တစ်ခုအတွက် Permission Checkboxes
* User ကို Role Assign လုပ်ခြင်း
* Permission Change Audit History

တွေပါနိုင်ပါတယ်။

ဒါပေမယ့် Admin UI က Dynamic RBAC ရဲ့ မဖြစ်မနေအစိတ်အပိုင်းမဟုတ်ပါဘူး။ Database Migration, Seed Data သို့မဟုတ် Internal Tool ကနေ Mapping ကို စီမံလို့လည်းရပါတယ်။

---

# Security Best Practices

Dynamic RBAC က Access Control System ဖြစ်တဲ့အတွက် Security အပိုင်းကို အပြည့်အဝထည့်စဉ်းစားရပါတယ်။

* Password ကို Plain Text မသိမ်းဘဲ Password Hasher သုံးပါ။
* Default အနေနဲ့ Deny လုပ်ပြီး လိုအပ်တဲ့ Permission ကိုသာ Grant လုပ်ပါ။
* Least Privilege Principle ကိုလိုက်နာပါ။
* Role/Permission ပြောင်းခွင့်ကို Trusted Admin အတွက်ပဲ ဖွင့်ပါ။
* Permission Assign/Remove Operation တိုင်းကို Audit Log သိမ်းပါ။
* Admin UI Form တွေမှာ CSRF Protection နဲ့ Server-side Authorization ထားပါ။
* Permission Name ကို User Input ကနေ မယုံဘဲ Valid Permission Catalog နဲ့စစ်ပါ။
* JWT Signing Key နဲ့ Database Secret တွေကို Secret Store မှာထားပါ။
* JWT ထဲ Sensitive Data မထည့်ပါနဲ့။
* Production မှာ HTTPS သုံးပါ။
* Permission Revocation ဘယ်လောက်မြန်ရမလဲဆိုတာ သတ်မှတ်ပါ။
* UI ဖျောက်ထားတာကို Security Boundary အဖြစ် မယူဆပါနဲ့။
* Role-Permission Query Error ဖြစ်ရင် Access ပေးမယ့်အစား Deny လုပ်ပါ။

Repository ထဲက In-Memory Database နဲ့ Simple Password တွေက သင်ခန်းစာ Flow အတွက်သာဖြစ်ပါတယ်။ Production မှာ Persistent Database, Password Hashing, Validation, Audit နဲ့ Secret Management တွေ ထည့်ရပါတယ်။

---

# Test Cases

Dynamic RBAC မှာ Role တစ်ခုချင်းစီ စမ်းရုံမက Permission ပြောင်းလဲမှုပါ စမ်းရပါတယ်။

## Authentication Tests

| Scenario | Expected Result |
|---|---|
| Login မဝင်ထား | 401 / Login Redirect |
| Invalid JWT | 401 |
| Expired JWT | 401 |
| Valid Cookie/JWT | Authentication Success |

## Authorization Tests

| Role | Permission | Endpoint | Expected Result |
|---|---|---|---|
| Admin | Product.Delete | Delete | Allowed |
| Staff | Product.View | List | Allowed |
| Staff | Product.Delete မရှိ | Delete | 403 |

## Dynamic Change Tests

```text
1. Staff မှာ Product.Create မရှိ → Create = Forbidden
2. Mapping ထည့်
3. Live Check → နောက် Request = Allowed
4. Claim Snapshot → Re-login/Refresh ပြီးမှ Allowed
5. Mapping ပြန်ဖယ်
6. Live Check → နောက် Request = Forbidden
```

## Failure Tests

* User ID Claim မရှိခြင်း
* User Record ဖျက်ထားခြင်း
* Role Record မရှိခြင်း
* Permission Mapping Duplicate ဖြစ်ခြင်း
* Database Connection Failure

Database Error ဖြစ်တဲ့အခါ Permission ပေးမယ့်အစား Request ကို Deny လုပ်သင့်ပါတယ်။

---

# Dynamic RBAC ရဲ့ အားသာချက်များ

* **Flexible** – Role-Permission Mapping ကို Code မပြင်ဘဲ ပြောင်းနိုင်ပါတယ်။
* **Centralized Management** – Permission Rule တွေ Database တစ်နေရာထဲမှာ စီမံနိုင်ပါတယ်။
* **Scalable Permission Model** – Module နဲ့ Action များလာတဲ့အခါ Table Data အနေနဲ့ တိုးနိုင်ပါတယ်။
* **Admin Control** – Admin UI ကနေ Role နဲ့ Permission စီမံနိုင်ပါတယ်။
* **Reusable Roles** – Role တစ်ခုကို User အများကြီး အသုံးပြုနိုင်ပါတယ်။
* **Immediate Revocation Option** – Live Database Check သုံးရင် Permission ကို နောက် Request ကစပြီး ရုပ်သိမ်းနိုင်ပါတယ်။
* **Audit လုပ်နိုင်တယ်** – ဘယ်သူက ဘယ် Permission ပြောင်းတယ်ဆိုတာ မှတ်တမ်းတင်နိုင်ပါတယ်။

---

# Dynamic RBAC ရဲ့ အားနည်းချက်များ

* Static RBAC ထက် Database Structure နဲ့ Code Flow ပိုများပါတယ်။
* Live Check သုံးရင် Request တိုင်း Database Query ဝင်နိုင်ပါတယ်။
* Cache သုံးရင် Permission ပြောင်းလဲမှုနဲ့ Cache Invalidation ကို မှန်ကန်စွာကိုင်တွယ်ရပါတယ်။
* JWT/Cookie Claim Snapshot သုံးရင် Permission အဟောင်း Stale ဖြစ်နေနိုင်ပါတယ်။
* Admin UI ကို မှားယွင်းကာကွယ်ထားရင် System Permission အကုန် ပေါက်နိုင်ပါတယ်။
* Role/Permission Data မမှန်ရင် User Access မှားနိုင်ပါတယ်။

---

# ဘယ်လို Project တွေမှာ Dynamic RBAC သုံးသင့်လဲ?

အောက်ပါအခြေအနေတွေမှာ Dynamic RBAC က သင့်တော်ပါတယ်။

* Role နဲ့ Permission များတဲ့ Enterprise Application
* Admin က Runtime မှာ Permission ပြောင်းရတဲ့ System
* Module အသစ် မကြာခဏထည့်တဲ့ SaaS Application
* User အများကြီးရှိပြီး Role အလိုက် စီမံရတဲ့ Back-office System
* Banking, ERP, HR, Inventory, E-commerce Admin System
* Permission Change Audit လိုအပ်တဲ့ System
* Code Deploy မလုပ်ဘဲ Access Rule ပြောင်းရမယ့် System

Role နည်းပြီး Permission Rule တည်ငြိမ်တဲ့ Project အသေးမှာတော့ Static RBAC က ပိုရိုးရှင်းပြီး လုံလောက်နိုင်ပါတယ်။

---

# Static RBAC ကနေ Dynamic RBAC ပြောင်းမယ်ဆိုရင်

အဆင့်လိုက်ပြောင်းနိုင်ပါတယ်။

## Step 1

Code နဲ့ User Record ထဲက Role/Permission List ကို စာရင်းကောက်ပါ။

## Step 2

`Roles`, `Permissions`, `RolePermissions` Tables တည်ဆောက်ပါ။

## Step 3

လက်ရှိ Static Mapping ကို Seed Data အဖြစ် Database ထဲထည့်ပါ။

## Step 4

Login Flow ကို Database က Role/Permissions ယူအောင် ပြောင်းပါ။

## Step 5

Claim Snapshot သို့မဟုတ် Live Database Check ဘယ်ဟာသုံးမလဲ ဆုံးဖြတ်ပါ။

## Step 6

Role-Permission Management နဲ့ Audit Flow ကို လိုအပ်မှ ထည့်ပါ။

အဆင့်အားလုံးကို တစ်ခါတည်းတည်ဆောက်စရာမလိုပါဘူး။ Database Mapping နဲ့ Authorization Check ကို အရင်တည်ငြိမ်အောင်လုပ်ပြီး Admin UI ကို နောက်မှထည့်နိုင်ပါတယ်။

---

# ASP.NET Core Policy-based Authorization နဲ့ Dynamic Policies

ASP.NET Core မှာ `[Authorize(Policy = "...")]`, `IAuthorizationRequirement`, `AuthorizationHandler` စတဲ့ Policy-based Authorization Features တွေရှိပါတယ်။

Policy အရေအတွက်များပြီး Runtime မှာ Database သို့မဟုတ် External Source ကနေ Policy တည်ဆောက်ရမယ်ဆိုရင် Custom `IAuthorizationPolicyProvider` ကို အသုံးပြုနိုင်ပါတယ်။

Repository ထဲက Example ကတော့ ရှိပြီးသား `PermissionAttribute` နဲ့ `PermissionFilter` ကို ပြန်သုံးထားတာကြောင့် MVC Controller နဲ့ Web API Controller တွေအတွက် တိုက်ရိုက်နားလည်ရလွယ်ပါတယ်။

Project တစ်ခုမှာ Pattern တစ်မျိုးကိုပဲ တည်ငြိမ်စွာသုံးသင့်ပြီး Controller တစ်ချို့မှာ Claim Policy၊ တစ်ချို့မှာ Live Filter၊ တစ်ချို့မှာ Role Check ရောထွေးနေမယ်ဆိုရင် Permission Change Effect Time ကို ရှင်းရှင်းလင်းလင်း Document လုပ်ထားရပါတယ်။

---

# Summary

**Dynamic RBAC** ဆိုတာ User, Role, Permission နဲ့ Role-Permission Mapping တွေကို Database ထဲမှာ ခွဲသိမ်းပြီး Role တစ်ခုရဲ့ Access Rule ကို Runtime မှာ စီမံနိုင်တဲ့ Authorization ပုံစံဖြစ်ပါတယ်။

အဓိက Database Flow က

```text
Users → Roles → RolePermissions → Permissions
```

ဖြစ်ပါတယ်။

JWT သို့မဟုတ် Cookie ထဲကို Login အချိန် Permission Claims ထည့်ပြီး စစ်တဲ့ **Claim Snapshot** ပုံစံက မြန်ပါတယ်။ Protected Request တိုင်း Database ကို စစ်တဲ့ **Live Database Check** ပုံစံက Permission ပြောင်းလဲမှုကို ချက်ချင်းအသုံးချနိုင်ပေမယ့် Query Cost ပိုရှိပါတယ်။

အရေးကြီးဆုံးက Dynamic ဆိုတဲ့နာမည်တစ်ခုတည်းကို မကြည့်ဘဲ

```text
Permission ပြောင်းရင် ဘယ်အချိန် Effect ဖြစ်ရမလဲ?
```

ဆိုတဲ့ Requirement ကို အရင်ဆုံးဆုံးဖြတ်ဖို့ဖြစ်ပါတယ်။ အဲ့ဒီဆုံးဖြတ်ချက်ပေါ်မူတည်ပြီး Claim Snapshot, Live Check သို့မဟုတ် Short-lived Cache ပုံစံကို ရွေးချယ်သင့်ပါတယ်။

---

# References

* [Introduction to authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authorization/introduction)
* [Role-based authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/security/authorization/roles)
* [Policy-based authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authorization/policies)
* [Custom authorization policy providers in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authorization/iauthorizationpolicyprovider)
