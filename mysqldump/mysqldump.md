# MySQL Logical Backup

## Introduction
The mysqldump client utility performs logical backups, producing a set of SQL statements that can be executed to reproduce the original database object definitions and table data. It dumps one or more MySQL databases for backup or transfer to another SQL server. 

In this lab you will see how mysqldump works.

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* execute a backup with mysqldump 
* execute a schema restore from a backup created with mysqldump 


> **Note:**
 * Server: mysql1

## Task 1: mysqldump - backup

If not already connected to app-srv and mysql1 then do the following
- a. Connect with your SSH client using the public IP and the provided ssh Example of connections from Linux, MAC, Windows Powershell

    ```
    <span style="color:green">shell></span> <copy> ssh -i id_rsa_app-srv opc@<public_ip></copy>
    ```
- b. Connect to <span style="color:green">shell-mysql1</span>
    ```
    <span style="color:green">shell-app-srv$</span> <copy> ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1 </copy>
    ```

1. Create the export folder
    ```
    <span style="color:green">shell></span> <copy>sudo mkdir -p /mysql/exports</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo chown mysqluser:mysqlgrp /mysql/exports/</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>sudo chmod 770 /mysql/exports/</copy>
    ```

2. Export all the data with mysqldump
    ```
    <span style="color:green">shell></span> <copy>mysqldump -uadmin -p -hmysql1 -P3307 --single-transaction --events --routines --flush-logs --all-databases > /mysql/exports/full.sql</copy>
    ```

3. View  the content of the file /mysql/exports/full.
    ```
    <span style="color:green">shell></span> <copy>nano /mysql/exports/full.sql</copy>
    ```

4. Export a specific database (employees)

    ```
    <span style="color:green">shell></span> <copy>mysqldump -uadmin -p -hmysql1 -P3307 --single-transaction --set-gtid-purged=OFF --databases employees > /mysql/exports/employees.sql</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>ls -l /mysql/exports/employees.sql</copy>
    ```

5. Verify the files created

6. Check the file content

## Task 2: mysqldump - restore

1. Drop employees database, using the mysql client
    ```
    <span style="color:green">shell></span> <copy>mysql -uadmin -p -hmysql1 -P3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>show databases;</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>DROP DATABASE employees;</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>show databases;</copy>
    ```

2. Import the dropped employees database.
    It can be done in shell (as when we loaded first example data) or from within the mysql client.
    ```
    <span style="color:blue">mysql></span> <copy>SOURCE /mysql/exports/employees.sql</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>show databases;</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>show tables in employees;</copy>
    ```

3. Check that content is restored

7. Exit from mysql client

    ```
    <span style="color:blue">mysql></span> <copy>exit</copy>
    ```
    
## Learn More
* https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html
* To use MySQL Shell at command line read: https://dev.mysql.com/doc/mysql-shell/8.0/en/command-line-integration-overview.html
* https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-dump-instance-schema.html
* https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html


## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
