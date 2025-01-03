# MySQL Logical Backup

## Introduction
MySQL Shell is an advanced client and code editor for MySQL. In addition to the provided SQL functionality, similar to mysql, MySQL Shell provides scripting capabilities for JavaScript and Python and includes APIs for working with MySQL.

MySQL Shell's includes
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
* execute similar commands with mysqldump and compare results

> **Note:**
 * Server: mysql1



## Task 1: MySQL Shell overview

If not already connected to app-srv and mysql1 then do the following
- a. Connect with your SSH client using the public IP and the provided ssh Example of connections from Linux, MAC, Windows Powershell

    ```
    <span style="color:green">shell></span> <copy> ssh -i id_rsa_app-srv opc@<public_ip></copy>
    ```
   
2. Connect with MySQL Shell
    ```
    <span style="color:green">shell></span> <copy>mysqlsh admin@mysql1:3307</copy>
    ```

## Task 2: MySQL Shell dump utility
3. Export teh full instance using javascript command mode.  
    Please **note the time required** with the default 4 threads.
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>><copy>\js</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>util.dumpSchemas(['employees'],'/mysql/exports/employees')</copy>
    ```

4. Export a specific schema (employees)  
    Please **note the time required** with the default 4 threads.
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>><copy>\js</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>util.dumpSchemas(['employees'],'/mysql/exports/employees')</copy>
    ```

4. Check the content of the directory /mysql/exports/employees
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\q</copy>
    ```
    ```
    <span style="color:green">shell></span> <copy>ls -l /mysql/exports/employees</copy>
    ```

5. Reconnect mysqlsh and drop employees database
    ```
    <span style="color:green">shell></span> <copy>mysqlsh admin@mysql1:3307</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>><copy> drop database employees;</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>><copy> show databases;</copy>
    ```

6. Load files is disabled by default, so enable it now to let MysQL Shell execute the load
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>><copy> set persist local_infile=ON;</copy>
    ```

6. Switch now in javascript command mode and re import the employees database.  
    Please **note the time required** with the default 4 threads.
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>><copy> \js</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>util.loadDump('/mysql/exports/employees')</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\sql show databases;</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\q</copy>
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
