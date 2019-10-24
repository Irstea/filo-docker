DOCKER-FILO-SCIENCE
==================
# Présentation

Le logiciel [Filo-Science](https://github.com/Irstea/filo-science) permet de saisir les informations issues des pêches électriques de poissons, ainsi que celles de pistage des animaux munis de balises. S'il est conçu pour fonctionner en mode web, son utilisation en mode déconnecté peut parfois être nécessaire, notamment au bord des cours d'eau.

Pour cela, le logiciel doit pouvoir être embarqué à bord d'un ordinateur de terrain (portable ou tablette Windows ou Linux, ou Raspberry Pi). La technologie choisie est celle basée sur les containers Docker, pour pouvoir installer une base de données Postgresql et un serveur Web Apache2 pour héberger le code PHP.

Les scripts fournis permettent d'installer deux containers Docker, l'un pour héberger la base de données, l'autre pour le serveur Web.

# Installation de Docker
## Debian, Ubuntu ou Raspbian

```
    sudo -s
    apt-get update
    apt-get install curl
    curl -fsSL https://get.docker.com/ | sh
    apt-get install docker-compose
    systemctl enable docker
    service docker start
    groupadd docker
    usermod -aG docker $USER
```
## Windows
Suivez les instructions décrites ici : [https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/).

Installez également le programme *PowerShell* de Windows, qui vous permettra d'ouvrir un terminal et de lancer les commandes manuelles.

# Installation des containers
Les commandes sont données pour Linux. Pensez à adapter la démarche à Windows (téléchargement manuel depuis un navigateur, décompression avec Windows, etc.).

Téléchargez le code de ce dépôt dans un dossier de votre ordinateur :
```
sudo apt-get install wget unzip
wget https://github.com/Irstea/filo-docker/archive/master.zip
unzip master.zip
cd filo-docker-master
```
Créez un volume Docker pour héberger la base de données Postgresql :
```
docker volume create --name filopgdata -d local
```
Créez les deux images et les containers associés :
```
docker-compose up --build -d filo-web
```
Si tout se passe bien, vous retrouverez les images suivantes :
```
docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
filo-docker_filo-web       latest              834d8fd9f504        18 hours ago        782MB
filo-docker_filo-db        latest              78d95ff5fea4        19 hours ago        888MB
```

Et les containers :
```
docker container ls
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                                      NAMES
125eafce92ac        filo-docker_filo-web   "/bin/sh -c /start.sh"   56 seconds ago      Up 54 seconds       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   filo-docker_filo-web_1
4f6b43a1261a        filo-docker_filo-db    "su - postgres -c 'P…"   17 hours ago        Up 10 minutes       0.0.0.0:5433->5432/tcp                     filo-docker_filo-db_1
```

**Attention :** le serveur web expose les ports 80 et 443. Si vous avez déjà un serveur web qui fonctionne dans votre ordinateur, vous devrez arrêter votre serveur web local avant de lancer le démarrage des containers.

Le serveur postgresql sera accessible depuis le port 5433, en localhost :
```
psql -U filo -h localhost -p 5433
Mot de passe pour l'utilisateur filo : filoPassword
psql (11.5 (Ubuntu 11.5-3.pgdg18.04+1))
Connexion SSL (protocole : TLSv1.3, chiffrement : TLS_AES_256_GCM_SHA384, bits : 256, compression : désactivé)
Saisissez « help » pour l'aide.

filo=#
```
## lancer l'application web
### Docker est installé dans l'ordinateur qui sert à accéder à l'application
C'est le cas d'un ordinateur portable Windows ou Linux. Dans un premier temps, récupérez l'adresse IP du serveur Web :
```
docker exec filo-docker_filo-web_1 ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.0.3  netmask 255.255.0.0  broadcast 172.19.255.255
```
Ici, le container s'est vu attribué l'adresse IP *172.19.0.3*.

Ajoutez une ligne dans votre fichiers /etc/hosts (Linux) ou    (Windows) :
```
172.19.0.3 filo-docker filo-docker.local
```
Dans votre navigateur, allez sur le site : [https://filo-docker.local](https://filo-docker.local). Acceptez l'exception de sécurité : vous devriez accéder à l'application.

Vous pouvez vous connecter avec le login *admin*, mot de passe *password* : il s'agit d'une installation par défaut.

## Quelques commandes utiles de docker

* docker images : affiche la liste des images disponibles
* docker container ls : affiche la liste des containers
* docker-compose up -d filo-web : démarre l'image filo-web et filo-db dans leurs containers respectifs, en les recréant
* docker-compose stop filo-web : arrête le container filo-web
* docker-compose start filo-web : démarre le container filo-web
* docker exec -ti filo-docker_filo-web_1 /bin/bash : se connecte au container et permet d'exécuter des commandes
* docker rmi filo-docker_filo-web --force : supprime brutalement l'image filo-web
* docker-compose up --build filo-web : recrée les deux images. Attention : la base de données va être recréée !
* docker update --restart=no filo-docker_filo-web_1 : désactive le démarrage automatique du container
* docker inspect filo-docker_filo-web_1 : affiche le paramétrage courant du container
* docker system prune -a : supprime toutes les images, pour réinitialiser docker

## Sauvegarde de la base de données
L'image *filo-db* intègre une sauvegarde automatique de la base de données, qui se déclenche tous les jours à 13:00. Vous la retrouverez dans votre ordinateur, dans le dossier *Dossier personnel/filopgbackup*. Pensez à la déplacer vers un autre emplacement sur le réseau, pour éviter de tout perdre en cas de crash ou de vol de l'ordinateur.


# Utilisation d'un Raspberry Pi
## Installation de Raspbian

Pour l'installation de Raspbian, consultez la [documentation d'installation de Raspberry](https://www.raspberrypi.org/documentation/).

Pensez à activer l'accès via ssh, et désactivez l'interface graphique au démarrage, qui consomme des ressources et est sans intérêt dans le contexte de Filo-Science.

## Connexion en ssh
Pour vous connecter à votre Raspberry, utilisez la commande :
```
ssh pi@adresse_ip
```
Une fois connecté, tapez la commande :
```
sudo -s
```
pour obtenir les droits *root*.

## Créer un réseau wifi pour connecter directement les terminaux

Suivez les instructions définies dans le premier chapitre de ce document : [https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md](https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md) (*Setting up a Raspberry Pi as an access point in a standalone network (NAT)*). 
Adpatez le contenu du fichier */etc/hostapd/hostapd.conf*, et notamment :
* ssid=filo-docker
* wpa_passphrase=votre_mot_de_passe

Editez ensuite le fichier */etc/dnsmasq.conf*, et rajoutez ces lignes :
```
server=8.8.8.8
address=/filo-docker.local/192.168.4.1
```
La ligne *server* correspond au serveur d'adresses web (DNS) de Google. Si vous souhaitez utiliser un autre DNS, par exemple celui de votre organisme, modifiez cette ligne.

Redémarrez le Raspberry, et connectez-vous au réseau wifi *filo-docker*. Testez la communication avec l'application, en entrant l'adresse suivante dans un navigateur : https://192.168.4.1. Vous devez accéder à la page d'accueil.

Cette configuration vous permet de charger les dalles Openstreetmap avant de partir sur le terrain :
* au bureau, connectez le Raspberry au réseau local avec un câble Ethernet
* connectez votre tablette au Raspberry en wifi
* lancez l'application, à l'adresse https://192.168.4.1.
* ouvrez le module *Paramètres>Mise en cache de la cartographie*, et téléchargez les dalles dont vous aurez besoin sur le terrain
* arrêtez le Raspberry, déconnectez le câble Ethernet, fermez le navigateur de votre tablette
* redémarrez le Raspberry, reconnectez la tablette au wifi, et rouvrez l'application : les dalles sont accessibles sans que vous soyez connectés à Internet.


# Remerciements

Les scripts sont issus de ceux élaborés par Julien Ancelin et Christine Plumejeaud-Perreau pour la diffusion via Docker de l'application [Collec-Science](https://github.com/Irstea/collec), et diffusés dans [Github](https://github.com/jancelin/docker-collec). Qu'ils en soient remerciés pour le travail réalisé.

# Licence

Les scripts sont diffusés sous licence MIT.