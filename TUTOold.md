Linux Embarqué : Mini Projet.

==============Matériel===============
* RaspberryPI 3
* Alimentation RaspberryPI
* Carte SD 512Mo (Minimum)
* Servo moteur 0-180°
* Adaptateur usb liaison série
* Caméra RaspberryPI
* Câbles de branchement
* Adaptateur usb liaison série


=========Installations requises======
* Docker      *sudo apt install docker*
* MatplotLib  *sudo pip install matplotlib*
* Pygame      *sudo pip install pygame*
* PIL         *sudo pip install Pillow*
* GTKterm     *sudo apt-get install gtkterm*

La suite des installations sont a faire dans le docker

* librairies libv4l & libjpeg *sudo apt install libjpeg-dev libv4l-dev autoconf automake libtool*
=====================================

# DEROULEMENT
* Flashage de la Raspberry
* Cross compilation
* Copie des dossier sur la Raspberry et votre Ordinateur
* Modification Adresse IP (RAPSBERRYPI et ORDINATEUR)
* Branchement Servo moteur & caméra
* Commande à lancer
* Règle du jeu

# Flashage de la Raspberry

On récupère le docker :

**$ docker pull pblottiere/embsys-rpi3-buildroot-video**

**$ docker run -it pblottiere/embsys-rpi3-buildroot-video /bin/bash**

**$ docker# cd /root**

**$ docker# tar zxvf buildroot-precompiled-2017.08.tar.gz**

On copie l'image, qui sera flasher sur la carte, sur notre machine hôte depuis le docker.
Ouvrez un autre terminal ou vous serez en dehors du docker et executez la commande suivante:

**$ docker cp <container_id>:/root/buildroot-precompiled-2017.08/output/images/sdcard.img .**

Vous trouvez le contener id en executant la commande **sudo docker ps -a**

Puis on Flash l'image sur la carte SD grâce à la commande _dd_

**$ sudo dd if=sdcard.img of=/dev/sdX bs=4096 status=progress**

_sdX_ étant le port sur lequel la carte SD est branché. On peut le récupérer a l'aide de _dmesg_.

#Cross Compilation (Dans le docker)

Commande à réaliser pour cross compiler votre fichier si vous voulez modifier le fichier C ou en créer un nouveau.

Dans le docker commencer par faire:
* _git clone https://github.com/dussotro/exam_2019.git_
* _cd exam_2019/server/camera/v4l2grab-master/
* _./autogen.sh_, puis
* _ac_cv_func_malloc_0_nonnull=yes ac_cv_func_realloc_0_nonnull=yes ./configure --host=arm-buildroot-linux-uclibcgnueabihf CC=/root/buildroot-precompiled-2017.08/output/host/usr/bin/arm-linux-gcc_
* _make_,pour cross compilé

le Fichier cross compiler pour votre RaspberryPi est **v4l2grab**


#Copier Fichier dans la RaspberryPi
Mettez vous dans le terminal ou vous n'êtes pas dans le Docker.
Copier les fichiers sur votre ordinateur, depuis le docker, dans un dossier:
**docker cp <container_id>:/root/exam_2019/servers/servo_server.py .**

**docker cp <container_id>:/root/exam_2019/servers/camera/v4l2grab-master/v4l2grab .**

**docker cp <container_id>:/root/exam_2019/servers/Makefile .**

Prendre la carte sd et la mettre sur l'ordinateur et déplacer les fichier à la main.

Mettre les fichiers dans le répertoire _/home/user_,créez un dossier server qui aura les fichiers:
* servo_server.py
* v4l2grab (fichier cross compilé)
* Makefile (vous pouvez prendre celui du Github)

Il faut aussi que vous copier les fichier sur la 1ère partition de la carte SD:
* _start_x.elf_
* _fixup_x.dat_
Utilisé la commande _cp_ ou faite le à la main.

Vous pouvez retrouver ces fichiers dans le docker ici: */buildroot-precompiled-2017.08/output/build/rpi-firmware-685b3ceb0a6d6d6da7b028ee409850e83fb7ede7/boot*

Modifier le fichier *config.txt* de la 1ère partition en ajoutant ces lignes:
**start_x=1
gpu_mem=128**

# Modification de l'adresse Ip de la RaspberryPi pour rendre l'IP statique

Afin de modifier l'adresse ip de la Raspberry
Connecter vous en liaison série avec votre RaspberryPi

Pour cela, brancher la Raspberry en liaison série avec votre ordinateur. Aller sur votre ordinateur, ouvrez _gtkterm_ et configurer le sur le bon port série *ttyUSB0* par exemple (check on _dmesg | grep tty_).
Mettre de Baud_rate à 115200.

Par la suite il faut effectuer les commandes suivantes sur gtkterm :

**$ sudo nano /etc/network/interfaces**

Il faut ensuite remplacer la ligne

*iface eth0 inet dhcp*

par

*auto eth0
allow-hotplug eth0
iface eth0 inet static
  address 172.20.21.164
  netmask 255.255.0.0
  geteway 172.20.255.255*

Si vous voulez changer aussi l'adresse wifi de votre carte et la mettre en static rajouter les ligne suivantes à la suite des autres. Mettez une adresse Ip libre de votre réseau wifi :

*iface wlan0 inet static*

*address XXX.XXX.XXX.XXX*
*netmask 255.255.0.0*


Adresse ip fixe de la RaspberryPi : _172.20.21.164_
Redémarré votre RaaspberryPi.


## Il faut ensuite faire correspondre Adresse IP fixe de l'ordinateur :
Pour l'ordinateur il faut effectuer la commande,

*ifconfig XXXXXXX 172.20.11.72*

 avec XXXXXXX, le nom de l'ethernet de votre pc

# Servo Moteur

On a choisit de brancher le servo moteur sur le port **GPIO4**.

Sur le servo moteur, on envoie une commande en angle entre 0 et 180 degrés.

# Caméra
A NE PAS FAIRE CAR DEJA PRESENT DANS LE MAKEFILE.

Pour crée la sortie vidéo de votre caméra, il faut lancer la commande *modprobe bcm2835-v4l2* sur le terminal gtkterm.
Cette commnde va créee votre sortie vidéo qui sera présente dans le répertoire _/dev/video0_.

# Lancement du code

Sur la RaspberryPI, aller dans _/home/user/server_, là où se trouve le Makefile et exécutez la commande *make*, cette commande vas exécuter les commandes nécessaires.
Et ensuite *make run* pour lancer les servers.
A cette instant les serveurs sont lancés.

Sur votre ordinateur, aller dans le dossier client et lancer la commande, *make*. A cette instant vous entrez dans la peau du client qui peut communiquer avec le server de la RaspberryPi.

# Règles du jeu ! Commandes chez le client

* Pour changer l'angle de la caméra il vous faudra appuyer sur les touches flèches *droite* et *gauche*. L'angle s'affiche sur l'écran pour savoir ou vous en êtes.

* Pour prendre une photo il faut appuyer sur la touche *s* de votre clavier pour sauvegarder l'image sur votre ordinateur et l'afficher. L'image est écrasée d'un appui à l'autre sur la touche *s*.

L'équipe vous remercie de la confiance accordée à leur travail.