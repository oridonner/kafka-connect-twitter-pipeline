# Start _docker_ containers  
Before starting up the cluster create a storage folder for _sqream storage_ inside local **data** folder:  
`docker run --rm -v $(pwd)/data:/mnt sqream:2.15-dev bash -c "./sqream/build/SqreamStorage -C -r /mnt/sqream_storage"`  

Create _Docker_ local network:   
`docker network create kafka-cluster`  
`docker-compose up`  
This command will start _sqreamd_, _zookeeper_, 2 _kafka_ brokers (broker-1, broker-2), _schema registry_, _kafka connect_.  

# Tests
First check status of the containers, executing this command:  
`docker-compose ps`  
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
     
List all subjects:  
`curl -X GET http://localhost:8081/subjects | jq`  

Delete subject:  
`curl -X DELETE http://localhost:8081/subjects/Kafka-key`  

### Test _kafka connect_
Check connector plugins:  
`curl localhost:8083/connector-plugins | jq`  
You shoud see _Twitter_ connector plugin among other built in connectors:  
> {  
    "class": "com.github.jcustenborder.kafka.connect.twitter.TwitterSourceConnector",  
    "type": "source",  
    "version": "0.2-SNAPSHOT"  
  },  
  
# Build Connectors
### Build _Twitter Source Connector_
Before starting the connector check existing topics:  
`docker run --net=kafka-cluster --rm confluentinc/cp-kafka:5.0.0 kafka-topics --zookeeper zookeeper:32181 --list`  

If **Twitter** topic exists, delete it:  
`docker run --net=kafka-cluster --rm confluentinc/cp-kafka:5.0.0 kafka-topics --zookeeper zookeeper:32181 --delete --topic twitter` 
 
 Create _Twitter Source Connector_ via REST API call to _kafka connect_ listens on 8083 port:
 `echo '{"name":"source-twitter","config":{"connector.class":"com.github.jcustenborder.kafka.connect.twitter.TwitterSourceConnector","tasks.max":"1","twitter.oauth.consumerKey":"WyBAPqB21h196UyENZATSnL3F","twitter.oauth.consumerSecret":"jQtSGi0tynigU51EkSfCNahqrBkHE18cH0xU2FttVzTKpNbcJO","twitter.oauth.accessToken":"908270057641463809-iju88vZOOTl2hiROMRA1XGLG1CnQPKI","twitter.oauth.accessTokenSecret":"r4L9oXix5wqoAD5GNIMtjRJOHuVO65mWLynhmmnD7sOW1","filter.keywords":"programming,java,kafka,scala","kafka.status.topic":"twitter","kafka.delete.topic":"twitter_del","process.deletes":"true"}}'| curl -X POST -d @- http://localhost:8083/connectors --header "content-Type:application/json"`  




