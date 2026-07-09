
CURL


basic authentication

-u 'username:password' curl will convert this to an "Authorization:
Basic xxxxxxxxxxxx" header



post JSON content

curl -i -X POST http://localhost:8001/services/test-service/routes 
-H \"Content-Type: application/json\" 
-d \'{\"name\": \"test-route\", \"paths\": \[ \"/path/one\",
\"/path/two\" \]}\'



post application/x-www-form-urlencoded

curl -i -X POST http://localhost:8001/services/test-service/routes 
-H "Content-Type: application/x-www-form-urlencoded" 
-d \"name=test-route\" 
-d \"paths\[1\]=/path/one\" 
-d \"paths\[2\]=/path/two\"




post multipart/form-data

curl -i -X POST http://localhost:8001/services/test-service/routes 
-H "Content-Type: multipart/form-data" 
-F \"name=test-route\" 
-F \"paths\[1\]=/path/one\" 
-F \"paths\[2\]=/path/two\" 
-F "config.access=@custom-auth.lua" \# this is sending a file

post text/plain

curl -X POST -H \"Content-Type: text/plain\" \--data \"this is raw
data\" http://78.41.xx.xx:7778/




WGET
