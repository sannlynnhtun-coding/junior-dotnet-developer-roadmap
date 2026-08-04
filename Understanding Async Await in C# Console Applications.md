# async/await ဆိုတာဘာလဲ?

`async` နဲ့ `await` ဆိုတာ C# မှာ **Asynchronous Code** ကို ပုံမှန် Code Flow လို ဖတ်ရလွယ်အောင် ရေးနိုင်စေတဲ့ Keyword နှစ်ခုဖြစ်ပါတယ်။

Application တစ်ခုက

* Database က Data ပြန်လာတာ
* External API က Response ပြန်လာတာ
* File ဖတ်ပြီးတာ
* Timer ပြည့်တာ

စတဲ့ I/O အလုပ်တစ်ခု ပြီးသွားမယ့်အချိန်ကို စောင့်နေရတဲ့အခါ Thread ကို ပိတ်ထားမယ့်အစား တခြားအလုပ်ကို ဆက်လုပ်နိုင်အောင် `async/await` ကို အသုံးပြုပါတယ်။

အဓိကရည်ရွယ်ချက်က

```text
စောင့်နေရတဲ့အချိန်မှာ Thread ကို မပိတ်ထားဖို့
```

ဖြစ်ပါတယ်။

`async/await` သုံးလိုက်တာနဲ့ အလုပ်တိုင်း ပိုမြန်သွားတာ မဟုတ်ပါဘူး။ စောင့်ဆိုင်းနေတဲ့အချိန်ကို ပိုကောင်းကောင်း အသုံးချနိုင်တာဖြစ်ပါတယ်။

---

# Synchronous နဲ့ Asynchronous ကွာခြားချက်

## Synchronous

အလုပ်တစ်ခုပြီးမှ နောက်အလုပ်တစ်ခုကို ဆက်လုပ်ပါတယ်။

```text
Start
  │
  ▼
Database ကိုခေါ်
  │
  ▼
Data ပြန်လာတဲ့အထိ Thread က စောင့်
  │
  ▼
နောက်အလုပ်ကို ဆက်လုပ်
```

## Asynchronous

စောင့်ရမယ့်အလုပ်ကို စတင်ပြီး Thread ကို လွှတ်ပေးထားပါတယ်။ အလုပ်ပြီးသွားတဲ့အခါ ကျန်တဲ့ Code ကို ဆက်လုပ်ပါတယ်။

```text
Start
  │
  ▼
Database ကိုခေါ်
  │
  ▼
စောင့်နေစဉ် Thread ကို လွှတ်ပေး
  │
  ├──► တခြားအလုပ်ကို လုပ်နိုင်
  │
  ▼
Data ပြန်လာရင် ကျန်တဲ့ Code ကို ဆက်လုပ်
```

| အချက် | Synchronous | Asynchronous |
|---|---|---|
| စောင့်နေစဉ် Thread | Block ဖြစ်နိုင် | Block မဖြစ်ဘဲ လွှတ်ပေးနိုင် |
| Code Flow | တစ်ခုပြီးမှ တစ်ခု | စောင့်နေစဉ် တခြားအလုပ် လုပ်နိုင် |
| သင့်တော်တဲ့နေရာ | မြန်မြန်ပြီးတဲ့ ရိုးရှင်းသောအလုပ် | Database, HTTP, File I/O |
| အဓိကအကျိုးကျေးဇူး | ရိုးရှင်း | Responsiveness နဲ့ Scalability ပိုကောင်း |

---

# async, await နဲ့ Task တို့ရဲ့ တာဝန်

`async/await` ကို နားလည်ဖို့ `Task` ကိုပါ တွဲနားလည်ရပါတယ်။

## async

`async` က Method ထဲမှာ `await` သုံးနိုင်ကြောင်း Compiler ကို ပြောပေးပါတယ်။

```csharp
static async Task MakeBreakfastAsync()
{
    await Task.Delay(1000);
}
```

