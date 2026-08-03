# RBAC ဆိုတာဘာလဲ?

**RBAC** ရဲ့ အရှည်ကောက်က

```text
Role-Based Access Control
```

ဖြစ်ပါတယ်။

RBAC ဆိုတာ User တစ်ယောက် System ထဲမှာ ဘာလုပ်ခွင့်ရှိသလဲဆိုတာကို သူ့ရဲ့ **Role (ရာထူး သို့မဟုတ် တာဝန်)** ပေါ်မူတည်ပြီး ဆုံးဖြတ်ပေးတဲ့ Access Control ပုံစံတစ်ခုဖြစ်ပါတယ်။

User တစ်ယောက်ချင်းစီကို Permission တစ်ခုချင်းစီ တိုက်ရိုက်ပေးမယ့်အစား

1. Permission တွေကို Role မှာပေးတယ်
2. Role ကို User မှာပေးတယ်
3. User က Role ကတစ်ဆင့် Permission တွေရတယ်

ဆိုတဲ့ပုံစံနဲ့ စီမံပါတယ်။

```text
User
  │
  ▼
Role
  │
  ▼
Permission
  │
  ▼
Resource / Action
```

ဥပမာ...

```text
Mg Mg ──► Admin ──► Product.View
                  ├─► Product.Create
                  ├─► Product.Update
                  └─► Product.Delete

Aye Aye ─► Staff ──► Product.View
```

ဒီ Example မှာ Mg Mg ကို Permission လေးခု တိုက်ရိုက်ပေးထားတာမဟုတ်ပါဘူး။ Mg Mg ကို `Admin` Role ပေးထားပြီး `Admin` Role ကတစ်ဆင့် Permission လေးခုလုံးရရှိတာဖြစ်ပါတယ်။

---

# ဘာကြောင့် Access Control လိုတာလဲ?

Application တစ်ခုမှာ Login ဝင်နိုင်တဲ့ User အားလုံးကို အလုပ်အားလုံး လုပ်ခွင့်ပေးလို့မရပါဘူး။

ဥပမာ Inventory System တစ်ခုမှာ

* Staff က Product List ကြည့်ရမယ်
* Manager က Product ပြင်နိုင်ရမယ်
* Admin က Product ဖျက်နိုင်ရမယ်
* Customer က Admin Page မဝင်ရဘူး

ဆိုတဲ့ Rule တွေရှိနိုင်ပါတယ်။

Access Control မရှိရင်

* ခွင့်မရှိသူက Data ဖျက်နိုင်တယ်
* Sensitive Report တွေ ကြည့်နိုင်တယ်
* User Account တွေ ပြင်နိုင်တယ်
* System Setting တွေ ပြောင်းနိုင်တယ်

ဒါကြောင့်

```text
ဘယ်သူက ဘာကို ဘယ်လိုလုပ်ခွင့်ရှိသလဲ?
```

ဆိုတာကို System က စစ်ပေးရပါတယ်။

RBAC က ဒီ Access Rule တွေကို Role အလိုက် စနစ်တကျ စီမံပေးတာဖြစ်ပါတယ်။

---

# Authentication နဲ့ Authorization

RBAC ကို နားလည်ဖို့ Authentication နဲ့ Authorization ကို အရင်ခွဲသိရပါတယ်။

## Authentication

Authentication ဆိုတာ

```text
ဒီ User က ဘယ်သူလဲ?
```

ဆိုတာ စစ်တာဖြစ်ပါတယ်။

ဥပမာ...

* Username နဲ့ Password စစ်ခြင်း
* JWT Token စစ်ခြင်း
* Authentication Cookie စစ်ခြင်း
* Fingerprint သို့မဟုတ် OTP စစ်ခြင်း

Login အောင်မြင်ရင် System က User ကို သိသွားပါတယ်။

## Authorization

Authorization ဆိုတာ

```text
ဒီ User က ဒီအလုပ်ကို လုပ်ခွင့်ရှိလား?
```

ဆိုတာ စစ်တာဖြစ်ပါတယ်။

ဥပမာ...

