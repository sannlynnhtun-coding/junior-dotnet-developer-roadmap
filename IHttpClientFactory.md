# IHttpClientFactory ဆိုတာဘာလဲ?

`IHttpClientFactory` က External API တွေခေါ်ဖို့လိုတဲ့ `HttpClient` ကို ဖန်တီးပြီး စီမံပေးတဲ့ .NET Built-in Feature ဖြစ်ပါတယ်။

ဥပမာ - Payment API, SMS API, Weather API စတာတွေကို ခေါ်တဲ့နေရာမှာ သုံးပါတယ်။

## ဘာကြောင့်သုံးတာလဲ?

Request တိုင်းမှာ `new HttpClient()` လုပ်ရင် Connection တွေ များလာပြီး **Socket Exhaustion** ဖြစ်နိုင်ပါတယ်။ `HttpClient` တစ်ခုတည်းကို အမြဲသုံးရင်လည်း DNS ပြောင်းလဲမှုကို အချိန်မီ မသိနိုင်ပါဘူး။

### Socket Exhaustion ဆိုတာဘာလဲ?

Application က HTTP Connection အသစ်တွေ အများကြီးဖွင့်တဲ့အခါ အသုံးပြုနိုင်တဲ့ Network Port တွေ ကုန်သွားတာကို **Socket Exhaustion** လို့ခေါ်ပါတယ်။ အဲဒီအခါ Request အသစ်တွေ ပို့မရတော့ဘဲ Error ဖြစ်နိုင်ပါတယ်။

`HttpClient` ကို Request တိုင်း အသစ်ဆောက်ပြီး Dispose လုပ်ရင် ပိတ်လိုက်တဲ့ Connection တွေက ချက်ချင်းမပျောက်ဘဲ `TIME_WAIT` အခြေအနေမှာ ခဏကျန်နေလို့ ဒီပြဿနာ ဖြစ်နိုင်ပါတယ်။

`IHttpClientFactory` က

- `HttpClient` ဖန်တီးပေးတယ်
- Connection Pool နဲ့ Handler Lifetime ကို စီမံပေးတယ်
- Base Address နဲ့ Header စတာတွေကို တစ်နေရာတည်းမှာ သတ်မှတ်နိုင်စေတယ်
- Logging နဲ့ Authentication တွေ ထည့်နိုင်စေတယ်

> `CreateClient()` ခေါ်တိုင်း `HttpClient` အသစ်ရပေမယ့် အောက်က `HttpMessageHandler` နဲ့ Connection Pool ကို Factory က Reuse လုပ်ပါတယ်။

## Typed Client (Recommended)

API ခေါ်တဲ့ Code ကို Class တစ်ခုထဲမှာ စုထားနိုင်လို့ Project အများစုအတွက် Typed Client က ရိုးရှင်းပြီး ထိန်းသိမ်းရလွယ်ပါတယ်။

### 1. Client ရေးမယ်

```csharp
using System.Net.Http.Json;

public class WeatherResponse
{
    public decimal TemperatureC { get; set; }
    public string Summary { get; set; } = string.Empty;
}

public class WeatherApiClient
{
    private readonly HttpClient _client;

    public WeatherApiClient(HttpClient client)
    {
        _client = client;
    }

    public Task<WeatherResponse?> GetWeatherAsync(
        CancellationToken cancellationToken)
    {
        return _client.GetFromJsonAsync<WeatherResponse>(
            "weather",
            cancellationToken);
    }
}
```

### 2. Register လုပ်မယ်

`Program.cs`

```csharp
builder.Services.AddHttpClient<WeatherApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.weather.example/");
});
```

### 3. Inject လုပ်ပြီး သုံးမယ်

```csharp
public class WeatherService
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

## Client အမျိုးအစားများ

| အမျိုးအစား | ဘယ်အချိန်သုံးမလဲ |
|---|---|
| Basic Client | API တစ်ခါတလေ ခေါ်ရုံပဲ |
| Named Client | Configuration မတူတဲ့ API အများကြီးကို Name နဲ့ရွေးချင်ရင် |
| Typed Client | API Logic ကို Class တစ်ခုထဲ စုချင်ရင် |

Named Client ဥပမာ -

```csharp
builder.Services.AddHttpClient("Payment", client =>
{
    client.BaseAddress = new Uri("https://payment.example.com/");
});

var client = factory.CreateClient("Payment");
```

## သတိထားရန်

- Factory က ဖန်တီးတဲ့ Client နဲ့ Typed Client ကို Singleton Service ထဲမှာ မသိမ်းပါနဲ့။
- `CancellationToken` ကို HTTP Call အထိ လက်ဆင့်ကမ်းပါ။
- `GetAsync()` သုံးရင် Status Code ကို စစ်ပါ။
- API Key နဲ့ Authorization Header တွေကို Source Code သို့မဟုတ် Log ထဲ မထည့်ပါနဲ့။

## အကျဉ်းချုပ်

`IHttpClientFactory` က `HttpClient` Configuration နဲ့ Connection Lifetime ကို စနစ်တကျ စီမံပေးပါတယ်။ Project အများစုမှာ **Typed Client** နဲ့ စတင်အသုံးပြုနိုင်ပါတယ်။

## ဆက်လက်လေ့လာရန်

- [Microsoft Learn - Use IHttpClientFactory](https://learn.microsoft.com/aspnet/core/fundamentals/http-requests)
- [Microsoft Learn - Guidelines for using HttpClient](https://learn.microsoft.com/dotnet/fundamentals/networking/http/httpclient-guidelines)
