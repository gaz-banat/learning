
# TO DO
might need to look at tuning scrape intervals so that we're not storing too many chunks per block in the prometheus database.




# ARCHITECTURE COMPONENTS

Prometheus Server
    Service Discovery             -   in charge of interfacing with external API systems, such as EC2, Consul, K8s, etc. 
                                      It polls these systems and compiles a list of hosts which metrics can be fetched from
    Scraping                      -   takes the list of hosts from service discovery and collects metrics from those hosts
    TSDB storage                  -   Time Series Database storage
    HTTP Server                   -   Prometheus Web UI
    API

Exporters                         -   an application that knows how to pull metrics from different sources, such as databases or an operating system
                                      the exportes outputs metrics in a format that Prometheus can read.

Pushgateway                       -  Recieves metrics from targets (targets push to the gateway), prometheus server then pulls from the pushgateway

Grafana

AlertManager                      -  prometheus pushes alerts to alertmanager

API clients



# CONCEPTS

Alert

Target

Instance

Sample                  -   a single value at a point in time in a time series

Scrape              		-		the act of collecting metrics

Scrapeconfig
  Job                   -   a collection of targets/instances to scrape

PROMQL                  -   the language for prometheus queries

Rule Group
  Rule


Label		                -		prometheus stores metrics with labels, labels are key value pairs that identify the source system and other relevant information
					                  labels are used in queries to select metrics, filter metrics and modify metrics
					                  ‘instance’ is a label that can indicate the source of the metrics
					                  relabeling refers to modifying labels on metrics before they are stored in prometheus



Metric                  -   a group of timeseries data, meaning asking prometheus for a metric could return many timeseries records


Time series             -   a stream of timestamped values and associated labels (key, value) that are grouped under a common metric


Example:
metric1{label1=value1, label2=value2, label3=value3}     XY       @<timestampe>
metric1{label1=value1, label2=value2, label3=value4}     XZ       @<timestampe>
metric1{label1=value1, label2=value5, label3=value6}     YZ       @<timestampe>


the metric part is
metric1

the timeseries part is
{label1=value1, label2=value2, label3=value3}     XY       @<timestampe>
{label1=value1, label2=value2, label3=value4}     XZ       @<timestampe>
{label1=value1, label2=value5, label3=value6}     YZ       @<timestampe>



## Data Storage in TSDB

NOTE: Read again and again - https://palark.com/blog/prometheus-architecture-tsdb/

Write-Ahead Log (WAL) Entry (DISK): 
Upon collection, each scraped sample is initially written to the Write-Ahead Log (WAL). 
This log serves as a buffer, ensuring that data is safely stored before being fully integrated into the TSDB. 
The WAL entry mechanism is essential for data durability; even if the system crashes, the WAL contains all the data that hasn't been committed yet, allowing for recovery.

Memory-Mapped Chunks (DISK): 
After the samples are logged (into the WAL), they are stored on disk as memory-mapped files called Chunks. 
These chunks are compressed sets of samples, optimized for both read and write access as the data is directly mapped from the file into memory.
Memory mapping allows reading the data from the file as if it was in memory, so the whole file need not be loaded into memory to read the data.
Every time you get a sample, you compress it in flight and store the chunk instead of storing the raw samples.

Head Block (MEMORY):
The most recent samples are kept in an in-memory structure called the head block. 
This is where new writes happen actively and it provides quick access to the most recent data.

Block Storage (DISK): 
Once the head block reaches a certain size or fills up, older chunks are moved to disk as immutable blocks, freeing up memory for incoming samples. 
This persistence is called Head Compaction. 
Additionally, blocks on disk are also compacted to optimize storage. 
Smaller chunks are merged into larger, more compact blocks. 
This not only frees up memory but also improves query performance by reducing the number of files that need to be read. 
Each block includes an index for faster data retrieval later.








# CONFIGURATION

Runtime and Build information


Configuration


Rules


Targets



Service Discovery







# TYPES OF METRICS


Counter
Counters go up, and reset when the process restarts.


Gauge
Gauges can go up and down.


Summary
Summaries track the size and number of events.


