---

## 6. Mappings
- A **mapping** is a schema definition for an index.
- Elasticsearch provides default mappings, but they can be customized.

### Create a Custom Mapping
```sh
curl -XPUT 127.0.0.1:9200/movies -d '{
  "mappings": {
    "properties": {
      "year": {
        "type": "date"
      }
    }
  }
}'
```

### Using Postman
- Send a `PUT` request to `https://127.0.0.1:9200/movies`.
- Use **Basic Auth** with `username:password`.
- Request body:
```json
{
  "mappings": {
    "properties": {
      "year": {
        "type": "date"
      }
    }
  }
}
```

### Verify Mapping Creation
```sh
curl https://127.0.0.1:9200/movies/_mapping
```

### Common Mappings
#### Field Types
```json
"properties": {
  "user_id": {
    "type": "long"
  }
}
```
#### Field Indexing
```json
"properties": {
  "genre": {
    "index": "not_analyzed"
  }
}
```
#### Field Analyzer
```json
"properties": {
  "description": {
    "analyzer": "english"
  }
}
```

### Analyzer Capabilities
- **Character filter**: Removes HTML, converts `&` to `and`.
- **Tokenizer**: Splits strings on whitespaces or commas.
- **Token filter**: Lowercase, stemming, synonyms.

### Create a Document in an Index
```sh
curl -XPOST https://127.0.0.1:9200/movies/_doc/109487 -d '{
  "genre": ["IMAX", "Sci-fi"],
  "title": "Interstellar",
  "year": 2014
}'
```

### Fetch All Data
```sh
curl https://127.0.0.1:9200/movies/_search?pretty
```





### Import Many Documents
```sh
curl --cacert http_ca.crt -u elastic:q+X7Gzuj4tvOmDtSu6RK -H "Content-Type: application/json" -XPUT https://127.0.0.1:9200/_bulk?pretty --data-binary @movies.json
```

### Example Bulk Data
```json
{ "create" : { "_index" : "movies", "_id" : "135569" } }
{ "id": "135569", "title" : "Star Trek Beyond", "year":2016 , "genre":["Action", "Adventure", "Sci-Fi"] }
{ "create" : { "_index" : "movies", "_id" : "122886" } }
{ "id": "122886", "title" : "Star Wars: Episode VII - The Force Awakens", "year":2015 , "genre":["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "movies", "_id" : "109487" } }
{ "id": "109487", "title" : "Interstellar", "year":2014 , "genre":["Sci-Fi", "IMAX"] }
{ "create" : { "_index" : "movies", "_id" : "58559" } }
{ "id": "58559", "title" : "Dark Knight, The", "year":2008 , "genre":["Action", "Crime", "Drama", "IMAX"] }
{ "create" : { "_index" : "movies", "_id" : "1924" } }
{ "id": "1924", "title" : "Plan 9 from Outer Space", "year":1959 , "genre":["Horror", "Sci-Fi"] }
```




### Updating Documents

In Elasticsearch, you **cannot** update documents directly. Instead, Elasticsearch creates a new version of the document, increments its version number, and deletes the old version asynchronously.

#### Full Document Replacement
```sh
curl -X POST https://127.0.0.1:9200/movies/_doc/109487 -d '{
    "genre": ["IMAX", "Sci-Fi"],
    "title": "Inter fooerrrrr",
    "year": 2014
}'
```

#### Partial Update
```sh
curl -X POST https://127.0.0.1:9200/movies/_update/109487 -d '{
    "doc": {"genre": ["IMAX", "yoyo"]}
}'
```



### Deleting Documents

Make a delete request here
https://127.0.0.1:9200/movies/_doc/109487


### Concurrency

This is handled via sequence number and _primary_term use retry on conflict.


Searching

Use analyzers to define mappings 
If analyzer is keyword type then it is an exact match, i.e. case-sensitive as well.
If analyzer is text type then it is not exact match, you can add analyzers on top of it like case insensitiveness, stemming,
stopwords removal(it, and, the,...).


Create a new mapping:
A put request to https://127.0.0.1:9200/movies
{
    "mappings": {
        "properties": {
            "id": {"type": "integer"},
            "year": {"type": "date"},
            "genre": {"type": "keyword"},
            "title": 
            {
                "type": "text", 
                "analyzer": "english"
            }
        }
    }
}

Now, no analyzers will be run on genre field.
Title will have analyzers, giving english will give synonyms to english language.

Put data into it,

curl --cacert http_ca.crt -u elastic:FnPD3RrchiJgbfYi=kF3 -H "Content-Type: application/json" -XPUT https://127.0.0.1:9200/_bulk?pretty --data-binary @movies.json


Now try searching:
https://127.0.0.1:9200/movies/_search


{ 
    "query": {
        "match_phrase": {
            "genre": "sci"
        }
    }
}

This will return nothing
Because sci is not run with analyzers

But, 


This will return some results
{ 
    "query": {
        "match_phrase": {
            "genre": "Sci-Fi"
        }
    }
}

### Data Modeling
Sometimes, having data replication to reduce the number of queries can be a good trade-off. In some cases, denormalizing the data is beneficial.

### Parent-Child Mappings
#### Create a New Mapping
```sh
curl -X PUT "https://127.0.0.1:9200/series" -H "Content-Type: application/json" -d '{
    "mappings": {
        "properties": {
            "film_to_franchise": {
                "type": "join",
                "relations": {
                    "franchise": "film"
                }
            }
        }
    }
}'
```

Here, **franchise** is the parent, and **film** is the child.

#### Inserting Documents
```json
{ "create" : { "_index" : "series", "_id" : "1", "routing" : 1} }
{ "id": "1", "film_to_franchise": {"name": "franchise"}, "title" : "Star Wars" }
{ "create" : { "_index" : "series", "_id" : "260", "routing" : 1} }
{ "id": "260", "film_to_franchise": {"name": "film", "parent": "1"}, "title" : "Star Wars: Episode IV - A New Hope", "year":"1977", "genre":["Action", "Adventure", "Sci-Fi"] }
```

#### Retrieve All Films of a Franchise
```sh
curl -X GET "https://127.0.0.1:9200/series/_search" -H "Content-Type: application/json" -d '{
    "query": {
        "has_parent": {
            "parent_type": "franchise",
            "query": {
                "match": {
                    "title": "Star Wars"
                }
            }
        }
    }
}'
```

### Flattened Data Types
Flattened datatypes map inner fields under one parent flattened field.
```sh
curl -X PUT "http://127.0.0.1:9200/demo-flattened/_mapping" -H "Content-Type: application/json" -d '{
  "properties": {
    "host": {
      "type": "flattened"
    }
  }
}'
```

### Mapping Process
- **Explicit**: Define datatypes upfront.
- **Dynamic**: Elasticsearch infers datatypes automatically.
