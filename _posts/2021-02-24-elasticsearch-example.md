---
layout: post
title: "Elasticsearch 시작하기"
tags: [Elasticsearch]
date: 2021-02-24
comments: true
---

<br>

# OverView

이번시간에는 검색 또는 분석엔진으로 사용되는 Elasticsearch의 특징과 설치방법에 대해서 간단하게 알아보고 실제로 RestAPI를 사용해서 문서를 색인, 검색, 분석하는 작업을 해보도록 하겠다.

# Elasticsearch란?

![화면 캡처 2021-02-24 082341](https://user-images.githubusercontent.com/30790184/108921402-a10c7280-7679-11eb-8c91-1f6bbdb99a43.png)

Elasticsearch는 Document기반의 NoSQL 저장소이다. Elasticsearch는 Lucene 기반의 오픈소스 검색 엔진이고 JSON기반의 문서를 저장한다. Elasticsearch는 검색뿐만 아니라 분석엔진으로도 활용된다.

Elasticsearch는 다음과같은 4가지 특징을 갖고 있다.

| **항목**           | **특징**                                                     |
| ------------------ | ------------------------------------------------------------ |
| 준실시간 검색 엔진 | 실시간이라고 생각할 만큼 색인된 데이터를 매우 빠르게 검색할 수 있음 |
| 클러스터 구성      | 한 대 이상의 노드를 클러스터로 구성하여 높은 수준의 안정성을 이루고 부하를 분산할 수 있음 |
| 스키마리스         | 입력될 데이터에 대해 미리 정의하지 않아도 동적으로 스키마를 생성할 수 있음 |
| REST API           | REST  API 기반의 쉬운 인터페이스를 제공하여 비교적 진입 장벽이 낮음 |

# Elasticsearch 설치하기

Elasticsearch는 다양한 설치 방법을 지원하기때문에 자세한 정보는 [공식 홈페이지](https://www.elastic.co/kr/downloads/elasticsearch)를 참고하면 좋고 이 포스팅에서는 단순하게 `tar.gz` 파일로 설치해보도록 하겠다.

```shell
#다운로드 & 설치
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz
tar -zxvf elasticsearch-7.11.1-linux-x86_64.tar.gz


#Elasticsearch 실행하기
./{elasticsearch_dir}/bin/elasticsearch
```

# Elasticsearch 사용해보기

Elasticsearch에 대한 기본 개념을 알아보기 이전에 RestAPI를 사용해서 Elasticsearch의 스키마리스 기능, 색인, 검색, 분석을 직접 확인해보도록 하겠다. 먼저 다음 명령어를 통해 Elasticsearch 서버가 잘 동작하는지 확인해보자.

```shell
curl localhost:9200

{
  "name" : "elasticsearch",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "urmOwg88Rkyczwwk4_Cqcw",
  "version" : {
    "number" : "7.10.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "747e1cc71def077253878a59143c1f785afa92b9",
    "build_date" : "2021-01-13T00:42:12.435326Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```



## 문서 색인하기

위에서 설명했듯이 Elasticsearch는 스키마리스 기능이 있어서 `user`라는 인덱스가 없어도 자동으로 생성해준다. 다음 명령어를 실행시켜보자.

```sh
curl -X PUT 'localhost:9200/user/_doc/1?pretty' \
-H 'Content-Type: application/json' \
-d '{
	 "username" : "sup2is"
}'


{
  "_index" : "user",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

```

위 요청은 `PUT`  메서드를 활용해서 `user`라는 인덱스 내부, `_doc` 이라는 타입에 `1번 문서`를 색인해달라는 쿼리이다. 실제 데이터는 `-d` 옵션에 JSON 형태로 저장을 한다.

요청이 잘 전달되었다면 `result` 필드가 `created`로 넘어오는것을 확인할 수 있다.

## 문서 조회하기

문서의 조회는 GET 메서드를 활용한다.

```sh
curl -X GET "localhost:9200/user/_doc/1?pretty"
```

위 요청은 `GET` 메서드를 활용해서 `user` 인덱스 내부, `_doc` 이라는 타입에 `1번 문서`를 조회하는 쿼리이다. 파라미터로 `pretty` 옵션을 주어 가독성이 있는 형태로 결과를 확인할 수 있다.

```json
{
  "_index" : "user",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "username" : "sup2is"
  }
}
```



## 매핑 확인하기

위에서 만든 `user`인덱스의 매핑정보(스키마)`_mappings` API를 사용해서 확인할 수 있다.

```sh
curl -s "localhost:9200/user/_mappings?pretty"


{
  "user" : {
    "mappings" : {
      "properties" : {
        "username" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

기존에 `user` 인덱스에는 `username` 필드만 있었지만 다음과 같이 `age`라는 필드를 추가한 뒤에 다시 `_mappings` API를 사용해서 매핑 정보를 확인해보자.

```sh
curl -X PUT 'localhost:9200/user/_doc/1?pretty' \
-H 'Content-Type: application/json' \
-d '{
	"username": "daniel",
	"age": 28 
}'

```



```sh
curl -s "localhost:9200/user/_mappings?pretty"

{
  "user" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "username" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

위 결과와 같이 새롭게 `age`라는 필드가 매핑되었음을 확인할 수 있다.

Elasticsearch의 스키마리스 기능은 최초에 저장된 데이터 타입에 따라 자동적으로 매핑을 결정하는데 만약 초기에 매핑된 데이터 타입과 다른 데이터가 들어온다면 다음과 같은 에러가 나오기때문에 주의해야한다.

```sh
curl -X PUT 'localhost:9200/user/_doc/1?pretty' \
-H 'Content-Type: application/json' \
-d '{
	"username": "daniel",
	"age": "invalid data type" 
}'


{
  "error" : {
    "root_cause" : [
      {
        "type" : "mapper_parsing_exception",
        "reason" : "failed to parse field [age] of type [long] in document with id '1'. Preview of field's value: 'invalid data type'"
      }
    ],
    "type" : "mapper_parsing_exception",
    "reason" : "failed to parse field [age] of type [long] in document with id '1'. Preview of field's value: 'invalid data type'",
    "caused_by" : {
      "type" : "illegal_argument_exception",
      "reason" : "For input string: \"invalid data type\""
    }
  },
  "status" : 400
}
```



## 문서 검색하기

Elasticsearch의 검색기능을 활용하기 이전에 먼저 검색에 사용할 대량의 데이터셋을 넣어주는 작업을 먼저 수행해보자.

대량의 JSON데이터는 `bulk` API를 활용해서 한번에 모든 문서를 색인할 수 있다.

```sh
curl -X POST "localhost:9200/accounts/_doc/_bulk?pretty&refresh" \
-H 'Content-Type: application/json' \
--data-binary "@accounts.json"

```

> accounts.json은 [https://github.com/sup2is/study/blob/master/elasticsearch/accounts.json](https://github.com/sup2is/study/blob/master/elasticsearch/accounts.json)에서 확인할 수 있다.



### Query String 사용하기

Elasticsearch의 검색은 `_search` API를 사용한다. 먼저 Query String을 사용한 방식을 설명하도록 하겠다.

```sh
curl -X GET "localhost:9200/accounts/_search?q=pyrami&pretty"
```

위 요청은 `q` 파라미터에 `*` 을주어 Elasticsearch 인덱스 전체를 검색하는 풀스캔 요청 쿼리이다. 

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        }
      },
      
      
//      ... 후략
```

다음은 `state`라는 필드가 `IN`으로 되어있는 문서를 검색하는 요청이다.

```sh
curl -X GET "localhost:9200/accounts/_search?q=IN&pretty"

{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 15,
      "relation" : "eq"
    },
    "max_score" : 4.1679144,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "_doc",
        "_id" : "32",
        "_score" : 4.1679144,
        "_source" : {
          "account_number" : 32,
          "balance" : 48086,
          "firstname" : "Dillard",
          "lastname" : "Mcpherson",
          "age" : 34,
          "gender" : "F",
          "address" : "702 Quentin Street",
          "employer" : "Quailcom",
          "email" : "dillardmcpherson@quailcom.com",
          "city" : "Veguita",
          "state" : "IN"
        }
      },


	//... 후략
