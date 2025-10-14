# 🛠️ MySQL Database Migration using AWS Database Migration Service (DMS)

## 📘 Project Overview
This project demonstrates **real-time data synchronization** between two MySQL databases — one as a **source** and the other as a **target** — using **AWS Database Migration Service (DMS)**.  

The migration includes:
- Migrating existing data (**Full Load**)
- Replicating ongoing changes (**CDC - Change Data Capture**)

---

## 🏗️ Architecture Overview

**Source Database:** MySQL (hosted on EC2 instance or RDS)  
**Target Database:** MySQL (on another EC2 instance or RDS)  
**Migration Tool:** AWS Database Migration Service (DMS)

---

## ⚙️ Step-by-Step Implementation

---

### 🧩 Step 1: Create Source and Target MySQL Databases

```bash
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