* Product ကြည့်ခွင့်ရှိလား?
* Order Approve လုပ်ခွင့်ရှိလား?
* User Delete လုပ်ခွင့်ရှိလား?
* Salary Report ကြည့်ခွင့်ရှိလား?

RBAC က Authorization လုပ်ဖို့ အသုံးပြုတဲ့ ပုံစံတစ်ခုဖြစ်ပါတယ်။

```text
Request
   │
   ▼
Authentication
User ဘယ်သူလဲ?
   │
   ▼
Authorization (RBAC)
User ရဲ့ Role နဲ့ Permission ကဘာလဲ?
   │
   ▼
Allow or Deny
```

Authentication မရှိဘဲ Authorization မလုပ်နိုင်ပါဘူး။ User ဘယ်သူမှန်း အရင်သိမှ သူ့ရဲ့ Role နဲ့ Permission ကို စစ်နိုင်မှာဖြစ်ပါတယ်။

---

# RBAC ရဲ့ အဓိကအစိတ်အပိုင်းများ

RBAC မှာ အောက်ပါအစိတ်အပိုင်းတွေကို အများဆုံးတွေ့ရပါတယ်။

1. User
2. Role
3. Permission
4. Resource
5. Action
6. User-Role Assignment
7. Role-Permission Assignment

---

# 1. User

User ဆိုတာ System ကို အသုံးပြုမယ့် လူ သို့မဟုတ် Account ဖြစ်ပါတယ်။

ဥပမာ...

```text
admin
manager01
staff05
customer100
```

User တစ်ယောက်မှာ

* Id
* Username
* Password Hash
* Email
* Role

စတဲ့ Data တွေရှိနိုင်ပါတယ်။

RBAC မှာ User က Permission ကို Role ကတစ်ဆင့် ရရှိပါတယ်။

---

# 2. Role

Role ဆိုတာ User တစ်ယောက်ရဲ့ ရာထူး၊ တာဝန် သို့မဟုတ် Access Level ကို ကိုယ်စားပြုပါတယ်။

ဥပမာ...

```text
Admin
Manager
Staff
Customer
Auditor
```

Role Name က လူနာမည်မဟုတ်ပါဘူး။ လူအများကြီးမှာ တူညီတဲ့ Role ရှိနိုင်ပါတယ်။

```text
Mg Mg   ─┐
Aye Aye ─┼─► Staff
Su Su   ─┘
```

Staff Role ရဲ့ Permission ပြောင်းလိုက်ရင် Staff Role ရှိတဲ့ User အားလုံးရဲ့ Access ကို တစ်နေရာထဲကနေ စီမံနိုင်ပါတယ်။

---

# 3. Permission

Permission ဆိုတာ System ထဲမှာ လုပ်ခွင့်ရှိတဲ့ Operation တစ်ခုကို ကိုယ်စားပြုပါတယ်။

ဥပမာ...

```text
Product.View
Product.Create
Product.Update
Product.Delete
```

Permission Name ကို

```text
Resource.Action
```

ပုံစံနဲ့ပေးရင် နားလည်ရလွယ်ပါတယ်။

နောက်ထပ်ဥပမာတွေကတော့

```text
Order.View
Order.Approve
Order.Cancel
User.Create
User.Disable
Report.Export
```

ဖြစ်ပါတယ်။

Permission ကို တိတိကျကျခွဲထားရင် Role တစ်ခုအတွက် လိုအပ်သလောက် Access ပဲပေးနိုင်ပါတယ်။

---

# 4. Resource

Resource ဆိုတာ User ဝင်ရောက်အသုံးပြုမယ့် Data သို့မဟုတ် Feature ဖြစ်ပါတယ်။

ဥပမာ...

* Product
* Order
* User
* Invoice
* Report

`Product.Delete` Permission မှာ `Product` က Resource ဖြစ်ပါတယ်။

---

# 5. Action

Action ဆိုတာ Resource တစ်ခုပေါ်မှာ လုပ်မယ့် Operation ဖြစ်ပါတယ်။

ဥပမာ...

* View
* Create
* Update
* Delete
* Approve
* Export

`Product.Delete` Permission မှာ `Delete` က Action ဖြစ်ပါတယ်။

---

# 6. User-Role Assignment

User ကို Role တစ်ခု သို့မဟုတ် တစ်ခုထက်ပိုပြီး သတ်မှတ်ပေးတာဖြစ်ပါတယ်။

```text
admin user ──► Admin role
staff user ──► Staff role
```

Project ရိုးရှင်းရင် User တစ်ယောက်မှာ Role တစ်ခုပဲရှိနိုင်ပါတယ်။

Enterprise System တချို့မှာတော့ User တစ်ယောက်မှာ Role အများကြီးရှိနိုင်ပါတယ်။

```text
Ko Ko ──► Manager
      └─► Auditor
```

---

# 7. Role-Permission Assignment

Role တစ်ခုကို Permission တွေ သတ်မှတ်ပေးတာဖြစ်ပါတယ်။

```text
Admin ──► Product.View
      ├─► Product.Create
      ├─► Product.Update
      └─► Product.Delete

Staff ──► Product.View
```

User က Permission ကို တိုက်ရိုက်မရဘဲ Role-Permission Assignment ကတစ်ဆင့် ရရှိပါတယ်။

---

# Role နဲ့ Permission မတူပါဘူး

Role နဲ့ Permission ကို မရောသင့်ပါဘူး။

## Role

```text
User က ဘာတာဝန်ရှိသလဲ?
```

ဆိုတာကို ဖော်ပြပါတယ်။

ဥပမာ...

```text
Admin
Staff
Manager
```

## Permission

```text
User က ဘာ Action လုပ်နိုင်သလဲ?
```

ဆိုတာကို ဖော်ပြပါတယ်။

ဥပမာ...

```text
Product.Delete
Order.Approve
Report.Export
```

Role က Permission အများကြီးကို စုထားတဲ့ Group တစ်ခုလိုဖြစ်ပါတယ်။

```text
Role = Responsibilities
Permission = Allowed Actions
```

---

# Permission Matrix ဥပမာ

Product Management System တစ်ခုမှာ ဒီလို Permission Matrix ရှိတယ်ဆိုပါစို့။

| Permission | Admin | Manager | Staff |
|---|---:|---:|---:|
| Product.View | ✅ | ✅ | ✅ |
| Product.Create | ✅ | ✅ | ❌ |
| Product.Update | ✅ | ✅ | ❌ |
| Product.Delete | ✅ | ❌ | ❌ |

ဒီ Table ကိုကြည့်ရုံနဲ့ Role တစ်ခုချင်းစီ ဘာလုပ်ခွင့်ရှိသလဲဆိုတာ သိနိုင်ပါတယ်။

ဥပမာ Staff က Product List ကြည့်နိုင်ပေမယ့် Create, Update, Delete မလုပ်နိုင်ပါဘူး။

---

# RBAC ဘယ်လိုအလုပ်လုပ်သလဲ?

Product Delete Request တစ်ခုနဲ့ စဉ်းစားကြည့်ရအောင်။

## Step 1

User က Login ဝင်ပါတယ်။

```text
Username: admin
Password: ********
```

## Step 2

System က User ကို Authentication လုပ်ပါတယ်။

```text
User = admin
Role = Admin
```

## Step 3

User က Product Delete Request ပို့ပါတယ်။

```http
DELETE /api/products/10
```

## Step 4

System က Endpoint အတွက် လိုအပ်တဲ့ Permission ကိုကြည့်ပါတယ်။

```text
Required Permission = Product.Delete
```

## Step 5

Admin Role မှာ `Product.Delete` Permission ရှိမရှိ စစ်ပါတယ်။

```text
Admin has Product.Delete = Yes
```

## Step 6

Permission ရှိတဲ့အတွက် Request ကို Allow လုပ်ပါတယ်။

```text
Product deleted successfully
```

Staff User က အတူတူ Request ပို့ရင်တော့

```text
Staff has Product.Delete = No
```

ဖြစ်တဲ့အတွက် Request ကို Deny လုပ်ပါတယ်။

---

# RBAC Authorization Flow

```text
Client Request
      │
      ▼
Authentication
      │
      ▼
Find User Roles
      │
      ▼
Find Role Permissions
      │
      ▼
Find Required Permission
      │
      ▼
Does the user have permission?
      │
  ┌───┴───┐
  ▼       ▼
 Yes      No
  │       │
  ▼       ▼
Allow    Deny
```

