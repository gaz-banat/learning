


# COMPONENTS

Cluster Operator

Entity Operator
    Topic Operator

    User Operator

Access Operator         -       Manages and shares Kafka cluster connection details. It is deployed independently of the Cluster Operator.


strimzi pod set




# CONCEPTS

KRaft
KRaft (Kafka Raft metadata) mode replaces Kafka’s dependency on ZooKeeper for cluster management. KRaft mode simplifies the deployment and management of Kafka clusters by bringing metadata management and coordination of clusters into Kafka.



# RESOURCES


Kafka
KafkaNodePool
KafkaConnect
KafkaConnector
KafkaMirrorMaker2
KafkaBridge
KafkaRebalance

KafkaTopic

KafkaUser

KafkaAccess



# CONFIGURATION OPTIONS

Topic Operator and User Operator for managing topics and clients

Cruise Control for cluster rebalancing

Kafka Exporter for lag metrics

Listeners for authenticated client access

Data storage

Rack awareness





# COMMANDS

```shell
NAMESPACE=dev-kafka
CLUSTER=dev-kafka
IMAGE=aplregistry.aarnet.edu.au/dhi.io/strimzi-kafka:0.49.1-kafka-4.1.1
TOPIC=tiered-storage-test

# produce to a topic
kubectl -n ${NAMESPACE} run kafka-producer -it --image=${IMAGE} --rm=true --restart=Never \
-- bin/kafka-console-producer.sh --bootstrap-server ${CLUSTER}-kafka-bootstrap:9092 \
--topic ${TOPIC}

# perf test the kafka cluster using a producer
kubectl -n ${NAMESPACE} run kafka-producer-perf-test -it --image=${IMAGE} --rm=true --restart=Never \
-- bin/kafka-producer-perf-test.sh \
--topic ${TOPIC} --throughput 1000000000 --num-records 10 \
--producer-props acks=all bootstrap.servers=${CLUSTER}-kafka-bootstrap:9092 \
--record-size 1000

# consume from a topic
kubectl -n ${NAMESPACE} run kafka-consumer -it --image=${IMAGE} --rm=true --restart=Never \
-- bin/kafka-console-consumer.sh --bootstrap-server ${CLUSTER}-kafka-bootstrap:9092 \
--topic ${TOPIC} --from-beginning
```