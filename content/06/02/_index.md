---
title: "Example"
description: "$ docker"
draft: false
weight: 2
---
#### Create a network between a **Database** container and a **Web Server** container

##### **Steps:**
1- Create a network named **`mynetwork`**.
```yaml
$ docker network create mynetwork
$ docker network ls
NETWORK ID     NAME        DRIVER    SCOPE
d26075b0d8d3   mynetwork   bridge    local
```

2.	Create the **Postgres (Database)** container:

```yaml
$ docker run -d --name postgres --network mynetwork -p 5432:5432 -v database:/var/lib/postgresql/data -e POSTGRES_PASSWORD=mysecretpassword  postgres:17
```

3. Use the `exec` command to create a table with data in the `postgres` database:

```yaml
$ docker exec -it postgres psql postgres postgres
psql (16.1 (Debian 16.1-1.pgdg120+1))
Type "help" for help.

postgres=# CREATE TABLE helloworld (_id INTEGER, name TEXT);
CREATE TABLE
postgres=# INSERT INTO helloworld VALUES (1, 'Bonjour le Monde');
INSERT 0 1
postgres=# INSERT INTO helloworld VALUES (2, 'Salut, ca va?');
INSERT 0 1
postgres=# SELECT * FROM helloworld;
 _id |       name       
-----+------------------
   1 | Bonjour le Monde
   2 | Salut, ca va?
(2 rows)
postgres=# \q
```

4. Create the container for **Jupyter (Web Server):** :

```yaml
docker run -p 8888:8888 --network mynetwork jupyter/scipy-notebook
```

5. Open a browser and access the Jupyter server website:

**http://127.0.0.1:8888/**
	

6. Test the connection between the Web Server and the Database

   6.1 On the Jupyter website, open a **Python3 Notebook.thon3**.
   
   6.2 Install the **`psycopg2-binary`** package to run queries from the web server to the database:
       
	**!pip install psycopg2-binary**   
   
	![](../../images/net2a.png?height=200&classes=border,shadow,inline)
   
   6.3 Import the `pandas` and `psycopg2` libraries, and the `extras` module from the `psycopg2` library:
   
    **import pandas as pd**
	
	**import psycopg2 as pg2**
	
	**import psycopg2.extras as pgex**

	![](../../images/net2b.png?height=100&classes=border,shadow,inline)
	
	6.4 Create the connection to the **postgres** database:
	
	**connexion = pg2.connect(host='postgres', user='postgres', database='postgres', password='mysecretpassword')**

	![](../../images/net2c.png?height=80&classes=border,shadow,inline)
	
	6.5 Create a cursor allowing us to read from the DB:
	
	**cursor = connexion.cursor(cursor_factory=pgex.RealDictCursor)**

	![](../../images/net2d.png?height=80&classes=border,shadow,inline)
	
	6.6 Use this cursor to execute a query and display the results:
	
	**cursor.execute("SELECT * FROM helloworld")**
	
	**data = cursor.fetchall()**
	
	**results = pd.DataFrame(data)**
	
	**print(results)**

	![](../../images/net2e.png?height=200&classes=border,shadow,inline)
