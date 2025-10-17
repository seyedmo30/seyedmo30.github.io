\c db

wal_level = logical

listen_addressess = "*" '*'


pg_hba.conf

host    sampledb    replicauser     192.168.10.10/32   scram-sha-256


CREATE PUBLICATION bookpub FOR TABLE book;

CREATE PUBLICATION my_publication FOR ALL TABLES;



CREATE SUBSCRIPTION booksub CONNECTION 'dbname=sampledb host=192.168.10.5 user=replicauser password=12345 port=5432' PUBLICATION bookpub



select * from pg_catalog.pg_publication;  

\dRp+




select * from pg_catalog.pg_subscription;

\dRs+



select * from pg_replication_slots ;

select pg_drop_replication_slot('booksub');


ALTER SUBSCRIPTION booksub DISABLE;
ALTER SUBSCRIPTION booksub SET (slot_name=NONE);
DROP SUBSCRIPTION booksub;


DROP publication bookpub ;


---------------------------------------------------------

postgresql.conf

wal_level = logical
max_replication_slots = 5
max_wal_senders = 10


SELECT * FROM pg_create_logical_replication_slot('replication_slot', 'pgoutput');


SELECT * FROM pg_replication_slots;


psql "dbname=postgres replication=database" -U postgres -h 192.168.13.248 -c "CREATE_REPLICATION_SLOT my_slot3 LOGICAL pgoutput;"

psql "dbname=phonebook replication=database" -U postgres -h 192.168.13.248 -c "IDENTIFY_SYSTEM;"


postgres://pglogrepl:secret@127.0.0.1/pglogrepl?replication=database


pg_receivewal (for physical replication)
https://www.postgresql.org/docs/current/app-pgreceivewal.html


 pg_recvlogical (for logical replication).
 https://www.postgresql.org/docs/current/app-pgrecvlogical.html
