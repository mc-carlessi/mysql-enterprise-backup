# INSTALL - MYSQL ENTERPRISE EDITION

## Introduction

Installation of MySQL Enterprise Edition 8 and MySQL Shell on Linux using RPMs.

MySQL Shell is an advanced client and code editor for MySQL. In addition to the provided SQL functionality, similar to mysql, MySQL Shell provides scripting capabilities for JavaScript and Python and includes APIs for working with MySQL.
In this lab we use the client as an alternative to the legacy mysql client.

Estimated Time: 15 minutes

### Objectives

In this lab, you will:

* Install MySQL Enterprise Edition
* Install MySQL Shell 
* Import a sample database


### Prerequisites

This lab assumes you have:
* A working Oracle Linux machine
* The MySQL Enterprise rpms for Oracle Linux inside /workshop directory
* The employees sample database ([sample databases are downloadable from dev.mysql.com](https://dev.mysql.com/doc/index-other.html))

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![green-dot](./images/green-square.jpg) shell>  
  The command must be executed in the Operating System shell
* ![orange-dot](./images/blue-square.jpg) mysql>  
  The command is SQL and must be executed in a client like MySQL, MySQL Shell or similar tool
* ![yellow-dot](./images/yellow-square.jpg) mysqlsh>  
  The command must be executed in MySQL shell javascript command mode
  
## Task 1: Install MySQL Enterprise Edition using Linux RPM's


1. Connect to your **server** instance using an ssh client (**Example:** ssh -i ~/.ssh/id_rsa opc@132.145.17….)

  **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i ~/.ssh/id_rsa opc@<your_server_public_ip></copy>
    ```

    ![CONNECT](./images/ssh-login-2.png " ")


2. We have the required software available in **/workshop** directory. Please note that in our folder there are also devel and router rpms, that are not needed in this workshop

  - Let's check the directory content

  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>cd /workshop</copy>
      ```
  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>ls -l</copy>
      ```

  - Let's create a new directory  
  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>mkdir not_needed</copy>
      ```

  - Move the devel package that contains development header files and libraries for MySQL database client applications  
  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>mv mysql-commercial-devel* not_needed/</copy>
      ```

  - Move the MySQL Router package, because is usually not installed in the mysql servers, but in the application servers  
  **![green-dot](./images/green-square.jpg) shell>**  
      ```
      <copy>mv mysql-router-commercial* not_needed/</copy>
      ```

3. You can see that the only packages are server and clients.

 **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>ls -l</copy>
    ```

4. Now install the RPM's.

 **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo yum -y install *.rpm</copy>
    ```


5.	Start your new mysql instance

 **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo systemctl start mysqld</copy>
    ```

6.	Verify that process is running and listening on the default ports (3306 for MySQL standard protocol and 33060 for MySQL XDev protocol)

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>ps -ef | grep mysqld</copy>
    ```

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>netstat -an | grep 3306</copy>
    ```


7.	Another way is searching the message “ready for connections” in error log as one of the last 

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo grep -i ready /var/log/mysqld.log</copy>
    ```

8.	Retrieve root password for first login:

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>sudo grep -i 'temporary password' /var/log/mysqld.log</copy>
    ```

9. Login to the the mysql-enterprise, change temporary password and check instance the status

    **![green-dot](./images/green-square.jpg) shell>** 
     ```
    <copy>mysqlsh root@localhost</copy>
    ```

10. Create New Password for MySQL Root

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>ALTER USER 'root'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\status</copy>
    ```

## Task 2: Create a new administrative user and import a sample database

1.	Create a new administrative user called 'admin' with remote access and full privileges

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>CREATE USER 'admin'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;</copy>
    ```

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\quit</copy>
    ```

2.	Login as the new user, saving the password. MySQL Shell save the password in a secure file (mysql_config_editor is the default) and set history autosave

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
  <copy>mysqlsh admin@127.0.0.1</copy>
  ```

 **![orange-dot](./images/orange-square.jpg) mysqlsh>**
    ```
    <copy>\quit</copy>
    ```

3. Import the employees demo database that is in /workshop/databases folder.

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>cd /workshop/database</copy>
    ```

  **![green-dot](./images/green-square.jpg) shell>** 
    ```
    <copy>mysqlsh admin@127.0.0.1 < ./employees.sql</copy>
    ```
You may now **proceed to the next lab**

## Learn More

* [MySQL Linux Installation](https://dev.mysql.com/doc/en/binary-installation.html)
* [MySQL Shell Installation](https://dev.mysql.com/doc/mysql-shell/en/mysql-shell-install.html)

## Acknowledgements

* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
