# Installation de l'OS

* version __ubuntu-20.04-server__

Ejecter et rebrancher le lecteur/graveur USB

## Première connection en SSH

Utiliser Putty (déteminer IP sur port 22)  
login : __ubuntu__  
password : __ubuntu__  
Change le mot de passe et se reconnecter  

## Désactiver les mises à jour automatique

```bash
nano /etc/apt/apt.conf.d/20auto-upgrades
```

Modifier

```bash
APT::Periodic::Unattended-Upgrade "0"
```

## Update

```bash
sudo apt update
sudo apt full-upgrade
```

## Installations

```bash
apt-get install zip nfs-kernel-server
chmod 600 /apps/traefik/acme.json
```

> Sous la version 20.04 il peut être nécessaire de rebooter entre update et upgrade

## Créer utilisateurs, groupes et permissions

```bash
sudo adduser tchube # si pas créé automatiquement à la première connexion
# si master
sudo groupadd cocooning
sudo adduser tchube cocooning
# master et worker
sudo groupadd docker
sudo adduser tchube sudo
sudo adduser tchube docker
```

```bash
sudo reboot
```

## clef SSH

Dans le répertoire /home/tchube

```bash
sudo mkdir .ssh
cd .ssh
sudo nano authorized_keys
```

Copier la clef publique "cocooning"

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCozFtSIuiagl4UCgTOWe7Mjz7pz4MkINJehuPsHxr0aIhRAherHnmKIb30l8951eYdXfE9OZ0TVvu80Y+7/WngXHS2laylr+W9U8tdJ0c5c3BRFX15/aP/vKyaEo1c/J89Jslpx2jILyDL4ZI3o5TQ/yl9LVPd7SMmo4Ztq8LqiDmQq2IOY4VkUmDhZl20BXWDyt7xVmcbKyAx7wkpos+HHbCkgMc7sKNyqxNF6CZHBecM/0Xk6NfyGZQ4zAQW0Ii4/L4m481NPpoLAyPEmUOWvEDGDSNJt2MDyf0axEoV1CxFfYb25NUwNXGElTLlJX6wBP1u2L/3WSr1xu+tsp0viEtZO6JgZujux9KCkPiJRc37zuHnwtbAFYJI0pIPyRGv26V7I4IrM9JF+zzm3LjdCs2G1SmdcxSnaOWX1VxJOMbQKMpF0XgIm4aM9BLOQmXwkFVGyzrhNVx/xauwZySM6UV0tBQT10vTUDpwput0AAzMpJSSNUVX48b9eQr9mW8= tchube@ubuntu-server
```

## Modifier access SSH (port 22...) si nécessaire

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
Subsystem sftp /usr/lib/openssh/sftp-server
#Subsystem  sftp  internal-sftp
#Match User tchube
#         ChrootDirectory /home/%u
#         ForceCommand internal-sftp
#         AllowTCPForwarding no
#         X11Forwarding no
```

```bash
sudo service sshd restart
```

## SFTP sous WinSCP

Adresse du server SFTP à modifier dans winscp

```bash
/usr/lib/openssh/sftp-server
```

## créer le répertoire .cocooning (si master)

```bash
cd /
sudo mkdir .cocooning # le propriétaire et le groupe sont root
cd .cocooning
sudo mkdir common
```

Initialiser git de l'utilisateur

```bash
sudo git config --global user.name "cocooning"
sudo git config --global user.email jp.tchube@cocooning.tech
```

Cloner le repository de l'utilisateur (https)

```bash
cd /.cocooning
sudo git clone 
```

Changer le propriétaire, le groupe du répertoire et les permissions pour le groupe et users

```bash
sudo chown -R tchube .cocooning
sudo chgrp -R cocooning .cocooning
# plus besoin de sudo si on est connecté "tchube"
chmod g+rwx -R .cocooning
chmod u+rwx -R .cocooning
```

## Changer le hostname

```bash
sudo nano /etc/hostname
```

Changer le hostname en fonction du type de noeud

```bash
master1
```

```bash
worker1
```

>Les hostname de chaque noeud du cluster doivent être différents

```bash
sudo reboot
```

## Activer le Wifi

Configurer Netplan  

```bash
sudo bash -c "echo 'network: {config: disabled}' > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg"
sudo nano /etc/netplan/10-my-config.yaml
```

Ajouter le code  
>Pas de tabulation dans ce fichier mais des "espaces"

```bash
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      dhcp6: no
      accept-ra: no
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
  wifis:
    wlan0:
      dhcp4: no
      dhcp6: no
      accept-ra: no
      addresses: [192.168.1.99/24]
      gateway4: 192.168.1.1
      access-points:
        "Livebox-XXXX":
          password: "XXXXX"
```

> Changer l'adresse IP selon la box et le node
Appliquer la configuration  

```bash
sudo netplan generate
sudo netplan apply
```

## Installer Docker.io et Docker compose

Sur master et worker

```bash
sudo apt-get install docker.io
```

Sur master uniquement

```bash
sudo apt-get install docker-compose
```

## Installation d'un server NFS

```bash
sudo su
apt-get install nfs-kernel-server
```

Créez une table d'export NFS

```bash
nano /etc/exports
```

Copier coller les chemins ci-dessous

```bash
/apps/hassio 192.168.1.100(rw,no_root_squash,aync,no_subtree_check)
```  

> Créer autant de ligne que de répertoire à partager  
Mettre à jour la table nfs

```bash
exportfs -ra
```

Ouvrez les ports utilisés par NFS.
Pour savoir quels ports NFS utilise, entrez la commande suivante:

```bash
rpcinfo -p | grep nfs
```

Ouvrez les ports générés par la commande précédente.

```bash
sudo ufw allow 2049
```

Relancer le service

```bash
sudo service nfs-kernel-server reload
```

## Initialisation de Docker

### Installation d'un master node

```bash
sudo su
docker swarm init --advertise-addr 192.168.1.100
```

### Installation d'un worker node

```bash
sudo su
docker swarm join --token SWMTKN-1-12i661gdni5wsvj4be9deu50uzu8fw76q2jvyi2ykhywwdde1f-2c0g31twep1n1kum3yb6aub0r 192.168.1.50:2377
```

Créer le network de type overlay (traefik,hassio...) : cocooning-network

## Installation de k3s  

Modifier le fichier

* __cmdline.txt__ sur __ubuntu-20.04-server__
* __nobtcmd.txt__ sur __ubuntu-18.04-server__
Ajouter en fin de ligne

```bash
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
```

### Mode High Availability with Embedded DB (Experimental) avec etcd

## Command line

Temprérature cpu

```bash
cat /sys/class/thermal/thermal_zone0/temp
```
