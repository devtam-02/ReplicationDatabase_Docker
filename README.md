Cấu trúc thư mục
Trước tiên, hãy tạo cấu trúc thư mục sau:

kotlin
Copy code
mysql-replication/
│
├── master/
│   ├── data/
│   └── my.cnf
│
└── slave/
    ├── data/
    └── my.cnf
Tệp cấu hình my.cnf
master/my.cnf
ini
Copy code
[mysqld]
server-id=1
log-bin=mysql-bin
binlog-do-db=testdb
slave/my.cnf
ini
Copy code
[mysqld]
server-id=2
relay-log=mysql-relay-bin
Tệp docker-compose.yml
Tạo tệp docker-compose.yml trong thư mục gốc mysql-replication/:

yaml
Copy code
version: '3.8'

services:
  mysql-master:
    image: mysql:latest
    container_name: mysql-master
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_REPLICATION_USER: replica
      MYSQL_REPLICATION_PASSWORD: replicapassword
    volumes:
      - ./master/data:/var/lib/mysql
      - ./master/my.cnf:/etc/mysql/my.cnf
    networks:
      - mysql-replication-network

  mysql-slave:
    image: mysql:latest
    container_name: mysql-slave
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_REPLICATION_USER: replica
      MYSQL_REPLICATION_PASSWORD: replicapassword
    volumes:
      - ./slave/data:/var/lib/mysql
      - ./slave/my.cnf:/etc/mysql/my.cnf
    networks:
      - mysql-replication-network

networks:
  mysql-replication-network:
    driver: bridge
Triển khai MySQL Replication
Chạy Docker Compose: Chuyển đến thư mục mysql-replication/ và chạy lệnh sau:
sh
Copy code
docker-compose up -d
Cấu hình Master: Kết nối vào MySQL Master để tạo cơ sở dữ liệu và người dùng replication:
sh
Copy code
docker exec -it mysql-master mysql -uroot -prootpassword

CREATE DATABASE testdb;
CREATE USER 'replica'@'%' IDENTIFIED BY 'replicapassword';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
Ghi lại File và Position từ kết quả của lệnh SHOW MASTER STATUS;.

Cấu hình Slave: Kết nối vào MySQL Slave và thiết lập replication:
sh
Copy code
docker exec -it mysql-slave mysql -uroot -prootpassword

CHANGE MASTER TO
MASTER_HOST='mysql-master',
MASTER_USER='replica',
MASTER_PASSWORD='replicapassword',
MASTER_LOG_FILE='mysql-bin.000001',  -- Thay thế bằng giá trị từ bước 2
MASTER_LOG_POS= 1234;  -- Thay thế bằng giá trị từ bước 2

START SLAVE;
Kiểm tra trạng thái replication:
sh
Copy code
docker exec -it mysql-slave mysql -uroot -prootpassword
SHOW SLAVE STATUS\G;
Mở khóa bảng trên Master:
sh
Copy code
docker exec -it mysql-master mysql -uroot -prootpassword
UNLOCK TABLES;
Kiểm tra hoạt động của Replication
Trên Master: Tạo bảng hoặc thêm dữ liệu vào cơ sở dữ liệu testdb.

Trên Slave: Kiểm tra xem bảng hoặc dữ liệu đã được replication chính xác chưa:

sh
Copy code
docker exec -it mysql-slave mysql -uroot -prootpassword
USE testdb;
SHOW TABLES;