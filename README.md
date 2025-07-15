# CKS_Jenkins
KUBERNETES DEPLOYMENT USING JENKINS

## Pre-requisite :

- Docker File
- Kubernete cluster
- Kubernetes deployment and service Yaml file


# Création d'un cluster Kubernetes avec Kubeadm

> Compatible avec Ubuntu 20.04, 22.04 et Kali Linux

---

## Prérequis

* Minimum 2 machines Linux (1 master + 1 ou plusieurs workers)
* 2 Go de RAM minimum, 2 CPU
* Accès SSH avec droits root ou sudo
* Accès Internet
* Noms d'hôte uniques, adresses IP fixes ou DHCP réservé
* Swap désactivé

---

## Étape 1 : Préparer tous les nœuds

À exécuter sur **tous les nœuds** (master + workers) :

```bash
# Définir le nom d'hôte (adapter selon le rôle)
sudo hostnamectl set-hostname master-node     # ou worker-node-1

# Désactiver le swap
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Activer les modules nécessaires
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

sudo modprobe br_netfilter

# Paramètres sysctl nécessaires à Kubernetes
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

---

## Étape 2 : Installer Docker

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

Vous pouvez remplacer Docker par `containerd` si besoin.

---

## Étape 3 : Installer kubeadm, kubelet et kubectl

```bash

## Étape 1 : Télécharger la clé publique de signature Kubernetes

Si le dossier `/etc/apt/keyrings` n'existe pas, créez-le avec les bonnes permissions :

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
```

Étape 1 :Télécharger ensuite la clé :

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

---

Étape 2 : Ajouter le dépôt officiel Kubernetes (v1.30)

⚠Cette commande **remplacera** toute configuration existante dans `/etc/apt/sources.list.d/kubernetes.list` :

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
```

---

Étape 3 : Mettre à jour et installer kubeadm, kubelet, kubectl

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

Activer kubelet au démarrage

```bash
sudo systemctl enable --now kubelet
```

---

Vérification

```bash
kubeadm version
kubectl version --client
```

---

Tu peux maintenant initialiser ton cluster avec :

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```



## Étape 4 : Initialiser le nœud maître

À exécuter uniquement sur le **nœud master** :

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

Notez bien la commande `kubeadm join` affichée à la fin pour les workers.

---

## Étape 5 : Configurer kubectl sur le master

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## Étape 6 : Installer le plugin réseau (CNI)

Exemple avec **Calico** :

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

Vous pouvez aussi utiliser Flannel, Weave, etc.

---

## Étape 7 : Joindre les nœuds workers

Sur chaque **nœud worker**, exécutez la commande `kubeadm join` obtenue précédemment :

```bash
sudo kubeadm join 192.168.1.10:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

---

## Étape 8 : Vérifier le cluster

Sur le **master** :

```bash
kubectl get nodes
```

## stuces supplémentaires

* Voir tous les pods : `kubectl get pods -A`
* Logs kubelet : `journalctl -u kubelet`
* Réinitialiser :

```bash
kubeadm reset && sudo rm -rf ~/.kube /etc/cni /var/lib/kubelet
```


## Installation Jenkins


### 2. Lancer Jenkins dans un conteneur Docker avec les droits nécessaires pour lui permettre d’exécuter d'autres conteneurs Docker depuis l'interface Jenkins.
```bash
docker run -u 0 --privileged --name jenkins -it -d \
  -p 8080:8080 -p 50000:50000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(which docker):/usr/bin/docker \
  -v /home/jenkins_home:/var/jenkins_home \
  jenkins/jenkins:latest
# 1. Run docker ps
docker ps

# 2. Run docker logs pouir trouver l'itial mot de passe
docker logs -f <conteneur_id>
```
### 3. On a le mot de pass.
```bash
993c3b3395ee4b7fa127fb5cd805c185
```

## Page Web Jenkins
### On peut accéder à la page web grâce à notre IP local
```bash
<IP_local>:8080
```
### 1. Cliquer sur **"Install plugins"**
### 2. Puis Créer le 1er utilisateur Administrateur
### 3. Pour installer Plugin " Docker Pipeline"
```bash
Guide : administrer Jenkins>Plugins>Plugins disponibles
Dans la barre de recherche, tapez : "Docker Pipeline", puis installez-le
```
### 4.  Installer le plugin Kubernetes CD manuellement
#### Télécharger le plugin via le lien ci-dessous, puis l’installer depuis l’interface Jenkins (Gérer Jenkins > Plugins > Ajouter depuis un fichier .hpi) :
```bash
https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbU1CZTRWdVpJdDVxM2NFSWVtOWdnNzQwNnltQXxBQ3Jtc0ttelBiT1NIcHpsNW80U255cHlqdHU3cUYteUlKTk5HUFVWV2dJRURIRmdxbWE3eGxiRXJSYy1WbTN3QlBJbWd5ZnY4RDI5VXJtQnhpbDhtaHZsbndBOWFXSXh6ZTg3YXFoeUY5bWdWRWdYd1dfRFUxcw&q=https%3A%2F%2Fupdates.jenkins.io%2Fdownload%2Fplugins%2Fkubernetes-cd%2F1.0.0%2Fkubernetes-cd.hpi&v=XE_mAhxZpwU
```
### 5. Ajouter des identifiants (Credentials)

#### Allez dans :
**Administrer Jenkins** > **Identifiants** > **Système** > **Identifiants globaux (illimités)**

#### Puis définir :
- **Nom d'utilisateur**
- **Mot de passe**
- **ID**

Utiliser cet ID :
```bash
dockerhublogin

```
### 6. Configuration Kubernetes CD

#### allez dans : 
**Administrer Jenkins** > **Identifiants** > **Système** > **Identifiants globaux (illimités)**
#### Dans le champ "Type" sélectionner :

Kubernetes configuration

#### Pour le "ID" et "Deqscritpion" mettez :
```bash
kubernetes
```
#### Dans le champ "kubeconfig", sélectionner :
Enter directly










# 4. Lancer l'application
python app.py
