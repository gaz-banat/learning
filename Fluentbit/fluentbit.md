


# CONCEPTS

Event/Record - looks like a single log entry
Filtering - the process of altering, enriching or dropping events (e.g.
append an IP address or other metadata to the event)
Tag - a fluent-bit internal string assigned to an event
Timestamp - the time when an Event was created. It is a numeric
fractional integer in the form of SECONDS.NANOSECONDS
Match - a rule to select events based on a tag
Structured Message - defines content of events as keys and values





Parser - convert from unstructured/raw data to structured message

Filter - alter the data before delivering it to some destination


# DATA FLOW / PIPELINE

Input ------> Parser -------> Filter -------> Buffer ---------> Output1, Output2, Output3



## Input

1. tail

## Parser

1. json format
2. regex format


## Filter
implemented via a plugin
alter the collected data before delivering - modify, enrich or drop records



## Output

types:

tcp socket

stdout


formats:

json_lines




# CONCEPTS

Chunk


Buffering


Backpressure


http_client (DONT GET RID OF THIS IN THE FUTURE, IT IS A PROPER COMPONENT OF FLUENTD)