# 🛠️ MySQL Database Migration using AWS Database Migration Service (DMS)

## 📘 Project Overview
This project demonstrates **real-time data synchronization** between two MySQL databases — one as a **source** and the other as a **target** — using **AWS Database Migration Service (DMS)**.  

The migration includes:
- Migrating existing data (**Full Load**)
- Replicating ongoing changes (**CDC - Change Data Capture**)

---

## 🏗️ Architecture Overview

| Component | Description |
|------------|-------------|
| **Source Database** | MySQL (hosted on EC2 instance or RDS) |
| **Target Database** | MySQL (on RDS) |
| **Migration Tool** | AWS Database Migration Service (DMS) |
# 🛠️ MySQL Database Migration using AWS Database Migration Service (DMS)

## 📘 Project Overview
This project demonstrates **real-time data synchronization** between two MySQL databases — one as a **source** and the other as a **target** — using **AWS Database Migration Service (DMS)**.  

The migration includes:
- Migrating existing data (**Full Load**)
- Replicating ongoing changes (**CDC - Change Data Capture**)

---

## 🏗️ Architecture Overview

| Component | Description |
|------------|-------------|
| **Source Database** | MySQL (hosted on EC2 instance or RDS) |
| **Target Database** | MySQL (on RDS) |
| **Migration Tool** | AWS Database Migration Service (DMS) |

---

## ⚙️ Step-by-Step Implementation

---

### 🧩 Step 1: Create Source and Target MySQL Databases

### Step 1: Create Source and Target MySQL Databases

```sql
-- Connect to MySQL
mysql -u root -p;

-- Create source and target databases
CREATE DATABASE source_db;
CREATE DATABASE target_db;

-- Create a sample table in source
USE source_db;
CREATE TABLE employees (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50),
  department VARCHAR(50),
  salary INT
);

-- Insert sample data
INSERT INTO employees (name, department, salary)
VALUES
  ('John Doe', 'IT', 60000),
  ('Alice Smith', 'HR', 55000),
  ('Bob Lee', 'Finance', 70000);
```

### 🧩 Step 2: Enable Binary Logging for CDC

Enable **Change Data Capture (CDC)** by configuring binary logs in MySQL on the **source EC2 instance**.

