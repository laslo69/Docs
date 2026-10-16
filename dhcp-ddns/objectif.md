# DHCP & DNS en haute disponibilité avec Kea et BIND

Un serveur DHCP en panne = plus aucun équipement ne peut rejoindre le réseau. Pas de réseau, pas de prod

J'ai donc monté une architecture DHCP/DNS sans point de défaillance unique : Kea en haute disponibilité + BIND en fail-over, avec mise à jour DNS dynamique et sécurisée

Ce lab a pour vocation de présenter une configuration d'un DHCP et DNS en haute disponibilité, d'expliquer pourquoi avoir choisi ces services, les problèmes résolus, avec des test de pannes

## Objectif

L'objectif premier, est de simplifier l'administration du réseau en automatisant l'adressage IP pour les clients de chaque LAN, lui fournir des options comme les DNS à utiliser et la gateway, mais aussi, le besoin de mettre en place une résolution DNS, qui est mis à jour automatiquement par les serveurs DHCP, va permettre d'automatiser le processus de création de la résolution de nom, que ce soit, résoudre le nom en IP, comme l'IP en nom

En automatisant ces processus, il est possible de réduire voire supprimer les configurations manuelle, réduire le risque de conflit d'adresse

Aussi, la mise en place d'une infrastructure DHCP et DNS en High availability va permettre, de garantir une disponibilité accrue en cas de panne d'un des serveurs en supprimant les SPOF, par exemple si un des serveurs vient à être hors-service, l'autre serveur est en mesure de pouvoir assurer la continuité de service

Plusieurs SPOF, peuvent être pris en compte et, voire comment une solution peut être apporté, pour réduire dans la mesure du possible, le nombre de SPOF présent

## Configuration

Pour ce lab, je me suis orienté sur `kea-dhcp` et `bind`, touts les deux gratuit et open-source

- `kea-dhcp` est le successeur direct et moderne de `isc-dhcp`, intégre nativement la possibilité d'enregistrer des adresses IP distribué dans une zone DNS, la prise en charge de la haute disponibilité, que ce soit load-balancing comme fail-over
- `bind` reste le DNS de référence, à la capacité de pouvoir répliquer les zones avec d'autres serveurs DNS, autorise les mises à jour dynamique qui peuvent être sécurisé par l'utilisation d'une clé TSIG

La mise en place de l'infrastructure se fera avec des VM, sur un serveur avec un hyperviseur proxmox et l'utilisation d'une VM opnsense qui fera office de routeur pour joindre mes différents LAN, toutes les VM, sont des Linux Debian 13

### DHCP

fichier lease

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

## Annexe

configuration complete dns + dhcp