
JSON


Understanding JSON:

key-value structures - like python dictionaries
arrays (ordered list of values) - like python lists
key - can only be strings in double quotes
values - strings in double quotes, numbers, true, false, null, arrays,
objects


JSON data format:

Template
{
json-data
}


Simple key and values
{
"alpha": "beta",
"gama": "theta"
}


A json list/array
{
"fruits" : \[
"apple",
"banana",
"mango"
\]
}


A json dictionary
{
"fruits" : {
"apples": "5",
"bananas": "10",
"mangoes": "21"
}
}


===========

JQ


The Whole Document:

Get the structure of a document using its keys

jq -r \'\[paths \| join(\".\")\]\'





Keys:

Get all top level keys

jq \'. \|= keys' OR jq 'keys'

Get the value of a top level key
jq '. \| .key'

Get the key and value of a top level key
jq '. \| {key1, key2, .....}'

Same as above but top level structure is an array
jq '.\[ \] \| .key'
jq '.\[ \] \| {key1, key2 ...}'





Arrays:

Extract all keys of an array, keys is a keyword
jq '.array-name \| keys'

Get the value of a specific key
jq '.array-name\[ \] \| .key'

Get specific key and values from a named array based on key
jq '.array-name\[ \] \| {key1, key2, ...}'

Select every item that has a particular key from a named array
jq '.array-name\[ \]\|select(.key)'

Select an item from a named array when you know the exact value of the
key
jq '.array-name\[ \] \| select(.key == "value")'

Select a key from a named array with pattern matching
jq '.array-name\[ \] \| select(.key \| match("value"))

Select an item from a named array and only get a specific value in that
item
jq '.array-name\[ \] \| select(.key1 == "value") \| {key2}'

The above with multiple selects
jq '.array-name\[ \] \| select((.key1 == "value") and (.key2 ==
"value")) \| {key3}'

The above but you only know part of the value
jq '.array-name\[ \] \| select(.key1 \| contains("value"))'

The above with PCRE
jq '.array-name\[ \] \| select(.key1 \| test("value1\\s+value2"))'




