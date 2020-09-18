Goal: Build fluentd service to forward application logs to ElasticSearch

### Steps

mkdir docker-compose-files/fluentd

Create docker-compose-files/fluentd/Dockerfile


```

FROM fluent/fluentd
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri", "--version", "4.0.8"]
```

###### Explain

```
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri", "--version", "4.0.8"]
```

We install fluent-plugin-elasticsearch to output logs to Elasticsearch

See output https://docs.fluentd.org/output

See all Fluentd output plugins here https://www.fluentd.org/plugins/all


#### Fluentd Conf

We need to instruct Fluentd how to handle data. To do it we need create Fluentd configuration file

```
mkdir docker-compose-files/fluentd/conf

touch docker-compose-files/fluentd/conf/fluent.conf
```

```
<source>
  @type forward
  port 24224
  bind 0.0.0.0
  @label @raw
</source>

<match pilgrim2go.**>
    @type copy  
    <store>
        # for debug (see /var/log/td-agent.log)
        type stdout
    </store>  
    <store>
        @type elasticsearch
        @log_level debug

        host elasticsearch
        port 9200
        user admin
        password admin
        scheme http
        ssl_verify false
        logstash_format true
        logstash_prefix pilgrim2go
        logstash_dateformat %Y%m%d
        include_tag_key true
        type_name access_log
        tag_key @log_name

        flush_interval 1s
    </store>
</match>    

```

Explaination

```
<source> (1)
  @type forward
  port 24224 (2)
  bind 0.0.0.0
  @label @raw
</source>
```  
(1) we use `input/forward` to listen to a TCP socket to receive the event stream.

See more https://docs.fluentd.org/input/forward

(2) Port to listen

With this port (`24224`), docker logs will forward like this

```
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: pilgrim2go.store-frontend        
```


Add Fluentd to docker-compose to build

```

  fluentd:
    image: elk_fluentd
    build: ./fluentd
    volumes:
      - ./fluentd/conf:/fluentd/etc
    ports:
      - "24224:24224"
      - "24224:24224/udp"

```

#### Test build

```
cd docker-compose-files
POSTGRES_USER=postgres POSTGRES_PASSWORD=postgres docker-compose -f pilgrim2go-docker-compose-fixed-instrumented-elk-v1.yml build fluentd

```