### LIEN DU GITHUB 

https://github.com/ClementGRC/E4_AWSORC_TP/tree/main/TP_Final_GRECO_AWSORC


## 🗒️ Introduction : 

Ce projet a pour objectif de déployer une infrastructure AWS pour un client souhaitant héberger une application web connectée à une base de données, un serveur de test, ainsi qu’un hébergement statique pour la documentation. La solution doit être sécurisée, scalable et économique. Une sauvegarde unique doit également être mise en place. Dans un second temps, un environnement isolé pour une équipe IA doit être ajouté.

# Modélisation

Afin de réaliser notre projet de manière efficace, nous avons tout d'abord mis au point cette modélisation qui nous permet d'obtenir une vue d'ensemble de l'architecture à établir :

![alt text](<TP_AWS - Cadre 1-1.jpg>)


# Features utilisées :

⚙️ Les différentes fonctionnalités utilisées dans l’infrastructure AWS
Pour réaliser ce projet, nous avons utilisé plusieurs services AWS essentiels, chacun jouant un rôle clé dans le bon fonctionnement de l’infrastructure :

## 🗄️ Amazon RDS (Relational Database Service)
Rôle : Héberger la base de données MySQL de l’application web.
Avantages : Ce service gère automatiquement les tâches d’administration telles que les sauvegardes, les mises à jour et la reprise après incident. Il garantit également une haute disponibilité et une mise à l’échelle facile, ce qui répond au besoin de performance du client.


## 📦 Amazon S3 (Simple Storage Service)
Rôle : Stocker et héberger de manière statique la documentation du projet.
Avantages : S3 permet de stocker des fichiers de manière sécurisée, avec la possibilité de les rendre accessibles publiquement grâce à l’activation du mode “Static Website Hosting”. Ce service est économique et évolutif, idéal pour un MVP.


## 🌐 NAT Gateway
Rôle : Permettre aux serveurs situés dans les sous-réseaux privés d’accéder à Internet tout en les gardant inaccessibles depuis l’extérieur.
Avantages : Ce service garantit la sécurité des ressources en évitant de leur attribuer une adresse IP publique tout en leur offrant la possibilité de télécharger des mises à jour ou de communiquer avec des services externes.

## 💾 Base de données MySQL
Rôle : Stocker et gérer les données de l’application web.
Avantages : MySQL est une base de données relationnelle populaire, connue pour sa fiabilité et ses performances. Son intégration avec AWS RDS facilite la gestion, la sauvegarde et la mise à l’échelle.

## 💻 Amazon EC2 (Elastic Compute Cloud)
Rôle : Héberger l’application web et le serveur de test.
Avantages : EC2 offre des serveurs virtuels configurables selon les besoins. Il permet de déployer rapidement une application et de la rendre accessible sur Internet grâce à une IP publique. L’utilisation du fichier Cloud-init a permis d’automatiser l’installation des paquets nécessaires (Docker, Git, etc.) et de cloner directement le projet.

## 🧩 Internet Gateway
Rôle : Permettre aux sous-réseaux publics de communiquer directement avec Internet.
Avantages : Ce service est essentiel pour rendre l’application web accessible depuis l’extérieur et pour permettre au NAT Gateway de fonctionner.

## 💾 AWS Backup
Rôle : Automatiser les sauvegardes des instances EC2 et de la base de données.
Avantages : AWS Backup simplifie la gestion des sauvegardes, assure la récupération des données en cas de panne et garantit la continuité du service.

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



## Création EC2 

# Fichier Cloud init 
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
## Optimisation de l'image du DockerFile

![alt text](image.png)

## Optimisation docker-compose.yml 

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


### ✅ **Conclusion**  

Ce projet AWS nous a permis de mettre en place une infrastructure cloud sécurisée, évolutive et économique, répondant aux besoins du client. En choisissant un seul VPC pour le MVP, nous avons réduit les coûts tout en gardant une gestion réseau simple et efficace. La séparation en sous-réseaux publics et privés garantit la sécurité des services tout en assurant de bonnes performances.  

L’application web, connectée à une base de données, ainsi que le serveur de test et l’hébergement statique via S3 ont été déployés en suivant les bonnes pratiques AWS. Nous avons également mis en place des sauvegardes automatisées pour assurer la fiabilité de l’ensemble.  

Cependant, faute de temps, nous n'avons pas pu finaliser la partie 2 du projet, qui consistait à créer un environnement isolé pour l’équipe IA. Malgré cela, l’infrastructure mise en place constitue une base solide, capable d’évoluer pour répondre aux futurs besoins du client.

