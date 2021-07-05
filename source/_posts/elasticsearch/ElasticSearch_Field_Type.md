---
title: Elasticsearch字段类型
date: 2021-05-24 22:25:50
index_img: /img/cover/Elasticsearch.jpg
cover: /img/cover/Elasticsearch.jpg
tags: 
- 字段类型
categories:
- Elasticsearch
---

#### ElasticSearch 字段类型

#### 普通类型

* `binary`: Binary value encoded as a Base64 string.

  > The `binary` type accepts a binary value as a [Base64](https://en.wikipedia.org/wiki/Base64) encoded string. The field is not stored by default and is not searchable:
  >
  > BINARY类型接受二进制值作为Base64编码字符串。**默认情况下，该字段不存储且不可搜索**
  >
  > The following parameters are accepted by `binary` fields:
  >
  > | 参数名称     | 说明                                                         |
  > | ------------ | ------------------------------------------------------------ |
  > | `doc_values` | 字段是否应该以跨列方式存储在磁盘上，以便以后可以用于排序、聚合或编写脚本？<br/>接受`true`或`false` |
  > | `store`      | 字段值是否应与`_source`字段分开存储和检索。<br/>接受`true`或`false` |

  ```
  PUT my-index-000001
  {
    "mappings": {
      "properties": {
        "name": {
          "type": "text"
        },
        "blob": {
          "type": "binary",
          "doc_values": false,
          "store": false
        }
      }
    }
  }
  
  PUT my-index-000001/_doc/1
  {
    "name": "Some binary blob",
    "blob": "U29tZSBiaW5hcnkgYmxvYg==" // Base64编码的二进制值不能嵌入换行符\n
  }
  ```

* `boolean`: `true` and `false` values.

  > Boolean fields accept JSON `true` and `false` values, but can also accept strings which are interpreted as either true or false:
  >
  > | 参数         | 说明                                    |
  > | ------------ | --------------------------------------- |
  > | False values | `false`, `"false"`, `""` (empty string) |
  > | True values  | `true`, `"true"`                        |
  >
  > The following parameters are accepted by `boolean` fields:
  >
  > | 参数                                                         | 说明                                                         |
  > | ------------------------------------------------------------ | ------------------------------------------------------------ |
  > | [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-boost.html) | Mapping field-level query time boosting. Accepts a floating point number, defaults to `1.0`. |
  > | [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html) | Should the field be stored on disk in a column-stride fashion, so that it can later be used for sorting, aggregations, or scripting? Accepts `true` (default) or `false`. |
  > | [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-index.html) | Should the field be searchable? Accepts `true` (default) and `false`. |
  > | [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/current/null-value.html) | Accepts any of the true or false values listed above. The value is substituted for any explicit `null` values. Defaults to `null`, which means the field is treated as missing. |
  > | [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html) | Whether the field value should be stored and retrievable separately from the [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html) field. Accepts `true` or `false` (default). |
  > | [`meta`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-field-meta.html) | Metadata about the field.                                    |
  >
  > 

* `Keywords`: The keyword family, including `keyword`, `constant_keyword`, and `wildcard`.

* `Numbers`:  Numeric types, such as long and double, used to express amounts

#### 别名

* `alias`: Defines an alias for an existing field.

  > An `alias` mapping defines an alternate name for a field in the index. The alias can be used in place of the target field in [search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html) requests, and selected other APIs like [field capabilities](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-field-caps.html).

  ```json
  PUT trips
  {
    "mappings": {
      "properties": {
        "distance": {
          "type": "long"
        },
        // 设置route_length_miles为distance的别名
        "route_length_miles": {
          "type": "alias",
          "path": "distance" // The path to the target field. Note that this must be the full path, including any parent objects (e.g. object1.object2.field). 目标字段必须是完整路径
        },
        "transit_mode": {
          "type": "keyword"
        }
      }
    }
  }
  
  GET _search
  {
    "query": {
      "range" : {
        "route_length_miles" : {
          "gte" : 39
        }
      }
    }
  }
  ```

* 别名的作用

  > Almost all components of the search request accept field aliases. In particular, aliases can be used in queries, aggregations, and sort fields, as well as when requesting docvalue_fields, stored_fields, suggestions, and highlights. Scripts also support aliases when accessing field values. 
  >
  > 搜索请求的几乎所有组件都接受字段别名。具体地说，**别名可以在查询、聚合和排序字段中使用**，也可以在请求docvalue_field、stored_field、suggestions和Highlight时使用。脚本还支持在访问字段值时使用别名。

* 别名的限制

  > There are a few restrictions on the target of an alias:
  >
  > - The target must be a concrete field, and not an object or another field alias. 目标必须是具体字段，而不是对象或其他字段别名。
  > - The target field must exist at the time the alias is created. 创建别名时，目标字段必须存在。
  > - If nested objects are defined, a field alias must have the same nested scope as its target. 如果定义了嵌套对象，字段别名必须与其目标具有相同的嵌套作用域。
  >
  > Additionally, a field alias can only have one target. This means that it is not possible to use a field alias to query over multiple target fields in a single clause.
  >
  > An alias can be changed to refer to a new target through a mappings update. A known limitation is that if any stored percolator queries contain the field alias, they will still refer to its original target.
  >
  > 别名的目标有几个限制：
  >
  > * **目标必须是具体字段，而不是对象或其他字段别名。**
  > * **创建别名时，目标字段必须存在。**
  > * **如果定义了嵌套对象，字段别名必须与其目标具有相同的嵌套作用域。**
  >
  > 此外，**一个字段别名只能有一个目标**。这意味着不能在单个子句中使用字段别名查询多个目标字段。
  >
  > 可以通过映射更新将别名更改为引用新目标。一个已知的限制是，如果任何存储的过滤器查询包含字段别名，它们仍将引用其原始目标。
  >
  > Writes to field aliases are not supported: attempting to use an alias in an index or update request will result in a failure. Likewise, aliases cannot be used as the target of `copy_to` or in multi-fields.Because alias names are not present in the document source, aliases cannot be used when performing source filtering. 
  >
  > 不支持写入字段别名：尝试在索引或更新请求中使用别名将导致失败。同样，别名不能作为`copy_to`的目标，也不能多字段使用，由于别名不存在于文档源中，因此在执行源筛选时不能使用别名。
  >
  > ```json
  > 错误示例
  > GET /_search
  > {
  >   "query" : {
  >     "match_all": {}
  >   },
  >   "_source": "route_length_miles"
  > }
  > ```

#### 日期

* Dates: Date types, including date and date_nanos.

  >  JSON doesn’t have a date data type, so dates in Elasticsearch can either be:
  >
  > - strings containing formatted dates, e.g. `"2015-01-01"` or `"2015/01/01 12:10:30"`. 日期格式的字符串
  > - a number representing *milliseconds-since-the-epoch*.  毫秒级的时间戳
  >   - Values for milliseconds-since-the-epoch must be non-negative. Use a formatted date to represent dates before 1970. 从纪元开始的毫秒数的值必须为非负数。使用格式化日期表示1970年前的日期。
  > - a number representing *seconds-since-the-epoch* 秒级的时间戳
  >
  > Date formats can be customised, but if no `format` is specified then it uses the default:
  >
  > ```js
  > "strict_date_optional_time||epoch_millis"
  > PUT my-index-000001
  > {
  >   "mappings": {
  >     "properties": {
  >       "date": {
  >         "type":   "date",
  >         "format": "strict_date_optional_time||epoch_second"
  >       }
  >     }
  >   }
  > }
  > ```
  >
  > The following parameters are accepted by `date` fields:
  >
  > | [`boost`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-boost.html) | Mapping field-level query time boosting. Accepts a floating point number, defaults to `1.0`. |
  > | ------------------------------------------------------------ | ------------------------------------------------------------ |
  > | [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/current/doc-values.html) | Should the field be stored on disk in a column-stride fashion, so that it can later be used for sorting, aggregations, or scripting? Accepts `true` (default) or `false`. |
  > | [`format`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html) | The date format(s) that can be parsed. Defaults to `strict_date_optional_time||epoch_millis`. |
  > | `locale`                                                     | The locale to use when parsing dates since months do not have the same names and/or abbreviations in all languages. The default is the [`ROOT` locale](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#ROOT), |
  > | [`ignore_malformed`](https://www.elastic.co/guide/en/elasticsearch/reference/current/ignore-malformed.html) | If `true`, malformed numbers are ignored. If `false` (default), malformed numbers throw an exception and reject the whole document. |
  > | [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-index.html) | Should the field be searchable? Accepts `true` (default) and `false`. |
  > | [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/current/null-value.html) | Accepts a date value in one of the configured `format`'s as the field which is substituted for any explicit `null` values. Defaults to `null`, which means the field is treated as missing. |
  > | [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-store.html) | Whether the field value should be stored and retrievable separately from the [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html) field. Accepts `true` or `false` (default). |
  > | [`meta`](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-field-meta.html) | Metadata about the field.                                    |

#### 对象类型

* `object`: A JSON object.
* `flattened`: An entire JSON object as a single field value.将整个JSON对象作为单个字段值。
* `nested`: A JSON object that preserves the relationship between its subfields. 保留其子字段之间关系的JSON对象。
* `join`: Defines a parent/child relationship for documents in the same index. 为同一索引中的文档定义父/子关系。

#### 特定类型

* `Range`: Range types, such as `long_range`, `double_range`, `date_range`, and `ip_range`.
* `ip`: IPv4 and IPv6 addresses.
* `version`: Software versions. 
* `murmur3`: Compute and stores hashes of values. 计算并存储值的散列。

#### 聚合数据类型

* `aggregate_metric_double`: Pre-aggregated metric values.
* `histogram`: Pre-aggregated numerical values in the form of a histogram. 以直方图形式预先聚合的数值。

#### 文本数据类型

* `text`: Analyzed, unstructured text.
* `annotated-text`: Text containing special markup. Used for identifying named entities.
* `completion`: Used for auto-complete suggestions.
* `search_as_you_type`: `text`-like type for as-you-type completion.
* `token_count`: A count of tokens in a text.

#### 文档排名类型

* `dense_vector`: Records dense vectors of float values.

* `sparse_vector`: Records sparse vectors of float values.

* `rank_feature`: Records a numeric feature to boost hits at query time.

* `rank_features`: Records numeric features to boost hits at query time.

#### 空间数据类型

* `geo_point`: Latitude and longitude points.

* `geo_shape`: Complex shapes, such as polygons.

* `point`: Arbitrary cartesian points.

* `shape`: Arbitrary cartesian geometries.

#### 数组

* Arrays

> In Elasticsearch, there is no dedicated `array` data type. Any field can contain zero or more values by default, however, all values in the array must be of the same data type.
>
> 在Elasticsearch中，没有专用的`array`数据类型。默认情况下，任何字段都可以包含零个或多个值，但是数组中的所有值必须属于相同的数据类型。
>
> When adding a field dynamically, the first value in the array determines the field `type`. All subsequent values must be of the same data type or it must at least be possible to [coerce](https://www.elastic.co/guide/en/elasticsearch/reference/current/coerce.html) subsequent values to the same data type.
>
> Arrays with a mixture of data types are *not* supported: [ `10`, `"some string"` ]
>
> An array may contain `null` values, which are either replaced by the configured [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/current/null-value.html) or skipped entirely. An empty array `[]` is treated as a missing field — a field with no values.
>
> 动态添加字段时，数组中的第一个值确定字段类型。所有后续值必须具有相同的数据类型，或者必须至少可以将后续值强制为相同的数据类型。
> 不支持混合数据类型的数组：[10，“某些字符串”]。
> 数组可以包含空值，可以由配置的NULL_VALUE替换，也可以完全跳过。空数组[]被视为缺少的字段 - 没有值的字段。



