# Remote-Desktop
Ici des solutions simples pour accéder à distance à un PC. Solution préférée : RustDesk, ~~tigervnc~~

# Powershell

Voir aussi https://github.com/lenainjaune/powershell

# RustDesk
TODO : consigner ici la procédure créée sur mon PC

J'ai fait une procédure très complète sur l'installation du client, du server et du server intermédaire pour mettre en relation le client et le server. Pour le server intermédiaire, j'ai rédigé les procédures d'installation sur un système Debian 11 et sur le NAS OpenMediaVault.

Voir admin_info/remote_desktop/rustdesk.odt qu'il faudra que j'intègre ici.

# XRDP pour Linux

DEBUG : service XRDP stoppé, xrdp-sesman -ns et xrdp -ns

TODO : à retester depuis Debian 11 fresh 32-bits et 64-bits, idem pour Debian 10 ; tester Debian 10/11 NON fresh

J'ai toujours galéré à installer et puis j'ai testé [How to Install Xrdp Server on Debian 11](https://itslinuxfoss.com/xrdp-server-on-debian-11/) sur une installation fresh de Debian 11 32-bits.

Ca a fonctionné nickel (voir commandes effectives dessous) !

Note : à la première ouverture de session j'ai eu un message "il est nécessaire de s'authentifier pour créer un périphérique avec gestion de couleurs" qui demandait l'accès root et j'ai résolu le problème avec [Comment se connecter en RDP à Debian 10 avec xRDP ?](https://www.it-connect.fr/comment-se-connecter-en-rdp-a-debian-10-avec-xrdp/#V_Resoudre_les_erreurs_de_connexion_xRDP) qui indique une protection de PolKit.

```sh
# Depuis fresh installation en VM de Debian Bullseye 11 (kernel 5.10.0-19-686-pae)

# Avant de suivre le tutoriel (tout n'est pas utile, la modif sshd_config est pour autoriser root par ssh) :
apt update
apt install -y ssh avahi-daemon
apt install -y vim htop locate less aptitude wget gawk man sshfs rsync tree curl net-tools gnupg2 rfkill util-linux nmap tcpdump binutils screen pv ntfs-3g
vim /etc/ssh/sshd_config
systemctl restart ssh

# Commandes basées sur tuto
# Notes : 
# - installation XFCE4 par tasksel comme à l'installation
# - user xrdp conforme à ce qui est dit
# - dans mon cas pas de FW à configurer (ufw non installé)
# - protection PolKit gérée à la fin
apt install tasksel
tasksel
apt install -y xfce4-goodies xorg dbus-x11 x11-xserver-utils
apt install -y xrdp
systemctl status xrdp
grep xrdp /etc/passwd
ls -l /etc/ssl/private/ssl-cert-snakeoil.key
groups xrdp
adduser xrdp ssl-cert
groups xrdp
systemctl restart xrdp
man ufw
vim /etc/polkit-1/localauthority.conf.d/02-allow-colord.conf
```
02-allow-colord.conf contiendra (copié depuis source) :
```
polkit.addRule(function(action, subject) {
if ((action.id == "org.freedesktop.color-manager.create-device" ||
 action.id == "org.freedesktop.color-manager.create-profile" ||
 action.id == "org.freedesktop.color-manager.delete-device" ||
 action.id == "org.freedesktop.color-manager.delete-profile" ||
 action.id == "org.freedesktop.color-manager.modify-device" ||
 action.id == "org.freedesktop.color-manager.modify-profile" ||
 action.id == "org.freedesktop.packagekit.system-sources-refresh" || action.id == "org.freedesktop.packagekit.system-network-proxy-configure") &&
 subject.isInGroup("{users}")) {
 return polkit.Result.YES;
 }
});
```

Problèmes ?

Voir instructions DEBUG dessus

# RDP to Windows Home

En vrac avant Gitlab

Source qui m'a amené à tester : https://www.ctrl.blog/entry/how-to-rdpwrapper-win10-home.html

RDP Wrapper permet d'accéder en RDP à une édition Home de Windows 7+ (donc pas Seven, ni XP) alors que Microsoft l'en empêche pour obliger les utilisateurs à se tourner vers des éditions plus chères (Pro, etc.)

Site : https://github.com/stascorp/rdpwrap

Légal ? Ce point fait litige ... (https://github.com/stascorp/rdpwrap/issues/26)

Installer (https://superuser.com/questions/1548272/difficulty-installing-rdpwrap-on-windows-10-to-get-multiple-remote-desktop-conn/1548312#1548312)

Récupérer l'archive zip des applications (dans mon cas : RDPWrap-v1.6.2.zip) et décompresser

Depuis CLI, exécuter le script install.bat

Si tout se passe bien (Successfully installed), on devrait avoir :
C:\Program Files\RDP Wrapper\rdpwrap.ini
C:\Program Files\RDP Wrapper\rdpwrap.dll

Note : si l'installation échoue, voir plus bas les cas d'erreur

Ensuite il faut vérifier RDPConf :
Wrapper state: Installed        ver. 1.5.0.0
Service state: Running          ver: 10.0.19041.789
Listener state: Not listening   [not supported]

Confirmé en CLI par :
wmic datafile where name="c:\\Windows\\System32\\termsrv.dll" get version
Version
10.0.19041.789

Stopper le service Terminal Server : net stop TermService

Ajouter au fichier C:\Program Files\RDP Wrapper\rdpwrap.ini
la configuration add_to_ini_for_10.0.19041.789_version_dll.txt (voir https://github.com/stascorp/rdpwrap/issues/1342#issuecomment-794558507)

Redémarrer : net start TermService

Erreurs

si lors de l'installation on a "Accès refusé", j'ai déjà résolu le problème en excluant le dossier de la Protection contre les virus et menaces de Windows 10

Voir ce lien pour plus de détails concernant la configuration dans le fichier INI : https://github.com/stascorp/rdpwrap/issues/1342#issuecomment-1334016736

# tigervnc
Brouillon/vrac

TODO : curseur de la souris parceque le point ... BOF (un patch [ici](https://github.com/TigerVNC/tigervnc/issues/1335))

TODO : en contexte multi-écran, prendre à distance un seul écran et définir lequel

TODO : gestion correcte d'un clavier FR (Juste pourquoi ? C'est quoi l'intérêt de faire un logiciel qui ne gère même pas correctement les frappes au clavier (voir [ici](https://github.com/TigerVNC/tigervnc/issues/528) et [là](https://bugzilla.redhat.com/show_bug.cgi?id=1437569) qui indique d'utiliser le paramètre **-RawKeyboard** ; la [doc officielle](https://tigervnc.org/doc/x0vncserver.html) de **x0vncserver** le propose mais apparemment sous Debian 9 Stretch ce n'est apparemment pas supporté !) => en attendant LA solution on peut utiliser un clavier virtuel comme [florence](https://www.xmodulo.com/onscreen-virtual-keyboard-linux.html)

[1] Erreur "No password configured for VNC Auth" => [Solution non sécure](https://github.com/TigerVNC/tigervnc/issues/457#issuecomment-301099245)
```sh
# Solution un peu plus "clés en main" (basé sur dessous)

# Notes : 
# - sur $SERVEUR : tigervnc-scraping-server permet de prendre à distance une session en cours
# - sur $SERVEUR : x0vncserver ... -SecurityTypes=none (voir [1])
# - F8 permet d'afficher le menu 

# Serveur (le système distant avec un serveur X installé ; ici Debian 9 Stretch)
 # en root
  502  apt update
  503  apt install -y tigervnc-standalone-server tigervnc-common dbus-x11
  504  apt install -y tigervnc-scraping-server
  505  su - $USER
 # en NON root
 1216  x0vncserver -display :0.0 -SecurityTypes=none
Tue Jul 26 16:47:27 2022
 Geometry:    Desktop geometry is set to 3840x1080+0+0
 Main:        XTest extension present - version 2.2
 Main:        Listening on port 5900 
 
# Client (le viewer ; ici Debian 10 Buster)
  502  apt update
  484  apt search tiger
tigervnc-viewer/oldstable,now 1.9.0+dfsg-3+deb10u3 amd64  [installé]
  logiciel client d’informatique virtuelle en réseau (VNC) pour⋅X
  485  apt install tigervnc-viewer/oldstable
  524  xtigervncviewer -h 2>&1 | grep -i cursor
DotWhenNoCursor - Show the dot cursor when the server sends an invisible
                   cursor (default=0)
  512  xtigervncviewer -SecurityTypes None -DotWhenNoCursor $SERVER:0
TigerVNC Viewer 64-bit v1.9.0
Built on: 2020-09-29 18:21
Copyright (C) 1999-2018 TigerVNC Team and many others (see README.rst)
See http://www.tigervnc.org for information on TigerVNC.

Tue Jul 26 16:54:40 2022
 DecodeManager: Detected 4 CPU core(s)
 DecodeManager: Creating 4 decoder thread(s)
 CConn:       connected to host bt port 5900
 CConnection: Server supports RFB protocol version 3.8
 CConnection: Using RFB protocol version 3.8
 CConnection: Choosing security type None(1)
 CConn:       Using pixel format depth 24 (32bpp) little-endian rgb888
 CConn:       Using Tight encoding
 CConn:       Enabling continuous updates
 CConn:       SetDesktopSize failed: 1
 ```



```sh
# TODO : comprendre pourquoi "-localhost no" est nécessaire pour démarrer le serveur

# SERVER non ROOT

	# Ouvrir session graphique et depuis une console obtenir l'identifiant de session graphique

	echo $DISPLAY
	:0.0

	# Mémoriser cet identifiant


# CLIENT non ROOT

	user@server:~$ ssh root@vm-bullseye-xfce


# SERVER ROOT

   19  sed -i "s/$HOSTNAME/server/g" /etc/hostname   
   20  sed -i "s/$HOSTNAME/server/g" /etc/hosts
   hostnamectl set-hostname server
   systemctl restart avahi-daemon
   

#  SI PRISE A DISTANCE AVEC NOUVELLE SESSION

	# https://computingforgeeks.com/install-and-configure-tigervnc-vnc-server-on-debian/
   
	#  si DE xfce absent (inutile si on ne veut pas prendre à distance x0 ; soit ouvrir une session existante pour prendre la main à distance)

	#  Manuellement avec le même programme qu'à l'installation du système   

   22  apt update   
   23  apt install tasksel -y
   24  tasksel
      
	#  OU automatiquement
	# https://www.techlear.com/blog/2022/02/09/how-to-install-vnc-server-on-debian-11/
   
   sudo apt install -y task-xfce-desktop
   
   
	# pas sur que nécessaire !
   
   25  systemctl set-default graphical.target
   26  reboot


	# dbus-x11 semble nécessaire pour empecher les erreurs de connection de socket
	# https://askubuntu.com/questions/1209147/tigervncviewer-unable-to-connect-to-socket-connection-refused-10061
	
   32  apt install -y tigervnc-standalone-server tigervnc-common dbus-x11
   
   
#  SI PRISE A DISTANCE SESSION EXISTANTE X0

	# https://serverfault.com/questions/27044/how-to-vnc-into-an-existing-x-session/924326#924326

	# note : x0vncserver est installé avec tigervnc-scraping-server (le reste n'est pas nécessaire)

	apt install -y tigervnc-scraping-server



#  DEMARRER SERVER AVEC USER non ROOT
   
   28  su - user


# SERVER non ROOT

#  SI PRISE A DISTANCE AVEC NOUVELLE SESSION

	vncserver -localhost no
	
	user@server:~$ vncserver -localhost no
	You will require a password to access your desktops.

	Password:
	Verify:
	Would you like to enter a view-only password (y/n)? n
	
	New Xtigervnc server 'server:2 (user)' on port 5902 for display :2.
	Use xtigervncviewer -SecurityTypes VncAuth,TLSVnc -passwd /home/user/.vnc/passwd server:2 to connect to the VNC server.


#  SI PRISE A DISTANCE SESSION EXISTANTE X0

	# Comme pour vncserver si on ne met pas "-localhost no" on a cette erreur :
	#  erreur : unable to connect to socket: Connection refused (10061)
	# https://askubuntu.com/questions/1209147/tigervncviewer-unable-to-connect-to-socket-connection-refused-10061

	user@server:~$ x0vncserver -localhost no -display :0.0

	You will require a password to access your desktops.

	Password:
	Verify:
	Would you like to enter a view-only password (y/n)? n

	New X0tigervnc server 'server:0 (user)' on port 5900 for display :0.
	Use xtigervncviewer -SecurityTypes VncAuth,TLSVnc -passwd /home/user/.vnc/passwd server:0 to connect to the VNC server.

	user@server:~$ x0vncserver -localhost no -display :0

	New X0tigervnc server 'server:0 (user)' on port 5900 for display :0.
	Use xtigervncviewer -SecurityTypes VncAuth,TLSVnc -passwd /home/user/.vnc/passwd server:0 to connect to the VNC server.



# CLIENT non ROOT




#  SI PRISE A DISTANCE SESSION EXISTANTE X0

	# Copier/coller de la commande indiquée par x0vncserver
	#  MAIS "server" n'étant par résolvable ici (avahi), on le modifie en "server.local"


	# Mot de passe interactif

	user@server:~$ xtigervncviewer -SecurityTypes VncAuth,TLSVnc server.local:0


	# OU mot de passe automatique

	user@server:~$ vncpasswd 
	Password:
	Verify:
	Would you like to enter a view-only password (y/n)? n

	user@server:~$ xtigervncviewer -SecurityTypes VncAuth,TLSVnc -passwd /home/user/.vnc/passwd server.local:0

	Visionneuse TigerVNC 64 bits v1.11.0
	Compilé sur : 2021-03-22 21:21
	Copyright © 1999-2020 L’équipe de TigerVNC et beaucoup d’autres (voir README.txt)
	Voir https://www.tigervnc.org pour plus d’informations sur TigerVNC.

	Sat Jun  4 17:23:47 2022
	 DecodeManager: Detected 4 CPU core(s)
	 DecodeManager: Creating 4 decoder thread(s)
	 CConn:       Connecté à l’hôte server.local par le port 5900
	 CConnection: Server supports RFB protocol version 3.8
	 CConnection: Using RFB protocol version 3.8
	 CConnection: Choosing security type VeNCrypt(19)
	 CVeNCrypt:   Choosing security type VncAuth (2)
	 CConn:       Utilisation du format de pixel depth 24 (32bpp) little-endian
				  rgb888
	 CConnection: Enabling continuous updates

	Sat Jun  4 17:23:51 2022
	 CConn:       SetDesktopSize échoué : 3


#  STOPPER SERVER AVEC USER non ROOT


#  SI PRISE A DISTANCE SESSION EXISTANTE X0

	user@server:~$ x0vncserver -list

	TigerVNC server sessions:

	X DISPLAY #	RFB PORT #	PROCESS ID	SERVER
	:0         	5900      	1403      	X0tigervnc
	user@server:~$ x0vncserver -kill :0
	Killing X0tigervnc process ID 1403... success!
















































user@server:~$ history 
    1  su -
    2  sudo -i
    3  su -
    4  ping localhost
    5  su -
    6  vncpasswd 
    7  vncserver -localhost no
    8  vncserver -list
    9  vncserver -kill :1
   10  vncserver
   11  vncserver -list
   12  vncserver -kill :1
   13  vim .vnc/xstartup
   15  chmod 777 .vnc/xstartup 
   16  vncserver
#   17  tigervncserver -xstartup /usr/bin/xterm
   18  vncserver -list
   19  vncpasswd 
   20  vncserver -list
   21  vncserver -kill :1
   22  vncserver -localhost no
#   23  tigervncserver -xstartup /usr/bin/xterm
   24  ps aux | grep vnc
   25  vncserver -list
   26  vncserver -kill :1
   27  ps aux | grep vnc
   28  vncserver
   29  vim /home/user/.Xresources
   
! Color theme to be placed in ~/.Xresources file
! Enable it at runtime with :
! $ xrdb ~/.Xresources
! or
! $ cat ~/.Xresources | xrdb

 *background: #000000
 *foreground: #ffffff
 *color0:     #000000
 *color1:     #d36265
 *color2:     #aece91
 *color3:     #e7e18c
 *color4:     #7a7ab0
 *color5:     #963c59
 *color6:     #418179
 *color7:     #bebebe
 *color8:     #666666
 *color9:     #ef8171
 *color10:    #e5f779
 *color11:    #fff796
 *color12:    #4186be
 *color13:    #ef9ebe
 *color14:    #71bebe
 *color15:    #ffffff


   30  vncserver -list
   31  vncserver
#   32  tigervncserver -xstartup /usr/bin/xterm
   33  ip ad
   34  vncserver -list
   38  vncserver -kill :1
   39  vncserver
   40  vncserver -list
   43  vim .vnc/xstartup
#!/bin/bash
xrdb $HOME/.Xresources
# Session startup via '~/.vnc/xstartup' cleanly exited too early (< 3 seconds)
# https://ubuntuforums.org/showthread.php?t=2470276
#startxfce4 &
startxfce4

   44  vncserver -kill :1
   45  vncserver -list
   46  vncserver
   47  vncserver -list
   48  sudo iptables --list
   49  su -
   50  vncserver --kill :*
   51  reboot
   52  su -
   53  vncserver -list
   
   
# erreur : unable to connect to socket: Connection refused (10061)
# https://askubuntu.com/questions/1209147/tigervncviewer-unable-to-connect-to-socket-connection-refused-10061
   54  vncserver -localhost no
   55  history 


root@server:~# history
   19  echo server > /etc/hostname 
   29  vim /etc/hosts
   20  reboot
   21  apt update
   22  apt install tasksel -y 
   24  tasksel
   25  systemctl set-default graphical.target
   26  reboot
   27  apt install tigervnc-standalone-server tigervnc-common dbus-x11
   28  su - user
   
   
   
   
   

   31  reboot
   
# https://askubuntu.com/questions/1209147/tigervncviewer-unable-to-connect-to-socket-connection-refused-10061
 
#   32  apt install dbus-x11
   34  iptables --list
   35  ufw status
   
# comme netstat ...
   36  lsof -P -i
   37  vncserver --kill :*
   38  reboot
   39  history 

# CLIENT   
   
# KO : ssh -L 5901:127.0.0.1:5901  debian server_ip
user@server:~$ ssh -L 5901:127.0.0.1:5901 user@server.local
user@server:~$ xtigervncviewer -SecurityTypes VncAuth,TLSVnc server.local:1

```
# x2go sous Linux
Pas assez mature, à mon sens.

Nouvelle technologie Open Source pour l'accès distant que je viens de découvrir, tester et que trouve très pratique. Entre autre, rapide à installaer, on peut ouvrir une session en cours en lecture seule (pratique pour remplacer NoMachine pour venir en aide à un utilisateur), on peut affiner les contraintes réseau (LAN/ADSL/etc. et la compression) pour avoir une configuration plus réactive. Aussi le copier/coller est fonctionnel nativement (en tout cas dans mon test).

Installer le serveur qui sera accédé de manière distante et le client sur une machine locale (pour l'instant basé sur [ceci](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-remote-desktop-with-x2go-on-debian-10?comment=89552).

Nota : depuis le [site officiel](https://wiki.x2go.org/doku.php/wiki:repositories:debian), je n'ai pas réussi à importer la clé
Installation du serveur

Installer le serveur (machine distante) :
```sh
root@remote_host:~# apt install x2goserver x2goserver-xsession
```
Installer le client (machine locale) :
```sh
root@local_host:~# apt install x2goclient
```
Nota : il y aura sans doûte bientôt la possibilité d'utiliser Remmina comme client x2go (voir [ici](https://remmina.org/x2go/))

# x11vnc sous Linux
Accéder à distance à un PC comme si on y était : on peut aider l'utilisateur ou simplement éviter de se déplacer (même souris, même affichage).

Testé avec succès pour accéder à distance à un notebook (Emmabuntüs DE 3 Debian Buster 32 bits avec gestionnaire de session LXQt) depuis mon PC perso avec Remmina (Ubuntu 16.04 LTS 64 bits).

## Démarrer le serveur
Basé sur [ceci](https://debian-facile.org/doc:reseau:x11vnc)

Depuis le PC distant "serveur VNC" (en root) :
```sh
apt update && apt install -y x11vnc
mkdir $HOME/.vnc
x11vnc -storepasswd ~/.vnc/passwd
# configurer le mot de passe pour accéder à la session VNC avec root
pgrep -a Xorg | grep -o '[-]auth [^ ]*'
# -auth /var/run/lightdm/root/:0
# -auth .Xauthority
# -display :0 car un seul écran
x11vnc -display :0 -auth /var/run/lightdm/root/:0
# ...
# The VNC desktop is:      pc-session2:0
# PORT=5900
# ...
# => en attente de connexion distante (ctrl+c pour stopper le serveur VNC)
```
Si on le souhaite, on peut démarrer automatiquement en tant que service (basé sur [ceci](https://debian-facile.org/doc:reseau:x11vnc#au-demarrage-du-systeme-avec-les-services-systemd)) :
```sh
root@host:~# cat /etc/systemd/system/x11vnc.service
[Unit]
Description=Service x11vnc
Requires=display-manager.service
After=display-manager.service
 
[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -forever -display :0 -auth /var/run/lightdm/root/:0

Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
root@host:~# systemctl daemon-reload && systemctl start x11vnc && systemctl enable x11vnc
```

## Installer le client
Depuis le PC local "client VNC", on effectue une connexion sur PC distant (server) sur le port (1) avec l'utilisateur root et son mot de passe (2).

Voici la partie réduite de la configuration de la connexion sur Remmina :
```sh
user@client:~$ cat ~/.remmina/xxx.remmina 
[remmina]
name=server
server=server.local:5900
username=root
password=xxxx=
```

(1)
Indiqué par la commande x11vnc du serveur

(2)
Configuré sur le serveur

# Serveur VNC sous Windows
Je recommande [TigerVNC](https://www.tightvnc.com/) qui est une solution Open Source facile à installer pour accéder à distance et est nécessaire quand on utilise une version de Windows qui n'autorise pas d'être utilisé en bureau à distance, donc qui n'autorise pas le RDP (cas de Windows Home). 

Pour configurer : click droit sur icône dans systray, en choisissant soit avec mot de passe, soit avec autorisation de l'hôte.

ping vnc-server.local doit résoudre l'IP et si ça s'arrête là, il est probable que le parefeu ne laisse pas passer => le désactiver ou créer une règle
# XRDP sous Linux
J'ai suivi [cette procédure](https://blog.eldernode.com/install-xrdp-on-debian-10-buster/) juste en installant et en vérifiant que xrdp était actif. Tout était OK.

J'ai remarqué un bug pénible : quand on a ouvre une session RDP distante et qu'on la ferme ensuite, puis qu'on tente de la rouvrir localement, la session ne s'ouvre pas et on revient à l'écran de login. Redémarrer fonctionne mais est peu pratique. J'ai trouvé [cette solution](https://askubuntu.com/questions/1054063/local-ubuntu-desktop-cannot-login-after-opened-xrdp-session/1054758#1054758) qui est censé rendre disponible une nouvelle connexion après 60s :
Modifier **KillDisconnected=yes** dans le fichier **/etc/xrdp/sesman.ini** (reboot nécéssaire ou **systemctl restart xrdp** suffisant ?)

=> fonctionne bien et en plus j'ai l'impression qu'il n'y a pas besoin d'attendre 60s avant la prochaine connexion (parfois oui, parfois non)
Par contre je suis obligé de déconnecter la session locale avant d'accèder à distance (sudo pkill -KILL -u mon_user pour fermer à distance)