Histogram
Histograms track the size and number of events in buckets. This allows for aggregatable calculation of quantiles.



## Some interesting metrics

up 
the list of targets scraped by prometheus



# ALERTMANAGER

## How Alertmanager works

Receives one or more alerts from prometheus
groups the alerts by some criteria (e.g. namespace)
      namespace=app1
        alert: alertname=alert1,label1=value1,label2=value2
each alert goes through routing 
each alert is put into the correct receivers bucket (for lack of a better term)
notification sent to the receiver with all the alerts in the receivers bucket



## DATA STRUCTURE PASSED TO NOTIFICATION TEMPLATE AND WEBHOOK PUSHES

Note: https://prometheus.io/docs/alerting/latest/notifications/#data-structures


Receiver
Status                          FIRING (if even one alert is firing), RESOLVED (no alerts are firing)
Alerts
  Alert
    Status
    Labels
    Annotations
    StartsAt
    EndsAt
    GeneratorURL
    Fingerprint
  Alert
  Alert  
  Firing()
  Resolved()
GroupLabels                     KV pairs
CommonLabels                    KV pairs  the labels common to all alerts
  alertname=<alertname>
CommonAnnotations
ExternalURL






# PROMQL


## VECTORS

Instant Vector (same timestamp)

node_cpu_seconds_total{}

            TIMESERIES                                    VALUE         TIMESTAMP
node_cpu_seconds_total{cpu="0", instance="server1"}       674478.07     March 3rd 08:05AM 
node_cpu_seconds_total{cpu="1", instance="server1"}       674626.76     March 3rd 08:05AM 
node_cpu_seconds_total{cpu="0", instance="server2"}       599112.10     March 3rd 08:05AM 
node_cpu_seconds_total{cpu="1", instance="server2"}       876541.15     March 3rd 08:05AM 



Range Vector (different timestamp)

node_cpu_seconds_total{instance="server1"}[3m]

            TIMESERIES                                    VALUE         TIMESTAMP
node_cpu_seconds_total{cpu="0", instance="server1"}       674478.07     March 3rd 08:05AM 
node_cpu_seconds_total{cpu="0", instance="server1"}       674626.76     March 3rd 08:04AM 
node_cpu_seconds_total{cpu="0", instance="server1"}       599112.10     March 3rd 08:03AM 
node_cpu_seconds_total{cpu="1", instance="server1"}       876541.15     March 3rd 08:05AM 
node_cpu_seconds_total{cpu="1", instance="server1"}       123456.78     March 3rd 08:04AM
node_cpu_seconds_total{cpu="1", instance="server1"}       975313.87     March 3rd 08:03AM


## TYPES OF METRICS

Counter
Counters go up, and reset when the process restarts.
e.g. network traffic on an interface, number of requests to an http endpoint


Gauge
Gauges can go up and down.
e.g. cpu utilization by a process


Histogram
Histograms track the size and number of events in buckets. This allows for aggregatable calculation of quantiles.
e.g. http requests - how many took <5ms, how many took <10ms, how many took <20ms, etc.
These metrics tend to have _bucket at the end of their name
e.g. coredns_dns_requests_size_bytes_bucket
Look for the le label (I THINK)


Summary
Summaries track the size and number of events.


## OPERANDS (EXPRESSIONS OR SUBEXPRESSIONS EVALUATE TO THESE )

Scalar
- a simple numeric floating point value 
e.g. : -3.14


String
- a simple string value
e.g. : foobar


Instant Vector (the most recent value)
- set of timeseries
- each timeseries has a single sample (value)
- all share the same timestamp
e.g. : node_cpu_seconds_total


Range Vector (values over a period of time)
- set of timeseries
- each timeseries contains a range of data points over time
e.g. : node_cpu_seconds_total[5m]


NOTE: a Vector is a set of related timeseries (timeseries - is a series that maps a timestamp to value)


## LABEL MATCHERS

=
!=
=~              -     node_filesystem_avail_bytes{device=~"/dev/sda.*"}
!~              -     node_filesystem_avail_bytes{mountpoint!~"/boot.*"}                Dont match mountpoints that start with /boot


