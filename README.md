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
* [Documentation](https://protect724.hp.com/community/arcsight/productdocs)
* [Software/Patches](https://h20575.www2.hp.com/usbportal/softwareupdate.do)
* [Knowledge Base](http://support.openview.hp.com/selfsolve/documents)
* [Support Center](http://support.openview.hp.com/casemanager/incident-index)

### Procedure to TroubleShoot ESM State Up/Down

```
ssh arcsight@arcsight_esm 
sudo su - 
lsof -i TCP:1521
```

* Note if there is no **ESTABLISHED** communication with **Oracle**

```
/etc/init.d/./arcsight_manager stop; tail -f /opt/arcsight/manager/manager-5.5/logs/default/server.std.log
/etc/init.d/./arcsight_manager start; tail -f /opt/arcsight/manager/manager-5.5/logs/default/server.std.log
```

### Stop|Start ArcSight Enterprise Security Manager Gracefully 

* Note ensure you are the **arcsight** user. 

* To gracefully stop the ArcSight ESM

```
/etc/init.d/./arcsight_manager stop
```

* To gracefully start the ArcSight ESM

```
/etc/init.d/./arcsight_manager start
```

### Startup ArcSight Enterprise Security Manager Via Commandline for Troubleshooting Purposes **ONLY**

* Note ensure you are the **arcsight** user. The **&** backgrounds the process. 

```
cd /opt/arcsight/manager/manager-5.5/bin/
./arcsight manager &
```

### Useful Rudiment Linux Command Line Strings 

* From the ArcSight ESM Determine State of ESM to Oracle Communications - Private Network Interface

```
lsof -i TCP:1521
tcpdump -i bond1.114 dst port 1521
```

* From ArcSight ESM Determine State of Smart Connector or ArcSight Loggers Feeding - Public Network Interface 

```
lsof -i TCP:8443 
tcpdump –I bond0:244:0 dst port 8443 
```


### Important ArcSight $PATH Locations 

* ArcSight Manager $HOME 

```
/opt/arcsight/manager/manager-5.5
```

* ArcSight DB HOME

```
/opt/arcsight/db/db-5.5
```

* Oracle 11g $HOME

```
/opt/oracle/OraHome11g
```

### Important ArcSight $LOG Locations 

* ArcSight Manager $LOGS

```
/opt/arcsight/manager/manager-5.5/logs/default/server.std.log 
```

* Oracle 11g Alert $LOG

```
/opt/oracle/OraHome11g/diag/rdbms/arcsight/arcsight/trace/alert_arcsight.log
```

### Some Troubleshooting Fundamentals and Solutions


#### Tailing ESM Exceptions

```
ssh arcsight@esm_node 
cd $ARCSIGHT_MANAGER_HOME/logs/default
tail -f server.std.log | ../../bin/./arcsight exceptions
```


#### Detecting Excessive IOWAIT 

* SSH to the Oracle node within the ArcSight Architecture 

```
iostat 5 -c
```

* IOWAIT of 5% or less is fine 
* IOWAIT greater than 5% is bad


#### How to determine the IP Address of the Oracle node within Veritas Cluster

* The Oracle node IP address will be the IP Address that the ESM IP Address is connected to on TCP Port 1521, unless Oracle listener has been configured to listen on a non-standard TCP Port. 

```
ssh arcsight@esm_node 
netstat -an | grep 1521 | grep ESTABLISHED
10.0.0.3:20783   10.0.0.4:1521   ESTABLISHED
```


#### How to reset the ArcSight ESM admin password if forgotten or disabled/locked out

* SSH to the ESM node within your ArcSight architecture
* NOTE: If your ESM is running under the non-privledged user account 'arcsight' ensure you are the arcsight user while performing the following procedure. 

```
ssh arcsight@esm_node 
cd $ARCSIGHT_MANAGER_HOME/bin/scripts 
../arcsight resetpwd
admin
ENTER NEW PASSWD
REPEAT
Yes
```

* Now you should be able to login to the ArcSight Console with the 'admin' user account utilizing the new password



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

#### Manual Java KeyTool - Keys - Certificate Signing Request - Certificate Chain Trust Import - CA Root Import 
```
cd /apps/arcsight/manager/jre/bin

cp ../../config/jetty/keystore ../../config/jetty/keystore.bak.`date +'%m%d%Y'`

cp ../../config/jetty/keystore ../../config/jetty/keystore.request

./keytool -genkeypair -keyalg rsa -keysize 2048 -dname "CN=<your-fqdn>, OU=<your-org>, O=<your-org-unit>, L=<your-city>, S=<your-state>, C=<your-country-code>" -validity 1095 -keystore /apps/arcsight/manager/config/jetty/keystore.request -storepass <your-password> -alias <your-alias>

./keytool -certreq -alias <yourpassword> -keystore /apps/arcsight/manager/config/jetty/keystore.request -storepass <your-password> -file arcsight.csr

./keytool -import -trustcacerts -file certificate.p12 -keystore /apps/arcsight/manager/jre/lib/security/cacerts -storepass <yourpassword> -alias <your-alias>

./keytool -import -alias <your alias> -file ca_signed.crt -keystore /apps/arcsight/manager/config/jetty/keystore.request -storepass <yourpassword> 

cp ../../config/jetty/keystore.request ../../config/jetty/keystore 

./keytool -list -keystore /apps/arcsight/manager/config/jetty/keystore.request -alias <youralias>
```
####Export ESM TLS Certificate Via Command Line

  **From ESM Keystore**

```
$MANAGER_HOME/./arcsight keytool -store managerkeys -exportcert -alias mykey -file /home/arcsight/esm.cer
```

  **From Smart Connector Keystore***

```
$SMART_CONNECTOR_HOME/./arcsight agent keytool -store clientcerts -importcert -alias mykey -file /home/arcsight/esm.cer
```

####Performance Tuning ArcSight 6.5.1 

* Disable Red Hat kernel enabled Transparent Huge Pages 

```
echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled
```

* Set /etc/sysctl parameters for ESM to the following, ensure that these parameters are not already set to other values 

```
kernel.shmmax = 16642998272
kernel.shmall = 4160749568
vm.swappiness=0
vm.hugetlb_shm_group=500
vm.nr_hugepages=7168
vm.min_free_kbytes=131072
vm.dirty_ratio=8
vm.dirty_background_ratio=4
```

* Add the following ulimit values for the arcsight user to  **/etc/security/limits.d/90-nproc.conf** and to **/etc/security/limits.conf**

```
@arcsight soft memlock 16252928
@arcsight hard memlock 16252928
```

* Edit  **/opt/arcsight/logger/current/arcsight/logger/bin/scripts/servers.sh**

```
ARCSIGHT_JVM_OPTIONS="-verbose:gc -Xms256m -Xmx1536m -XX:+UseLargePages -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=utf-8 "
```

* Edit **/opt/arcsight/manager/config/server.wrapper.conf**

```
wrapper.java.additional.12=-XX:+UseLargePages
wrapper.java.additional.13=-XX:ReservedCodeCacheSize=256m

```

* Edit /opt/arcsight/logger/data/mysql/my.cnf
```
[mysqld]
large-pages
key_buffer_size=1G
sort_temp_limit = 50G
innodb_buffer_pool_size = 5G
```

*	**REBOOT THE SYSTEM**
```
init 6
```

####Increase the number of ArcSight ESM Log Files 

* Edit **server.properties**
```
log.channel.file.property.maxbackupindex=100
```

####Fixing SerializationException Error Problem Set Impacts all versions of AS SC to current 7.0.6

* Delete any cached events from the smart connector cache under $CONNECTOR_HOME/user/agent/agentdata
* Add the following parameters to a given smart connector agent.properties file

```
serializer.check.string.bytes=true
size.use.utf8.bytes=true
```

* Restart the smart connector 

####Logfu Set JVM Memory 

```
ARCSIGHT_JVM_OPTION=-Xmx1024m
```
