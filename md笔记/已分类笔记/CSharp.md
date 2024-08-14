#### (1) 数据类型

int 有符号整型 4字节 有符号32 位整数

Unit  无符号整型 4字节 无符号32 位整数

sbyte 一半是负数 8位整数

short 16

long 64

char 2字节 存储单个字符 底层是char数组 大小不可变

字符串数值转换

int.Parse(变量) 

123.ToString()

默认小数是双精度double 

字符串拼接

![WeChatcad7e1391fada1d10dc045297493880c](../../../笔记/md笔记/img/WeChatcad7e1391fada1d10dc045297493880c.jpg)

枚举类型 成员不赋值默认按上一个成员的值递增

![WeChat188a76bf291db2da011867ae22424b14](../../../笔记/md笔记/img/WeChat188a76bf291db2da011867ae22424b14.jpg)

枚举类型强转

![WeChat0da8b8472151a25c24fd1fba31cb99b4](../../../笔记/md笔记/img/WeChat0da8b8472151a25c24fd1fba31cb99b4.jpg)

continue和break同java用法

int[] ary = new int[] {1,2,3} 初始化语法

#### (2)结构 类

结构是值类型，它在栈中分配空间；而类是引用类型，它在堆中分配空间，栈中保存的只是引用。都默认继承自object，是所有类型的终极父类

被函数访问修改时，结构深拷贝，类浅拷贝，如果需要结构类型也被修改需要使用关键字ref

描述一个轻量级对象的时候，结构可提高效率（直接访问数值地址，不需要垃圾回收），成本更低。不过假如我们在传值的时候希望传递的是对象的引用地址而不是对象的拷贝，就应该使用类

class默认成员变量是 private的

定义常量用 const 

partial 关键字  用于将一个类、结构体、接口或方法分割成多个部分，允许你在不同的文件中定义同一个实体的不同部分，需要确保每个部分的访问修饰符、方法签名等保持一致

方法签名：方法名称 + 参数个数、类型、顺序

property 属性 看起来像访问成员变量实际是函数调用

![WeChat4e55a203ee4a83c2f6ccb5e95a9ad1a2](../../../笔记/md笔记/img/WeChat4e55a203ee4a83c2f6ccb5e95a9ad1a2.jpg)

![WeChat9c39badf57c94a82899c2da5d76f41e3](../../../笔记/md笔记/img/WeChat9c39badf57c94a82899c2da5d76f41e3.jpg)

#### (3)继承 

Class B: A{

}  代表B类继承自A，继承属性和方法

父类声明protected 变量 子类和自己能访问 实例不能访问

父类声明 virtual 方法，子类可对该方法 用override进行覆盖

struct 不能继承struct 或 class 但可以继承接口

class 可以继承一个或零个基类 (单继承) 可以继承任意多个接口

#### (4)接口

接口只声明函数，不能给函数加修饰符

#### (5)==、Equals、ReferenceEquals

https://juejin.cn/post/6844904067332833293

#### (6)泛型

可以对泛型参数进行约束

#### (7)容器

List 动态数组

Dictionary 键值队

dic[key值] = value 修改对应的value值 

dict.TryGetValue(key, out string val) 如果存在则把key对于的value赋给val

并从容器中去除

dict.Keys dict.Values

HashSet 唯一性



List和ArrayList比较

由于 `List<T>` 是泛型集合类，它在运行时不需要装箱和拆箱操作（boxing/unboxing），因此通常比 `ArrayList` 更加高效。

`ArrayList` 中的元素存储为 `object`，所以当需要从 `ArrayList` 中获取元素并使用时，需要进行类型转换，这会带来额外的性能开销。

`List<T>` 支持 LINQ 查询，可以使用 LINQ 方法来对集合进行过滤、投影、排序等操作。

`ArrayList` 不支持 LINQ，需要手动编写循环来进行类似的操作。



#### (8)delegate 关键字 用于保存多个函数指针 

保存的函数进行回调时会按保存顺序依次调用

 Action 无返回值类型的delegate

Func 有返回值类型的delegate

