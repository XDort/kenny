### 项目中ElasticSearch

https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/usage.html

注册ElasticSearch客户端服务

ElasticsearchExtensions扩展方法类

其中RegisterElasticClient方法用于在module中注册ES客户端，配置完成后注册IElasticClient服务，用于后续注入后执行ES操作

该类其他方法都是通过工厂方法注册各种ES的Service类

```c#
builder.Register<Func<string, IEnumerable<IElasticsearchSyncService>>>
```

通过autofac注册一个委托（简易工厂方法）



初始化项目Elasticsearch索引

ElasticsearchStartInitializationService类继承 autofac的IStartable接口和用于统一注册服务的IService接口

IStartable接口主要作用是在容器构建完成后立即执行一些初始化逻辑。实现该接口的类会在容器构建完成后被自动创建和启动，需要实现start()方法。

```c#
public void Start()
{
    if(_runTimeProvider.Type!= RunTimeType.Worker)
        return;
    
    Console.WriteLine("in ElasticsearchStartInitializationService Start");

    var initIndicesMethodInfos = this.GetType().GetMethods().Where(x => x.Name.Contains("Indices"));

    initIndicesMethodInfos.ToList().ForEach(m =>
    {
        try
        {
            Console.WriteLine($"{m.Name} begin");
            m.Invoke(this, null);
        }
        catch (Exception e)
        {
            Log.Error(e, "ElasticsearchStartInitializationService Error");
        }
    });
}
```

以上代码通过反射调用当前类的其他方法初始化ES中的索引



.DisableDirectStreaming()

禁用直接流式传输以捕获请求和响应字节。在生产应用程序中，您可能不会调用它，因为它会增加每次调用的开销。

一般用于debug



### ES中的索引

每个索引可以设置为静态的或者是动态的

静态索引：只能在创建的时候和关闭了的索引上设置静态索引

动态索引：可以使用 update-index-settings API 在实时索引上更改它们



更改封闭索引上的静态或动态索引设置可能会导致不正确的设置，如果不删除并重新创建索引，则无法纠正这些设置。



在 Elasticsearch（ES）中，文档、索引和分片是三个核心概念，它们之间有着密切的关系。理解它们的关系有助于更好地设计和管理 Elasticsearch 集群。

### 文档（Document）

- **基本单元**：文档是 Elasticsearch 中的基本数据单元。它是一个 JSON 格式的对象，包含各种字段和对应的值。
- **唯一标识**：每个文档都有一个唯一标识符 `_id`，用于在索引中唯一标识这个文档。

### 索引（Index）

- **文档的集合**：索引是文档的集合，它相当于关系数据库中的表（Table）。每个索引包含一个或多个文档，这些文档具有相似的结构。
- **唯一标识**：每个索引都有一个唯一的名字，这个名字用于在集群中引用这个索引。

### 分片（Shard）

- **分布存储**：分片是索引数据的物理存储单元。为了实现数据分布和提高查询性能，索引的数据会被拆分成多个更小的部分，这些部分就是分片。

- 两种类型

  ：

  - **主分片（Primary Shard）**：存储实际的数据，每个文档都存在一个且只存在一个主分片上。
  - **副本分片（Replica Shard）**：主分片的副本，用于故障转移和提高查询性能。

### 关系

1. **一个文档属于一个索引**：每个文档都属于一个索引，存储在该索引的一个主分片上。
2. **一个索引由多个分片组成**：为了扩展和并行处理，索引的数据被分成多个主分片，每个主分片可以有多个副本分片。
3. **分片的分布**：分片被分布到集群中的多个节点上，这样可以平衡负载和提高容错能力。

### 图示

```
plaintext
复制代码
Elasticsearch 集群
  ├── 节点1
  |    ├── 分片1 (主分片)
  |    └── 分片2 (副本分片)
  ├── 节点2
  |    ├── 分片2 (主分片)
  |    └── 分片3 (副本分片)
  ├── 节点3
       ├── 分片3 (主分片)
       └── 分片1 (副本分片)

索引（my_index）
  ├── 主分片1
  |    ├── 文档1
  |    └── 文档2
  ├── 主分片2
  |    ├── 文档3
  |    └── 文档4
  └── 主分片3
       ├── 文档5
       └── 文档6
```

