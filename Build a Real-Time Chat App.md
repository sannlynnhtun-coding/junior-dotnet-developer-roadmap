# Build a Real-Time Chat App with ASP.NET Core SignalR

---

### 1. Create the Project

Visual Studio 2022 ကိုဖွင့်ပြီး Create a new project ကို နှိပ်ပါ။

ASP.NET Core Web App (Model-View-Controller) ကို ရွေးပြီး Next ကို နှိပ်ပါ။

ကျွန်တော်တို့ရဲ့ project ကို `DotNetTrainingBatch0.SignalRChat`လို့ နာမည်ပေးပါမယ်။

Framework အတွက် .NET 8.0 (LTS) ကို ရွေးပြီး Create ကို နှိပ်လိုက်ပါ။

---

### 2. Add the SignalR Client Library

**Solution Explorer** ထဲမှာ ကျွန်တော်တို့ရဲ့ project ကို right-click နှိပ်ပြီး **Add > Client-Side Library** ကို ရွေးပါ။

- **Provider:** unpkg
- **Library:** @microsoft/signalr@latest
- **Choose specific files:** `dist/browser/signalr.js` နဲ့ `signalr.min.js` ကို ရွေးပါ။
- **Target Location:** `wwwroot/js/signalr/` လို့ သတ်မှတ်ပြီး **Install** ကို နှိပ်လိုက်ပါ။

---

### 3. Create the Hub

Project ထဲမှာ `Hubs` ဆိုတဲ့ **Folder** အသစ်တစ်ခု ဆောက်ပါမယ်။ ပြီးရင် အဲ့ဒီ Folder ထဲမှာ `ChatHub.cs` ဆိုတဲ့ class file တစ်ခု ဆောက်ပါမယ်။

```csharp
using Microsoft.AspNetCore.SignalR;

namespace DotNetTrainingBatch0.SignalRChat.Hubs
{
    public class ChatHub : Hub
    {
        // Client ဘက်ကနေ ဒီ method ကို ခေါ်လိုက်တဲ့အခါ Server က message ကို လက်ခံရရှိဖို့ ဖြစ်ပါတယ်
        public async Task ServerReceiveMessage(string user, string message)
        {
            // Server ကနေ ချိတ်ဆက်ထားတဲ့ Client တွေအားလုံးဆီကို message ပြန်လည်ဖြန့်ဝေပေးမှာ ဖြစ်ပါတယ်
            await Clients.All.SendAsync("ClientReceiveMessage", user, message);
        }
    }
}
```

---

### 4. Configure SignalR in Program.cs

ကျွန်တော်တို့ရဲ့ `Program.cs` file ကို အောက်ပါအတိုင်း ပြင်ဆင်ရေးသားပါမယ်။

```csharp
using DotNetTrainingBatch0.SignalRChat.Hubs; // ကျွန်တော်တို့ရဲ့ ChatHub ကို သုံးဖို့ ထည့်သွင်းတာ ဖြစ်ပါတယ်

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();
builder.Services.AddSignalR(); // SignalR service ကို project မှာ ထည့်သွင်းအသုံးပြုဖို့ ဖြစ်ပါတယ်

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

// MVC အတွက် ပုံမှန် route ဖြစ်ပါတယ်
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// SignalR Hub အတွက် route ဖြစ်ပါတယ် (ဒီလိပ်စာကို ပြောင်းလို့ရပါတယ်၊ ဒါပေမဲ့ client ဘက်မှာလည်း တူညီဖို့ လိုအပ်ပါတယ်)
app.MapHub<ChatHub>("/chatHub");

app.Run();
```

---

### 5. Update the Layout for Scripts Section

`Views/Shared/_Layout.cshtml` file ကိုဖွင့်ပြီး file ရဲ့အောက်ဆုံးမှာ `scripts` section ကို အင်္ဂလိပ်အက္ခရာအသေးနဲ့ သုံးထားတာ သေချာအောင်လုပ်ဖို့ လိုအပ်ပါတယ်။

```csharp
    ...
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>

    @await RenderSectionAsync("scripts", required: false)
</body>
</html>
```

---

### 6. Build the UI (Index.cshtml)

`Views/Home/Index.cshtml` file ထဲက ရှိပြီးသား code တွေအားလုံးကို အောက်မှာ ရေးပြပါမယ့် code နဲ့ အစားထိုးလိုက်ပါ။

