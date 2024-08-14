1、集成DbUp

导入nuget包 dbup-core、dbup-mysql



2、编写DbUpRunner配置类

配置数据库、指定sql脚本输出路径

```
var upgradeEngine = DeployChanges.To.MySqlDatabase(_connectionString)
  .WithScriptsAndCodeEmbeddedInAssembly(typeof(DbUpRunner).Assembly, s => s.EndsWith(".cs")) //把嵌入在程序集中以cs结尾的脚本添加到迁移中
    .WithScriptsFromFileSystem(outPutDirectory) //从指定目录加载脚本
    .WithTransaction() //迁移时开启事务
    .LogToAutodetectedLog()
    .LogToConsole()
    .Build();
```



3、在csproj中添加配置

    <ItemGroup>
    
        <Content Include="DbUp\*\*.sql">
    
            <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    
        </Content>
    
    </ItemGroup>

作用是：

- **将 `.sql` 文件包含到项目中**：确保项目中的所有 `.sql` 文件都被包含在项目中，使得它们可以在开发和构建过程中被管理和使用。
- **复制到输出目录**：通过 `CopyToOutputDirectory` 的设置，确保在每次构建项目时，`.sql` 文件都会被复制到输出目录（例如 `bin\Debug` 或 `bin\Release` 目录）。这样，在运行时，这些 `.sql` 文件可以在输出目录中被访问到，例如用于数据库迁移、初始化脚本等操作。



4、在指定的目录写编写sql脚本或使用DbUp提供的继承IScript的方式编写脚本



5、在program.cs配置在程序启动时启动DbUpRunner执行脚本

```
var configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json")
    .AddEnvironmentVariables()
    .Build();

new DbUpRunner(new ConnectionString(configuration).Value).Run();
```

创建了IConfiguration，用于从appsetting.json或环境变量中读取应用程序的配置信息