![WeChat3e58e07288fd5971731503bfe2889922](../../../笔记/md笔记/img/WeChat3e58e07288fd5971731503bfe2889922.jpg)

#### (9)匿名函数

匿名函数操作变量会引用上下文同名变量 闭包

#### (10)attribute 特性

类似JAVA的注解



##### [Obsolete] 标记方法弃用的特性

[Conditional("DEBUG")]  定义了#DEBUG 这个宏后

标记了该特性的方法不被调用



##### 调用者信息相关的特性

[CallerLineNumber]、CallerLineNumber、CallerFilePath

需要在参数前标记特性，并初始化参数

`public void LogMessage(string message, [CallerLineNumber] int lineNumber = 0)
{
    Console.WriteLine($"Line {lineNumber}: {message}");
}`



##### 自定义特性

编写sealed修饰的特性类（以Attribute结尾命名）继承系统提供的Attribute类

在该特性类上使用特性[AttributeUsage(AttributeTargets.Xxx)]

表明可使用该特性的地方



Type.IsDefined() 用于判断某个类是否标记了某个特性，

加参数可判断是否标记了其父类特性



#### (11)反射

通过Typeof获取Type类

可以获取到类的字段、属性、方法等信息，可以绕过限制访问私有成员

#### (12)宏

在using前定义

#if 定义了对应宏执行后续代码 #else #endif 结束

#### (13)所有异常基类Exception

#### (14)扩展方法

```csharp
namespace ExtensionMethods
{
    public static class MyExtensions
    {
        public static int WordCount(this string str)
        {
            return str.Split(new char[] { ' ', '.', '?' },
                             StringSplitOptions.RemoveEmptyEntries).Length;
        }
    }
}
```

可使用此 `WordCount` 指令将 `using` 扩展方法置于范围中：

```csharp
using ExtensionMethods;
```

而且，可以使用以下语法从应用程序中调用该扩展方法：

```csharp
string s = "Hello Extension Methods";
int i = s.WordCount();
```

#### (15)LINQ 详见LINQ.md

#### (16)顶级语句

顶级语句

C# 9.0之后Program.cs使用了顶级语句（语法糖），隐式的生成Main方法，它会根据其中的代码环境来生成Main方法的签名，可以直接使用args 变量

using System;

namespace Application
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}

优化成一行可执行代码
Console.WriteLine("Hello, World!");

#### (17)params参数数组

params参数数组，允许方法接受可变数量的参数

`params` 关键字只能用于方法的最后一个参数位置。

方法只能有一个 `params` 参数。

如果方法有多个重载，编译器会优先匹配非 `params` 参数的重载，而不是将所有参数都放入 `params` 数组中。

`params` 参数可以是任何类型，包括自定义类和接口。

#### (18) 隐藏方法和虚方法的区别

隐藏方法父类方法不声明virtual，子类重写要加new声明方法

只要变量是哪个类的，就调用那个类的方法，

也就是只有子类变量才能调用子类重写的方法

而虚方法，父类方法声明virtual，子类重写加override

调用哪个类的方法取决于new 的对象是哪个类的

#### (19)访问权限

访问权限

public

可以在任何地方访问，包括其他程序集（DLL）中的代码。

private

只能在定义该成员的类或结构体内部访问。

protected

可以在定义该成员的类及其派生类中访问。

internal

只能在同一程序集（DLL）中的代码中访问。默认

protected internal

同一程序集都可以

对于不在同一程序集中的代码，只有派生自包含 `protected internal` 成员的类的代码可以访问。



#### 闭包的概念

```
int multiplier = 2;

Func<int, int> multiply = x => x * multiplier;

Console.WriteLine(multiply(3)); // 输出 6
```

这个 lambda 表达式会被编译器转换成类似下面的类：

```
public class DisplayClass
{
    public int multiplier;

    public int Multiply(int x)
    {
        return x * this.multiplier;
    }
}
```

使用这个类的代码大概如下：

```
DisplayClass displayClass = new DisplayClass();

displayClass.multiplier = 2;

Func<int, int> multiply = displayClass.Multiply;

Console.WriteLine(multiply(3)); // 输出 6
```

