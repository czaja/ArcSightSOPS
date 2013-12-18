### Connector Sensor Tier Health Checks
* [Current ArcSight Smart Connector Supported Devices ](http://www.hpenterprisesecurity.com/collateral/HP_ArcSight_Supported_Products.pdf)
* UP/Down Check 
* Connector Event Rate Check 
* Cache Check 
* Log Check 
* Configuration Check 

### Connector Appliance Management Tier Health Checks 
* CPU & Memory Check 
* Network Settings Check 
* Configuration Backup Check 

### Enterprise Security Manager(ESM) Application Tier Health Checks 
* Event Throughput Dashboard Check 
* Current Event Sources Dashboard Check 
* CPU/Memory Utilization Check 
* Active/Session List Utilization Check 
* Rules Engine Check 
* Event Persistence Performance Check(Event Insertion Rate)
* Error Check 
* Scheduled Task Check 
* server.properties Check 
* Agent & Console Threads Check 

### Oracle Data Tier Health Checks 
* DBCheck & Oracle RDA
* Database Performance Statistics Dashboard 
* Partition Check
* Trend Job Check 
* Hardware and Operating System Check 
* CPU & Memory Utilization Check 
* Oracle Version and Patch Level Check 
* Oracle Alert Log Check 
* Oracle Memory Parameters Check 
* ESM Database Storage Check 

### HP/ArcSight Support URL Reference
* [Software/Patches](https://support.openview.hp.com/selfsolve/patches)
* [Knowledge Base](http://support.openview.hp.com/selfsolve/documents)
* [Support Center](http://support.openview.hp.com/casemanager/incident-index)

### Some Troubleshooting Fundamentals and Solutions

#### Connectivity between the ESM and Oracle data tier lost 

* The ESM will generate an **ERROR** event within $ARCSIGHT_HOME/logs/default/server.std.log

```
ERROR:java.sql.SQLException: Got minus one from a read call
```

* Perform the following troubleshooting actions from the Oracle node of the ESM Architecture 
NOTE: utilize the **oracle** user account!!

```
sqlplus '/ as sysdba' 
SQL> select instance_name, status, from gv$instance;
```

* Next Check the status of the Oracle listener to ensure that the listener is in state **LISTENING**

```
cd $ARCSIGHT_DB_HOME/db/bin
./arcdbutil lsnrctl status
```

* From the Oracle node of the ESM Architecture check **sqlnet.ora** ensure TCP.INVITED_NODES parameter allows the ESM hostname

```
cat $ORACLE_HOME/OraHome11g/network/admin/sqlnet.ora
```

* Next from the ESM system 

```
telnet oracle.hostname.com 1521
```

#### Determine the Startup Time of the Oracle ArcSight database instance 
NOTE: As the **oracle** user

```
select to_char(startup_time, 'HH24:MI DD-MON-YY') "Startup Time"
from v$instance
/
```

#### Determine from SQL current Database size and available free space
NOTE: As the **oracle** user copy and past on the Oracle node the following script and name the file 
**check_current_free.sql**
[check_current_free.sql](https://gist.github.com/alienone/8031362/raw/c79fe2cb39122fc58e437690eda68a1f002b6d54/check_current_free.sql)

```
vim check_current_free.sql
I #(INSERT)
:wq! #(WRITE AND QUIT FILE)

sqlplus '/ as sysdba'
!ls 
@check_current_free.sql
```

#### Determine available space for each table space within a given Oracle DB instance 
NOTE: As the **oracle** user - download and save the following script: [free_space_table.sql](https://gist.github.com/alienone/8031427/raw/73943d970cace46d357966055d77b2cc5c9fd95a/free_space_table.sql)

```
vim check_current_free.sql
I #(INSERT)
:wq! #(WRITE AND QUIT FILE)

sqlplus '/ as sysdba'
!ls 
@free_space_table.sql
``` 
