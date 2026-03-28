CONCEPTS
             
topic - persistence for an event stream in kafka (an event stream at rest)  (a table in a db, a folder in a filesystem)
event/record -  synonymous with a message, for kafka this is an array of
bytes
batch   -  a collection of events/records produced to the same partition and topic
stream   -  related events in motion
retention -  for how long kafka should retain a message in a topic
partition -  a subset of a records(messages) in a topic
replica -  a copy of a partition
offset -                    incremental and immutable number that
indicates position of record in a partition
partition key -                    a value derived from an application
context (e.g. userid) to determine which parition to write data to
consumer group -                    consumers in the same group have the
same group-id value. consumer group concept ensures that a messae is
only ever read by a single consumer in the group
When a consumer group consumes the partitions of a topic, Kafka makes
sure that each partition is consumed by exactly one consumer in the
group

segment (aka log segment) - a section of a partition
at least one segment is active and that is the one being written to
(append only)

dirty log segment - a segment that has duplicate keys and is still not
compacted

clean log segment - no duplicate keys will be in that segment

each segment goes into its own log file




COMPONENTS


Brokers
group coordinator - a broker that acts on behalf of a consumer group to
store all the offsets for that consumer group


Producers
clientId - A logical grouping of clients with a meaningful name chosen
by the client application
is a string, is passed to the kafka server when making requests

Consumers
group - a group of consumers, kafka does not allow more than one
consumer in a consumer group to read from a partition, but a consumer in
a consumer group can read more than one partition
client id
consumer id
consumer id
consumer id
client id
consumer id
consumer id



Client API
Producer API
Consumer API




Topic
metadata
Partition - CG1-CON1
segment (active)
segment
segment
segment
Partition - CG1-CON2
segment (active)
segment
segment



Zookeeper


Kafka Streams

First what is a data stream - unbounded (no start no finish), ever
growing sequence of data in small packets
e.g. log entries, social media activites, traffic cam display, click
streams, transactions

- A Java/Scala Library
- Input data must be in Kafka topic
- ootb parallel processing, scalability, fault tolerance
- deploy anywhere




Kafka Connect

sits between data source and kafka
can work like an independent producer to read from data source and send
to broker (source connector) - Java classes SourceConnector and
SourceTask
can work like an independent consumer to read from broker and send to
data source (sink connector) - Java classes SinkConnector and SinkTask
based on KafkaConnect Framework

Worker
each KafkConnect instance is a worker in the KafkaConnect cluster
each worker has the same group id in the cluster
A worker (kafkaconnect) is actually a container that starts another
process (kafkaconnector, task)
worker is also responsible for talking to Kafka broker

Connector
Can be started with command line or API call
Responsible for defining and creating a list of tasks
source and sink connectors can run in the same KafkaConnect cluster
parallelism of tasks - some connectors have parallelism built into their
code, others need to be configured
Configuration eg for DB Connector - DB connection details, List of
Source Tables, Polling Frequency, Max Parallelism
Task
Started by worker
does the actual source or sink work with external system
does not talk to the Kafka cluster, the worker does that




FILES

node
server.properties - the main configuration when node is in KRaft running
as controller and broker

broker.properties - the main configuratoin when node is running as
broker only

controller.properties - the main configuration when node is running as
controller only

\<topic-dir\>/
\*.log - actual segment containing records up to a specific offset. The
name of the file defines the starting offset of the records in that log
\*.index - offset to position index, helps Kafka know what part of a
segment to read to find a message
an index that maps a logical offset (the record id) to the byte offset
of the record within the .log file
\*.timeindex - timestamp to offset index, allows Kafka to find messaged
with a specific timestamp
\*.txnindex
\*.snapshot - contains a snapshot of the producer state regarding
sequence IDs used to avoid duplicate records.
It is used when, after a new leader is elected, the preferred one comes
back and needs such a state in order to become leader again.





RECORD

Partition - 0 based index
Offset - 64 bit signed integer
Timestamp - Unix timestamp
Key - optional, not unique, array of bytes
Headers
Value - optional, array of bytes, the message or payload,


Tombstone
a record that has payload of 'null'. It will be cleaned up in
compaction





INTERNAL TOPICS

\_\_consumer_offsets - a list of consumers and their committed offsets
on various topics
each message/record/event is -
Key=\<consumer-group-id\>,\<topic\>,\<partition\> Value=\<int: next
offset to be read\>
transaction_state


OFFSETS

offset - is a position within a partition for the next message to be
sent to a consumer
current offset - the offset is a simple integer number that is used by
Kafka to maintain the current position of a consumer (Sent records)
This is used to avoid resending same records again to the same
consumer.
committed offset - is a pointer to the last record that a consumer has
successfully processed. For example, the consumer received 20 records.
It is processing them one by one, and after processing each record, it
is committing the offset. (Processed Records)

It is used to avoid resending same records to a new consumer in the
event of partition rebalance.


Auto Offset commits and At least once processing
There is a guarantee that each record will be processed at least once,
offsets are committed on no errors and wait timer expired automatically
Consumer must have idempotency

Manual Offset commits and At least once procssing
Auto commits are disabled, offset commit is manual

