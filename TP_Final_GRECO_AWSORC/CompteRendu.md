
## 🗒️ Introduction : 

Ce projet a pour objectif de déployer une infrastructure AWS pour un client souhaitant héberger une application web connectée à une base de données, un serveur de test, ainsi qu’un hébergement statique pour la documentation. La solution doit être sécurisée, scalable et économique. Une sauvegarde unique doit également être mise en place. Dans un second temps, un environnement isolé pour une équipe IA doit être ajouté.


## 📋 Sommaire
1. [Introduction](#introduction)
2. [Architecture de l’infrastructure](#architecture-de-linfrastructure)
3. [Partie 1 : Déploiement de l’infrastructure](#partie-1--déploiement-de-linfrastructure)
   - [VPC et sous-réseaux](#vpc-et-sous-réseaux)
   - [Application web](#application-web)
   - [Base de données](#base-de-données)
   - [Serveur de test](#serveur-de-test)
   - [Hébergement statique](#hébergement-statique)
   - [Sauvegardes](#sauvegardes)
4. [Partie 2 : Ajout de l’environnement IA](#partie-2--ajout-de-lenvironnement-ia)
5. [Repository Git](#repository-git)
6. [Diagramme de l’infrastructure](#diagramme-de-linfrastructure)
7. [Conclusion](#conclusion)


# Modélisation

Afin de réaliser notre projet de manière efficace, nous avons tout d'abord mis au point cette modélisation qui nous permet d'obtenir une vue d'ensemble de l'architecture à établir :

![Alt text](ModélisationInfra.jpg)

_En cas de difficulté, vous pouvez ouvrir les fichiers images directement, celles-ci sont présentes dans le répertoire._

# 🌐 Partie 1 : Déploiement de l’infrastructure

VPC et sous réseaux : 

## VPCs

L'utilisation d'un seul VPC (Virtual Private Cloud) permet de réduire les coûts en évitant les frais liés à plusieurs VPC, tout en simplifiant la gestion et la configuration du réseau. La sécurité est assurée grâce à des sous-réseaux privés pour la base de données, le serveur de test et l’environnement IA, qui ne sont pas accessibles directement depuis Internet. Cette approche garantit également de bonnes performances et une évolutivité adaptée aux besoins du client, tout en respectant les contraintes d’un MVP.

| VPC Name | CIDR Block   | VPC ID               |
|----------|--------------|----------------------|
| VPC      | 10.0.0.0/16  | vpc-0fc433c41a5c9e2b6

## Sous réseaux 

Les sous-réseaux sont organisés en fonction de leur rôle et répartis sur deux zones de disponibilité (**east-1a** et **east-1b**) pour assurer une meilleure disponibilité.  

- **Subnets publics** (10.0.0.0/20 et 10.0.16.0/20) : Ils hébergent l’application web et le NAT Gateway, ce qui leur permet de communiquer directement avec Internet.  
- **Subnets privés** (10.0.128.0/20 et 10.0.144.0/20) : Ils accueillent la base de données, le serveur de test et l’environnement IA. Ces sous-réseaux ne sont pas accessibles depuis Internet, mais peuvent tout de même y accéder pour des mises à jour grâce au NAT Gateway.

| **Subnet Name**            | **CIDR Block**       | **Subnet ID**                              |
|----------------------------|---------------------|------------------------------------------|
| Subnet public east-1a      | 10.0.0.0/20         | subnet-0fbf3b0e0cbcb4e34                  |
| Subnet public east-1b      | 10.0.16.0/20        | subnet-012145f447cae47d9                  |
| Subnet private east-1a     | 10.0.128.0/20       | subnet-0783fcdaf38de992c                  |
| Subnet private east-1b     | 10.0.144.0/20       | subnet-01c83fab94a708636                  |



![alt text](image.png)



## Création EC2 



# Fichier CLoud init 
```
#cloud-config

# Configurer le fuseau horaire
timezone: Europe/Paris

# Mise à jour des paquets
package_update: true
package_upgrade: true

# Installation des paquets nécessaires
packages:
  - git
  - docker
  - docker-compose

# Configuration de l'utilisateur et des groupes
groups:
  - docker
users:
  - default
  - name: ubuntu
    groups: docker
    shell: /bin/bash

runcmd:
  # Activer et démarrer Docker
  - systemctl enable --now docker

  # Ajouter l'utilisateur "ubuntu" au groupe Docker
  - usermod -aG docker ubuntu

  # Appliquer les changements immédiatement sans redémarrer (évite d'attendre un relog)
  - newgrp docker

  # Cloner l'application Django Volt Dashboard
  - git clone https://github.com/app-generator/flask-datta-able.git /home/ubuntu/flask-datta-able

```


## Optimisation de l'application 

Voici le fichier .gitignore qu'on a bien optimisé. 

```
Byte-compiled / optimized / DLL files
pycache/
*.py[cod]
*.pyo
*.pyd
*.so
*.dll

Tests and coverage
*.pytest_cache
.coverage
htmlcov/
nosetests.xml
coverage.xml
*.cover
*.py,cover
.cache
junitxml/
*.log
.tox/
.nox/
.envrc

Database & logs
.db
#.sqlite3
*.log
.sql

Virtual environments
env/
env_linux/
venv/
venv/
pip-log.txt
pip-delete-this-directory.txt

System files
.DS_Store
Thumbs.db
desktop.ini
*.swp
*.swo

Sphinx docs
_build/
_static/
_templates/

Node.js / JS dependencies
node_modules/
package-lock.json
yarn.lock
apps/static/assets/node_modules/
apps/static/assets/yarn.lock
apps/static/assets/.temp/

Migrations (optional, uncomment if you don't want to track migrations)
#migrations/

IDE and editor files
.vscode/
.vscode/symbols.json
.idea/
*.iml

Docker related
docker-compose.override.yml
.env
.env.local
.env.production
.env.test
.env.sample
.envrc

Nginx and deployment-related files
nginx/
*.crt
*.key
*.pem

Python-specific
pypackages/
Pipfile
Pipfile.lock
requirements.txt
pyproject.toml
setup.py
setup.cfg

Security and auth
*.pem
*.key
*.crt
*.pfx
*.p12

```
Optimisation de l'image du DockerFile

![alt text](image.png)

Optimisation docker-compose.yml 

![alt text](image-1.png)

## Connexion depuis le navigateur internet
Cette application est accessible par défaut sur le port 5085
![alt text](image-2.png)


## Création de la base de donnée 

![alt text](image-3.png)

On se rend dans le service RDS pour créer la Base de donnée MySQL 

Nous choisissons "l'offre gratuite"

![alt text](image-4.png)


Base de donnée avec une haute mise à l'échelle 

![alt text](image-5.png)




![alt text](image-6.png)



![alt text](image-7.png)


![alt text](image-8.png)

Nous passons donc à la création de la DB 

Autorisation du port : 

![alt text](image-9.png)

![alt text](image-10.png)



## Connexion à la base de donnée : 



![alt text](image-12.png)




## Création de User + visualisation de la table : 


![alt text](image-11.png)

![alt text](image-13.png)




## Création d'une autre instance 

![alt text](image-14.png)

On créé une passerelle NAT : 

![alt text](image-15.png)



Autorisation de l'adresse NAT dans les tables de routage : 

![alt text](image-17.png)

![alt text](image-16.png)

La connexion à internet fonctionne bien : 

![alt text](image-18.png)


##Création d'un bucket 

![alt text](image-19.png)

![alt text](image-20.png)


Activer les ACLs

![alt text](image-26.png)

![alt text](image-27.png)

![alt text](image-28.png)
![alt text ](image-25.png)

![alt text](image-29.png)

Sauvegarde / backup 

Pour l'EC2 

![alt text](image-30.png)

![alt text](image-31.png)

Pour la DB 

![alt text](image-32.png)
![alt text](image-33.png)

# Partie 2 

Création d'un nouveau VPC 

![alt text](image-34.png)

![alt text](image-35.png)

![alt text](image-36.png)

![alt text](image-37.png)

![alt text](image-38.png)

![alt text](image-39.png)

![alt text](image-40.png)

![alt text](image-41.png)


Creation de l'instance IA

![alt text](image-42.png)

![alt text](image-43.png)








## Internet Gateway
| Resource           | InternetGateway ID  |
|--------------------|---------------------|
| Internet Gateway   | igw-XXX|

## Route Tables
| Route Table Name | Route Table ID       |
|------------------|----------------------|
| RTB 1            | rtb-XXX|


