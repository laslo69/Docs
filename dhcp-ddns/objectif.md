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

presentation projet

## Objectif

contexte, problematique

Sujet

Techno

Distro

Schema

Adressage IP

## Configuration

### DHCP

### DNS

## Test

### DHCP

### DNS
