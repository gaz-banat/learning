
FIT THIS SOMEWHERE

checkpoint - WAL writer process flushes the WAL buffer to disk, this is
known as a checkpoint



NAMING CONVENTIONS


Common Name PostgreSQL Name

Table or Index Relation
Row Tuple
Column Attribute
Data Block Page (on disk)
Page Buffer (when block is in memory)




PAGE

Smallest unit of data storage in postgres
Every relation (table or index) is an array of pages of fixed size
It is not possible to have pages of different sizes in the same database
Default page size is 8 Kb
In a database all pages are logically equivalent and any row can be stored in any page


Page Layout
Page Header 24 bytes, page size, version, and availability of space
(free space pointers)
ItemIdData 4 bytes, pointer to Tuple
Free Space if any, the unallocated space in the page, ItemIdData is put
at the start of this space, new Tuples inserted from the end of the
space
Tuple a row of data
Special Index access method specific data, e.g.btree indexes. Different
methods store different data. Empty is ordinary tables





ENVIRONMENT VARIABLES

PGDATA - the data directory for postgresql
POSTGRES_PASSWORD





COMPONENTS


Processes:
Postmaster - supervisor process, first to get started, monitors and
starts/restarts other processes
as a listener it recieves new connection requests
responsible for authentication and authorization
spawns new process called Postgres for each new connection
Postgres x N

Stats Collector - responsible for collecting and reporting information
about server activity then update the information to optimizer
dictionary (pg_catalog)
Checkpointer - checkpoint is invoked every 5 minutes or when
max_wal_size value is exceeded. It syncs all the buffers from the shared
buffer area to the data files
does this by sending a signal to bgwriter
Wal (Write Ahead Log) Writer - Writes the WAL buffer to the WAL file
Auto Vacuum Cleaner - responsible to carry vacuum operations on bloated
tables (if enabled)
Logwriter\\Logger - Write the error message to the log file
Bgwriter\\Writer - periodically writes the dirty shared buffer to a data
file
Archiver - provides additional security by copying WAL files to a
specified directory


Memory Segments:
Shared Buffer - is considered dirty when it has data not committed to
data files
WAL Buffer - transaction log buffer,
contains metadata changes made to the actual data, sufficient to
reconstruct actual data during database recovery operations
contents of this buffer are written to WAL Files (wal segements or
checkpoint segments) by WAL writer
CLOG Buffer (Commit Log) - an area dedicated to holding commit log
pages
commit logs have commit status of all transactions and indicate whether
or not a transaction has been completed
Work Memory Buffer - memory reserved for a single sort or hash table
Maintenance Work Memory
Temp Buffers - these are used for access to temporary tables in a user
session during large sort and hash table


Disk Files:
Data files
WAL files
Log files - stderr, csvlog, syslog
Archive Logs



FILES

postgresql.conf
shared_buffer
wal_buffer
work_mem
maintenance_work_mem
temp_buffers


pg_hba.conf (host based authentication)


pg_ident.conf ident configuration file




STRUCTURE

Database Cluster
Database
Schema
Table
Table
Schema
Database




DATABASE CLUSTER

What
a collection of databases that is managed by a single instance on a
server


How
initdb -D /var/lib/pgsql/data
pg_ctl -D /var/lib/pgsql/data


Types of shutdown
Smart - disallow new connections, let current connections finish their
tasks then shutdown
Fast - disallow new connections, abort current connection transactions,
exit gracefully
Immediate - aborts without proper shutdown which leads to recovery from
WAL files on next startup



SCHEMA

What:
It is a namespace that contains named objects like tables, data types,
functions and operators
one database can have multiple schemas
schemas help with separation of data between different applications



PRIVILIGES

what:
- the right to execute a particular type of SQL statement
- the right to access and act on another user's object (table, view,
sequence)

types:
cluster level - granted by superuser (ALTER)
object level - granted by superuser or owner of object or role with
grant privileges (GRANT, REVOKE)







PG SYSTEM CATALOGS

postgres - a database that comes with PostgreSQL
pg_catalog - a schema to store metadata information about the database
and cluster (no manual intervention required!)
pg_database - a table to store general database info
pg_stat_database - a table that contains stats informatio of database
pg_tablespace - a table that contains tablespace information
pg_operator - a table that contains operator information
pg_avalable_extensions - a table to list all avalable extensions
pg_shadow - a table to list all users in the system
pg_timezone_names - a table to list available timezones
pg_locks - a table to list locks on database tables
pg_tables - a table to list all tables in current database and
pg_catalog schema
pg_settings - a table to list configuration information
pg_indexes - a table to display indexes
pg_views - a table to list all views


NOTE: pg_catalog schema can be accessed from any database



POSTGRESQL DATATYPES

Character types - char, varchar, text these types are collatable
Numeric types - integer, floating point
Boolean types
Temporal types - date, time, timestamp, interval
Array types
JSON
Special types - network address, geometric data
Other types - serial




OPERATORS


Comparision
Mathematical






CONSTRAINTS

what:
rules enforced on data columns on table

can be enforced at column level or table level


types:
NOT NULL
UNIQUE
PRIMARY
FOREIGN
CHECK - ensures that all values in a column satisfy certain conditions




OBJECTS

database
table/relation
sequence - a user-defined schema-bound object that generates a sequence
of integers based on a specified specification
view
function
index




TABLE INHERITANCE




TABLE PARTITIONING

Paritioning Methods
range
list



TABLESPACES

