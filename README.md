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

Dans le **Pod Mysql**, nous pouvons aussi associer un volume simple qui sera stocké dans **/data/wordpress**

Ce qui sera stocké à l'intérieur de ce volume sera le contenu **/var/www/html** de notre pod frontal.

Tous ces objets ainsi que toutes les ressources seront dans un **Namespace** appelé **WordPress**. 

Afin que l'appication soit consommée par une personne venant de l'extérieure, celle-ce sera obligée de passer par le **service nodeport**.


## Illustration graphique de notre solution



## Réalisation du mini projet

### Les outils utilisés et fichiers du projet

- Visual Studio Code
- L'hyperviseur Oracle VM VirtualBox
- kubernetes (Orchestrateur)
- Vagrantfile pour créer notre VM kunernetes
- Un fichier de provisionning qui va automatiser l'intallation des outils nécessaire dans la VM
- Les manifestes de deployement (WordPress et Mysql)
- Les manifestes des services (clusterip et nodeport)
- Les manifestes du namespace et du secret
- Le docker-compose pour automatiser l'installation de Mysql

### Mise en place de la VM Kubernetes

1. On lance la VM
```
$ vagrant up --provision
```

2. Je me connecte à ma VM minikube via ssh
```
$ vagrant ssh
```
3. Je démarre minikube
```
$ minikube start --driver=none
```
4. Je vérifie s'il exite dans la VM les pods, nodes ou deployment qui tournent par hazard
```
$ kubectl get po
```
```
$ kubectl get node
```
Nous devons avoir le node **minikube** en statut **Ready** comme le rendu ci-dessous
```
NAME       STATUS   ROLES                  AGE   VERSION
minikube   Ready    control-plane,master   30s   v1.23.3
```
```
$ kubectl get deploy
```
Pas de **Pods** ni de **deploy**, nous passons à phase de test.

5. Lancer le déploiement
- Je me positionne dans le repertoire où se trouve les manifestes
```
$ cd [nom_dossier] 
$ cd mini-projet-k8s
```
- Dans le répertoire, je lance la commande ci-dessous pour créer tous les objets
```
$ kubectl -f ./
```
- Résultat de la création des objets avec succès
```
[vagrant@minikube mini-projet-k8s]$ kubectl apply -f ./
namespace/wordpress created
secret/app-wordpress-secret created
deployment.apps/wp-mysql created
service/wp-mysql created
service/wordpress created
deployment.apps/wordpress created
````

6. Visualisons les ressources créées
```
- Je visualise le namespace
$ kubectl get namaspace
```
```
NAME              STATUS   AGE
default           Active   39m
kube-node-lease   Active   39m
kube-public       Active   39m
kube-system       Active   39m
wordpress         Active   18m
```
Nous avons le **namespace** qui a été créé

- Je viualise les **pods** du namespace qui sont **wordpress** et **mysql** et s'ils sont en statut **Running**
```
$ kubectl get pod -n wordpress
```
```
NAME                        READY   STATUS    RESTARTS   AGE
wordpress-d546d667-rbq5j    1/1     Running   0          22m
wp-mysql-84bbbdc74f-zjj6p   1/1     Running   0          22m
```
- Je visualise que mes deux **pods** sont bien dans un **deployment**
```
$ kubectl get deploy -n wordpress
```
```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
wordpress   1/1     1            1           27m
wp-mysql    1/1     1            1           27m
```

- Je vérifie que mes deux services sont bien en place
```
$ kubectl get svc -n wordpress
```
```
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
wordpress   NodePort    10.101.105.90    <none>        80:30008/TCP   30m
wp-mysql    ClusterIP   10.111.119.113   <none>        3306/TCP       30m
```
Nous avons bien nos deux services et leurs types **NodePort** et **ClusterIP**

Le service **NotePort** est visible sur le port 30008 en TCP

- Je vais regarder mes secret
```
$  kubectl get secret -n wordpress
```
```
NAME                   TYPE                                  DATA   AGE
app-wordpress-secret   Opaque                                3      34m
default-token-nsq5d    kubernetes.io/service-account-token   3      34m
```

Nous pouvons conclure que tout est opérationnel

## Test au navigateur

Pour ce faire, je récupère l'ip de ma machine qui a été fourni par Vagrant
```
$ ip a
```
Puis je vais taper cette ip dans le navigateur en **http://** sur le port **30008** de mon **NodePort**
```
http://192.168.56.29:30008
```
### Visaulisation de l'application WordPress

Schéma

### Vérification du stockage
```
$ cd /data/
```
```
$ ll
```
```
[vagrant@minikube data]$ ll
total 8
drwxr-xr-x. 6 polkitd root 4096 Apr  4 20:35 mysql
drwxr-xr-x. 5      33 tape 4096 Apr  4 20:45 wordpress
```
Nous avons belle et bien les répertoires mysqi et wordpress qui ont été créé

Ce qui me reste à faire c'est de finaliser mon installation de WordPress via le navigateur en choissant la langue et en cliquant sur **continuer**.

Puis je reseigne les informations demandées:
- Titre
- identifiant
- mot de passe
- Votre e-mail
- Cliquer sur le bouton**Installer WordPress**


***********************************************************************

# Commandes impératives
1. Pour créer le namespace
```
kubectl create namespace [nom_namespace] --dry-run=client -o yaml
```
On prend la sortie que nous mettons dans le **manifeste** puis on l'exécute

```
kubectl apply -f namespace.yaml
```
2. Pour créer le *deployment* du pod *db_MySql*

```
kubectl create deploy db_MySql --image mysql:5.7 --replicas=1 --dry-run=client -o yaml > deploy_mysql.yaml
```
Quand je valide la commande, j'ai le manifeste qui est directement créé.

3. Pour créer le *deployment* du pod *Wordpress*

```
kubectl create deploy Wordpress --image mysql:5.7 --replicas=1 --dry-run=client -o yaml > deploy_wordpress.yaml
```
Quand je valide la commande, j'ai le manifeste qui est directement créé.
