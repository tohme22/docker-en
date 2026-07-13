---
title: "Solution"
description: "$ docker"
draft: true
weight: 4
---
### Solution - Creating and Linking Two Containers with Docker Compose

```yaml
vim .env

DB_NAME=ma_base_de_donnees
DB_USER=mon_utilisateur
DB_PASS=mon_mot_de_passe
WEB_PORT=8080
```

```yaml
vim docker-compose.yml
version: '3.8'
services:
  db:
    image: postgres
    container_name: db1
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASS}
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend
web:
  image: php:apache
  container_name: web1
  volumes:
    - ./www:/var/www/html 

    - "${WEB_PORT}:80"
  environment:
    DB_HOST: db
    DB_NAME: ${DB_NAME}
    DB_USER: ${DB_USER}
    DB_PASS: ${DB_PASS}
  depends_on:
    - db
  networks:
    - backend
networks:
  backend:
volumes:
  db_data:
```

![](../../images/docker-comp1.png?height=200&classes=border,shadow,inline))


