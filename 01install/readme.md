
# Elasticsearch Installation & Usage via Docker

## Installation

### 1. Create a Docker Network
```sh
docker network create elastic
```

### 2. Run Elasticsearch Container
```sh
sudo docker run --name es01 --net elastic -p 9200:9200 -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:8.17.1
```
- The password for the `elastic` user will be displayed during startup.
- **Export the password for reuse:**
  ```sh
  export ELASTIC_PASSWORD="your_password"
  ```

### 3. Copy SSL Certificate
Copy the security certificate from the container to your local system:
```sh
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
```

## Accessing Elasticsearch
You can access Elasticsearch via:
- **REST API**
- **Programming languages (Python, Java, etc.)**
- **Web UI tools like Kibana**

### Reuse Existing Container
To reuse the same container after stopping it:
```sh
docker start es01
docker exec -it es01 /bin/bash
```

Alternatively, if using a different container:
```sh
docker run --net elastic -p 9200:9200 -it -m 1GB <container_id>
```

### Verify Elasticsearch is Running
```sh
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
```

---

# Elasticsearch Operations

## 1. Create an Index
```sh
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD \
    -H "Content-Type: application/json" \
    -X PUT 127.0.0.1:9200/shakespeare \
    --data-binary @shakes-mapping.json
```

## 2. Insert Data into an Index
```sh
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD \
    -H "Content-Type: application/json" \
    -X POST "https://localhost:9200/shakespeare/_bulk" \
    --data-binary @shakespeare_8.0.json
```

## 3. Query Elasticsearch
Example: Search for "to be or not to be" in the text field.
```sh
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD \
    -H "Content-Type: application/json" \
    -X GET "https://localhost:9200/shakespeare/_search?pretty" \
    -d '{
        "query": {
            "match_phrase": {
                "text_entry": "to be or not to be"
            }
        }
    }'
```

---

## Additional Resources
- [Elasticsearch Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Kibana](https://www.elastic.co/kibana) for visualization and management


