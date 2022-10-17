# KUBERNETES

Déployement d'une application de vote chien/chat avec une base de données Redis au sein d'un cluster kubernetes


## Déployement du AKS de Azure

        az aks create -g ressourcegroup -n clusteraks --ssh-key-value ./.ssh/id_rsa.pub --node-count 2
        
Le service AKS d'azure permet de déployer l'ensemble des éléments permettant l'administration des nodeset des pods gérés avec kubernetes :
- un load balancer
-  un node qui contient :
      - un pod avec l'application
      - un pod avec redis.

Une fois le cluster déployé, il faut le connecter avec l'environnement de travail en utilisant la commande :

          az aks get-credentials -n clusteraks -g ressourcegroup
         
    

---

## Container Redis
- création manifest kubernetes: déploiement container faisant tourner redis grace kubectl apply -f redis.yaml
- définition des variables d'environnement pour utiliser une connection par mot de passe entre l'application et redis
- Le service redis est définit en ClusterIP

---

## Container Voting App
-  création manifest kubernetes: déploiement container de l'application : kubectl apply -f voteApp.yaml
-  Sans la gateway, le service attaché à l'application est de type "loadbalancer" pour lui attribuer une IP publique et la rendre accessible de l'extérieur. Ce même service est changé en "ClusterIp" lors de l'utilisation de la gateway pour que l'application ne soit accessible qu'au travers de la gateway.
- définition des variables d'environnement pour utiliser le mot de passe de redis.

---

## Password Redis
- création secret kubernetes avec un manifest de type secret kubectl apply -f secret.yaml
- le secret en encodé en base64 : bash echo -n 'password' |base64

---

## Stockage Redis
- Création manifest permettant le déploiement d'un storage class pour un file share afin de rendre les données de redis persistantes : storage.yaml
- un autre manifest persistant volume claim permet d'allouer une partie du stockage déployé.
- Ajout des spécifications au manifes redis.yaml pour lui allouer ce stockage et monter les données vers ce point.
- suppression et redéployement de redis, les données sont concervées.

---

## Application gateway avec AKS
- création avec az cli d'un cluster AKS avec l'add on AGIC: 

                az aks create -g ressourcegroup -n clusteraks --ssh-key-value ./.ssh/id_rsa.pub --node-count 4 --enable-managed-identity -a ingress-                     appgw --appgw-name Applicationgateway --appgw-subnet-cidr "10.225.0.0/16"
                
- Je crée un manifest créant une ressource ingress qui route les arrivées sur l'ip publique de la gateway vers l'app.

---

## Nom de domaine
- Modification des enregistrements DNS pour diriger le sous domaine vote.simplon-lion.space vers l'adresse IP public de la gateway du cluster kubernetes. 

---

## Certificat TLS pour App
- je crée un certificat avec le client certbot de gandi : sudo certbot certonly --authenticator dns-gandi --dns-gandi-credentials .../gandi.ini -d domain.name
- j'importe le certificat via un secret kubernetes que j'inclus dans le manifest d'ingress pour qu'il soit implémenté à chaque déploiement.

---

## Scaling
- modification de la variable replicas à 2, création manifest de type HorizontalPodAutoscaler configuré avec les règles imposées en ajoutant des specs régulant l'utilisation du cpu par l'app.
- Je réutilise le script du brief 4 et on constate le scale out des pods et leur scale in une fois la charge passée.

---

# Fonctionnement de Kubernetes
- Système de gestion de conteneurs
- automatise les processus
- cluster k8S --> ensemble de nodes

---

- control plan et data plan
- nodes gèrent des pods déployant des conteneurs en leur sein
- fonctionne sur la base d'un état défini et un réel
- contrôleurs gèrent l'état du cluster pour qu'il corresponde à l'état souhaité
