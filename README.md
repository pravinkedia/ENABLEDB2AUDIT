# ENABLEDB2AUDIT
Steps to Enabled DB2 Audit inside CP4D DV Engine Pod

### Create Directory to store the Audit logs on Persistent DV volumes 

mkdir /mnt/PV/versioned/db2audit /mnt/PV/versioned/db2audit/data /mnt/PV/versioned/db2audit/archivedata

### Enable DB2 Audit for all parameters

db2 connect to bigsql

db2 "update dbm cfg using AUDIT_BUF_SZ 256"

db2audit configure scope all status both errortype normal datapath /mnt/PV/versioned/db2audit/data archivepath /mnt/PV/versioned/db2audit/archivedata

### Check the DB2 Audit is enabled for complete monitoring

[bigsql@dv cli]# db2audit describe
DB2 AUDIT SETTINGS:

Audit active: "FALSE "
Log audit events: "BOTH"
Log checking events: "BOTH"
Log object maintenance events: "BOTH"
Log security maintenance events: "BOTH"
Log system administrator events: "BOTH"
Log validate events: "BOTH"
Log context events: "BOTH"
Return SQLCA on audit error: "FALSE "
Audit Data Path: "/var/log/bigsql/cli/db2audit/data/"
Audit Archive Path: "/var/log/bigsql/cli/db2audit/archivedata/"

AUD0000I Operation succeeded.

### Create Audit policy
db2 "create audit policy auditPolicy1 categories all status both error type normal"

db2 commit

### Enable Audit at DB level
db2 AUDIT DATABASE USING POLICY auditPolicy1 

### Start DB2 Audit
db2audit start

[bigsql@dv archivedata]# db2 commit
DB20000I The SQL command completed successfully.

### Check Audit is Enabled
[bigsql@dv archivedata]# db2audit describe
DB2 AUDIT SETTINGS:

Audit active: "TRUE "
Log audit events: "BOTH"
Log checking events: "BOTH"
Log object maintenance events: "BOTH"
Log security maintenance events: "BOTH"
Log system administrator events: "BOTH"
Log validate events: "BOTH"
Log context events: "BOTH"
Return SQLCA on audit error: "FALSE "
Audit Data Path: "/var/log/bigsql/cli/db2audit/data/"
Audit Archive Path: "/var/log/bigsql/cli/db2audit/archivedata/"

AUD0000I Operation succeeded.

### Run Queries and Check if they are being Audited in the DB2 Audit Log File

db2 "select * from CUSTOMER_DETAILS.ddd"

db2 "select * from CUSTOMER_DETAILS.RB_ACCOUNTS"

### Stop DB2 Audit and Flush the Archive Log File

db2audit stop

db2audit flush

### Archive the DB2 Audit File
db2audit archive database bigsql

### Extract the Audit data
cd /var/log/bigsql/cli/db2audit/archivedata/

db2audit extract file audit2.aud from files db2audit.db.BIGSQL.log.0.20210216175131

