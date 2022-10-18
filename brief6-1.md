---
description: View the slide with "Slide Mode".
slideOptions:
    theme: black
    transition: 'concave'
    backgroundTransition: 'convex'
    transitionSpeed: 'default'
    parallaxBackgroundImage: 'https://github.com/simplon-paul-lion/brief_6/fond.jpg'
---
  <!-- .slide: data-background="https://cncf-branding.netlify.app/img/projects/kubernetes/stacked/black/kubernetes-stacked-black.png" data-background-size="1500px" -->
# Brief 6: executive summary
[github du brief](https://github.com/simplon-paul-lion/brief_6)
[résultat](http://vote.simplon-lion.space)
[présentation](https://hackmd.io/p/pxfhrvbSSHORAlPZGTul7Q)

---



# KUBERNETES

Déployement d'une application de vote chien/chat avec une base de données Redis au sein d'un cluster kubernetes

---

# Fonctionnement de Kubernetes

Kubernetes fonctionne sur la base d'un état défini et un état réel. Le contrôle plane gèrent l'état du cluster pour qu'il corresponde à l'état souhaité

---

- Orchestrateur : 
   - permet d'administrer un ensemble de conteneurs de manière centralisée
   - permet de prendre en compte les besoins et spécificitées de chaque conteneur
   - permet d'organiser les conteneurs en fonction de leur affinité

---

## le cluster K8s

- 1 le controle plane
- 
    c'est le controle plane qui reçoit les instruction décrivant l'infrastructure désirée et qui met le cluster en conformité avec ce qui lui a été déclaré.
    
---

## le cluster K8s
    
- 2 le data plane
   
   Il est composé des différents éléments qui ont été déclarés au controle plane.
   Il peut compremdre : 
       - les nodes qui vont héberger les pods, les kube proxy, etc ...
       - les pods hébergent les instances des conteneurs
       - les loadbalancers, les stockages persistants, les keyvaults, etc ...

---


## Déployement du AKS de Azure

    az aks create -g ressourcegroup -n clusteraks --ssh-key-value ./.ssh/id_rsa.pub --node-count 2
        
Le service AKS d'azure permet de déployer l'ensemble des éléments permettant l'administration des nodes et des pods gérés avec kubernetes :
- un load balancer
- un node qui contient :
      - un pod avec l'application
      - un pod avec redis.

---


 ## connection cluster

Une fois le cluster déployé, il faut le connecter avec l'environnement de travail en utilisant la commande :

    az aks get-credentials -n clusteraks -g ressourcegroup
         
    

---

## Container Redis

- déploiement container faisant tourner redis         kubectl apply -f redis.yaml
- définition des variables d'environnement pour mot de passe entre l'application et redis
- Le service redis est définit en ClusterIP

---

## Container VoteApp

- déploiement du container de l'application          kubectl apply -f voteApp.yaml
-  Sans la gateway: le service de voteApp est de type "loadbalancer" => IP publique.
-  Avec gateway Ce même service est changé en "ClusterIp" => voteApp accessible qu'au travers de la gateway.
- définition des variables d'environnement pour utiliser le mot de passe de redis.

---

## Password Redis

- création secret kubernetes avec un manifest de type secret kubectl apply -f secret.yaml
- le secret en encodé en base64 : bash echo -n 'password' |base64

---

## Stockage Redis

- déclaration d'un storage class pour un file share afin de rendre les données de redis persistantes.
- déclaration d'un persistant volume claim pour allouer le stockage à Redis.
- Ajout des spécifications à redis.yaml pour lui allouer ce stockage et monter les données vers ce point.
- suppression et redéployement de redis, les données sont concervées.

---

## Application gateway avec AKS

- création avec az cli d'un cluster AKS avec l'add on AGIC: 

                az aks create -g ressourcegroup -n clusteraks --ssh-key-value ./.ssh/id_rsa.pub --node-count 4 --enable-managed-identity -a ingress-appgw --appgw-name Applicationgateway --appgw-subnet-cidr "10.225.0.0/16"
                
- déclaration d'un ingress qui route les requêtes de l'ip publique de la gateway vers l'app.

---

## Nom de domaine

- Modification des enregistrements DNS pour diriger le sous domaine vote.simplon-lion.space vers l'adresse IP public de la gateway du cluster kubernetes. 

---

## Certificat TLS pour App
- création du certificat avec le client certbot de gandi : sudo certbot certonly --authenticator dns-gandi --dns-gandi-credentials .../gandi.ini -d domain.name
- déclaration du certificat dans un secret kubernetes
- spécification du certificat TLS dans ingress pour qu'il soit utilisé par la gateway

---

## Scaling

- voteApp : replicas à 2
- déclaration d'un type HorizontalPodAutoscaler avec les specs régulant l'utilisation du cpu par l'app.
- test de charge pour scale out et scale in

---

 
