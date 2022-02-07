# PROJET INFRA

## Lien vers le sujet : https://gitlab.com/it4lik/b3-projet-2021/-/blob/main/projet.md

# PRESENTATION DU PROJET

## **SUJET DU PROJET**

Serveur nextcloud qui stocke des fichiers (films / séries / photos / cours...).

Le serveur tourne sous Kubernetes et Docker. 

Le serveur doit être accessible depuis l'extérieur (lien https avec nom de domaine).

Les utilisateurs doivent pouvoir avoir accès aux serveurs depuis le lien htpps via un compte personnel sur l'application, et doivent pouvoir accéder et utiliser les différentes données partagées.

##  **DESCRIPTION DU PROJET** 

Les serveurs seront créer et gérer via un ESX sous Vmware.

- **Liste des serveurs mis en place :**
    - **vm_k8s** (serveur master + supervision) : ce serveur servira de Master pour Kubernetes et hébergera aussi le système de supervision (Prometheus + Grafana)
    - **srv_nextcloud** (serveur d'application) : ce serveur hébergera l'application Nextcloud, et sera un node Worker dans le Cluster Kubernetes
    - **srv_bakcup** (serveur de sauvegarde) : ce serveur servira de backup, il stockera les fichiers de conf des serveurs, ainsi que des dossiers/fichiers spécifiques (choisis) présent sur le serveur Nextcloud

- **Base de données (MariaDB) :**
    - Pour le serveur Master (vm_k8s) : petite base de données pour stocker les logs des utilisateurs pour pouvoir identifier les actions/commandes passées.
    - Pour le serveur Nextcloud (srv_nextcloud) : gestion d'utilisateurs ayant un accès restreint et spécifique sur le Nextcloud 


- **Monitoring (Prometheus + Grafana + Loki) :** a détailler une fois mis en place 
Ajout de dashboards Grafana personnalisés pour chaque serveur

    - **Alerting :**  
Mise en place d'envoi de notification (via métriques Grafana) 
        - Chaque jour : envoi d'une notification avec l'état général de tous les serveurs 
        - Dès qu'un problème est détecté sur un ou plusieurs serveurs : envoi d'une notification (plusieurs fois si possible)
        - Dès qu'un serveur d'approche d'un seuil d'alarme (ex : espace de stockage saturée, charge CPU...) : envoi de message d'avertissement 


- **Backup :** 
    - Sauvegarde incrémentale des données présentent sur le serveurs Nextcloud
    - Sauvegarde des différentes configuration des serveurs et des logs

- **Sécurité :** 
    - Filtre des flux autorisés via le fw des serveurs
    - Accès aux serveurs restreint (+ droits bien spécifiques et défini pour les utilisateurs)

- **Automatisation (déploiement de scripts) :**
    - redémarrage automatique des services en cas d'arrêt + envoi de mail d'alerte si le service reste down. 
    - Notification envoyé aux utilisateurs lors de l'ajout / suppression d'un ou plusieurs fichiers. 

## **SUIVI DU PROJET** 

Le suivi/description des tâches ainsi que leurs dates de réalisation se fera via des cartes **Trello**.

Elles seront mises à jour pendant et/ou après chaque fin de séance. 

Elles contienneront un suivi global et assez représentatif de l'avancement/l'avancée du projet. 

Lien Trello : https://trello.com/b/lJonT4xD/suivi-projet-infra


## DOCUMENTATIONS 

Vous trouverez aussi dans ce repositorry : 
    - la documentation d'installation 
    - un schéma de représentation de la solution 
    - un guide d'utilisation de la solution pour un utilisateur final

    
