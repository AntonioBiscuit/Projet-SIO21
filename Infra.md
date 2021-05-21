# Configuration de l'infrastructure

- [Configuration de l'infrastructure](#configuration-de-linfrastructure)
  - [Commandes communes à tous les équipements](#commandes-communes-à-tous-les-équipements)
    - [Hostname](#hostname)
    - [Sécuriser l'accès privilégié](#sécuriser-laccès-privilégié)
    - [Configuration Telnet](#configuration-telnet)
  - [Switchs](#switchs)
    - [Étapes de la configuration dans l'ordre:](#étapes-de-la-configuration-dans-lordre)
  - [Routeurs](#routeurs)
    - [Étapes de la configuration dans l'ordre:](#étapes-de-la-configuration-dans-lordre-1)
    - [Configurer les interfaces](#configurer-les-interfaces)
    - [Configurer RIPv2 (routage dynamique)](#configurer-ripv2-routage-dynamique)
      - [Étapes dans l'ordre](#étapes-dans-lordre)
      - [Exemple en déclarant trois réseaux](#exemple-en-déclarant-trois-réseaux)
    - [Configurer le protocole HSRP](#configurer-le-protocole-hsrp)
      - [Étapes dans l'ordre](#étapes-dans-lordre-1)
      - [Exemple](#exemple)

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


## Switchs

### Étapes de la configuration dans l'ordre:

Sur un switch vierge, en se connectant avec un câble Serial et l'utilitaire de son choix:

- Configurer le hostname
- Sécuriser l'accès privilégié
- Configurer le Telnet

La configuration initiale sur un switch vierge doit se faire avec un cable Serial.
Toutes les commandes sont communes à tous les équipements, il faudra simplement ajuster en fonction de l'adressage et du réseau.



## Routeurs

### Étapes de la configuration dans l'ordre:

- Configurer le hostname
- Sécuriser l'accès
- Configurer chaque interface du routeur
- Configurer Telnet
- Configurer RIPv2 (routage dynamique)
- Configurer le protocole HSRP (failover)

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