# AMQ Streams Demo

AMQ Streams Demo (Kafka on Openshift)

## Pre-req

You need a kafka cluster running on Openshift.

## Kafka CLI

### Kafka Topics

#### Create Topic

Creating a topic `first_topic`

```bash
oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --topic first_topic --create --partitions 3 --replication-factor 3
```

#### List Topics

List all topics

```bash
oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --list
```

#### Describe Topic

Describe topic `first_topic`

```bash
oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --topic first_topic --describe
```

#### Delete Topic

Creating a topic to be deleted

```bash
oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --topic second_topic --create --partitions 3 --replication-factor 3
```

List the topics

```bash
oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --list
```

Deleting the topic

```bash
oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --topic second_topic --delete
```

### Producers

#### Producing messages

```bash
oc run kafka-producer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic first_topic
```

#### Producing messages with properties

```bash
oc run kafka-producer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic first_topic --producer-property acks=all
```

#### Producing messages to an non-existing topic

```bash
oc run kafka-producer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic new-topic
```

### Consumer

#### Consuming from the end

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic
```

Open another terminal, and run:

```bash
oc run kafka-producer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic first_topic
```

#### Consuming from the beginning

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic --from-beginning
```

#### Consuming using consumer groups

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic --group my-first-application
```

Open another terminal, and run the producer:

```bash
oc run kafka-producer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic first_topic
```

Using another terminal, open another consumer

```bash
oc run kafka-consumer2 -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic --group my-first-application
```

> Note when there are 2 consumers, messages are load balanced between them. Messages will be load balanced based on the number of partitions

```bash
oc run kafka-consumer3 -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic --group my-first-application
```

#### Consuming using offset commits

Run the producer:

```bash
oc run kafka-producer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic first_topic
```

Run the consumer with a new consumer group:

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic --group my-second-application --from-beginning
```

When we run the consumer, we will see all messages from topic `first_topic`. Now, stop the consumer and run it again:

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic --group my-second-application --from-beginning
```

Even though we have specified `--from-beginning` the consumer won't show all the messages again. This happens because the offsets have been commit to kafka for group `my-second-application`.

#### Consumer groups cli

##### List Consumer Groups

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --list
```

##### Describe Consumer Groups

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --describe --group my-first-application
```

Now run a consumer:

```bash
oc run kafka-consumer2 -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic --group my-first-application
```

And execute again the describe command:

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --describe --group my-first-application
```

### Resetting Offsets

#### Reset to the earliest

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092  --group my-first-application --reset-offsets --to-earliest --execute --topic first_topic
```

Now execute the consumer:

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic first_topic --group my-first-application
```

Then you will see all messages from topic `first_topic`.

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --describe --group my-first-application
```

#### Reset to shift by

Forward:

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092  --group my-first-application --reset-offsets --shift-by 2 --execute --topic first_topic
```

Backward:

```bash
oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-consumer-groups.sh --bootstrap-server my-cluster-kafka-bootstrap:9092  --group my-first-application --reset-offsets --shift-by -2 --execute --topic first_topic
```

## Kafka Java Programming

### Producer Hello World

### Producer Quarkus

#### Creating topic

```bash
oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/kafka-topics.sh --zookeeper localhost:21810 --topic prices --create --partitions 3 --replication-factor 3
```

#### Creating our application

```bash
mkdir /tmp/demo && cd /tmp/demo

mvn io.quarkus:quarkus-maven-plugin:0.15.0:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=kafka-quickstart \
    -Dextensions="reactive-kafka,io.quarkus:quarkus-vertx"

code .

mkdir -p src/main/java/org/acme/kafka/quickstart
```

```java
cat <<EOF > src/main/java/org/acme/kafka/quickstart/PriceGenerator.java
package org.acme.kafka.quickstart;

import io.reactivex.Flowable;
import org.eclipse.microprofile.reactive.messaging.Outgoing;

import javax.enterprise.context.ApplicationScoped;
import java.util.Random;
import java.util.concurrent.TimeUnit;

/**
 * A bean producing random prices every 5 seconds.
 * The prices are written to a Kafka topic (prices). The Kafka configuration is specified in the application configuration.
 */
@ApplicationScoped
public class PriceGenerator {

    private Random random = new Random();

    @Outgoing("generated-price")                        // 
    public Flowable<Integer> generate() {               // 
        return Flowable.interval(5, TimeUnit.SECONDS)
                .map(tick -> random.nextInt(100));
    }

}
EOF
```


