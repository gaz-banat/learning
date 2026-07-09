

PORTS

9200 - http api calls
9300 - custom binary protocol for communication between nodes in a
cluster





COMPONENTS

ElasticSearch

Index - stored in a set of shards,
    Type - '\_type' field is added to every document
        JSON Document with properties
        JSON Document with properties
        JSON Document with properties
        JSON Document with properties
    Type

    Type

Index


Index





FIELD TYPES

String
byte
short
long
integer
float
double
boolean
date
keyword
text
join




FIELD ANALYZERS

Standard
Whitespace
Simple
English





CRUD


Create:

Single - curl -X PUT -H "Content-type: application/json"
\<server\>:9200/\<index\>/\_doc/\<doc_id\> -d '\<data\>' OR
\--data-binary @\<file\>
Bulk - curl -X POST -H \"Content-type: application/json\"
'\<server\>:9200/\<index\>/\_bulk\' \--data-binary @\<file\> (Be mindful
of the data format in the file}
curl -X PUT -H "Content-type: applications/json"
\<server\>:9200/\_bulk?pretty \--data-binary @\<file\> (This needs a
create line before the actual document line)

Read:

curl -X GET -H "Content-type: application/json"
\<server\>:9200/\<index\>/\_doc/\<doc_id\>?pretty


Update:

Partial - curl -X POST -H "Content-type: application/json"
\<server\>:9200/\<index\>/\_doc/\<doc_id\>/\_update -d ' { "doc" : {
\<data\> } }' OR \--data-binary @\<file\>
Complete - curl -X PUT -H "Content-type: application/json"
\<server\>:9200/\<index\>/\_doc/\<doc_id\> -d \<data\> OR \--data-binary
@\<file\>

For concurrency - curl -X PUT -H "Content-type: application/json"
"\<server\>:9200/\<index\>/\_doc/\<doc_id\>?if_seq_no=\<seq_no\>&if_primary_term=\<primary_term\>"
-d '\<data\>' curl -X POST -H "Content-type: application/json"
"\<server\>:9200/\<index\>/\_doc/\<doc_id\>/\_update?retry_on_conflict=\<num_retries\>"
-d ' { "doc": { \<data\> } }'

Delete:

curl -X DELETE -H "Content-type: application/json"
\<server\>:9200/\<index\>/\_doc/\<doc_id\>





SEARCH
