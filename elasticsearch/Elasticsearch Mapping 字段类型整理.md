Elasticsearch Mapping 常见字段类型整理
---

> 官方文档：[字段数据类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html#mapping-types)

## 一、核心数据类型

- 字符串：`text`, `keyword`
- 数值型：`long`, `integer`, `short`, `byte`, `double`, `float`, `half_float`, `scaled_float`
- 布尔型：`boolean`
- 日期型：`date`, `date_nanos`
- 二进制：`binary`
- 范围型：`integer_range`, `float_range`, `long_range`, `double_range`, `date_range`

### 1. 字符串

#### [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)

text 类型的字段数据会被分词，在生成倒排索引以前，字符串会被分词器分成一个一个词项。
text 类型的字段不用于排序，很少用于聚合(termsAggregation除外)。
如果一个字段需要被全文搜索或模糊匹配，比如文章内容、产品描述、新闻内容等，应该使用text类型。

#### [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)

keyword 类型的字段内容不会被分词。
keyword 类型的字段只能通过精确值搜索到，用于过滤、排序、聚合。
适用于索引结构化的字段，比如IP地址、性别和地区等。

### 2. [数值型](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)

#### 整数

|类型|最小值|最大值|说明|
|-|-|-|-|
|byte|-128|127|8 位有符号整数（1个字节），相当于MySQL中有符号的 tinyint|
|short|-32768|32767|16 位有符号整数（2个字节），相当于MySQL中有符号的 smallint|
|integer|-2147483648<br>(-2^31^)|2147483647<br>(2^31^-1)|32 位有符号整数（4个字节），相当于MySQL中有符号的 int|
|long|-9223372036854775808)<br>(-2^63^) |9223372036854775807<br>(2^63^-1)|64 位有符号整数（8个字节），相当于MySQL中有符号的 bigint|

对于整数类型的字段，在满足需求的情况下，要尽可能选择范围小的数据类型。比如某个字段的取值最大值不会超过100，那么选择byte类型即可。迄今为止,吉尼斯世界记录的人类的年龄的最大值为134岁，对于年龄字段，short足矣。字段的长度越短，索引和搜索的效率越高。

#### 小数

|类型|最小值|最大值|说明|
|-|-|-|-|
|half_float|2^-24^ |65504 |16位半精度浮点数|
|float|2^-149^ |(2-2^-23^)·2^127^ |32位单精度浮点数|
|double|2^-1074^ |(2-2^-52^)·2^1023^ |64位双精度浮点数|
|scaled_float|||缩放类型浮点数

处理浮点数时，优先考虑使用scaled float类型。scaled float 是通过缩放因子把浮点数变成long类型，比如价格只需要精确到分，price字段的取值为57.34，设置放大因子为100，存储起来就是5734，所有的API都会把price的取值当作浮点数，事实上Elasticsearch底层存储的是整数类型，因为压缩整数比压缩浮点数更加节省存储空间。

### 3. [布尔型](https://www.elastic.co/guide/en/elasticsearch/reference/current/boolean.html)

如果一个字段是布尔类型，可接受的值为 `true`, `false`。
Elasticsearch 5.4版本以前，可以接受可被解释为 `true` 或 `false` 的字符串和数字。
5.4版本以后只接受 `true`, `false`, `"true"`, `"false"`。

### 4. 日期型

#### [date](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html)

JSON 没有日期型数据类型，所以在Elasticsearch中，日期可以是：
* 包含格式化日期的字符串，例如"2015-01-01"或者"2015/01/01 12:10:30"
* 代表时间毫秒数的长整型数字。
* 代表时间秒数的整数。

Elasticsearch内部会把日期转换为 UTC （世界标准时间），并将其存储为代表时间毫秒数的长整数。
日期格式可以自定义，如果没有指定格式，则使用默认值：
	
	"strict_date_optional_time||epoch_millis"

这种情况下可以解析下面三种日期格式：

	"2020-05-01"
	"2020-05-01T12:10:30Z"
	1591234567890

#### [date_nanos](https://www.elastic.co/guide/en/elasticsearch/reference/current/date_nanos.html)

此数据类型是对日期数据类型的补充。现有的 date 类型可以存储毫秒级时间。而 date_nanos 可以存储纳秒级时间。

### 5. 二进制

#### [binary](https://www.elastic.co/guide/en/elasticsearch/reference/current/binary.html)

二进制数据类型接受Base64编码字符串的二进制值。字段不以默认方式存储而且不能搜索。
Base64编码二进制值不能嵌入换行符`\n`

### 6. [范围型](https://www.elastic.co/guide/en/elasticsearch/reference/current/range.html)

