
# Oracle
## SQL plus from oracle container
sqlplus sys/password@//localhost:1521/ORCLCDB as sysdba
sqlplus system/password@//localhost:1521/ORCLCDB
sqlplus pdbadmin/password@//localhost:1521/ORCLPDB1

## Prepare the DB for debezium
(Source: https://debezium.io/documentation/reference/stable/connectors/oracle.html)

From the Oracle container terminal:

mkdir opt/oracle/oradata/recovery_area

sqlplus
CONNECT sys/top_secret AS SYSDBA
alter system set db_recovery_file_dest_size = 10G;
alter system set db_recovery_file_dest = '/opt/oracle/oradata/recovery_area' scope=spfile;
shutdown immediate
startup mount
alter database archivelog;
alter database open;
-- Should now "Database log mode: Archive Mode"
archive log list

exit;


## Kafka-ui URL
http://localhost:8080/