```

위에서 사용한 QueryString 방식은은 단순하지만 복잡한 질의를 하기엔 제한되는 단점때문에 효율적인 검색을 위해 Query DSL을 많이 사용하는 편이다.



### Query DSL 사용하기

Elasticsearch는 Query DSL을 제공해서 검색에 대한 도메인 특정 언어를 지원한다. Query DSL에 대한 자세한 정보는 [https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) 에서 확인할 수 있다.

실제로 Query DSL을 사용해서 문서를 검색해보자.

```sh
curl -X GET "localhost:9200/accounts/_search?pretty" \
-H 'Content-Type: application/json' \
-d '{
  "query": {
    "match": {
      "city": "Veguita"
    }
  }
}'

```

위 쿼리로 `city` 라는 필드가 `Veguita`인 문서를 검색할 수 있다.

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 6.5059485,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "_doc",
        "_id" : "32",
        "_score" : 6.5059485,
        "_source" : {
          "account_number" : 32,
          "balance" : 48086,
          "firstname" : "Dillard",
          "lastname" : "Mcpherson",
          "age" : 34,
          "gender" : "F",
          "address" : "702 Quentin Street",
          "employer" : "Quailcom",
          "email" : "dillardmcpherson@quailcom.com",
          "city" : "Veguita",
          "state" : "IN"
        }
      }
    ]
  }
}
```

