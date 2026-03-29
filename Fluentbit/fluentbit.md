

# DATA FLOW

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