### 举例说明

假设我们有一个名为 `my_index` 的索引，包含以下文档：

```
json
复制代码
{
  "_id": "1",
  "title": "Document 1"
},
{
  "_id": "2",
  "title": "Document 2"
}
```

- **索引的创建**：在创建索引时，我们可以指定主分片和副本分片的数量。例如：

  ```
  json
  复制代码
  PUT /my_index
  {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1
    }
  }
  ```

  这里，我们创建了一个名为 `my_index` 的索引，包含 3 个主分片和 1 个副本分片。

- **文档的分配**：文档会根据其 `_id` 的哈希值被分配到不同的主分片上。例如：

  - 文档 `{"_id": "1", "title": "Document 1"}` 可能被分配到主分片1上。
  - 文档 `{"_id": "2", "title": "Document 2"}` 可能被分配到主分片2上。

- **副本分片的作用**：为了实现高可用性和数据冗余，每个主分片会有一个副本分片。副本分片存储在不同的节点上。例如：

  - 主分片1 的副本分片可能存储在节点2上。
  - 主分片2 的副本分片可能存储在节点3上。



### 静态索引的设置

**index.number_of_shards**

索引应具有的主分片数量。默认为 1。此设置只能在创建索引时设置。它不能在关闭的索引上更改。



**index.number_of_routing_shards**

用于将文档路由到主分片，Elasticsearch 在拆分索引时使用此值。

拆分索引（Split Index）是 Elasticsearch 提供的一项功能，用于将一个现有的索引分裂为多个分片更多的索引。这个功能在索引的分片数量需要增加以应对数据增长时特别有用。通过拆分索引，您可以提高索引的容量和并行处理能力，而无需重新索引数据。

拆分索引 API 允许您将现有索引拆分为新索引，其中每个原始主分片在新索引中拆分为两个或多个主分片。

索引可以拆分的次数（以及每个原始分片可以拆分成的分片数量）由index.number_of_routing_shards 设置决定。路由分片的数量指定内部使用的哈希空间，以通过一致的哈希值跨分片分发文档。例如，number_of_routing_shards 设置为 30 (5 x 2 x 3) 的 5 分片索引可以按 2 或 3 的因子进行拆分。换句话说，可以按如下方式拆分：

- `5` → `10` → `30` (split by 2, then by 3)5 → 10 → 30（除以 2，然后除以 3）
- `5` → `15` → `30` (split by 3, then by 2)5 → 15 → 30（除以 3，然后除以 2）
- `5` → `30` (split by 6)5 → 30（除以 6）



### 动态索引的设置

**index.number_of_replicas** 每个主分片拥有的副本数量。默认为 1。

详见https://www.elastic.co/guide/en/elasticsearch/reference/8.14/index-modules.html



### 文本分析

https://www.elastic.co/guide/en/elasticsearch/reference/8.14/analysis-overview.html#tokenization

*分析器*（无论是内置的还是自定义的）只是一个包含三个较低级别的构建块的包：字符过滤器、分词器和令牌过滤器。



### ES的REST API

https://www.elastic.co/guide/en/elasticsearch/reference/8.14/api-conventions.html



CRUD API文档  https://www.elastic.co/guide/en/elasticsearch/reference/8.14/docs.html



ES的查询过程

在Elasticsearch中，查询通常分为两个阶段：查询阶段和获取阶段。这两个阶段分别对应倒排索引的查找和文档的提取过程。

查询阶段（Query Phase）

在查询阶段，Elasticsearch 使用倒排索引来查找与查询条件匹配的文档。这一阶段的主要步骤如下：

1. **解析查询条件**：Elasticsearch 解析并理解查询条件。
2. **倒排索引查找**：通过倒排索引，Elasticsearch 找到所有包含查询词的文档ID。这一步是非常高效的，因为倒排索引将词语映射到文档ID。
3. **评分和排序**：Elasticsearch 对找到的文档进行评分，并按照相关性排序。此时仅涉及文档ID和评分数据，而不需要读取实际的文档内容。

获取阶段（Fetch Phase）

在获取阶段，Elasticsearch 根据查询阶段得到的文档ID列表，去检索实际的文档内容。主要步骤如下：

