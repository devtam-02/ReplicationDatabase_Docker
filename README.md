# MySQL Master-Slave Replication Setup

This guide outlines the steps to initialize and manage a Master-Slave MySQL replication using Docker.

## Initialize the Master-Slave Replication

### Start the Containers

Run the following command to start both the master and the slave MySQL instances:

```bash
docker-compose up -d
Configure the Master
Connect to the Master Database:

bash
Copy code
docker exec -it mysql-master mysql -uroot -pmasterpassword
Create a Replication User and Grant Replication Privileges:

Execute the following SQL commands inside the MySQL shell:

sql
Copy code
CREATE USER 'repl_user'@'%' IDENTIFIED WITH mysql_native_password BY 'repl_pass';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;
SHOW MASTER STATUS;
Note the File and Position from the output of SHOW MASTER STATUS; for later use.

Configure the Slave
Connect to the Slave Database:

bash
Copy code
docker exec -it mysql-slave mysql -uroot -pslavepassword
Configure the Master Connection Using GTID for Automatic Position Management:

Execute the following SQL commands inside the MySQL shell:

sql
Copy code
CHANGE MASTER TO
MASTER_HOST='mysql-master',
MASTER_USER='repl_user',
MASTER_PASSWORD='repl_pass',
MASTER_AUTO_POSITION=1;
Start the Slave:

sql
Copy code
START SLAVE;
Run the following command to check the replication status:

sql
Copy code
SHOW SLAVE STATUS\G
Ensure that both Slave_IO_Running and Slave_SQL_Running fields are Yes to confirm that replication is active and running smoothly.

kotlin
Copy code

Save this text in a `README.md` file in your project directory where your `docker-compose.yml` file is located. This README will help users or administrators understand how to set up and verify the replication process.