`async` ထည့်လိုက်တာနဲ့ Method က Background Thread ပေါ် အလိုအလျောက် ရောက်သွားတာ မဟုတ်ပါဘူး။

## Task

`Task` က လက်ရှိလုပ်နေပြီး နောက်မှပြီးမယ့် Asynchronous Operation တစ်ခုကို ကိုယ်စားပြုပါတယ်။

```text
Task = အလုပ်ပြီးသွားမယ့်အချိန်နဲ့ ရလဒ်ကို ကိုယ်စားပြုတဲ့ Object
```

Result မပြန်တဲ့ Async Method က `Task` ပြန်ပါတယ်။

```csharp
static async Task SaveAsync()
{
    await Task.Delay(1000);
}
```

Result ပြန်တဲ့ Async Method က `Task<T>` ပြန်ပါတယ်။

```csharp
static async Task<string> GetNameAsync()
{
    await Task.Delay(1000);
    return "Mg Mg";
}
```

## await

`await` က `Task` ပြီးတဲ့အထိ လက်ရှိ Method ရဲ့ ကျန်တဲ့ Code ကို ခဏရပ်ထားပါတယ်။

Task မပြီးသေးရင် Control ကို Caller ဆီ ပြန်ပေးပြီး Thread ကို Block မလုပ်ပါဘူး။ Task ပြီးသွားရင် `await` နောက်က Code ကို ဆက်လုပ်ပါတယ်။

```csharp
string name = await GetNameAsync();
Console.WriteLine(name);
```

Task က ပြီးပြီးသားဆိုရင်တော့ `await` က ခဏရပ်စရာမလိုဘဲ Result ကို ချက်ချင်းပြန်ပေးပါတယ်။

---

# async/await အလုပ်လုပ်ပုံ

```text
Caller
  │
  ▼
Async Method ကိုခေါ်
  │
  ▼
await မတိုင်ခင် Code ကို ပုံမှန် Run
  │
  ▼
မပြီးသေးတဲ့ Task ကို await တွေ့
  │
  ├──► Control ကို Caller ဆီ ပြန်ပေး
  │
  └──► Task ပြီးတာကို Thread မပိတ်ဘဲ စောင့်
             │
             ▼
      Task ပြီးသွားပြီ
             │
             ▼
      await နောက်က Code ကို ဆက်လုပ်
```

Compiler က Async Method ကို State Machine အဖြစ် ပြောင်းပေးပြီး `await` မတိုင်ခင်နဲ့ `await` နောက်ပိုင်း Code Flow ကို စီမံပေးပါတယ်။ Developer က Callback တွေ ကိုယ်တိုင်ဆက်ရေးစရာမလိုဘဲ ပုံမှန် Code လို ရေးနိုင်ပါတယ်။

---

# Real World Example

မနက်စာပြင်တဲ့အခါ ပေါင်မုန့်မီးကင်တာနဲ့ ကော်ဖီဖျော်တာကို စဉ်းစားကြည့်ပါမယ်။

## Synchronous ပုံစံ

1. ပေါင်မုန့်ကို မီးကင်စက်ထဲထည့်မယ်။
2. ပေါင်မုန့်ကျက်တဲ့အထိ စက်ရှေ့မှာ စောင့်မယ်။
3. ပေါင်မုန့်ကျက်ပြီးမှ ကော်ဖီဖျော်မယ်။

စောင့်နေတဲ့အချိန်မှာ တခြားအလုပ် မလုပ်ဖြစ်ပါဘူး။

## Asynchronous ပုံစံ

1. ပေါင်မုန့်ကို မီးကင်စက်ထဲထည့်ပြီး အလုပ်စမယ်။
2. ပေါင်မုန့်ကျက်တာကို ရပ်စောင့်မနေဘဲ ကော်ဖီဖျော်မယ်။
3. ကော်ဖီဖျော်ပြီးရင် ပေါင်မုန့်ကျက်တာကို `await` လုပ်မယ်။
4. ပေါင်မုန့်ကျက်သွားရင် မနက်စာကို ဆက်ပြင်မယ်။

