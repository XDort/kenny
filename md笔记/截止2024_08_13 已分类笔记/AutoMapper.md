踩坑

```
CreateMap<Foods, FoodDto>();
```

这样配置映射是支持映射返回集合的，不用多加个List<>不然会报错

```
return _mapper.Map<List<FoodDto>>(foods);
```



#### 项目中AutoMapper的高级用法

主要使用到了ForAllMaps和ForAllPropertyMaps这两个automapper的内部api，没有官方文档，官方认为这两个api与automapper太耦合了

https://docs.automapper.org/en/latest/11.0-Upgrade-Guide.html#forallmaps-forallpropertymaps-advanced-and-other-missing-apis

用于在映射实体时配置多语言，详见YamimealMappingProfile.cs，该类作为其他mapping类的基类



PickupPlatformModule.cs中的RegisterAutoMapper则调用了SetupMultiLanguageAfterMap方法用于配置多语言

和识别IgnoreMapAttribute特性用于忽略某些属性的映射



SetupMultiLanguageAfterMap方法详解：

首先检查实体是否可映射为多语言实体（多语言实体要实现IHaveMultiLanguageField）

```c#
if (!typeof(IEntity).IsAssignableFrom(map.SourceType) ||
    !typeof(IHaveMultiLanguageField).IsAssignableFrom(map.DestinationType)) 
    return;
```



再从请求头中获取languageCode

```c#
string languageCode = _httpContextAccessor.HttpContext?.Request?.Headers["language_code"];
```



autofac还没初始化好，RequestServices获取IServiceProvide容器直接解析IMemoryCache

```c#
var memoryCache =
    _httpContextAccessor.HttpContext?.RequestServices?.GetService(typeof(IMemoryCache)) as IMemoryCache;
```



默认使用英文，英文无需翻译

```c#
if (string.IsNullOrEmpty(languageCode) ||
    languageCode == MultiLanguagesService.DefaultLanguageCode || memoryCache == null)
    return;
```



```c#
var entityType = src.GetType().FullName; //src实体的全类名
var propsWithMultiLangAttribute = dest.GetType().GetProperties().Where( //拿到标记了特性的属性集合
    prop => Attribute.IsDefined(prop, typeof(MultiLanguageField)));
```



遍历propsWithMultiLangAttribute设置对应多语言

```c#
propsWithMultiLangAttribute.ForEach(prop =>
{
    var att =prop.GetCustomAttribute<MultiLanguageField>();
    if (att is null)
        return;
    var matchedPropertyName =
        string.IsNullOrEmpty(att.FromEntityPropertyIfNamesAreDifferent)
            ? prop.Name //使用属性名
            : att.FromEntityPropertyIfNamesAreDifferent; //使用特性标记时取的别名
    var entityId = ((IEntity) src).Id;
    var key = MultiLanguagesService.GetCacheKey(languageCode, entityType, matchedPropertyName, entityId);
            //获取根据参数拼接的字符串key
    if(memoryCache.TryGetValue(key, out EntityLanguage entityLanguage)) //从内存中拿到key对应entityLanguage
    {
        if (entityLanguage == null) return;
                
        if (prop.PropertyType.IsAssignableFrom(typeof(IEnumerable<string>)))
        {
            prop.SetValue(dest, 
                JsonConvert.DeserializeObject<IEnumerable<string>>(entityLanguage.FieldValue)); 
          			//反序列化拿到翻译后的字符串并更新到目标实体
        }
        else
        {
            prop.SetValue(dest, entityLanguage.FieldValue);
        }
    }
    else
    {
        if (!att.SetFieldValueWhenIsNullOrEmpty) return;
                
        prop.SetValue(dest,
            prop.PropertyType.IsAssignableFrom(typeof(IEnumerable<string>))
                ? new List<string>()
                : string.Empty);
    }
});
```



多语言内存缓存相关参见MultiLanguagesService.cs的InitializeMultiLanguagesCache方法，由AdminController.cs的MemoryInit方法控制缓存的初始化
