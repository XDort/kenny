#### 1、在使用了autofac下，配置EFCore连接Mysql

（1）导nuget包

Microsoft.EntityFrameworkCore

Pomelo.EntityFrameworkCore.MySql

（2）配置appsettings.json并为依赖注入读取做准备

在appsettings.json中配置mysql连接

可以去https://www.connectionstrings.com/ 查看对应版本的模版

创建IConfigurationSetting接口，有一个可以指定泛型属性的子接口

```
public interface IConfigurationSetting
{
}

public interface IConfigurationSetting<TValue> : IConfigurationSetting
{
    TValue Value { get; set; }
}
```

创建一个ConnectionString类，用于从appsetting中读取ConnectionStrings

```
public class ConnectionString : IConfigurationSetting<string>
{
    public string Value { get; set; }
    
    public ConnectionString(IConfiguration configuration)
    {
        Value = configuration.GetConnectionString("Default");
    }
}
```

（3）创建实体类

创建IEntity基类，提供有Id属性的泛型子接口

创建测试实体类继承基类，通过特性指定Id为数据库自增主键，指定表名



（4）DbContext配置数据库连接和实体类

编写自己的Context类继承EF提供的DbContext

构造函数注入ConnectionString配置类

重写OnConfiguring方法配置数据库连接信息

`optionsBuilder.UseMySql(_connectionString.Value,new MySqlServerVersion(new Version(8,4,0)));`

重写OnModelCreating方法

```
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    typeof(PractiseForKennyDbContext).GetTypeInfo().Assembly.GetTypes()
        .Where(t => typeof(IEntity).IsAssignableFrom(t) && t.IsClass).ToList()
        .ForEach(x =>
        {
            if (modelBuilder.Model.FindEntityType(x) == null)
                modelBuilder.Model.AddEntityType(x);
        });
}
```

把PractiseForKennyDbContext所在程序集的中实现了 `IEntity` 接口的所有类，作为数据库模型的实体类型添加到 `modelBuilder` 中



（5）在autofac的module类注册ConnectionString类和自己编写的DbContext类

```
private void RegisterDbContext(ContainerBuilder builder)
{
    builder.RegisterType<PractiseForKennyDbContext>()
        .AsSelf()
        .As<DbContext>()
        .AsImplementedInterfaces()
        .InstancePerLifetimeScope();
}

private void RegisterSettings(ContainerBuilder builder)
{
    var settingTypes = typeof(PractiseForKennyDbContext).Assembly.GetTypes()
        .Where(t => t.IsClass && typeof(IConfigurationSetting).IsAssignableFrom(t))
        .ToArray();

    builder.RegisterTypes(settingTypes).AsSelf().SingleInstance();
}
```



（6）编写控制器和实现类

在实现类注入自己编写的DbContext类

在方法中通过注入的DbContext类获取dbset类

```
var dbset = _practiseForKennyDbContext.Set<Foods>();
```

再调用对应api进行数据库操作



