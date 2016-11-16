# Solr 基础

因为 Solr 包装并扩展了 Lucene，所以他们两者使用很多相同的术语。更重要的是，Solr 创建的索引与 Lucene 搜索引擎库完全兼容。通过对 Solr 进行适当的配置，某些情况下可能需要进行编码，Solr 可以阅读好使用构建到其他 Lucene 应用程序中的索引。在 Solr 和 Lucene 中，使用一个或多个 Document 来构建索引。Document 包括一个或多个 Field。Field 包括名称、内容以及告知 Solr 如何处理内容的元数据。

例如，Field 可以包含字符串、数字、布尔型或者日期，也可以包含你想添加的任何类型，只需用在 Solr 的配置文件中进行相应的配置即可。Field 可以使用大量的选项来描述，这些选项告知 Solr 在索引和搜索期间如何处理内容。

下表列出了两个重要属性的子集：

|属性名称|描述|
|:---|:---|
|Indexed|Indexed Field 可以进行搜索和排序。还可以在 Indexed Field 上运行 Solr 分析过程，此过程可修改内容以改进或更改结果。|
|Stored|Stored Field 内容保存在索引中。这对于检索和高亮显示内容很有帮助，但对于实际搜索则不是必需的。例如，很多应用程序存储指向内容位置的指针而不是存储实际的文件内容。|

## 模式配置 managed-schema

managed-schema 配置文件在创建`core`的`/usr/local/Cellar/solr/6.1.0/server/solr/test/conf`目录下，`test`为创建的`core`名称。它是 Solr 模式关联的文件。打开这个配置文件，可以发现有详细的注释。模式组织主要分为以下 3 个重要配置：

* fieldType
* field
* 其他配置
    * uniqueKey
    * defaultSearchField
    * solrQueryParser
    
### fieldType

fieldType 是一些常用的可重用定义，定义了 Solr（和 Lucence）如何处理 Field。也就是添加到索引中的 xml 文件属性中的类型，如 int、string、date 等。
```
<fieldType name="int" class="solr.TrieIntField" docValues="true" precisionStep="0" positionIncrementGap="0"/>
<fieldType name="string" class="solr.StrField" sortMissingLast="true" docValues="true"/>
<fieldType name="date" class="solr.TrieDateField" docValues="true" precisionStep="0" positionIncrementGap="0"/>
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100" multiValued="true">
<analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <!-- in this example, we will only use synonyms at query time
    <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
    -->
    <filter class="solr.LowerCaseFilterFactory"/>
</analyzer>
<analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
    <filter class="solr.LowerCaseFilterFactory"/>
</analyzer>
</fieldType>
```

参数说明：

|属性|描述|
|:---|:---|
|name|标识|
|class|和其他属性决定了这个 fieldType 的实际行为|
|sortMissingLast|设置成 true 没有该 filed 的数据排在有该 field 的数据之后，而不管请求时的排序规则，默认是设置成 false|
|sortMissingFirst|与之相反，默认是设置为 false|
|analyzer|字段类型指定的分词器|
|type|当前分词用于的操作。index 代表生成索引时使用的分词器 query 代码在查询时使用的分词器|
|tokenizer|分词器类|
|filter|分词后应用的过滤器。过滤器调用顺序与配置相同|

### field

