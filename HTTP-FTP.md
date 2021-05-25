# Mise en place du conteneur HTTP + FTP

- [Mise en place du conteneur HTTP + FTP](#mise-en-place-du-conteneur-http--ftp)
  - [Installation de Debian](#installation-de-debian)
  - [Installation de Docker sur le serveur](#installation-de-docker-sur-le-serveur)
    - [Mise à jour et installation de paquets de base pour Docker](#mise-à-jour-et-installation-de-paquets-de-base-pour-docker)
    - [Ajout du dépôt Docker](#ajout-du-dépôt-docker)
  - [Création du conteneur](#création-du-conteneur)
  - [Préparation du FTP dans le conteneur](#préparation-du-ftp-dans-le-conteneur)
    - [Installation des paquets de base du conteneur](#installation-des-paquets-de-base-du-conteneur)
    - [Configuration](#configuration)
      - [Création d'un utilisateur `ftp` sans droits](#création-dun-utilisateur-ftp-sans-droits)
      - [Vous n'avez pas le droit Monsieur !](#vous-navez-pas-le-droit-monsieur-)
      - [Appliquer les modifications](#appliquer-les-modifications)

## Installation de Debian

![](img/debian-logo.png)

Suivre la procédure d'installation de Debian de base, non graphique avec simplement un serveur SSH et les dossiers `/var`, par exemple; séparés.

## Installation de Docker sur le serveur

![](img/docker-logo.png)

### Mise à jour et installation de paquets de base pour Docker

Une fois l'installation de Debian sans GUI complétée, se connecter avec le compte mettre à jour la liste des paquets et les paquets:

    apt update
    apt upgrade

Installer ensuite les paquets:

    apt install apt-transport-https ca-certificates curl gnupg lsb-release

### Ajout du dépôt Docker

Télécharger la clé GPG de Docker:

    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

Ajouter le dépôt de Docker à la liste des dépots de Debian:

```bash
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Enfin, installer les paquets Docker:

    apt update
    apt install docker-ce docker-ce-cli containerd.io


## Création du conteneur

![](img/container-icon.png)


Télécharger la dernière version de l'image Debian pour Docker:

    docker pull debian

> Il est tout à fait possible d'utiliser une autre base que Linux pour faire le même conteneur. Alpine Linux, par exemple, est idéal de par sa taille (6MB) pour créer des petits conteneurs.

Notre conteneur aura besoin d'avoir les ports HTTP et FTP d'ouverts (`80, 20, 21`).

Créer un conteneur en ouvrant les ports nécessaires à partir d'une image Debian :

    docker run -dit --name web-server -p 20:20 -p 21:21 -p 80:80 debian

On pourra se connecter sur notre conteneur et continuer la préparation avec la commande:

    docker exec -it web-server bash


## Préparation du FTP dans le conteneur

### Installation des paquets de base du conteneur
Une fois dans le conteneur, mettre à jour la liste des paquets et installer `nginx`, `vsftpd` ainsi que `nano` pour éditer les fichiers: 

    apt update
    apt install nginx vsftpd nano



### Configuration

Utiliser les fichiers de configurations suivants pour nginx et vsftpd

#### Création d'un utilisateur `ftp` sans droits

Pour créer un utilisateur sans shell et sans possibilité d'intéragir avec Linux, il nous faut d'abord déclarer `nologin` comme étant un "shell" valide:

    nano /etc/shells

Ajouter sur une nouvelle ligne (si pas encore existant): `/usr/sbin/nologin`

Créer un utilisateur pour utiliser FTP avec:

    useradd --shell /usr/sbin/nologin Toto

#### Vous n'avez pas le droit Monsieur !

Pour des raisons de droits, il sera impossible pour notre nouvel utilisateur d'accéder ou de modifier le contenu du dossier `/usr/share/nginx/`.

Rendre l'utilisateur propriétaire du dossier et donner les droits totaux à Toto sur le dossier:

    chown /usr/share/nginx/html Toto
    chmod 777 /usr/share/nginx/html

> Le processus nginx doit être exécuté avec le nom d'utilisateur du FTP pour lui garantir l'accès en lecture !


#### Appliquer les modifications

Redémarrer (ou démarrer) avec les commandes:

    etc/init.d/vsftpd (re)start
    etc/init.d/nginx (re)start