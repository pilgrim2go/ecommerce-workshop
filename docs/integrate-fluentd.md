With Fluentd service up and running we now can forward logs to Elasticsearch and search logs using Kibana


### Recap

Remember Fluentd Logs?

```
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

This one is a guide path to setup Fluend Logging

Let's start

Forward Logs of `frontend` to Fluentd

```
  frontend:
    image: "pilgrim2go/ddtraining/store-frontend-instrumented-fixed"
    command: sh docker-entrypoint.sh
    ports:
      - "3000:3000"
    .....
    .....
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: pilgrim2go.store-frontend
```        

Note: we need add tag `tag: pilgrim2go.store-frontend` so Fluentd can route event properly


Note: Make sure fluentd is included 

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

#### Testing 
