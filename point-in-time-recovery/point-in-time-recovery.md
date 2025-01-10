# MySQL Enterprise Backup

## Introduction
Point-in-time recovery (PiTR) refers to recovery of data changes up to a given point in time. Typically, this type of recovery is performed after restoring a full backup that brings the server to its state as of the time the backup was made. 
The full backup can be made in several ways (import from a mysqldump file, MySQL Shell load, MySQL Enterprise BAckup restore).
Binary logs are used to achieve a PiTR binary and the utility is mysqlbinlog.
To speed up the process is also possible to use replica concepts, as explained in [KB 2009693.1](https://support.oracle.com/epmos/faces/DocumentDisplay?id=2009693.1) and [KB 2277457.1](https://support.oracle.com/epmos/faces/DocumentDisplay?id=2277457.1), both available on support website [https://support.oracle.com](https://support.oracle.com).

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


## Task 1: prepare the instance and create the full backup

1. If not already connected, connect to your mysql server

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>ssh -i $HOME/sshkeys/id_rsa opc@<your_server_public_ip></copy>
    ```

2. MySQL binary logs are  enabled by default in MySQL 8. But it's recommended to use GTID, not enabled by default.
    Let's enable now the binary logs

    a. Edit my.cnf

        **![green-dot](./images/green-square.jpg) shell>**  
        ```
        <copy>sudo nano /etc/my.cnf</copy>
        ```
    b. Add these lines to my.cnf and save the file

    c. Restart the instance

3. Create a directory where store our backups

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mkdir -p /home/opc/backupdir/full_pitr</copy>
    ```

4. We are now ready to create a full backup 
    Note: we are creating a new backup, and not using one previously created, because we just enabled GTID mode.

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlbackup --defaults-file=/etc/my.cnf --login-path=mysqlbackup --backup-dir=/home/opc/backupdir/full_pitr backup-and-apply-log</copy>
    ```

## Task 2: Execute some commands for the PiTR


## Task 3: Save binary logs


## Task 3: Destroy teh instance and restore the backup
1.  Stop the server, to exclude unexpected behaviors from running processes

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo systemctl stop mysqld</copy>
    ```
    
2. Empty the datadir (if you don't know where is the datadir, read the section [Useful SQL Statements](../mysql-shell/mysql-shell.md#task-3-useful-sql-statements))

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>sudo ls -l /var/lib/mysql/*</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>sudo rm -rf /var/lib/mysql/*</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    </span> <copy>sudo ls -l /var/lib/mysql/*</copy>
    ```

3. Restore the backup. We use ***<code>sudo</code>*** because of ownership

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>sudo mysqlbackup --defaults-file=/etc/my.cnf --backup-dir=/home/opc/backupdir/full/ copy-back</copy>
    ```

5. Before start the isntance, if needed, rename the files backup-auto.cnf and backup-mysqld-auto.cnf

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

7. Verify that data are restored

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost -e "SHOW DATABASES;"</copy>
    ```

    **![green-dot](./images/green-square.jpg) shell>**  
    ```
    <copy>mysqlsh admin@localhost -e "SELECT * FROM employees.employees limit 10;"</copy>
    ```

## Task 4: Execute the PiTR with binary logs

You may now **proceed to the next lab**

## Learn More
* https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/mysqlbackup.tasks.html
* https://dev.mysql.com/doc/mysql-enterprise-backup/8.4/en/mysqlbackup.privileges.html


## Acknowledgements

* **Author** - Marco Carlessi, Principal Sales Consultant
* **Last Updated By/Date** - Marco Carlessi, MySQL Solution Engineering, January 2025
