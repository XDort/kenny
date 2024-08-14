包管理器是 NuGet。 它类似于 java中的Maven



#### windows下C盘空间不足，修改Rider缓存和nuget仓库位置

setting搜索file，将所有可自定义的路径修改到C盘外的文件夹



查看相关nuget缓存位置

![240803a](../../../笔记/md笔记/img/240803a.png)

进入以上位置修改NuGet.config

添加

```
<configuration>
  <config>
    <add key="globalPackagesFolder" value="D:\CustomNuGetCache" />
  </config>
</configuration>
```



HTTP Cache仍需要手动清理