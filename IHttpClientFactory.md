# IHttpClientFactory ဆိုတာဘာလဲ?

ASP.NET Core Application တစ်ခုက **External API** တစ်ခုကို ခေါ်ချင်တဲ့အခါ

ဥပမာ

* Payment Gateway API
* SMS API
* Email API
* Weather API
* Banking API

စတဲ့ API တွေကို ခေါ်ဖို့ **HttpClient** ကို အသုံးပြုရပါတယ်။

ဒါပေမယ့် `HttpClient` ကို Request တိုင်းမှာ အသစ်ဆောက်ပြီး အသုံးပြုရင်

* Socket Exhaustion
* DNS Change မသိခြင်း
* Configuration နေရာအနှံ့ ပျံ့နေခြင်း

စတဲ့ ပြဿနာတွေ ဖြစ်လာနိုင်ပါတယ်။

ဒါကြောင့် .NET က **IHttpClientFactory** ဆိုတဲ့ Built-in Feature ကို ထည့်ပေးထားတာ ဖြစ်ပါတယ်။

`IHttpClientFactory` က

* `HttpClient` ဖန်တီးပေးတယ်
* `HttpMessageHandler` နဲ့ Connection Pool ကို စီမံပေးတယ်
* Base Address, Header, Timeout စတဲ့ Configuration တွေကို တစ်နေရာတည်းမှာ ထားနိုင်စေတယ်
* Logging, Delegating Handler နဲ့ Resilience Policy တွေ ထည့်နိုင်စေတယ်

---

# HttpClient ဆိုတာဘာလဲ?

`HttpClient` ဆိုတာ HTTP Request တွေ ပို့ပြီး HTTP Response တွေ လက်ခံဖို့ အသုံးပြုတဲ့ Class ဖြစ်ပါတယ်။

```text
ASP.NET Core API
        │
        ▼
HttpClient
        │
        ▼
External API
```

Service တစ်ခုက နောက် Service တစ်ခုကို REST API နဲ့ ခေါ်တဲ့အခါ `HttpClient` ကို အသုံးပြုပါတယ်။

---

# HttpClient ကို Request တိုင်း အသစ်ဆောက်ရင် ဘာဖြစ်လဲ?

Beginner တော်တော်များများက ဒီလိုရေးတတ်ပါတယ်။

```csharp
public async Task<string> GetUsersAsync()
{
    using var client = new HttpClient();

    return await client.GetStringAsync(
        "https://api.example.com/users");
}
```

ဒီ Code က Request နည်းတဲ့အချိန်မှာ အလုပ်လုပ်ပါတယ်။

ဒါပေမယ့် Request တိုင်း `HttpClient` အသစ်ဆောက်ပြီး Dispose လုပ်တဲ့အခါ Client တစ်ခုစီက ကိုယ်ပိုင် Connection Pool တစ်ခုစီ သုံးပါတယ်။ ပိတ်လိုက်တဲ့ TCP Connection တွေကလည်း ချက်ချင်း မပျောက်ဘဲ `TIME_WAIT` State မှာ ခဏကျန်နေနိုင်ပါတယ်။

Request များလာတဲ့အခါ Available Port တွေကုန်ပြီး **Socket Exhaustion** ဖြစ်နိုင်ပါတယ်။

```text
Request 1 ──► new HttpClient() ──► Connection Pool 1
Request 2 ──► new HttpClient() ──► Connection Pool 2
Request 3 ──► new HttpClient() ──► Connection Pool 3
```

---

# Long-lived HttpClient နဲ့ DNS Change

`HttpClient` ကို Application တစ်လျှောက် တစ်ခုတည်းထားသုံးတာက Socket Exhaustion ကို ရှောင်နိုင်ပါတယ်။

ဒါပေမယ့် `HttpClient` က DNS ကို Request တိုင်း ပြန်မစစ်ပါဘူး။ Connection အသစ်ဆောက်တဲ့အချိန်မှာပဲ DNS ကို Resolve လုပ်ပါတယ်။

