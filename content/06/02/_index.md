---
title: "Demo 2"
description: "$ docker"
draft: false
weight: 2
---
### Demo 2

- Créer un réseau entre un containeur **Base de données** avec un containeur **Serveur Web**

##### **Étapes:**
1- Créer un réseau nommé **`monresau`**.
```yaml
$ docker network create monreseau
$ docker network ls
NETWORK ID     NAME        DRIVER    SCOPE
d26075b0d8d3   monreseau   bridge    local
```

2.	Créer le conteneur **Postgres (Base de données)** :

```yaml
$ docker run -d --name postgres -v database:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD=mysecretpassword --network monreseau postgres
```

3. Utilsier la command e`exec` pour créer une table avec des données dans la base données `postgres`

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

4. Créer le conteneur pour ** Jupyter (Serveur Web)** :

```yaml
docker run -p 8888:8888 --network monreseau jupyter/scipy-notebook
```

5. Ouvrez un navigateur et accéder au site web du serveur Jupyter:

    **http://127.0.0.1:8888/**
	

6. **Tester la connexion entre le Serveur Web et la Base de données**

   6.1 Sur le site de Jupyter, ouvrer un **Noteboook Python3**.
   
   6.2 Installer le paquet **`psycopg2-binary`** pour pouvoir exécuter une requête du serveur web à la base deonnées :
       
	**!pip install psycopg2-binary**   
   
	![](../../images/net2a.png?height=200&classes=border,shadow,inline)
   
   6.3 Importer les bibliothèques `pandas`, `psycopg2` et le module `extras` de la bibliothèque `psycopg2` :
   
    **import pandas as pd**
	
	**import psycopg2 as pg2**
	
	**import psycopg2.extras as pgex**

	![](../../images/net2b.png?height=100&classes=border,shadow,inline)
	
	6.4 Créer la connexion avec la base de données **postgres** :
	
	**connexion = pg2.connect(host='postgres', user='postgres', database='postgres', password='mysecretpassword')**

	![](../../images/net2c.png?height=80&classes=border,shadow,inline)
	
	6.5 Créer un curseur qui nous permet de lire à partir de la BD :
	
	**cursor = connexion.cursor(cursor_factory=pgex.RealDictCursor)**

	![](../../images/net2d.png?height=80&classes=border,shadow,inline)
	
	6.6 Utiliser ce curseur pour exécuter une requête et afficher les résultats :
	
	**cursor.execute("SELECT * FROM helloworld")**
	
	**data = cursor.fetchall()**
	
	**results = pd.DataFrame(data)**
	
	**print(results)**

	![](../../images/net2e.png?height=200&classes=border,shadow,inline)
