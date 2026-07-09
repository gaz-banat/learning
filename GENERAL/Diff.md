

Think about it this way - how can i make first file like second file


Normal DIff

d - delete or more accurately remove
a - append or more accurately insert
c - change or more accurately d + a (remove and insert)

3d2 - at line number 3 in first file, delete it (you have now matched to
line 2 in second file)
5a5 - add line number 5 in first file, use line 5 in second file to do
this
4a4,5 - at line number 4 in first file add lines 4,5 from second file
6,8c5 - from line number 6 to 8 in first file, change it to what is line
number 5 in second file

\< *some texta* - the line in first file
\> *some textb* - the line in second file




Unified Diff

\-\-- healthcheck.lua.orig        2022-11-03 07:12:14.000000000 +1300
the first file
+++ healthcheck.lua.fix 2023-04-21 15:14:05.541567580 +1200 the second
file
@@ -930,7 +930,8 @@ This hunk is about starting at line 930 in first
file and counting 7 lines, starting at line 930 in second file and
counting 8 lines
       session, err = sock:tlshandshake({
         verify = self.checks.active.https_verify_certificate,
         client_cert = self.ssl_cert,
-        client_priv_key = self.ssl_key remove this line from first file
(it is not in the second file)
+        client_priv_key = self.ssl_key, add this line in first file (it
is in second file)
+        server_name = hostname add this line in first file (it is in
second file)
       })
     else
       session, err = sock:sslhandshake(nil, hostname,
