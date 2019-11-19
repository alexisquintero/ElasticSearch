# ElasticSearch commands based on 6.2 documentation
Commands instructions are meant for vim, ignore backticks in commands unless specified.\
Run commands by typing `:! <c-r><c-l><CR>` while having the cursor on the desired line.\
Edit command by pressing `<c-f>` on command-line mode.\
Make multiline queries into singleline by running `vipJ`, assuming `vip` selects the whole query, `u` to undo.\
Get output from command on new buffer by typing `<c-w><c-s>:enew<CR>:read !!<CR>` after running the command once.\
'index' should be replaced with actual index, removing '. "localhost/'index'/_doc/" -> "localhost/customer/_doc/".\
Remember to delete commas while editing multiline queries.\

## [Cluster health](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_cluster_health.html)

```
curl -X GET "localhost:9200/_cat/health?v&pretty"
```

### Nodes in cluster

```
curl -X GET "localhost:9200/_cat/nodes?v&pretty"
```

## [List all indices](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_list_all_indices.html)

```
curl -X GET "localhost:9200/_cat/indices?v&pretty"
```

## [Add and Get data](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_index_and_query_a_document.html)
### Add data to 'index'

```

curl -X PUT "localhost:9200/'index'/_doc/1?pretty&pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}'

```

### Get data from 'index'

```
curl -X GET "localhost:9200/'index'/_doc/1?pretty&pretty"
```

## [Delete data](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_delete_an_index.html)
### Delete 'index'

```
curl -X DELETE "localhost:9200/'index'?pretty&pretty"
```

### ElasticSearch commands pattern `<REST Verb> /<Index>/<Type>/<ID>`

## [Update document](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_updating_documents.html)
### Does not do in-place updates, deletes old document and then re-index new one
### Update document in 'index'

```

curl -X POST "localhost:9200/'index'/_doc/1/_update?pretty&pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe" }
}'

```

### Update document in 'index' using script

```

curl -X POST "localhost:9200/'index'/_doc/1/_update?pretty&pretty" -H 'Content-Type: application/json' -d'
{
  "script" : "ctx._source.age += 5"
}'

```

## [Delete document](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_deleting_documents.html)
### Delete document in 'index'

```
curl -X DELETE "localhost:9200/'index'/_doc/2?pretty&pretty"
```

## [Batch](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_batch_processing.html)
### Does not stop if one item fails
### Batch processing in 'index'
#### Index document with id1 and id2

```

curl -X POST "localhost:9200/'index'/_doc/_bulk?pretty&pretty" -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }'

```

#### Update document with id1 and delete document with id2

```

curl -X POST "localhost:9200/customer/_doc/_bulk?pretty&pretty" -H 'Content-Type: application/json' -d'
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}'

```

## [Load data from file](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_exploring_your_data.html)
### Load data to 'index' from 'file.json'

```
curl -H "Content-Type: application/json" -XPOST "localhost:9200/'index'/_doc/_bulk?pretty&refresh" --data-binary "@file.json"
```

## [Search](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_the_search_api.html)
### Search through request URI (replace 'field')

```
curl -X GET "localhost:9200/'index'/_search?q=*&sort='field':asc&pretty&pretty"
```

### Search through request body, remove sort to get all matches unsorted

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
    "sort": [
    { "'field'": "asc" }
    ]
}'

```
```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
    "sort": [
    { "'field'": "asc" }
    ]
}'

```

## [Query language](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_introducing_the_query_language.html)

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10,
  "sort": { "'field'": { "order": "desc" } },
  "_source": ["'field1", "'field2'"]
}'

```

#### "query"   -> actual query
#### "from"    -> starting field
#### "size"    -> amount of items
#### "sort"    -> sorting
#### "_source" -> similar to SELECT from SQL, filters the returned items

## [Searches](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_executing_searches.html)
## [Match query](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl-match-query.html): search done
 against specific fields
### Returns account numbered 20

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "account_number": 20 } }
}'

```

### Returns all accounts containing the term "mill" OR "lane" in the address

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}'

```

### Returns all accounts containing the phrase "mill lane" in the address

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}'

```

## Bool query: allows to compose smaller queries using boolean logic
### Returns all accounts containing "mill" AND "lane" in the address

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'

```

### Returns all accounts containing "mill" OR "lane" in the address

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'

```

### Returns all accounts that contain neither "mill" NOR "lane" in the address

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'

```

## [Score](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_executing_filters.html): indicates how relevant
 the document is, it's not always produced
### Return all accounts with balances between 20000 and 30000, inclusive.
#### "range" -> filters by range

```
curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}'

```

## [Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_executing_aggregations.html): group and
 extract statistics from the data. Similar to GROUP BY in SQL

```

curl -X GET "localhost:9200/'index'/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}'

```

## [Query and filter](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-filter-context.html)
### The behaviour of a query clause depends on whether it is used in\
### - Query: "How well does this document match this query?"
### - Filter: "Does this document match this query? Yes or No"

## [Match all](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl-match-all-query.html)
### Matches all documents, giving them all a score of 1.0
### The _score can be changed with the boost parameter

```

curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query": {
        "match_all": { "boost" : 1.2 }
    }
}'

```

### Matches no documents

```

curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query": {
        "match_none": {}
    }
}'

```

## [Full text queries](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/full-text-queries.html)
### Used for running full text queries on full text fields like the body of an email.

## [Match query](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl-match-query.html)
### 

```

curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query": {
        "match" : {
            "message" : {
                "query" : "to be or not to be",
                "operator" : "and",
                "zero_terms_query": "all",
                "cutoff_frequency" : 0.001
            }
        }
    }
}'

```

#### "operator"         -> "and"|"or", default "or"
#### "zero_terms_query" -> "all|none", default "none"
#### "cutoff_frequency" -> [0..1), high frequency terms are moved into an optional subquery. Operates on a per-shard-level

## [Match phrase](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl-match-query-phrase.html)
### TODO

## [Match phrase prefix](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl-match-query-phrase-prefix.html)
### TODO

## [Multi match](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl-multi-match-query.html)
### TODO