```text
ToastBreadAsync စတင်
        │
        ├──► ပေါင်မုန့် မီးကင်နေတယ်
        │
        ├──► ကြားထဲမှာ ကော်ဖီဖျော်တယ်
        │
        ▼
await toastTask
        │
        ▼
မနက်စာ အဆင်သင့်ဖြစ်တယ်
```

---

# Console Application Example

`Task.Delay` နဲ့ အချိန်ကြာတဲ့ I/O Operation တစ်ခုကို တုပထားပါမယ်။

```csharp
using System;
using System.Threading.Tasks;

internal static class Program
{
    private static async Task Main()
    {
        Console.WriteLine("မနက်စာ စပြင်ပါမယ်...");

        Task<string> toastTask = ToastBreadAsync();

        PourCoffee();

        string toast = await toastTask;

        Console.WriteLine(toast);
        Console.WriteLine("မနက်စာ အဆင်သင့်ဖြစ်ပါပြီ!");
    }

    private static async Task<string> ToastBreadAsync()
    {
        Console.WriteLine("ပေါင်မုန့် စကင်ပါပြီ...");

        await Task.Delay(3000);

        return "ပေါင်မုန့် ကျက်ပါပြီ!";
    }

    private static void PourCoffee()
    {
        Console.WriteLine("ကော်ဖီ ဖျော်နေပါပြီ...");
    }
}
```

ဒီ Code ကို Run လိုက်ရင်

```text
မနက်စာ စပြင်ပါမယ်...
ပေါင်မုန့် စကင်ပါပြီ...
ကော်ဖီ ဖျော်နေပါပြီ...
(၃ စက္ကန့်ခန့်ကြာပြီးနောက်)
ပေါင်မုန့် ကျက်ပါပြီ!
မနက်စာ အဆင်သင့်ဖြစ်ပါပြီ!
```

လို့ တွေ့ရပါမယ်။

---

# Code Flow ကို အဆင့်လိုက်ကြည့်မယ်

## Step 1 - Async Method ကို စတင်တယ်

```csharp
Task<string> toastTask = ToastBreadAsync();
```

`ToastBreadAsync()` က `await Task.Delay(3000)` အထိ Run ပြီး မပြီးသေးတဲ့ `Task<string>` ကို ပြန်ပေးပါတယ်။

## Step 2 - တခြားအလုပ်ကို ဆက်လုပ်တယ်

```csharp
PourCoffee();
```

ပေါင်မုန့်ကျက်တာကို ဒီနေရာမှာ မစောင့်သေးတဲ့အတွက် ကော်ဖီဖျော်တဲ့ Method ကို ဆက်လုပ်ပါတယ်။

## Step 3 - Result လိုတဲ့နေရာမှာ await လုပ်တယ်

```csharp
string toast = await toastTask;
```

Task မပြီးသေးရင် `Main` Method က ခဏရပ်ပြီး Control ကို Runtime ဆီ ပြန်ပေးပါတယ်။ Task ပြီးသွားရင် Result ကို `toast` ထဲထည့်ပြီး နောက် Code ကို ဆက်လုပ်ပါတယ်။

---

# Task.Delay နဲ့ Thread.Sleep ကွာခြားချက်

```csharp
Thread.Sleep(3000);
```

`Thread.Sleep` က လက်ရှိ Thread ကို ၃ စက္ကန့် Block လုပ်ထားပါတယ်။ အဲ့ဒီ Thread က ကြားထဲမှာ တခြားအလုပ် မလုပ်နိုင်ပါဘူး။

```csharp
await Task.Delay(3000);
```

`Task.Delay` က Timer ပြည့်တာကို Asynchronously စောင့်ပြီး Thread ကို Block မလုပ်ပါဘူး။