ဒါကြောင့် Long-lived Client သုံးမယ်ဆိုရင် `SocketsHttpHandler.PooledConnectionLifetime` ကို သတ်မှတ်ပေးရပါတယ်။

ASP.NET Core Project မှာ API တစ်ခုချင်းစီအတွက် Configuration, Logging နဲ့ Resilience တွေ လိုလာရင်တော့ `IHttpClientFactory` သုံးတာ ပိုအဆင်ပြေပါတယ်။

---

# IHttpClientFactory က ဘယ်လိုဖြေရှင်းပေးလဲ?

```text
Application
    │
    ▼
IHttpClientFactory
    │
    ├──► HttpClient အသစ်
    │
    └──► HttpMessageHandler Pool ကို Reuse
              │
              ▼
          External API
```

`CreateClient()` ခေါ်တိုင်း `HttpClient` Instance အသစ်ရပါတယ်။

ဒါပေမယ့် အောက်မှာရှိတဲ့ `HttpMessageHandler` နဲ့ Connection Pool ကို Factory က စီမံပြီး Reuse လုပ်ပေးပါတယ်။

ဒါကြောင့် Factory က ဖန်တီးပေးတဲ့ `HttpClient` ကို Short-lived အဖြစ် သုံးပြီး Dispose လုပ်လို့ရပါတယ်။

---

# Basic Client

Weather API တစ်ခုကို ခေါ်မယ်ဆိုပါစို့။

## Step 1 - Register လုပ်မယ်

`Program.cs`

```csharp
builder.Services.AddHttpClient();
```

## Step 2 - IHttpClientFactory ကို Inject လုပ်မယ်

```csharp
public sealed class WeatherService
{
    private readonly IHttpClientFactory _factory;

    public WeatherService(IHttpClientFactory factory)
    {
        _factory = factory;
    }

    public async Task<string> GetWeatherAsync(
        CancellationToken cancellationToken)
    {
        using var client = _factory.CreateClient();

        return await client.GetStringAsync(
            "https://api.weather.example/weather",
            cancellationToken);
    }
}
```

ဒီနည်းက ရိုးရှင်းပေမယ့် API များလာတဲ့အခါ URL နဲ့ Configuration တွေ Service ထဲ ပျံ့သွားနိုင်ပါတယ်။

အဲ့ဒီအခါ **Named Client** သို့မဟုတ် **Typed Client** ကို သုံးသင့်ပါတယ်။

---

# Named Client

Project မှာ External API နှစ်ခုရှိတယ်ဆိုပါစို့။

* Payment API
* SMS API

Client တစ်ခုချင်းစီမှာ Base Address နဲ့ Timeout မတူနိုင်ပါတယ်။

`Program.cs`

```csharp
builder.Services.AddHttpClient("Payment", client =>
{
    client.BaseAddress = new Uri("https://payment.example.com/");
    client.Timeout = TimeSpan.FromSeconds(15);
});

builder.Services.AddHttpClient("Sms", client =>
{
    client.BaseAddress = new Uri("https://sms.example.com/");
    client.Timeout = TimeSpan.FromSeconds(10);
});
```

အသုံးပြုတဲ့အခါ Client Name ကို `CreateClient()` ထဲ ထည့်ပေးရပါတယ်။

```csharp
public sealed class PaymentService
{
    private readonly IHttpClientFactory _factory;

    public PaymentService(IHttpClientFactory factory)
    {
        _factory = factory;
    }

    public async Task<string> GetTransactionAsync(
        string transactionId,
        CancellationToken cancellationToken)
    {
        var client = _factory.CreateClient("Payment");

        using var response = await client.GetAsync(
            $"transactions/{transactionId}",
            cancellationToken);

        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsStringAsync(
            cancellationToken);
    }
}
```

`Payment` လို့ခေါ်လိုက်တာနဲ့ Payment API အတွက် သတ်မှတ်ထားတဲ့ Base Address နဲ့ Timeout ကို အလိုအလျောက် ရပါတယ်။

Named Client က