## UNARY OPERATOR

- (unary minus)

NOTE: There is only one unary operator, the minus sign


## BINARY OPERATORS

operators are listed in precedence top before bottom and left before right

arithmetic      -       ^ * / % atan2 + -
comparison      -       == != <= < >= > 
logical/set     -       and, unless, or




## VECTOR MATCHING

The idea of vector matching is to take a left side vector and a right side vector of an operator and match the vectors using one of 3 relationships.
Then run an operation based on an operator
The matching produces a new vector


### Relationships

one to one        -       this is the easiest, the left side and the right side vectors must have unique entries that match, the unique matching entry goes into the resultant vector
many to one       -       each vector element on the one side can match with multiple vector elements on the many side
one to many       -       each vector element on the one side can match with multiple vector elements on the many side



### Group modifiers

group_left (<label list>)

is a modifier used in vector matching that handles many-to-one relationships between metrics
It is used when a single time series on the right side of an operation needs to be joined with multiple time series on the left side (with whatever operation is being used)

the left side vector is the many side
the right side vector is the one side

it ensures that labels, 'on' which the operation is done, from the "left" side are preserved and labels from the <label-list> on the "right" side are added to the result.
therefore <label list> contains additional labels from the one side to be included in the result vector

the one side (the right side) must return a unique set of values for the labels in the labels-list
meaning the value of each label itself (not the metric value!) should be unique across the vector on the right side


Explanation:

(vector1) on (label1, label2) group_left(label3) (vector2)

the lhs is the many side and the rhs is one side
for each entry on lhs and rhs that have the same values for "label1 AND label2"
create a new entry in the resulting time series and add label3 from rhs to that entry



group_right (<label list>)

is a modifier used in binary operations to specify that the results of a many-to-one or one-to-many relationship should preserve the labels from the right side of the expression




### Keywords

on (<label list>)           -     the matching operation needs to be on the list of labels

ignoring (<label list>)     -     the matching operation needs to ignore the list of labels






### Examples for group_left()

Example 1 - Succeeds

