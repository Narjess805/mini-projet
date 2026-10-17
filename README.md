#  Mini-projet 1 : Déployer une application web à deux niveaux (Kubernetes / K3s)

![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.36.5-blue?logo=kubernetes)
![K3s](https://img.shields.io/badge/K3s-Lightweight-orange?logo=rancher)
![Traefik](https://img.shields.io/badge/Ingress-Traefik-24A1C1?logo=traefik)

##  Contexte du projet
Ce dépôt contient les manifestes YAML et la documentation du **Mini-projet 1**. 
L'objectif est de déployer une application à deux niveaux (un frontend et une API) sur un cluster Kubernetes (K3s), de les faire communiquer en interne, de les exposer à l'extérieur via un Ingress, et de démontrer une mise à jour sans coupure ainsi qu'un diagnostic de panne.

## Architecture cible
- **Frontend** : Nginx (2 réplicas)
- **API** : Podinfo (3 réplicas)
- **Routage** : Ingress (Traefik) sur `web.k3s.local` et `api.k3s.local`
- **Namespace** : `projet-narjess`

##  Structure du dépôt
```text
.
├── README.md                      # Ce fichier
├── manifests/                     # Manifestes Kubernetes
│   ├── api-deployment.yaml        # Deployment de l'API (Podinfo)
│   ├── frontend-deployment.yaml   # Deployment du Frontend (Nginx)
│   ├── services.yaml              # Services ClusterIP (api-svc et frontend-svc)
│   └── ingress.yaml               # Ingress pour l'accès externe
└── captures/                      # Captures d'écran du projet
    ├── 00_namespace.png
    ├── 01_api_pods.png
    ├── 02_frontend_pods.png
    ├── 03_services_communication.png
    ├── 04_ingress.png
    ├── 05_rolling_update.png
    ├── 06_diagnostic.png
    └── 07_nettoyage.png
 Guide de déploiement
1. Prérequis
Un cluster K3s fonctionnel (au moins 1 master et 1 worker).

kubectl configuré sur votre machine.

Traefik installé (inclus par défaut avec K3s).

2. Déployer l'application
Toutes les ressources sont décrites dans le dossier manifests/. Pour tout déployer d'un coup, exécutez la commande suivante depuis la racine du dépôt :

bash
kubectl create namespace projet-narjess
kubectl config set-context --current --namespace=projet-narjess
kubectl apply -f manifests/
3. Vérifier le déploiement
Vérifiez que tous les Pods sont en Running :

bash
kubectl get pods -o wide
Vérifiez que les Services sont bien créés avec leurs endpoints :

bash
kubectl get services
kubectl get endpoints
Vérifiez que l'Ingress a bien récupéré une adresse IP :

bash
kubectl get ingress
4. Tester l'accès externe
Ajoutez l'IP de votre nœud master dans le fichier hosts de votre machine :

text
192.168.56.20  web.k3s.local api.k3s.local
Accédez aux applications depuis votre navigateur ou avec curl :

Frontend : http://web.k3s.local

API : http://api.k3s.local

📸 Preuves d'exécution (Captures)
1. Création du Namespace
https://./captures/00_namespace.png

2. Déploiement de l'API (3 réplicas)
https://./captures/01_api_pods.png

3. Déploiement du Frontend (2 réplicas)
https://./captures/02_frontend_pods.png

4. Communication interne (Frontend -> API)
https://./captures/03_services_communication.png

5. Exposition avec Ingress
https://./captures/04_ingress.png

6. Mise à jour sans coupure (Rolling Update)
https://./captures/05_rolling_update.png

7. Diagnostic de panne
https://./captures/06_diagnostic.png

8. Nettoyage
https://./captures/07_nettoyage.png

🧠 Réponses aux questions de réflexion
Étape 0 : Pourquoi isoler un projet dans un namespace ?

Le namespace permet d'isoler les ressources (Pods, Services, etc.) pour éviter les conflits de noms, faciliter la gestion des droits (RBAC) et cibler les commandes plus facilement.

Étape 1 : Sur quels nœuds les Pods sont-ils répartis, et qui a pris cette décision ?

Les Pods sont répartis sur les différents nœuds (Master/Workers). C'est le Scheduler de Kubernetes qui prend cette décision en fonction des ressources disponibles et des contraintes.

Étape 2 : Que se passerait-il si les deux Deployments utilisaient exactement les mêmes labels ?

Les Services (et l'Ingress) ne sauraient plus quel Pod cibler. Un Service frontend-svc risquerait d'envoyer du trafic vers l'API (et inversement), créant un dysfonctionnement total.

Étape 3 : Quel est le nom DNS complet d'un Service ?

Le nom DNS complet (FQDN) est de la forme : <nom-du-service>.<nom-du-namespace>.svc.cluster.local. Exemple : api-svc.projet-narjess.svc.cluster.local.

Étape 4 : Pourquoi préfère-t-on un Ingress à deux Services de type NodePort ?

L'Ingress offre un point d'entrée unique (ports 80/443) avec routage par nom d'hôte et par chemin. Les NodePort nécessitent d'ouvrir un port différent pour chaque application, ce qui est moins sécurisé et moins lisible.

Étape 5 : Combien de ReplicaSets existent après la mise à jour, et pourquoi l'ancien est-il conservé ?

Il y a 2 ReplicaSets (un ancien à 0 réplicas, un nouveau à 3 réplicas). L'ancien est conservé pour permettre un rollback rapide (kubectl rollout undo) en cas de problème avec la nouvelle version.

Étape 6 : Quels symptômes vous ont mis sur la piste, et quelle est la cause racine ?

Symptômes : Les Pods restent en ImagePullBackOff/ErrImagePull.

Cause racine : Le tag de l'image spécifié dans le manifeste n'existe pas sur le registre (GHCR). Kubernetes ne peut donc pas la télécharger.

⚠️ Difficultés rencontrées
Erreur de label : Si les labels des Services ne correspondent pas à ceux des Deployments, les endpoints restent à <none>.

Délai de mise à jour : Les rolling updates prennent du temps si les images doivent être téléchargées depuis Internet.

Fichier hosts : Sous Windows, le fichier hosts doit être édité en tant qu'administrateur et sauvegardé sans extension.

💻 Auteur
Nom : [Dabbech]

Prénom : [Narjess]

Classe : [M2 Devops]

GitHub : @Narjess805

Date : Octobre 2026
