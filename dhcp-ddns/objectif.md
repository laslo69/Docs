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

Ouvre sur le problème, pas sur l'outil. "Comment garantir la continuité du service DHCP/DNS" accroche plus que "J'ai installé Kea"

Justifie les choix techniques : pourquoi le hook HA de Kea plutôt que keepalived/VRRP, hot-standby vs load-balancing, pourquoi du DDNS côté Kea plutôt que des scripts nsupdate

Montre des extraits de config, pas des fichiers entiers : 5-10 lignes ciblées par point clé suffisent (les blocs lease-management, le allow-update avec TSIG, etc.)

La section tests est ce qui crédibilise : scénarios de panne concrets + résultats mesurés (temps de bascule, pertes ou non de baux)

Pièges rencontrés : c'est souvent la partie la plus partagée/commentée (sync des leases, réconciliation DDNS après failover, TTL, etc.)

Format LinkedIn : posts longs sont tronqués — publie un post court avec le rapport en PDF/carrousel, ou découpe en articles LinkedIn. Garde des phrases courtes et des sous-titres.

# DHCP - DDNS

presentation projet

## Objectif

sujet, contexte, problematique

Techno, Distro

Schema, Adressage IP

## Configuration

### DHCP

### DNS

## Test

### DHCP

### DNS
