# N-Layered Architecture ဆိုတာဘာလဲ?

N-Layered Architecture ဆိုသည်မှာ Software Application တစ်ခုကို **တာဝန် (Responsibility)** အလိုက် Layer (အလွှာ) များအဖြစ် ခွဲခြားရေးသားသည့် Software Architecture Pattern တစ်ခုဖြစ်ပါတယ်။

"N" ဆိုတာ Number ကို ကိုယ်စားပြုတာဖြစ်ပြီး Layer ဘယ်နှစ်ခုရှိမယ်ဆိုတာကို သတ်မှတ်ထားတာမဟုတ်ပါဘူး။

Project ရဲ့ အရွယ်အစားနဲ့ ရှုပ်ထွေးမှုပေါ်မူတည်ပြီး

* 3 Layers
* 4 Layers
* 5 Layers
* 6 Layers

စသဖြင့် လိုအပ်သလို တိုးချဲ့နိုင်ပါတယ်။

ဥပမာ...

Project အသေးဆိုရင်

* Presentation
* Business
* Data

၃ လွှာနဲ့တင် လုံလောက်နိုင်ပါတယ်။

ဒါပေမယ့် Enterprise Project ကြီးတွေမှာတော့

* Presentation
* API
* Application
* Domain
* Infrastructure
* Database

လိုမျိုး Layer အများကြီး ခွဲရေးကြပါတယ်။

---

## ဘာကြောင့် Layer ခွဲရေးရတာလဲ?

Layer မခွဲဘဲ Project တစ်ခုလုံးကို File တစ်ခု၊ Folder တစ်ခုထဲမှာ ရေးမယ်ဆိုရင်

* Code တွေ ရှုပ်ထွေးလာမယ်
* Bug ရှာရခက်လာမယ်
* Feature အသစ်ထည့်ရခက်လာမယ်
* Developer များလာရင် အလုပ်လုပ်ရခက်လာမယ်

ဒါကြောင့် တာဝန်အလိုက် Layer ခွဲပြီးရေးကြတာ ဖြစ်ပါတယ်။

Layer ခွဲရေးထားခြင်းကြောင့်

* Maintainability ကောင်းလာတယ်
* Scalability ကောင်းလာတယ်
* Reusability ရလာတယ်
* Testing လုပ်ရလွယ်လာတယ်
* Team နဲ့အလုပ်လုပ်ရ ပိုလွယ်လာတယ်

---

# 3-Layered Architecture

Beginner တွေအတွက် အများဆုံးတွေ့ရတဲ့ Architecture ကတော့ 3-Layered Architecture ဖြစ်ပါတယ်။

ဒီ Architecture မှာ

1. Presentation Layer
2. Business Logic Layer
3. Data Access Layer

ဆိုပြီး Layer သုံးခု ပါဝင်ပါတယ်။

---

# 1. Presentation Layer

ဒီ Layer က User နဲ့ တိုက်ရိုက်ထိတွေ့တဲ့ Layer ဖြစ်ပါတယ်။

User မြင်ရတဲ့

* Website
* Mobile App
* Desktop Application

အားလုံးက ဒီ Layer ထဲမှာ ပါဝင်ပါတယ်။

ဥပမာ

User က Login Button ကို နှိပ်လိုက်တယ်ဆိုပါစို့။

Presentation Layer ရဲ့ အလုပ်က

* Username ကိုယူတယ်
* Password ကိုယူတယ်
* Business Layer ကို ပို့ပေးတယ်

ဒါပဲဖြစ်ပါတယ်။

ဒီ Layer က

Password မှန်မမှန်

Database ထဲသွားစစ်တာ

Business Logic တွက်တာ

မလုပ်သင့်ပါဘူး။

---

## Presentation Layer ရဲ့တာဝန်

* User Input လက်ခံခြင်း
* Result ပြသခြင်း
* Error Message ပြခြင်း
* Loading ပြခြင်း
* Business Layer ကို Request ပို့ခြင်း

---