* Client Configuration အများကြီးရှိတဲ့အခါ
* Client ကို Runtime မှာ Name နဲ့ ရွေးချယ်ချင်တဲ့အခါ

သင့်တော်ပါတယ်။

---

# Typed Client

Typed Client က API ခေါ်တဲ့ Code အားလုံးကို Class တစ်ခုထဲမှာ စုထားပေးပါတယ်။

```text
WeatherService
       │
       ▼
WeatherApiClient
       │
       ▼
Weather API
```

## Step 1 - Response Model

```csharp
public sealed record WeatherResponse(
    decimal TemperatureC,
    string Summary);
```

## Step 2 - Typed Client

```csharp
using System.Net.Http.Json;

public sealed class WeatherApiClient
{
    private readonly HttpClient _client;

    public WeatherApiClient(HttpClient client)
    {
        _client = client;
    }

    public async Task<WeatherResponse?> GetWeatherAsync(
        CancellationToken cancellationToken)
    {
        return await _client.GetFromJsonAsync<WeatherResponse>(
            "weather",
            cancellationToken);
    }
}
```

## Step 3 - Register လုပ်မယ်

`Program.cs`

```csharp
builder.Services.AddHttpClient<WeatherApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.weather.example/");
});
```

## Step 4 - Service မှာ သုံးမယ်

```csharp
public sealed class WeatherService
{
    private readonly WeatherApiClient _weatherApi;

    public WeatherService(WeatherApiClient weatherApi)
    {
        _weatherApi = weatherApi;
    }

    public Task<WeatherResponse?> GetWeatherAsync(
        CancellationToken cancellationToken)
    {
        return _weatherApi.GetWeatherAsync(cancellationToken);
    }
}
```

Typed Client က

* String Name မလိုဘူး
* IntelliSense နဲ့ Compile-time Checking ရတယ်
* API နဲ့သက်ဆိုင်တဲ့ Code ကို တစ်နေရာတည်းမှာ ထားနိုင်တယ်

ဒါကြောင့် API တစ်ခုကို နေရာအများကြီးက ခေါ်ရတဲ့ Project တွေမှာ သင့်တော်ပါတယ်။

> Typed Client ကို DI က Transient အဖြစ် Register လုပ်ပါတယ်။ Singleton Service ထဲမှာ Typed Client ကို သိမ်းမထားသင့်ပါဘူး။

---

# Delegating Handler

Delegating Handler က Outgoing HTTP Request အတွက် Middleware လို အလုပ်လုပ်ပါတယ်။

```text
WeatherApiClient
       │
       ▼
ApiKeyHandler
       │
       ▼
Weather API
```

Authentication Header, Correlation ID, Logging စတဲ့ Cross-cutting Concern တွေကို API Method တိုင်းမှာ ထပ်မရေးဘဲ Handler တစ်နေရာတည်းမှာ ရေးနိုင်ပါတယ်။

## ApiKeyHandler

```csharp
public sealed class ApiKeyHandler : DelegatingHandler
{
    private readonly IConfiguration _configuration;

    public ApiKeyHandler(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        var apiKey = _configuration["WeatherApi:ApiKey"]
            ?? throw new InvalidOperationException(
                "Weather API key is not configured.");

        request.Headers.Add("X-API-Key", apiKey);

        return base.SendAsync(request, cancellationToken);
    }
}
```

`Program.cs`

```csharp
builder.Services.AddTransient<ApiKeyHandler>();

builder.Services
    .AddHttpClient<WeatherApiClient>(client =>
    {
        client.BaseAddress = new Uri("https://api.weather.example/");
    })
    .AddHttpMessageHandler<ApiKeyHandler>();
```

API Key ကို Source Code ထဲ မရေးဘဲ User Secrets, Environment Variable သို့မဟုတ် Secret Store ကနေ ထည့်ပေးသင့်ပါတယ်။

Handler အများကြီးထည့်ရင် Register လုပ်ထားတဲ့ Order အတိုင်း Request ဝင်သွားပြီး Reverse Order နဲ့ Response ပြန်ထွက်လာပါတယ်။

