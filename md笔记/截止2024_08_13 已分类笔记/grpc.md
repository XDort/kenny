#### gPRC使用

##### 编写Protobuf文件

详见[为 .NET 应用创建 Protobuf 消息](https://learn.microsoft.com/zh-cn/aspnet/core/grpc/protobuf?view=aspnetcore-8.0)

Protobuf IDL 是一种中性语言格式，用于指定 gRPC 服务发送和接收的消息。

```
syntax = "proto3";

option csharp_namespace = "GrpcGreeter";

package greet;

service Greeter {
  
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

##### 将proto文件添加到项目中（客户端和服务端）

```
<ItemGroup>  

	<Protobuf Include="Protos\greet.proto" GrpcServices="Server" /> 

</ItemGroup>
```

默认情况下，`<Protobuf>` 引用将生成具体的客户端和服务基类。 可使用引用元素的 `GrpcServices` 特性来限制 C# 资产生成。 有效 `GrpcServices` 选项如下：

- `Both`（如果不存在，则为默认值）
- `Server`
- `Client`
- `None`

##### 引入 Grpc.Tools包

需要工具包 [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) 才能从 `.proto` 文件生成 C# 资产。 生成的资产（文件）：

- 在每次生成项目时按需生成。
- 不会添加到项目中或是签入到源代码管理中。
- 是包含在 obj 目录中的生成工件。

服务端引用\<PackageReference Include="Grpc.AspNetCore" Version="2.32.0" />就行，包含Grpc.Tools的引入

客户端则引入：

\<PackageReference Include="Google.Protobuf" Version="3.18.0" /> 

\<PackageReference Include="Grpc.Net.Client" Version="2.52.0" />

\<PackageReference Include="Grpc.Tools" Version="2.40.0"> 

​	\<IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive\</IncludeAssets>

​	\<PrivateAssets>all\</PrivateAssets> 

\</PackageReference>



工具包会生成表示在所包含 `.proto` 文件中定义的消息的 C# 类型。

对于服务器端资产，会生成抽象服务基类型。 基类型包含 `.proto` 文件中所含的所有 gRPC 调用的定义。 创建一个派生自此基类型并为 gRPC 调用实现逻辑的具体服务实现。 对于 `greet.proto`（前面所述的示例），会生成一个包含虚拟 `SayHello` 方法的抽象 `GreeterBase` 类型。 具体实现 `GreeterService` 会替代该方法，并实现处理 gRPC 调用的逻辑。

```csharp
public class GreeterService : Greeter.GreeterBase
{
    private readonly ILogger<GreeterService> _logger;
  
    public GreeterService(ILogger<GreeterService> logger)
    {
        _logger = logger;
    }

    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloReply
        {
            Message = "Hello " + request.Name
        });
    }
}
```

GreeterBase： `.proto` 文件生成的基类

对于客户端资产，会生成一个具体客户端类型。 `.proto` 文件中的 gRPC 调用会转换为具体类型中的方法，可以进行调用。一个方法对应一个请求处理

```c#
using var channel = GrpcChannel.ForAddress("https://localhost:7042");

var client = new Greeter.GreeterClient(channel);

var reply = await client.SayHelloAsync(
                  new HelloRequest { Name = "GreeterClient" });
```

##### ASP.NET Core集成gRPC

注册服务

builder.Services.AddGrpc();

添加到路由管道

app.MapGrpcService\<GreeterService>();

