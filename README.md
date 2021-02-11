# Remote-Desktop
Ici des solutions simples pour accéder à distance à un PC.

# x11vnc
Accéder à distance à un PC comme si on y était : on peut aider l'utilisateur ou simplement éviter de se déplacer (même souris, même affichage).

Testé avec succès pour accéder à distance à un notebook (Emmabuntüs Debian Buster 32 bits avec gestionnaire de session LightDM) depuis mon PC perso avec Remmina (Ubuntu 16.04 LTS 64 bits).

## Installer
Basé sur [ceci](https://debian-facile.org/doc:reseau:x11vnc)

Depuis le PC distant "serveur VNC" (en root) :
```sh
apt update && apt install x11vnc
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

Depuis le PC local "client VNC", on effectue une connexion sur PC distant sur le port (1) avec l'utilisateur root et son mot de passe (2).

Voici la partie réduite de la configuration de la connexion sur Remmina :
```sh
user@host:~$ cat ~/.remmina/xxx.remmina 
[remmina]
name=pc-remote
server=pc-remote.local:5900
username=root
password=xxxx=
```



(1)
Indiqué par la commande x11vnc du serveur

(2)
Configuré sur le serveur