# 2. Business Logic Layer (BLL)

ဒီ Layer ကတော့ Application ရဲ့ ဦးနှောက်လို့ ပြောလို့ရပါတယ်။

User က Request ပို့လာတာကို

Business Rule အတိုင်း စဉ်းစားဆုံးဖြတ်ပေးတဲ့ Layer ဖြစ်ပါတယ်။

ဥပမာ...

Login လုပ်မယ်ဆိုရင်

ဒီ Layer က

* Username ရှိလား
* Password မှန်လား
* Account Lock ဖြစ်နေလား
* User Active ဖြစ်လား
* Login လုပ်ခွင့်ရှိလား

ဆိုတာတွေကို စစ်ပေးပါတယ်။

Database ထဲက Data လိုရင်တော့ DAL ကို ခေါ်ပါတယ်။

---

## Business Layer ရဲ့တာဝန်

* Validation
* Business Rules
* Calculation
* Workflow
* Authorization
* DAL ကိုခေါ်ပြီး Data ယူခြင်း

---

# 3. Data Access Layer (DAL)

ဒီ Layer က Database နဲ့ တိုက်ရိုက်အလုပ်လုပ်တဲ့ Layer ဖြစ်ပါတယ်။

Business Layer က

"User တစ်ယောက်ကိုရှာပေး"

လို့ ပြောရင်

DAL က

SQL Server ထဲသွားပြီး

Data ကိုယူလာပေးပါတယ်။

ဒီ Layer မှာ

* SQL
* Entity Framework Core
* Dapper
* ADO.NET

စတာတွေကို အသုံးပြုနိုင်ပါတယ်။

---

## DAL ရဲ့တာဝန်

* Insert
* Update
* Delete
* Select
* Database Connection
* Transaction

---

# Data Flow

Application တစ်ခုမှာ Request တစ်ခု ဘယ်လိုစီးဆင်းလဲ?

```text
User
 │
 ▼
Presentation Layer
 │
 ▼
Business Layer
 │
 ▼
Data Access Layer
 │
 ▼
Database
```

Database က Result ပြန်လာရင်

```text
Database
 │
 ▲
Data Access Layer
 │
 ▲
Business Layer
 │
 ▲
Presentation Layer
 │
 ▲
User
```

ဒီလို Layer အလိုက်သာ သွားလာပါတယ်။

Presentation Layer က Database ကို တိုက်ရိုက်မခေါ်သင့်ပါဘူး။

---

# Real World Example (Internet Banking)

ဥပမာ...

Internet Banking မှာ Transfer Money လုပ်တယ်ဆိုပါစို့။

### Step 1

User က

Transfer Button ကို နှိပ်တယ်။

ဒါက Presentation Layer ဖြစ်ပါတယ်။

---

### Step 2

Business Layer က

* Sender Account ရှိလား
* Receiver Account ရှိလား
* Balance လုံလား
* Daily Limit ကျော်လား
* Account Freeze ဖြစ်နေလား

ဆိုတာတွေ စစ်ပါတယ်။

---

### Step 3

Database ထဲက

* Sender Balance
* Receiver Account
* Transaction History

တွေကို DAL က သွားယူပါတယ်။

---

### Step 4

Business Layer က

ငွေလွှဲလို့ရတယ်ဆိုရင်

DAL ကို

* Sender Balance Update
* Receiver Balance Update
* Transaction Insert

လုပ်ခိုင်းပါတယ်။

---

### Step 5

Presentation Layer က

"Transfer Successful"

ဆိုတဲ့ Message ကို User ကိုပြပေးပါတယ်။

---

# စားသောက်ဆိုင် ဥပမာ

ဒီ Architecture ကို စားသောက်ဆိုင်နဲ့ နှိုင်းကြည့်ရင် ပိုနားလည်လွယ်ပါတယ်။

### Customer

ဟင်းမှာတယ်။

↓

### Waiter (Presentation Layer)

Customer မှာတဲ့ Order ကိုယူတယ်။

↓