1. **分片内获取**：每个相关分片根据文档ID列表提取相应的文档内容。
2. **合并结果**：各个分片的结果合并到一起，生成最终的查询结果。



### ES .net 客户端

注意8.0版本文档还在建设中，可以查看7.0版本的文档，比较全面，但是8.0版本与7.0版本有差异

使用高级客户端NEST，提供强类型查询 DSL，内部仍使用Elasticsearch.Net这个低级客户端



https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/serialization.html

默认使用系统的System.Text.Json库进行序列化，还可以使用自定义的JsonConverter进行序列化



ES字段数据类型

https://www.elastic.co/guide/en/elasticsearch/reference/7.17/mapping-types.html

https://www.cnblogs.com/imcati/p/17862264.html



创建索引时，需要将.net中的实体类型映射到ES中的类型

默认会自动映射对应的类型

https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/auto-map.html

```csharp
public abstract class Document
{
    public JoinField Join { get; set; }
}

public class Company : Document
{
    public string Name { get; set; }
    public List<Employee> Employees { get; set; }
}

public class Employee : Document
{
    public string LastName { get; set; }
    public int Salary { get; set; }
    public DateTime Birthday { get; set; }
    public bool IsManager { get; set; }
    public List<Employee> Employees { get; set; }
    public TimeSpan Hours { get; set; }
}
```

```csharp
var createIndexResponse = _client.Indices.Create("myindex", c => c
    .Map<Document>(m => m
        .AutoMap<Company>() 
        //或.AutoMap(typeof(Employee)) 
    )
);
```

会根据实体类序列化成对应的JSON请求

其中string类型，映射到带有`"keyword"`子字段的`"text"` 。请参阅[多字段](https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/multi-fields.html)。

`Employee`类型是递归的，因为`Employee`类本身包含`Employee`类型的集合。默认情况下， `.AutoMap()`在遇到这样的递归实例时只会遍历单个深度，这样做是为了防止堆栈溢出和无限递归，一定要遍历可以通过传参指定次数



通过属性映射

https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/attribute-mapping.html

在对应实体类上加上NEST提供的特性

.AutoMap会优先使用指定的类型



通过Fluent mapping 显示地指定每个属性的映射

```csharp
var createIndexResponse = _client.Indices.Create("myindex", c => c
    .Map<Company>(m => m
        .Properties(ps => ps
            .Text(s => s
                .Name(n => n.Name)
            )
            .Object<Employee>(o => o
                .Name(n => n.Employees)
                .Properties(eps => eps
                    .Text(s => s
                        .Name(e => e.FirstName)
                    )
                    .Text(s => s
                        .Name(e => e.LastName)
                    )
                    .Number(n => n
                        .Name(e => e.Salary)
                        .Type(NumberType.Integer)
                    )
                )
            )
        )
    )
);
```



通常使用Fluent和AutoMap结合的方式来映射，比如

```c#
var createIndexResponse = _client.Indices.Create("myindex", c => c
    .Map<Company>(m => m
        .AutoMap()
        .Properties(ps => ps
            .Nested<Employee>(n => n
                .Name(nn => nn.Employees)
            )
        )
    )
);
```

重写`Employees`属性以使其成为[嵌套数据类型](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/nested.html)，因为默认情况下`.AutoMap()`将推断`List<Employee>`属性作为`object`数据类型



为什么要使用嵌套数据类型 .Nested

因为ES 对`object` 类型会将嵌套对象扁平化到文档的顶层，这可能导致查询复杂性增加或数据不一致。`nested` 类型保持了嵌套结构，避免了这种问题。可以查询满足特定条件的嵌套对象，而不会受到其他嵌套对象的影响。

防止映射爆炸限制：每个索引默认最多有50个nested类型的字段，每个文档默认最多10000个nested类型的JSON对象



