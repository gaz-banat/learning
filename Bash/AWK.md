

AWK


number of fields in a record - \$NF
entire row - \$0
first field, second field, third field, etc - \$1, \$2, \$3, etc


Functions:

getline - move to the nextline
close




GENERAL FORMAT

cli:
awk \[options\] 'BEGIN { \<do something\> }; { \<do something\> }; END {
\<do something\> }'

awk file
BEGIN {
\<do something\>
}
{
\<do something\>
}
END {
\<do something\>
}


OPERATORS

! - logical not


SPECIAL VARIABLES

\_ - a hash variable


SYNTAX

Printing:
awk '{print \$1 "\\t\\t" \$2 }'


Using If:
awk '{ if (\<condition\>) { do this } }'



Pattern matching:

awk '/PATTERN/ { do this }'

awk '\$2\~/PATTERN/ {do this}'



SOLUTIONS

Read a bunch of certificates from a ca bundle:

awk -v cmd=\'openssl x509 -noout -subject\' \'/BEGIN/{close(cmd)};{print
\| cmd}\' \< /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem

