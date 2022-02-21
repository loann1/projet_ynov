# DOCUMENTATION

## **Serveur Master (vm_k8s)**

- VM : 
    - Nom : vm_k8s
    - OS : ubuntu 20.04
    - Adresse IP : 192.168.0.76
    - CPU : 2 
    - RAM : 4096Mo 
    - Disque : 100Go 


# **Installation de Kubernetes**

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
kubeadm init --pod-network-cidr=192.168.1.0/16 --apiserver-advertise-address=192.168.0.76
```

- Configuration de kubectl 

```sh
mkdir -p $HOME/.kube
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- Check des infos du Cluster 

```sh
kubectl cluster-in
```

