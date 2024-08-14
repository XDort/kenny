#### 构建镜像部署容器

阶段构建允许你在一个 Dockerfile 中使用多个 `FROM` 指令来创建多个构建阶段。每个阶段都可以基于不同的基础镜像。

通过使用 `AS` 关键字，你可以为每个构建阶段指定一个名称，之后可以在同一个 Dockerfile 中引用这些阶段。



**`aspnet:8.0`**: 用于运行 ASP.NET Core 应用程序，适合用于生产环境，体积较小，只包含运行时。

**`sdk:8.0`**: 用于开发和编译 .NET 应用程序，包含所有开发和编译所需的工具，体积较大，通常用于构建阶段。



EXPOSE 5099 容器中暴露的端口



WORKDIR /app 指定当前工作目录

如果不存在目录，WORKDIR 命令会自动创建指定的工作目录



COPY ["主机上的文件或路径", "容器内的文件或目录路径"]

COPY命令也会自动创建缺失的目录



`COPY . .` 是 Dockerfile 中的一条常见指令，用于将当前目录（`Dockerfile` 所在的目录）下的所有内容复制到容器的当前工作目录。

### 

在 Dockerfile 中使用 `COPY ["src/xxx.Api/xxx.Api.csproj", "src/xxx.Api/"]` 这样的指令

是为了优化构建过程，特别是在处理依赖还原（`dotnet restore`）时。这种做法可以充分利用 Docker 的缓存机制，加速镜像构建。

通过单独复制 `.csproj` 文件并提前执行 `dotnet restore`，可以显著优化镜像构建过程。



```
RUN dotnet build "xxx.Api.csproj" -c Release -o /app/build
```

-c（--configuration）：指定构建的配置。常见的配置包括 `Debug` 和 `Release`。默认情况下，`dotnet build` 使用 `Debug` 配置。

-o（--output）：指定构建输出的目录。通过这个选项，你可以将编译生成的文件输出到指定的目录，而不是默认的 `bin` 目录。



Api project依赖了core和message，publish只需指定api下的csproj文件