#### LINQ

linq 常用API

##### 查询方法

（1）**`Select`**：

- 用于从序列中选择特定的数据。

```c#
var names = people.Select(p => p.Name);
```

（2）**`Where`**：

- 根据指定条件筛选序列中的元素。

```c#
var adults = people.Where(p => p.Age >= 18);
```

（3）**`OrderBy`** 和 **`OrderByDescending`**：

- 用于对序列进行升序或降序排序。

```c#
var sortedNames = names.OrderBy(n => n);
var sortedNamesDesc = names.OrderByDescending(n => n);
```

（4）**`GroupBy`**：

- 根据指定的键对序列中的元素进行分组。

```c#
var groupedPeople = people.GroupBy(p => p.City);
```

（5）**`Join`** 和 **`GroupJoin`**：

- `Join`用于将两个序列中的元素关联起来，而`GroupJoin`则将一个序列中的元素与另一个序列中匹配的元素进行分组。

```c#
var query = people.Join(cities,
                        person => person.CityId,
                        city => city.Id,
                        (person, city) => new { Person = person, CityName = city.Name });

var queryGrouped = people.GroupJoin(cities,
                                    person => person.CityId,
                                    city => city.Id,
                                    (person, cityGroup) => new { Person = person, Cities = cityGroup });
```

##### 聚合方法

（1）**`Count`**：

- 返回序列中元素的数量。

```c#
var count = people.Count();
```

（2）**`Sum`**、**`Average`**、**`Min`**、**`Max`**：

- 分别用于计算序列中元素的总和、平均值、最小值和最大值。

```c#
var totalAge = people.Sum(p => p.Age);
var averageAge = people.Average(p => p.Age);
var minAge = people.Min(p => p.Age);
var maxAge = people.Max(p => p.Age);
```

##### 其他方法

（1）**`First`**、**`FirstOrDefault`**、**`Single`**、**`SingleOrDefault`**：

- 用于获取序列中的第一个元素或满足条件的唯一元素。

```c#
var firstPerson = people.First();
var firstAdult = people.First(p => p.Age >= 18);
var singlePerson = people.Single(p => p.Id == 1);
var singleOrDefaultPerson = people.SingleOrDefault(p => p.Id == 1);
```

（2）**`Skip`** 和 **`Take`**：

- `Skip`用于跳过序列中指定数量的元素，`Take`用于从序列中获取指定数量的元素。

```c#
var skippedPeople = people.Skip(5);
var takenPeople = people.Take(10);
```



#### 内连接组合多个键

```c#
IEnumerable<string> query =
    from teacher in teachers
    join student in students on new
    {
        FirstName = teacher.First,
        LastName = teacher.Last
    } equals new
    {
        student.FirstName,
        student.LastName
    }
    select teacher.First + " " + teacher.Last;
```

确定哪些教师同时还是学生

或使用join方法

```c#
IEnumerable<string> query = teachers
    .Join(students,
        teacher => new { FirstName = teacher.First, LastName = teacher.Last },
        student => new { student.FirstName, student.LastName },
        (teacher, student) => $"{teacher.First} {teacher.Last}"
 );
```



#### 左外联接

from xxx in xxx 遍历某个集合

into 临时存储查询结果

可以通过对分组联接的结果调用 [DefaultIfEmpty](https://learn.microsoft.com/zh-cn/dotnet/api/system.linq.enumerable.defaultifempty) 方法来执行左外部联接

```c#
var query =
    from student in students
    join department in departments on student.DepartmentID equals department.ID into gj
    from subgroup in gj.DefaultIfEmpty()
    select new
    {
        student.FirstName,
        student.LastName,
        Department = subgroup?.Name ?? string.Empty
    };
```

以上例子将查询所有学生的姓名和公寓名，没有关联公寓则公寓名为空



#### selectMany 用法

可以完全用linq语法替代 详见 https://www.cnblogs.com/cncc/p/9840463.html

```c#
List<Person> personList = new List<Person>
{
    new Person
    {
        Name = "P1", Age = 18, Gender = "Male",
        Gogs = new Dog[]
        {
            new Dog { Name = "D1" },
            new Dog { Name = "D2" }
        }
    },
    new Person
    {
        Name = "P2", Age = 19, Gender = "Male",
        Gogs = new Dog[]
        {
            new Dog { Name = "D3" }
        }
    },
    new Person
    {
        Name = "P3", Age = 17,Gender = "Female",
        Gogs = new Dog[]
        {
            new Dog { Name = "D4" },
            new Dog { Name = "D5" },
            new Dog { Name = "D6" }
        }
    }
};
var results = personList.SelectMany(p => p.Dogs, (p, d) => new { PersonName = p.Name, DogName = d.Name });
foreach (var result in results)
{
    Console.WriteLine($"{result.PersonName},{result.DogName}");
}
```

这种重载中，d指向 p => p.Dogs 这个已筛选的大集合，d和p作为参数构建匿名对象的新集合返回

方法声明：

```c#
public static IEnumerable<TResult> SelectMany<TSource, TCollection, TResult>
  (this IEnumerable<TSource> source, Func<TSource, IEnumerable<TCollection>> collectionSelector, Func<TSource, TCollection, TResult> resultSelector);
```



#### Linq中连用两个from 关键字的情况

嵌套集合展开：

```C#
List<List<string>> listOfLists = new List<List<string>>
{
    new List<string> { "apple", "banana" },
    new List<string> { "cherry", "date" },
    new List<string> { "fig", "grape" }
};

var flattenedList = from list in listOfLists
                    from fruit in list
                    select fruit;
```

生成笛卡尔积：

```c#
List<int> numbers = new List<int> { 1, 2, 3 };
List<string> letters = new List<string> { "A", "B", "C" };

var combinations = from number in numbers
                   from letter in letters
                   select new { number, letter };
```

遍历嵌套集合进行组合：

```c#
List<Student> students = new List<Student>
{
    new Student { Name = "Alice", Courses = new List<string> { "Math", "Science" } },
    new Student { Name = "Bob", Courses = new List<string> { "History", "Math" } },
    new Student { Name = "Charlie", Courses = new List<string> { "Math", "Science", "History" } }
};

var studentCourses = from student in students
                     from course in student.Courses
                     select new { student.Name, course };
```



#### let字句

[官方文档](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/let-clause)

该关键字创建一个新的范围变量并通过提供的表达式结果初始化该变量。 使用值进行初始化后，范围变量不能用于存储另一个值。



#### ThenBy 字句

ThenBy是OrderBy的扩展方法，用于指定多个排序字段



#### Enumerable 类方法和Queryable类方法

[官方文档](https://learn.microsoft.com/en-us/dotnet/api/system.linq.queryable?view=net-8.0)

Orderby方法 默认按字符的Unicode值进行比较



#### LINQ筛选器的使用时机

通过在Dbcontext 的OnConfiguring方法配置控制台输出log

```c#
var foodsEnumerable = _db.Set<Foods>().AsEnumerable().Where(x => x.Name == "没有异常的任务");

foreach (var foods in foodsEnumerable)
{
    Console.WriteLine(foods.Name);
}

return Ok();
```

断点打到foreach时，不会立即执行查询，且由于是在AsEnumerable()方法后调用的筛选器，会在查询全部数据后再进行筛选

而使用ToList()方法则会立即查询

在AsEnumerable和ToList之前执行的筛选会添加数据库查询条件

在它们之后执行的筛选是在应用内存中执行的

数据量小的时候可以用AsEnumerable再进行筛选

主要还是注意查询走索引