
# Prerequisites
## Create Kafka-connect docker image
This step will add the debezium for oracle connector to the base kafka-connect image

```
cd kafka-connect
docker build -t cp-kafka-connect-dbz-ora:0.1 .
```
The new image is named: **cp-kafka-connect-dbz-ora:0.1**


## Create Oracle docker image

```
cd oracle/dockerfiles
./buildContainerImage.sh -v 19.3.0 -i -e
```

The new image is named: **oracle/database:19.3.0-ee**

# Start docker-compose

```
docker-compose up -d
```

# Prepare Oracle database
## SQL plus from oracle container
```
sqlplus sys/password@//localhost:1521/ORCLCDB as sysdba
sqlplus system/password@//localhost:1521/ORCLCDB
sqlplus pdbadmin/password@//localhost:1521/ORCLPDB1
```
## Prepare the DB for debezium
(Source: https://debezium.io/documentation/reference/stable/connectors/oracle.html)

From the Oracle container terminal:
```
mkdir opt/oracle/oradata/recovery_area
```

```
sqlplus

CONNECT sys/password AS SYSDBA
alter system set db_recovery_file_dest_size = 10G;
alter system set db_recovery_file_dest = '/opt/oracle/oradata/recovery_area' scope=spfile;
shutdown immediate
startup mount
alter database archivelog;
alter database open;
-- Should now "Database log mode: Archive Mode"
archive log list

ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;

-- Create user 'appuser01' and schema 'appuser01'
CREATE USER appuser01 IDENTIFIED BY password;

-- Grant privileges to 'appuser01'
GRANT CONNECT, RESOURCE, DBA TO appuser01;
GRANT CREATE SESSION TO appuser01;
GRANT UNLIMITED TABLESPACE TO appuser01;

exit;
```


### Creating the connector’s LogMiner user
```
sqlplus sys/password@//localhost:1521/ORCLCDB as sysdba
CREATE TABLESPACE logminer_tbs DATAFILE '/opt/oracle/oradata/ORCLCDB/logminer_tbs.dbf' SIZE 25M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;
exit;
```
```
sqlplus sys/password@//localhost:1521/ORCLPDB1 as sysdba
CREATE TABLESPACE logminer_tbs DATAFILE '/opt/oracle/oradata/ORCLCDB/ORCLPDB1/logminer_tbs.dbf' SIZE 25M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;
exit;
```
```
sqlplus sys/password@//localhost:1521/ORCLCDB as sysdba

CREATE USER c##dbzuser IDENTIFIED BY dbz DEFAULT TABLESPACE logminer_tbs QUOTA UNLIMITED ON logminer_tbs CONTAINER=ALL;

GRANT CREATE SESSION TO c##dbzuser CONTAINER=ALL;
GRANT SET CONTAINER TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$DATABASE to c##dbzuser CONTAINER=ALL;
GRANT FLASHBACK ANY TABLE TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ANY TABLE TO c##dbzuser CONTAINER=ALL;
GRANT SELECT_CATALOG_ROLE TO c##dbzuser CONTAINER=ALL;
GRANT EXECUTE_CATALOG_ROLE TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ANY TRANSACTION TO c##dbzuser CONTAINER=ALL;
GRANT LOGMINING TO c##dbzuser CONTAINER=ALL;

GRANT CREATE TABLE TO c##dbzuser CONTAINER=ALL;
GRANT LOCK ANY TABLE TO c##dbzuser CONTAINER=ALL;
GRANT CREATE SEQUENCE TO c##dbzuser CONTAINER=ALL;

GRANT EXECUTE ON DBMS_LOGMNR TO c##dbzuser CONTAINER=ALL;
GRANT EXECUTE ON DBMS_LOGMNR_D TO c##dbzuser CONTAINER=ALL;

GRANT SELECT ON V_$LOG TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$LOG_HISTORY TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$LOGMNR_LOGS TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$LOGMNR_CONTENTS TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$LOGMNR_PARAMETERS TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$LOGFILE TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$ARCHIVED_LOG TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$ARCHIVE_DEST_STATUS TO c##dbzuser CONTAINER=ALL;
GRANT SELECT ON V_$TRANSACTION TO c##dbzuser CONTAINER=ALL;

exit;
```

# Kafka
## Kafka-ui URL
http://localhost:8080/


## Deploy Debezium connector  for Oracle
```
curl --location --request POST 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data-raw '{
"name": "inventory-connector",
"config": {
"connector.class" : "io.debezium.connector.oracle.OracleConnector",
"tasks.max" : "1",
"database.server.name" : "server1",
"database.user" : "c##dbzuser",
"database.password" : "dbz",
"database.url": "jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS_LIST=(LOAD_BALANCE=OFF)(FAILOVER=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=oracle)(PORT=1521)))(CONNECT_DATA=SERVICE_NAME=)(SERVER=DEDICATED)))",
"database.dbname" : "ORCLCDB",
"database.pdb.name" : "ORCLPDB1",
"database.history.kafka.bootstrap.servers" : "kafka:9092",
"database.history.kafka.topic": "schema-changes.inventory"
}
}
'
'
```