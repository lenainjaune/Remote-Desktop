# Remote-Desktop
Ici des solutions simples pour accéder à distance à un PC.

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
