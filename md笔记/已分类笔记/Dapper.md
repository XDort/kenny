#### Dapper简单使用

- `Query`: 用于返回一个泛型列表，其类型由查询返回的列定义。
- `QueryFirstOrDefault`: 用于返回查询结果中的第一行。
- `QueryMultiple`: 用于同时执行多个查询，并返回多个结果集。
- `Execute`: 用于执行没有返回结果的查询。

通过一个`IDbConnection`对象调用。

```csharp
            // 查询年龄大于等于30的人的信息
            var adults = connection.Query<Person>("SELECT * FROM Persons WHERE Age >= @Age", new { Age = 30 });
            foreach(var adult in adults)
            {
                Console.WriteLine("{0} {1}, Age {2}", adult.FirstName, adult.LastName, adult.Age);
            }
            
            // 插入一行数据
            var result = connection.Execute("INSERT INTO Persons (FirstName, LastName, Age) VALUES (@FirstName, @LastName, @Age)", new { FirstName = "Jane", LastName = "Doe", Age = 25 });
            Console.WriteLine("{0} rows affected", result);
```

