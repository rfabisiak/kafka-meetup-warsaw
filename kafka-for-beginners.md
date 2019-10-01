# Kafka Administration for beginners


### Docker

$ git clone https://github.com/wurstmeister/kafka-docker.git\
$ cd kafka-docker\
$ docker-compose up -d\
$ docker-compose top\
$ docker exec -ti kafka-docker_kafka_1 bash\

## Kafka Commands


### Topic list
kafka-topics.sh --zookeeper zookeeper  --list

### Create new topic
kafka-topics.sh --zookeeper zookeeper --create -topic warsaw
```
Missing required argument "[partitions]"
Option                                   Description
------                                   -----------
--alter                                  Alter the number of partitions,
                                           replica assignment, and/or
                                           configuration for the topic.    
```

kafka-topics.sh --zookeeper zookeeper --create -topic warsaw --partitions 1 --replication-factor 1
```
Created topic warsaw.
```

kafka-topics.sh --zookeeper zookeeper --list

kafka-topics.sh --zookeeper zookeeper --describe
```
Topic:warsaw	PartitionCount:1	ReplicationFactor:1	Configs:
	Topic: warsaw	Partition: 0	Leader: 1001	Replicas: 1001	Isr: 1001
```

### Produce message
kafka-console-producer.sh --broker-list kafka:9092 --topic warsaw
```
>warsaw
>kafka
>meetup
```

### Read message
cd /kafka/kafka-logs-*/
```
bash-4.4# ls
cleaner-offset-checkpoint         meta.properties                   replication-offset-checkpoint
log-start-offset-checkpoint       recovery-point-offset-checkpoint  warsaw-0
```

kafka-run-class.sh kafka.tools.DumpLogSegments --print-data-log --deep-iteration --files 00000000000000000000.log
```
Dumping 00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1569913518427 size: 74 magic: 2 compresscodec: NONE crc: 2753224873 isvalid: true
| offset: 0 CreateTime: 1569913518427 keysize: -1 valuesize: 6 sequence: -1 headerKeys: [] payload: warsaw
```


### Performance test part 1
kafka-producer-perf-test.sh --print-metrics  --topic warsaw --num-records 1000000 --record-size 100 --throughput 15000000 --producer-props acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 compression.type=none batch.size=8196
```
ops acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 compression.type=none batch.size=8196
826349 records sent, 165269.8 records/sec (15.76 MB/sec), 1844.7 ms avg latency, 2362.0 ms max latency.
1000000 records sent, 175654.312313 records/sec (16.75 MB/sec), 1902.11 ms avg latency, 2362.00 ms max latency, 1942 ms 50th, 2299 ms 95th, 2344 ms 99th, 2359 ms 99.9th.
```

### Performance test part 2
kafka-topics.sh --zookeeper zookeeper --alter --topic warsaw --partitions 9
```
WARNING: If partitions are increased for a topic that has a key, the partition logic or ordering of the messages will be affected
Adding partitions succeeded!
```
kafka-producer-perf-test.sh --print-metrics  --topic warsaw --num-records 1000000 --record-size 100 --throughput 15000000 --producer-props acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 compression.type=none batch.size=8196
```ps acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 compression.type=none batch.size=8196
1000000 records sent, 387596.899225 records/sec (36.96 MB/sec), 158.45 ms avg latency, 279.00 ms max latency, 167 ms 50th, 244 ms 95th, 254 ms 99th, 262 ms 99.9th.
```

### Kafka cluster 3 brokers
$ docker-compose scale kafka=3\
$ docker-compose top


### Zookeeper down
$ docker stop kafka-docker_zookeeper_1

bash-4.4#  kafka-topics.sh --zookeeper zookeeper --create -topic warsaw_test --partitions 1 --replication-factor 1
```
[2019-10-01 09:50:06,263] WARN Session 0x0 for server zookeeper:2181, unexpected error, closing socket connection and attempting reconnect (org.apache.zookeeper.ClientCnxn)
java.lang.IllegalArgumentException: Unable to canonicalize address zookeeper:2181 because it's not resolvable
	at org.apache.zookeeper.SaslServerPrincipal.getServerPrincipal(SaslServerPrincipal.java:65)
	at org.apache.zookeeper.SaslServerPrincipal.getServerPrincipal(SaslServerPrincipal.java:41)
	at org.apache.zookeeper.ClientCnxn$SendThread.startConnect(ClientCnxn.java:1001)
	at org.apache.zookeeper.ClientCnxn$SendThread.run(ClientCnxn.java:1060)
[2019-10-01 09:50:07,373] WARN Session 0x0 for server zookeeper:2181, unexpected error, closing socket connection and attempting reconnect (org.apache.zookeeper.ClientCnxn)
```

