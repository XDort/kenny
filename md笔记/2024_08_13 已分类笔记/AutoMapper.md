踩坑

```
CreateMap<Foods, FoodDto>();
```

这样配置映射是支持映射返回集合的，不用多加个List<>不然会报错

```
return _mapper.Map<List<FoodDto>>(foods);
```