PostgreSQL stores data logically in tablespaces and physically in
datafiles (mapping a logical name to a physical location)
PostgreSQL uses a tablespace to map a logical name to a physical
location on disk
Tablespaces allow the user to control the disk layout of PostgreSQ
e.g. seperate indexes into one volume and tables into another volume
e.g. WAL files object on fast media and archive data on slow media

Default Tablespaces with PostgreSQL
pg_default - stores user data
default tablespace for template0 and template1 databases
pg_global - stores global data

Note: location of default tablespaces is data/ directory


Temporary Tablespaces:

these are created by PostgreSQL when it needs to hold large datasets
temporarily for completing a query, e.g. sorting
these do not store any data and there is no persistent file left when
the database is shutdown




INDEXES



VIEWS



FUNCTIONS

Aggregate - Avg(), Count(), Max(), Min(), Sum()
String - Chr, Concat, format, Initcap, Lower, Rtrim, Ltrim, Substring,
Upper
Date and Time - Age(Timestamp), now(), current_date, current_time,
current_timestamp, transaction_timestamp


Example custom function:

create or replace function on_insert() returns trigger as \$\$
begin
if(new.booking_date \>= date \'2022-01-01\' and new.booking_date \<=date
\'2022-01-31\') then
insert into jan_bookings values(new.\*);
elsif (new.booking_date \>= date \'2022-02-01\' and new.booking_date
\<=date \'2022-02-28\') then
insert into feb_bookings values(new.\*);
else
raise exception \'Enter valid booking date\';
end if;

return null;
end;
\$\$ LANGUAGE plpgsql;






TRIGGERS


Example custom trigger:

create trigger booking_entry before insert on bookings for each row
execute procedure on_insert();







COMMANDS

psql - work with
pg_ctl - looks like work with postgresql daemon
pg_stat_activity - a system view that allows to monitor the. databases
processes in real time
pg_controldata



PSQL COMMANDS

psql -V - get version of postgresql
psql -U \<user\> -W -d \<database\> - connect to a postgresql instance
with a specific database

\\? - help on psql commands
\\conninfo - show connection information
\\l - list databases

\\c \<database\> - connect to \<database\>
\\d - list tables (relations), views and sequences
\\d \<table\> - describe \<table\>

\\q - quit

show search_path; - show order in which schemas will be checked for a
query

show server_encoding; - show server encoding:

select now() as current; - show current time and date

select pg_postmaster_start_time(); - show uptime/last restarted


show all; - show all settings
select current_setting('\<setting-name\>') - show a setting, e.g. select
current_setting('max_parallel_workers');



copy a table with data:
create table \<new-table-name\> AS TABLE \<existing-table-name\>;

copy a table without data;
create table \<new-table-name\> AS TABLE \<existing-table-name\> WITH NO
DATA;

show filepath for a table

select pg_relation_filepath('\<table-name\>');


create a tablespace:
make a dir at a location, e.g. /var/lib/postgresql/ts_hrd

create tablespace hrd location \'/var/lib/postgresql/ts_hrd\';


create a temporary tablespace:
first - create tablespace \<tablespace-name\> owner \<owner-name\>
location '/path/in/file/system'
then - set temp_tablespaces=\<tablespace-name\> in postgresql.conf and
reload configuration
Note: creating a directory before hand is not required for temporary
tablespaces




SQL COMMANDS


select
insert
update
delete


select count(\*) from \<table\>; - get number of rows in table



TERMS

Identifier
Partition Key








PGBOUNCER

Pooling
Session pooling - when a client connects a server connection will be
assigned to it for the whole duration the client stays connected. When
the client disconnects the server connection will be put back into the
pool
Transaction pooling - a server connection is assigned to a client only
during a transaction. When pgbouncer notices the transaction is over the
server connection will be put back into the pool
Statement pooling - the server connection will be put back into the pool
immediately after a query completes.



===========================

[ ]{.underline}
[Connect to Psql]{.underline}

•       Connect to Specific Database with user and password

Syntax: psql -d database -U user --W (-d =Database, -U = User, -W =
Password)

•       Connect to Database on a different host/machine.

 Syntax: psql -h host -d database -U user --W

•       Connect using SSL Mode

   Syntax: psql -U user -h host \"dbname=db sslmode=require\"

[Psql Commands]{.underline}
[ ]{.underline}

•       Switch connection to a new database

 postgres=# \\c test1
 You are now connected to database \"test1\" as user \"postgres\".

•       List available databases

 postgres=# \\l

•       List available tables

 postgres=# \\ dt

•       List users and their roles(+ to get more info)

    postgres=# \\du

•       List available sequence(+ to get more info)

   postgres=# \\ds

•       Execute the previous command

    postgres=# \\g

•       Command history

    postgres=# \\s
 

•       Save Command History to file:

   postgres=# \\s filename

•       Get help on psql commands

    postgres=# \\?

•       Turn on\\off query execution time

    postgres=# *\\timing*

•       Edit statements in editor

   postgres=# \\e

•       Edit Functions in editor

   postgres=# \\ef

•       set output from non-aligned to aligned column output.

     postgres=# *\\a*

•         Formats output to HTML format.

     postgres=# *\\H*

•       Connection Information

    postgres=# *\\conninfo*

•       Quit psql

    postgres=# *\\q*

[Psql File Operations]{.underline}
[ ]{.underline}

•       Run sql statements from a file.

   psql -d test1 -U test1 -f test1.sql ( command line)

•       Send the output to a file.

   postgres=# **\\o \<filename\>

•       Save query buffer to filename.

   postgres=# **\\w filename

•       Turn off auto commit on session level

  \\set AUTOCOMMIT off
