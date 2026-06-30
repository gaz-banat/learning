
# COMPONENTS

## Processes:

Postmaster                      - supervisor process, first to get started, monitors and starts/restarts other processes 
                                  as a listener it recieves new connection requests
                                  responsible for authentication and authorization
                                  spawns new process called Postgres for each new connection

Postgres (x n)      

Stats Collector                 - responsible for collecting and reporting information about server activity then update the information to optimizer dictionary (pg_catalog)

Checkpointer                    - checkpoint is invoked every 5 minutes or when max_wal_size value is exceeded. It syncs all the buffers from the shared buffer area to the data files
                                  does this by sending a signal to bgwriter

WAL Writer                      - Writes the WAL buffer to the WAL file

Auto Vacuum Cleaner             - responsible to carry vacuum operations on bloated tables (if enabled)

Logwriter\Logger                - Write the error message to the log file

Bgwriter\Writer                 - periodically writes the dirty shared buffer to a data file

Archiver                        - provides additional security by copying WAL files to a specified directory


## Memory Segments:     
Shared Buffer                   - is considered dirty when it has data not committed to data files
WAL Buffer                      - transaction log buffer,
                                  contains metadata changes made to the actual data, sufficient to reconstruct actual data during database recovery operations
                                  contents of this buffer are written to WAL Files (wal segements or checkpoint segments) by WAL writer
CLOG Buffer (Commit Log)        - an area dedicated to holding commit log pages
                                  commit logs have commit status of all transactions and indicate whether or not a transaction has been completed
Work Memory Buffer              - memory reserved for a single sort or hash table
Maintenance Work Memory     
Temp Buffers                    - these are used for access to temporary tables in a user session during large sort and hash table


## Disk Files:
Data files
WAL files
Log files - stderr, csvlog, syslog
Archive Logs



# CONCEPTS


## NAMING CONVENTIONS

Common Name                         PostgreSQL Name

Table or Index                      Relation
Row                                 Tuple
Column                              Attribute
Data Block                          Page (on disk)
Page                                Buffer (when block is in memory)




## PAGE (BLOCK)

Smallest unit of data storage in postgres
Every relation (table or index) is an array of pages of fixed size (so i think each row is a page)
NOTE: tables are in heap memory, indexes are B+Tree type indexes
It is not possible to have pages of different sizes in the same database
Default page size is 8 KB
In a database all pages are logically equivalent and any row can be stored in any page


Page Layout
    Page Header         24 bytes, page size, version, and availability of space (free space pointers)
    ItemIdData          4 bytes, pointer to Tuple
    Free Space          if any, the unallocated space in the page, ItemIdData is put at the start of this space, new Tuples inserted from the end of the space
    Tuple               a row of data
    Special             Index access method specific data, e.g.btree indexes. Different methods store different data. Empty is ordinary tables


The page from what i understand is related to the main database file


Dirty page      -       the data in the page in memory is different to the data in the data file


# FILES

postgresql.conf
    shared_buffer
    wal_buffer
    work_mem
    maintenance_work_mem
    temp_buffers


/var/lib/pgsql/<optiona_path>/data/pg_hba.conf (host based authentication)


/var/lib/pgsql/<optiona_path>/data/pg_ident.conf ident configuration file


pg_wal

pg_xact         -       commit logs



# STRUCTURE

Database Cluster
    Database
        Schema
            Table
            Table
        Schema
    Database




# DATABASE CLUSTER

What
a collection of databases that is managed by a single instance on a server


How
initdb -D /var/lib/pgsql/data
pg_ctl -D /var/lib/pgsql/data


Types of shutdown
Smart           - disallow new connections, let current connections finish their tasks then shutdown
Fast            - disallow new connections, abort current connection transactions, exit gracefully
Immediate       - aborts without proper shutdown which leads to recovery from WAL files on next startup



# SCHEMA

What:
It is a namespace that contains named objects like tables, data types, functions and operators
one database can have multiple schemas
schemas help with separation of data between different applications



# PRIVILIGES

what:
- the right to execute a particular type of SQL statement
- the right to access and act on another user's object (table, view, sequence)

types:
cluster level       - granted by superuser (ALTER)
object level        - granted by superuser or owner of object or role with grant privileges (GRANT, REVOKE)



# WAL

The purpose of WAL is to ensure durability and crash recovery

## Transaction logs go to WAL

WAL record                  -   each WAL record is the description of a change to the main database
synchronous_commit=on       -   WAL Buffer is fsynced to WAL files and then client is informed that transaction was committed
synchronous_commit=off      -   client is informed of commit as soon as transactions are written to WAL buffer

I think 'COMMIT;' command may also be forcing WAL buffer to WAL file
sychronous_commit is on by default


