

CONCEPTS
=========

Job			-		a related set of targets to scrape

Scrape		-		the act of collecting metrics

Label		-		prometheus stores metrics with labels, labels are key value pairs that identify the source system and other relevant information
					labels are used in queries to select metrics, filter metrics and modify metrics
					‘instance’ is a label that can indicate the source of the metrics
					relabeling refers to modifying metrics before they are stored in prometheus



COMPONENTS
=============

Runtime and Build information


Configuration


Rules


Targets



Service Discovery







TYPES OF METRICS
=================

Counter
Counters go up, and reset when the process restarts.


Gauge
Gauges can go up and down.


Summary
Summaries track the size and number of events.


Histogram
Histograms track the size and number of events in buckets. This allows for aggregatable calculation of quantiles.






PROMQL
========


TYPES

NOTE: a Vector is a set of related timeseries (timeseries - is a series that maps a timestamp to value)


Instant Vector
- set of timeseries
- each timeseries has a single sample
- all share the same timestamp

e.g. : node_cpu_seconds_total



Range Vector
- set of timeseries
- each timeseries contains a range of data points over time

e.g. : node_cpu_seconds_total[5m]




Scalar
- a simple numeric floating point value 

e.g. : -3.14



String
- a simple string value

e.g. : foobar




OPERATORS


arithmetic

binary comparison		-	>, <

logical binary

aggregation




REGEX with PromQL

(?i).*				-		zero or more characters, case insensitive





FUNCTIONS

count()
sum()
delta()





ALERTS


Alerting with Prometheus is split into two parts
- alerting rules
- an alert manager

