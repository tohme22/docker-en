---
title: "Exercice"
description: "$ docker"
draft: false
weight: 3
---
### Exercise - Creating and Linking Two Containers with Docker Compose

#### Target:
- Create a `docker-compose.yml` file that will launch two services: a web application and a database.
- These services must communicate with each other and use environment variables for flexible configuration.

#### Exercise Statement:
1. Run Database Setup
- Create a database service with the **postgres** image.
- Call container **db1**.
- Use environment variables to set the database name **(DB_NAME)**, user **(DB_USER)**, and password **(DB_PASS)**.
- Create a volume named **db_data** that maps to the container's internal folder**/var/lib/postgresql/data**.

2. Application Setup:

- Use the **php:apache** image for the web service.
- Call the **web1** container.
- Map the folder**./www** to the container's internal folder**/var/www/html** _(Volume Bindmount)_
- Use the external port **8080** to access the application from a web browser.
- Configure the application to connect to the database using the provided environment variables.
- Add the following directive in the **web** section:
```yaml
   depends_on:
    - db
```

3. Docker network _(in both containers)_:
- Configure a custom Docker network, named **backend**, to facilitate communication between containers. _Tip: Create a host network._
```yaml
networks:
    - backend	
```

4. Environment Variables

- Add additional environment variables to configure more advanced aspects of your services (e.g., **DB_NAME** for the database, **WEB_PORT** for the web application). _Tip: Use an **.env** file_

5. Fichir index.php to test the conexion:

- On your host create the **www** folder.
- Go to **www** and create the **index.php** file with the following script:
```bash
<?php
$hostname = "db"; 
$port = 5432; // Port par défaut pour PostgreSQL

$fp = fsockopen($hostname, $port, $errno, $errstr, 10);
if (!$fp) {
    echo "Échec de la connexion : $errstr ($errno)\n";
} else {
    echo "Connexion réussie au serveur de base de données.\n";
    fclose($fp);
}
?>
```
---
#### Steps:
1. Create the necessary files: **docker-compose.yml** and **.env**.
2. Run the following command to launch the services defined in your file: **docker-compose up -d**.
3. Check the Condition of the Containers, volumes and networks:                                                                         **docker ps -a**, **docker volume ls** and **docker network ls**.

4. Test Connection Between Containers:
- To test whether the web application can communicate with the database, you must first install the mysqli extension and then restart the Apache server. Use these two commands:
```yaml
$ docker exec -it web1 docker-php-ext-install mysqli 
$ docker exec -it web1 apachectl restart
```
- Open a browser and go to **http://localhost:8080**.
- You should see a message that the connection to the database was successful.
5. Stop and Clean:
   Once you have completed the tests, you can stop and delete the containers with the command: **docker-compose down**.






