# HumansBestFriend app? CATs or DOGs ?

## Investigateurs

- Mayane MAMAN
- Maxime JANEZ
- Geoffroy RODRIGUEZ


## Introduction
Le projet HumansBestFriend consiste en la conception d'une infrastructure de vote virtuelle, en utilisant les concepts de virtualisation et de conteneurisation.

Nous voulons créer une application permettant de voter en temps réel pour le chien ou pour le chat, et afficher les résultats.


## Prérequis

Pour réaliser cette application, nous devons créer un environnement virtualisé à l'aide de VMware ESXi pour exécuter une machine virtuelle Ubuntu qui permetra d'utiliser Docker et docker compose.
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

## Architecture

![Architecture diagram](architecture.png)
   

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

À présent, ouvrons notre navigateur et saisissons l'URL de recherche que nous avons définie antérieurement dans le fichier "docker-compose.yml" que nous avons rédigé : adresseIP:port 
Nous arrivons alors sur l’interface où nous pouvons choisir notre animal de compagnie favori.


## Lancer l'application avec Kubernetes

La mise en place d'un cluster Kubernetes permet une gestion avancée des conteneurs et renforce la flexibilité ainsi que la robustesse de notre infrastructure.

Après l'installation de kubernetes et de kubectl, nous utilisons Minikube qui permet de mettre en place un cluster Kubernetes local sur une machine individuelle.
$ minikube start  

Les fichiers yaml pour lancer l’application dans un cluster Kubernetes se trouvent dans notre répositoire k8s-specification. 

On instancie les déploiements à partir des fichiers YAML : $ kubectl create -f k8s-specifications/ 
Vérifions que les configurations ont bien été déployés :  $ kubectl get deployments       et     $ kubectl get pods  

Notre cluster kubernetes fonctionne, on passe par l'IP du cluster de Minikube pour accéder à l'application:
$ minikube ip  
    192.168.58.2






