# Implémentation d’une infrastructure réseau

- [Implémentation d’une infrastructure réseau](#implémentation-dune-infrastructure-réseau)
  - [Schéma, plan et adressage IP de l’infrastructure](#schéma-plan-et-adressage-ip-de-linfrastructure)
  - [Solutions choisies](#solutions-choisies)
- [Mots de passe et identifiants](#mots-de-passe-et-identifiants)
    - [Serveur Rennes (`rennes-svr`)](#serveur-rennes-rennes-svr)
    - [Serveur DMZ (`SERVEUR-DMZ`)](#serveur-dmz-serveur-dmz)
    - [Ferme Proxmox](#ferme-proxmox)
    - [FTP](#ftp)
    - [DHCP](#dhcp)
    - [Active Directory](#active-directory)
    - [PHP My Admin](#php-my-admin)
    - [Serveur Mail](#serveur-mail)
  - [Switches et routeurs](#switches-et-routeurs)
- [Mise en place du conteneur HTTP + FTP](#mise-en-place-du-conteneur-http--ftp)
  - [Prérequis](#prérequis)
  - [Installation de Docker sur Debian 10](#installation-de-docker-sur-debian-10)
    - [Mise à jour et installation de paquets de base pour Docker](#mise-à-jour-et-installation-de-paquets-de-base-pour-docker)
    - [Ajout du dépôt Docker](#ajout-du-dépôt-docker)
  - [Création du conteneur](#création-du-conteneur)
  - [Préparation du FTP dans le conteneur](#préparation-du-ftp-dans-le-conteneur)
    - [Installation des paquets de base du conteneur](#installation-des-paquets-de-base-du-conteneur)
    - [Configuration](#configuration)
      - [Création d'un utilisateur `ftp` sans droits](#création-dun-utilisateur-ftp-sans-droits)
      - [Vous n'avez pas le droit Monsieur !](#vous-navez-pas-le-droit-monsieur-)
      - [Appliquer les modifications](#appliquer-les-modifications)
  - [Préparation de Nginx dans le conteneur](#préparation-de-nginx-dans-le-conteneur)
- [Configurer un serveur DHCP sur Debian 10](#configurer-un-serveur-dhcp-sur-debian-10)
  - [Kézako ?](#kézako-)
  - [Prérequis](#prérequis-1)
  - [Installation d'ISC DHCP Server:](#installation-disc-dhcp-server)
  - [Indiquer l'interface réseau à utiliser](#indiquer-linterface-réseau-à-utiliser)
  - [Spécifier les options du DHCP](#spécifier-les-options-du-dhcp)
    - [Définir le DNS distribué par le DHCP](#définir-le-dns-distribué-par-le-dhcp)
    - [Définir l'IP réseau et le masque de sous-réseau](#définir-lip-réseau-et-le-masque-de-sous-réseau)
    - [Définir la plage d'IP à distribuer](#définir-la-plage-dip-à-distribuer)
    - [Définir la passerelle](#définir-la-passerelle)
  - [Démarrage et test](#démarrage-et-test)
  - [Ça ne marche pas ?](#ça-ne-marche-pas-)
    - [Regarder dans les logs](#regarder-dans-les-logs)
    - [Erreurs répétées](#erreurs-répétées)
- [Configuration de l'infrastructure](#configuration-de-linfrastructure)
  - [Résumé de la configuration des Switchs](#résumé-de-la-configuration-des-switchs)
  - [Résumé de la configuration des Routeurs](#résumé-de-la-configuration-des-routeurs)
  - [Commandes communes à tous les équipements](#commandes-communes-à-tous-les-équipements)
    - [Hostname](#hostname)
    - [Sécuriser l'accès privilégié](#sécuriser-laccès-privilégié)
    - [Configuration Telnet](#configuration-telnet)
  - [Commandes et procédures des switchs en détail](#commandes-et-procédures-des-switchs-en-détail)
  - [Commandes et procédures des routeurs en détail](#commandes-et-procédures-des-routeurs-en-détail)
    - [Configurer les interfaces](#configurer-les-interfaces)
    - [Configurer RIPv2 (routage dynamique)](#configurer-ripv2-routage-dynamique)
      - [Étapes dans l'ordre](#étapes-dans-lordre)
      - [Exemple en déclarant trois réseaux](#exemple-en-déclarant-trois-réseaux)
    - [Configurer le protocole HSRP](#configurer-le-protocole-hsrp)
      - [Étapes dans l'ordre](#étapes-dans-lordre-1)
      - [Exemple](#exemple)
- [Proxmox](#proxmox)
  - [Connexion](#connexion)
- [Fichiers de configuration](#fichiers-de-configuration)
  - [Routeurs](#routeurs)
  - [Switches](#switches)
  - [Docker](#docker)
    - [/etc/nginx/nginx.conf](#etcnginxnginxconf)
    - [/etc/nginx/conf.d/default.conf](#etcnginxconfddefaultconf)
    - [/etc/vsftpd.conf](#etcvsftpdconf)

## Schéma, plan et adressage IP de l’infrastructure

Nous avons choisi d'utiliser le nom de domaine `ggp.local`

![](adressage.svg)

Connectivité entre les différents matériels:

- RG1 à RW1 PORT SERIAL 1 ↔ PORT SERIAL 0
- RG2 à RW1 PORT SERIAL 0 ↔ PORT SERIAL 1
- RW1 à RW2 PORT FA 0/0 ↔ PORT GI 0/1
- RW2 à RR1 PORT GI 0/0 ↔ PORT FA 0/1
- RR1 à SR1 PORT FA0/0 ↔ FA 0/1-24
- RG1 à SG1 PORT FA 0/0 ↔ FA0/1-24 

## Solutions choisies

Nos solutions tournent sur un mix de machines virtuelles, de conteneurs et de bare-metal.

Nos machines virtuelles tournent sur une ferme Proxmox tandis que nos conteneurs tournent à l'aide de Docker.

Nous avons choisi d'utiliser Debian la majorité du temps car considéré comme "standard". 

|Service|Serveur|Solution|Environnement d'exécution|OS / Image|IP/masque
|---|---|---|---|---|---|
|Active Directory + DNS|||Proxmox 6.4 (VM)|Windows Server 2019||
|Mail|||Proxmox 6.4 (VM)|Debian 10||
|DHCP||ISC|Proxmox 6.4 (VM)|Debian 10||
|HTTP + FTP||Nginx + Vsftpd|Conteneur Docker 20.x (Serveur Debian 10)|Image Debian 10||
|Base de données||MariaDB|Bare-metal|Debian 10||
|Extranet|||Bare-metal|Debian 10||
|Pare-feu||PfSense 2.5.1 |Bare-metal|PfSense 2.5.1 (FreeBSD)||

---

# Mots de passe et identifiants

![](img/Password.png)

Fichier rassemblant l'ensemble des mots de passes pour les routeurs, switches et serveurs de l'infrastructure

Le nom entre parethèses correspond au hostname du serveur ou de l'équipement.

### Serveur Rennes (`rennes-svr`)

| User | Password |
|---|---|
|root|root|
|admin-rennes|admin|


### Serveur DMZ (`SERVEUR-DMZ`)

| User | Password |
|---|---|
|root|root|
|admin-dmz|admin|

### Ferme Proxmox

| User | Password |
|---|---|
|root|admin|

### FTP

| User | Password |
|---|---|
|usertest|test|

### DHCP

| User | Password |
|---|---|
|admin-dhcp||

### Active Directory

| User | Password |
|---|---|
|Administrateur|Sio1$|
|client1|Test123!|
|client2|Test123!|
|client3|Test123!|

### PHP My Admin


### Serveur Mail

| User | Password |
|---|---|
|root|root|


## Switches et routeurs
|Connexion|Mot de passe|
|---|---|
|Telnet | admin|
|Local Privilégié| cisco |

---

# Mise en place du conteneur HTTP + FTP

## Prérequis

- Installation Debian fonctionnelle
- Configuration et/ou interface réseau fonctionnelle avec un accès à Internet

## Installation de Docker sur Debian 10

![](img/docker-logo.png)

Docker ne se trouvant malheureusement pas dans les dépôts officiels Debian, il faut manuellement ajouter le dépôt et un certain nombre de dépendances pour pouvoir y télécharger les paquets nécessaires.

### Mise à jour et installation de paquets de base pour Docker

Mettre à jour les paquets et la liste des paquets:

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


## Préparation de Nginx dans le conteneur

Maintenant que les

---

# Configurer un serveur DHCP sur Debian 10

## Kézako ?

Un serveur DHCP (Dynamic Host Configuration Protocol) a pour but de délivrer automatiquement une configuration IP valide aux divers équipements qui se connectent sur un réseau.

Pour rappel, un serveur DHCP doit déliver impérativement ces 3 choses au client: 
- Une adresse IP
- Un temps de bail, c'est à dire une durée de validité de l'adresse IP donnée
- Un masque de sous réseau, sans quoi l'adresse IP est inexploitable   

Optionnellement, il peut distribuer l'adresse d'un serveur DNS, mais aussi une passerelle. Notre DHCP distribuera ces 5 choses.


## Prérequis

- Installation Debian fonctionnelle
- Configuration et/ou interface réseau fonctionnelle avec un accès à Internet
- Accès Root

> Avant d'éditer chaque fichier, nous en ferons une sauvegarde afin de pouvoir retrouver un fichier exploitable en cas de pepin.  
> Nous ferons simplement une copie du fichier en rajoutant un `.old` à la fin du nom de ce dernier


## Installation d'ISC DHCP Server:
Mettre à jour la liste des paquets et installer `isc-dhcp-server`

    apt update  
    apt install isc-dhcp-server

Un message d'erreur provenant du serveur DHCP s'affichera juste à la fin de l'installation. 
Le serveur a en effet essayé de démarrer mais n'a pas réussi, ce qui est tout à fait normal puisqu'il n'est pas encore été configuré.


## Indiquer l'interface réseau à utiliser

Backup:  
``cp /etc/default/isc-dhcp-server /etc/default/isc-dhcp-server.old``

Éditer le fichier:  
``nano /etc/default/isc-dhcp-server``

Spécifier vers le bas de celui-ci l'interface réseau entre les guillemets de la ligne ``INTERFACESv4=""``  
Commenter la ligne ``INTERFACESv6`` puisque nous n'utiliserons pas d'IPv6.  
Exemple avec enp0s3:

![interface](img/DHCP/interface.png)

Quitter avec `CTRL+X`, puis confirmer pour écraser le fichier.

## Spécifier les options du DHCP
Backup:  
``cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.old``

Éditer le fichier:  
``nano /etc/dhcp/dhcpd.conf``

Nous allons seulement changer les options qui seront vraiment nécessaires pour que le DHCP puissse tourner:

### Définir le DNS distribué par le DHCP
Vers le haut du fichier ``dhcpd.conf``, il sera possible de définir un nom de domaine (``domaine-name``) ainsi qu'un ou plusieurs DNS (``domaine-name-servers``).
Nous mettrons un domaine en ``quelquechose.local`` et un DNS comme ``1.1.1.1``, celui de CloudFlare.   
Les DNS doivent être séparés par une virgule si l'on souhaite en mettre plusieurs:

![DNS](img/DHCP/DNS.png)

Il est possible de modifier les temps de bail (``default-lease-time`` et ``max-lease-time``). Ces temps sont donnés en secondes.

### Définir l'IP réseau et le masque de sous-réseau
Nous allons décommenter (retirer les #) autour de la ligne 30 de sorte à avoir ceci:

![uncomment](img/DHCP/uncomment.png)

Nous pouvons ensuite sur cette ligne (la seule en blanche sur l'image) changer l'adresse IP et le masque pour correspondre à notre réseau.

### Définir la plage d'IP à distribuer
Nous allons ajouter cette option sur une nouvelle ligne entre les crochets. Elle se présente ainsi:  
``range adresse_IP_début adresse_IP_fin;``  

En sachant que les adresses IP début et fin sont distribuées.

Ne pas oublier le point virgule ( ; ) en fin de ligne !

Le résultat devrait ressembler à ceci:

![result](img/DHCP/result.png)

### Définir la passerelle

Comme pour la plage d'IP nous allons ajouter une option:  
``option routers IP_passerelle;``

![gateway](img/DHCP/gateway.png)

Enfin, écraser le fichier, et confirmer.

## Démarrage et test

Redémarrer le serveur avec:  
``systemctl restart isc-dhcp-server``

Regarder si le serveur est fonctionnel avec:  
``systemctl status isc-dhcp-server``

![bingo](img/DHCP/bingo.png)

On voit ici que le serveur est en train de tourner et qu'il ne met pas d'erreurs.


## Ça ne marche pas ?

### Regarder dans les logs 
Afficher les logs à partir du bas pour voir ce qui ne va pas:
``tail -n 25 /var/log/syslog``

> Il s'agira très souvent d'un `;` manquant, d'une petite faute de frappe dans le fichier de configuration ou d'une interface mal déclarée

``tail`` Affiche à partir du bas du fichier (le fichier log peut-être TRÈS LONG).  
``-n`` Spécifie le nombre de lignes à récupérer

### Erreurs répétées

Supprimer le fichier dhcpd.pid:  
``rm /var/run/dhcpd.pid``

Puis tenter de redémarrer le serveur:  
``systemctl restart isc-dhcp-server``

---

# Configuration de l'infrastructure

Ce fichier rassemble l'ensemble des procédures liées à l'infrastructure réseau, à savoir l'installation et la configuration de l'ensemble des switchs et des routeurs.

Les procédures d'installation sont tout d'abord présentées en résumé avant de rentrer plus en détail pour chaque commande.


## Résumé de la configuration des Switchs

Sur un switch vierge, en se connectant avec un câble Serial et l'utilitaire de son choix:

- Configurer le hostname
- Sécuriser l'accès privilégié
- Configurer le Telnet



## Résumé de la configuration des Routeurs

Sur un routeur vierge, en se connectant avec un câble Serial et l'utilitaire de son choix:

- Configurer le hostname
- Sécuriser l'accès
- Configurer chaque interface du routeur
- Configurer Telnet
- Configurer RIPv2 (routage dynamique)
- Configurer le protocole HSRP (failover)

---

## Commandes communes à tous les équipements

### Hostname

    hostname <nom_switch>

### Sécuriser l'accès privilégié

    enable secret <mot_de_passe>

### Configuration Telnet

Attribuer une IP sur le VLAN 1

```
line vty 0 4
password <mot_de_passe>
login
```


## Commandes et procédures des switchs en détail

## Commandes et procédures des routeurs en détail

### Configurer les interfaces

Contrairement à un switch, les interfaces d'un routeur sont toutes inactives par défaut. Il faut alors les sélectionner, leur attribuer une IP puis les activer manuellement:

```
(config)# interface <interface>
(config-if)# ip address <ip> <masque>
(config-if)# no shutdown
```

### Configurer RIPv2 (routage dynamique)

#### Étapes dans l'ordre

Accéder à la configuration RIP:

    (config)# router rip

Activer le protocole RIPv2:

    (config-router)# version 2

Déclarer chaque réseau en lien avec le routeur en répétant autant de fois que nécessaire:

    (config-router)# network <IP_réseau>

Désactiver la récapitulation automatique:

    (config-router)# no auto-summary

Enfin, quitter la configuration RIP:

    (config-router)# exit

> On peut vérifier si RIPv2 est bien actif et voir quels réseaux ont été déclarés avec 
> ```
> show ip protocols
> show ip route
> ``` 

    

#### Exemple en déclarant trois réseaux
```
(config)# router rip
(config-router)# version 2
(config-router)# network <IP_réseau_1>
(config-router)# network <IP_réseau_2>
(config-router)# network <IP_réseau_3>
(config-router)# no auto-summary
(config-router)# exit
```


### Configurer le protocole HSRP

Le protocole HSRP permet d'assurer une disponibilité des services en offrant une tolérance de pannes d'un des routeurs du réseau. L’objectif est de disposer d’une sûreté de fonctionnement d'une passerelle IP dont la disponibilité est primordiale. Il s'agit de fail-over ("roue de secours") et non de load-balancing (équilibrage de charge).

#### Étapes dans l'ordre

Attribuer une IP au routeur virtuel

#### Exemple

```
(config)# interface <interface>
(config-if)# standby 1 ip <ip>
(config-if)# standby 1 preempt
```

---

# Proxmox


## Connexion

L'adresse permettant de se connecter à l'interface web de Proxmox est la suivante: https://172.16.200.1:8006 

Le `https` est **nécessaire** pour se connecter !

---

# Fichiers de configuration

## Routeurs

## Switches

## Docker

### /etc/nginx/nginx.conf 

```Nginx config
    user  nginx;

    # Nombre de procesus (un thread / process)
    worker_processes  1;

    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;

    # Nombre de slots = worker_processes * worker_connections
    events 
    {
        worker_connections  1024;
    }


    http 
    {
        # Liste des types de fichiers média à inclure
        include       /etc/nginx/mime.types;
        
        default_type  application/octet-stream;
        
        # Format de logs
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';

        # Emplacment des logs
        access_log  /var/log/nginx/access.log  main;
        
        # Optimisation du traffic
        sendfile        on;
        #tcp_nopush     on;
        
        # Temps de maintien de la connection en secondes
        keepalive_timeout  65;
        
        # Emplacment du fichier de configuration annexe
        include /etc/nginx/conf.d/*.conf;
    }
```

### /etc/nginx/conf.d/default.conf
```Nginx config
    server 
    {
        # Ports d'ecoute IPv4 et IPv6 respectivement
        listen       80;
        listen  [::]:80;
        server_name  localhost;

        #access_log  /var/log/nginx/host.access.log  main;
        
        # Emplacement des ressources html et de l'index
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        
        # Pages d'erreur
        error_page   500 502 503 504  /50x.html;
        location = /50x.html 
        {
            root   /usr/share/nginx/html;
        }
    }
```

### /etc/vsftpd.conf
```properties
    # Racine du partage FTP
    local_root=/usr/share/nginx

    # Desactiver le mode passif
    pasv_enable=NO

    # Autoriser la connection depuis le port 20
    connect_from_port_20=YES

    # Autoriser l'ecriture en FTP
    write_enable=YES

    listen=NO
    listen_ipv6=YES

    # Interdire les connections
    anonymous_enable=NO

    # Autoriser les comptes locaux a se connecter
    local_enable=YES

    #local_umask=022

    dirmessage_enable=YES

    # Utiliser l'heure du serveur pour le système de fichiers
    use_localtime=YES

    # Logger les uploads/downloads.
    xferlog_enable=YES

    #chown_uploads=YES
    #chown_username=whoever

    # Emplacement et format des logs (par défaut)
    #xferlog_file=/var/log/vsftpd.log
    #xferlog_std_format=YES

    # Timeouts des connections
    #idle_session_timeout=600
    #data_connection_timeout=120

    secure_chroot_dir=/var/run/vsftpd/empty

    # PIM PAM POUM
    pam_service_name=vsftpd

    rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
    rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
    ssl_enable=NO

    #utf8_filesystem=YES
```