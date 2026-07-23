---
title: "Demo-Apache-PHP-MariaDB"
description: "$ docker"
draft: false
weight: 5
---

### Architecture

- **Site 3 LAN:** `10.10.60.0/24`
- **Docker Server:** `10.10.60.200`
  - **Port 80** → Apache + PHP
  - **Port 8080** → phpMyAdmin

#### Component Breakdown

* **`site3_frontend`**
  * Apache + PHP
  * phpMyAdmin

* **`site3_backend`**
  * Apache + PHP
  * phpMyAdmin
  * MariaDB

#### Topology Diagram

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

### Create Setup Files

#### 1. Project Directory & Dockerfile

Create the project directory:
```bash
sudo mkdir /docker-site3
cd /docker-site3
```

Create the `Dockerfile`:
```bash
sudo vim Dockerfile
```

Add the following content:
```dockerfile
FROM php:8.4-apache
RUN docker-php-ext-install mysqli pdo pdo_mysql
RUN a2enmod rewrite
WORKDIR /var/www/html
```

#### 2. Environment Variables

Create the `.env` file:
```bash
sudo vim .env
```

Add your database configuration:
```env
MARIADB_ROOT_PASSWORD=Passw0rd$
MARIADB_DATABASE=site3db
MARIADB_USER=site3user
MARIADB_PASSWORD=Passw0rd$
```

Secure the `.env` permissions:
```bash
sudo chmod 600 .env
```

---

### Create Docker Compose File

Create the Compose file:
```bash
sudo vim compose.yaml
```

Add the following configuration:
```yaml
services:
  mariadb:
    image: mariadb:11
    container_name: site3-mariadb
    restart: unless-stopped

    environment:
      MARIADB_ROOT_PASSWORD: ${MARIADB_ROOT_PASSWORD}
      MARIADB_DATABASE: ${MARIADB_DATABASE}
      MARIADB_USER: ${MARIADB_USER}
      MARIADB_PASSWORD: ${MARIADB_PASSWORD}

    volumes:
      - mariadb_data:/var/lib/mysql

    networks:
      - site3_backend

  apache:
    build:
      context: .
      dockerfile: Dockerfile

    image: site3-apache-php:8.4
    container_name: site3-apache-php
    restart: unless-stopped

    ports:
      - "10.10.60.200:80:80"

    volumes:
      - apache_web_data:/var/www/html

    networks:
      - site3_frontend
      - site3_backend

    depends_on:
      mariadb:
        condition: service_healthy

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: site3-phpmyadmin
    restart: unless-stopped

    environment:
      PMA_HOST: mariadb
      PMA_PORT: 3306
      UPLOAD_LIMIT: 64M

    ports:
      - "10.10.60.200:8080:80"

    volumes:
      - phpmyadmin_sessions:/sessions

    networks:
      - site3_frontend
      - site3_backend

    depends_on:
      - mariadb
        
networks:
  site3_frontend:
    name: site3_frontend
    driver: bridge

  site3_backend:
    name: site3_backend
    driver: bridge
    internal: true

volumes:
  apache_web_data:
    name: site3_apache_web_data

  mariadb_data:
    name: site3_mariadb_data

  phpmyadmin_sessions:
    name: site3_phpmyadmin_sessions
```

---

### Validate and Launch Containers

#### Validate Configuration
```bash
sudo docker compose config
```

#### Start Containers
```bash
sudo docker compose up -d --build
```

#### Verify Running Services
```bash
sudo docker compose ps
sudo docker compose logs
```

---

### Verification & Configuration

#### Verify PHP Container Functionality
```bash
sudo docker exec site3-apache-php php --version
sudo docker exec site3-apache-php php -m | grep -E "mysqli|PDO|pdo_mysql"
```

#### Add a Persistent PHP Landing Page in Apache
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

### Network & Firewall Setup

#### Configure AlmaLinux Firewall
```bash
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

---

### Web Browser Access

### Test Apache Locally
Open browser to:
`http://10.10.60.200`

### Test phpMyAdmin Locally
Open browser to:
`http://10.10.60.200:8080`

- **Username:** `site3user`
- **Password:** `Passw0rd$`

> **Note:** You can also use the `root` account and its designated MariaDB password.

---

### Database Operations

#### Test MariaDB Connection
```bash
sudo docker exec -it site3-mariadb mariadb -u site3user -p site3db
```

#### Create a Test Table
Run the following SQL queries inside the database console:
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

#### Create a PHP Page Reading from MariaDB
```bash
sudo docker exec -it site3-apache-php bash

cat > /var/www/html/database-test.php <<'EOF'
<?php
$conn = new mysqli(
    "mariadb",
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

### Final Testing and Inspection

#### 1. Local Database Test
Navigate to:
`http://10.10.60.200/database-test.php`

#### 2. View Tables via phpMyAdmin
1. Navigate to `http://10.10.60.200:8080`
2. Select database: `site3db`
3. Click table: `verification`
4. Select **Browse**

**Expected Output:**

| id | message |
|---|---|
| 1 | MariaDB data is persistent |

#### 3. Inspect Persistent Volumes
```bash
sudo docker volume ls

sudo docker volume inspect site3_apache_web_data
sudo docker volume inspect site3_mariadb_data
sudo docker volume inspect site3_phpmyadmin_sessions
```

#### 4. Test from Another Virtual Machine
Navigate to:
`http://10.10.60.200/database-test.php`
