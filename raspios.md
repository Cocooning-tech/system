# Installation de l'OS

login : pi  
password : raspberry

## Préparer la SD Card

Créer un fichier ssh.txt dans le répertoire windows pour activer directement le server ssh

## Configuration

```bash
sudo raspi-config
```

Changer le hostname (master1 worker1...)  
Activer le WIFI  
Enable serial port  

## Update

```bash
sudo apt update
sudo apt full-upgrade
```

## Static IP configuration et DNS

```bash
sudo nano /etc/dhcpcd.conf
```

```bash
interface eth0
static ip_address=192.168.1.100/24
#static ip6_address=fd51:42f8:caae:d92e::ff/64
static routers=192.168.1.1
static domain_name_servers=192.168.100

interface wlan0
static ip_address=192.168.1.50/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.50
```

Vérification après reboot

```bash
sudo nano /etc/resolv.conf
```

## Créer user

```bash
sudo adduser tchube
sudo usermod -aG sudo username
```

```bash
sudo reboot
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
docker swarm init --advertise-addr 192.168.1.101
```

### Installation d'un worker node

```bash
sudo su
docker swarm join --token SWMTKN-1-2q2nqvmoevas7lym46zvce7niu6zoemhiruqr2xdav494f32r8-5gkcg3t5638ewg5mmjejdni63 192.168.1.50:2377
```

Créer le network de type overlay (traefik,hassio...) : cocooning-network

## Commandes

Afficher la température du processeur

```bash
/opt/vc/bin/vcgencmd measure_temp
```