类型| 说明
-|-
`integer_range`|   32 位有符号整数的范围值，-2^31^ ~ 2^31^-1
`long_range`|  62 位有符号整数的范围值，-2^63^ ~ 2^63^-1
`float_range`|    32位单精度浮点数范围值
`double_range`|   64位单精度浮点数范围值
`date_range`|    以64位无符号整数形式表示的日期值范围
`ip_range` |  [IPv4](https://en.wikipedia.org/wiki/IPv4) 或 [IPv6](https://en.wikipedia.org/wiki/IPv6) 的范围值


## 二、复合数据类型

### 1. 对象类型

#### [object](https://www.elastic.co/guide/en/elasticsearch/reference/current/object.html)

用于存储单个JSON对象。
JSON本质上具有层级关系，文档包含内部对象，内部对象本身还可以包含内部对象。

### 2. 嵌套类型

#### [nested](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)

用于存储多个JSON对象组成的数组。
`nested` 类型是 `object` 类型中的一个特例，可以让对象数组独立索引和查询。Lucene没有内部对象的概念，所以Elasticsearch将对象层次扁平化，转化成字段名字和值构成的简单列表。

## 三、地理位置类型

### 1. 地理坐标类型

#### [geo_point](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html)

用于存储经纬度坐标对，可用来
查找一定范围内的地理点,这个范围可以是相对于一个中心点的固定距离,也可以是多边形或者地理散列单元。
通过地理位置或者相对于中心点的距离聚合文档。
整合距离到文档的相关性评分中。

用于存储地理位置信息的经纬度坐标对，可用于以下几种场景：
* 查找一定范围内的地理位置。
* 通过地理位置或者相对中心点的距离来聚合文档。
* 把距离因素整合到文档的评分中。
* 通过距离对文档排序。

### 2. 地理形状类型

#### [geo_shape](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html)

地理形状数据类型有利于索引和搜索任意地理形状，例如矩形、三角形或者其他多边形。无论是数据被索引还是在查询执行的过程中，都可以使用地理形状数据类型在地理点的基础上包含地理形状。
Elasticsearch 使用 `GeoJSON` 格式来表示地理形状。
`GeoJSON` 是一种对各种地理数据结构进行编码的格式，对象可以表示几何、特征或者特征集合,支持点、线、面、多点、多线、多面等几何类型。
`GeoJSON` 里的特征包含一个几何对象和其他属性，特征集合表示一系列特征。
想了解更多关于 `GeoJSON` 的资料可参考[《GeoJSON格式规范说明》](http://www.oschina.netranslate/geojson-spec)

## 四、特殊类型

#### [IP](https://www.elastic.co/guide/en/elasticsearch/reference/current/ip.html)
	
IP地址类型，存储 IPv4 和 IPv6 地址

#### [Completion datatype](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester)

`completion` 提供自动补全建议

#### [Token count](https://www.elastic.co/guide/en/elasticsearch/reference/current/token-count.html)

`token_count` 用于统计字符串分词后的词项个数，本质上是一个整数型字段。
例如：映射中指定 name 为 text 类型，增加 name_length  字段用于统计分词后词项的长度，类型为 token_count，分词器为标准分词器。

#### [`mapper-murmur3`](https://www.elastic.co/guide/en/elasticsearch/plugins/7.6/mapper-murmur3.html)

`murmur3` 在索引时计算值的哈希值并将它们存储在索引中

#### [`mapper-annotated-text`](https://www.elastic.co/guide/en/elasticsearch/plugins/7.6/mapper-annotated-text.html)

`annotated-text` 索引包含特殊标记的文本（通常用于标识命名实体）

#### [Percolator](https://www.elastic.co/guide/en/elasticsearch/reference/current/percolator.html)
	
接受来自 `query-dsl` 的查询

#### [Join](https://www.elastic.co/guide/en/elasticsearch/reference/current/parent-join.html)
	
为同一索引中的文档定义父/子关系

#### [Rank feature](https://www.elastic.co/guide/en/elasticsearch/reference/current/rank-feature.html)
#### [Rank features](https://www.elastic.co/guide/en/elasticsearch/reference/current/rank-features.html)

排名功能，记录数字特性以提高查询时的命中率

#### [Dense vector](https://www.elastic.co/guide/en/elasticsearch/reference/current/dense-vector.html)
	
密集向量，记录浮点值的密集向量

#### [Sparse vector](https://www.elastic.co/guide/en/elasticsearch/reference/current/sparse-vector.html)  

稀疏向量，记录浮点值的稀疏向量

#### [Search-as-you-type](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html)

按类型搜索，类似文本的字段，为查询进行优化，以实现按类型完成

#### [Alias](https://www.elastic.co/guide/en/elasticsearch/reference/current/alias.html) 
	
别名，定义现有字段的别名

#### [Flattened](https://www.elastic.co/guide/en/elasticsearch/reference/current/flattened.html) 

允许将整个JSON对象作为单个字段编入索引。

#### [Shape](https://www.elastic.co/guide/en/elasticsearch/reference/current/shape.html) 

`shape` for arbitrary cartesian geometries.

#### [Histogram](https://www.elastic.co/guide/en/elasticsearch/reference/current/histogram.html) 

`histogram` for pre-aggregated numerical values for percentiles aggregations.


## 五、数组类型
数组类型不需要专门指定数组元素的类型，任何字段类型都可以包含在数组内，但是数组中的所有值必须具有相同的数据类型。

- 字符型数组: `["one", "two"]`
- 整型数组：`[1, 2]`
- 数组型数组：`[1, [2, 3]]` 等同于 `[1, 2, 3]`
- 对象数组：`[{"name": "Mary", "age": 12}, {"name": "John", "age": 10}]`

