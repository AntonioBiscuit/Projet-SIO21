# Implémentation d’une infrastructure réseau

- [Implémentation d’une infrastructure réseau](#implémentation-dune-infrastructure-réseau)
  - [Schéma de l’infrastructure](#schéma-de-linfrastructure)
  - [Solutions choisies](#solutions-choisies)
  - [Configuration de l’infrastructure réseau](#configuration-de-linfrastructure-réseau)
    - [Configuration des switchs](#configuration-des-switchs)
    - [Choix du hostname](#choix-du-hostname)
    - [Mot de passe pour le mode privilégié](#mot-de-passe-pour-le-mode-privilégié)
    - [Ajout d’ une adresse ip pour l’administration du switch](#ajout-d-une-adresse-ip-pour-ladministration-du-switch)
    - [Ajout d’une connexion en telnet](#ajout-dune-connexion-en-telnet)
    - [Configuration des routeurs](#configuration-des-routeurs)
    - [Choix du hostname](#choix-du-hostname-1)
    - [Mot de passe pour le mode privilégié](#mot-de-passe-pour-le-mode-privilégié-1)
    - [Ajout d’une adresse ip pour chaque interface connectée](#ajout-dune-adresse-ip-pour-chaque-interface-connectée)
    - [Ajout d’une connexion en telnet](#ajout-dune-connexion-en-telnet-1)
    - [Mise en place du protocole RIP](#mise-en-place-du-protocole-rip)
    - [Mise en place du protocole HSRP](#mise-en-place-du-protocole-hsrp)

## Schéma de l’infrastructure

Nous avons choisi d'utiliser le nom de domaine `ggp.local`

## Solutions choisies

Certaines de nos solutions tournent dans un environnement virtuel alors que d'autres tournent bare-metal.

Nos solutions tournent sur un mix de machines virtuelles et de conteneurs mais aussi sur du bare-metal.

Nos machines virtuelles tournent sur une ferme Proxmox tandis que nos conteneurs tournent à l'aide de Docker. Il aurait été tout à fait possible d'utiliser une image Alpine Linux pour le conteneur `HTTP + FTP`

Nous avons choisi d'utiliser Debian la majorité du temps car considéré comme standard.

|Service|Environnement d'exécution|OS|
|---|---|---|
|Active Directory + DNS|Proxmox (VM)|Windows Server 2019|
|Mail|Proxmox (VM)|Debian|
|HTTP + FTP|Docker (Conteneur)|Image Debian
|Base de données|Bare-metal|Debian|
|Extranet|Bare-metal|Debian|
|Pare-feu|Bare-metal|PfSense (FreeBSD)|


## Configuration de l’infrastructure réseau

### Configuration des switchs

### Choix du hostname 

### Mot de passe pour le mode privilégié 

### Ajout d’ une adresse ip pour l’administration du switch

### Ajout d’une connexion en telnet 

### Configuration des routeurs

### Choix du hostname 

### Mot de passe pour le mode privilégié

### Ajout d’une adresse ip pour chaque interface connectée

### Ajout d’une connexion en telnet

### Mise en place du protocole RIP

### Mise en place du protocole HSRP
