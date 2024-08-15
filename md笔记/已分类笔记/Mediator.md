## Mediator

### (1)理解

传统模式请求来到controller，controller需要引入不同类来实现业务处理，不同类之间耦合程度高。

而引入Mediator后，请求来到controller先解析请求参数为command或request，然后发送message给Mediator，Mediator再从它的注册表中寻找对应的commandHandler，由commandHandler去调用具体的Services，Services的实现类中再进行具体的逻辑实现，处理完后在实现类中返回event给commandHandler，commandHandler再异步给eventHandler发布一个订阅，用于某些状态改变后的处理，处理完后commandHandler构建response返回controller

由此controller不再直接依赖于各种类，便于测试和改动。

在请求转发到handler前，又应用了pipeline的思想，可以用pipeline中一系列的middleware来做一些处理，比如logging，unit of work

### (2)引入Mediator

在api、core、message层引入Mediator.Net

在api、core层引入Mediator.Net.Autofac

#### `Command` 和 `Request`

在 Mediator.Net 中，`Command` 和 `Request` 是两种不同的消息模式

- `Command` 是一种单向的消息，即发送者发送 `Command` 消息后，不会直接接收到响应或者返回值。

  `Command` 通常用于执行命令式操作，例如创建、更新或删除数据，或者执行特定的业务逻辑。

- `Request` 是一种双向的消息，发送者发送 `Request` 消息后，通常会收到一个响应消息或者返回值。

  `Request` 通常用于查询数据或者获取系统状态信息。

#### 改造hello world接口

在message层创建HelloWorldCommand类定义请求要传递的属性

```
public class HelloWorldCommand : ICommand
{
    public string Message { get; set; }
}
```

⚠️继承Mediator.Net提供的ICommand接口

在message层创建HelloWorldRequest类文件定义请求的响应类(controller不需要request不用写request类)

```
public class HelloWorldResponse : IResponse
{
    public string Message { get; set; }
}
```

ps. 项目规范貌似Request类的一个文件允许同时定义request和repose类



在message层创建HelloWorldEvent类

```
public class HelloWorldEvent : IEvent
{
    public string Message { get; set; }
}
```



在core层引用对message层

PractiseForKenny.Core.csproj :

```
<ItemGroup>
  <ProjectReference Include="..\PractiseForKenny.Messages\PractiseForKenny.Messages.csproj" />
</ItemGroup>
```



在Core层创建Service接口和实现类

接口：

```
public interface IHelloWorldService : IService
{ 
    HelloWorldEvent HelloWorld(HelloWorldCommand command, CancellationToken cancellationToken);
}
```

返回值是IEvent类型，参数有ICommand类型和CancellationToken

实现：

```
public HelloWorldEvent HelloWorld(HelloWorldCommand command, CancellationToken cancellationToken)
{
    return new HelloWorldEvent { Message = "Hello World!" };
}
```



在core层创建HelloWorldCommandHandler

实现ICommandHandler接口，指定入参出参泛型，注入Service类，实现handle方法，

```
public class HelloWorldCommandHandler : ICommandHandler<HelloWorldCommand, HelloWorldResponse>
{
    
    private readonly IHelloWorldService _helloWorldService;

    public HelloWorldCommandHandler(IHelloWorldService helloWorldService)
    {
        _helloWorldService = helloWorldService;
    }
    
    public async Task<HelloWorldResponse> Handle(IReceiveContext<HelloWorldCommand> context, CancellationToken cancellationToken)
    {
        var @event = _helloWorldService.HelloWorld(context.Message, cancellationToken);

        await context.PublishAsync(@event, cancellationToken).ConfigureAwait(false);

        return new HelloWorldResponse()
        {
            Message = @event.Message
        };
    }
}
```



在core层创建HelloWorldEventHandler

实现IEventHandler接口，指定IEvent类泛型，实现handle方法

```
public class HelloWorldEventHandler : IEventHandler<HelloWorldEvent>
{
    public Task Handle(IReceiveContext<HelloWorldEvent> context, CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }
}
```



在autofac的Module类把Mediator和它的所有Handler注册进容器

```
private void RegisterMediator(ContainerBuilder builder)
{
    var mediatorBuidler = new MediatorBuilder();

    mediatorBuidler.RegisterHandlers(_assemblies);

    builder.RegisterMediator(mediatorBuidler);
}
```



在api层创建HelloWorldController

注入Mediator，通过Mediator发送异步消息给commandHandler执行Services的调用

```
private readonly IMediator _mediator;

public HelloWorldController(IMediator mediator)
{
    _mediator = mediator;
}

[Route("helloWorld"), HttpGet]
public async Task<IActionResult> HelloWorldAsync([FromQuery] HelloWorldCommand command)
{
    var response = await _mediator.SendAsync<HelloWorldCommand, HelloWorldResponse>(command).ConfigureAwait(false);

    return Ok(response);
}
```



测试跑通👏