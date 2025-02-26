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

3. Create a table and add some records 

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>USE test;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>CREATE TABLE pets (id BIGINT UNSIGNED NOT NULL, pet varchar(50), primary key(id));</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>INSERT INTO pets VALUES(1,'dog'),(2,'cat'),(3,'bunny');</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>select * from pets;</copy>
    ```

## Task 2: Save binary logs

1. List binary logs 

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SHOW BINARY LOGS;</copy>
    ```

2. Now retrieve where binary logs are saved. In our installation we see '/var/lib/mysql/' as path and 'binlog' as basename

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>SELECT @@log_bin_basename;</copy>
    ```

3. Exit from MySQL Shell

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>\q</copy>
    ```

4. Create a directory where store our binary logs

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mkdir /home/opc/backupdir/binlogs</copy>
    ```

5. The file backup\_variables.txt created by MySQL Enterprise Backup in 'meta' directory of the backup destination (/home/opc/backupdir/full_PiTR/) contains many useful info about the backup execution.

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

6. We can now save (dump) the binary logs using MySQL Shell using the position just retrieved 

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

7. Now we can check what we saved.

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

## Task 5: Execute the PiTR

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

3. Execute the load in dryRun mode to check it. We want to restore everything, we don't need other options than dryRun 

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.loadBinlogs('/home/opc/backupdir/binlogs/',{dryRun:true})</copy>
    ```

4. If there are no errors, execute hte load with dryRun:false

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.loadBinlogs('/home/opc/backupdir/binlogs/',{dryRun:false})</copy>
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
    <copy>SELECT * FROM test.pets;</copy>
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
