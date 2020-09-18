Create `docker-compose-files/pilgrim2go-docker-compose-fixed-instrumented-elk-v1.yml` with following content


```
version: '3'
services:
  elasticsearch:
    image: 'docker.io/bitnami/elasticsearch:7-debian-10'
    environment:
      - ELASTICSEARCH_NODE_TYPE=ingest
    volumes:
      - 'elasticsearch_data:/bitnami'
  kibana:
    image: 'docker.io/bitnami/kibana:7-debian-10'
    ports:
      - '5601:5601'
    volumes:
      - 'kibana_data:/bitnami'
    depends_on:
      - elasticsearch
volumes:
  elasticsearch_data:
    driver: local
  kibana_data:
    driver: local      

```

### Setup Kibana 

`docker-compose -f pilgrim2go-docker-compose-fixed-instrumented-elk.yml up -d`

Open http://localhost:5601 to see it's working


### DB Service

add db service 

```
  db:
    image: postgres:11-alpine
    restart: always
    environment:
      - POSTGRES_PASSWORD
      - POSTGRES_USER
```


### Discount Service

add discount service 

```
  discounts:
    environment:
      - FLASK_APP=discounts.py
      - FLASK_DEBUG=1
      - POSTGRES_PASSWORD
      - POSTGRES_USER
      - DD_SERVICE=discounts-service
      - DD_AGENT_HOST=agent
      - DD_LOGS_INJECTION=true
      - DD_TRACE_ANALYTICS_ENABLED=true
      - DD_VERSION=1.1
    image: "pilgrim2go/ddtraining/discounts-service-fixed:latest"
    command: ddtrace-run flask run --port=5001 --host=0.0.0.0
    ports:
      - "5001:5001"
```

### Ads Service

add Advertisement service 

```
advertisements:
    environment:
      - FLASK_APP=ads.py
      - FLASK_DEBUG=1
      - POSTGRES_PASSWORD
      - POSTGRES_USER
      - DD_SERVICE=advertisements-service
      - DD_AGENT_HOST=agent
      - DD_LOGS_INJECTION=true
      - DD_TRACE_ANALYTICS_ENABLED=true
      - DD_VERSION=1.0
    image: "pilgrim2go/ddtraining/ads-service"
    command: ddtrace-run flask run --port=5002 --host=0.0.0.0
    ports:
      - "5002:5002"
    depends_on:
      - db

```      

Add Frontend Service

```
  frontend:
    environment:
      - DD_AGENT_HOST=agent
      - DD_LOGS_INJECTION=true
      - DD_TRACE_ANALYTICS_ENABLED=true
      - DD_SERVICE=store-frontend
      - DB_USERNAME
      - DB_PASSWORD
      - DD_VERSION=1.1
      - DD_CLIENT_TOKEN
      - DD_APPLICATION_ID
    image: "pilgrim2go/ddtraining/store-frontend-instrumented-fixed"
    command: sh docker-entrypoint.sh
    ports:
      - "3000:3000"
    depends_on:
      - db
      - discounts
      - advertisements

```

### Complete file


```
version: '3'
services:
  discounts:
    environment:
      - FLASK_APP=discounts.py
      - FLASK_DEBUG=1
      - POSTGRES_PASSWORD
      - POSTGRES_USER
      - DD_SERVICE=discounts-service
      - DD_AGENT_HOST=agent
      - DD_LOGS_INJECTION=true
      - DD_TRACE_ANALYTICS_ENABLED=true
      - DD_VERSION=1.1
    image: "pilgrim2go/ddtraining/discounts-service-fixed:latest"
    command: ddtrace-run flask run --port=5001 --host=0.0.0.0
    ports:
      - "5001:5001"
    depends_on:
      - db
  frontend:
    environment:
      - DD_AGENT_HOST=agent
      - DD_LOGS_INJECTION=true
      - DD_TRACE_ANALYTICS_ENABLED=true
      - DD_SERVICE=store-frontend
      - DB_USERNAME
      - DB_PASSWORD
      - DD_VERSION=1.1
      - DD_CLIENT_TOKEN
      - DD_APPLICATION_ID
    image: "pilgrim2go/ddtraining/store-frontend-instrumented-fixed"
    command: sh docker-entrypoint.sh
    ports:
      - "3000:3000"
    depends_on:
      - db
      - discounts
      - advertisements
  advertisements:
    environment:
      - FLASK_APP=ads.py
      - FLASK_DEBUG=1
      - POSTGRES_PASSWORD
      - POSTGRES_USER
      - DD_SERVICE=advertisements-service
      - DD_AGENT_HOST=agent
      - DD_LOGS_INJECTION=true
      - DD_TRACE_ANALYTICS_ENABLED=true
      - DD_VERSION=1.0
    image: "pilgrim2go/ddtraining/ads-service"
    command: ddtrace-run flask run --port=5002 --host=0.0.0.0
    ports:
      - "5002:5002"
    depends_on:
      - db
  db:
    image: postgres:11-alpine
    restart: always
    environment:
      - POSTGRES_PASSWORD
      - POSTGRES_USER

  elasticsearch:
    image: 'docker.io/bitnami/elasticsearch:7-debian-10'
    environment:
      - ELASTICSEARCH_NODE_TYPE=ingest
    volumes:
      - 'elasticsearch_data:/bitnami'
  kibana:
    image: 'docker.io/bitnami/kibana:7-debian-10'
    ports:
      - '5601:5601'
    volumes:
      - 'kibana_data:/bitnami'
    depends_on:
      - elasticsearch
volumes:
  elasticsearch_data:
    driver: local
  kibana_data:
    driver: local      

```

### Running

```

cd docker-compose-files
POSTGRES_USER=postgres POSTGRES_PASSWORD=postgres docker-compose -f pilgrim2go-docker-compose-fixed-instrumented-elk-v1.yml up -d

```


Open http://localhost:3000/ we can see the frontend is up

![storedog](https://github.com/DataDog/ecommerce-workshop/raw/master/images/storedog.png)
