---
title: "Demos"
description: "$ $ docker"
draft: false
weight: 2
---
### docker-compose.yml

#### Demo 1:
```yaml
$ mkdir rep1
$ cd rep1
$ vim docker-compose.yaml
version: "3.8"   
services:                    
    jupyter:
        image: jupyter/scipy-notebook
        ports:
            - "8888:8888"
        restart: always

$ docker-compose up

Open a browser:
http://127.0.0.1:8888

From another terminal:
$ docker ps -a ; docker network ls 

$ cd rep1
$ docker-compose down
```
#### Demo 2:
```yaml
$ mkdir /data 
$ vim .env
NB_USER=jovyan

$ vim docker-compose.yaml
version: "3.8"
services:
    jupyter:
        image: jupyter/scipy-notebook
        ports:
            - 8888:8888
        user: root
        volumes:
            - ./data:/home/jovyan/dataset                     
        environment:
            - CHOWN_HOME=yes
            - CHOWN_HOME_OPTS=-R
            - NB_USER=jovyan
        working_dir: "/home/${NB_USER}"

$ docker-compose up

Open a browser:
http://127.0.0.1:8888

From another terminal:
$ cd rep1
$ docker-compose down
```

#### Demo 3:
```yaml
$ vim .env
NB_USER=jovyan

$ vim docker-compose.yaml
version: "3.8"
networks:
  mynetwork:
    ipam:
      driver: default
      config:
        - subnet: "192.168.5.0/24"
services:
  jupyter:
    image: jupyter/scipy-notebook
    container_name: jupyter1
    ports:
      - 8888:8888
    user: root
    volumes:
      - ./dataset:/home/jovyan/dataset
    environment:
      - CHOWN_HOME=yes
      - CHOWN_HOME_OPTS=-R
      - NB_USER=${NB_USER}
    working_dir: "/home/${NB_USER}"
    networks:
      - mynetwork
  mydb:
    image: postgres
    container_name: mydb
    ports:
      - 5432:5432
    environment:
      POSTGRES_PASSWORD: mysecretpassword
    volumes:
      - database:/var/lib/postgresql/data
    networks:
      - mynetwork
volumes:
 database:
 
$ docker-compose up

From another terminal - Create a table in the Database:
$ cd rep1
$ docker-compose ps
$ docker network ls
$ docker inspect jupyter1
$ docker inspect mydb
$ docker volume ls 
$ docker exec -it mydb psql postgres postgres

CREATE TABLE helloworld (_id INTEGER, name TEXT);
INSERT INTO helloworld VALUES (1, 'Hello everyone');
INSERT INTO helloworld VALUES (2, 'How are you?');
SELECT * FROM helloworld;
\q

Open a browser:
http://127.0.0.1:8888
Paste the token.

Create a jupyter notebook and test the connection by reading the table:
! pip install psycopg2-binary 

import pandas as pd
import psycopg2 as pg2
import psycopg2.extras as pgex

# Creating the connection with the Postgres database
connexion = pg2.connect(host='mydb', user='postgres', database='postgres', password='mysecretpassword')

# create a cursor to query the database 
cursor = connexion.cursor(cursor_factory=pgex.RealDictCursor)

# run a SQL query
cursor.execute("SELECT * FROM helloworld")

# Store the results in a DataFrame and display the DataFrame
data = cursor.fetchall()
results = pd.DataFrame(data)
print(results)
```
![](../../images/docker-compose1.png?height=500&classes=border,shadow,inline))

```yaml
Go back to the Shell terminal:
$ docker-compose down
```