```bash
# Edit MySQL configuration file
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

Restart MySQL:

sudo systemctl restart mysql


Verify Binary Logging:

mysql -u root -p -e "SHOW VARIABLES LIKE 'log_bin';"


---

### 🔑 Step 3: Configure User for DMS Access

Create a MySQL user that AWS DMS will use to connect.

mysql -u root -p
CREATE USER 'dms_user'@'%' IDENTIFIED BY 'DmsUser@123';
GRANT ALL PRIVILEGES ON . TO 'dms_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;


---

### 🧰 Step 4: Create IAM Roles for AWS DMS

In **Account B** (where DMS will run):  
Navigate to **IAM → Roles → Create Role**.  
Attach the following managed policies:
- AmazonDMSVPCManagementRole
- AmazonDMSCloudWatchLogsRole
- AmazonDMSRedshiftS3Role

These allow DMS to manage networking and write logs to CloudWatch.

---

### ☁️ Step 5: Create a DMS Replication Instance

In AWS DMS Console → **Replication Instances → Create**:

| Setting | Value |
|----------|--------|
| Name | mysql-dms-instance |
| Instance class | dms.t3.medium |
| Storage | 50 GB |
| VPC | Same as RDS |
| Multi-AZ | Optional |
| Publicly accessible | Yes (if EC2 source is public) |

✅ Wait for the status to become **Available**.

---

### 🌐 Step 6: Create Source and Target Endpoints

#### Source Endpoint

| Setting | Value |
|----------|--------|
| Identifier | source-mysql-endpoint |
| Engine | MySQL |
| Server | `<Source EC2 Public IP>` |
| Port | 3306 |
| Username | dms_user |
| Password | DmsUser@123 |
| Database name | source_db |
| SSL mode | none |

#### Target Endpoint

| Setting | Value |
|----------|--------|
| Identifier | target-mysql-endpoint |
| Engine | MySQL |
| Server | `<RDS Endpoint>` |
| Port | 3306 |
| Username | root |
| Password | `<RDS password>` |
| Database name | target_db |
| SSL mode | none |

✅ Test both connections — should show successful.

---

### ⚙️ Step 7: Configure and Create Migration Task

| Setting | Value |
|---------|--------|
| Task name | mysql-realtime-sync |
| Replication instance | mysql-dms-instance |
| Source endpoint | source-mysql-endpoint |
| Target endpoint | target-mysql-endpoint |
| Migration type | Migrate existing data and replicate ongoing changes (Full load + CDC) |

#### Advanced Task Settings

| Setting | Value |
|---------|--------|
| Create recovery table | ✅ On |
| Target table preparation mode | Truncate |
| Stop after full load completes | Do not stop |
| Include LOB columns | Full LOB mode |
| Data validation | Validation with data migration |
| Turn on CloudWatch logs | ✅ Enabled |
| Batch optimized apply | ✅ Enabled |

---

### ▶️ Step 8: Start Migration Task

Go to **AWS DMS → Tasks → Start/Resume**.  
Observe the progress:

- Full Load → Completed  
- CDC → Ongoing  

✅ This means all existing data is migrated and ongoing transactions are replicated in real time.

---

### 🧾 Step 9: Verify Data Migration and Real-time Sync

On the target database:

mysql -u root -p
USE target_db;
SELECT * FROM employees;


✅ You should see all the data from the source.

Test real-time synchronization:

USE source_db;
INSERT INTO employees (name, department, salary)
VALUES ('Charlie', 'Marketing', 65000);

-- Verify on target
USE target_db;
SELECT * FROM employees;


✅ The new record appears instantly — confirming CDC replication is active.

---

### 🧩 Step 10: Tune MySQL Timeouts (Optional)

If DMS shows timeout warnings, update MySQL timeouts.

mysql -u root -p
SET GLOBAL net_read_timeout = 600;
SET GLOBAL net_write_timeout = 600;
SET GLOBAL wait_timeout = 600;


Make these permanent in `/etc/mysql/mysql.conf.d/mysqld.cnf`:

net_read_timeout = 600
net_write_timeout = 600
wait_timeout = 600


Restart MySQL:

sudo systemctl restart mysql


---

## ✅ Final Outcome

- Full load + real-time replication (CDC) successfully configured  
- Zero downtime migration achieved  
- Continuous sync from EC2 MySQL → RDS MySQL  

Once verified, the old EC2 instance can be safely decommissioned.




---

## ⚙️ Step-by-Step Implementation

---

### 🧩 Step 1: Create Source and Target MySQL Databases

Connect to MySQL
mysql -u root -p

Create source and target databases
CREATE DATABASE source_db;
CREATE DATABASE target_db;

Create a sample table in source
USE source_db;
CREATE TABLE employees (
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(50),
department VARCHAR(50),
salary INT
);

Insert sample data
INSERT INTO employees (name, department, salary)
VALUES
('John Doe', 'IT', 60000),
('Alice Smith', 'HR', 55000),
('Bob Lee', 'Finance', 70000);

text

---

### 🧠 Step 2: Enable Binary Logging for CDC

Enable **Change Data Capture (CDC)** by configuring binary logs in MySQL on the **source EC2 instance**.

sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

text

Add or update the following lines under `[mysqld]`:

[mysqld]
server-id = 1
log_bin = mysql-bin
binlog_format = ROW
binlog_row_image = FULL
expire_logs_days = 10
binlog_checksum = NONE

text

Restart MySQL:

sudo systemctl restart mysql

text

Verify Binary Logging:

mysql -u root -p -e "SHOW VARIABLES LIKE 'log_bin';"

text

---

### 🔑 Step 3: Configure User for DMS Access

Create a MySQL user that AWS DMS will use to connect.

mysql -u root -p
CREATE USER 'dms_user'@'%' IDENTIFIED BY 'DmsUser@123';
GRANT ALL PRIVILEGES ON . TO 'dms_user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

text

---

### 🧰 Step 4: Create IAM Roles for AWS DMS

In **Account B** (where DMS will run):  
Navigate to **IAM → Roles → Create Role**.  
Attach the following managed policies:
- AmazonDMSVPCManagementRole
- AmazonDMSCloudWatchLogsRole
- AmazonDMSRedshiftS3Role

These allow DMS to manage networking and write logs to CloudWatch.

---

### ☁️ Step 5: Create a DMS Replication Instance

In AWS DMS Console → **Replication Instances → Create**:

| Setting | Value |
|----------|--------|
| Name | mysql-dms-instance |
| Instance class | dms.t3.medium |
| Storage | 50 GB |
| VPC | Same as RDS |
| Multi-AZ | Optional |
| Publicly accessible | Yes (if EC2 source is public) |

✅ Wait for the status to become **Available**.

---

### 🌐 Step 6: Create Source and Target Endpoints

#### Source Endpoint

| Setting | Value |
|----------|--------|
| Identifier | source-mysql-endpoint |
| Engine | MySQL |
| Server | `<Source EC2 Public IP>` |
| Port | 3306 |
| Username | dms_user |
| Password | DmsUser@123 |
| Database name | source_db |
| SSL mode | none |

#### Target Endpoint

| Setting | Value |
|----------|--------|
| Identifier | target-mysql-endpoint |
| Engine | MySQL |
| Server | `<RDS Endpoint>` |
| Port | 3306 |
| Username | root |
| Password | `<RDS password>` |
| Database name | target_db |
| SSL mode | none |

✅ Test both connections — should show successful.

---

### ⚙️ Step 7: Configure and Create Migration Task

| Setting | Value |
|---------|--------|
| Task name | mysql-realtime-sync |
| Replication instance | mysql-dms-instance |
| Source endpoint | source-mysql-endpoint |
| Target endpoint | target-mysql-endpoint |
| Migration type | Migrate existing data and replicate ongoing changes (Full load + CDC) |

#### Advanced Task Settings

| Setting | Value |
|---------|--------|
| Create recovery table | ✅ On |
| Target table preparation mode | Truncate |
| Stop after full load completes | Do not stop |
| Include LOB columns | Full LOB mode |
| Data validation | Validation with data migration |
| Turn on CloudWatch logs | ✅ Enabled |
| Batch optimized apply | ✅ Enabled |

---

### ▶️ Step 8: Start Migration Task

Go to **AWS DMS → Tasks → Start/Resume**.  
Observe the progress:

- Full Load → Completed  
- CDC → Ongoing  

✅ This means all existing data is migrated and ongoing transactions are replicated in real time.

---

### 🧾 Step 9: Verify Data Migration and Real-time Sync

On the target database:

mysql -u root -p
USE target_db;
SELECT * FROM employees;

text

✅ You should see all the data from the source.

Test real-time synchronization:

USE source_db;
INSERT INTO employees (name, department, salary)
VALUES ('Charlie', 'Marketing', 65000);

-- Verify on target
USE target_db;
SELECT * FROM employees;

text

✅ The new record appears instantly — confirming CDC replication is active.

---

### 🧩 Step 10: Tune MySQL Timeouts (Optional)

If DMS shows timeout warnings, update MySQL timeouts.

mysql -u root -p
SET GLOBAL net_read_timeout = 600;
SET GLOBAL net_write_timeout = 600;
SET GLOBAL wait_timeout = 600;

text

Make these permanent in `/etc/mysql/mysql.conf.d/mysqld.cnf`:

net_read_timeout = 600
net_write_timeout = 600
wait_timeout = 600

text

Restart MySQL:

sudo systemctl restart mysql

text

---

## ✅ Final Outcome

- Full load + real-time replication (CDC) successfully configured  
- Zero downtime migration achieved  
- Continuous sync from EC2 MySQL → RDS MySQL  

Once verified, the old EC2 instance can be safely decommissioned.
