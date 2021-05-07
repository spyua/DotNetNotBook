
# 關於DotNet Core Middleware

近期在整合Akka分散系統與DotNet Core Web，在一開始設計將Akka的啟動點作在Starup的Configure。看到許多Middleware功能設置，但一直沒認真去研究過，故寫篇關於在DotNet Core Middleware的整理。

## 一、Middleware概念
Middleware在概念上基本上就是一個計算軟件架構，他介於操作與應用之間，便於系統或軟體元件之間的溝通、共享資源或統一處理加工。在 Web servers, Application Servers很常見到，讓每個Request進來時第一時間進中介處裡才轉到實際的程式。

如果要講古的話...其實最早具有Middleware技術概念的是IBM的[CICS](https://en.wikipedia.org/wiki/CICS)，有興趣的話可以至Wiki閱讀一番。如過對於他的設計原理有興趣，可以去讀[Middleware Architecture with Patterns and Frameworks](http://lig-membres.imag.fr/krakowia/Files/MW-Book/Chapters/Basic/basic.html)裡面會從0開始講Middleware的緣起與設計概念。

若比較懶的話，大致理解現成Framework怎麼使用設定其實也可以，但個人建議至少對Proxy、Chain Of Responsibility、Deligate與DI至少有些概念，在使用DotNet Core Middleware 設置會比較不會覺得空空的。

但這裡我還是稍微解釋一下Middleware有哪些應用類別

 - 類別一、Database 
     - 在使用資料庫交易上我們在實現後端功能，會使用不同的語言與資料庫(MySql, MSSql...)。此時我們就會須要有個Middleware去轉接提供操作與連結不同DB的服務功能(DataBase Gateway)。讓在不同情境下都可讓User輕鬆使用資料庫，例如常見的ODBC與JDBC，而在DotNet的EF Framework與Dapper都提供具有這樣的功能。在DB有Middleware的設計下，使用者只需專注在實現DataAccess功能即可。
     - ![](https://i.imgur.com/PGREMGD.png)

     
 - 類別二、Messaging
     - 舉一個簡單的例子，如果一個系統裡面有不同的Application要互相傳輸資料，若無一個中介管理，則會向左圖一樣，除了要實作資料傳輸之外，各Application之間的關係呈現一個很亂的狀態。此時加入Middleware設計，使用者只需專注在該如何傳輸資訊即可，在現有的Middleware Framework，如Akka、MSMQ等...會幫你處理好異部通知與資料序列化與反序列化的實作，因此使用者只需專注在資料要傳給誰。
     - ![](https://i.imgur.com/WrTSg0o.png)
 
 - 類別三、Transaction
     - 在分散式系統中，點跟點之間如果單純做Transaction(交易，資料傳輸)，成功與否彼此都會知道。但是如果Transaction跨過不少交易點(A點-B點-C點-回A點)，那中間如果失敗了，其實對各點來說，彼此是沒辦法知道這次Transaction是否成功。此時我們會需要一個Coordinator來協調這些點的Transaction。這概念就是上述提到的IBM CISC，他是一種Tp Monitor(遠程處理監視器)，監視本地和遠程終端之間的數據傳輸，以確保事務處理成功，或者在發生錯誤時採取適當的措施。 
     - ![](https://i.imgur.com/D1UtGiR.png)


 - 類別四、Integration Broker
     - Integration其實是一種統稱，只要具有可在系統交換消息時，統一處裡驗證、變換路由。調節應用程式的通信等..基本上都可以視為Integration Broker，簡單來說它就是Inter-Application Communication的技術。
     - ![](https://i.imgur.com/HHHIgDC.png)

上述是在於軟工較於常見的分類，至於Embeded的世界，也常會使用此觀念去設計架構。在有中介層的設計下，對於整體的執行環境與邏輯的差異性可縮減不少複雜度。在每層Layer、Component或Node的職責設計也較專依，並降低彼此的耦合性提高設計彈性。

基本上Middleware是[AOP (Aspect-Oriented Programming)](/@41MKMGSpR_K11_wgmtcRgw/HJqmkKYDO)的一種典型應用，大致都會配合Proxy、Chain Of Responsibility、Deligate去實作。

一開始在了解AOP Filter與Middleware會覺得兩者很相似，但AOP Filter更貼近邏輯業務，而Middleware較關注於系統上整體的應用。兩者關注點職責其實是有很大的差異性。


## 二、DotCore Middleware

### a.簡述
在上述大致簡略闡述關於Middleware的概念後，應該會對於他的職責設計目的與應用有些了解。

在Web Service運作中，大部分框架都會把http請求封裝成一個Pipeline ，每一次的請求都是經過Pipeline一系列操作後才進到我們的實作方法中，所以一個Request有可能在此管線中經過好幾個中介處理。

換句話說，你可以想像Web Service就是一個流水線工廠，每個Request就是一筆訂單，訂單進來後會進流水線(Pipleline)開始處裡。中介軟體就是在這管道中的一道統一處理程序，用來攔截Request進行一些處理，有點像是每個站點檢查加工的概念，如[下圖](https://ithelp.ithome.com.tw/articles/10192682)


![](https://i.imgur.com/0dAjS6n.png)

在流水線處理完後再進到各自其餘處理機台(Action)，處理完後再送回流水線產出。

這種串接處理其實就是種[Chain Of Responsibility](https://www.youtube.com/watch?v=qI_v31n1ZTk)的實作，在First Middleware負責工作完成後，在串接丟到Secnod Middleware處理，一路往下串到Action。

Pipeline中每一個Middleware都可以對請求進行攔截並可以決定是否將請求轉移給下一個Middleware，最後到Action，處裡完後將Reponse再從Pipeline打回去給User。這概念簡化原本DotNet的[HTTP Modules 及 HTTP Handlers](https://dotblogs.com.tw/ASPNETShare/2016/03/20/201603191-introMiddleware)的原本的使用方式並也把Pipline概念設計進去。

### b.使用
而在DotNet Core現成框架中，實現Middleware設計只需簡單在Starup Configuration設置即可，而設置部分會專注於管線的流向設計。

Starup為DotNet Core的起始點，基本在實作上會作兩種設置，一個就是在ConfigureServices透過IServiceCollection去設置系統DI，另一個就是在Configure透過IApplicationBuilder與IWebHostEnvironment分別去設置Middleware與判斷目前執行環境。

我們試著新增基礎Web專案(MVC)上，會在Starup Configure 上看到已有的Middleware基礎功能設置。

```csharp=
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
   if (env.IsDevelopment())
   {
     app.UseDeveloperExceptionPage();
   }
   else
   {
        app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
        app.UseHsts();
    }
    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
       endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
        });
    }
```

此段Request經過這些Middleware流程，職責解說如下
 
 - UseDeveloperExceptionPage, UseExceptionHandler, UseHsts
 
    一開始會判斷開發環境去作對應的錯誤處理
    
    開發環境-> UseDeveloperExceptionPage :當例外事件發生，會回報應用程式執行階段錯誤，此時會顯示一個錯誤資訊頁面顯示相關錯誤Log。

    生產執行環境-> UseExceptionHandler :當例外事件發生，指定錯誤處理頁面作相關處理。
    
    生產執行環境-> HTTP 靜態傳輸安全性通訊協定[(HSTS)](https://reurl.cc/NXWAMx)，會新增Strict-Transport-Security 標頭。


```csharp=
 if (env.IsDevelopment())
 {
   app.UseDeveloperExceptionPage();
 }
 else
 {
    app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}
```

 - UseHttpsRedirection : 將HTTP要求重新導向到HTTPS。
 - UseStaticFiles : 傳回靜態檔案並縮短進一步的要求處理時間。
 - UseRouting : 將路由對應新增至中介軟體管線。 此中介軟體會查看應用程式中定義的端點集合，並根據要求選取[最符合](https://reurl.cc/NXWRZ9)的條件。
 - UseAuthorization : 在允許使用者存取安全資源之前先驗證使用者。
 - UseEndpoints : 請求最終端處理點，在此點後的Middleware的設置將不會被執行，預設邏輯為 URL / 時顯示 Hello World!，其餘路徑顯示 HTTP 404。

實際的Request Pipleline處理流程如下圖

![](https://i.imgur.com/5uv1ICF.png)


上述簡單概要的闡述預設Web專案的Middleware使用設置有哪些功能，基本上在網頁後端的設計上，這些需求功能都是常見必須的。

如果要自我設計Middleware去安插設置，分成Use使用方法與Run、Map擴充方法，使用上解說推薦可以直接看John Wu鐵人邦的[ASP.NET Core 2 系列 - Middleware](https://reurl.cc/MZeRKX)文章，在此我根據此文章直接擷取內容來解釋這三個方法使用。

 #### .app.Use()

Use為IApplicationBuilder註冊方法，大部分註冊Middleware都是以Use開頭。你可以試看看用IDE新增Middleware檔案。

![](https://i.imgur.com/IT25yiR.png)

預設新增檔案程式碼如下，主要撰寫區塊為Task Invoke，我們會在此實作Middleware行為。

```csharp=
// You may need to install the Microsoft.AspNetCore.Http.Abstractions package into your project
public class Middleware
{
    private readonly RequestDelegate _next;

    public Middleware(RequestDelegate next)
    {
        _next = next;
    }

    public Task Invoke(HttpContext httpContext)
    {
            //實作處理...

        return _next(httpContext);
    }
}

// Extension method used to add the middleware to the HTTP request pipeline.
public static class MiddlewareExtensions
{
    public static IApplicationBuilder UseMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<Middleware>();
    }
}
```

行為實作完後，預設已有包裝好的靜態IApplicationBuilder Extenstion擴充方法(UseXXXX)，使用上只需在Starup Configure使用此靜態方法即可。

```csharp=
 app.UseMiddleware();
```

使用Use在Pipleline中註冊Middleware，當Middleware處理完後會往下繼續，你可以看預設新增的Middleware Class Invoke部分，最後會透過Request委派作下一個Middleware處理。

```csharp=
 private readonly RequestDelegate _next;

 public Middleware(RequestDelegate next)
 {
      _next = next;
 }
 public Task Invoke(HttpContext httpContext)
{
    //實作處理...
     return _next(httpContext);
}
```

我們寫一段測試Code來測試，到Starup Configure 用最簡單的方式，直接用app.Use新增三個Middleware(First, Second, Third)以及app.Run最後一個Middleware模擬註冊。因為只Demo Middleware管線流向，故不實作內部功能，只顯示Print Log在頁面上。

```csharp=
app.Use(async (context, next) =>
{
    //Logic
   await context.Response.WriteAsync("First Middleware in. \r\n");
   await next.Invoke();
   //Logic
   await context.Response.WriteAsync("First Middleware out. \r\n");
});

app.Use(async (context, next) =>
{
    //Logic
   await context.Response.WriteAsync("Secnod Middleware in. \r\n");
   await next.Invoke();
   //Logic
   await context.Response.WriteAsync("Second Middleware out. \r\n");
});

app.Use(async (context, next) =>
{
  //Logic
  await context.Response.WriteAsync("Third Middleware in. \r\n");
  await next.Invoke();
  //Logic
  await context.Response.WriteAsync("Third Middleware out. \r\n");
});

app.Run(async context =>
{
   await context.Response.WriteAsync("Simple Middleware Flow Test! \r\n");
});
```

我們可以看到實際運行狀況(下圖左)，Request到Response的顯示資訊，流向猶如下圖右顯示。


![](https://i.imgur.com/QrUlzao.png)

我們也可不讓Flow走到最終端，試著將Second Middleware next部分註解調

```csharp=
 app.Use(async (context, next) =>
{
   await context.Response.WriteAsync("Secnod Middleware in. \r\n");
   //await next.Invoke();
   await context.Response.WriteAsync("Second Middleware out. \r\n");
});
```

此時就會看到Request只到第二層Middleware就不會繼續往下走。

![](https://i.imgur.com/CJ4O74j.png)

在敘述過程中，我們可以看到在Third Middleware後面使用到的app.Run。一下來闡述一下app.Run的功用。

#### .app.Run()
Run是Middleware 的最後一個行為，不像 Use 能串聯其他 Middleware，但 Run 還是能完整的使用 Request 及 Response。簡單來說它用起來就像Use註冊Middleware，但在他之後的Middleware行為將不會被註冊(可參考上述app.Map範例)。

稍微看一下Run Extension Method程式解說 

![](https://i.imgur.com/GkDt7D9.png)

Run Extension Method它不像Use有IApplicationBuilder的回傳，故IApplicationBuilder Middleware若使用到Run註冊，在他之後的註冊都不會被執行。

<span style="color:red">一般來說不會直接使用。通常都使用 app.UseEndpoints() 來建立自己所需要的項目。
</span>.

#### .app.Map() 

最後來介紹Map，Map為IApplicationBuilder延伸擴充方法，用來處理一些簡單路由依照不同的 URL 指向不同的 Run 及註冊不同的 Use Middleware。

<span style="color:red">一般來說不會直接使用。通常都使用 app.UseEndpoints() 來建立自己所需要的項目。
</span>。根據上述Use所舉例的範例，我們在UseEndPoint加入Map的註冊，指定只有在URL為/Map的時候額外執行Map Middleware。

因為針對URL設置Middleware註冊，我們在EndPoint前加入app.UseRouting()，程式碼如下

```csharp=
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.Use(async (context, next) =>
    {
            await context.Response.WriteAsync("First Middleware in. \r\n");
            await next.Invoke();
            await context.Response.WriteAsync("First Middleware out. \r\n");
    });  

    app.Use(async (context, next) =>
    {
            await context.Response.WriteAsync("Secnod Middleware in. \r\n");
            await next.Invoke();
            await context.Response.WriteAsync("Second Middleware out. \r\n");
    });

    app.Use(async (context, next) =>
    {
            await context.Response.WriteAsync("Third Middleware in. \r\n");
            await next.Invoke();
            await context.Response.WriteAsync("Third Middleware out. \r\n");
    });

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.Map("/", async context =>
        {
            await context.Response.WriteAsync("Middleware Testing\r\n");
        });

        endpoints.Map("/Map", async context =>
        {
            await context.Response.WriteAsync(" ====  Map Middleware Testing ====\r\n");
        });
    });
}

```

運行時，在URL為/..，Map Middleware不會被註冊執行到，唯有在URL有/map....才會看到Map Middleware執行。簡單來說<span style="color:red">Map可以針對自訂的路由設定獨立的管線。
</span>

![](https://i.imgur.com/UqKmwu8.png)

## 自定義Middleware
在上述app.Use有提到，DotNet Core可直接新增Middleware Class，會有預設使用功能樣板提供使用。

自訂議Middleware有幾個功能需實作

 - 必須提供傳入一個 RequestDelegate（可傳入其他需要注入的服務）
 - 必須有一個方法 Invoke 或是 InvokeAsync，並且有一個參數 HttpContext，方法內的 next 前後的程式碼相當於 ing、ed 要做的事情，若沒有呼叫 next 則不會繼續執行後面的 Middleware
 - IApplicationBuilder 靜態Extension Method可在Starup Configure直接使用(app.UseMiddleware())。


```csharp=
// You may need to install the Microsoft.AspNetCore.Http.Abstractions package into your project
public class Middleware
{
    private readonly RequestDelegate _next;

    public Middleware(RequestDelegate next)
    {
        _next = next;
    }

    public Task Invoke(HttpContext httpContext)
    {
            //實作處理...

        return _next(httpContext);
    }
}

// Extension method used to add the middleware to the HTTP request pipeline.
public static class MiddlewareExtensions
{
    public static IApplicationBuilder UseMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<Middleware>();
    }
}
```

一般在Startup註冊為全域註冊，每個Request都會使用到在Startup註冊的Middleware功能。若不使用全域註冊，也可以個別Controller使用MiddlewareFilter屬性掛載註冊如下

```csharp=
// ..
[MiddlewareFilter(typeof(Middleware))]
public class HomeController : Controller
{
// ...

    [MiddlewareFilter(typeof(Middleware))]
    public IActionResult Index()
    {
        // ...
    }
}
```

## Summary

針對Middleware概念及在DotNet Core使用方式做一個簡單的整理，希望可協助不了解的人可快速理解DotNet Core的中介設計概念與使用方法。

## 參考
[1.ASP.NET Core Middleware](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-5.0)
[2.Middleware](https://en.wikipedia.org/wiki/Middleware)
[3.CICS](https://en.wikipedia.org/wiki/CICS)
[4.淺談ASP.NET Core 中介軟體詳解及專案實戰](https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/206001/)
[5.Middleware Architecture with Patterns and Frameworks](http://lig-membres.imag.fr/krakowia/Files/MW-Book/Chapters/Intro/intro.html)
[6.[基礎觀念系列] 讓任務排隊吧：Message Queue](https://medium.com/starbugs/%E8%AE%93%E4%BB%BB%E5%8B%99%E6%8E%92%E9%9A%8A%E5%90%A7-message-queue-1-de949e274c43)
[7.Message Brokers](https://www.ibm.com/cloud/learn/message-brokers)
[8.ASP.NET Core 的 Middleware](https://dotblogs.com.tw/ASPNETShare/2016/03/20/201603191-introMiddleware)
[9.官方附屬Middleware現有功能列表](https://reurl.cc/pmGLxr)
[10.ASP.NET Core 2 系列 - Middleware](https://reurl.cc/MZeRKX)