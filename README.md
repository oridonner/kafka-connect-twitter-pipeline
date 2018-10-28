# Start _Kafka Cluster_
Before starting up the cluster create a storage folder for _sqream storage_ inside local **data** folder:  
`docker run --rm -v $(pwd)/data:/mnt sqream:2.15-dev bash -c "./sqream/build/SqreamStorage -C -r /mnt/sqream_storage"`  

Create _Docker_ local network:   
`docker network create kafka-cluster`  
`docker-compose up`  
This command will start _sqreamd_, _zookeeper_, 2 _kafka_ brokers (broker-1, broker-2), _schema registry_, _kafka connect_.  

# Tests
First check status of the containers:  
`docker-compose ps`  
You should get this results:  
>                   Name                               Command            State                           Ports                        
-------------------------------------------------------------------------------------------------------------------------------------  
twitter-sqream-pipeline_broker-1_1          /etc/confluent/docker/run   Up      0.0.0.0:9092->9092/tcp                               
twitter-sqream-pipeline_broker-2_1          /etc/confluent/docker/run   Up      9092/tcp, 0.0.0.0:9093->9093/tcp                     
twitter-sqream-pipeline_connect_1           /etc/confluent/docker/run   Up      0.0.0.0:8083->8083/tcp, 9092/tcp                     
twitter-sqream-pipeline_schema-registry_1   /etc/confluent/docker/run   Up      0.0.0.0:8081->8081/tcp                               
twitter-sqream-pipeline_sqreamd_1           ./sqream/build/sqreamd      Up      0.0.0.0:5000->5000/tcp                               
twitter-sqream-pipeline_zookeeper_1         /etc/confluent/docker/run   Up      2181/tcp, 2888/tcp, 0.0.0.0:2181->32181/tcp, 3888/tcp  

If all containers  are up and running start the following funcionality tests.  

### Test _Kafka Broker_
Test if _Kafka Broker_ is running by createing a topic:  
`docker run --net=kafka-cluster --rm confluentinc/cp-kafka:5.0.0 kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181`

List all existing topics:  
`docker run --net=kafka-cluster --rm confluentinc/cp-kafka:5.0.0 kafka-topics --list --zookeeper zookeeper:32181`  

Delete testing topic:  
`docker run --net=kafka-cluster --rm confluentinc/cp-kafka:5.0.0 kafka-topics --delete --topic foo --zookeeper zookeeper:32181`  


### Test _Schema Registry_
Code examples from [here](https://github.com/confluentinc/schema-registry#quickstart):  

Register a new version of a schema under the subject "Kafka-key"  
`curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" --data '{"schema": "{\"type\": \"string\"}"}' http://localhost:8081/subjects/Kafka-key/versions`  

Register a new version of a schema under the subject "Kafka-value":  
`curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" --data '{"schema": "{\"type\": \"string\"}"}' http://localhost:8081/subjects/Kafka-value/versions`  
     
List all subjects:  
`curl -X GET http://localhost:8081/subjects | jq`  


### Test _kafka connect_
Check connector plugins:  
`curl localhost:8083/connector-plugins | jq`  
You shoud see _Twitter_ connector plugin:  
> {  
    "class": "com.github.jcustenborder.kafka.connect.twitter.TwitterSourceConnector",  
    "type": "source",  
    "version": "0.2-SNAPSHOT"  
  },  
  