အရေးကြီးတာက Controller Action မလုပ်ခင် Authorization ကို စစ်ပြီးသားဖြစ်ရပါတယ်။

---

# Real World Example - စားသောက်ဆိုင်

RBAC ကို စားသောက်ဆိုင်တစ်ခုနဲ့ နှိုင်းကြည့်ရအောင်။

## Owner

* Menu ပြင်နိုင်တယ်
* Staff ခန့်နိုင်တယ်
* Sales Report ကြည့်နိုင်တယ်
* System Setting ပြောင်းနိုင်တယ်

## Cashier

* Order လက်ခံနိုင်တယ်
* Payment ယူနိုင်တယ်
* Receipt ထုတ်နိုင်တယ်

## Chef

* Order ကြည့်နိုင်တယ်
* Cooking Status ပြောင်းနိုင်တယ်

Cashier က Kitchen Order Status ပြောင်းစရာမလိုသလို Chef က Sales Report ကြည့်စရာမလိုပါဘူး။

Role တစ်ခုချင်းစီကို သူ့အလုပ်အတွက် လိုအပ်တဲ့ Permission ပဲပေးထားတာက RBAC ဖြစ်ပါတယ်။

---

# Least Privilege Principle

RBAC Design မှာ အရေးကြီးဆုံး Principle တစ်ခုက **Least Privilege** ဖြစ်ပါတယ်။

အဓိပ္ပာယ်က

```text
User ကို သူ့အလုပ်လုပ်ဖို့ လိုအပ်သလောက် Permission ပဲပေးပါ
```

ဖြစ်ပါတယ်။

Staff က Product ကြည့်ဖို့ပဲလိုရင်

```text
Product.View
```

ပဲပေးသင့်ပါတယ်။

အလွယ်တကူအလုပ်ဖြစ်အောင် `Admin` Role ပေးလိုက်တာမျိုး မလုပ်သင့်ပါဘူး။

Permission ပိုပေးထားလေလေ

* User အမှားလုပ်တဲ့အခါ Damage ပိုများနိုင်တယ်
* Account ခိုးခံရရင် Attacker လုပ်နိုင်တာ ပိုများတယ်
* Audit နဲ့ Compliance ပိုခက်တယ်

ဖြစ်ပါတယ်။

---

# Separation of Duties

အရေးကြီးတဲ့လုပ်ငန်းစဉ်တစ်ခုလုံးကို လူတစ်ယောက်တည်း မလုပ်နိုင်အောင် Role ခွဲထားတာကို **Separation of Duties** လို့ခေါ်ပါတယ်။

ဥပမာ ငွေပေးချေမှုတစ်ခုမှာ

```text
Staff   ──► Payment Request Create
Manager ──► Payment Request Approve
```

လို့ ခွဲထားနိုင်ပါတယ်။

Staff တစ်ယောက်တည်းက Payment Create လည်းလုပ်၊ Approve လည်းလုပ်နိုင်ရင် Fraud Risk ပိုများပါတယ်။

RBAC Role တွေကို တာဝန်ခွဲထားခြင်းက ဒီ Risk ကို လျှော့ချပေးနိုင်ပါတယ်။

---

# Role Hierarchy

တချို့ System တွေမှာ Role အဆင့်ဆင့်ရှိနိုင်ပါတယ်။

```text
Admin
  │
  ▼
Manager
  │
  ▼
Staff
```

Admin က Manager Permission တွေကို ရပြီး Manager က Staff Permission တွေကို ရတယ်ဆိုတဲ့ပုံစံဖြစ်ပါတယ်။

ဒါကို Role Hierarchy လို့ခေါ်ပါတယ်။

ဒါပေမယ့် Project အသေးမှာ Hierarchy မလိုရင် မထည့်သင့်ပါဘူး။ Role တစ်ခုချင်းစီအတွက် Permission ကို တိုက်ရိုက်သတ်မှတ်တာက ပိုရှင်းပါတယ်။

---