$ docker start kafka-docker_zookeeper_1
### Zookeeper CLI




### Kafka HA
$ docker exec -ti kafka-docker_kafka_1 bash

kafka-topics.sh --zookeeper zookeeper --create -topic meetup --partitions 3 --replication-factor 2
```Created topic meetup.```

kafka-topics.sh --zookeeper zookeeper --describe
```Topic:meetup	PartitionCount:3	ReplicationFactor:2	Configs:
	Topic: meetup	Partition: 0	Leader: 1003	Replicas: 1003,1001	Isr: 1003,1001
	Topic: meetup	Partition: 1	Leader: 1001	Replicas: 1001,1002	Isr: 1001,1002
	Topic: meetup	Partition: 2	Leader: 1002	Replicas: 1002,1003	Isr: 1002,1003
```

kafka-producer-perf-test.sh --print-metrics  --topic meetup --num-records 1000000 --record-size 100 --throughput 15000000 --producer-props acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 compression.type=none batch.size=8196

$ docker stop kafka-docker_kafka_1

$ docker exec -ti kafka-docker_kafka_2 bash
kafka-topics.sh --zookeeper zookeeper:2181 --describe
```Topic:meetup	PartitionCount:3	ReplicationFactor:2	Configs:
	Topic: meetup	Partition: 0	Leader: 1003	Replicas: 1003,1001	Isr: 1003
	Topic: meetup	Partition: 1	Leader: 1002	Replicas: 1001,1002	Isr: 1002
	Topic: meetup	Partition: 2	Leader: 1002	Replicas: 1002,1003	Isr: 1002,1003
Topic:warsaw	PartitionCount:9	ReplicationFactor:1	Configs:
	Topic: warsaw	Partition: 0	Leader: -1	Replicas: 1001	Isr: 1001
	Topic: warsaw	Partition: 1	Leader: -1	Replicas: 1001	Isr: 1001
	Topic: warsaw	Partition: 2	Leader: -1	Replicas: 1001	Isr: 1001
```
kafka-topics.sh --zookeeper zookeeper:2181 --describe --under-replicated-partitions
```Topic: meetup	Partition: 0	Leader: 1003	Replicas: 1003,1001	Isr: 1003
	Topic: meetup	Partition: 1	Leader: 1002	Replicas: 1001,1002	Isr: 1002
```
$ docker start kafka-docker_kafka_1

kafka-topics.sh --zookeeper zookeeper:2181 --describe

### Kafka consumer, lag

kafka-consumer-groups.sh --bootstrap-server kafka:9092  --list

kafka-console-consumer.sh  --bootstrap-server kafka:9092 --topic warsaw --from-beginning --consumer-property group.id=warsaw-meetup

kafka-consumer-groups.sh --bootstrap-server kafka:9092  --describe  --group warsaw-meetup
```
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                     HOST            CLIENT-ID
warsaw-meetup   warsaw          4          111111          111111          0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
warsaw-meetup   warsaw          5          111111          111111          0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
warsaw-meetup   warsaw          6          111111          111111          0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
warsaw-meetup   warsaw          7          111111          111111          0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
warsaw-meetup   warsaw          2          111111          111111          0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
warsaw-meetup   warsaw          3          111112          111112          0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
warsaw-meetup   warsaw          1          111111          111111          0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
warsaw-meetup   warsaw          8          111111          111111          0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
warsaw-meetup   warsaw          0          1111117         1111117        0               consumer-1-7898de52-6afb-46bb-9610-cb53db3fa598 /172.19.0.1     consumer-1
```
Stop kafka-console-consumer.sh

kafka-console-producer.sh --broker-list kafka:9092 --topic warsaw
Produce few messages

kafka-consumer-groups.sh --bootstrap-server kafka:9092  --describe  --group warsaw-meetup
```
Consumer group 'warsaw-meetup' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
warsaw-meetup   warsaw          4          111111          111111          0               -               -               -
warsaw-meetup   warsaw          5          111111          111112          1               -               -               -
warsaw-meetup   warsaw          6          111111          111111          0               -               -               -
warsaw-meetup   warsaw          7          111111          111111          0               -               -               -
warsaw-meetup   warsaw          2          111111          111112          1               -               -               -
warsaw-meetup   warsaw          3          111112          111112          0               -               -               -
warsaw-meetup   warsaw          1          111111          111111          0               -               -               -
warsaw-meetup   warsaw          8          111111          111111          0               -               -               -
warsaw-meetup   warsaw          0          1111117         1111118         1               -               -               -
```


logi kafki gdzie ich szukac
zookeeper id borkerow i topic
retention by time and size
consumer grupy, lag, high water mark, low watermark
skasowanie topicu

