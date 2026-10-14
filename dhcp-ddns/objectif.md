1. bind9 — socle DNS
Zones directes et inverses
Clés TSIG (dnssec-keygen) + allow-update / update-policy

2. Kea DHCP — socle DHCP
DHCPv4 de base : sous-réseaux, pools, baux -> OK
Réservations host-reservation-identifiers

3. kea-ddns — intégration DHCP ↔ DNS
kea-ddns (serveur dédié) vs mode direct dans kea-dhcp
ddns-confgen, NCR (Name Change Requests)c

Test bout en bout : bail → enregistrement A/PTR

4. HA bind9 — redondance DNS
Primary/secondary : notify, transferts de zone, also-notify
DDNS multi-serveurs : kea-ddns vers plusieurs cibles
Test de bascule (arrêt du primary)

5. HA Kea DHCP — redondance DHCP
Hook ha : hot-standby vs load-balancing
Réplication des baux et heartbeat entre pairs
Contrôle via lease_cmds + monitoring

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