# ASP.NET Core မှာ RBAC ကို ဘယ်လိုဖော်ပြလဲ?

ASP.NET Core မှာ User ကို `ClaimsPrincipal` နဲ့ ကိုယ်စားပြုပါတယ်။

Role ကို Role Claim အဖြစ် ထည့်နိုင်ပါတယ်။

```csharp
new Claim(ClaimTypes.Role, "Admin")
```

Permission ကို Custom Claim အဖြစ် ထည့်နိုင်ပါတယ်။

```csharp
new Claim("permission", "Product.Delete")
```

Controller ကို Role နဲ့ ကာကွယ်နိုင်ပါတယ်။

```csharp
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly()
{
    return Ok();
}
```

Permission Policy နဲ့လည်း ကာကွယ်နိုင်ပါတယ်။

```csharp
[Authorize(Policy = "ProductDelete")]
public IActionResult DeleteProduct(int id)
{
    return Ok();
}
```

Authentication ကို JWT သို့မဟုတ် Cookie နဲ့လုပ်နိုင်ပေမယ့် RBAC ရဲ့ အဓိကရည်ရွယ်ချက်က အတူတူပါပဲ။

```text
JWT/Cookie = User ဘယ်သူလဲဆိုတာ သယ်ဆောင်ပေးတယ်
RBAC       = User ဘာလုပ်ခွင့်ရှိလဲဆိုတာ ဆုံးဖြတ်တယ်
```

အသေးစိတ် Implementation ကို [Static RBAC](Static%20RBAC.md) နဲ့ [Dynamic RBAC](Dynamic%20RBAC.md) မှာ ဆက်လေ့လာနိုင်ပါတယ်။

---

# Role Check နဲ့ Permission Check

Role ကို တိုက်ရိုက်စစ်နိုင်ပါတယ်။

```csharp
[Authorize(Roles = "Admin")]
```

ဒီပုံစံက Admin-only Feature လို Role ကိုယ်တိုင် အဓိပ္ပာယ်ရှိတဲ့နေရာမှာ ရိုးရှင်းပါတယ်။

Permission ကိုစစ်ရင်တော့ Endpoint ရဲ့လိုအပ်ချက် ပိုတိကျပါတယ်။

```text
Product Delete Endpoint
        requires
Product.Delete Permission
```

Role အသစ်ထည့်လာရင် Endpoint Code ကို Role Name အသစ်နဲ့ပြင်စရာမလိုဘဲ Role အသစ်ကို `Product.Delete` Permission ပေးနိုင်ပါတယ်။

ဒါကြောင့် Feature များတဲ့ System တွေမှာ Permission-based Check က ပိုပြောင်းလွယ်ပြင်လွယ်ရှိပါတယ်။

---

# 401 Unauthorized နဲ့ 403 Forbidden

RBAC မှာ ဒီ HTTP Status Code နှစ်ခုကို ခွဲသိဖို့လိုပါတယ်။

## 401 Unauthorized

User ကို Authentication မလုပ်နိုင်တဲ့အခါ ဖြစ်ပါတယ်။

ဥပမာ...

* Login မဝင်ထားခြင်း
* JWT မပါခြင်း
* JWT Expire ဖြစ်ခြင်း
* Invalid Authentication Cookie

အဓိပ္ပာယ်က

```text
ဒီ Request ကို ဘယ်သူပို့တာလဲ မသိဘူး
```

ဖြစ်ပါတယ်။

## 403 Forbidden

User ကို Authentication လုပ်ပြီးပြီ၊ ဒါပေမယ့် လိုအပ်တဲ့ Role သို့မဟုတ် Permission မရှိတဲ့အခါ ဖြစ်ပါတယ်။

အဓိပ္ပာယ်က

```text
ဒီ User ကို သိတယ်၊ ဒါပေမယ့် ဒီအလုပ်လုပ်ခွင့်မရှိဘူး
```

ဖြစ်ပါတယ်။

---

# UI ဖျောက်ထားတာက Security မဟုတ်ပါဘူး

Permission မရှိတဲ့ User ကို Delete Button မပြတာက User Experience အတွက် ကောင်းပါတယ်။