avg_over_time(volume_read_latency{volume="dummy"}[2d])
{cluster="umeng-aff300-01-02", svm="hasvm2", volume="dummy”, node="umeng-aff300-02", style="flexvol",  aggr="test1"} 					0
{cluster="umeng-aff300-01-02", svm="hasvm2", volume="dummy”, node="umeng-aff300-03", style="flexvol",  aggr="umeng_aff300_aggr2"}		0.012811504590593971


volume_labels{volume="dummy"}
volume_labels{cluster="umeng-aff300-01-02”, svm="hasvm2", volume="dummy”, node="umeng-aff300-02", style="flexvol", snapshot_policy="default", state="online",  svm_root="false", type="rw”, aggr="test1"}			1



avg_over_time(volume_read_latency{volume="dummy"}[2d]) * on(cluster, svm, volume) group_left(aggr) volume_labels{cluster="umeng-aff300-01-02"}
{cluster="umeng-aff300-01-02", svm="hasvm2", volume="dummy”, node="umeng-aff300-02", style="flexvol", aggr="test1"}					0
{cluster="umeng-aff300-01-02", svm="hasvm2", volume="dummy”, node="umeng-aff300-03", style="flexvol", aggr="test1"}					0.012827728530556734


——

Example 2 - Fails


avg_over_time(volume_read_latency{volume="dummy"}[10d])
{cluster="umeng-aff300-01-02", svm="hasvm2", volume="dummy”, node="umeng-aff300-02", style="flexvol", aggr="test1"}					0
{cluster="umeng-aff300-01-02", svm="hasvm2", volume="dummy”, node="umeng-aff300-02", style="flexvol", aggr="umeng_aff300_aggr2"}		0
{cluster="umeng-aff300-01-02", svm="hasvm2", volume="dummy”, node="umeng-aff300-03", style="flexvol", aggr="umeng_aff300_aggr2"}		0.005758743393763292



volume_labels{volume="dummy"}
volume_labels{cluster="umeng-aff300-01-02”, svm="hasvm2", volume="dummy”, node="umeng-aff300-02", style="flexvol", snapshot_policy="default", state="online",  svm_root="false", type="rw”, aggr="test1"}			1



avg_over_time(volume_read_latency{volume="dummy"}[10d]) * on(cluster, svm, volume) group_left(aggr) volume_labels{cluster="umeng-aff300-01-02"}

This query fails because the resulting vector is ambiguous. It's ambiguous because group_left(aggr) copies the aggr value from the right side into the resulting vector, which creates a label collision in the result.
In other words, when the left and right vectors are combined with group_left(aggr), the resulting vector would no longer be unique since the first two metrics are now identical:

{cluster="umeng-aff300-01-02", node="umeng-aff300-02", svm="hasvm2", volume="dummy”, aggr="test1"}
{cluster="umeng-aff300-01-02", node="umeng-aff300-02", svm="hasvm2", volume="dummy”, aggr="test1"}
{cluster="umeng-aff300-01-02", node="umeng-aff300-03", svm="hasvm2", volume="dummy”, aggr="test1" }




## AGGREGATION OPERATORS

Prometheus supports the following built-in aggregation operators that can be used to aggregate the elements of a single instant vector
resulting in a new vector of fewer elements with aggregated values

NOTE: dimensions just means the values of the timeseries

sum(v)                    -       calculate sum over dimensions
avg(v)                    -       calculate the arithmetic average over dimensions
min(v)                    -       select minimum over dimensions
max(v)                    -       select maximum over dimensions
bottomk(k, v)             -       smallest k elements by sample value
topk(k, v)                -       largest k elements by sample value
limitk(k, v)              -       sample k elements, experimental, must be enabled with --enable-feature=promql-experimental-functions
limit_ratio(r, v)         -       sample a pseudo-random ratio r of elements, experimental, must be enabled with --enable-feature=promql-experimental-functions
group(v)                  -       all values in the resulting vector are 1
count(v)                  -       count number of elements in the vector
count_values(l, v)        -       count number of elements with the same value
stddev(v)                 -       calculate population standard deviation over dimensions
stdvar(v)                 -       calculate population standard variance over dimensions
quantile(φ, v)            -       calculate φ-quantile (0 ≤ φ ≤ 1) over dimensions


Using by and without

sum by (<label>) (v)                -       group entries on <label>, so if <label> say has 4 unique values then the result will have 4 entries, and the sum for each entry will be shown
sum by (<label1>, <label2>) (v)     -       if <label1> and <label2> put together have say 6 unique combinations then the result will have 6 entries
sum without (<label>) (v)           -



## REGEX with PromQL

.*            -   zero or more characters
(?i).*				-		zero or more characters, case insensitive




## MODIFIERS

node_memory_active_bytes{instance="node1"} offset 5m                      -     value from 5 minutes ago
node_memory_active_bytes{instance="node1"} @1663265188                    -     value from a specific time based on unix timestamp
node_memory_active_bytes{instance="node1"}[2m] @1663265188 offset 5m      -     2 minutes worth of data 10 minutes before Sep 15 2022 6:06:28 GMT
 


## FUNCTIONS

count(<query>)
number of records in the record set resulting from query

rate()
avg number of 'something' over the time window


delta()
difference between first and last value


increase()


histogram_quantile()

query:
histogram_quantile(0.95, sum by (le) http_request_duration_seconds_bucket{instance="10.1.2.3"})

explanation
for the metric http_request_duration_seconds_bucket for instance 10.1.2.3
sum by the le label  (here sum is just working as a grouping on le label)
then retrieve the value for 95% of the requests






# PROMETHEUSRULES

groups:
  - name: group1
    rules:
      - expr: 
        labels:
        record:
      - expr:
        labels:
        record:
  - name: group2
    rules:
      - expr: 
        labels:
        record:




# ALERTS


Alerting with Prometheus is split into two parts
- alerting rules
- an alert manager




# TROUBLESHOOTING

Prometheus has queries built into itself to understand things about user queries

prometheus_rule_group_last_duration_seconds