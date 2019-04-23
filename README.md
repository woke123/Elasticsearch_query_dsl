# elasticsearch常用查询语法整理

## 全文查询

##### match 

    模糊匹配和短语查询
    params
    query     查询的关键词
    operator  查询的条件，默认为  or   可以改为  and
    analyzer  分词器，默认使用standard
    lenient   报错处理，为true的时候会忽略异常报错，默认false
    
    {
      "query": {
        "match": {
           "intro": {
              "query":"is test",
              "operator": "and",
              "analyzer":"standard",
              "lenient":true
           }
        }
      }
    }
    
##### match_phrase  

    与match查询类似，但用于匹配精确短语或单词近似匹配
    params
    query     查询的关键词
    slop      关键词位置的距离，默认0 例如："this is test" 当设置slop为1时，可以搜索 "this test"
    analyzer  分词器，默认使用standard
    
    {
      "query": {
        "match_phrase": {
           "intro": {
              "query":"test",
              "slop":0,
              "analyzer":"standard"
           }
        }
      }
    }
    
##### match_phrase_prefix          

    与match_phrase数查询类似，但对最后一个单词进行前缀匹配
    params 和 match_phrase 一致
    {
      "query": {
        "match_phrase_prefix": {
           "intro":  {
              "query":"is te"
           }
        }
      }
    }
    
##### multi_match

    match查询的多字段版本
    params
    query     查询的关键词
    operator  查询的条件，默认为  or   可以改为  and
    fields    查询的字段
    
    {
      "query": {
        "multi_match": {
              "query":"is es",
              "operator":"and",
              "fields": ["intro","email"]
        }
      }
    }

##### query_string  

    Lucence查询语句。简单的例子，这一块的查询语法很多，参考文档
    params   
    query    查询的条件
    fields   查询的字段
    
    {
      "query": {
        "query_string": {
              "query": "(13000111510@163.com OR 15209134122@163.com) AND (this is test) OR (this is es)",
              "fields": ["email", "intro"]
        }
      }
    }
    
##### simple_query_string

    是query_string的简单版本，使用+/|/- 替换了AND/OR/NOT。语法错误会忽略，不会抛异常
    params
    
    {
      "query": {
        "simple_query_string": {
              "query": "(13000111510@163.com | 15209134122@163.com) + (this is test) | (this is es)",
              "fields": ["email", "intro"]
        }
      }
    }
    
## 术语查询

##### term  

    查找字段里面是否包含某个关键词
    
    params
    query 查询的关键词
    {
      "query":{
        "term": {
          "intro": "test"
        }
      }
    }
    
##### terms
    字段包含多个关键词的
    {
      "query":{
        "terms": {
          "intro":  ["this","test"]
        }
      }
    }
    
##### range
    指定字段的范围查询（日期，数字，字符串）
    params
    gte      大于等于
    gt       大于
    lte      小于等于
    lt       小于
    {
      "query": {
        "range": {
          "age": {
            "gt": 10,
            "lt": 30
          }
        }
      }
    }

##### prefix
    前缀匹配查询
    value   查询的前缀
    {
      "query": {
        "prefix": {
          "email": {
            "value": "15"
          }
        }
      }
    }
    
##### ids
    多个id查询
    values  查询id
    {
      "query": {
          "ids": {
            "values": ["nSE4SGoBrVGGGwRI_BBh","nCE0SGoBrVGGGwRI1BCy"]
          }
      }
    } 

## 复合查询

##### constant_score
    复合查询，可以包装一个其他类型的查询，可以修改查询的_score
    {
      "query": {
          "constant_score": {
            "filter": {
    		    "prefix": {
    		      "email": {
    		        "value": "15"
    		      }
    		    }
            },
            "boost":"1.5"
          }
      }
    }

##### bool 
        
        用于组合多个查询语句
        must： #条件必须满足，相当于and，对于一个文档，所有的查询都必须为真，这个文档才能够匹配成功 
        should: #相当于or，对于一个文档，查询列表中，只要有一个查询匹配，那么这个文档就被看成是匹配的 
        must_not: #相当于not，对于一个文档，查询列表中的的所有查询都必须都不为真，这个文档才被认为是匹配的 
        filter： #只过滤符合条件的文档，不计算相关性得分
        {
          "query": {
              "bool": {
                 "must":[
                    {
        	            "prefix": {
        			      "email": {
        			        "value": "15"
        			      }
        			    }
        		    },
        		    {
        			    "term": {
        				  "intro": "test"
        				}
        			}
                 ],
                 "should":[
                    {
        			    "term": {
        				  "age": "25"
        				}
        			},
        			{
        				"term": {
        				  "status": 1
        				}
        			}
                 ]
              }
          }
        } 
