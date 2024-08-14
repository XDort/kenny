#### 1、项目结构

| 目录/文件                      | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| 依赖项                         | ASP.NET Core 开发、构建和运行过程中的依赖项，一般都是 NuGet 包和一些 SDK |
| Properties                     | 配置，存放了一些 .json 文件用于配置 ASP.NET Core 项目        |
| Propertics/launchSettings.json | 启动配置文件，为一个 ASP.NET Core 应用保存特有的配置标准，用于应用的启动准备工作，包括环境变量，开发端口等 |
| wwwroot                        | 网站跟目录，存放类似于 CSS、JS 和图片、还有 HTML 文件等静态资源文件的目录 |
| Program.cs                     | 这个文件包含了 ASP.NET Core 应用的 Main 方法，负责配置和启动应用程序 |
| Startup.cs                     | Startup.cs 文件是 ASP.NET Core 的项目的入口启动文件          |

#### 2、项目启动流程

ASP .net Core 6 以前的项目创建时会自动建立Program.cs和Startup.cs两个类文件

Program.cs 作为 Web应用程序的默认入口，不做任何修改的情况下，会调用同目录下 Startup.cs 中的 ConfigureServices 方法 和 Configure 方法。

应用启动流程：

![69af00b8cff7bef36c47b6cf2e4e5b4e](./../img/69af00b8cff7bef36c47b6cf2e4e5b4e.jpeg)

#### 3、验证容器配置

创建容器时 ValidateOnBuild = true 

可以让应用启动编译时检查被依赖的服务是否已注册、是否循环依赖

避免在运行时出现异常

如果使用autofac默认是开启的

![WeChat17e728751dd9e8ad3b27be0e3e5f2fef](./../img/WeChat17e728751dd9e8ad3b27be0e3e5f2fef.jpg)

通过根容器（创建的第一个IServiceProvider）创建Scope类型的实例，

会导致scope不能释放，要通过子容器创建，如：`container.CreateScope().ServiceProvider.GetRequiredService<T>();`



#### 4、ActivatorUtilities工具类

`ActivatorUtilities` 类会利用依赖注入容器中注册的服务提供程序来解析构造函数所需的服务依赖。

`T instance = ActivatorUtilities.CreateInstance<T>(serviceProvider, arguments);`



#### 5、通过扫描类注册程序集下的服务

可以通过自定义特性，在特性中使用扩展方法扩展 IServiceCollection，指定泛型

通过该泛型扫描该类型程序集下的服务并注册

自定义特性：

![WeChatbdfaa35fa9b8860dbb781b6985537cb5](./../img/WeChatbdfaa35fa9b8860dbb781b6985537cb5.jpg)

![WeChatecff1f592bdf0730f536929a152e6399](./../img/WeChatecff1f592bdf0730f536929a152e6399.jpg)

![WeChat393535e65b1afd4aca8f3e207321154b](./../img/WeChat393535e65b1afd4aca8f3e207321154b.jpg)

#### 6、单例工厂模式创建实例

⚠️使用单例工厂模式创建实例，传入的是根容器，

工厂创建的实例如果是多例的或依赖了多例的实例，则会报错

也不能在工厂内再创建子容器创建Scope，因为工厂是单例的



#### 7、HttpContext用法

ControllerBase类中提供了HttpContext、Request、Response等属性，可以直接获取和设置值

```
var userAgent = request.Headers.UserAgent;    
var customHeader = request.Headers["x-custom-header"];    
return Results.Ok(new { userAgent = userAgent, customHeader = customHeader });
```



#### 8、配置标签用法

\<Compile Remove="FolderToRemove\**\*.cs" />

项目文件列表中移除源代码文件，并不会物理删除文件。被移除的文件不会参与编译和生成过程

\<EmbeddedResource Remove="..." /> 主要用于管理项目中要嵌入到生成的程序集中的资源文件。

\<None Remove="..." />  用于项目中的其他文件，通常是配置文件、文档、图像等，不会被编译、嵌入为资源，也不会被包含在生成的程序集中

\<CopyToOutputDirectory>主要用于控制项目构建时文件的复制行为，即输出到 bin 目录或调试/发布目录。

\<CopyToPublishDirectory> 主要用于控制项目发布时文件的复制行为，即输出到最终发布目录。



#### 9、IStartupFilter添加自定义中间件并调整执行顺序

在ConfigureServices中向IServiceCollection注册IStartupFilter

services.AddTransient<IStartupFilter, MyStartupFilter>();

在IStartupFilter自己的Configure方法中向IApplicationBuilder配置中间件

app.UseMiddleware\<MyCustomMiddleware>();

这样配置的IStartupFilter中间件，会在UseRouting和UseEndpoints等内置中间件之后执行

如果直接在Startup.Configure中配置中间件则能控制执行顺序在其他中间件之前



多个IStartupFilter实现的中间件要控制顺序，按IStartupFilter在服务容器中注册的顺序来控制



#### 10.net 构建主机流程

在应用程序启动时，首先会通过 `WebHostBuilder` 或 `HostBuilder` 创建和配置主机。

在这个过程中，会调用 `ConfigureServices` 方法来注册应用程序的服务，这些服务包括应用程序需要的各种依赖关系和组件。

`IServiceCollection` 接口提供了一系列方法，如 `AddTransient`、`AddScoped`、`AddSingleton` 等，用于向依赖注入容器注册服务。这些服务可以是应用程序自定义的服务，也可以是框架提供的服务。



完成服务的注册后，接下来会调用 `Configure` 方法来配置应用程序的请求处理管道。

`IApplicationBuilder` 接口提供了一系列方法，如 `UseMiddleware`、`UseRouting`、`UseEndpoints` 等，用于按照顺序配置中间件，从而定义 HTTP 请求的处理流程。



#### 11.获取httpContext

服务实现类方法参数添加ServerCallContext context

​        var httpContext = context.GetHttpContext();



#### 12、获取当前登陆用户信息

在AuthenticationHandler中认证成功后封装AuthenticateResult返回

同时通过Request.HttpContext.User = claimPrincipal;设置了用户信息的声明

```c#
private AuthenticateResult SetupPrincipal(User userInfo)
{
    var identity = new ClaimsIdentity(new[]
    {
        new Claim(ClaimTypes.Email,
            userInfo.Email),
        new Claim(ClaimTypes.Name, userInfo.DisplayName),
      ....
    var claimPrincipal = new ClaimsPrincipal(identity);
    var ticket = new AuthenticationTicket(claimPrincipal, new AuthenticationProperties { IsPersistent = false },
        Scheme.Name);
    Request.HttpContext.User = claimPrincipal;
    return AuthenticateResult.Success(ticket);
}
```

在ICurrentPrincipalDataProvider中

通过 _httpContextAccessor.HttpContext?.User.Identity.Claims.SingleOrDefault(x => x.Type == type)?.Value

获取到claims中的用户信息
