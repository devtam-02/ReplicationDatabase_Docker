
# MySQL Master-Slave Replication Setup

This guide outlines the steps to initialize and manage a Master-Slave MySQL replication using Docker.

## Initialize the Master-Slave Replication

### Start the Containers

Run the following command to start both the master and the slave MySQL instances:

```bash
docker-compose up -d
```

### Configure the Master

1. **Connect to the Master Database:**

   ```bash
   docker exec -it mysql-master mysql -uroot -pmasterpassword
   ```

2. **Create a Replication User and Grant Replication Privileges:**

   Execute the following SQL commands inside the MySQL shell:

   ```sql
   CREATE USER 'devtam'@'%' IDENTIFIED WITH mysql_native_password BY 'Dev22x22';
   GRANT REPLICATION SLAVE ON *.* TO 'devtam'@'%';
   FLUSH PRIVILEGES;
   SHOW MASTER STATUS;
   ```

   Note the `File` and `Position` from the output of `SHOW MASTER STATUS;` for later use.

### Configure the Slave

1. **Connect to the Slave Database:**

   ```bash
   docker exec -it mysql-slave mysql -uroot -pslavepassword
   ```

2. **Configure the Master Connection Using GTID for Automatic Position Management:**

   Execute the following SQL commands inside the MySQL shell:

   ```sql
   CHANGE MASTER TO
   MASTER_HOST='mysql-master',
   MASTER_USER='devtam',
   MASTER_PASSWORD='Dev22x22',
   MASTER_AUTO_POSITION=1;
   ```

3. **Start the Slave:**

   ```sql
   START SLAVE;
   ```

   Run the following command to check the replication status:

   ```sql
   SHOW SLAVE STATUS\G
   ```

   Ensure that both `Slave_IO_Running` and `Slave_SQL_Running` fields are `Yes` to confirm that replication is active and running smoothly.

Đặt log database
```sql
   SET global general_log = 1;
   SET global general_log_file = 'logfile.log';
   SET GLOBAL time_zone = '+07:00';
   
   ```

Cập nhật timezone
```bash
   echo "Asia/Bangkok" | tee /etc/timezone
   ln -sf /usr/share/zoneinfo/Asia/Bangkok /etc/localtime
   dpkg-reconfigure --frontend noninteractive tzdata
```