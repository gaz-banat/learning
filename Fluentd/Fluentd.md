
==============
FLUENTD
==============

PLUGINS

Fluentd has 6 types of plugins

Input - source of the data (e.g files, os logs like syslog, windows
event log
tail
Parser - incoming log entries can be parsed to convert from their raw
format to something more usable
a parser relies on a formatter
Filter - here you can modify the contents, add metadata or remove the
content
Output - filtered log entries can be forwarded to different outputs
(e.g. databases, managed services)
stdout
counter - just a count of the number of entries

Formatter - a specification for fluentd to tell it in what format is the
log entry
regex (then of course you specify the regex)
json
apache2, apache_error, nginx, syslog, csv, tsv, ltsv, json_lines,
multiline, none
Buffer




PROCESSING PIPELINE

Input \-\-\-\--\> Parser \-\-\-\--\> Filter \-\-\-\-\-\--\> Output





================
FLUENT-BIT
================



CONCEPTS

Event/Record - looks like a single log entry
Filtering - the process of altering, enriching or dropping events (e.g.
append an IP address or other metadata to the event)
Tag - a fluent-bit internal string assigned to an event
Timestamp - the time when an Event was created. It is a numeric
fractional integer in the form of SECONDS.NANOSECONDS
Match - a rule to select events based on a tag
Structured Message - defines content of events as keys and values


PIPELINE

Input ---\> Parser ---\> Filter ---\> Buffer ---\> Route ---\> Output


Parser - convert from unstructured/raw data to structured message

Filter - alter the data before delivering it to some destination