```text
Request  ──► Handler A ──► Handler B ──► API
Response ◄── Handler A ◄── Handler B ◄── API
```

---

# Refit

Refit က REST API Interface တစ်ခုရေးပေးရုံနဲ့ `HttpClient` Code ကို Generate လုပ်ပေးတဲ့ Third-party Library ဖြစ်ပါတယ်။

Endpoint များပြီး `GetAsync()`, `PostAsJsonAsync()` နဲ့ JSON Mapping Code တွေ ထပ်ခါထပ်ခါရေးနေရတဲ့အခါ အသုံးဝင်ပါတယ်။

## Step 1 - Package ထည့်မယ်

```bash
dotnet add package Refit.HttpClientFactory
```

## Step 2 - API Interface ရေးမယ်

```csharp
using Refit;

public interface IWeatherApi
{
    [Get("/weather")]
    Task<WeatherResponse> GetWeatherAsync(
        CancellationToken cancellationToken);
}
```

## Step 3 - Register လုပ်မယ်

```csharp
builder.Services
    .AddRefitClient<IWeatherApi>()
    .ConfigureHttpClient(client =>
    {
        client.BaseAddress = new Uri("https://api.weather.example/");
    });
```

## Step 4 - Inject လုပ်ပြီး သုံးမယ်

```csharp
public sealed class WeatherService
{
    private readonly IWeatherApi _weatherApi;

    public WeatherService(IWeatherApi weatherApi)
    {
        _weatherApi = weatherApi;
    }

    public Task<WeatherResponse> GetWeatherAsync(
        CancellationToken cancellationToken)
    {
        return _weatherApi.GetWeatherAsync(cancellationToken);
    }
}
```

Refit Client ကလည်း အောက်မှာ `IHttpClientFactory` ကိုပဲ သုံးပါတယ်။ ဒါကြောင့် Delegating Handler နဲ့ Resilience Handler တွေကို ဆက်ထည့်နိုင်ပါတယ်။

```csharp
builder.Services
    .AddRefitClient<IWeatherApi>()
    .ConfigureHttpClient(client =>
    {
        client.BaseAddress = new Uri("https://api.weather.example/");
    })
    .AddHttpMessageHandler<ApiKeyHandler>();
```

Endpoint နည်းနည်းပဲရှိရင် Typed Client နဲ့တင် လုံလောက်ပါတယ်။ Refit က Boilerplate တကယ်များလာတဲ့အခါမှ ထည့်သင့်ပါတယ်။

---

# Polly နဲ့ Resilience

External API က အမြဲတမ်း အဆင်ပြေနေမယ်လို့ အာမခံလို့ မရပါဘူး။

* Network ခဏပြတ်နိုင်တယ်
* API က `500` ပြန်နိုင်တယ်
* Request က Timeout ဖြစ်နိုင်တယ်
* API က `429 Too Many Requests` ပြန်နိုင်တယ်

.NET 8 နဲ့အထက်မှာ `Microsoft.Extensions.Http.Resilience` Package က Polly ကို အခြေခံပြီး Standard Resilience Handler ပေးထားပါတယ်။

## Step 1 - Package ထည့်မယ်

```bash
dotnet add package Microsoft.Extensions.Http.Resilience
```

## Step 2 - Standard Resilience Handler ထည့်မယ်

```csharp
using Microsoft.Extensions.Http.Resilience;

builder.Services
    .AddHttpClient<WeatherApiClient>(client =>
    {
        client.BaseAddress = new Uri("https://api.weather.example/");
    })
    .AddStandardResilienceHandler(options =>
    {
        options.Retry.DisableForUnsafeHttpMethods();
    });
```

Standard Resilience Handler မှာ

* Rate Limiter
* Total Timeout
* Retry
* Circuit Breaker
* Attempt Timeout

တွေ ပါပြီးသားဖြစ်ပါတယ်။

`POST`, `PUT`, `PATCH`, `DELETE` လို Request တွေကို မစဉ်းစားဘဲ Retry လုပ်ရင် Payment နှစ်ခါဖြတ်တာ၊ Record နှစ်ခါ Insert ဖြစ်တာမျိုး ဖြစ်နိုင်ပါတယ်။

