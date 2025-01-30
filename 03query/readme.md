```markdown
# Elasticsearch Query Guide

### Querying Elasticsearch with a GET Request

You can use a query to retrieve data from Elasticsearch by adding a request body into a GET HTTP request.

### Example Request

```http
GET https://127.0.0.1:9200/movies/_search
```

### Request Body

```json
{
  "query": {
    "match": {
      "title": "star"
    }
  }
}
```

Filters ask a question yes or no.
Queries return data.
Filters are better performant and are cacheable.

```http
GET https://127.0.0.1:9200/movies/_search
```

```json
{
  "query": {
    "bool": {
      "must": {"term": {"title": "trek"}},
      "filter": {"range": {"year": {"gte": 2010}}}
    }
  }
}
```

### To Perform a Phrase Search

```json
{
  "query": {
    "match_phrase": {
      "title": "trek"
    }
  }
}
```
```