이런 단순한 검색뿐만아니라 범위, 중첩 등등 다양한 QueryDSL 문법을 제공한다. 다음 요청은 `age` 값이 `20`보다 큰 문서만 검색하는 요청이다.

```sh
curl -X GET "localhost:9200/accounts/_search?pretty" \
-H 'Content-Type: application/json' \
-d '{
  "query": {
    "bool": {
      "must": {"match_all": {}},
      "filter": {
        "range": {
          "age": {
            "gte": 20
          }
        }
      }
    }
  }
}'


{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        }
      },
      
      
      //...후략
```



## 문서 분석하기

Elasticsearch는 검색작업을 바탕으로 분석작업을 할 수 있다. 이러한 분석 작업을 aggregation이라고 하고 검색에 사용한 `_search` API를 기반으로 동작한다.

다음 요청은 인덱스 내부에 가장 많은 `state` 를 가진 문서들의 총 개수만큼 내림차순으로 정렬한 결과를 반환하는 쿼리이다.

```shell
curl -X GET "localhost:9200/accounts/_search?pretty" \
-H 'Content-Type: application/json' \
-d '{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}'

{
  "took" : 47,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30
        },
        {
          "key" : "MD",
          "doc_count" : 28
        },
        {
          "key" : "ID",
          "doc_count" : 27
        },
        
        // ... 후략
```

aggregation 역시 중첩된 구조로도 사용할 수 있다.

다음 쿼리는 `state` 를 가진 문서들의 총 개수와 평균 나이를 분석하는 쿼리이다.

```sh
curl -X GET "localhost:9200/accounts/_search?pretty" \
-H 'Content-Type: application/json' \
-d '{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}'


{
  "took" : 22,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30,
          "average_age" : {
            "value" : 28.566666666666666
          }
        },
        {
          "key" : "MD",
          "doc_count" : 28,
          "average_age" : {
            "value" : 30.928571428571427
          }
        },
        {
          "key" : "ID",
          "doc_count" : 27,
          "average_age" : {
            "value" : 31.59259259259259
          }
        },
        
        
        
        // ... 후략
```



# 마무리

이번 시간에는 Elasticsearch의 간단한 특징과 사용방법에 대해서 알아봤다. 다음시간에는 Elasticsearch의 기본 개념들인 클러스터, 노드, 샤드, 세그먼트들에 대해서 알아보는 시간을 가져보도록 하겠다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

- 엘라스틱서치 실무 가이드 - 위키북스
- 기초부터 다지는 ELASTICSEARCH 운영 노하우 - 인사이트
- [https://www.elastic.co/guide/index.html](https://www.elastic.co/guide/index.html)
