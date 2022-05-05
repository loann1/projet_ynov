## Création du secret 

```sh

kubectl create secret generic nextcloud-db-secret \
    --from-literal=MYSQL_ROOT_PASSWORD=admin \
    --from-literal=MYSQL_USER=nextcloud \
    --from-literal=MYSQL_PASSWORD=admin
```

## Deploiement des fichiers de conf 

```sh
kubectl apply -f nextcloud-shared-pvc.yml 
kubectl apply -f nextcloud-db.yml
kubectl apply -f nextcloud-server.yml 
kubectl apply -f cluster-ingress.yml 
```
