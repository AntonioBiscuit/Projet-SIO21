# Implémentation d’une infrastructure réseau

- [Implémentation d’une infrastructure réseau](#implémentation-dune-infrastructure-réseau)
  - [Sommaire](#sommaire)
  - [Schéma, plan et adressage IP de l’infrastructure](#schéma-plan-et-adressage-ip-de-linfrastructure)
  - [Solutions choisies](#solutions-choisies)
  - [Serveurs](#serveurs)

## Sommaire

- [Identifiants](Passwords.md)
- [Fichiers de configuration](Confs.md)
- [Configuration de l’infrastructure réseau](Infra.md)
- [Mise en place du serveur HTTP / FTP](HTTP-FTP.md)

## Schéma, plan et adressage IP de l’infrastructure

Nous avons choisi d'utiliser le nom de domaine `ggp.local`

![](adressage.svg)

## Solutions choisies

Nos solutions tournent sur un mix de machines virtuelles, de conteneurs et de bare-metal.

Nos machines virtuelles tournent sur une ferme Proxmox tandis que nos conteneurs tournent à l'aide de Docker.

Nous avons choisi d'utiliser Debian la majorité du temps car considéré comme "standard". 

|Service|Serveur|Environnement d'exécution|OS / Image|
|---|---|---|---|
|Active Directory + DNS||Proxmox (VM)|Windows Server 2019|
|Mail||Proxmox (VM)|Debian|
|HTTP + FTP||Conteneur Docker (Serveur Debian)|Image Debian|
|Base de données||Bare-metal|Debian|
|Extranet||Bare-metal|Debian|
|Pare-feu||Bare-metal|PfSense (FreeBSD)|

## Serveurs

