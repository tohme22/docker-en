---
title: "Demo"
description: "$ docker"
draft: false
weight: 4
---


# Demo 

### 1. Architecture

- **Site 3 LAN:** `10.10.60.0/24`
- **Docker Server:** `10.10.60.200`
  - **Port 80** → Apache + PHP
  - **Port 8080** → phpMyAdmin

#### Networks Overview

- **`site3_frontend`**
  - Apache + PHP
  - phpMyAdmin

- **`site3_backend`**
  - Apache + PHP
  - phpMyAdmin
  - MariaDB

### 2. Topology Diagram

```text
               Site 3 LAN: 10.10.60.0/24
                          │
                          ▼
             Docker Server: 10.10.60.200
               │
               ├── Port 80   → Apache + PHP
               └── Port 8080 → phpMyAdmin


                 site3_frontend
           --------------------------
          |                          |
Browser --> Apache + PHP             |
          |        ▲                 |
          |        │                 |
          |   phpMyAdmin             |
           --------------------------
                       ▲
                       │
           --------------------------
          |      site3_backend       |
          |                          |
          | Apache + PHP             |
          | phpMyAdmin               |
          | MariaDB                  |
           --------------------------
```

---

### 3. Create Project Directory

```bash
sudo mkdir /docker-site3
cd /docker-site3
```

---

### 4. Create Dockerfile

```bash
sudo vim dockerfile
```

Add the following content:

```dockerfile
FROM php:8.4-apache
RUN docker-php-ext-install mysqli pdo pdo_mysql
RUN a2enmod rewrite
WORKDIR /var/www/html
```

---

### 5. Build Apache/PHP Image

```bash
sudo docker build -t site3-apache-php:8.4 .
sudo docker image ls
```

---

### 6. Create Docker Networks

```bash
sudo docker network create site3_frontend
sudo docker network create --internal site3_backend
sudo docker network ls
```

---

### 7. Create Docker Volumes

```bash
sudo docker volume create site3_apache_web_data
sudo docker volume create site3_mariadb_data
sudo docker volume create site3_phpmyadmin_sessions
sudo docker volume ls
```

---

### 8. Create MariaDB Container

```bash
sudo vim .env
```

Add the environment variables:

```env
MARIADB_ROOT_PASSWORD=Passw0rd$
MARIADB_DATABASE=site3db
MARIADB_USER=site3user
MARIADB_PASSWORD=Passw0rd$
```

Set appropriate permissions and run the container:

```bash
sudo chmod 600 .env

sudo docker run -d \
  --name site3-mariadb \
  --restart unless-stopped \
  --network site3_backend \
  -v site3_mariadb_data:/var/lib/mysql \
  --env-file .env \
  mariadb:11

sudo docker ps
```

---

### 9. Create Apache/PHP Container

```bash
sudo docker run -d \
  --name site3-apache-php \
  --restart unless-stopped \
  --network site3_frontend \
  -p 10.10.60.200:80:80 \
  -v site3_apache_web_data:/var/www/html \
  site3-apache-php:8.4

sudo docker ps
```

Connect the container to the Backend network:

```bash
sudo docker network connect site3_backend site3-apache-php
sudo docker inspect site3-apache-php
```

---

### 10. Create phpMyAdmin Container

```bash
sudo docker run -d \
  --name site3-phpmyadmin \
  --restart unless-stopped \
  --network site3_frontend \
  -p 10.10.60.200:8080:80 \
  -v site3_phpmyadmin_sessions:/sessions \
  -e PMA_HOST=site3-mariadb \
  phpmyadmin:latest

sudo docker ps
```

Connect the container to the Backend network:

```bash
sudo docker network connect site3_backend site3-phpmyadmin
```

---

### 11. Verify Docker Infrastructure

```bash
sudo docker network inspect site3_frontend
sudo docker network inspect site3_backend
sudo docker volume ls
```

---

### 12. Verify PHP Container Functionality

```bash
sudo docker exec site3-apache-php php --version
sudo docker exec site3-apache-php php -m | grep -E "mysqli|PDO|pdo_mysql"
```

---

### 13. Add Persistent PHP Page in Apache

```bash
sudo docker exec -it site3-apache-php bash

cat > /var/www/html/index.php <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Site 3</title>
</head>
<body>
<h1>Apache and PHP are working!</h1>
<p>Server: Site 3</p>
<p>PHP Version:
<?php
echo PHP_VERSION;
?>
</p>
</body>
</html>
EOF
rm -f /var/www/html/index.html
exit
```

---

### 14. Configure AlmaLinux Firewall

```bash
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

---

### 15. Test Apache Locally in Web Browser

Navigate to: `http://10.10.60.200`

---

### 16. Test phpMyAdmin Locally in Web Browser

Navigate to: `http://10.10.60.200:8080`

- **Username:** `site3user`
- **Password:** `Passw0rd$`

> **Note:** You can also log in using the `root` account and its designated MariaDB password.

---

### 17. Test MariaDB

```bash
sudo docker exec -it site3-mariadb mariadb -u site3user -p site3db
```

Create a test table and insert sample data:

```sql
CREATE TABLE verification (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message VARCHAR(100)
);

INSERT INTO verification (message)
VALUES ('MariaDB data is persistent');

SELECT * FROM verification;
EXIT;
```

---

### 18. Create PHP Page Reading from MariaDB

```bash
sudo docker exec -it site3-apache-php bash

cat > /var/www/html/database-test.php <<'EOF'
<?php
$conn = new mysqli(
    "site3-mariadb",
    "site3user",
    "Passw0rd$",
    "site3db"
);
if ($conn->connect_error) {
    die("Database connection failed: " . $conn->connect_error);
}
$result = $conn->query("SELECT * FROM verification");
echo "<table border='1' cellpadding='8'>";
echo "<tr><th>ID</th><th>Message</th></tr>";
while ($row = $result->fetch_assoc()) {
    echo "<tr>";
    echo "<td>" . htmlspecialchars($row["id"]) . "</td>";
    echo "<td>" . htmlspecialchars($row["message"]) . "</td>";
    echo "</tr>";
}
echo "</table>";
$conn->close();
?>
EOF

exit
```

---

### 19. Local & Remote Verification

#### Local Database PHP Test
Navigate to: `http://10.10.60.200/database-test.php`

#### View Tables via phpMyAdmin
1. Navigate to: `http://10.10.60.200:8080`
2. Select database `site3db` → `verification` table → **Browse**

Expected output:
| id | message |
|---|---|
| 1 | MariaDB data is persistent |

---

### 20. Verify Persistent Volumes

```bash
sudo docker volume ls

sudo docker volume inspect site3_apache_web_data
sudo docker volume inspect site3_mariadb_data
sudo docker volume inspect site3_phpmyadmin_sessions
```

---

### 21. Test from Another VM

Navigate to: `http://10.10.60.200/database-test.php`
