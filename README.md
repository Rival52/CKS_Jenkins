# CKS_Jenkins
KUBERNETES DEPLOYMENT USING JENKINS

## Pre-requisite :

- Docker File
- Kubernete cluster
- Kubernetes deployment and service Yaml file

## ⚙️ Installation
### 1. Installation Docker

```bash
# 1. Cloner le dépôt
git clone https://github.com/votre-utilisateur/nom-du-projet.git

# 2. Install Docker 
sudo apt install docker.io
# 4. ajoutl’utilisateur au groupe docker
sudo usermod -aG docker $USER
```

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
### clic sur " Install plugins"



# 4. Lancer l'application
python app.py
