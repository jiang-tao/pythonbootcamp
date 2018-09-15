#Kafka Service

## Overview
The Confluent Kafka docker images are used to test the streaming pipeline.

## Start Kafka Service
docker-compose up -d

## Tierdown Kafka Service
docker-compose down

## Create Kafka Topics

```
docker-compose exec broker kafka-topics --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic users

docker-compose exec broker kafka-topics --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic pageviews

#list topics
docker-compose exec broker kafka-topics --list --zookeeper zookeeper:2181
```

## Create Sample Data
```
docker-compose run ksql-datagen ksql-datagen quickstart=pageviews format=delimited topic=pageviews maxInterval=100 \
propertiesFile=/etc/ksql/datagen.properties bootstrap-server=broker:9092

docker-compose run ksql-datagen ksql-datagen quickstart=users format=delimited topic=users maxInterval=100 \ propertiesFile=/etc/ksql/datagen.properties bootstrap-server=broker:9092
```

## Show Sample Data
```
docker-compose exec broker kafka-console-consumer --bootstrap-server localhost:9092 --topic users --from-beginning

docker-compose exec broker kafka-console-consumer --bootstrap-server localhost:9092 --topic users --offset 5 --partition 0
```

## Start KSQL CLI
```
docker-compose exec ksql-cli ksql http://ksql-server:8088

#show topics
ksql > show topics;

#show ksql tables
ksql > show tables;

#show ksql streams
ksql > show streams;

#select from stream
ksql > select * from PAGEVIEWS limit 5;

#create stream
ksql > CREATE STREAM pageviews (viewtime BIGINT, userid VARCHAR, pageid VARCHAR) WITH (KAFKA_TOPIC='pageviews', VALUE_FORMAT='DELIMITED');
ksql > CREATE STREAM pageviews_female AS SELECT users.userid AS userid, pageid, regionid, gender FROM pageviews LEFT JOIN users ON pageviews.userid = users.userid WHERE gender = 'FEMALE';

#create table
ksql > CREATE TABLE users1 (registertime BIGINT, gender VARCHAR, regionid VARCHAR, userid VARCHAR, interests array<VARCHAR>, contact_info map<VARCHAR, VARCHAR>) WITH (KAFKA_TOPIC='users', VALUE_FORMAT='JSON', KEY = 'userid');
ksql > CREATE TABLE pageviews_regions AS SELECT gender, regionid , COUNT(*) AS numusers FROM pageviews_female WINDOW TUMBLING (size 30 second) GROUP BY gender, regionid HAVING COUNT(*) > 1;
```

## Upload Schema
