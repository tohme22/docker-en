---
title: "Demo"
description: "docker"
draft: false
weight: 2
---
### UCommandes utilisées

```yaml
# Lister toutes les images Docker disponibles localement
$ docker image ls

# Télécharger l’image Ubuntu depuis Docker Hub
$ docker pull ubuntu

# Créer et exécuter un conteneur basé sur Ubuntu pour exécuter la commande 'ls'
$ docker run ubuntu ls

# Supprimer un conteneur (remplacer <container_name> par le nom du conteneur)
$ docker rm <container_name>

# Créer et exécuter un conteneur interactif nommé 'ubuntu_container' basé sur Ubuntu
$ docker run -it --name ubuntu_container ubuntu

Depuis un autre terminal :
---------------------------
# Afficher en temps réel les statistiques d’utilisation (CPU, mémoire, etc.) des conteneurs
$ docker stats 

Revenir au premier terminal :
------------------------------
# Quitter le shell du conteneur
root@9dad023013d9:/# exit

# Renommer le conteneur ubuntu_container en ubuntu_cont
$ docker rename ubuntu_container ubuntu_cont

# Lister tous les conteneurs (actifs et arrêtés)
$ docker ps -a
```

```yaml
# Lancer un conteneur interactif basé sur l’image Node.js
$ docker run -it node 

# Effectuer une addition dans le REPL Node.js
> 1 + 1
2
# Quitter le REPL Node.js
> .exit

# Lancer un conteneur interactif basé sur l’image Python
$ docker run -it python 

# Effectuer une addition dans l’interpréteur Python
>>> 1 + 1
2
# Quitter l’interpréteur Python
>>> exit()

# Lister tous les conteneurs (actifs et arrêtés)
$ docker ps -a

# Démarrer un conteneur (remplacer <container_name> par le nom du conteneur)
$ docker start <container_name>

# Démarrer un conteneur en mode interactif
$ docker start -i <container_name>

# Arrêter un conteneur en cours d’exécution
$ docker stop <container_name>

# Forcer l’arrêt immédiat d’un conteneur (kill)
$ docker kill <container_name>

# Afficher en temps réel les statistiques d’utilisation (CPU, mémoire, etc.)
$ docker stats 

# Afficher les logs d’un conteneur
$ docker logs <container_name>

# Afficher toutes les informations détaillées d’un conteneur en format JSON
$ docker inspect <container_name>

# Lancer un conteneur Redis en arrière-plan
$ docker run -d redis

# Ouvrir une session redis-cli dans un conteneur existant (remplacer <container_name>)
$ docker exec -it <container_name> redis-cli 

# Quitter l’invite redis-cli
127.0.0.1:6379> exit

# Lister tous les conteneurs
$ docker ps -a

# Arrêter un conteneur
$ docker stop <container_name>
```

```yaml
# Lancer un conteneur Jupyter TensorFlow Notebook et mapper le port 12345 vers 8888
$ docker run --rm -p 12345:8888 jupyter/tensorflow-notebook

Ouvrir un navigateur Internet :
-------------------------------
# Accéder à Jupyter Notebook dans le navigateur
http://127.0.0.1:12345
# Copier le token affiché dans le terminal Docker et le coller dans le navigateur
```
```yaml
# Arrêter tous les conteneurs en cours et arrêtés
$ docker stop $(docker ps -a -q)

# Supprimer tous les conteneurs
$ docker rm $(docker ps -a -q)

# Supprimer toutes les images Docker
$ docker rmi $(docker images -a -q)

# Supprimer uniquement les images non utilisées par aucun conteneur
$ docker image prune -a

# Nettoyer toutes les ressources Docker inutilisées pour libérer de l’espace disque
$ docker system prune
```
