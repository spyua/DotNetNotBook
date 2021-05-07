
# 使用Studio Code開發Dot Net Core

## 安裝環境

 - [Step1:安裝Visual Studio Code](https://code.visualstudio.com/download)
 - [Step2:安裝Dot Net Core SDK](https://dotnet.microsoft.com/download)
 <font color="#f00">註:目前還是建議用Dot Net Core3.0版本，此版本是Long Term維護。</font>
 - [Step3:安裝C# Microsoft](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

以上步驟安裝完成後，即可至Console Key
```
dotnet --version
```
![](https://i.imgur.com/2cQ6K4x.png)

若有看到版本號出來，代表已可開始使用Dot Net CLI。

## 利用Dot Net Cli新增一個Web MVC Template Project

接下來我們要下dotnet cli新增一個MVC Web專案，我們在PowerShell下dotnet new指令，此時就會列出所有Template選項

![](https://i.imgur.com/sKL75Qf.png)

除了Template選項，也會很貼心的給Example指令參考

![](https://i.imgur.com/o0YTpaJ.png)

此時我們至D槽新增一個名newproject的web mvc專案

請下指令
 - D:
 - mkdir dotnettemp
 - cd dottnetemp
 - dotnet new mvc -o newproject

![](https://i.imgur.com/SkPUjcz.png)

此時成功新增一個dotnet web mvc資料夾，讓我們用Visual Studio Code打開他，檔案架構完全與Visual Studio新增出來的專案無異。

![](https://i.imgur.com/x7JIKjU.png)

此時我們打開Visual Studio Code的終端機下

```
dotnet run
```

![](https://i.imgur.com/FgKRPCA.png)

當專案啟動時，接著可以用網頁打開所啟動的Url網頁

![](https://i.imgur.com/nFEbLZB.png)


## 新增一個LogSender類別庫

現在我們練習新增一個名叫LogSender的類別庫，至Visual Studio Code下

```
dotnet new classlib -o LogSender
```

![](https://i.imgur.com/cfYhmzw.png)

此時就會看到LogSender在目錄中出現，也有自己的csproj檔。

## 使用NutGet Package Manager

在Studio Code若要使用NutGet則需額外安裝NutGet Package Manager外掛，

![](https://i.imgur.com/zf5M1nn.png)


當安裝完後我們按

```=
ctrl+shift+p
```
搜尋NuGet Package Manager，並按Add Package

![](https://i.imgur.com/d3kAx7h.png)

就會跳出讓你輸入Package Name的提示

![](https://i.imgur.com/9o4h4Fe.png)


此時我們輸入Package Name:Akka接著按Enter就會跳出Akka名稱相關的套件選擇

![](https://i.imgur.com/H6VUCpr.png)


選擇Akka.Logger.Nlog，接著就會跳出版本選擇，選完版本後按Enter

![](https://i.imgur.com/7jrdOgU.png)


最後就選擇要裝在哪一個類別庫中，選完後就完成安裝了

![](https://i.imgur.com/iC6b8vu.png)
