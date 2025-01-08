# MySQL Logical Backup

## Introduction
MySQL Shell's can be used to efficiently create a logical export of the instance
- instance dump utility util.dumpInstance(), to export the full instance content
- schema dump utility util.dumpSchemas(), to export all schemas or a selected schema
- table dump utility util.dumpTables(), to export a selection of tables or views from a schema

MySQL Shell's dump loading utility util.loadDump() supports the import into a MySQL Server instance of schemas or tables dumped using MySQL Shell's dump utilities.
The dump loading utility provides data streaming from remote storage, parallel loading of tables or table chunks, progress state tracking, resume and reset capability, and the option of concurrent loading while the dump is still taking place.

In this lab you will see how Mysql Shell dump & load works, and compare performances with mysqldump.

Estimated Lab Time: 15 minutes

### Objectives
In this lab, you will:
* Introduction to MySQL Shell
* execute a backup with MySQL Shell dump utility
* execute a schema restore with MySQL Shell load utility 
* compare execution time between mysqldump and MySQL Shell

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



## Task 1: MySQL Shell overview

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@<your_server_public_ip></copy>
    ```
   
2. Connect with MySQL Shell to your instance

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost</copy>
    ```

## Task 2: MySQL Shell dump utility
1. Switch to javascript command mode to check the export of the full instance.  
    To just check if export parameters are correct, we can execute it in dry-run

    **![orange-dot](./images/orange-square.jpg) mysqlsh>**  
    ```
    <copy>\js</copy>
    ```

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.dumpSchemas(['employees'],'/mysql/exports/employees',{dryRun:true})</copy>
    ```
2. If there are no errors, execute the export without dryRun.  
    Please **note the time required** with the default 4 threads.

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.dumpSchemas(['employees'],'/mysql/exports/employees')</copy>
    ```

3. Export a specific schema (employees)  
    Please **note the time required** with the default 4 threads.

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.dumpSchemas(['employees'],'/mysql/exports/employees')</copy>
    ```

5. Check the content of the directory /mysql/exports/employees

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>\q</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ls -l /mysql/exports/employees</copy>
    ```

5. Reconnect mysqlsh and drop employees database

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**  
    ```
    <copy>DROP DATABASE employees;</copy>
    ```

    **![orange-dot](./images/orange-square.jpg) mysql>**  
    ```
    <copy>SHOW DATABASES;</copy>
    ```

6. To load files, MySQL Shell requires local_infile variable enable (for security reasons is disabled by defautl)

    **![orange-dot](./images/orange-square.jpg) mysql>**  
    ```
    <copy>SET PERSIST local_infile=ON;</copy>
    ```

7. Switch to javascript command mode and test if parameters are correct using dryRun mode

    **![orange-dot](./images/orange-square.jpg) mysql>**  
    ```
    <copy> \js</copy>
    ```

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.loadDump('/mysql/exports/employees',{dryRun:true})</copy>
    ```

6. If there are no errors, load the employees database.  
    Please **note the time required** with the default 4 threads.

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>util.loadDump('/mysql/exports/employees')</copy>
    ```

7. Check that the database employees is imported. It's interesting to note that the ***"<code>\sql</code>"*** prefix let us execute SQL statements without swith to SQL mode

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>\sql SHOW DATABASES;</copy>
    ```

    **![yellow-dot](./images/yellow-square.jpg) mysqlsh>**  
    ```
    <copy>\q</copy>
    ```

7. We can also calculate the time with mysqldump. To use mysqldump in a more secure way we store the credential in login-path file (discussed in the module dedicated to security)
    * Let's create the login-path
    ```
    <span style="color:green">shell></span> <copy>mysql_config_editor set --login-path=mysql1 --host=mysql1 --port=3307 --user=admin --password</copy>
    ```
    * Let's test the login-path
    ```
    <span style="color:green">shell></span> <copy>mysql --login-path=mysql1 -e "SHOW DATABASES"</copy>
    ```

8. Now we can test the export.  
    Please **compare the mysqldump time** with the MysQL Shell dump.
    ```
    <span style="color:green">shell></span> <copy>time mysqldump --login-path=mysql1 --single-transaction --set-gtid-purged=OFF --databases employees > /mysql/exports/employees_time_test.sql</copy>
    ```

9. And now drop the database employees.  
    ```
    <span style="color:green">shell></span> <copy>mysql --login-path=mysql1 -e "drop database employees"</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>mysql --login-path=mysql1 -e "show databases"</copy>
    ```

10. And we can check the import time.  
    Please **compare the mysql import time** with the MysQL load.

    ```
    <span style="color:blue">mysql></span> <copy>time mysql --login-path=mysql1 < /mysql/exports/employees_time_test.sql</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>mysql --login-path=mysql1 -e "show databases"</copy>
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
