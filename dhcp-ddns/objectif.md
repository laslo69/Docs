1. bind9 — socle DNS
Zones directes et inverses, Clés TSIG + allow-update
2. Kea DHCP — socle DHCP
sous-réseaux, pools, baux, Réservations host
3. kea-ddns — intégration DHCP ↔ DNS
kea-ddns, NCR (Name Change Requests)c
4. HA bind9 — redondance DNS
Primary/secondary, notify, transferts de zone, also-notify, DDNS multi-serveurs
5. HA Kea DHCP — redondance DHCP
load-balancing, Réplication des baux et heartbeat entre pairs

# DHCP - DDNS

Dans le cadre de ce projet, j'ai mis en place une infrastructure DHCP et DNS dynamique (DDNS) sous Linux, afin d'automatiser l'attribution des adresses IP et la mise à jour des enregistrements DNS.

## Objectif

L'objectif est de simplifier l'administration du réseau en assurant la cohérence entre les adresses IP attribuées aux machines et leurs noms d'hôte, pour répondre à la problématique de l'adressage IP qui se retrouve facilité, en réduisant considérablement les tâches manuelles.

Aussi, la mise en place d'une infrastructure DHCP et DNS en High availability va permettre, de garantir une disponibilité accrue en cas de panne d'un des serveurs en supprimant les SPOF



Schema, Adressage IP

## Configuration

### DHCP

### DNS

## Test

### DHCP

### DNS
