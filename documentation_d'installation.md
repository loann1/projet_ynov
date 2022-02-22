# DOCUMENTATION

## Sommaire

- [Documentation](#documentation)
  - [Sommaire](#sommaire)
  - [Serveur Master (vm_k8s)](#serveur_master_(vm_k8s))
    - [Installation Kubernetes](#installation_kubernetes)
    - [Installation Calico](#installation_calico)
    - [Installation Local Storage Provisionner](#installation_local_storage_provisionner)
    - [Installation Helm](#installation_helm)
    - [Installation Ingress Nginx](#installation_nginx)
    - [Installation Monitoring](#installation_monitoring)

## **Serveur Master (vm_k8s)**

- VM : 
    - Nom : vm_k8s
    - OS : ubuntu 20.04
    - Adresse IP : 192.168.0.76
    - CPU : 2 
    - RAM : 4096Mo 
    - Disque : 100Go 


## **Installation de Kubernetes**

- Update du serveur 

```sh
   sudo apt update
```

- /!\ Désactivation de la swap (pas compatible avec K8S)
Commentez la ligne de la swap dans le fichier /etc/fstab

```sh
   sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   sudo swapoff -a
```

- Installation kubelet, kubeadm and kubectl

```sh
   sudo apt update
   sudo apt -y install curl apt-transport-https
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - 
   echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

- Installation des paquets nécessaires

```sh
   sudo apt update
   sudo apt -y install vim git curl wget kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
```

- Activation des modules kernel et configuration du sysctl

```sh
# Enable kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Add some settings to sysctl
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system
```


- Installation de Docker Runtime 

```sh
# Add repo and Install packages
sudo apt update
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io docker-ce docker-ce-cli

# Create required directories
sudo mkdir -p /etc/systemd/system/docker.service.d

# Create daemon json config file
sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# Start and enable Services
sudo systemctl daemon-reload 
sudo systemctl restart docker
sudo systemctl enable docker
```

- Initialisation du node Master

    - Activer le kubelet service
```sh
sudo systemctl enable kubelet
```
- Pull des images des containers 
```sh
sudo kubeadm config images pull
```
- Création du Cluster 

```sh
kubeadm init --pod-network-cidr= 10.0.2.0/16 --apiserver-advertise-address=192.168.0.76
```

- Configuration de kubectl 

```sh
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- Check des infos du Cluster 

```sh
kubectl cluster-info
```

# **Installation Calico**

- Déploiement Calico 

```sh
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
```

- Création d’un fichier avec les paramètres pour préciser le réseau (CIDR) crée pour le cluster : 

```sh
nano fichier_calico_cidr.yaml
```
- Application du fichier créer ci-dessus 

```sh
kubectl apply –f fichier_calico_cidr.yaml
```
- Désactivation des taint : 

```sh
kubectl taint nodes --all node-role.kubernetes.io/master-
```
- Check de l'état des pods 

```sh
kubectl get pods --all-namespaces
```

- Check du node 

```sh
kubectl get nodes -o wide
```

# **Installation Local Storage Provisioner**
 
- Déploiement Local Storage Provisioner 

```sh
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

- Check de la création 

```sh
kubectl -n local-path-storage get pod
```

- Ajout du storage par défaut 

```sh
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

- Vérification 

```sh
kubectl get storageclass
```

# **Installation de Helm**

```sh
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

# **Installation Ingress Nginx**

- Ajout repo Helm 

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

- Création du namespace web

```sh
kubectl create ns web
```

- Création d’un fichier nginx-ingress.yaml pour implémenter la config perso du ingress nginx

```sh
nano /etc/ nginx-ingress.yaml
helm install nginx-ingress -f nginx-ingress.yaml ingress-nginx/ingress-nginx -n web
```

# **Installation Monitoring**

- Ajout du repo helm de pour Prometheus : 

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

- Ajout des CRDs : 

```sh
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.50.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagerconfigs.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.50.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagers.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.50.0/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.50.0/example/prometheus-operator-crd/monitoring.coreos.com_probes.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.50.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.50.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.50.0/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.50.0/example/prometheus-operator-crd/monitoring.coreos.com_thanosrulers.yaml
```

- Création du fichier de configuration personnalisé de Prometheus : 

```sh
nano /etc/prometheus.yaml
```
- Fichier de conf : projet_ynov/monitoring/prometheus.yaml

- Installation de Prometheus avec Helm (dans le namespace Admin) : 

```sh
cd /etc
helm install prometheus -f prometheus.yaml prometheus-community/kube-prometheus-stack -n admin
```

- Check de la présence et de l'état des pods Prometheus & Grafana 
```sh
kubectl get pods -n admin
OU (pour afficher tous les pods)
kubectl get pods -A 
```

- Ajout du nom de domaine et de l’adresse du serveur sur le PC pour faire la redirection vers le Grafana : 

```sh
C:\Windows\System32\drivers\etc\hosts
```
```sh
192.168.0.76 grafana-tp-k8s.fr
```

# **Installation Loki Stack**

- Ajout des repo Helm de Loki-Stack + récupération des valeurs du fichier yaml

```sh
helm repo add loki https://Grafana.github.io/loki/charts
help repo update
helm fetch --untar loki/loki-stack
cd loki-stack
cp values.yaml my-values.yaml
```

- Fichier des valeurs mise dans le fichier yaml : projet_ynov/monitoring/loki_values.yaml

- Application du fichier customisé 

```sh
helm template  -f my-values.yaml --output-dir /loki-stack/my-values.yaml --namespace admin --name loki .
```

- Ajout de la Datasource de Loki sur le Grafana : 
  - Dans Grafana allez dans configuration / datasources et ajouter en un nouveau de type loki