### Chef (Business Layer)

ဟင်းချက်နည်းအတိုင်း ချက်တယ်။

↓

### Store Room (Data Access Layer)

လိုအပ်တဲ့ အသား၊ ဟင်းသီးဟင်းရွက်၊ ဆီ၊ ဆားတွေကို သွားယူတယ်။

↓

### Chef

ဟင်းချက်ပြီး

↓

### Waiter

Customer ဆီ ပြန်ပို့တယ်။

Customer က Store Room ထဲကို တိုက်ရိုက်သွားယူလို့ မရသလို Waiter ကလည်း Store Room ထဲဝင်ပြီး ပစ္စည်းယူတာထက် Chef ရဲ့ လုပ်ငန်းစဉ်အတိုင်း ဆောင်ရွက်တာက ပိုစနစ်ကျပါတယ်။ Software မှာလည်း Layer တစ်ခုက နောက်တစ်ခုကို ကျော်ပြီး မသွားသင့်ပါဘူး။

---

# N-Layered Architecture ရဲ့ အားသာချက်များ

* **Separation of Concerns** – Layer တစ်ခုစီမှာ သီးခြားတာဝန်ရှိတာကြောင့် Code ကို နားလည်ရလွယ်ပြီး Developer တွေ အလုပ်ခွဲလုပ်ရလည်း လွယ်ပါတယ်။
* **Maintainability** – UI ပြောင်းချင်ရင် UI Layer ကိုပဲ ပြင်နိုင်ပြီး Business Logic ကို ထိစရာမလိုတဲ့ အခြေအနေတွေ များပါတယ်။
* **Scalability** – Project ကြီးလာတဲ့အခါ Layer အသစ်တွေ ထပ်တိုးနိုင်ပြီး အစိတ်အပိုင်းလိုက် Scale လုပ်နိုင်ပါတယ်။
* **Testability** – Business Logic ကို UI နဲ့ Database မပါဘဲ Unit Test ရေးနိုင်တာကြောင့် Testing လုပ်ရ ပိုလွယ်ပါတယ်။
* **Reusability** – Business Logic နဲ့ Data Access ကို Application တစ်ခုထက်ပိုပြီး ပြန်လည်အသုံးပြုနိုင်ပါတယ်။

---

# Summary

N-Layered Architecture ဆိုတာ Application တစ်ခုကို **တာဝန်အလိုက် Layer (အလွှာ)** များခွဲပြီး တည်ဆောက်တဲ့ Software Architecture Pattern တစ်ခုဖြစ်ပါတယ်။

Beginner Project တွေမှာတော့ **Presentation Layer → Business Logic Layer → Data Access Layer** ဆိုတဲ့ 3-Layered Architecture ကို အများဆုံးတွေ့ရပါတယ်။ Enterprise Project တွေမှာတော့ လိုအပ်ချက်အရ **Application Layer**, **Domain Layer**, **Infrastructure Layer**, **API Layer** စတဲ့ Layer တွေကို ထပ်မံခွဲထုတ်ပြီး ပိုမိုစနစ်ကျအောင် တည်ဆောက်ကြပါတယ်။

အရေးကြီးဆုံးအချက်ကတော့ **Layer တစ်ခုချင်းစီမှာ ကိုယ်ပိုင်တာဝန်သာရှိပြီး၊ အလွှာတစ်ခုက နောက်တစ်ခုရဲ့ တာဝန်ကို မယူသင့်တာ** ဖြစ်ပါတယ်။ ဒီလိုတာဝန်ခွဲခြားထားခြင်းကြောင့် Code တွေကို ဖတ်ရှုနားလည်ရ လွယ်ကူလာသလို၊ ပြုပြင်ထိန်းသိမ်းခြင်း၊ Feature အသစ်ထည့်သွင်းခြင်းနဲ့ စနစ်ကို အနာဂတ်မှာ ချဲ့ထွင်ခြင်းတို့ကိုလည်း ပိုမိုလွယ်ကူစေပါတယ်။
