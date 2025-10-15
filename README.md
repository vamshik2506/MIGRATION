🛠️ MySQL Database Migration using AWS Database Migration Service (DMS)
📘 Project Overview
This project demonstrates real-time data synchronization between two MySQL databases — one as a source and the other as a target — using AWS Database Migration Service (DMS).

The migration includes:

Migrating existing data (Full Load)

Replicating ongoing changes (CDC - Change Data Capture)

🏗️ Architecture Overview
Component	Description
Source Database	MySQL (hosted on EC2 instance or RDS)
Target Database	MySQL (on RDS)
Migration Tool	AWS Database Migration Service (DMS)
⚙️ Step-by-Step Implementation
🧩 Step 1: Create Source and Target MySQL Databases
bash
# Connect to MySQL
mysql -u root -p

# Create source and target databases
CREATE DATABASE source_db;
CREATE DATABASE target_db;

# Create a sample table in source
USE source_db;
CREATE TABLE employees (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50),
  department VARCHAR(50),
  salary INT
);

# Insert sample data
INSERT INTO employees (name, department, salary)
VALUES
('John Doe', 'IT', 60000),
('Alice Smith', 'HR', 55000),
('Bob Lee', 'Finance', 70000);
🧠 Step 2: Enable Binary Logging for CDC
Enable Change Data Capture (CDC) by configuring binary logs in MySQL on the source EC2 instance.

bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
Add or update the following lines:

text
[mysqld]
server-id = 1
log_bin = mysql-bin
binlog_format = ROW
binlog_row_image = FULL
expire_logs_days = 10
binlog_checksum = NONE
Restart MySQL to apply changes:

bash
sudo systemctl restart mysql
Verify binary logging:

bash
mysql -u root -p -e "SHOW VARIABLES LIKE 'log_bin';"
🔑 Step 3: Configure User for DMS Access
Create a MySQL user that AWS DMS will use to connect.

sql
mysql -u root -p
CREATE USER 'dms_user'@'%' IDENTIFIED BY 'DmsUser@123';
GRANT ALL PRIVILEGES ON *.* TO 'dms_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
🧰 Step 4: Create IAM Roles for AWS DMS
In Account B (where DMS will run):

Navigate to IAM → Roles → Create Role.

Attach the following managed policies:

AmazonDMSVPCManagementRole

AmazonDMSCloudWatchLogsRole

AmazonDMSRedshiftS3Role

These allow DMS to manage networking and write logs to CloudWatch.