Consumer must have idempotency
There is a guarantee that each record will be processed at least once

Manual Offset commits and At most once processing
Auto commits are disabled, offset commit is manual
offset commits happens before processing
if there is an error while processing then tough luck, because offsets
are committed we cant get back the corresponding records from kafka

Manual Offset commits and Exactly once processing
Auto commits are disabled, offset commit is manual
Process a record, commit the offset, go to process next record and so
on
Producer to topic/partitions should have idempotency


CONSUMER GROUP REBALANCE
The leader of a consumer group is a consumer that is additionally
responsible for the partition assignment in a consumer group
topic partitions are reassigned between consumers of a consumer group
such that only one consumer can read from a partition

consumer processing gets blocked while kafka is rebalancing

https://medium.com/bakdata/solving-my-weird-kafka-rebalancing-problems-c05e99535435



GLOBAL CONFIGURATION

Configuration for Broker

broker.id - the id for the broker, e.g. 0, 1, 2
listeners - port at which kafka listens
log.dirs - log directory where partition data will be stored

Configuration for Topics

https://kafka.apache.org/documentation/#topicconfigs


Partition Level
by time, the first setting found in the following order is used:
log.retention.ms - time based reterntion/deletion of data in a
partition
once the configured retention time has been reached for a segment it is
marked for deletion or compaction depending on the configured policy
looks like this is done by comparing time difference between first
offset and last offset timestamps
log.retention.minutes
log.retention.hours

by size:
log.retention.bytes - largest amount of data a partition can hold before
it starts to delete data


Segment level
log.segment.ms - Max amount of time to wait to close an active segment
(default 7 days, 604800000) and create a new active segment
log.segment.bytes - Max size of a single metadata log file 1G
log.roll.ms - segments maximum age 168 hours or 7 days or 1 week

Retenttion, Compaction, Deletion
log.cleanup.policy - compact,delete

Cleaner
log.cleaner.threads - the number of background threads to use for log
cleaning, e.g. log compaction
log.cleaner.enable - i guess true or false to enable or disable log
cleaning
log.cleaner.backoff.ms - the amount of time to sleep when there are no
logs to clean (15 seconds)
log.cleaner.delete.retention.ms - Controls how long delete records and
transaction markers are retained after they are eligible for deletion
(default 24 hours)
log.cleaner.min.compaction.lag.ms - This can be used to prevent messages
newer than a minimum message age from being subject to compaction
if not set all log segments are eligible for compaction except for the
last (active) segment
log.clearner.max.compaction.lag.ms - the maximum amount of time a
message can go without being compacted
if not set, logs that do not exceed min.cleanable.dirty.ratio are not
compacted
not a hard guarantee - it is subject to availability of log cleaner
threads
and actual compaction time (monitor the uncleanable-partitions-count,
max-clean-time-secs, max-compaction-delay-secs metrics)

min.cleanable.dirty.ratio - the minimum ration of dirty log to total log
for a log to become eligible for cleaning 0.5




TOPIC LEVEL CONFIGURATION

general settings
compression.type
max.message.bytes
message.format.version
partitions
replicas

retention settings
before compaction or deletion is decided
retention.ms - once the configured retention time has been reached for a
message it is marked for deletion or compaction depending on the
configured policy (default is 7 days)
retention.bytes - total number of bytes allocated for messages for each
partition of the topic. If this value is exceeded for a partition then
the cleanup.policy will kick in.


if compacting:
cleanup.policy - compact only dirty segments that have been closed off
will be compacted
delete.retention.ms - the amount of time to retain delete tombstone
markers for log compacted topics
the amount of time to keep deleted records so that consumers can still
see them ????

if deleting:
cleanup.policy - delete an entire segment file will be deleted once all
messages in that segment file are older than retention.ms


segment settings (when to create a new segment):
segment.ms - the maximum age of messages in a segment file (i think this
is calculated by the time difference between latest message and earliest
message) segment.bytes - the maximum size of a log segment file


Compaction settings (cleanup.policy=compact)
min.cleanable.dirty.ratio
min.compaction.lag.ms - the minimum length of time that must pass after
a message is written before it could be compacted (lower bound on how
long each message will remain in the uncompacted head)
max.copaction.lag.ms - the maximum delay between the time a message is
written and the time the message becomes eligible for compaction








API's


Admin - Topics, Brokers, Other Objects
Producer
Consumer
Streams
Connect





SCHEMA REGISTRY






STRIMZI/AMQ-STREAMS


Operators

Cluster operator via Deployment and ConfigMap

Resources

Service Account, ClusterRole x 5, ClusterRoleBinding x 2, RoleBinding x
2,

CRDs/Types

Kafka

KafkaConnect
KafkaConnects2i

KafkaTopic

KafkaUser

KafkaMirrormaker

Kafkabridge

Kafkarebalance






KSQL

it is an SQL interface to Kafka Streams
runs in 2 modes -
interactive - good for dev environments
headless - good for production environments

has 3 components
KSQL Server
KSQL engine
REST Interface
KSQL Client (CLI/UI)

allows you to use a topic as a table and run SQL queries against those
topics
grouping and aggregating over kafkat topics
grouping and aggregating over a time window
