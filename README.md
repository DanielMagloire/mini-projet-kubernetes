# mini-projet-kubernetes
Déploiement de l'application WordPress avec persistance des données

Pour réaliser notre mini projet, il est important de connaitre que l'application WordPress est structurée en deux parties:

- Une partie front end (IHM) 
- Une partie backend (Base de données)

## Indications littérales

Pour ce faire, nous devons:
- Déployer un **pod** Mysql.
- Ce pod Mysql sera dans un objet de type **deployment**.
- Nous allons définir un seul **replicat**.
- Le pod Mysql aura en frontal un service de type **clusterIP** qui permettra d'exposer le **pod mysql** en interne dans k8s.
- Tout ceci sera couplé à un **pod WordPress**.
- Le **pod WordPress**, créé sera lui aussi dans un objet de type **deployment**.
- A ce **pod WordPress** sera apposé un service de type **nodeport**. Ce service sera le point d'entrée de l'application WordPress.
- Notre application WordPress viendra interroger l'application Mysql en passant par le service **clusterIP**.

## Persistance des données

L'application Mysql devra avoir un stockage persistant pour stocker ses données.

Ici, le volume sera fait comme dans le cas de docker. Ce volume sera mis dans le dossier **data** et dans le répertoire mysql (**/data/mysql**)

Ce qui sera stocker dans ce volume est /var/lib/mysql