| Method | Thread ကို Block လုပ်လား? | အသုံးများတဲ့နေရာ |
|---|---|---|
| `Thread.Sleep` | လုပ်တယ် | တကယ် Thread ကို ခဏရပ်ချင်တဲ့ ရှားပါးအခြေအနေ |
| `Task.Delay` | မလုပ်ဘူး | Async Delay, Retry Delay, Simulation |

Async Method ထဲမှာ စောင့်ဆိုင်းဖို့ဆိုရင် အများအားဖြင့် `Task.Delay` ကို သုံးသင့်ပါတယ်။

---

# Async Method Return Types

## Task

အလုပ်ပြီးဆုံးမှုကိုပဲ စောင့်ချင်ပြီး Result မလိုတဲ့အခါ သုံးပါတယ်။

```csharp
static async Task SendEmailAsync()
{
    await Task.Delay(1000);
}
```

## Task<T>

အလုပ်ပြီးသွားတဲ့အခါ Result တစ်ခု ပြန်လိုတဲ့အခါ သုံးပါတယ်။

```csharp
static async Task<int> GetProductCountAsync()
{
    await Task.Delay(1000);
    return 10;
}
```

Caller မှာ `await` လုပ်ရင် `T` Type Result ကို ရပါတယ်။

```csharp
int count = await GetProductCountAsync();
```

## void

`async void` ကို Event Handler ကလွဲပြီး မသုံးသင့်ပါဘူး။ Caller က စောင့်လို့မရသလို Exception ကိုလည်း ပုံမှန် `Task` လို ကိုင်တွယ်လို့မရပါဘူး။

```csharp
private async void Button_Click(object sender, EventArgs e)
{
    await SaveAsync();
}
```

ပုံမှန် Async Method တွေအတွက် `Task` သို့မဟုတ် `Task<T>` ကို သုံးပါ။

---

# Sequential နဲ့ Concurrent Async

`await` ရေးထားရုံနဲ့ အလုပ်နှစ်ခုက တစ်ပြိုင်နက် အလိုအလျောက် စတင်တာ မဟုတ်ပါဘူး။

## Sequential

```csharp
User user = await GetUserAsync();
Order order = await GetOrderAsync();
```

`GetUserAsync()` ပြီးမှ `GetOrderAsync()` ကို စတင်ပါတယ်။

```text
GetUserAsync  ─────────► ပြီး
                         GetOrderAsync ─────────► ပြီး
```

## Concurrent

အလုပ်နှစ်ခုက တစ်ခုနဲ့တစ်ခု မမှီခိုဘူးဆိုရင် အရင်စတင်ထားပြီး `Task.WhenAll` နဲ့ စောင့်နိုင်ပါတယ်။

```csharp
Task<User> userTask = GetUserAsync();
Task<Order> orderTask = GetOrderAsync();

await Task.WhenAll(userTask, orderTask);

User user = await userTask;
Order order = await orderTask;
```

```text
GetUserAsync   ─────────► ပြီး
GetOrderAsync  ─────────► ပြီး
```

`Task.WhenAll` က Thread အသစ်ဖန်တီးပေးတာ မဟုတ်ပါဘူး။ စတင်ထားပြီးသား Task အားလုံး ပြီးဆုံးတာကို စောင့်ပေးတာဖြစ်ပါတယ်။ အလုပ်တွေ တစ်ခုနဲ့တစ်ခု မှီခိုနေရင်တော့ Sequential ပုံစံကို သုံးရပါတယ်။

---

# I/O-bound နဲ့ CPU-bound

## I/O-bound

တခြား System တစ်ခုရဲ့ Result ကို စောင့်ရတဲ့အလုပ်ဖြစ်ပါတယ်။

* Database Query
* HTTP Request
* File Read/Write
* Network Operation

ဒီလိုအလုပ်တွေမှာ Async API ကို `await` လုပ်ပါ။ `Task.Run` ထပ်ပတ်စရာမလိုပါဘူး။

