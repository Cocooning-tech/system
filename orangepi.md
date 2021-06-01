# Installation de l'OS

login : root  
password : 1234  
Ne pas paramétrer le timezone

## Update

```bash
sudo apt update
sudo apt full-upgrade
```

## Configuration

```bash
sudo armbian-config
```

Activer le WIFI  
Enable serial port  
Avahi

## Créer utilisateurs, groupes et permissions

```bash
sudo useradd tchube # si pas créé automatiquement à la première connexion
sudo groupadd cocooning
sudo groupadd docker
sudo adduser tchube cocooning
sudo adduser tchube sudo
sudo adduser tchube docker
```

## créer le répertoire .cocooning

```bash
cd /
sudo mkdir .cocooning # le propriétaire et le groupe sont root
# changer le propriétaire, le groupe du répertoire et les permissions pour le groupe et user
sudo chown -R tchube .cocooning
sudo chgrp -R cocooning .cocooning
# plus besoin de sudo si on est connecté "tchube"
chmod g+rwx -R .cocooning
chmod u+rwx -R .cocooning
```

## clef SSH

Dans le répertoire /home/tchube

```bash
sudo su
nano authorized_keys
```

Copier la clef publique "cocooning"

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCozFtSIuiagl4UCgTOWe7Mjz7pz4MkINJehuPsHxr0aIhRAherHnmKIb30l8951eYdXfE9OZ0TVvu80Y+7/WngXHS2laylr+W9U8tdJ0c5c3BRFX15/aP/vKyaEo1c/J89Jslpx2jILyDL4ZI3o5TQ/yl9LVPd7SMmo4Ztq8LqiDmQq2IOY4VkUmDhZl20BXWDyt7xVmcbKyAx7wkpos+HHbCkgMc7sKNyqxNF6CZHBecM/0Xk6NfyGZQ4zAQW0Ii4/L4m481NPpoLAyPEmUOWvEDGDSNJt2MDyf0axEoV1CxFfYb25NUwNXGElTLlJX6wBP1u2L/3WSr1xu+tsp0viEtZO6JgZujux9KCkPiJRc37zuHnwtbAFYJI0pIPyRGv26V7I4IrM9JF+zzm3LjdCs2G1SmdcxSnaOWX1VxJOMbQKMpF0XgIm4aM9BLOQmXwkFVGyzrhNVx/xauwZySM6UV0tBQT10vTUDpwput0AAzMpJSSNUVX48b9eQr9mW8= tchube@ubuntu-server
```

## Modifier access SSH (port 22...)

Configuration

```bash
sudo nano /etc/ssh/sshd_config
```

```bash
Include /etc/ssh/sshd_config.d/*.conf
PermitRootLogin yes
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
# Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem  sftp  internal-sftp
Match User tchube
         ChrootDirectory /home/%u
         ForceCommand internal-sftp
         AllowTCPForwarding no
         X11Forwarding no
```

```bash
sudo service sshd restart
```

## SFTP sous WinSCP

Adresse du server SFTP à modifier dans winscp

```bash
/usr/lib/openssh/sftp-server
```

## installations

```bash
sudo su
apt-get update
apt-get install 
```

## Static IP configuration et DNS

```bash
sudo nmtui
```

## Installer Docker et Docker compose

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

```bash
sudo usermod -aG docker tchube
```

```bash
sudo apt-get install docker-compose
```

### Installation d'un master node

```bash
sudo su
docker swarm init --advertise-addr 192.168.1.100
```

### Installation d'un worker node

```bash
sudo su
docker swarm join --token SWMTKN-1-2q2nqvmoevas7lym46zvce7niu6zoemhiruqr2xdav494f32r8-5gkcg3t5638ewg5mmjejdni63 192.168.1.50:2377
```

Créer le network de type overlay (traefik,hassio...) : cocooning-network

## Git

### Commande line

Git contient un outil appelé git config pour vous permettre de voir et modifier les variables de configuration qui contrôlent tous les aspects de l’apparence et du comportement de Git. Ces variables peuvent être stockées dans trois endroits différents :

[chemin]/etc/gitconfig : Contient les valeurs pour tous les utilisateurs et tous les dépôts du système. Si vous passez l’option --system à git config, il lit et écrit ce fichier spécifiquement. Parce que c’est un fichier de configuration au niveau système, vous aurez besoin de privilèges admnistrateur ou super-utilisateur pour le modifier.

Fichier ~/.gitconfig : Spécifique à votre utilisateur. Vous pouvez forcer Git à lire et écrire ce fichier en passant l’option --global et cela affecte tous les dépôts avec lesquels vous travaillez sur ce système.

Fichier config dans le répertoire Git (c’est-à-dire .git/config) du dépôt en cours d’utilisation : spécifique au seul dépôt en cours. Vous pouvez forcer Git à lire et écrire dans ce fichier avec l’option --local`, mais c’est en fait l’option par défaut. Sans surprise, le répertoire courant doit être dans un dépôt Git pour que cette option fonctionne correctement.

La première chose à faire après l’installation de Git est de renseigner votre nom et votre adresse de courriel. C’est une information importante car toutes les validations dans Git utilisent cette information et elle est indélébile dans toutes les validations que vous pourrez réaliser :

```bash
sudo git config --global user.name "cocooning"
sudo git config --global user.email jp.tchube@cocooning.tech
```

Encore une fois, cette étape n’est nécessaire qu’une fois si vous passez l’option --global, parce que Git utilisera toujours cette information pour tout ce que votre utilisateur fera sur ce système. Si vous souhaitez surcharger ces valeurs avec un nom ou une adresse de courriel différents pour un projet spécifique, vous pouvez lancer ces commandes sans option --global lorsque vous êtes dans ce projet.

### Avec nano

```bash
sudo nano /root/.gitconfig
```

Ajouter l'identification (attention espace ou tab ?)

```bash
[user]
    email = jp.tchube@cocooning.tech
    name = cocooning
```

## Commandes

### Etteindre

```bash
sudo halt
```

### Température cpu

```bash
htop
```

ou

```bash
cat /sys/class/thermal/thermal_zone0/temp
```