闭包的实现

闭包是指在函数内部定义的函数可以捕获和保存其外部范围的变量。编译器通过生成一个包含这些捕获变量的类来实现闭包。

捕获的变量

在上面的例子中，`multiplier` 是一个被捕获的变量。编译器会将捕获的变量转换成类的成员变量，并将 lambda 表达式转换成这个类的方法。

捕获变量的生命周期

捕获的变量的生命周期与闭包对象的生命周期一致。这意味着即使在 lambda 表达式执行完成后，只要闭包对象仍然存在，捕获的变量也仍然存在。



#### select new {}语法

`select new {}` 语法用于创建匿名类型，编译器会自动根据你提供的值推断属性类型。

例子：

```c#
var query = from rUser in _dbContext.Set<RUser>()
    join user in _dbContext.Set<User>() on rUser.UserId equals user.Id
    select new { rUser, userDto = _mapper.Map<UserDto>(user) };

var result = await query.ToListAsync(cancellationToken).ConfigureAwait(false);
```

连接了rUser和User两张表，返回包含两个实体类的匿名类型，其中对user实体做了映射

执行查询后返回的是一个匿名类型的集合，应该使用LINQ表达式转换为元组类型的集合返回

```c#
return result.Select(x => (x.rUser, x.userDto)).ToList()
```



#### 匿名类型 vs 元组

`List<{UserDto userDto, RUser rUser}>`

- 这是一个匿名类型（Anonymous Type）的列表。
- 匿名类型具有命名属性，例如 `userDto` 和 `rUser`。
- 它们是在编译时生成的，只能在方法体内使用，因为匿名类型无法跨方法或类边界传递。

`List<(UserDto, rUser)>`

- 这是一个元组（Tuple）的列表。
- 元组类型的属性是按位置访问的，例如 `Item1` 和 `Item2`。
- 元组可以跨方法或类边界传递，并且可以通过使用元组解构来方便地访问它们的元素。



#### 语法糖

只读属性

```c#
public string str => "string";
```

这个属性的行为相当于：

```c#
private readonly string _str = "string";    

public string str    
{       

	 get { return _str; }    

}
```



#### 检查集合属性

`orderItems is { Count: > 0 }`

 这是 C# 8.0 中引入的一种**模式匹配**语法，用于检查 `orderItems` 集合是否为空。具体来说，这种语法检查 `orderItems` 是否是一个实现了 `Count` 属性的集合，并且其 `Count` 属性的值大于 `0`。Count:  0 是否等于0

⚠️如果orderItems该集合为null，会直接返回false



#### yield实现简易迭代器，迭代器调用时机测试

```c#
foreach (var i in new int[] { 1, 2, 3 })
{
    Console.WriteLine("第" + i + "次");
  
    var enumerator = TestEnumerator(i);

    foreach (var j in enumerator)
    {
        Console.WriteLine(j);
    }
}

IEnumerable<int> TestEnumerator(int i)
{
    if (i % 2 == 1)
    {
        yield return 3;
        yield return 1;
    }
    else
    {
        yield return 2;
        yield return 0;
    }

    yield return -1;
}
```

通过断点打到foreach (var j in enumerator)这一行，可以得知，每通过循环调用一次TestEnumerator，都会执行一次TestEnumerator方法通过yield return返回一个IEnumerable\<int>，比如第一次循环yield return 3; 第二次会根据状态机保存位置继续执行，直到遇到yield return语句，即yield return 1;

即每执行一次IEnumerable中的MoveNext方法yield return 返回一次



#### as关键字

`as` 关键字在 C# 中用于进行**引用类型**或**可空类型**的转换。与传统的类型转换操作符 `(T)` 不同，`as` 关键字在转换失败时不会抛出异常，而是返回 `null`。这种方式在处理可能失败的类型转换时更加安全和高效。



#### 匿名类型自动重写的方法

匿名类型在创建时自动重写了 `Equals` 和 `GetHashCode` 方法，因此在使用匿名类型的集合时无需显式重写这些方法。