```csharp
string content = await File.ReadAllTextAsync("data.txt");
```

## CPU-bound

CPU နဲ့ တွက်ချက်နေရတဲ့အလုပ်ဖြစ်ပါတယ်။

* Image Processing
* Large Calculation
* Data Compression

`async/await` တစ်ခုတည်းက CPU အလုပ်ကို ပိုမြန်အောင် မလုပ်ပေးပါဘူး။ UI App မှာ Screen မခဲအောင် လိုအပ်ရင် CPU-bound အလုပ်ကို `Task.Run` နဲ့ Background Thread ပေါ် ရွှေ့နိုင်ပါတယ်။

```csharp
int result = await Task.Run(() => Calculate());
```

ASP.NET Core မှာ I/O-bound အလုပ်တိုင်းကို `Task.Run` နဲ့ ထပ်ပတ်တာက Thread Pool ကို အလဟဿသုံးစေနိုင်လို့ မလုပ်သင့်ပါဘူး။

---

# Exception Handling

Async Method ထဲက Exception ကို `await` လုပ်တဲ့နေရာမှာ ပုံမှန် `try/catch` နဲ့ ကိုင်တွယ်နိုင်ပါတယ်။

```csharp
try
{
    string data = await LoadDataAsync();
    Console.WriteLine(data);
}
catch (IOException ex)
{
    Console.WriteLine(ex.Message);
}
```

Task မပြီးခင် Exception ဖြစ်ရင် အဲ့ဒီ Exception ကို Task ထဲမှာ သိမ်းထားပြီး `await` လုပ်တဲ့အခါ ပြန် Throw လုပ်ပါတယ်။

---

# CancellationToken လက်ဆင့်ကမ်းခြင်း

အချိန်ကြာနိုင်တဲ့ Async Method တွေကို Cancel လုပ်နိုင်ဖို့ `CancellationToken` လက်ခံပြီး Async API ဆီ ဆက်ပို့သင့်ပါတယ်။

```csharp
static async Task WaitAsync(
    CancellationToken cancellationToken)
{
    await Task.Delay(5000, cancellationToken);
}
```

ASP.NET Core မှာ User က Request ကို Cancel လုပ်လိုက်ရင် Database သို့မဟုတ် External API Call ကိုပါ ရပ်နိုင်အောင် `CancellationToken` ကို အောက် Layer အထိ လက်ဆင့်ကမ်းသင့်ပါတယ်။

---

# သတိထားရမယ့်အချက်များ

## 1. .Result နဲ့ .Wait() ကို မသုံးပါနဲ့

```csharp
string data = GetDataAsync().Result;
GetDataAsync().Wait();
```

ဒီနှစ်ခုက Thread ကို Block လုပ်ပါတယ်။ UI App နဲ့ အချို့သော Environment တွေမှာ Deadlock ဖြစ်စေနိုင်ပါတယ်။ Async Flow တစ်လျှောက် `await` ကို ဆက်သုံးပါ။

```csharp
string data = await GetDataAsync();
```

## 2. async void ကို ရှောင်ပါ

Event Handler မဟုတ်ရင် `async void` အစား `async Task` သုံးပါ။ ဒါမှ Caller က စောင့်နိုင်ပြီး Exception ကို ကိုင်တွယ်နိုင်ပါတယ်။

## 3. Task ကို မေ့မထားပါနဲ့

```csharp
SaveAsync();
```

`await` မလုပ်ဘဲထားရင် Method မပြီးခင် Caller က ဆက်သွားနိုင်ပြီး Exception ကို လွတ်သွားနိုင်ပါတယ်။ Fire-and-forget ကို တကယ်လိုအပ်တဲ့အခါမှ Lifecycle နဲ့ Error Handling ကို သီးသန့်စီမံပြီး အသုံးပြုသင့်ပါတယ်။