但当有大量数据时不适合使用.Nested，`flattened`类型提供了一种替代方法，其中整个对象被映射为单个字段。给定一个对象， `flattened`映射将解析其叶值并将它们作为关键字索引到一个字段中，对于索引具有大量或未知数量的唯一键的对象非常有用。为整个 JSON 对象仅创建一个字段映射，这有助于防止由于具有太多不同的字段映射而导致[映射爆炸](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/mapping.html#mapping-limit-settings)。



总体优先顺序：Fluent > attribute > automap



Fluent api 还可以映射 运行时字段

运行时字段（runtime fields）是一种动态计算字段的机制，允许你在查询时基于文档的现有数据动态生成新的字段。这些字段不会被存储在索引中，而是在查询时临时计算。运行时字段提供了灵活性和扩展性，使得在不重新索引数据的情况下，可以实现各种复杂的查询和数据处理需求。



忽略属性的映射

特性方式

```c#
[Text(Ignore = true)]
public string PropertyToIgnore { get; set; }

[PropertyName("anotherPropertyToIgnore", Ignore = true)]
public string AnotherPropertyToIgnore { get; set; }

[Ignore, JsonIgnore]
public string JsonIgnoredProperty { get; set; }
```

fluent方式

```c#
var connectionSettings = new ConnectionSettings(new InMemoryConnection()) 
    .DisableDirectStreaming() 
    .DefaultMappingFor<CompanyWithAttributesAndPropertiesToIgnore>(m => m
        .Ignore(p => p.FluentMappingPropertyToIgnore)
    );
```

使用.DefaultMappingFor映射时还可以忽略或者覆盖继承的属性

https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/ignoring-properties.html



通配符查询

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-wildcard-query.html

```c#
var response = await _client.SearchAsync<Domain.Users.User>(s => s.Index(index)
        .Query(q => q.Wildcard(w => w.Field("displayName.keyword").Value($"*{keyword}*").CaseInsensitive(true)))
...
```

通过将字段类型设置为 `.wildcard`，你可以显著提高通配符查询的性能，特别是在处理大数据集或需要频繁进行通配符查询的场景中。



### ES NEST库 索引api

最好不要去索引还没有创建索引的对象，Elasticsearch 会自动创建这个索引并使用动态映射来推断字段类型。动态映射可能会错误地推断数据类型，导致查询和分析问题。而且索引中包含大量不必要的字段，可能会导致映射膨胀，影响性能。



#### 文档索引的批量操作

https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/indexing-documents.html



#### IndexManyAsync方法

对多个文档建立索引，不适合非常大的文档集合

在项目中仅在测试方法使用



#### 使用`Bulk`和`BulkAsync`方法并使用描述符来自定义批量调用

```csharp
var asyncBulkIndexResponse = await client.BulkAsync(b => b
    .Index("people")
    .IndexMany(people)
); 
```

在项目中，通过构造BulkDescriptor对象来传递参数调用，而不是使用Fulent API方式

```c#
if (bulkOperation.Count > 0)
{
    var bulkRequest = new BulkRequest(_elasticsearchXxxItemSetting.Value)//索引名
    {
        Operations = bulkOperation
    };
    var response = await _client.BulkAsync(x => bulkRequest, cancellationToken);
    response.CheckIfTheResponseIsInvalid();                
}
```

构造bulkOperation：

```c#
var insertList = _mapper.Map<IEnumerable<ElasticsearchOrderCommentItem>>(orderCommentItemList)
    .Select(x =>
    {
        x.MultiLanguages =
            _mapper.Map<List<ElasticsearchMerchFoodMultiLanguage>>(
                entityLanguages.Where(e => e.EntityId == x.FoodId));
        return new BulkIndexOperation<ElasticsearchOrderCommentItem>(x); //主要将对象封装成BulkIndexOperation类
    });

bulkOperation.AddRange(insertList);
```

其他BulkOperation类型

![240807a](../../../笔记/md笔记/img/240807a.jpg)



由于使用Bulk，大量文档是在单个 HTTP 请求中发送的。应该考虑 HTTP 请求的总体大小。

所以对于索引大量文档，执行多个单独的`Bulk`调用或使用[`BulkAllObservable`](https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/indexing-documents.html#bulkall-observable)



#### 使用BulkAllObservable和`BulkAll`方法

可以实现索引失败时自动重试/退避的功能，以及控制在单个 HTTP 请求中索引的文档数量的功能。

```c#
var bulkAllObservable = client.BulkAll(people, b => b
    .Index("people")
    .BackOffTime("30s") //设置重试间隔时间为 30 秒
    .BackOffRetries(2) //设置重试次数为 2 次
    .RefreshOnCompleted() //在所有批量操作完成后刷新索引，使其立即可被搜索
    .MaxDegreeOfParallelism(Environment.ProcessorCount)
    .Size(1000) //每个请求将包含 1000 个文档，如果存在大量文档，这可能会导致许多 HTTP 请求
)
.Wait(TimeSpan.FromMinutes(15), next => //执行索引并等待最多 15 分钟，BulkAll 调用是异步的
{
    // do something e.g. write number of pages to console
});
```

BulkAllObservable还有以下方法，可以用来控制流程

- `BufferToBulk`用于在将批量请求分派到服务器之前自定义批量请求中的各个操作
- `RetryDocumentPredicate`用于决定是否应重试索引失败的文档
- `DroppedDocumentCallback`用于确定在文档未编入索引的情况下要执行的操作（即使在重试之后）

详见文档最下方



#### 摄取管道 Ingest Pipelines

https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/pipelines.html

可以设置一个管道，可以在索引时指定使用一个管道，可以在数据进入索引之前执行一系列处理步骤，例如重命名字段、修改字段值、删除字段、解析日期等。

```csharp
var indexResponse = client.Index(person, p => p.Index("people").Pipeline("person-pipeline"));
```



### 配置分析器

https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/writing-analyzers.html

分析器（Analyzer）是用于文本分析的组件，它将输入文本分解为词元（tokens），这些词元可以用于索引和搜索。分析器通常在索引文档和查询时使用，以确保文本的一致性和提高搜索的准确性。

一个分析器由以下几个部分组成：

1. **字符过滤器（Character Filters）**：在分词之前处理文本，执行一些预处理任务，例如删除HTML标签、替换字符等。
2. **分词器（Tokenizer）**：将文本分解为基础词元。分词器决定了词元的边界。（分析器只能也必须有一个）
3. **词元过滤器（Token Filters）**：在分词之后进一步处理词元，例如将词元转换为小写、删除停用词、词干化等。



通常在索引创建时创建类型映射时或者新增索引字段时，可以在text数据类型上指定分析器。但是不能在已有分析器时添加新分析器或者修改一个字段的分析器，否则已有的索引的数据去搜索不会出现分析器预期结果。需要使用[Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docs-reindex.html)重新索引数据，或者删除再重新索引。



可以使用ES内置的分析器，比如

**standard**：默认的标准分析器，适用于大多数语言。使用 Unicode 文本分割算法。

**simple**：简单分析器，只分隔字母字符，并将所有字符转换为小写。

**whitespace**：以空白字符（空格、制表符等）分词，不修改词元。

**stop**：类似于 `standard` 分析器，但会去除常见的停用词。

**keyword**：不分词，将整个输入作为一个词元。

**pattern**：使用正则表达式进行分词。

**language-specific analyzers**：为特定语言定制的分析器，如 `english`、`french`、`german` 等，它们包括语言特定的字符过滤器、分词器和词元过滤器。



也可以使用自定义的分析器，官方示例：

```c#
var createIndexResponse = _client.Indices.Create("questions", c => c
    .Settings(s => s
        .Analysis(a => a
            .CharFilters(cf => cf //设置自定义的字符过滤器
                .Mapping("programming_language", mca => mca //命名为programming_language
                    .Mappings(new []
                    {
                        "c# => csharp", //将C#和c#分别映射到"CSharp"和"csharp" （因此#不会被分词器剥离）
                        "C# => Csharp"
                    })
                )
            )
            .Analyzers(an => an //设置自定义分析器，命名为question
                .Custom("question", ca => ca 
                    .CharFilters("html_strip", "programming_language") //使用自定义的字符过滤器和内置的html过滤器
                    .Tokenizer("standard") //使用内置的标准分词器
                    .Filters("lowercase", "stop") //使用内置的小写和删除停用词的词元过滤器
                )
            )
        )
    )
    .Map<Question>(mm => mm
        .AutoMap()
        .Properties(p => p
            .Text(t => t
                .Name(n => n.Body)
                .Analyzer("question") //对该字段使用自定义分析器
            )
        )
    )
);
```



官方测试分析器的示例

https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/testing-analyzers.html



### ES NEST库 查询api

匹配全部文档 示例

```c#
var searchRequest = new SearchRequest<Project>
{
    Query = new MatchAllQuery()
};

searchResponse = _client.Search<Project>(searchRequest);
```

也可以使用Fluent API用法

```c#
var searchResponse = _client.Search<Project>(s => s
    .Query(q => q
        .MatchAll()
    )
);
```



传入其他参数参见 https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/reference-search.html



查询操作通常分为三类

- [Structured search 结构化搜索](https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/writing-queries.html#structured-search)
- [Unstructured search 非结构化搜索](https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/writing-queries.html#unstructured-search)
- [Combining queries 组合查询](https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/writing-queries.html#combining-queries)

可以针对具体用法进行api的学习和调用



比如 Boolean query 布尔查询 https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-bool-query.html

 

#### Query and filter context 查询和过滤上下文

详见 https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-filter-context.html

默认情况下，Elasticsearch 按**相关性得分**对匹配的搜索结果进行排序，相关性得分衡量每个文档与查询的匹配程度。

相关性分数是一个正浮点数，在搜索API 的`_score`数据字段中返回。 `_score`越高，文档的相关性越高。虽然每种查询类型可以以不同的方式计算相关性分数，但分数计算还取决于查询子句是在**查询上下文**中运行还是在**筛选**上下文中运行。



### 查询只返回文档中的部分字段

##### 通过不存储某些字段实现

默认情况下，在索引文档时，Elasticsearch 会将最初发送的 JSON 文档存储在名为 [_source](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/mapping-source-field.html) 的特殊字段中。从搜索查询返回的文档将从每次命中时从 Elasticsearch 返回`的_source`字段具体化。_source存储了字段的原始值

通过在创建映射时，指定对应字段时添加.Store(false)禁用字段的_source存储，比如

```c#
var response = await _client.Indices.CreateAsync(index, i => i.Map<Xxx>(m => m
        .Properties(p => p
            .Keyword(k=>k.Name(n=>n.Id).Store(false))
            ...
```



##### 使用Source filtering 源过滤

```c#
var searchResponse = _client.Search<Project>(s => s
    .Source(sf => sf
        .Includes(i => i //包含
            .Fields(
                f => f.Name,
                f => f.StartedOn,
                f => f.Branches
            )
        )
        .Excludes(e => e //排除
            .Fields("num*") //支持通配符模式
        )
    )
    .Query(q => q
        .MatchAll()
    )
);
```



#### NEST还支持发送滚动请求用于返回大量的文档

https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/scrolling-documents.html



### ES中的聚合查询

用法参见 https://www.elastic.co/guide/en/elasticsearch/client/net-api/7.17/reference-aggregations.html

Elasticsearch 中的聚合（Aggregations）功能是一种强大的数据处理工具，用于分析和总结大量数据。通过聚合，可以从大规模数据集中提取统计信息、趋势、分布等有用信息，而不仅仅是简单的搜索和检索。

Elasticsearch 提供了多种类型的聚合，每种聚合适用于不同的分析需求。常见的聚合类型包括：

1. **度量聚合（Metric Aggregations）**：计算单个值，如平均值、总和等。
   - `avg`：计算平均值。
   - `sum`：计算总和。
   - `max`：计算最大值。
   - `min`：计算最小值。
   - `cardinality`：计算唯一值的数量。
2. **桶聚合（Bucket Aggregations）**：将文档分组到桶中，每个桶代表一个数据子集。
   - `terms`：按特定字段的值进行分组。
   - `histogram`：按数值范围进行分组。
   - `date_histogram`：按时间范围进行分组。
   - `range`：按特定范围进行分组。
3. **嵌套聚合（Nested Aggregations）**：支持在一个聚合中嵌套另一个聚合，实现多级聚合。
4. **管道聚合（Pipeline Aggregations）**：基于其他聚合的结果进行进一步计算。
   - `derivative`：计算派生值。
   - `moving_avg`：计算移动平均值。



项目中的应用可以参考IElasticsearchMerchFoodService.cs中GetEvenlyDistributedMerchantFoodIdsByName方法

使用了桶聚合和terms聚合进行分组查询，并使用size限制返回桶的数量，最外层查询使用size(0)表示不关心返回的文档而是聚合的结果

并从聚合结果中提取需要的数据