```csharp
@{
    ViewData["Title"] = "Chat";
}

<div class="container">
    <div class="row p-1">
        <div class="col-1">User</div>
        <div class="col-5"><input type="text" id="userInput" /></div>
    </div>
    <div class="row p-1">
        <div class="col-1">Message</div>
        <div class="col-5"><input type="text" class="w-100" id="messageInput" /></div>
    </div>
    <div class="row p-1">
        <div class="col-6 text-end">
            <input type="button" id="sendButton" value="Send Message" />
        </div>
    </div>
    <div class="row p-1">
        <div class="col-6"><hr /></div>
    </div>
    <div class="row p-1">
        <div class="col-6"><ul id="messagesList"></ul></div>
    </div>
</div>

@section scripts {
    <script src="~/js/signalr/dist/browser/signalr.js"></script>
    <script src="~/js/chat.js"></script>
}
```

---

### 7. Add Client Logic (chat.js)

`wwwroot/js/` Folder အောက်မှာ `chat.js` ဆိုတဲ့ file အသစ်တစ်ခု ဆောက်ပြီး အောက်က code တွေကို ရေးပြပါမယ်။

```csharp
"use strict";

// Program.cs မှာ သတ်မှတ်ခဲ့တဲ့ hub URL နဲ့ တူညီဖို့ လိုအပ်ပါတယ်
var connection = new signalR.HubConnectionBuilder()
    .withUrl("/chatHub")
    .build();

// Connection မတည်ဆောက်ခင်မှာ send button ကို ခဏပိတ်ထားမှာ ဖြစ်ပါတယ်
document.getElementById("sendButton").disabled = true;

// ✅ Connection ကို မစတင်ခင်မှာ event handler တွေကို အရင်ဆုံး register လုပ်ထားဖို့ အရေးကြီးပါတယ်
connection.on("ClientReceiveMessage", function (user, message) {
    var li = document.createElement("li");
    li.textContent = `${user} says ${message}`;
    document.getElementById("messagesList").appendChild(li);
});

// Connection ကို စတင်ပါမယ်
connection.start().then(function () {
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    return console.error(err.toString());
});

// "Send" button ကို နှိပ်လိုက်တဲ့အခါ Server ဆီကို message ပို့ဖို့ ဖြစ်ပါတယ်
document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    // Server ဘက်က ServerReceiveMessage ဆိုတဲ့ method ကို လှမ်းခေါ်တာ ဖြစ်ပါတယ်
    connection.invoke("ServerReceiveMessage", user, message).catch(function (err) {
        return console.error(err.toString());
    });
    event.preventDefault();
});
```

---

### 8. Run and Test

F5 ကို နှိပ်ပြီး project ကို run ပါ။

Browser tab နှစ်ခုမှာ ကျွန်တော်တို့ရဲ့ app ကို ဖွင့်လိုက်ပါ။

Tab တစ်ခုမှာ username နဲ့ message ကို ရိုက်ထည့်ပြီး Send Message ကို နှိပ်ပါ။

Message ဟာ tab နှစ်ခုလုံးမှာ ချက်ချင်းဆိုသလို ပေါ်လာတာကို တွေ့ရမှာ ဖြစ်ပါတယ်။

---

### ✅ Final Flow

- **Client → Server:** `ServerReceiveMessage` method ကို ခေါ်ဆိုပါတယ်။
- **Server → Clients:** `ClientReceiveMessage` method ကနေတစ်ဆင့် client အားလုံးဆီကို message ပြန်လည်ဖြန့်ဝေပေးပါတယ်။
- **Hub URL:** ပုံမှန်အားဖြင့် `/chatHub` ဖြစ်ပေမယ့်၊ `Program.cs` နဲ့ `chat.js` နှစ်ခုစလုံးမှာ တူညီနေသရွေ့ `/myRealtimeHub` လိုမျိုး ကြိုက်နှစ်သက်ရာ ပြောင်းလဲနိုင်ပါတယ်။
- **Best Practice:** `connection.start()` ကို မခေါ်ခင်မှာ `connection.on(...)` handler တွေကို အမြဲတမ်း register လုပ်ထားဖို့ ဖြစ်ပါတယ်။
- **Layout Fix:** `_Layout.cshtml` မှာ `scripts` section ကို အက္ခရာအသေးနဲ့ သုံးပါမယ်၊ ပြီးတော့ ကျွန်တော်တို့ရဲ့ JS file တွေကို `Index.cshtml` ထဲက `@section scripts` ထဲမှာ ထည့်သွင်းအသုံးပြုပါမယ်။

---
