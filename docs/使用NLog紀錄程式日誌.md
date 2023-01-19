# 使用NLog紀錄程式日誌

#### 開發環境
* Microsoft Visual Studio Professional 2019
* .NET 5
***

#### 內容

1. 編輯Convience.ManagentApi.csprog，在ItemGroup段落加入NLog主要的套件
```
<ItemGroup>
    <PackageReference Include="NLog" Version="4.7.9" />
    <PackageReference Include="NLog.Web.AspNetCore" Version="4.12.0" />
</ItemGroup>
```

2. 在根目錄建立nlog.config，此為NLog的設定檔。
```
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true">

  <!-- enable asp.net core layout renderers -->
  <extensions>
    <add assembly="NLog.Targets.ElasticSearch"/>
    <add assembly="NLog.Web.AspNetCore"/>
  </extensions>

  <!-- the targets to write to -->
  <targets async="true">

    <!-- 本地文件日志target -->
    <target xsi:type="File" name="allfile" archiveAboveSize="10485760"
            fileName="${basedir}/Logs/${machinename}-all-${shortdate}/${shortdate}.log"
            layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}" />

  </targets>

  <!-- rules to map from logger name to target -->
  <rules>
    <!--All logs, including from Microsoft-->
    <logger name="*" minlevel="Trace" writeTo="allfile" />
    <logger name="*" minlevel="Trace" writeTo="ElasticSearch" />

    <!--Skip non-critical Microsoft logs and so log only own logs-->
    <logger name="Microsoft.*" maxlevel="Info" final="true" />
    <!-- BlackHole without writeTo -->
    <logger name="*" minlevel="Trace" writeTo="ownFile-web" />
  </rules>
</nlog>
```


3. 編輯Program.cs
```
public class Program
    {
        public static void Main(string[] args)
        {

            var logger = NLogBuilder.ConfigureNLog("nlog.config").GetCurrentClassLogger();
            try
            {
                logger.Debug("Debug init main");
                logger.Info("Info init main");
                var host = CreateHostBuilder(args).Build();
                host.InitialDataBase<SystemIdentityDbContext>(DbContextSeed.InitialApplicationDataBase);
                host.Run();
            }
            catch (Exception ex)
            {
                // 眸袙雄祑都
                logger.Error(ex, "Program stopped because of exception");
                //throw;
            }
            finally
            {
                NLog.LogManager.Shutdown();
            }
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
                .ConfigureLogging(logging =>
                {
                    logging.ClearProviders();
                    logging.SetMinimumLevel(LogLevel.Trace);
                    logging.AddConsole();
                }).UseNLog();
            

    }
```

4. 調整專案設定檔appsettings.json
```
{
  "Logging": {
    "LogLevel": {
      "Default": "Trace",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

5. 宣告NLog
```
using Microsoft.Extensions.Logging;
namespace CookbookApi.Controllers
{
    [Route("api/[controller]")]
    public class TestController : Controller
    {
        private readonly ILogger<RecipeController> _logger;
 
        public TestController(ILogger<RecipeController> logger)
        {
            _logger = logger;
        }
        public void Test(){
            //Log的等級
            _logger.LogCritical("");
            _logger.LogDebug("");
            _logger.LogError("");
            _logger.LogInformation("");
            _logger.LogTrace("");
            _logger.LogWarning("");
        }
}
```