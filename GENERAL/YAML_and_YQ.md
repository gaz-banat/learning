
YAML

\## A simple entry
key: value

key: \> \# the value can span multiple lines (multi-line string),
indentation and line breaks in value will be ignored, one line break
will be inserted at end (the value is a 'multi line folded scalar')
value

key: \> - \# same as above, but line break at end will be eliminated
value

key: \| \# the value can span multiple lines (multi-line string),
indentation and line breaks in value will be preserved, one line break
will be inserted at end (the value is a 'multi line literal scalar')
value

e.g.
shell: \|
cd /path/to/app/dir
./app_start

key: \|- \# same as above, but line break at end will be terminated
value


\## A simple list
list1:
- value1
- value2
- value3

list2: \[value1, value2, value3\]


\## A simple dictionary
dict1:
key1: value1
key2: value2


\## A list where each list item is a dictionary can be done in 2 ways
list1:
- key1: value1
key2: value2
- key1: value3
key2: value4

list1:
- { key1: value1, key2: value2 }
- { key1: value3, key2: value4 }



\## Anchors

The general principle of anchors
key : &anchor_name - define the anchor

key: \*anchor_name
\<\<: \*anchor_name - call the anchor


Example 1: anchor a simple value

name: &the_name John
age: 29
address: 29 Carlton St., Gore
hobbies:
- scuba diving
- cooking
- reading
self: \*the_name - when the yaml is parsed the key 'self' will get the
value 'John'


Example 2: anchor an object that has multiple keys

base_person: &base
city: Wellington
country: New Zealand

person:
\<\<: \*base - the base_person objects keys will show up here
name: Garfield
age: 31
isMale: true
likes: \[facebook, linked, twitter\]


==========

YQ


Get an entire yaml structure
yq -y . file - from file
yq -y . - - from stdin



Get the value of a specific key
yq -y '.key' file
yq -y '.key1.key2' file


Get the value of a specific key, but the key has dots or dashes in the
name
yq -y '.key1\["key2"\]' file - key2 has dots or dashes
yq -y '.\["key1"\]' file - the top level key key1 has dots or dashes


Strip quotes from the value of a key
yq -ry '.key1' file


Many yaml documents in a file, select only those documents where a key
has a specific value
yq -y 'select(.key == "value")' file



\# delete every entry in .metadata.annotations
yq -y  \'del(.metadata.annotations \| select(.))\' os-gui-service.yaml

\# take .metadata.annotations, convert its value to an internal map
using with_entries (to allow matching keys), select those keys that
pattern match \"opensearch-gui\"
yq -y \'.metadata.annotations \| with_entries(select(.key \|
test(\"opensearch-gui\")))\' os-gui-service.yaml

\# we are doing a delete
\# start with ..
\# pick the annotations map if it exists
\#
yq -y \'del(.. \| .annotations? \| select(.!= null) \| .\[keys \| .\[\]
\| select(startswith(\"io.rancher\"))\])\'
