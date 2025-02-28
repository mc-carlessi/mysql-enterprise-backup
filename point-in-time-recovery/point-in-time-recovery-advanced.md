# MySQL Enterprise Backup

## Introduction
Point-in-time recovery (PiTR) refers to recovery of data changes up to a given point in time. Typically, this type of recovery is performed after restoring a full backup that brings the server to its state as of the time the backup was made. 
The full backup can be made in several ways (import from a mysqldump file, MySQL Shell load, MySQL Enterprise Backup restore).
Binary logs are used to achieve a PiTR binary, apllying the, using the utility is mysqlbinlog, or better the MySQL Shell.
To speed up the PiTR process is also possible to use replica concepts, as explained in [KB 2009693.1](https://support.oracle.com/epmos/faces/DocumentDisplay?id=2009693.1) and [KB 2277457.1](https://support.oracle.com/epmos/faces/DocumentDisplay?id=2277457.1), both available on support website [https://support.oracle.com](https://support.oracle.com).

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* Create a full backup
* Apply some changes
* Restore the backup with the full backup and the binary logs

### Prerequisites

This lab assumes you have:
- All previous labs successfully completed

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![green-dot](./images/green-square.jpg) shell>  
  The command must be executed in the Operating System shell
* ![orange-dot](./images/orange-square.jpg) mysql>  
  The command is SQL and must be executed in a client like MySQL, MySQL Shell or similar tool
* ![yellow-dot](./images/yellow-square.jpg) mysqlsh>  
  The command must be executed in MySQL shell javascript command mode


## Task 1: Execute some commands to test PiTR

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. Connect with MySQL Shell to your instance

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

3. To simplify our lab, let's rotate the binary log before do anything

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SHOW BINARY LOGS;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>FLUSH BINARY LOGS;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SHOW BINARY LOGS;</copy>
    ```

3. Create a table and add some records 

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>USE test;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>INSERT INTO mytab VALUES(1,'Amanda'),(2,'Mark'),(3,'Viktor'),(4,'Sofia');</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SELECT * FROM mytab;</copy>
    ```

4. ***Now we do a mistake!*** Instead of delete a single record, delete the full content of the pets table forgetting to specify a WHERE condition.
    In next Task we do a recovery skipping the mistake

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>DELETE FROM mytab;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>select * from mytab;</copy>
    ```

## Task 2: Save binary logs

1. Exit from MySQL Shell

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>\q</copy>
    ```

2. Create a directory where store our binary logs

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir /home/opc/backupdir/binlogs</copy>
    ```

3. The file backup\_variables.txt created by MySQL Enterprise Backup in 'meta' directory of the backup destination (/home/opc/backupdir/full_PiTR/) contains many useful info about the backup execution.

    a. Have a look of the full content

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>cat /home/opc/backupdir/full/meta/backup_variables.txt</copy>
    ```

    b. We can also see the binlog file and position at the end of the backup. Please note this information

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>grep binlog_position /home/opc/backupdir/full/meta/backup_variables.txt</copy>
    ```

4. We can now save (dump) the binary logs using MySQL Shell using the position just retrieved 

    a. Connect with MySQL Shell to our instance

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

    b. Switch to javascript command mode

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>\js</copy>
    ```

    c. Execute the dump command in dryRun mode to check it. But before execute, please change '...' with the content of binlog_position variables (e.g. 'binlog.000004:198')

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.dumpBinlogs('/home/opc/backupdir/binlogs/',{startFrom:'...',dryRun:true})</copy>
    ```

    d. If there are no errors, repeat above command with dryRun=false

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.dumpBinlogs('/home/opc/backupdir/binlogs/',{startFrom:'...',dryRun:false})</copy>
    ```

5. Now we can check what we saved.

    a. Exit from the shell

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>\q</copy>
    ```

    b. You will see in the destination directory a directory for each dump and a file that contains the dump info ('@.binlog.info.json') 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l /home/opc/backupdir/binlogs</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>cat /home/opc/backupdir/binlogs/@.binlog.info.json</copy>
    ```

    c. inside the sub-directory there are the binlog compressed, each one with a json file that describe the content

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l /home/opc/backupdir/binlogs/*</copy>
    ```

## Task 3: Identify the wrong command position in the binary logs

1. Go where binlog are saved (our datadir) 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>cd /var/lib/mysql</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l binlog.*</copy>
    ```

2. To inspect the content of binary log we can use mysqlbinlog utility.
    This command displays row events encoded as base-64 strings (difficult to read for humans) in the last binlog  

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbinlog $(ls binlog.0* | tail -1 )</copy>
    ```

3. To inspect the content of the last binary log in a human readable form let's decode the content.
    See ![here](https://dev.mysql.com/doc/refman/8.4/en/mysqlbinlog-row-events.html) for mysqlbinlog output display options
    Please note
        * You will see that last commands are 'DELETE FROM `test`.`mytab`' and 
        * Before that command there is 'SET @@SESSION.GTID_NEXT= ...' variable setting

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbinlog --base64-output=DECODE-ROWS --verbose $(ls binlog.0*| tail -1 )</copy>
    ```

    SAMPLE OUTPUT:
    ```
    ...
    SET @@SESSION.GTID_NEXT= 'd66fca97-f421-11ef-bd8d-02001704792f:651'/*!*/;  
    # at 617  
    #250226 13:44:25 server id 1  end_log_pos 692 CRC32 0xb9ad93bc  Query   thread_id=8     exec_time=0     error_code=0  
    SET TIMESTAMP=1740573865/*!*/;  
    BEGIN  
    /*!*/;  
    # at 692  
    #250226 13:44:25 server id 1  end_log_pos 751 CRC32 0xb0bcfcae  Table_map: `test`.`mytab` mapped to number 98  
    # has_generated_invisible_primary_key=0  
    # at 751  
    #250226 13:44:25 server id 1  end_log_pos 847 CRC32 0xa39220ed  Delete_rows: table id 98 flags: STMT_END_F  
    ### DELETE FROM `test`.`mytab`  
    ### WHERE  
    ###   @1=1  
    ###   @2='Amanda'  
    ### DELETE FROM `test`.`mytab`  
    ### WHERE  
    ###   @1=2  
    ###   @2='Mark'  
    ### DELETE FROM `test`.`mytab`  
    ### WHERE  
    ###   @1=3  
    ###   @2='Viktor'  
    ### DELETE FROM `test`.`mytab`  
    ### WHERE  
    ###   @1=4  
    ###   @2='Sofia'  
    # at 847  
    #250226 13:44:25 server id 1  end_log_pos 878 CRC32 0x924831e8  Xid = 104  
    COMMIT/*!*/;  
    SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;  
    DELIMITER ;  
    # End of log file  
    /*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;  
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;  
    ```

4. ***IMPORTANT:*** In previous output ***note down*** the gtid in 'SET @@SESSION.GTID_NEXT= ...' just before the wrong delete.
    To simplify the search you can use this command

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy> mysqlbinlog --base64-output=DECODE-ROWS --verbose $(ls binlog.0*| tail -1 ) | grep -e 'SESSION.GTID_NEXT' -e 'DELETE FROM'</copy>
    ```

## Task 4: Destroy the instance and restore the backup
1.  Stop the server, to exclude unexpected behaviors from running processes

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl stop mysqld</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl status mysqld</copy>
    ```

2. Empty the datadir (if you don't know where is the datadir, read the section [Useful SQL Statements](../mysql-shell/mysql-shell.md#task-3-useful-sql-statements))

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>ls /var/lib/mysql/</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>sudo rm -rf /var/lib/mysql/*</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>ls /var/lib/mysql/</copy>
    ```

3. Restore the backup. We use ***<code>sudo</code>*** because of ownership

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mysqlbackup --defaults-file=/etc/my.cnf --backup-dir=/home/opc/backupdir/full/ copy-back</copy>
    ```

5. Before start the instance, if needed, rename the files backup-auto.cnf and backup-mysqld-auto.cnf

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mv /var/lib/mysql/backup-auto.cnf /var/lib/mysql/auto.cnf</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mv /var/lib/mysql/backup-mysqld-auto.cnf /var/lib/mysql/mysqld-auto.cnf</copy>
    ```

6. Set the correct ownership

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo chown -R mysql:mysql /var/lib/mysql/</copy>
    ```

7. Start the server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl start mysqld</copy>
    ```

7. Verify that data are restored. You will see that the new database test is not available (backup was taken before it's creation)

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost --table -e "SHOW DATABASES"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost --table -e "SELECT * FROM employees.employees limit 10"</copy>
    ```

## Task 5: Execute the PiTR till the wrong command

1. Connect with MySQL Shell 

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

2. Switch to javascript command mode

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>\js</copy>
    ```

3. Execute the load in dryRun mode to check it. We want to restore everything, we don't need other options than dryRun.
    Please change '...' in the 'stopBefore' with the GTID of the wrong delete

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.loadBinlogs('/home/opc/backupdir/binlogs/',{stopBefore:'...',dryRun:true})</copy>
    ```

4. If there are no errors, execute hte load with dryRun:false

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.loadBinlogs('/home/opc/backupdir/binlogs/',{stopBefore:'...',dryRun:false})</copy>
    ```

5. Now check that the data within test database are restored

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>\sql</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SHOW DATABASES;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SELECT * FROM test.mytab;</copy>
    ```

This **ends our workshop**

## Learn More
* ![MySQL Shell binlog utilities](https://dev.mysql.com/doc/mysql-shell/9.2/en/mysql-shell-utilities-dump-load-binlogs.html)
* ![Point in Time Recovery](https://dev.mysql.com/doc/refman/8.4/en/point-in-time-recovery.html)
* ![mysqlbinlog utility](https://dev.mysql.com/doc/refman/8.4/en/mysqlbinlog.html)
* ![MySQL Enterprise Backup Point in Time Recovery](https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/advanced.point.html)

## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
