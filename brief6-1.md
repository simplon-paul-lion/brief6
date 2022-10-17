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
         
    

