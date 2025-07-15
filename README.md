# CKS_Jenkins
KUBERNETES DEPLOYMENT USING JENKINS

## Pre-requisite :

- Docker File
- Kubernete cluster
- Kubernetes deployment and service Yaml file

## ⚙️ Installation

```bash
# 1. Cloner le dépôt
git clone https://github.com/votre-utilisateur/nom-du-projet.git

# 2. Install Docker 
sudo apt install docker.io
# 4. ajoutl’utilisateur au groupe docker
sudo usermod -aG docker $USER

# 5. Lancer Jenkins dans un conteneur Docker avec les droits nécessaires pour lui permettre d’exécuter d'autres conteneurs Docker depuis l'interface Jenkins.
docker run -u 0 --privileged --name jenkins -it -d \
  -p 8080:8080 -p 50000:50000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(which docker):/usr/bin/docker \
  -v /home/jenkins_home:/var/jenkins_home \
  jenkins/jenkins:latest
# 🔍 Détails :
-u 0 → exécute le conteneur en tant que root

--privileged → donne des droits étendus (attention à la sécurité)

-v $(which docker):/usr/bin/docker → monte l’exécutable Docker dans le conteneur

-v /var/run/docker.sock:/var/run/docker.sock → permet à Jenkins d’exécuter Docker dans Docker

-v /home/jenkins_home:/var/jenkins_home → dossier persistant pour Jenkins

# 4. Lancer l'application
python app.py
