
http protocol uses CR-LF for line endings


METHODS

GET

POST

PUT

PATCH

DELETE




HEADERS


Content-Type

application/java-archive
application/EDI-X12
application/EDIFACT
application/javascript (obsolete)
application/octet-stream
application/ogg
application/pdf
application/xhtml+xml
application/x-shockwave-flash
application/json
application/ld+json
application/xml
application/zip
application/x-www-form-urlencoded
audio/mpeg
audio/x-ms-wma
audio/vnd.rn-realaudio
audio/x-wav
image/gif
image/jpeg
image/png
image/tiff
image/vnd.microsoft.icon
image/x-icon
image/vnd.djvu
image/svg+xml
multipart/mixed
multipart/alternative
multipart/related (using by MHTML (HTML mail).)
multipart/form-data
text/css
text/csv
text/html
text/javascript
text/plain
text/xml
video/mpeg
video/mp4
video/quicktime
video/x-ms-wmv
video/x-msvideo
video/x-flv
video/webm
application/vnd.android.package-archive
application/vnd.oasis.opendocument.text
application/vnd.oasis.opendocument.spreadsheet
application/vnd.oasis.opendocument.presentation
application/vnd.oasis.opendocument.graphics
application/vnd.ms-excel
application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
application/vnd.ms-powerpoint
application/vnd.openxmlformats-officedocument.presentationml.presentation
application/msword
application/vnd.openxmlformats-officedocument.wordprocessingml.document
application/vnd.mozilla.xul+xml



Content-Range: - It can vary a bit, but each content-range header in a
series of requests tells the server at the destination where the
information in the request fits
with all the other requests that comprise the chunked file you're
sending






SOME CONTENT-TYPES:

application/octet-stream - its like i dont know what the content is but
it is binary.

This header is used for files where it needs to be opened using another
application, and likely, the title is going to help determine what
application it needs to be opened. 


application/x-www-form-urlencoded - This is used for sending simple text
values in a query string
Characters need to be URL encoded, e.g. space becomes %20 (meaning that
reserved and non-alphanumeric characters are replaced by '%HH, 2
hexadecimal digits representing the ASCII code)
the whole form data is sent as a long query string


Content-Type: application/x-www-form-urlencoded
e.g. key1=value1&key2=value2

In curl we use -H "Content-Type: application/x-www-form-urlencoded" -d
"key1=value1&key2=value2&key3=value3"


multipart/form-data - This content-type is for indicating to the server
'Hey, I'm sending you a bunch of files, and some of the information I'm
sending might be from a form.'
Then, in your request, you attach a bunch of files, and any form data
you wanted to send.
You can attach as many files as the server will permit, or as much data
as the server will permit.

Used for sending key value pairs
Data is sent in chunks to a server where the boundary is made by a
character which should not appear in the content
Often used when uploading files to a server


text/plain (aka Raw) - Technically this is any kind of string but
generally is XML or JSON
{\"firstName\":\"Pied\",\"lastName\":\"Piper\"}




NOTE:

Binary - application/octet-stream
GraphQL - application/json





WebHook

A webhook is an HTTP PUSH API that allows for a callback