## 4. Async Method Name ကို Async နဲ့ အဆုံးသတ်ပါ

```csharp
GetProductAsync()
SaveOrderAsync()
SendEmailAsync()
```

ဒါက Compiler Rule မဟုတ်ပေမယ့် .NET Naming Convention ဖြစ်ပြီး Method က Async ဖြစ်ကြောင်း ချက်ချင်းသိနိုင်စေပါတယ်။

## 5. အလုပ်တိုင်းကို Async မလုပ်ပါနဲ့

ရိုးရှင်းတဲ့ Calculation နဲ့ Memory ထဲမှာ ချက်ချင်းပြီးနိုင်တဲ့အလုပ်ကို အကြောင်းမဲ့ `async` မလုပ်သင့်ပါဘူး။ တကယ်စောင့်ရတဲ့ Async Operation ရှိတဲ့အခါ သုံးပါ။

---

# ASP.NET Core မှာ ဘာကြောင့်အရေးကြီးလဲ?

Web Server တစ်ခုမှာ User အများကြီးက တစ်ပြိုင်နက် Request ပို့နိုင်ပါတယ်။

```text
Request 1 ──► Database ကို စောင့်
Request 2 ──► External API ကို စောင့်
Request 3 ──► File ကို စောင့်
```

Synchronous I/O သုံးရင် Request တစ်ခုစီအတွက် Thread က စောင့်ပြီး Block ဖြစ်နေနိုင်ပါတယ်။ Async I/O သုံးရင် စောင့်နေစဉ် Thread ကို တခြား Request အတွက် ပြန်အသုံးပြုနိုင်ပါတယ်။

```text
HTTP Request
     │
     ▼
Controller
     │ await
     ▼
Service
     │ await
     ▼
Database / External API
     │
     ▼
Result ပြန်လာရင် Response ဆက်တည်ဆောက်
```

ဒါကြောင့် `async/await` က Request တစ်ခုရဲ့ Database Query ကို ပိုမြန်စေတာမဟုတ်ပေမယ့် Server က Concurrent Request များများကို ပိုကောင်းကောင်း လက်ခံနိုင်စေပါတယ်။

---

# Summary

`async/await` ဆိုတာ C# မှာ Asynchronous Code ကို ဖတ်ရလွယ်ပြီး ထိန်းသိမ်းရလွယ်အောင် ရေးနိုင်စေတဲ့ Language Feature ဖြစ်ပါတယ်။

* `async` က Method ထဲမှာ `await` သုံးနိုင်စေပါတယ်။
* `Task` သို့မဟုတ် `Task<T>` က နောက်မှပြီးမယ့် အလုပ်ကို ကိုယ်စားပြုပါတယ်။
* `await` က Task ပြီးတာကို Thread မပိတ်ဘဲ စောင့်ပြီး ပြီးသွားရင် ကျန်တဲ့ Code ကို ဆက်လုပ်ပါတယ်။
* I/O-bound အလုပ်တွေမှာ Async API ကို တိုက်ရိုက် `await` လုပ်သင့်ပါတယ်။
* `async/await` က အလုပ်ကို အလိုအလျောက် ပိုမြန်စေတာ၊ Thread အသစ်ဖန်တီးတာ မဟုတ်ပါဘူး။
* `.Result`, `.Wait()` နဲ့ မလိုအပ်တဲ့ `async void` ကို ရှောင်သင့်ပါတယ်။

မှတ်ထားရမယ့် Flow ကတော့

```text
Start Task → await → Thread ကို လွှတ်ပေး → Task ပြီး → Code ဆက်လုပ်
```

ဖြစ်ပါတယ်။

---

# ဆက်လက်လေ့လာရန်

* [Microsoft Learn - Asynchronous programming with async and await](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
* [Microsoft Learn - Asynchronous programming scenarios](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios)
* [Microsoft Learn - Async return types](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types)
* [Microsoft Learn - await operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await)