ဒါကြောင့် အပေါ်က Example မှာ Unsafe HTTP Method တွေအတွက် Retry ကို ပိတ်ထားတာဖြစ်ပါတယ်။ Write Operation ကို Retry လုပ်ဖို့လိုရင် API ဘက်မှာ Idempotency Key ထည့်ပြီးမှ လုပ်သင့်ပါတယ်။

> Client တစ်ခုမှာ Standard Resilience Handler တစ်ခုတည်း ထည့်ပါ။ Resilience Handler အများကြီး Stack မလုပ်သင့်ပါဘူး။

---

# N-Layered Architecture မှာ ဘယ် Layer မှာသုံးလဲ?

```text
Presentation Layer
------------------
WeatherController
        │
        ▼
Business Layer
--------------
WeatherService
        │
        ▼
Infrastructure Layer
--------------------
WeatherApiClient + IHttpClientFactory
        │
        ▼
External Weather API
```

**Presentation Layer**

* HTTP Request လက်ခံတယ်
* HTTP Response ပြန်တယ်

**Business Layer**

* Business Rule တွေ လုပ်တယ်
* Weather API Client ကို ခေါ်တယ်

**Infrastructure Layer**

* `IHttpClientFactory` Configuration ထားတယ်
* External API ကို တကယ်ခေါ်တယ်
* Authentication Header နဲ့ Resilience ကို စီမံတယ်

Controller နဲ့ Business Service ထဲမှာ URL, Header နဲ့ Retry Code တွေ မထည့်ဘဲ Infrastructure Client ထဲမှာ စုထားတာ ပိုထိန်းသိမ်းရလွယ်ပါတယ်။

---

# Real World Example

Internet Banking Project မှာ Customer Transfer လုပ်တယ်ဆိုပါစို့။

```text
TransferController
        │
        ▼
TransferService
        │
        ▼
PaymentApiClient
        │
        ├──► ApiKeyHandler
        ├──► Resilience Handler
        ▼
Bank Payment API
```

ဒီ Flow မှာ

* `TransferController` က Request နဲ့ Response ကို ကိုင်တယ်
* `TransferService` က Balance နဲ့ Transfer Rule တွေ စစ်တယ်
* `PaymentApiClient` က Bank API ကို ခေါ်တယ်
* `ApiKeyHandler` က Authentication Header ထည့်တယ်
* Resilience Handler က Transient Error တွေကို ကိုင်တယ်

---

# ဘယ် Client Type ကို ရွေးမလဲ?

| လိုအပ်ချက် | သင့်တော်တဲ့နည်း |
|---|---|
| API တစ်ခါနှစ်ခါပဲ ခေါ်မယ် | Basic Client |
| Configuration မတူတဲ့ Client အများကြီးကို Name နဲ့ ရွေးမယ် | Named Client |
| API Logic ကို Class တစ်ခုထဲ စုမယ် | Typed Client |
| Endpoint များပြီး Boilerplate လျှော့မယ် | Refit |
| Header, Logging, Authentication ကို နေရာတစ်ခုတည်းမှာ လုပ်မယ် | Delegating Handler |
| Retry, Timeout, Circuit Breaker လိုမယ် | Resilience Handler |

---

# သတိထားရမယ့်အချက်များ

## 1. Factory က Client ကို Reuse လုပ်တာ မဟုတ်ဘူး

`CreateClient()` ခေါ်တိုင်း `HttpClient` အသစ်ရပါတယ်။ Factory က Reuse လုပ်တာက အောက်က `HttpMessageHandler` နဲ့ Connection Pool ဖြစ်ပါတယ်။

## 2. Factory Client ကို Singleton ထဲ မသိမ်းထားပါနဲ့

Factory က ဖန်တီးပေးတဲ့ Client နဲ့ Typed Client တွေကို Short-lived အဖြစ် သုံးဖို့ ရည်ရွယ်ထားပါတယ်။