Client ----- Transaction(INSERT,UPDATE,DELETE) ----> WAL Buffer (memory) ---- COMMIT (fsync())---> WAL Data files (pg_wal/ on disk)
                                                                                                         0000000C000000000000004C
                                                                                                         0000000C000000000000004D
                                                                                                         0000000C000000000000004E
                                                                                                         0000000C000000000000004F
                                                                                                         0000000C0000000000000050
                                                                                                         0000000C0000000000000051
                                                                                                        ...
Client <--------------- yes the transaction is committed ----------------- POSTGRESQL SYSTEM


The pgdata directory will most likely be at /var/lib/postgresql/data/pgdata OR /var/lib/postgresql/<version>/data/pgdata

each WAL file is a 16MB segment, writes are sequential

When is the WAL buffer written (fsync()) to WAL files?
This is dependent on a few things
- synchronous_commit setting
- the buffer is full
- the wal_writer_delay setting
- a checkpoint happens


## WAL buffer is written to data files because of a checkpoint

checkpoint - WAL writer process flushes the WAL buffer to data files on disk so that data files on disk reflect the information in the log
WAL buffer -----------> data files on disk


search for checkpoint settings in postgresql.conf


## WAL archiving

WAL archiving  - it is the process of copying WAL files that have been committed (aka completed) to a backup location

WAL archiving is the foundation of PITR (Point In Time Recovery)


wal_level = replica (or higher)     -   Ensures the logs contain enough information for archiving and replication.
archive_mode = on                   -   Enables the archiving subsystem.
archive_command:                    -   A shell command (e.g., cp %p /mnt/archive/%f) that PostgreSQL executes every time a WAL segment is filled.
                                        The placeholder %p is the path to the file, and %f is the filename.
archive_timeout:                    -   Forces a WAL segment to be archived after a set time (e.g., 60s), even if it isn't full, to limit potential data loss.


# Commit Log (CLOG)

The purpose of commit log is transaction visibility, 
i.e. should a row modified by one transaction be visible to other transactions (meaning transactions in other sessions)

It is a combination of the Transaction ID and a 2 bit field indicating the final status of the transaction

the commit log records the final status of each transaction. It tells PostgreSQL whether a transaction:
- committed (committed successfully)
- aborted (transaction was rolled back)
- in_progress (transaction is still in progress)
- sub-committed (transaction is committed but parent is not yet finished)


# HOW TRANSACTIONS WORK WITH WAL AND CLOG AND DATA FILES

Transaction performs inserts or updates.
WAL records are generated for these changes.
Somehow a commit occurs for the transaction - so a commit record is written to WAL and PostgreSQL flushes WAL buffer to WAL file.
The commit log is updated to mark the transaction as committed.
Other transactions can now see the committed data.

The above process keeps repeating for transactions

Then a checkpoint happens because either of
- checkpoint_timeout was reached
- max_wal_size was reached
- admin dude ran the 'CHECKPOINT;' command
- clean database shutdown
- online backup was started

checkpoint causes dirty pages to be written to data files

WAL is updated to mark the point at which WAL records are now in the database data file


# PG SYSTEM CATALOGS

postgres                        - a database that comes with PostgreSQL
    pg_catalog                  - a schema to store metadata information about the database and cluster (no manual intervention required!)
        pg_database             - a table to store general database info
        pg_stat_database        - a table that contains stats informatio of database
        pg_tablespace           - a table that contains tablespace information
        pg_operator             - a table that contains operator information
        pg_avalable_extensions  - a table to list all avalable extensions
        pg_shadow               - a table to list all users in the system
        pg_timezone_names       - a table to list available timezones
        pg_locks                - a table to list locks on database tables
        pg_tables               - a table to list all tables in current database and pg_catalog schema
        pg_settings             - a table to list configuration information
        pg_indexes              - a table to display indexes
        pg_views                - a table to list all views


NOTE: pg_catalog schema can be accessed from any database



# POSTGRESQL DATATYPES

Character types                 - char, varchar, text these types are collatable
Numeric types                   - integer, floating point
Boolean types
Temporal types                  - date, time, timestamp, interval
Array types
JSON
Special types                   - network address, geometric data
Other types                     - serial




# OPERATORS

Comparision
Mathematical






# CONSTRAINTS

what:
rules enforced on data columns on table
can be enforced at column level or table level


types:
NOT NULL
UNIQUE
PRIMARY
FOREIGN
CHECK           - ensures that all values in a column satisfy certain conditions




# OBJECTS

database
table/relation
sequence            - a user-defined schema-bound object that generates a sequence of integers based on a specified specification
view
function
index




# TABLE INHERITANCE




# TABLE PARTITIONING

Paritioning Methods:
range
list



# TABLESPACES