```cshtml
@if (User.HasClaim("permission", "Product.Delete"))
{
    <button>Delete</button>
}
```

ဒါပေမယ့် User က HTTP Request ကို တိုက်ရိုက်ပို့နိုင်ပါတယ်။

```http
DELETE /api/products/10
```

ဒါကြောင့် Controller, API Endpoint သို့မဟုတ် Business Operation မှာ Server-side Authorization မဖြစ်မနေ စစ်ရပါတယ်။

```text
Hide Button       = User Experience
Protect Endpoint  = Security
```

---

# Static RBAC နဲ့ Dynamic RBAC

RBAC ကို Role-Permission Rule ဘယ်မှာစီမံသလဲဆိုတာပေါ်မူတည်ပြီး Static နဲ့ Dynamic အဖြစ် ခွဲလေ့ရှိပါတယ်။

## Static RBAC

Role, Permission နဲ့ Policy Rule တွေကို Code သို့မဟုတ် ကြိုတင်သတ်မှတ်ထားတဲ့ User Data ထဲမှာ ထားပါတယ်။

```text
Admin = View + Create + Update + Delete
Staff = View
```

Rule က တည်ငြိမ်ပြီး Project အသေးအတွက် ရိုးရှင်းပါတယ်။

အသေးစိတ်ကို [Static RBAC.md](Static%20RBAC.md) မှာ ဖတ်နိုင်ပါတယ်။

## Dynamic RBAC

Roles, Permissions နဲ့ Role-Permission Mapping ကို Database ထဲမှာ ခွဲသိမ်းပါတယ်။

```text
Users → Roles → RolePermissions → Permissions
```

Admin က Mapping ကိုပြောင်းပြီး Role တစ်ခုရဲ့ Permission တွေကို Runtime မှာ စီမံနိုင်ပါတယ်။

အသေးစိတ်ကို [Dynamic RBAC.md](Dynamic%20RBAC.md) မှာ ဖတ်နိုင်ပါတယ်။

---

# RBAC ရဲ့ အားသာချက်များ

* **စီမံရလွယ်တယ်** – User တစ်ယောက်ချင်းစီထက် Role အလိုက် Permission စီမံနိုင်ပါတယ်။
* **Security ပိုကောင်းတယ်** – လိုအပ်တဲ့ Permission ပဲပေးနိုင်ပါတယ်။
* **User အသစ်ထည့်ရလွယ်တယ်** – သင့်တော်တဲ့ Role ပေးလိုက်ရုံဖြစ်ပါတယ်။
* **Permission ပြောင်းရလွယ်တယ်** – Role Permission ပြောင်းရင် Role ရှိတဲ့ User အားလုံးအပေါ် သက်ရောက်နိုင်ပါတယ်။
* **Audit လုပ်ရလွယ်တယ်** – ဘယ် Role က ဘာလုပ်ခွင့်ရှိလဲဆိုတာ Matrix နဲ့စစ်နိုင်ပါတယ်။
* **Separation of Duties လုပ်နိုင်တယ်** – အရေးကြီးတဲ့တာဝန်တွေကို Role ခွဲထားနိုင်ပါတယ်။
* **Application ကြီးလာရင် ချဲ့နိုင်တယ်** – Module နဲ့ Permission အသစ်တွေ ထည့်နိုင်ပါတယ်။

---

# RBAC ရဲ့ ကန့်သတ်ချက်များ

RBAC က Problem အားလုံးကို ဖြေရှင်းပေးနိုင်တာမဟုတ်ပါဘူး။

## Role Explosion

အခြေအနေတိုင်းအတွက် Role အသစ်တည်ဆောက်ရင် Role အရေအတွက် အလွန်များလာနိုင်ပါတယ်။

```text
YangonManager
MandalayManager
DayShiftManager
NightShiftManager
```

လိုမျိုး Role တွေ ထပ်လာနိုင်ပါတယ်။

## Resource Ownership

`Editor` Role ရှိရုံနဲ့ Document အားလုံးပြင်ခွင့်ပေးလို့ မရနိုင်ပါဘူး။

```text
ဒီ Document ကို ဖန်တီးထားသူကိုယ်တိုင်လား?
ဒီ Order က ဒီ Customer ပိုင်တာလား?
```