## 3. CancellationToken လက်ဆင့်ကမ်းပါ

User က Request ကို Cancel လုပ်လိုက်ရင် External API Call ကိုလည်း Cancel လုပ်နိုင်ဖို့ `CancellationToken` ကို HTTP Method အထိ လက်ဆင့်ကမ်းသင့်ပါတယ်။

## 4. Response Status ကို စစ်ပါ

`GetAsync()` နဲ့ `SendAsync()` သုံးရင် Status Code ကို စစ်ပြီး Error ကို သင့်တော်သလို Handle လုပ်ရပါတယ်။

```csharp
using var response = await client.GetAsync(
    "weather",
    cancellationToken);

response.EnsureSuccessStatusCode();
```

## 5. Secret နဲ့ Sensitive Data ကို Log မလုပ်ပါနဲ့

Authorization Header, API Key, Password နဲ့ Personal Data တွေကို Request/Response Log ထဲ မထည့်သင့်ပါဘူး။

---

# IHttpClientFactory ရဲ့ အားသာချက်များ

### 1. Connection ကို စနစ်တကျ စီမံပေးတယ်

`HttpMessageHandler` နဲ့ Connection Pool ကို Reuse လုပ်ပေးတာကြောင့် Socket Exhaustion ဖြစ်နိုင်ခြေကို လျှော့ပေးပါတယ်။

### 2. DNS Change ကို Handle လုပ်နိုင်တယ်

Handler Lifetime ပြည့်တဲ့အခါ Handler အသစ်ပြောင်းပေးတာကြောင့် Connection အသစ်က DNS ကို ပြန် Resolve လုပ်နိုင်ပါတယ်။

### 3. Configuration တစ်နေရာတည်း ထားနိုင်တယ်

Base Address, Header, Timeout နဲ့ Authentication Configuration တွေကို `Program.cs` မှာ စုထားနိုင်ပါတယ်။

### 4. Logging ပါပြီးသားဖြစ်တယ်

Factory က ဖန်တီးတဲ့ Client တွေအတွက် `ILogger` နဲ့ Outgoing Request Logging ရပါတယ်။

### 5. Pipeline တိုးချဲ့နိုင်တယ်

Delegating Handler, Refit နဲ့ Polly-based Resilience Handler တွေကို လိုအပ်သလို ပေါင်းစပ်နိုင်ပါတယ်။

---

# Summary

`IHttpClientFactory` ဆိုတာ ASP.NET Core မှာ `HttpClient` Configuration နဲ့ အောက်က `HttpMessageHandler` Lifetime ကို စီမံပေးတဲ့ Built-in Factory ဖြစ်ပါတယ်။

Project ရိုးရိုးမှာ Basic Client သုံးနိုင်ပြီး Configuration မတူတဲ့ Client များရင် Named Client၊ API Logic ကို သီးသန့်စုချင်ရင် Typed Client သုံးနိုင်ပါတယ်။ Endpoint များလာပြီး Boilerplate တကယ်များလာရင် Refit ကို ထည့်နိုင်ပါတယ်။

Authentication နဲ့ Logging အတွက် Delegating Handler၊ Retry, Timeout နဲ့ Circuit Breaker အတွက် `Microsoft.Extensions.Http.Resilience` ကို သုံးနိုင်ပါတယ်။

Enterprise ASP.NET Core Project မှာတော့ **Typed Client + Delegating Handler + Resilience Handler** နဲ့စပြီး Refit ကို လိုအပ်မှ ထည့်တာ ရိုးရှင်းပြီး ထိန်းသိမ်းရလွယ်ပါတယ်။

---

# ဆက်လက်လေ့လာရန်

* [Microsoft Learn - Use IHttpClientFactory](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/http-requests)
* [Microsoft Learn - Guidelines for using HttpClient](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines)
* [Microsoft Learn - HTTP resilience](https://learn.microsoft.com/en-us/dotnet/core/resilience/http-resilience)
* [Refit Documentation](https://github.com/reactiveui/refit)
