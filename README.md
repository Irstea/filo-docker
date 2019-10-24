DOCKER-FILO-SCIENCE
==================
(translation from README-FR.md, realized with [deepl.com](https://deepl.com/translator))
# Presentation

The software[Filo-Science](https://github.com/Irstea/filo-science) allows you to enter information from electric fish fisheries, as well as information on the tracking of animals equipped with tags. If it is designed to operate in web mode, its use in offline mode may sometimes be necessary, especially at the edge of watercourses.

To do this, the software must be able to be embedded on a field computer (Windows or Linux laptop or tablet, or Raspberry Pi). The technology chosen is the one based on Docker containers, to be able to install a Postgresql database and an Apache2 Web server to host the PHP code.

The scripts provided allow you to install two Docker containers, one to host the database and the other for the web server.

# Docker installation
## Debian, Ubuntu or Raspbian

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
Follow the instructions described here: [https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/).

Also install the Windows *PowerShell* program, which will allow you to open a terminal and launch manual commands.

# Installation of containers
The commands are given for Linux. Remember to adapt the approach to Windows (manual download from a browser, decompression with Windows, etc.).

Download the code of this deposit in a folder on your computer:
```
sudo apt-get install wget unzip
wget https://github.com/Irstea/filo-docker/archive/master.zip
unzip master.zip
cd filo-docker-master
```
Create a Docker volume to host the Postgresql database:
```
docker volume create --name filopgdata -d local
```
Create the two images and the associated containers:
```
docker-compose up --build -d filo-web
```
If all goes well, you will find the following images:
```
docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
filo-docker_filo-web latest 834d8fd9f504 18 hours ago 782MB
filo-docker_filo-db latest 78d95ff5fea4 19 hours ago 888MB
```

And the containers:
```
docker container ls
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
125eafce92ac filo-docker_filo-web "/bin/sh -c /start.sh" 56 seconds ago Up 54 seconds 0.0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp filo-docker_filo-web_1
4f6b43a1261a filo-docker_filo-db "su - postgres -c'P..."   17 hours ago Up 10 minutes 0.0.0.0.0.0:5433->5433->5432/tcp filo-docker_filo-db_1
```

**Warning:** the web server exposes ports 80 and 443. If you already have a web server running on your computer, you will need to shut down your local web server before starting the containers.

The postgresql server will be accessible from port 5433, in localhost:
```
psql -U filo -h localhost -p 5433
Password for the filo user: filoPassword
psql (11.5 (Ubuntu 11.5-3.pgdg18.04+1))
SSL connection (protocol: TLSv1.3, encryption: TLS_AES_256_GCM_SHA384, bits: 256, compression: disabled)
Enter "help" for help.

filo=#
```
## launch the web application
#### Docker is installed in the computer that is used to access the application
This is the case with a Windows or Linux laptop. First, retrieve the IP address of the web server:
```
docker exec filo-docker_filo-web_1 ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
        inet 172.19.0.3 netmask 255.255.0.0.0 broadcast 172.19.255.255.255
```
Here, the container has been assigned the IP address *172.19.0.3*.

Add a line to your /etc/hosts (Linux) or (Windows) files:
```
172.19.0.3 filo-docker filo-docker.local
```
In your browser, go to the site: [https://filo-docker.local](https://filo-docker.local). Accept the security exception: you should access the application.

You can connect with the login *admin*, password *password*: this is a default installation.



## Some useful docker commands

* docker images: displays the list of available images
* docker container ls: displays the list of containers
* docker-composes up -d filo-web: starts the filo-web and filo-db image in their respective containers, recreating them
* docker-composes stop filo-web: stops the filo-web container
* docker-composes start filo-web: starts the filo-web container
* docker exec -ti filo-docker_filo-web_1 /bin/bash: connects to the container and allows to execute commands
* docker rmi filo-docker_filo-web --force: suddenly deletes the filo-web image
* docker-composes up --build filo-web: recreates both images. Warning: the database will be recreated!
* docker update --restart=no filo-docker_filo-web_1 : disables the automatic start of the container
* docker inspect filo-docker_filo-web_1: displays the current container settings
* docker system prune -a : delete all images, to reinitialize docker

## Database backup
The *filo-db* image includes an automatic database backup, which is triggered every day at 13:00. You will find it in your computer, in the folder *Personal folder/filopgbackup*. Remember to move it to another location on the network, to avoid losing everything in the event of a computer crash or theft.


# Use of a Raspberry Pi
## Raspbian installation

For the installation of Raspbian, refer to the[Raspberry installation documentation] (https://www.raspberrypi.org/documentation/).

Remember to enable access via ssh, and disable the graphical user interface at startup, which consumes resources and is irrelevant in the context of Filo-Science.

## Ssh connection
To connect to your Raspberry, use the command :
```
ssh pi@address_ip
```
Once connected, type the command:
```
sudo -s
```
to get the *root* rights.

# Acknowledgements

The scripts are based on those developed by Julien Ancelin and Christine Plumejeaud-Perreau for the distribution via Docker of the application[Collec-Science](https://github.com/Irstea/collec), and distributed in[Github](https://github.com/jancelin/docker-collec). We would like to thank them for their work.

# License

The scripts are released under MIT license.