PostgreSQL stores data logically in tablespaces and physically in datafiles (mapping a logical name to a physical location)
PostgreSQL uses a tablespace to map a logical name to a physical location on disk
Tablespaces allow the user to control the disk layout of PostgreSQ
e.g. seperate indexes into one volume and tables into another volume
e.g. WAL files object on fast media and archive data on slow media

Default Tablespaces with PostgreSQL:
pg_default      - stores user data default tablespace for template0 and template1 databases
pg_global       - stores global data

Note: location of default tablespaces is data/ directory


Temporary Tablespaces:

these are created by PostgreSQL when it needs to hold large datasets temporarily for completing a query, e.g. sorting
these do not store any data and there is no persistent file left when the database is shutdown




# INDEXES



# VIEWS



# FUNCTIONS

Aggregate           - Avg(), Count(), Max(), Min(), Sum()
String              - Chr, Concat, format, Initcap, Lower, Rtrim, Ltrim, Substring, Upper
Date and Time       - Age(Timestamp), now(), current_date, current_time, current_timestamp, transaction_timestamp


Example custom function:
```shell
create or replace function on_insert() returns trigger as $$
begin
    if(new.booking_date \>= date \'2022-01-01\' and new.booking_date <=date '2022-01-31') then
        insert into jan_bookings values(new.\*);
    elsif (new.booking_date >= date '2022-02-01' and new.booking_date <=date '2022-02-28') then
        insert into feb_bookings values(new.*);
    else
        raise exception 'Enter valid booking date';
    end if;

    return null;
end;
$$ LANGUAGE plpgsql;
```





# TRIGGERS


Example custom trigger:

create trigger booking_entry before insert on bookings for each row execute procedure on_insert();


# VACUUM


## ENVIRONMENT VARIABLES

PGDATA                      - the data directory for postgresql
POSTGRES_PASSWORD
PGHOST                      - the database server host
PGPORT
PGUSER




# COMMANDS

psql                - work with postgresql databases
pg_ctl              - looks like work with postgresql daemon
pg_stat_activity    - a system view that allows to monitor the databases processes in real time
pg_controldata


```shell

# get version of postgresql
psql -V

# connect to a postgresql instance with a specific database
psql -h <host> -p <port> -U <user> -W -d <database> 
psql postgresql://<user>:<password>@<host>:<port>/<database>


# psql interactive menu
\?              - help on psql commands
\conninfo       - show connection information
\l              - list databases

\c <database>   - connect to <database>
\d              - list tables (relations), views and sequences
\d <table>      - describe <table>

\q - quit

## DISCOVERY
# show all settings
show all;

# show a setting
select current_setting('<setting-name>') 
e.g. select current_setting('max_parallel_workers');

# show order in which schemas will be checked for a query
show search_path; 

# show server encoding:
show server_encoding; 

# show current time and date
select now() as current; 

# show uptime/last restarted
select pg_postmaster_start_time(); 

# see the page/block size in memory
show block_size();

# show the data directory
show data_directory();





# copy a table with data
create table \<new-table-name\> AS TABLE \<existing-table-name\>;

# copy a table without data
create table <new-table-name> AS TABLE <existing-table-name> WITH NO DATA;

# show filepath for a table
select pg_relation_filepath('\<table-name\>');


# to create a tablespace
## make a dir at a location
e.g. /var/lib/postgresql/ts_hrd
## create tablespace hrd location 
'/var/lib/postgresql/ts_hrd';


# create a temporary tablespace:
## first - create tablespace 
create tablespace <tablespace-name> owner <owner-name> location '/path/in/file/system'
## then set tablespace in postgresql.conf
temp_tablespaces=<tablespace-name>
# reload configuration
systemctl restart <postgresql.service>

# Note: creating a directory before hand is not required for temporarytablespaces


# see the current WAL file
SELECT pg_walfile_name(pg_current_wal_lsn());


# take a backup using pg_dump
# NOTE: pg_dump does not cause a CHECKPOINT, rather it snapshots the underlying filesystem
# so there could be dirty pages in memory (meaning data in buffers not yet written to data files on disk)
pgdump -h <host> -p <port> <db_name> > <outfile>

# do a restore
psql -h <host> -p <port> <db_name> < <infile>       # database must already exist, users must already exist
```


## SQL COMMANDS

select
insert
update
delete


select count(\*) from \<table\>; - get number of rows in table



## TERMS

Identifier
Partition Key








# PGBOUNCER

Pooling
    
1. Session pooling
When a client connects a server connection will be assigned to it for the whole duration the client stays connected.
When the client disconnects the server connection will be put back into the pool

2. Transaction pooling
a server connection is assigned to a client only during a transaction. 
When pgbouncer notices the transaction is over the server connection will be put back into the pool

3. Statement pooling
the server connection will be put back into the pool immediately after a query completes.
