---
title: "Demo"
description: "$ docker"
draft: false
weight: 2
---
### Comamndes utilisées

```yaml
# Télécharger l’image Ubuntu depuis Docker Hub
$ docker pull ubuntu

# Lister toutes les images Docker disponibles localement
$ docker image ls

# Créer et exécuter un conteneur Ubuntu en mode interactif
$ docker run -it ubuntu
# Lister les fichiers dans le conteneur (dans le shell root du conteneur)
root$ ls

Ouvrez un autre terminal Shell :
--------------------------------
# Lister tous les conteneurs (actifs et arrêtés)
$ docker ps -a        

# Arrêter un conteneur en cours d’exécution (remplacer container_name par le nom du conteneur)
$ docker stop container_name

# Supprimer un conteneur (remplacer container_name par le nom du conteneur)
$ docker rm container_name

# Télécharger l’image Ghost depuis Docker Hub
$ docker pull ghost   

# Lancer un conteneur Ghost en arrière-plan avec des variables d’environnement et mappage de port
$ docker run -d --name ghost1 -e NODE_ENV=development -e url=http://localhost:3001 -p 3001:2368 ghost

Ouvrez un navigateur Internet :
-------------------------------
# Accéder à l’application Ghost via le navigateur
http://localhost:3001/

# Lister tous les conteneurs (actifs et arrêtés)
$ docker ps -a  

# Arrêter le conteneur Ghost nommé ghost1
$ docker stop ghost1

# Supprimer le conteneur Ghost nommé ghost1
$ docker rm ghost1
```

