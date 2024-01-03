# HumansBestFriend app? CATs or DOGs ?

## Investigateurs

- Mayane MAMANE
- Maxime JANEZ
- Geoffroy RODRIGUEZ


## Introduction
Le projet HumansBestFriend consiste en la conception d'une infrastructure de vote virtuelle, en utilisant les concepts de virtualisation et de conteneurisation.

Nous voulons créer une application permettant de voter en temps réel pour le chien ou pour le chat, et afficher les résultats.

Pour ce faire, nous créons un environnement virtualisé avec VMware ESXI pour exécuter une machine virtuelle Ubuntu et utiliser des conteneurs Docker.


## Prérequis

Pour réaliser cette application, nous avons besoin d'une machine virtuelle ESXI sur VMware qui permet d'utiliser Docker et docker compose.
L'installation de ces outils est détaillée dans le rapport du projet.


## Technologies et composants utilisés

1. Front-End Web App en Python pour Voter : Cette application offre une interface utilisateur conviviale permettant aux utilisateurs de
    voter entre deux options, chats et chiens, dans un environnement web. 

2. Redis pour la Collecte des Votes : Utilisé comme système de stockage clé-valeur, Redis joue un rôle essentiel dans la collecte instantanée
    et la gestion des nouveaux votes entrants. 

3. Travailleur .NET pour la Gestion des Votes : Ce composant consomme les votes enregistrés et les stocke dans une base de données PostgreSQL,
    assurant ainsi la persistance des données de manière efficace et fiable. 

4. Base de Données PostgreSQL avec Volume Docker : PostgreSQL est utilisé comme système de gestion de base de données relationnelle,
    intégré dans un conteneur Docker avec un volume dédié pour garantir la préservation des données. 

5. Application Web Node.js pour Afficher les Résultats en Temps Réel : Cette application présente les résultats du vote de manière dynamique
    et en temps réel, offrant une expérience utilisateur enrichie.

   

## Création des fichiers docker-compose.build.yml et docker-compose.yml

D'abord nous clonons le projet sur notre machine Ubuntu: $ git clone https://github.com/pascalito007/esiea-ressources.git

1. Création du fichier docker-compose.build.yml.
   
Ce fichier gère la création des images d'application à partir du contenu du fichier Docker fourni.
Il permet de décrire et d'orchestrer les différentes parties de l'application, simplifiant ainsi le processus de déploiement et de gestion des conteneurs.

Exécution du fichier: $ docker-compose -f docker-compose.build.yml build 
Vérifier si les images sont bien lancées: $ docker-compose -f docker-compose.build.yml up 

2. Création du fichier docker-compose.yml

Ce fichier permet le lancement de l'application.
Il définit les paramètres pour démarrer les conteneurs de manière cohérente et pour interconnecter les différents services d'une application,
ce qui facilite le déploiement et la gestion de l'application dans différents environnements, comme le développement, le test et la production.  

La commande suivante permet de démarrer, d’exécuter et de connecter entre eux tous les services du fichier “docker-compose.yml“, 
simplifiant ainsi le déploiement et la gestion d'une application complexe:

$ docker-compose up

Elle extrait d'abord les données puis crée les conteneurs.



## Lancement de l'application depuis l'URL de recherche


### For `vote` service

- map a volume at `/usr/local/app` that that is inside the container. This need to be a bind mount for example

```shell
volumes:
     - ./vote:/usr/local/app
```

- The `vote` service listen on port `80`. Feel free to expose for example `5002` outside.

- The `vote` service need to be inside two networks `front-tier` and `back-tier`
  ad below option for `vote`

```shell
healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s
```

- Below is the Dockerfile of `vote` service

```shell
# Define a base stage that uses the official python runtime base image
FROM python:3.11-slim AS base

# Add curl for healthcheck
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Set the application directory
WORKDIR /usr/local/app

# Install our requirements.txt
COPY requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Define a stage specifically for development, where it'll watch for
# filesystem changes
FROM base AS dev
RUN pip install watchdog
ENV FLASK_ENV=development
CMD ["python", "app.py"]

# Define the final stage that will bundle the application for production
FROM base AS final

# Copy our code from the current folder to the working directory inside the container
COPY . .

# Make port 80 available for links and/or publish
EXPOSE 80

# Define our command to be run when launching the container
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]
```

The `vote` app will be running at [http://localhost:5002](http://localhost:5002), and the `results` will be at [http://localhost:5001](http://localhost:5001).

### For `seed-data`

- Note: add for `seed`

```shell
profiles: ["seed"]
    depends_on:
      vote:
        condition: service_healthy
    restart: "no"
```

- It need to be inside `front-tier` network

- Below the Dockerfile

```shell
FROM python:3.9-slim

# add apache bench (ab) tool
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    apache2-utils \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /seed

COPY . .

# create POST data files with ab friendly formats
RUN python make-data.py

CMD /seed/generate-votes.sh
```

### For `result`

- Inside the container the port is `80`. Feel free to expose outside with for example `5001`
  I strongly recommands you use below:

```shell
entrypoint: nodemon --inspect=0.0.0.0 server.js
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./result:/usr/local/app
    ports:
      - "5001:80"
      - "127.0.0.1:9229:9229"
```

- Below the Dockerfile for result

```shell
FROM node:18-slim

# add curl for healthcheck
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl tini && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /usr/local/app

# have nodemon available for local dev use (file watching)
RUN npm install -g nodemon

COPY package*.json ./

RUN npm ci && \
 npm cache clean --force && \
 mv /usr/local/app/node_modules /node_modules

COPY . .

ENV PORT 80
EXPOSE 80

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "server.js"]
```

2. The images that are build need to be published first in your publish docker registry and then in a private registry. Please make sure you have a frontend for the private registry as we saw in the course.

3. Create another file called `compose.yml` and it will be responsible of deploying the application and all the needed containers. Please make sure that the images are referencing the one in your private registry.

### For postgres `db`

- It need to be inside `back-tier` network
- Please make use of below in your compose file. Please use `postgres:15-alpine` for the postgres image

```shell
    volumes:
      - "db-data:/var/lib/postgresql/data"
      - "./healthchecks:/healthchecks"
    healthcheck:
      test: /healthchecks/postgres.sh
      interval: "5s"
```

- for the volume `db-data` make sure you create it in your compose file

### For `redis` service

Please make use of below in your compose file.

```shell
      - "./healthchecks:/healthchecks"
    healthcheck:
      test: /healthchecks/redis.sh
      interval: "5s"
```

- It need to be inside `back-tier` network

## Architecture

![Architecture diagram](architecture.png)

- A front-end web app in [Python](/vote) which lets you vote between two options
- A [Redis](https://hub.docker.com/_/redis/) which collects new votes
- A [.NET](/worker/) worker which consumes votes and stores them in…
- A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
- A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The `HumansBestFriend` application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.


