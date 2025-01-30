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


### Pagination

```json
{
  "from": 2,
  "size": 2,
  "query": {
    "match_phrase": {
      "title": "trek"
    }
  }
}
```


### Sorting
This is a little tricky because of storage in inverted index.
```http
GET https://127.0.0.1:9200/movies/_search?sort=year
```

Text fields are not optimized for sorting.

### Fuzzy searches

```json
{
  "query": {
    "fuzzy": {
      "title": {"value": "intersttellar", "fuzziness": 2}
    }
  }
}
```
```
1) For search as you type do search as above and use a slop value for between the 1.
2) n-grams
star is the word
unigram s t a r
bigram st ta ar
trigram sta tar
4gram star

Create an autocomplete analyzer
Create a mapping with custom analyzer
3) Completion suggestors
4) There is a datatype called search as you type -- this is better
```


