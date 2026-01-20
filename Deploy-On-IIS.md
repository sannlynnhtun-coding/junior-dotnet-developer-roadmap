# Part A — Windows 10/11 မှာ IIS Install လုပ်နည်း

## 1) Windows Features ကိုဖွင့်

IIS က Windows Optional Feature တစ်ခုပါ။

- **Control Panel** → **Programs** → **Programs and Features**
- ဘေးဘက်မှာ **Turn Windows features on or off** ကိုနှိပ်ပါ
    
    (သို့) Run dialog (Win + R) → `optionalfeatures.exe` လို့ရိုက်ပြီး Enter (Windows Features ပေါ်လာမယ်)
    

## 2) IIS ကို Enable လုပ်

Windows Features dialog ထဲမှာ—

✅ **Internet Information Services** ကို check လုပ်ပါ

ပြီးရင် **OK** နှိပ် → Windows က components install လုပ်ပြီးပြီဆိုရင် restart တောင်းရင် restart လုပ်ပါ။

## 3) IIS ရဲ့ မဖြစ်မနေလိုတဲ့ sub-features (Recommended)

IIS ကို check လုပ်တဲ့အချိန် Expand လုပ်ပြီး အောက်ပါ items တွေကိုလည်း သေချာ check လုပ်ထားရင် အဆင်ပြေပါတယ်—

- **Web Management Tools**
    - ✅ IIS Management Console
- **World Wide Web Services**
    - ✅ Application Development Features (default လောက် OK)
    - ✅ Common HTTP Features

> သင်တန်း/Dev machine အတွက်တော့ အပေါ်ကလောက်နဲ့ လုံလောက်ပါတယ်။
> 

## 4) IIS Manager ဖွင့်ပြီး စမ်းသပ်

Start menu မှာ **IIS Manager** လို့ရှာပြီးဖွင့်ပါ။

Browser မှာ `http://localhost` သွားကြည့်ရင် IIS default page ပေါ်လာရင် IIS OK ဖြစ်ပါပြီ။

---

# Part B — .NET 8 Hosting Bundle Install (IIS မှာ ASP.NET Core App Run ဖို့)

ASP.NET Core app ကို IIS ပေါ် run မယ်ဆိုရင် **ASP.NET Core Module (ANCM)** လိုပါတယ်။

Microsoft က **.NET Hosting Bundle** ထဲမှာ Runtime + ANCM ကိုအတူတကွထည့်ပေးထားပါတယ်။ ([Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/hosting-bundle?view=aspnetcore-8.0))

## 1) .NET 8 Hosting Bundle ကို Download

Microsoft Learn / .NET download page ကနေ **.NET 8 Hosting Bundle** ကို download လုပ်ပါ (Windows အတွက် recommended)။ ([Microsoft](https://dotnet.microsoft.com/en-us/download/dotnet/8.0))

## 2) Install လုပ်

Downloaded installer ကို Run → Next → Install

Install ပြီးရင် **IIS ကို restart** လုပ်ပါ (အရေးကြီး)

- CMD (Admin) → `iisreset`
    
    (သို့) Server/PC restart
    

> Hosting bundle က ASP.NET Core Module + Runtime ကို install လုပ်ပေးတာကြောင့် IIS restart လုပ်ရင် changes apply ဖြစ်ပါတယ်။ (Microsoft Learn)
> 

---

# Part C — .NET 8 Web App ကို IIS ပေါ် Deploy လုပ်နည်း (Step-by-step)

## 1) App ကို Publish လုပ် (Folder publish)

Visual Studio မှာ—

- Project → Right click → **Publish**
- **Folder** ကိုရွေး
- Target folder တစ်ခု သတ်မှတ် (ဥပမာ `C:\Publish\MyApp`)
- **Publish** နှိပ်

> IIS အတွက် သာမန်အားဖြင့် Framework-dependent publish လုပ်ပြီး Hosting Bundle ရှိရင် run လို့ရပါတယ်။
> 

---

## 2) IIS Manager မှာ Website/Create App ပြုလုပ်

IIS Manager ဖွင့် → ဘယ်ဘက် tree မှာ

### Option A: New Website (သီးသန့် site)

- **Sites** → Right click → **Add Website…**
    - Site name: `MyDotNet8App`
    - Physical path: publish folder (`C:\Publish\MyApp`)
    - Port: `8080` (ဥပမာ)
- OK

### Option B: Existing site အောက်မှာ Application

- `Default Web Site` → Right click → **Add Application…**
    - Alias: `myapp`
    - Physical path: publish folder
- OK

---

## 3) Application Pool Setting (အရေးကြီး)

ASP.NET Core (.NET 8) app က “ASP.NET Framework” မဟုတ်တဲ့အတွက် app pool ကို **No Managed Code** ထားရပါတယ်။

- IIS Manager → **Application Pools**
- သင့် app pool ကိုရွေး → **Basic Settings…**
- **.NET CLR version** = **No Managed Code**
- OK

> ASP.NET Core app ကို IIS က ANCM တဆင့် Kestrel ကို forward လုပ်တာကြောင့် No Managed Code သုံးတာက standard ပါ။ (Microsoft Learn)
> 

---

## 4) Folder Permission ပေး (IIS မှာ run ဖို့)

App folder ကို IIS worker process က read/execute လုပ်နိုင်ဖို့ permission လိုပါတယ်။

Publish folder (`C:\Publish\MyApp`) ကို right click → Properties → Security → Edit → Add

- `IIS_IUSRS` ကို add
- Allow: Read & Execute, List folder contents, Read

---

## 5) Run & Test

Browser မှာ—

- New website port သုံးထားရင်: `http://localhost:8080`
- Application alias သုံးထားရင်: `http://localhost/myapp`

---

# Part D — Common Errors / Troubleshooting (အမြန်ပြင်နိုင်အောင်)

## 1) 500.31 / 500.30 (Runtime / Hosting bundle မတင်ထား)

- Hosting Bundle မရှိ/မမှန်တာများပါတယ် → Hosting Bundle (.NET 8) install ပြန်စစ်ပါ ([Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/hosting-bundle?view=aspnetcore-10.0&utm_source=chatgpt.com))
- Install ပြီးရင် `iisreset`

## 2) 502.5 Process Failure

- App start မဖြစ်လို့ crash ဖြစ်တတ်
- Event Viewer (Windows Logs → Application) စစ်ပါ
- Publish output folder မှာ `web.config` ရှိ/မရှိ စစ်ပါ (publish လုပ်ရင် auto ထွက်ပါတယ်)

## 3) Logs ဖွင့်ပြီး error အတိအကျကြည့်ချင်ရင်

`web.config` မှာ stdout log enable (temporary only) လုပ်လို့ရတတ်ပါတယ် (production မှာ disk fill မဖြစ်အောင် ပိတ်ထားတာကောင်းပါတယ်)

> IIS hosting & deploy guide မှာ Troubleshooting/logging လမ်းညွှန်တွေပါပါတယ်။
> 

---

# Quick Checklist

✅ IIS enabled (IIS Manager ရ)

✅ IIS Management Console installed

✅ .NET 8 Hosting Bundle installed + `iisreset` လုပ်ပြီး

✅ App published to folder

✅ IIS site/app points to that folder

✅ App Pool = No Managed Code

✅ Folder permission ok

---