```java
cat <<EOF > src/main/java/org/acme/kafka/quickstart/PriceConverter.java
package org.acme.kafka.quickstart;

import io.smallrye.reactive.messaging.annotations.Broadcast;
import org.eclipse.microprofile.reactive.messaging.Incoming;
import org.eclipse.microprofile.reactive.messaging.Outgoing;

import javax.enterprise.context.ApplicationScoped;

/**
 * A bean consuming data from the "prices" Kafka topic and applying some conversion.
 * The result is pushed to the "my-data-stream" stream which is an in-memory stream.
 */
@ApplicationScoped
public class PriceConverter {

    private static final double CONVERSION_RATE = 0.88;

    @Incoming("prices")                                 // 
    @Outgoing("my-data-stream")                         // 
    @Broadcast                                          // 
    public double process(int priceInUsd) {
        return priceInUsd * CONVERSION_RATE;
    }

}
EOF
```

```java
cat <<EOF > src/main/java/org/acme/kafka/quickstart/PriceResource.java
package org.acme.kafka.quickstart;

import io.smallrye.reactive.messaging.annotations.Stream;
import org.reactivestreams.Publisher;

import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

/**
 * A simple resource retrieving the in-memory "my-data-stream" and sending the items to a server sent event.
 */
@Path("/prices")
public class PriceResource {

    @Inject
    @Stream("my-data-stream") Publisher<Double> prices; // 

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "hello";
    }

    @GET
    @Path("/stream")
    @Produces(MediaType.SERVER_SENT_EVENTS)             // 
    public Publisher<Double> stream() {                 // 
        return prices;
    }
}
EOF
```

```properties
cat <<EOF > src/main/resources/application.properties
# Configure the Kafka sink (we write to it)
smallrye.messaging.sink.generated-price.type=io.smallrye.reactive.messaging.kafka.Kafka
smallrye.messaging.sink.generated-price.topic=prices
smallrye.messaging.sink.generated-price.bootstrap.servers=my-cluster-kafka-bootstrap:9092
smallrye.messaging.sink.generated-price.key.serializer=org.apache.kafka.common.serialization.StringSerializer
smallrye.messaging.sink.generated-price.value.serializer=org.apache.kafka.common.serialization.IntegerSerializer
smallrye.messaging.sink.generated-price.acks=1

# Configure the Kafka source (we read from it)
smallrye.messaging.source.prices.type=io.smallrye.reactive.messaging.kafka.Kafka
smallrye.messaging.source.prices.topic=prices
smallrye.messaging.source.prices.bootstrap.servers=my-cluster-kafka-bootstrap:9092
smallrye.messaging.source.prices.key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
smallrye.messaging.source.prices.value.deserializer=org.apache.kafka.common.serialization.IntegerDeserializer
smallrye.messaging.source.prices.group.id=my-group-id
EOF
```

```html
cat <<EOF > src/main/resources/META-INF/resources/prices.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Prices</title>

    <link rel="stylesheet" type="text/css"
          href="https://cdnjs.cloudflare.com/ajax/libs/patternfly/3.24.0/css/patternfly.min.css">
    <link rel="stylesheet" type="text/css"
          href="https://cdnjs.cloudflare.com/ajax/libs/patternfly/3.24.0/css/patternfly-additions.min.css">
</head>
<body>
<div class="container">

    <h2>Last price</h2>
    <div class="row">
    <p class="col-md-12">The last price is <strong><span id="content">N/A</span>&nbsp;&euro;</strong>.</p>
    </div>
</div>
</body>
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script>
    var source = new EventSource("/prices/stream");
    source.onmessage = function (event) {
        document.getElementById("content").innerHTML = event.data;
    };
</script>
</html>
EOF
```

Now execute the application:

```bash
./mvnw package -DskipTests
```

And deploy to Openshift

```bash
# Import image stream
oc import-image redhat-openjdk-18/openjdk18-openshift --from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift --confirm -n myproject

# Create build 
oc new-build --name=quarkus-kafka \
   --image-stream=openjdk18-openshift \
   --env="JAVA_APP_JAR=kafka-quickstart-1.0-SNAPSHOT-runner.jar" \
   --binary=true -n myproject

# Start Build
oc start-build quarkus-kafka --from-file=./target -n myproject

# Create app
oc new-app quarkus-kafka -n myproject

# Expose
oc expose svc/quarkus-kafka -n myproject

# Open now this url
oc get --no-headers route -n myproject | awk '{ print $2}'
```