ဆိုတဲ့ Resource-specific Rule တွေလည်း လိုနိုင်ပါတယ်။

## Context-based Rules

တချို့ Rule တွေက Role တစ်ခုတည်းနဲ့ မလုံလောက်ပါဘူး။

ဥပမာ...

* Office Hour အတွင်းပဲ Approve လုပ်ရမယ်
* ကိုယ့် Branch က Data ပဲကြည့်ရမယ်
* Amount 1,000,000 ထက်ကြီးရင် Senior Manager လိုမယ်

ဒီလိုအခြေအနေတွေမှာ Policy-based, Resource-based သို့မဟုတ် Attribute-based Authorization ကို RBAC နဲ့တွဲသုံးရနိုင်ပါတယ်။

---

# RBAC Design လုပ်တဲ့အခါ မေးသင့်တဲ့မေးခွန်းများ

RBAC မတည်ဆောက်ခင် အောက်ပါမေးခွန်းတွေ ဖြေထားသင့်ပါတယ်။

1. System ထဲမှာ ဘာ Resources တွေရှိသလဲ?
2. Resource တစ်ခုစီမှာ ဘာ Actions တွေရှိသလဲ?
3. ဘာ Roles တွေလိုသလဲ?
4. Role တစ်ခုစီရဲ့ တာဝန်ကဘာလဲ?
5. Role တစ်ခုစီမှာ ဘာ Permissions တွေလိုသလဲ?
6. User တစ်ယောက်မှာ Role တစ်ခုပဲလား၊ အများကြီးလား?
7. Permission ပြောင်းရင် ချက်ချင်း Effect ဖြစ်ရမလား?
8. ဘယ်သူက Role နဲ့ Permission ပြောင်းခွင့်ရှိသလဲ?
9. Permission Change ကို Audit Log သိမ်းရမလား?
10. Resource Owner, Branch, Amount စတဲ့ Context Rule တွေလိုသလား?

ဒီမေးခွန်းတွေ ဖြေပြီးမှ Static သို့မဟုတ် Dynamic RBAC ကို ရွေးချယ်သင့်ပါတယ်။

---

# Permission Naming Best Practices

Permission Name ကို တစ်သမတ်တည်းပေးထားရင် Code နဲ့ Database ကို နားလည်ရလွယ်ပါတယ်။

အများဆုံးအသုံးပြုနိုင်တဲ့ပုံစံက

```text
Resource.Action
```

ဖြစ်ပါတယ်။

ဥပမာ...

```text
Product.View
Product.Create
Product.Update
Product.Delete
Order.Approve
Report.Export
```

အောက်ပါလို မရှင်းတဲ့ Name တွေကို ရှောင်သင့်ပါတယ်။

```text
Permission1
CanDoThing
FullAccess2
```

Permission တစ်ခုက Action တစ်ခုကို တိတိကျကျကိုယ်စားပြုသင့်ပါတယ်။

---

# Security Best Practices

RBAC က Security Feature ဖြစ်တဲ့အတွက် အောက်ပါအချက်တွေကို မလျှော့သင့်ပါဘူး။

* Default အနေနဲ့ Access Deny လုပ်ပါ။
* Least Privilege Principle ကိုလိုက်နာပါ။
* Permission မရှိတဲ့ UI Element ကိုဖျောက်ထားရုံနဲ့ မလုံလောက်ပါဘူး။
* Endpoint နဲ့ Business Operation မှာ Server-side Authorization စစ်ပါ။
* Role နဲ့ Permission ပြောင်းခွင့်ကို Trusted Admin အတွက်ပဲ ပေးပါ။
* Role/Permission Change တွေကို Audit Log သိမ်းပါ။
* Password ကို Plain Text မသိမ်းဘဲ Password Hasher သုံးပါ။
* Production မှာ HTTPS သုံးပါ။
* JWT ထဲ Password နဲ့ Sensitive Data မထည့်ပါနဲ့။
* Permission ရုပ်သိမ်းမှု ဘယ်အချိန် Effect ဖြစ်မလဲဆိုတာ သတ်မှတ်ပါ။
* Authorization စစ်တဲ့အခါ Error ဖြစ်ရင် Access ပေးမယ့်အစား Deny လုပ်ပါ။

