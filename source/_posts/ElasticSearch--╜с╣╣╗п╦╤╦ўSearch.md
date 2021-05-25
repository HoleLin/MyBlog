---
title: Elasticsearch--结构化搜索Search
date: 2021-05-24 22:25:50
tags: elasticsearch
categories: Elasticsearch
---

## ElasticSearch

#### 参考文献

* [白日梦的Elasticsearch笔记---进阶篇、32种查询方法、15中聚合方式、7种优化后的查询技巧](https://mp.weixin.qq.com/s/m7PLNAJdN9dAzq_Nsv6Gqw)

#### 目录

* <a href="#搜索基本信息">搜索基本信息</a>
  * <a href="#使用URL限制搜索范围">使用URL限制搜索范围</a>
  * <a href="#搜索请求的基本模块">搜索请求的基本模块</a>
  * <a href="#基于URL的搜索请求">基于URL的搜索请求</a>
  * <a href="#基于请求主体的搜索请求">基于请求主体的搜索请求</a>
  
* <a href="#搜索用途分类">搜索用途</a>

  * 精确值检索

    * <a href="#term">term</a>
    * <a href="#boolfilter">bool fliter</a>
    * <a href="#terms">terms</a>
    * <a href="#terms_set">terms_set </a>
  * 范围检索

    * <a href="#range">range</a>
  * 存在与否检索
    * <a href="#exists">exists</a>
  * 前缀检索
    * <a href="#prefix">prefix</a>
  * 通配符检索
    * <a href="#wildcard">wildcard </a>
  * 正则表达式检索
    * <a href="#regexp">regexp</a>
  * 模糊检索
    * <a href="#fuzzy">fuzzy</a>
  * ID检索
    * <a href="#ids">ids</a>
  * 全文检索
    * <a href="#match">分词全文检索 match</a>
    * <a href="#match_phrase">短语检索match_phrase</a>
    * <a href="#intervals">间隔检索intervals</a>
    * <a href="#match_phrase_prefix">短语前缀检索: match_phrase_prefix</a>
    * <a href="#multi_match">多字段匹配检索: `multi_match`</a>
    * <a href="#query_string">query_string</a>
    * <a href="#simple_query_string">simple_query_string</a>
    * <a href="#match_boolean_prefix">match_boolean_prefix</a>
  * 复合检索
    * <a href="#constant_score">固定得分检索 constant_score</a>
    * 改变评分检索
      * <a href="#dis_max">dis_max</a>
      * <a href="#function_score">function_score </a>
      * <a href="#boosting">boosting</a>
  
  * <a href="#match_all">match_all</a>

### <a id="搜索基本信息">搜索基本信息</a>

#### <a id="搜索请求的基本模块">搜索请求的基本模块</a>

> **query** : 搜索请求最重要的组成部分.
>
> **size** : 代表了返回文档的数量
>
> **from** : 和size一起使用,fom用于分页操作. from的参数是从0开始的
>
> **_source** : 指定`_source`字段如何返回.默认是返回完整的`_source`字段.通过配置`_source`,将过滤返回的字段
>
> **sort** : 默认的排序是基于文档的得分

#### <a id="基于URL的搜索请求">基于URL的搜索请求</a>

```shell
curl 'localhost:9200/get-together/_search?from=10&size=10&sort=date:asc&_source=title,date&q=elasticsearch' 
```

#### <a id="基于请求主体的搜索请求">基于请求主体的搜索请求</a>

```shell
curl 'localhost:9200/get-together/_search' -d 
'{
	"query": {
		"match_all":{}
	},
	// 返回从第10页开始的结果
	"from": 10,
	// 总共返回最多10个结果
	"size": 10
	// 搜索回复中返回名字和日期字段
	"_sourcce": ["name","date"]
}'
```

* ##### `_source` 返回字段中通配符

  ```shell
  curl 'localhost:9200/get-together/_search' -d
  '{
  	"query": {
  		"match_all":{}
  	},
  	"_source": {
  		// 在搜索回复中返回以location开头的字段和日期字段
  		"include": ["location.*","deate"],
  		// 不要返回location.getolocation
  		"exclude": ["location.geolocation"]
  	}
  }'
  ```

* ##### 结果的排序

  ```shell
  curl 'localhost:9200/get-together/_search' -d 
  '{
  	"query": {
  		"match_all": {}
  	},
  	"sort": [
  		// 首先按照创建日期来排序 从最老到最新的
  		{"created_on": "asc"},
  		// 然后按照分组的名称来排序 按倒排的字母顺序
  		{"name": "desc"},
  		// 最终按照相关性得分(_score)来排序
  		"_score"
  	]
  }'
  ```

* ##### 返回的结果

  ```json
  {
      //查询所用的毫秒数
      "took": 27,
      // 表明是否分片超时,也就是说是否只返回了部分结果
      "timed_out": false,
      "_shards": {
          // 成功响应该请求和未能成功响应该请求的分片数量
          "total": 15,
          "successful": 15,
          "skipped": 0,
          "failed": 0
      },
      // 返回中的包含了命中(hits)的键,其值是命中文档的数组
      "hits": {
          "total": {
              // 该请求所有匹配结果的数量
              "value": 3,
              "relation": "eq"
          },
          // 这个搜索结果中的最大得分
          "max_score": 2.0000367,
          // 命中(hits)关键词元素中的命中文档数组
          "hits": [
              {
                  // 结果文档的索引
                  "_index": "xxxxx",
                  // 结果文档的Elasticsearch类型
                  "_type": "_doc",
                  // 结果文档的ID
                  "_id": "vZATIXYB5sSlphETLNfC",
                  // 结果的相关性得分
                  "_score": 2.0000367,
                  "_source": {}
              }
          ]
      }
  }
  ```

### <a id="ES检索分类">ES检索分类</a>

* 结构化检索

  * 精确值检索

    * <a id="term">单个精确值检索:  `term query`</a>

      ```json
      {
        "query": {
          "term": {
            "FIELD": {
              "value": "VALUE",
              "boost": 2
            }
          }
        }
      }
      示例: 
      GET /kibana_sample_data_logs/_search
      {
        "query": {
          "term": {
            "_id": {
              "value": "4fMq6HMB4DKMP4_06Fl3"
            }
          }
        }
      }
      JavaAPI:
      searchSourceBuilder.query(QueryBuilders.termQuery(“text.keyword”, “xxx”));
      ```

    * <a id="boolfilter">布尔过滤器: `bool filter`</a>

      ```json
      {
         "bool" : {
            "must" :     [],
            "should" :   [],
            "must_not" : [],
            "filter":    []
         }
      }
      must ——所有的语句都 必须（must） 匹配，与 AND 等价。
      must_not ——所有的语句都 不能（must not） 匹配，与 NOT 等价。
      should ——至少有一个语句要匹配，与 OR 等价。
      filter——必须匹配，运行在非评分&过滤模式。
      GET /kibana_sample_data_logs/_search
      {
        "query": {
          "bool": {
            "must": [
              {
                "match": {
                  "url": "https://www.elastic.co/downloads/elasticsearch"
                }
              }
            ],
            "should": [
              {
                "term": {
                  "host": {
                    "value": "artifacts.elastic.co"
                  }
                }
              }
            ],
            "must_not": [
              {
                "term": {
                  "clientip": {
                    "value": "3.44.54.93"
                  }
                }
              }
            ],
            "filter": [
              {
                "match": {
                  "url": "https://www.elastic.co/downloads/elasticsearch"
                }
              }
            ],
            // 最小的should子句匹配数,满足这个数量文档才能作为结果返回
            "minimum_should_match": 1
          }
        }
      }
      ```

    * <a id="terms">多个精确值检索: `terms query` </a>

      ```json
      GET /kibana_sample_data_logs/_search
      {
        "query": {
          "terms": {
            "FIELD": [
              "VALUE1",
              "VALUE2"
            ],
            "boost":2
          }
        }
      }
      ```
      
    * <a id="terms_set">Terms set query </a>

      > Returns documents that contain a minimum number of **exact** terms in a provided field.
      >
      > The `terms_set` query is the same as the [`terms` query](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-terms-query.html), except you can define the number of matching terms required to return a document. For example:
      >
      > - A field, `programming_languages`, contains a list of known programming languages, such as `c++`, `java`, or `php` for job candidates. You can use the `terms_set` query to return documents that match at least two of these languages.
      > - A field, `permissions`, contains a list of possible user permissions for an application. You can use the `terms_set` query to return documents that match a subset of these permissions.

      ```json
      PUT /job-candidates
      {
        "mappings": {
          "properties": {
            "name": {
              "type": "keyword"
            },
            "programming_languages": {
              "type": "keyword"
            },
            "required_matches": {
              "type": "long"
            }
          }
        }
      }
      
      PUT /job-candidates/_doc/1?refresh
      {
        "name": "Jane Smith",
        "programming_languages": [ "c++", "java" ],
        "required_matches": 2
      }
      
      PUT /job-candidates/_doc/2?refresh
      {
        "name": "Jason Response",
        "programming_languages": [ "java", "php" ],
        "required_matches": 2
      }
      
      GET /job-candidates/_search
      {
        "query": {
          "terms_set": {
            "programming_languages": {
              "terms": [ "c++", "java", "php" ],
              // (可选，字符串)包含返回文档所需的匹配词数量的数值字段。
              "minimum_should_match_field": "required_matches"
            }
          }
        }
      }
      
      GET /job-candidates/_search
      {
        "query": {
          "terms_set": {
            "programming_languages": {
              "terms": [
                "c++",
                "java",
                "php"
              ],
              // (可选，字符串)包含返回文档所需的匹配词数量的自定义脚本。
              "minimum_should_match_script": {
                "source": "Math.min(params.num_terms, doc['required_matches'].value)"
              },
              "boost": 1
            }
          }
        }
      }
      ```

  * <a id="range">范围检索: `range query`</a>

    ```json
    // range查询
    // gte 大于等于(Greater-than or equal to)
    // gt  大于 (Greater-than)
    // lte 小于等于(Less-than or equal to)
    // lt  小于(Less-than)
    // 可作用与支持date,numeric,字符串字段
    curl 'localhost:9200/get-together/_search' -d 
    '{
        "query": {
            "range": {
                "updateTime": {
                    "gt": "1591321611000",
                    "lt": "1607132852000"
                }
            }
        }
    }'
    // 查找birthday大于等于2020-08-17 11:20:00 小于等于2020-08-18的文档记录
    curl 'localhost:9200/get-together/_search' -d 
    '{
        "query": {
            "range": {
                "updateTime": {
                    "gte": "2020-08-17T11:20:00||-1d",
                    "lte": "2020-08-18"
                }
            }
        }
    }'
    // 比较操作符合"向下舍"的关系
    // gt: 2020-11-18||/M  ES解析:  gt:  2020-11-30T23:59:59.999
    // gte: 2020-11-18||/M ES解析:  gte: 2020-11-01
    // lt: 2020-11-18||/M  ES解析:  lt:  2020-11-01
    // lte: 2020-11-18||/M ES解析:  lte: 2020-11-30T23:59:59.999
    curl 'localhost:9200/get-together/_search' -d 
    '{
        "query": {
            "range": {
                "updateTime": {
                    "gte": "now-8h/d",
                    "lte": "now/d"
                }
            }
        }
    }'
    // 查询参数date格式化
    curl 'localhost:9200/get-together/_search' -d 
    '{
        "query": {
            "range": {
                "updateTime": {
                    "gte": "17/08/2020",
                    "lte": "18/08/2020",
                    "format":"dd/MM/yyyy||yyyy"
                }
            }
        }
    }'
    // Date Math
    // 可在基准日志(anchor date 比如now或者以||结束的date字符串)之后添加1个或多个数学表达式:
    // +1h 加上1小时
    // -1d 减去1天
    // /d  向下舍至最近的一天
    // 例子:
    // now+1h    当前时间加上一个小时
    // now+1h+1m 当前时间加上1个小时1分钟
    // now+1h/d  当前时间加上1个小时,向下舍至最近的一天
    // 2020-01-01||+1M/d 2020-01-01加上一个月,向下舍至最近的一天
    // 支持的时间单位有:
    // 单位		含义
    //  y		  年
    //  M		  月
    //  w		  周
    //  d		  日
    //  h		  小时
    //  H		  小时
    //  m		  分钟
    //  s		  秒
    // range过滤器
    curl 'localhost:9200/get-together/_search' -d 
    '{
        "query": {
            "bool": {
                "must": {
                    "match_all": {}
                },
                "filter": {
                    "range": {
                        "updateTime": {
                            "gt": "1591321611000",
                            "lt": "1607132852000"
                        }
                    }
                }
            }
        }
    }'
    ```

  * <a id="exists">存在与否检索: `exists query`</a>

    ```json
    返回包含字段索引值的文档。
    由于多种原因，文档字段的索引值可能不存在：
    * 源JSON中的字段是null或[]
    * 该字段已"index" : false在映射中设置
    * 字段值的长度超出ignore_above了映射中的设置
    * 字段值格式错误，并且ignore_malformed已在映射中定义
    
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "exists": {
          "field": "memory"
        }
      }
    }
    
    要查找缺少字段索引值的文档，请在 查询中使用must_not exists查询。
    以下搜索返回缺少该user.id字段的索引值的文档。
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must_not": {
            "exists": {
              "field": "ip"
            }
          }
        }
      }
    }
    
    exists过滤器
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "filter": {
            "exists": {
              "field": "referer"
            }
          }
        }
      }
    }
    ```

  * <a id="prefix">前缀检索: `prefix query`</a>

    ```json
    // prefix查询
    // 默认情况下,prefix的匹配对象是针对单个单词的
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "prefix": {
          "referer": {
            "value": "http://facebook.com"
          }
        }
      }
    }
    // prefix过滤器
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must": {
            "match_all": {}
          },
          "filter": {
            "prefix": {
              "referer": "http://facebook.com"
            }
          }
        }
      }
    }
    ```

  * <a id="wildcard">通配符检索: `wildcard query` </a>

    ```json
    This parameter supports two wildcard operators:
    ?, which matches any single character
    *, which can match zero or more characters, including an empty one
    
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "wildcard": {
          "referer": {
            "wildcard": "*success*"
          }
        }
      }
    }
    
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "bool": {
          "must": {
            "match_all": {}
          },
          "filter": {
            "wildcard": {
              "referer": {
                "wildcard": "*success*"
              }
            }
          }
        }
      }
    }
    ```

  * <a id="regexp">正则表达式检索: `regexp query`</a>

    > **value中支持操作符**:
    >
    > * .
    >   Matches any character. For example:
    >
    >   ab.     # matches 'aba', 'abb', 'abz', etc.
    >
    > * ?
    >    Repeat the preceding character zero or one times. Often used to make the preceding character optional. For example:
    >     
    >     abc?     # matches 'ab' and 'abc'
    >    
    > * +
    >  Repeat the preceding character one or more times. For example:
    > 
    >   ab+     # matches 'ab', 'abb', 'abbb', etc.
    >
    > * *
    >  Repeat the preceding character zero or more times. For example:
    > 
    >   ab*     # matches 'a', 'ab', 'abb', 'abbb', etc.
    >
    > * {}
    >  Minimum and maximum number of times the preceding character can repeat. For example:
    > 
    >   a{2}    # matches 'aa'
    >   a{2,4}  # matches 'aa', 'aaa', and 'aaaa'
    >   a{2,}   # matches 'a` repeated two or more times
    > 
    > * |
    >  OR operator. The match will succeed if the longest pattern on either the left side OR the right side matches. For example:
    > 
    >   abc|xyz  # matches 'abc' and 'xyz'
    >
    > * ( … )
    >  Forms a group. You can use a group to treat part of the expression as a single character. For example:
    > 
    >   abc(def)?  # matches 'abc' and 'abcdef' but not 'abcd'
    >
    > * [ … ]
    >    Match one of the characters in the brackets. For example:
    > 
    >   [abc]   # matches 'a', 'b', 'c'
    >    Inside the brackets, - indicates a range unless - is the first character or escaped. For example:
    > 
    >   [a-c]   # matches 'a', 'b', or 'c'
    >    [-abc]  # '-' is first character. Matches '-', 'a', 'b', or 'c'
    >   [abc\-] # Escapes '-'. Matches 'a', 'b', 'c', or '-'
    >   A ^ before a character in the brackets negates the character or range. For example:
    > 
    >   [^abc]      # matches any character except 'a', 'b', or 'c'
    >    [^a-c]      # matches any character except 'a', 'b', or 'c'
    >   [^-abc]     # matches any character except '-', 'a', 'b', or 'c'
    >   [^abc\-]    # matches any character except 'a', 'b', 'c', or '-'
    > 
    > **flags:参数值**
    >
    >   * ALL (Default)
    >  Enables all optional operators
    >
    >* COMPLEMENT
    >   Enables the ~ operator. You can use ~ to negate the shortest following pattern. For example:
    >   a~bc   # matches 'adc' and 'aec' but not 'abc'
    >
    > * INTERVAL
    >   Enables the <> operators. You can use <> to match a numeric range. For example:
    >   foo<1-100>      # matches 'foo1', 'foo2' ... 'foo99', 'foo100'
    >    foo<01-100>     # matches 'foo01', 'foo02' ... 'foo99', 'foo100'
    > 
    > * INTERSECTION
    >   Enables the & operator, which acts as an AND operator. The match will succeed if patterns on both the left side AND the right side matches. For example:
    >   aaa.+&.+bbb  # matches 'aaabbb'
    >
    > * ANYSTRING
    >   Enables the @ operator. You can use @ to match any entire string.
    >   You can combine the @ operator with & and ~ operators to create an "everything except" logic. For example:
    >    @&~(abc.+)  # matches everything except terms beginning with 'abc'
    
    ```json
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "regexp": {
          "user.id": {
            "value": "k.*y",
            "flags": "ALL",
            "case_insensitive": true,
            "max_determinized_states": 10000,
            "rewrite": "constant_score"
          }
        }
      }
    }
    ```
    
  * <a id="fuzzy">模糊检索: `fuzzy query`</a>

    | **参数名**     | **含义**                                                     |
    | -------------- | ------------------------------------------------------------ |
    | fuzziness      | 定义最大的编辑距离，默认为AUTO，即按照es的默认配置。<br/><br/>fuzziness可选的值为0,1,2，也就是说编辑距离最大只能设置为2.<br/><br/>AUTO策略：<br/><br/>在AUTO模式下，es将根据输入查询的term的长度决定编辑距离大小。用户也可以自定义term长度边界的最大和最小值，AUTO:[low],[high]，如果没有定义的话，默认值为3和6，即等价于 AUTO:3,6，即按照以下方案：<br/><br/>输入查询term的长度：<br/><br/>0-2：必须精确匹配<br/><br/>3-5：编辑距离为1<br/>>5：编辑距离为2 |
    | prefix_length  | 定义最初始不会被“模糊”的term的数量。这是基于用户的输入一般不会在最开始犯错误的设定的基础上设置的参数。这个参数的设定将减少去召回限定编辑距离的的term时，检索的term的数量。默认参数为0. |
    | max_expansions | 定义fuzzy query会扩展的最大term的数量。默认为50.             |
    | transpositions | 定义在计算编辑聚利时，是否允许term的交换（例如ab->ba）,实际上，如果设置为true的话，计算的就是Damerau,F,J distance。默认参数为false。 |

    ```json
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "fuzzy": {
          "url": {
            "value": "https:",
            "fuzziness": "AUTO",
            "max_expansions": 50,
            "prefix_length": 0,
            "transpositions": true,
            "rewrite": "constant_score"
          }
        }
      }
    }
    GET /your_index/your_type/_search
    {    
       # Fuzzy Query 模糊查询会提供容错的处理
       "query": {
            "fuzzy" : {
                "user" : {
                    "value": "白日梦",
                    "boost": 1.0,
                    # 最大的纠错次数，一般设为之AUTO
                    "fuzziness": 2,
                    # 不会被“模糊化”的初始字符数。这有助于减少必须检查的术语的数量。默认值为0。
                    "prefix_length": 0,
                    # 模糊查询将扩展到的最大项数。默认值为50
                    "max_expansions": 100 
                    # 是否支持模糊变换(ab→ba)。默认的是false
                    transpositions:true 
                }
            }
        }
    }
    ```

  * <a id="ids">ID检索: `ids query`</a>

    > 根据其ID返回文档。 该查询使用存储在_id字段中的文档ID。
    
    ```json
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "ids": {
          "values": [
            "4fMq6HMB4DKMP4_06Fl3",
            "4vMq6HMB4DKMP4_06Fl3",
            "4_Mq6HMB4DKMP4_06Fl3"
          ]
        }
      }
    }
    ```

* 全文检索

  * <a id="match">分词全文检索: `match query`</a>

    > Returns documents that match a provided text, number, date or boolean value. The provided text is analyzed before matching.
    >
    > The `match` query is the standard query for performing a full-text search, including options for fuzzy matching.
    >
    > 返回与提供的文本、数字、日期或布尔值匹配的文档。
    > 在匹配之前对提供的文本进行分析。
    >
    > 匹配查询是执行全文搜索的标准查询，包括模糊匹配选项。

    ```json
    // 1. match查询,子弹值默认不区分大小写
    curl 'localhost:9200/_search' -d 
    '{
    	"query": {
    		"bool": {
    			"must": {
    				"match": {
    					"firstname":"xxx"
    				}
    			},
    			"filter": {
    				"term":{
    					"xxx": ""
    				}
    			}
    		}
    	}
    }'
    // 2. fistname也可以替换为_all表示匹配所有字段
    curl 'localhost:9200/_search' -d 
    '{
    	"query": {
    		"bool": {
    			"must": {
    				"match": {
    					"_all":"xxx"
    				}
    			},
    			"filter": {
    				"term":{
    					"xxx": ""
    				}
    			}
    		}
    	}
    }'
    // 3. match查询也是一种boolean查询,也就是说match查询接受到text类型的数据后,会对数据进行分析,然后根据提供的text类型数据构造一个boolean查询
    // 即所谓的"分词".通过operator标记(默认值为or)来控制boolean语句
    // 分词: 针对中文,每个文字被拆分为一个待匹配值,针对英文,空格相隔的每个单词被拆分为一个待匹配值
    // 可在建立索引时,设置mapping,为索引的字段指定"index":"not_analyzed"来禁用分词
    POST /_serch?pretty
    {
    	// 会匹配包含"小"和"林"的文档记录
    	"query":{
            "match":{
            	"firstname":"小林"
            }
    	}
    }
    POST /_serch?pretty
    {
    	// 会匹配"小林"
    	"query":{
            "match":{
            	"firstname":"小林",
            	"operator":"and"
            }
    	}
    }
    ```

  * <a id="match_phrase">短语检索: `match_phrase query`</a>

    > match_phrase查询分析文本，并从分析文本中创建短语查询。
    > 类似 match 查询， match_phrase 查询首先将查询字符串解析成一个词项列表，然后对这些词项进行搜索，但只保留那些包含 全部 搜索词项，且 位置 与搜索词项相同的文档
    >
    > 短语检索：要求doc的该字段的值和你给定的值完全相同，顺序也不能变，所以它的精确度很高，但是召回率低。

    ```json
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "match_phrase": {
          "message": {
            "query": "70.133.115.149"
          }
        }
      }
    }
    GET /your_index/your_type/_search
    {   
      # 短语检索 
      # 顺序的保证是通过 term position来保证的
      # 精准度很高，但是召回率低
      "query": {  
          # 只有name字段中包含了完整的 白日梦 这个doc才算命中
           # 不能是单个 ”白日“，也不能是单个的 “梦”，也不能是“白日xxx梦”
           # 要求 短语相连，且顺序也不能变
             "match_phrase": { 
                 "name": "白日梦"
         }
      }
    }
    // slop提高召回率
    GET /your_index/your_type/_search
    {    
       # 短语检索
       "query": {
            # 指定了slop就不再要求搜索term之间必须相邻，而是可以最多间隔slop距离。
             # 在指定了slop参数的情况下，离关键词越近，移动的次数越少， relevance score 越高。
             # match_phrase +  slop 和 proximity match 近似匹配作用类似。
             # 平衡精准度和召回率。
             "match_phrase": { 
                 "address": "mill lane",
            # 指定搜索文本中的几个term经过几次移动后可以匹配到一个doc
                 "slop":2
              } 
      }
    }
    
    // 混合使用match和match_phrase 平衡精准度和召回率
    GET /your_index/your_type/_search
    {    
       # 混合使用match和match_phrase 平衡精准度和召回率
       "query": { 
          "bool": {  
           "must":  {
               # 全文检索虽然可以匹配到大量的文档，但是它不能控制词条之间的距离
               # 可能i love world在doc1中距离很近，但是它却被ES排在结果集的后面
               # 它的性能比match_phrase和proximity高
               "match": {
                 "title": "i love world" 
               } 
            },
           "should": {
                # 因为slop有个特性：词条之间间隔的越近，移动的次数越少 最终的得分就越高
               # 于是可以借助match_phrase+slop感知term position的功能
               # 实现为距离相近的doc贡献分数，让它们靠前排列
               "match_phrase":{
                   "title":{
                       "query":"i love world",
                       "slop":15
                   }
               }
           }
       }
    }
    ```

  * <a id="intervals">间隔检索: intervals query</a>

    >  Returns documents based on the order and proximity of matching terms.
    >
    > The `intervals` query uses **matching rules**, constructed from a small set of definitions. These rules are then applied to terms from a specified `field`.
    >
    > The definitions produce sequences of minimal intervals that span terms in a body of text. These intervals can be further combined and filtered by parent sources.
    >
    > 根据匹配词的顺序和接近程度返回文档。
    > `intervals`查询使用**匹配规则**，由一小部分定义构建而成。然后，将这些规则应用于来自指定`字段`的术语。定义产生跨越正文中术语的最小间隔序列。这些间隔可以按父源进一步组合和过滤。
    >
    > (Required, rule object) Field you wish to search.
    >
    > The value of this parameter is a rule object used to match documents based on matching terms, order, and proximity.
    >
    > Valid rules include:
    >
    > - `match`
    >
    >  | 参数     | 描述                                                         |
    >  | -------- | ------------------------------------------------------------ |
    >  | query    | 用户查询的字符串                                             |
    >  | max_gaps | 字符串中每个词在text field中出现的最大词间距，超过最大间距的将不会被检索到；默认值是-1，即不限制，设置为0的话，query中的字符串必须彼此相连不能拆分 |
    >  | ordered  | query中的字符串是否需要有序显示，默认值是false，即不考虑先后顺序 |
    >  | analyzer | 对query参数中的字符串使用什么分词器，默认使用mapping时该field配置的 search analyzer |
    >  | filter   | 可以为query搭配一个intervals filter，该filter不同于Boolean filter 有自己的语法结构 |
    >
    > - `prefix`
    >
    > - `wildcard`
    >
    > - `fuzzy`
    >
    > - `all_of`
    >
    >  | 参数      | 描述                                                         |
    >  | --------- | ------------------------------------------------------------ |
    >  | intervals | 一个interval集合，集合里面的所有match需要同时在一个文档数据上同时满足才行 |
    >  | max_gaps  | 多个interval查询在一个文档中允许的最大间距，超过最大间距的将不会被检索到；默认值是-1，即不限制，设置为0的话，所有的interval query必须彼此相连不能拆分 |
    >  | ordered   | 配置 intervals 出现的先后顺序，默认值false                   |
    >  | filter    | 可以为query搭配一个intervals filter，该filter不同于Boolean filter 有自己的语法结构 |
    >
    > - `any_of`
    >
    >  | 参数      | 描述                                                         |
    >  | --------- | ------------------------------------------------------------ |
    >  | intervals | 一个interval集合，集合里面的所有match不需要同时在一个文档数据上同时满足 |
    >  | filter    | 可以为query搭配一个intervals filter                          |
    >
    >   

    ```json
    POST _search
    {
      "query": {
        "intervals" : {
          "my_text" : {
            "all_of" : {
              "ordered" : true,
              "intervals" : [
                {
                  "match" : {
                    "query" : "my favorite food",
                    "max_gaps" : 0,
                    "ordered" : true
                  }
                },
                {
                  "any_of" : {
                    "intervals" : [
                      { "match" : { "query" : "hot water" } },
                      { "match" : { "query" : "cold porridge" } }
                    ]
                  }
                }
              ]
            }
          }
        }
      }
    }
    ```

  * <a id="match_phrase_prefix">短语前缀检索: `match_phrase_prefix query`</a>

    > Returns documents that contain the words of a provided text, in the **same order** as provided. The last term of the provided text is treated as a [prefix](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-prefix-query.html), matching any words that begin with that term.
    >
    > 以与提供的顺序相同的顺序返回包含所提供文本的单词的文档。提供的文本的最后一个术语被视为前缀，与以该术语开头的任何单词匹配。
    >
    > **`query`**
    >
    > (Required, string) Text you wish to find in the provided `<field>`.
    >
    > The `match_phrase_prefix` query [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/analysis.html) any provided text into tokens before performing a search. The last term of this text is treated as a [prefix](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-prefix-query.html), matching any words that begin with that term.
    >
    > **`analyzer`**
    >
    > (Optional, string) [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/analysis.html) used to convert text in the `query` value into tokens. Defaults to the [index-time analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/specify-analyzer.html#specify-index-time-analyzer) mapped for the `<field>`. If no analyzer is mapped, the index’s default analyzer is used.
    >
    > **`max_expansions`**
    >
    > (Optional, integer) Maximum number of terms to which the last provided term of the `query` value will expand. Defaults to `50`.
    >
    > **`slop`**
    >
    > (Optional, integer) Maximum number of positions allowed between matching tokens. Defaults to `0`. Transposed terms have a slop of `2`.
    >
    > **`zero_terms_query`**
    >
    > (Optional, string) Indicates whether no documents are returned if the `analyzer` removes all tokens, such as when using a `stop` filter. Valid values are:
    >
    > - **`none` (Default)**
    >
    >   No documents are returned if the `analyzer` removes all tokens.
    >
    > - **`all`**
    >
    >   Returns all documents, similar to a [`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-match-all-query.html) query.

    ```json
    // 与match_phrase类似,不同点在于输入文本中,最后一项term允许前缀匹配
    GET /kibana_sample_data_logs/_search
    {
      "query": {
         # 前缀匹配，相对于全文检索，前缀匹配是不会对前缀进行分词的。
         # 而且每次匹配都会扫描整个倒排索引，直到扫描完一遍才会停下来
         # 前缀搜索不会计算相关性得分所有的doc的得分都是1
         # 前缀越短能匹配到的doc就越多，性能越不好
        "match_phrase_prefix": {
          "url": {
            "query": "www.elastic.co",
            "max_expansions": "10"
          }
        }
      },
      "_source": [
        "url"
      ]
    }
    // 查询会先创建词组clever brown(即目标记录必须包含 clever且后面跟brown,clever和brown之间至少一个空格或tab),然后查询fo开头的项,比如fox,fog....然后添加到词组,组成新的词组clever brown fox,clever brown fog,...然后以match_phrese的方式去查找这些匹配这些新词组的文档记录
    // 搜索推荐：match_phrase_prefix，最终实现的效果类似于百度搜索，当用户输入一个词条后，将其它符合条件的词条的选项推送出来。
    GET /your_index/your_type/_search
    {    
       "query": {
          # 前缀匹配（关键字）
          "match_phrase_prefix" : {
            "message" : {
               # 比如你搜索关注白日梦，经过分词器处理后会得到最后一个词是：“白日梦”
                # 然后他会拿着白日梦再发起一次搜索，于是你就可能搜到下面的内容：
                    # “关注白日梦的微信公众号”
               # ”关注白日梦的圈子“
                    "query" : "关注白日梦",
                    # 指定前缀最多匹配多少个term，超过这个数量就不在倒排索引中检索了，提升性能
                    "max_expansions" : 10,
                    # 提高召回率，使用slop调整term persition，贡献得分
                    "slop":10
                }
           } 
      }
    }
    ```

  * <a id="multi_match">多字段匹配检索: `multi_match query`</a>

    > The way the `multi_match` query is executed internally depends on the `type` parameter, which can be set to:
    >
    > | 参数            | 说明                                                         |
    > | --------------- | ------------------------------------------------------------ |
    > | `best_fields`   | (**default**) Finds documents which match any field, but uses the `_score` from the best field. See [`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-multi-match-query.html#type-best-fields).优先返回某个field匹配到更多关键字的doc。 |
    > | `most_fields`   | Finds documents which match any field and combines the `_score` from each field. See [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-multi-match-query.html#type-most-fields).优先返回有更多的field匹配到你给定的关键字的doc。most_fields不支持使用`minimum_should_match`去长尾。 |
    > | `cross_fields`  | Treats fields with the same `analyzer` as though they were one big field. Looks for each word in **any** field. See [`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-multi-match-query.html#type-cross-fields). |
    > | `phrase`        | Runs a `match_phrase` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-multi-match-query.html#type-phrase). |
    > | `phrase_prefix` | Runs a `match_phrase_prefix` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-multi-match-query.html#type-phrase). |
    > | `bool_prefix`   | Creates a `match_bool_prefix` query on each field and combines the `_score` from each field. See [`bool_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-multi-match-query.html#type-bool-prefix). |

    ```json
    // multi_match与match类似,不同的是针对一个待查询的内容,可在多个指定名称字段中进行匹配
    // 同match,multi_match要查询的内容不分大小写
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "multi_match": {
          "query": "elastic",
          "fields": [
            "message",
            "url"
          ]
        }
      },
      "_source": [
        "message",
        "url"
      ]
    }
    
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "multi_match": {
          "query": "elastic",
          "fields": [
            "url",
            "re*"
          ]
        }
      },
      "_source": [
        "url",
         "re*"
      ]
    }
    // cross_fields
    GET /your_index/your_type/_search
    {    
        "query": { 
           "multi_match":{
               "query":"golang java",
                # cross_fields 要求golang：必须在title或者在content中出现
                # cross_fields 要求java：必须在title或者在content中出现
               "type":"cross_fields",
               "fields":["title","content"]
           }
        }
    }
    ```

  * <a id="query_string">支持与或非的字符串检索: `query_string`</a>

    > 一个使用查询解析器解析其内容的查询。
    > query_string查询提供了以简明的简写语法执行多匹配查询 multi_match queries ，布尔查询 bool queries ，提升得分 boosting ，模糊匹配 fuzzy matching ，通配符 wildcards ，正则表达式 regexp 和范围查询 range queries 的方式。

    ```json
    curl -XPOST 'http://localhost:9200/get-together/_search?pretty' -d 
    '{
    	"query": {
    		"query_string": {
    			// 由于查询中没有指定字段,所以使用了默认字段"description"
    			"default_field": "description",
    			// 带解析的查询字符串
    			"query": "nosql",
    			// 多字段,支持通配符
    			"fields":["name","first*"],
    			// 指定默认的操作符
    			"default_operator":"OR"
    		}
    	}
    }'
    // Query String 语法
    // 1. Field Name
    // 1) field_name:content 字段field_name包含内容content
    curl -XPOST 'http://localhost:9200/get-together/_search?pretty' -d 
    '{
    	"query": {
    		"query_string": {
    			"query": "addr:suzhou"
    		}
    	}
    }'
    // 2) field_name:(content1 OR content2)/field_name:(content1 content2)  字段field_name包含内容content1或content2
    curl -XPOST 'http://localhost:9200/get-together/_search?pretty' -d 
    '{
    	"query": {
    		"query_string": {
    			"query": "addr:(suzhou OR shanghai)"
    		}
    	}
    }'
    // 3) field_name:"content1 content2" 字段field_name包含内容content1 content2
    curl -XPOST 'http://localhost:9200/get-together/_search?pretty' -d 
    '{
    	"query": {
    		"query_string": {
    			"query": "addr:\"suzhou wuzhong\""
    		}
    	}
    }'
    // 4) field_name:"content1 content2" 字段field_name包含内容content1 content2
    curl -XPOST 'http://localhost:9200/get-together/_search?pretty' -d 
    '{
    	"query": {
    		"query_string": {
    			"query": "addr:\"suzhou wuzhong\""
    		}
    	}
    }'
    // 5) _exists_:field_name 存在字段field_name且field_name不为空,不为null
    curl -XPOST 'http://localhost:9200/get-together/_search?pretty' -d 
    '{
    	"query": {
    		"query_string": {
    			"query": "_exists_:addr"
    		}
    	}
    }'
    // 6) Range 支持date,numeric,字符串字段.闭区间[min TO max],开区间{min TO max}
    curl -XPOST 'http://localhost:9200/get-together/_search?pretty' -d 
    '{
    	"query": {
    		"query_string": {
    			"query": "birthday:[2020-12-01 TO 2020-12-31]"
    			// "query": "birthday:[* TO 2020-12-31}"
    			// "query": "age:[10 TO 30]"
    			// "query": "age:[15 TO *]" <==> "query": "age:>=15"
    			// "query": "age:[15 TO 25}" <==> "query": "age:(>=15 AND <25)"
    		}
    	}
    }'
    // 7) bool操作符
    // +content1: 包含content
    // -content: 不包含content
    curl -XPOST 'http://localhost:9200/get-together/_search?pretty' -d 
    '{
    	"query": {
    		"query_string": {
    			"query": "addr:suzhou +wuzhong -huqiu"
    		}
    	}
    }'
    ```

  * <a id="simple_query_string">简单化的字符串检索: `simple_query_string`</a>

    > 一个使用SimpleQueryParser解析其上下文的查询。 与常规query_string查询不同，simple_query_string查询永远不会抛出异常，并丢弃查询的无效部分。

    ```json
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "simple_query_string" : {
            "query": "+elastic +(blog | potato) -weblogs",
            "fields": ["referer^5", "url"],
            "default_operator": "and"
        }
      },
      "_source": [
        "url",
         "re*"
      ]
    }
    ```
    
  * <a id="match_boolean_prefix">`match_boolean_prefix`</a>

    > An important difference between the `match_bool_prefix` query and [`match_phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-match-query-phrase-prefix.html) is that the `match_phrase_prefix` query matches its terms as a phrase, but the `match_bool_prefix` query can match its terms in any position. The example `match_bool_prefix` query above could match a field containing `quick brown fox`, but it could also match `brown fox quick`. It could also match a field containing the term `quick`, the term `brown` and a term starting with `f`, appearing in any position.

    ```json
    GET /_search
    {
      "query": {
        "match_bool_prefix" : {
          "message" : "quick brown f"
        }
      }
    }
    ==>
    GET /_search
    {
      "query": {
        "bool" : {
          "should": [
            { "term": { "message": "quick" }},
            { "term": { "message": "brown" }},
            { "prefix": { "message": "f"}}
          ]
        }
      }
    }
    ```

* 复合检索

  * <a id="constant_score">固定得分检索 constant_score query</a>

    > Wraps a [filter query](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl-bool-query.html) and returns every matching document with a [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-filter-context.html#relevance-scores) equal to the `boost` parameter value.

    ```json
    GET /kibana_sample_data_logs/_search
    {
      "query": {
        "constant_score": {
          "filter": {
            "term": {
              "machine.os.keyword": "win xp"
            }
          },
          "boost": 1.2
        }
      }
    }
    
    GET /your_index/your_type/_search
    {    
      "query": {
            "function_score": {
                "query": { 
                "match": {
                 "query":"es"
                 } 
           },
           “field_value_factor”:{
                  "field":"star",
                  "modifier":"log1p",
                  # 使用factor将star字段对权重的影响降低成1/10
                  # newScore = oldScore + log( 1 + star*factor )
            "factor":0.1
                }
                "boost_mode":"multiply",
          "maxboost":3
            }
        }
    }
    ```

  * bool组合检索

    > bool查询允许你在单独查询中组合任意数量的查询,执行查询子句表明哪些部分是**必须(must)匹配**,**应该(should)匹配**或者是**不能(must_not)匹配**上Elasticsearch索引里的数据
    >
    > *  如果执行bool查询的某部分是must匹配,只有匹配上这些查询的结果才会被返回;
    > *  如果指定了bool查询的某部分是should匹配,只有匹配上指定数量子句的文档才会被返回;
    > *  如果没有执行must匹配的子句,文档至少要匹配一个should子句才能返回;
    > *  must_not子句会使得匹配其的文档被移除结果集合;

    | bool查询子句 | 等价的二元操作                                               | 含义                                                         |
    | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | must         | 为了组合多个子句,使用二元操作and<br />(query1 AND query2 AND query3) | 在must子句中的任何搜索必须匹配上文档,小写的and是功能,大写的AND是操作符 |
    | must_not     | 使用二元操作not组合多个子句                                  | 在must_not子句中任何搜索不能是文档的一部分.多个子句通过not二元操作符进行组合(NOT query1 AND NOT query2 AND NOT query3) |
    | should       | 使用二元操作or组合多个子句(query1 OR query2 OR query3)       | 在should子句中搜索,可以匹配也可以不匹配一篇文档,但是匹配数至少达到minimum_should_match参数所设置的数量(如果没有使用must那么默认私用1,如果使用了must,默认是0),和二元查找操作OR类型(query1 OR query2 OR query3) 类似 |

  * 改变评分检索

    * <a id="dis_max">最佳字段查询 Disjunction max query</a>

      > Returns documents matching one or more wrapped queries, called query clauses or clauses.
      >
      > If a returned document matches multiple query clauses, the `dis_max` query assigns the document the highest relevance score from any matching clause, plus a tie breaking increment for any additional matching subqueries.
      >
      > You can use the `dis_max` to search for a term in fields mapped with different [boost](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/mapping-boost.html) factors.
      >
      > 分离最大化查询（*Disjunction Max Query*） 。分离（Disjunction）的意思是 或（or） ，这与可以把结合（conjunction）理解成 与（and） 相对应。分离最大化查询（*Disjunction Max Query*）指的是： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回 ：
      >
      > **`queries`**
      >
      > (Required, array of query objects) Contains one or more query clauses. Returned documents **must match one or more** of these queries. If a document matches multiple queries, Elasticsearch uses the highest [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-filter-context.html).
      >
      > **`tie_breaker`**
      >
      > (Optional, float) Floating point number between `0` and `1.0` used to increase the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-filter-context.html#relevance-scores) of documents matching multiple query clauses. Defaults to `0.0`.
      >
      > You can use the `tie_breaker` value to assign higher relevance scores to documents that contain the same term in multiple fields than documents that contain this term in only the best of those multiple fields, without confusing this with the better case of two different terms in the multiple fields.
      >
      > If a document matches multiple clauses, the `dis_max` query calculates the relevance score for the document as follows:
      >
      > 1. Take the relevance score from a matching clause with the highest score.
      > 2. Multiply the score from any other matching clauses by the `tie_breaker` value.
      > 3. Add the highest score to the multiplied scores.
      >
      > If the `tie_breaker` value is greater than `0.0`, all matching clauses count, but the clause with the highest score counts most.

      ```json
      GET /kibana_sample_data_logs/_search
      {
        "query": {
          "dis_max": {
            "queries": [
              { "term": { "url": "elastic" } },
              { "term": { "message": "elastic" } }
            ],
            "tie_breaker": 0.7
          }
        }
      }
      ```

    * <a id="function_score">function_score query</a>

      >  The `function_score` allows you to modify the score of documents that are retrieved by a query. This can be useful if, for example, a score function is computationally expensive and it is sufficient to compute the score on a filtered set of documents.
      >
      > To use `function_score`, the user has to define a query and one or more functions, that compute a new score for each document returned by the query.

      ```
      GET /_search
      {
        "query": {
          "function_score": {
            "query": { "match_all": {} },
            "boost": "5",
            "random_score": {}, 
            "boost_mode": "multiply"
          }
        }
      }
      ```

    * <a id="boosting">Boosting query</a>

      > Returns documents matching a `positive` query while reducing the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-filter-context.html#relevance-scores) of documents that also match a `negative` query.
      >
      > You can use the `boosting` query to demote certain documents without excluding them from the search results.
      >
      > - **`positive`**
      >
      >   (Required, query object) Query you wish to run. Any returned documents must match this query.
      >
      > - **`negative`**
      >
      >   (Required, query object) Query used to decrease the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-filter-context.html#relevance-scores) of matching documents.If a returned document matches the `positive` query and this query, the `boosting` query calculates the final [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-filter-context.html#relevance-scores) for the document as follows:Take the original relevance score from the `positive` query.Multiply the score by the `negative_boost` value.
      >
      > - **`negative_boost`**
      >
      >   (Required, float) Floating point number between `0` and `1.0` used to decrease the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-filter-context.html#relevance-scores) of documents matching the `negative` query.

      ```json
      GET /kibana_sample_data_logs/_search
      {
        "query": {
          "boosting": {
            "positive": {
              "terms": {
                "tags": ["error","security","warning"]
              }
            },
            "negative": {
              "terms": {
                "tags": ["success","info"]
              }
            },
            "negative_boost": 0.5
          }
        },
        "_source": ["tags"]
      }
      ```

* <a id="match_all">Match all query,Match none query</a> 

  ```
  The most simple query, which matches all documents, giving them all a _score of 1.0.
  GET /kibana_sample_data_logs/_search
  {
    "query": {
      "match_all": {}
    }
  }
  
  This is the inverse of the match_all query, which matches no documents.
  GET /kibana_sample_data_logs/_search
  {
    "query": {
      "match_none": {}
    }
  }
  ```

* 跨度检索 Span queries

    * Span containing query
    * Span field masking query
    * Span first query
    * Span multi-term query
    * Span near query
    *  Span not query
    * Span or query
    * Span term query
    * Span within query

* 特定检索 Specialized queries

    * Distance feature query
    * 相似文章检索 More like this query
    * Percolate query
    * Rank feature query
    * 脚本检索 Script query
    * Script score query
    * Wrapper query
    * Pinned Query

* 联接查询
  
    * 嵌套查询 Nested query
    * Has child query
    * Has parent query
    * Parent ID query
    
* Geo类型检索
    * 地理边框检索 Geo-bounding box query
    * 地理距离查询 Geo-distance query
    * 地理距离范围查询 Geo-polygon query
    * 地理形状查询 Geo-shape query
    
    











