# Kafka

## Installation

    curl -o strimzi.zip https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.8.0/strimzi-0.8.0.zip
    unzip strimzi.zip
    cd strimzi-0.8.0
    sed -i 's/namespace: .*/namespace: myproject/' install/cluster-operator/*RoleBinding*.yaml
    oc apply -f strimzi-0.8.0/install/cluster-operator -n myproject
    oc apply -f strimzi-0.8.0/examples/templates/cluster-operator -n myproject
    oc apply -f strimzi-0.8.0/examples/kafka/kafka-ephemeral.yaml -n myproject
    oc apply -f strimzi-0.8.0/examples/kafka-connect/kafka-connect.yaml -n myproject

## Kafka Basics

### Get cluster kafka

    oc get Kafka
    oc describe kafka my-cluster

### Get cluster topics

#### Kafka way

    oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --list
    oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --describe

#### Openshift way

    oc get KafkaTopic
    oc describe kafkatopic connect-cluster-status

### Create topic

#### Kafka way

    oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --create --topic created-in-kafka --partitions 1 --replication-factor 1

#### Openshift way

    oc create -f strimzi-0.8.0/examples/topic/kafka-topic.yaml

### Adding partitions

    oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --alter --topic created-in-kafka --partitions 2

### Deleting topic

    oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --delete --topic created-in-kafka

### Producing and consuming messages

#### Producing Messages

    oc run --rm kafka-producer -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic

##### Producing using Kafkacat

    oc run --rm kafkacat -ti --image=ryane/kafkacat --restart=Never --command -- echo "foo" \| kafkacat -P -b my-cluster-kafka-bootstrap:9092 -t my-topic -p 0

#### Consuming Messages

    oc run --rm kafka-consumer -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning

##### Consuming using kafkacat

    oc run --rm kafkacat -ti --image=ryane/kafkacat --restart=Never -- kafkacat -C -b my-cluster-kafka-bootstrap:9092 -t my-topic

#### Reading again from de beginning

    oc run --rm kafka-consumer -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning

#### View consumer groups

    oc run --rm kafka-consumer -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --list

    oc run --rm kafka-consumer -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --describe

#### Checking Consumer position

    oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper localhost:21810 --group mygroup

#### Get last 3 messages

    oc run --rm kafkacat -ti --image=ryane/kafkacat --restart=Never -- -C -b my-cluster-kafka-bootstrap:9092 -t my-topic -p 0 -o -3 -e

### Ver kafka connect

    oc get KafkaConnect

### Ver kafka Mirror Maker

    oc get KafkaMirrorMaker

## Kafka Streams

### Create stream topic

    oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --create --topic streams-plaintext-input --partitions 1 --replication-factor 1

### Listen to this topic

    oc run --rm kafka-consumer -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic streams-plaintext-input --from-beginning

### Import file

    oc run --rm kafka-producer -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bash -c "echo -e 'all streams lead to kafka\nhello kafka streams\njoin kafka summit' > /tmp/file-input.txt && cat /tmp/file-input.txt | bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic streams-plaintext-input"

### Run stream class

    oc exec -it my-cluster-kafka-0 -c kafka -- bin/kafka-run-class.sh org.apache.kafka.streams.examples.wordcount.WordCountDemo

### Listen do new stream topic

    oc run --rm kafka-consumer-2 -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic streams-wordcount-output --from-beginning --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer

### Insert another word

    oc run --rm kafka-producer -ti --image=strimzi/kafka:0.8.0 --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic streams-plaintext-input

## IoT Demo

### Cloning repo

    git clone https://github.com/luszczynski/strimzi-lab.git && cd strimzi-lab

### Create iot-temperature and iot-temperature-max topics

    oc create -f strimzi-lab/iot-demo/stream-app/resources/topic-temperature-max.yml
    oc create -f strimzi-lab/iot-demo/stream-app/resources/topic-temperature.yml

### Create Consumer

    oc create -f strimzi-lab/iot-demo/consumer-app/resources/consumer-app.yml

### Create Stream App

    oc create -f strimzi-lab/iot-demo/stream-app/resources/stream-app.yml

### Create Device App

    oc create -f strimzi-lab/iot-demo/device-app/resources/device-app.yml
    oc scale deployment device-app --replicas=5

### Clean up

    oc delete all -l app=iot-demo