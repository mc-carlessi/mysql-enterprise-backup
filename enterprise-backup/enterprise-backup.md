# MySQL Enterprise Backup

## Introduction

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* We work now with <span style="color:red">mysql-advanced</span> instance

> **Note:**
 * Server: mysql1

## Task 1: Create a backup

1. MySQL Enterprise Backup is now available inside MySQL Enterprise Distributions like a tool, so you don’t have to install it.

2. Create directories to store backups
    ```
    <span style="color:green">shell></span> <copy>sudo mkdir -p /backupdir/full</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo mkdir /backupdir/compressed</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo chown -R mysqluser:mysqlgrp /backupdir</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo chmod 770 -R /backupdir</copy>
    ```

3. Create a user in <span style="color:red">mysql-advanced</span> with grants options for backup. 
    To simplify user creations we have a script with minimal grants for this user (see the manual for additional privileges required for specific features like TTS, SBT integration, encrypted). You can also have a look on the privileges opening the file /workshop/support/mysqlbackup_user_grants.sql

    ```
    <span style="color:green">shell></span> <copy>mysqlsh admin@mysql1:3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>CREATE USER 'mysqlbackup'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>source /workshop/support/mysqlbackup_user_grants.sql;</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>\q</copy>
    ```

4. Create a full backup 
    ```
    <span style="color:green">shell></span> <copy>mysqlbackup --port=3307 --host=127.0.0.1 --user=mysqlbackup --password --backup-dir=/backupdir/full backup-and-apply-log</copy>
    ```

5. Create a second backup with compression 
    ```
    <span style="color:green">shell></span> <copy>mysqlbackup --port=3307 --host=127.0.0.1 --user=mysqlbackup --password --backup-dir=/backupdir/compressed --compress backup-and-apply-log</copy>
    ```

6. Have a look of the content of the backup folders
    ```
    <span style="color:green">shell></span> <copy>cd /backupdir/full</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>ls -l</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>cd /backupdir/compressed</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>ls -l</copy>
    ```

7. Check the size of the two backups, the one uncompressed and the one compressed

    ```
    <span style="color:green">shell></span> <copy>cd /backupdir</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>du -sh *</copy>
    ```


## Task 2: Restore
1.  Stop the server
    ```
    <span style="color:green">shell></span> <copy>sudo systemctl stop mysqld-advanced</copy>
    ```
    
2. (<span style="color:red">destroy time!</span>) Empty datadir before restore the instance
    
    ```
    <span style="color:green">shell></span> <copy>sudo rm -rf /mysql/data/*</copy>
    ```

3. Make a copy of existing binary logs and create a directory to store the new ones  
    ```
    <span style="color:green">shell></span> <copy>sudo mv /mysql/binlog/ /mysql/binlog.old/</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo mkdir /mysql/binlog</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo chown mysqluser:mysqlgrp /mysql/binlog</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo chmod -R 750 /mysql/binlog</copy>
    ```

4. Restore the backup 
    We execute mysqlbackup as root because of permission in destination folders
    ```
    <span style="color:green">shell></span> <copy>sudo /mysql/mysql-latest/bin/mysqlbackup --defaults-file=/mysql/etc/my.cnf --backup-dir=/backupdir/full/ copy-back</copy>
    ```

5. Rename backup-auto.cnf, backup-mysqld-auto.cnf
    ```
    <span style="color:green">shell></span> <copy>sudo mv /mysql/data/backup-auto.cnf /mysql/data/auto.cnf</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo mv /mysql/data/backup-mysqld-auto.cnf /mysql/data/mysqld-auto.cnf</copy>
    ```

6. Set the ownership and privileges
    ```
    <span style="color:green">shell></span> <copy>sudo chown -R mysqluser:mysqlgrp /mysql/data /mysql/binlog</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo chmod -R 750 /mysql/data /mysql/binlog</copy>
    ```

7. Start the server and verify the data

    ```
    <span style="color:green">shell></span> <copy>sudo systemctl start mysqld-advanced</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>mysqlsh admin@mysql1:3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>SHOW DATABASES;</copy>
    ```
        

## Learn More
* https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/mysqlbackup.tasks.html
* https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/mysqlbackup.privileges.html


## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
