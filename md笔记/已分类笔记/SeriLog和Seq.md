#### Seq和Serilog

Seq 是一个功能强大的日志聚合和可视化工具，用于集中管理和分析应用程序的日志。它支持结构化日志记录，帮助开发者和运维人员从海量日志数据中快速找到问题根源。

Serilog 是一个灵活且功能强大的 .NET 日志记录库，特别设计用于支持结构化日志记录。它广泛用于 .NET 应用程序中，因为它易于使用且高度可扩展，能够与各种目标系统（称为“sinks”）集成。



#### 集成Seq和Serilog

Package:
Serilog.AspNetCore

Serilog.Sinks.Seq

Serilog.Sinks.MariaDB



因为希望在主机启动时就能记录日志，所以直接在program类main方法中创建静态Log类

https://docs.datalust.co/docs/getting-started-with-docker

docker pull datalust/seq

PH=$(echo '123456' | docker run --rm -i datalust/seq config hash)

```shell
docker run \
  --name seq \
  -d \
  --restart unless-stopped \
  -e ACCEPT_EULA=Y \
  -e SEQ_FIRSTRUN_ADMINPASSWORDHASH="$PH" \
  -v /Users/seq:/data \
  -p 80:80 \
  -p 5341:5341 \
  datalust/seq
```



集成Serilog并写入日志到seq

https://github.com/serilog/serilog-aspnetcore#serilogaspnetcore---

https://github.com/serilog/serilog/wiki/Configuration-Basics



配置MinimumLevel

`MinimumLevel` 配置对象提供要指定为最小值的日志事件级别之一。在上面的示例中，将处理级别为 `Debug` 和更高级别的日志事件，并最终将其写入控制台。

| `Verbose`     | Verbose is the noisiest level, rarely (if ever) enabled for a production app. 详细是最嘈杂的级别，很少（如果有的话）为生产应用启用。 |
| ------------- | ------------------------------------------------------------ |
| `Debug`       | Debug is used for internal system events that are not necessarily observable from the outside, but useful when determining how something happened. Debug 用于内部系统事件，这些事件不一定可以从外部观察到，但在确定某些事情是如何发生的时很有用。 |
| `Information` | Information events describe things happening in the system that correspond to its responsibilities and functions. Generally these are the observable actions the system can perform. 信息事件描述了系统中发生的与其职责和功能相对应的事情。通常，这些是系统可以执行的可观察操作。 |
| `Warning`     | When service is degraded, endangered, or may be behaving outside of its expected parameters, Warning level events are used. 当服务降级、受到威胁或行为可能超出其预期参数时，将使用警告级别事件。 |
| `Error`       | When functionality is unavailable or expectations broken, an Error event is used. 当功能不可用或预期被打破时，将使用 Error 事件。 |
| `Fatal`       | The most critical level, Fatal events demand immediate attention. 最关键的级别，致命事件需要立即关注。 |



重写特定命名空间事件的日志等级

```c#
.MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
```

比如Microsoft.AspNetCore.Hosting、Microsoft.AspNetCore.Mvc、Microsoft.AspNetCore.Routing



添加Enrichers

https://github.com/serilog/serilog/wiki/Enrichment

Enrichers是添加、删除或修改附加到日志事件的属性的简单组件。

```c#
.Enrich.WithProperty
```

```
.Enrich.FromLogContext()
```

`Serilog.Context.LogContext` 可用于在环境“执行上下文”中动态添加和删除属性;例如，在事务期间写入的所有消息都可能带有该事务的 ID，依此类推。

将属性推送到上下文中将覆盖任何具有相同名称的现有属性，直到从 `PushProperty（）` 返回的对象被释放，如示例中的属性 `A` 所示。

示例：

```c#
log.Information("No contextual properties");

using (LogContext.PushProperty("A", 1))
{
    log.Information("Carries property A = 1");

    using (LogContext.PushProperty("A", 2))
    using (LogContext.PushProperty("B", 1))
    {
        log.Information("Carries A = 2 and B = 1");
    }

    log.Information("Carries property A = 1, again");
}
```



添加Sub-loggers

```c#
.WriteTo.Seq("http://localhost:5341/")
```

```c#
.WriteTo.Conditional(Func<LogEvent,bool> condition, 
    Action<LoggerSinkConfiguration> configureSink)
```

通过过滤LogEvent中某个属性使表达式为true，来配置写入某个sink（接收器）

可以参考项目中的应用，DataCollectionService.cs类的Collect方法，主要针对一些订单新增的接口做更细致的Log，并保存到数据库中



#### Request logging 请求日志记录

ASP.NET Core 实现的默认请求日志记录是嘈杂的，每个请求会发出多个事件。

Serilog包拥有更智能的 HTTP 请求日志记录的中间件，中间件将这些压缩到一个事件中，该事件携带方法、路径、状态代码和计时信息

要启用中间件，请先在记录器配置或*appsettings.json*文件中将嘈杂的 ASP.NET Core 日志源的最低级别更改为 `Warning`：

```c#
.MinimumLevel.Override("Microsoft.AspNetCore.Hosting", LogEventLevel.Warning)
.MinimumLevel.Override("Microsoft.AspNetCore.Mvc", LogEventLevel.Warning)
.MinimumLevel.Override("Microsoft.AspNetCore.Routing", LogEventLevel.Warning)
```

```c#
app.UseSerilogRequestLogging();
```

`UseSerilogRequestLogging（）` 调用必须出现在 MVC 等处理程序*之前*，这一点很重要。中间件不会对管道中出现在它之前的组件进行计时或记录。（这可用于从日志记录中排除干扰处理程序，例如 `UseStaticFiles（），`方法是将 `UseSerilogRequestLogging（）` 放在它们之后。



还可以使用 `UseSerilogRequestLogging（）` 上的options回调来修改用于请求完成事件的消息模板、添加其他属性或更改事件级别：

```c#
app.UseSerilogRequestLogging(options =>
{
    // Customize the message template
    options.MessageTemplate = "Handled {RequestPath}";
    
    // Emit debug-level events instead of the defaults
    options.GetLevel = (httpContext, elapsed, ex) => LogEventLevel.Debug;
    
    // Attach additional properties to the request completion event
    options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
        diagnosticContext.Set("RequestScheme", httpContext.Request.Scheme);
    };
});
```

项目中则从请求头和httpContext中获取了一系列值存储



对创建主机的过程进行log

```c#
try
{
    Log.Information("Configuring web host ({ApplicationContext})...", AppName);
    var webHost = CreateWebHostBuilder(args).Build();

    Log.Information("Starting web host ({ApplicationContext})...", AppName);
    webHost.Run();

    return 0;
}
catch (Exception ex)
{
    Log.Fatal(ex, "Program terminated unexpectedly ({ApplicationContext})!", AppName);
    return 1;
}
finally
{
    Log.CloseAndFlush();
}
```



使用admin 123456查看seqUI中的log

