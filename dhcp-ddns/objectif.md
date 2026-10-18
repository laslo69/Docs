# DHCP & DNS en haute disponibilité avec Kea et BIND

## Projet

### Contexte

Dans le cadre de ce projet, j'ai mis en place une infrastructure réseau sous Linux associant un service DHCP à un service DNS dynamique (DDNS).

L'objectif est d'automatiser l'attribution des adresses IP aux clients et de maintenir à jour les enregistrements DNS correspondants.

Un serveur DHCP en panne = plus aucun équipement ne peut rejoindre le réseau. Pas de réseau, pas de prod

### Problematique

Comment automatiser l'attribution des adresses IP et la mise à jour des enregistrements DNS afin de simplifier l'administration d'un réseau ?

### Objectif

- Automatiser l'attribution des adresses IP via DHCP.
- Configurer un service DNS pour la résolution des noms de machines.
- Mettre en place les mises à jour DNS dynamiques.
- Vérifier la cohérence entre les adresses IP attribuées et les enregistrements DNS.
- Valider le fonctionnement de l'infrastructure à l'aide de tests.
- Reduction des SPOF

## Architecture et Environnement technique

### Environnement utilisé

- Hyperviseur : Proxmox
- Système d'exploitation : OPNsense, Debian
- Service DHCP : kea-dhcp
- Service DNS : Bind9
- Réseau :
    - LAN local : 192.168.0.0/24
    - LAN 10 : 192.168.10.0/24
    - LAN 20 : 192.168.20.0/24

![topologie](./illustration/topologie.png)

## Installation et configuration

### OPNsense

Un des premier éléments à mettre en place, est le DHCP relay pour permettre à mon LAN clients de communiquer avec les serveurs DHCP sur un autre LAN à part

![dhcp relay](./illustration/dhcp-relay.png)

### DHCP

Kea-dhcp enregistre touts les leases dans un fichier créer lors de l'installation, qui se situe dans : `/var/lib/kea/kea-leases4.csv`

Le fichier de configuration de kea-dhcp contient des instructions pour mettre à jour le fichier automatiquement

kea-dhcp4.conf



kea-ddns.conf

load-balance

### DNS

namd.conf.local

namd.conf.options

creation zone

## Test

serie de test

### DHCP

dhcp primary seul pour attribution ip

dhcp en ha pour replication fichier lease

dhcp 1 off pour voir si dhcp 2 attribue une ip

### DNS

dns 1 seul ajout dhcp client dans dns

dns 1 + 2 pour check axfr/ixfr

dns2 pour voir si dhcp ajoute bien dans secondary

dns1+2 pour voir si dns2 forward l'update vers le master
