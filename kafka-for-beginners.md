# Kafka Administration for beginners


### Preparation
docker network rm bdp_kafka_network
docker network create --driver bridge bdp_kafka_network
docker network ls

docker run --name zookeeper.osecforum \
    -p 2181:2181 \
    --network=bdp_kafka_network \
    --hostname=zookeeper.meetup \
    -d \
    wurstmeister/zookeeper:latest

    docker run --name kafka.meetup \
    -p 9092:9092 \
    --env KAFKA_ZOOKEEPER_CONNECT=zookeeper.osecforum \
    --network=bdp_kafka_network \
    --hostname=kafka.meetup \
    -d \
    wurstmeister/kafka:1.0.1

## Kafka Commands

### Topic list
kafka-topics.sh --zookeeper zookeeper.meetup --list

### Create new topic
kafka-topics.sh --zookeeper zookeeper.meetup --create -topic warsaw


kafka-topics.sh --zookeeper zookeeper.meetup --create -topic warsaw --partitions 1 --replication-factor 1
```
Created topic warsaw.
```

### Produce message
kafka-console-producer.sh --broker-list kafka:9092 --topic warsaw


kafka-topics.sh --zookeeper zookeeper.meetup --alter --topic warsaw --partitions 9

### Read message 
cd /dir
kafka-run-class.sh kafka.tools.DumpLogSegments --print-data-log --deep-iteration --files 00000000000000000000.log
Dumping 00000000000000000000.log


### Performance test part 1
kafka-producer-perf-test.sh --print-metrics  --topic warsaw --num-records 1000000 --record-size 100 --throughput 15000000 --producer-props acks=1 bootstrap.servers=localhost:9092 buffer.memory=67108864 compression.type=none batch.size=8196

### Performance test part 2 
kafka-topics.sh --zookeeper zookeeper.meetup --alter --topic warsaw --partitions 9

kafka-producer-perf-test.sh --print-metrics  --topic warsaw --num-records 1000000 --record-size 100 --throughput 15000000 --producer-props acks=1 bootstrap.servers=localhost:9092 buffer.memory=67108864 compression.type=none batch.size=8196



!!! SPRAWDZIC CZY SA WPISY W HOSTS !!!

docker stop zookeeper.meetup
docker stop kafka.meetup

docker rm zookeeper.meetup
docker rm kafka.meetup

docker ps -a -f name=meetup