---

# စမ်းသပ်သင့်တဲ့ Test Cases

Permission Matrix တစ်ခုတည်ဆောက်ပြီး Role တစ်ခုချင်းစီကို စမ်းသင့်ပါတယ်။

| User State | Role | Required Permission | Expected Result |
|---|---|---|---|
| Login မဝင်ထား | - | Product.View | 401 / Login Redirect |
| Login ဝင်ထား | Admin | Product.View | Allowed |
| Login ဝင်ထား | Admin | Product.Delete | Allowed |
| Login ဝင်ထား | Staff | Product.View | Allowed |
| Login ဝင်ထား | Staff | Product.Delete | 403 / Access Denied |

ဒါ့အပြင်

* Role ပြောင်းပြီးနောက် Access
* Permission ထည့်ပြီးနောက် Access
* Permission ရုပ်သိမ်းပြီးနောက် Access
* Invalid Token/Cookie
* Expired Session
* User Record မရှိတော့ခြင်း

တွေကိုလည်း စမ်းသင့်ပါတယ်။

---

# ဘယ်လို Project တွေမှာ RBAC သုံးသင့်လဲ?

RBAC ကို အောက်ပါ Project တွေမှာ အသုံးများပါတယ်။

* Admin Dashboard
* Inventory Management System
* HR Management System
* Banking System
* ERP System
* E-commerce Back Office
* Hospital Management System
* School Management System
* SaaS Application

User အားလုံး Access တူတဲ့ Application အသေးမှာ RBAC မလိုနိုင်ပါဘူး။ Role အလိုက် တာဝန်နဲ့ လုပ်ပိုင်ခွင့် ကွဲပြားလာတဲ့အခါ RBAC က အသုံးဝင်ပါတယ်။

---

# လေ့လာသင့်တဲ့အစီအစဉ်

RBAC ကို အောက်ပါအစီအစဉ်နဲ့ လေ့လာရင် နားလည်ရလွယ်ပါတယ်။

```text
1. What is RBAC?
        │
        ▼
2. Static RBAC
        │
        ▼
3. Dynamic RBAC
```

ဒီ Article ပြီးရင်

1. [Static RBAC](Static%20RBAC.md) – Role နဲ့ Permission Rule တွေကို ကြိုတင်သတ်မှတ်တဲ့ပုံစံ
2. [Dynamic RBAC](Dynamic%20RBAC.md) – Role-Permission Mapping ကို Database ကနေ စီမံတဲ့ပုံစံ

ကို ဆက်ဖတ်နိုင်ပါတယ်။

---

# Summary

**RBAC (Role-Based Access Control)** ဆိုတာ User တစ်ယောက်ရဲ့ Access ကို သူ့ရဲ့ Role နဲ့ ဆုံးဖြတ်ပေးတဲ့ Authorization ပုံစံဖြစ်ပါတယ်။

RBAC ရဲ့ အခြေခံ Flow က

```text
User → Role → Permission → Resource/Action
```

ဖြစ်ပါတယ်။

User တစ်ယောက်ချင်းစီကို Permission တိုက်ရိုက်ပေးမယ့်အစား Permission တွေကို Role မှာစုထားပြီး Role ကို User မှာပေးတာကြောင့် Access Control ကို ပိုမိုစနစ်တကျ စီမံနိုင်ပါတယ်။

RBAC Design မှာ Role Name ထက် Permission ကို တိတိကျကျခွဲခြားခြင်း၊ Least Privilege ကိုလိုက်နာခြင်း၊ UI မှာတင်မဟုတ်ဘဲ Server-side Endpoint မှာ Authorization စစ်ခြင်းနဲ့ Permission ပြောင်းလဲမှု ဘယ်အချိန် Effect ဖြစ်မလဲဆိုတာ သတ်မှတ်ခြင်းတို့က အရေးကြီးပါတယ်။

---

# References

* [Introduction to authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authorization/introduction)
* [Role-based authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/security/authorization/roles)
* [Policy-based authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authorization/policies)
