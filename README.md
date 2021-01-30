# docker-elk-logstash
Config logs for docker containers

Based on the official Docker images from Elastic:

* [Elasticsearch](https://github.com/elastic/elasticsearch/tree/master/distribution/docker)
* [Logstash](https://github.com/elastic/logstash/tree/master/docker)
* [Kibana](https://github.com/elastic/kibana/tree/master/src/dev/build/tasks/os_packages/docker_generator)

### Bringing up the stack

Clone this repository onto the Docker host that will run the stack, then start services locally using Docker Compose:

```console
$ docker-compose up
```

You can also run all services in the background (detached mode) by adding the `-d` flag to the above command.

**:warning: You must rebuild the stack images with `docker-compose build` whenever you switch branch or update the
version of an already existing stack.**

If you are starting the stack for the very first time, please read the section below attentively.
## Initial setup
### set up log file path and config logstash
```yml
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.5.2
    volumes:
      - ./containers/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"

  logstash:
    image: docker.elastic.co/logstash/logstash:7.5.2
    command: -f /etc/logstash/conf.d/
    volumes:
      # mount config logstash files to container
      - ./containers/logstash/logstash.conf:/etc/logstash/conf.d/logstash.conf
      - ./containers/logstash/auth-logs.conf:/etc/logstash/conf.d/auth-logs.conf
      # mount logs file to container
      - /home/tinhnx/backend/logs:/home/logs
      - /home/tinhnx/backend-auth/logs:/home/logs/auth
    ports:
      - "5000:5000"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.5.2
    volumes:
      - ./containers/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
```
- config mount log file and config pipline logstash to logstash container
```yml
 volumes:
      # mount config logstash files to container
      - ./containers/logstash/logstash.conf:/etc/logstash/conf.d/logstash.conf
      - ./containers/logstash/auth-logs.conf:/etc/logstash/conf.d/auth-logs.conf
      # mount logs file to container
      - /home/tinhnx/backend/logs:/home/logs
      - /home/tinhnx/backend-auth/logs:/home/logs/auth
```
- config pipleine in .conf file

1. set input file with path logfile in container
```yml
   file {
    type => "java"
	# set path to file log in logstash container
    path => "/home/logs/app/application.log"
    start_position => "beginning"
  }
```

2. set index name (what will be used in kibana)
```yml
 elasticsearch{
    hosts => "elasticsearch:9200"
	# set index name to config index in kibana
    index => "backend_app"
  }
```

### add logstash pipeline in kibana
go to path http://localhost:5601
add index pattern 
![image](https://user-images.githubusercontent.com/26466129/106347510-94eafa80-62f1-11eb-9382-7161ff475991.png)
add index with name in (2. set index name )
![image](https://user-images.githubusercontent.com/26466129/106347546-f0b58380-62f1-11eb-873d-05395bd1e934.png)
view index and search or config time refresh internal
![image](https://user-images.githubusercontent.com/26466129/106347562-32462e80-62f2-11eb-97db-9252f69155bf.png)

## Reference
https://github.com/deviantony/docker-elk
