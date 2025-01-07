# INSTALL - MYSQL USEFUL COMMANDS  

## Introduction

Goal:
    Verify the new MySQL Installation on Linux and import test databases

Objectives:

- understand better how MySQL connection works
- have a look on useful statements

Estimated Time: -- 10 minutes

### Objectives

In this lab, you will:

- Connect to Ports
- Learn Useful SQL Statements

### Prerequisites

This lab assumes you have:

- All previous labs successfully completed

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>  
  The command must be executed in the Operating System shell
* ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>  
  The command must be executed in a client like MySQL, MySQL Shell or similar tool
* ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>  
  The command must be executed in MySQL shell

1. If not already connected, connect to your mysql server

    ```
    <span style="color:green">shell-app-srv$</span> <copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```

2. Our installation package is under /workshop/linux/ directory

    ```
    <span style="color:green">shell-mysql1></span> <copy>ls -l /workshop/linux/mysql-shell-commercial-8.*.rpm</copy>
    ```

2. Install Mysql Shell, a new client that can be used in devOps organizations (you’ll learn more about it during the course) 

    ```
    <span style="color:green">shell-mysql1></span> <copy>sudo yum install /workshop/linux/mysql-shell-commercial-8.*.rpm</copy>
    ```

3. Launch MySQL shell

    ```
    <span style="color:green">shell-mysql1></span> <copy>mysqlsh</copy>
    ```

4. Discover the command auto-completion feature, type \h and press TAB twice

    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\h</copy> [TAB][TAB]
    ```

5. Check the available options. Add the letter “e” to “\h” and press TAB again to see that the command will automatically complete for you. Press enter and explore the help menu

    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\he</copy> [TAB]
    ```

6. Activate the command history autosave in MySQL shell  
    MySQL Shell comes with the option of automatically saving the history of command disabled by default. Therefore we need to check and to activate it.
    * Show settings and look for history.autoSave

    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\option -l</copy>
    ```
    
    * Activate autosave history
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\option --persist history.autoSave=1</copy>
    ```

7. Close and reopen the session and in the new one **uses the arrow up key** to verify that the data from previous session are available

    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\q</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span> <copy>mysqlsh</copy>
    ```

## Task 2: MySQL as a SQL Client

1. MySQL Shell can be used as SQL client.  
   Connect to the newly installed mysql-advanced instance. Enter the password when requested and press enter. 
    
    When the prompt asks to save the password choose No and press Enter.

    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\c admin@mysql1:3307</copy>
    ```
2. Now you can submit SQL commands

    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>show databases;</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>SELECT * FROM world.city LIMIT 10;</copy>
    ```

3. MySQL Shell has also nice features. Here we see one of them: reporting.
   You can check the actual sessions with one of these commands
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>show processlist;</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>select * from performance_schema.processlist;</copy>
    ```

3. But continously check connection can be annoying, so we use now the command “query” that continuously repeat a query (default interval refresh time: 5 seconds).
    To exit from reporting press CTRL+C
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>\watch query --interval=2 select now(), processlist.* from performance_schema.processlist;</copy>
    ```

4. Switch to Python command mode, then Javascript command mode, check how prompt change
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>\py</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:blue">Py</span></span>><copy>\js</copy>
    ```

5. Exit from MySQL Shell with “\q” or “\exit”
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>><copy>\q</copy>
    ```

6. Command line connection from MySQL Client and MySQL Shell are similar. Try these:
    ```
    <span style="color:green">shell-mysql1></span> <copy>mysql -uadmin -p -hmysql1 -P3307</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>exit</copy>
    ```
    ```
    <span style="color:green">shell-mysql1></span> <copy>mysqlsh -uadmin -p -hmysql1 -P3307</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>SHOW DATABASES;</copy>
    ```
    ```
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="color:white"><span style="background-color:orange">SQL</span></span>><copy>\exit</copy>
    ```

7. From now on, if you like, the labs may be completed with MySQL Shell instead of classic mysql client.

## Task 1: MySQL Connection

Please note that password can be saved in an obfuscated file, to be reused, using the utility mysql_config_editor.  
This approach let you configure more secure and flexible scripts.

1. Set the login path **local_admin** to be used for easier connections 

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql_config_editor set --login-path=local_admin --user=admin --host=127.0.0.1 -p</copy>
    ```

2. Test the connection with mysql and mysqlsh clients

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql --login-path=local_admin</copy>
    ```
 **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>exit</copy>
    ```

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysqlsh --login-path=local_admin</copy>
    ```
 **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>\quit</copy>
    ```

2. List existing settings, please note that password are obfuscated

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql_config_editor print --all</copy>
    ```

## Task 2: Learn Useful SQL Statements

1. Connect to your instance

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

2. You can check the values of a specific variable, like the version of your server to know if it's updated to last release or not  
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>SHOW VARIABLES LIKE "%version%";</copy>
    ```

3. InnoDB provides the best Storage Engine in the general use case. You can check if all thethere are tables in a different format (of course, excluding system schemas)
    ```
    <copy>SELECT table_schema table_name, engine FROM INFORMATION_SCHEMA.TABLES where engine <> 'InnoDB' and table_schema not in ('mysql','information_schema', 'sys', 'performance_schema', 'mysql_innodb_cluster_metadata');</copy>
    ```

4. You can check the amount of data inside all the databases  
    ```
    <copy>SELECT table_schema AS 'Schema', SUM( data_length ) / 1024 / 1024 AS 'Data MB', SUM( index_length ) / 1024 / 1024 AS 'Index MB', SUM( data_length + index_length ) / 1024 / 1024 AS 'Sum' FROM information_schema.tables GROUP BY table_schema ;</copy>
    ```

5. You can check the amount of data inside all tables in a specific database ('employees' in the example)  
    ```
    <copy>SELECT table_schema AS 'Schema', table_name, SUM( data_length ) / 1024 / 1024 AS 'Data MB', SUM( index_length ) / 1024 / 1024 AS 'Index MB', SUM( data_length + index_length ) / 1024 / 1024 AS 'Sum' FROM information_schema.tables WHERE table_schema='employees' GROUP BY table_name;</copy>
    ```

6. You can check the size of tablespaces files for a specific database ('employees' in the example)  
    ```
    <copy>SELECT name, space AS 'tablespace id', allocated_size /1024 /1024 AS 'size (MB)', encryption FROM innodb_tablespaces WHERE name LIKE 'employees/%';</copy>
    ```

7. The “\G” is like “;” with a different way to show results

  **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>SHOW GLOBAL VARIABLES LIKE 'version%';</copy>
    ```
    ```
    <copy>SHOW GLOBAL VARIABLES LIKE 'version%'\G</copy>
    ```

8. Show connections

  **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>SHOW FULL PROCESSLIST;</copy>
    ```

9. You can now exit
    ```
    <copy>\q</copy>
    ```

## Learn More

* [MySQL Tutorial](https://dev.mysql.com/doc/en/tutorial.html)

## Acknowledgements

- **Author** - Dale Dasker, MySQL Solution Engineering
- **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
