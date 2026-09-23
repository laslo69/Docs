
Protocole permettant d’automatiser la configuration réseau d’un client et de ses paramètres réseau. Il permet de simplifier la configuration en automatisant le processus de configuration, surtout sur des grands réseaux

Déconseillé d’utiliser sur des hôtes hébergeant des services critiques, nécessitant une configuration statique ( serveur web, serveur SQL etc… )

## Message 

Tout les messages DHCP utilisent la même structure, lors d'une capture wireshark, il est possible qu'une ligne `magic cookie` existe, elle permet de faire la différence entre le protocole BOOTP et DHCP.

```bash
0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     op (1)    |   htype (1)   |   hlen (1)    |   hops (1)    |
+---------------+---------------+---------------+---------------+
|                            xid (4)                            |
+-------------------------------+-------------------------------+
|           secs (2)            |           flags (2)           |
+-------------------------------+-------------------------------+
|                          ciaddr  (4)                          |
+---------------------------------------------------------------+
|                          yiaddr  (4)                          |
+---------------------------------------------------------------+
|                          siaddr  (4)                          |
+---------------------------------------------------------------+
|                          giaddr  (4)                          |
+---------------------------------------------------------------+
|                                                               |
|                          chaddr  (16)                         |
|                                                               |
|                                                               |
+---------------------------------------------------------------+
|                                                               |
|                          sname   (64)                         |
+---------------------------------------------------------------+
|                                                               |
|                          file    (128)                        |
+---------------------------------------------------------------+
|                                                               |
|                          options (variable)                   |
+---------------------------------------------------------------+
```


### DHCPDISCOVER

Le client émet une trame DHCP DISCOVER sur son réseau local avec une adresse IP 0.0.0.0:68 et la mac adresse de son interface réseau 00:0b:82:01:fc:42 à destination du serveur DHCP 192.168.0.1  avec l’adresse IP 255.255.255.255:67, mac FFFF:FFFF:FFFF. Lors de l’émission de la trame, le paquet est suivi grace à un numéro de transaction xID( Transaction ID 0x3d1d, par exemple)

### DHCPOFFER

Lors de réception de la trame par le ou les serveurs, il répond sur l’adresse IP qui est proposé a l’hôte. Par exemple si l’adresse ip proposé est 192.168.0.10 ( champ  yiaddr qui permet de proposer une IP au client ) , il répondera avec les informations du serveur  IP 192.168.0.1:67 mac 00:08:74:ad:f1:9b et transaction ID 0x3d1d à destination de 192.168.0.10:68 00:06:82:01:FC:42. Des options peuvent être proposé en même temps.

### DHCPREQUEST

Lors du DHCP REQUEST, le client émet une trame DHCP REQUEST avec une adresse IP 0.0.0.0:68 mac 00:0b:82:01:fc:42 à destination de 255.255.255.255:67 mac FFFF:FFFF:FFFF . Le DHCP REQUEST possède un paramètre, l’option 50 ( Requested IP Address ) qui permet de définir l’adresse que le serveur DHCP à proposé dans le DHCP OFFER précédent, et que le client demande la réservation. Le numéro de transaction peut être le même ou il peut être différend, les deux cas sont autorisé par la RFC 2131. Les autres offres seront refusées.

Si un client possédais déja une adresse IP avec une durée de validité positive, le client émet un DHCPREQUEST à destination du serveur DHCP responsable de sa configuration, avec son IP en option50 et les options associés.

### DHCPACK

Lors de réception de la trame DHCP REQUEST, le serveur répond à partir de 192.168.0.1:67 mac 00:08:74:ad:f1:9b transaction ID à destination de, soit broadcast 255.255.255.255:68 ffff:ffff:ffff et permet de confirmer que l’adresse lui est bien attribué et réservé. La combinaison du champ " chaddr " et de l'adresse IP attribué permet de constitué un identifiant unique. Dans le ACK, le champ yiaddr est rempli par l'adresse IP attribué au client.

Une fois la configuration terminé, le client doit effectué un contrôle ARP pour renseigner sa présence et son lien MAC <-> IP pour confirmer son unicité.

Si le serveur reçoit un DHCPREQUEST d'un client avec une configuration déja existante, le serveur vérifie que l'adresse IP n'est pas utilisé par un autre hôte, sa validité et autorise le client à reprendre son IP et actualise le bail

### DHCPDECLINE

Dans le cas ou, l’adresse proposé par le serveur DHCP lors du DHCP OFFER est déja prise, le client émet un message DHCP DECLINE à destination du serveur DHCP 0.0.0.0:67 FF:FF:FF:FF:FF:FF pour le prévenir et mettre fin à l’échange, le processus DORA recommence à partir de DHCPOFFER en proposant une autre adresse IP. Le client test l’adresse IP proposé via ICMP ou ARP, et dans le cas d’une réponse, indique qu’elle est déja active et retournera le message DHCP DECLINE, l’adresse sera retiré temporairement du pool, le client devra attendre 10 secondes avant de pouvoir recommencer le processus de configuration.


### DHCPNAK

Lors du processus DORA, il peut arriver que, au lieu de recevoir un DHCP ACK, nous recevons un DHCPNAK qui peut arriver quand, le client demande une IP qui n’est plus disponible, l’IP est en dehors du pool DHCP,le délais d'attente pour le DHCPACK est trop long ou le client n’est pas reconnu par le serveur DHCP. Quand le client reçois le DHCPNAK, le processus DORA doit être repris du début

Le client émet un DHCP REQUEST 0.0.0.0:68 00:0b:82:01:fc:42 avec l’option 50 contenant l’adresse IP proposé  vers le serveur 255.255.255.255:67 FFFF:FFFF:FFFF

Le serveur répond avec un DHCPNAK à partir de son IP 192.168.0.1:67  00:08:74:ad:f1:9b xid à destination du client 255.255.255.255:68  00:0b:82:01:fc:42, ici le serveur répond avec une adresse 0.0.0.0 proposé dans l’option 53 car aucune adresse ne peut être fournis

Dans le cas ou, un message DHCPACK n'est pas reçue, une retransmission du DHCPREQUEST est effectué au maximum 4 fois sur une durée de 60 secondes en total, si aucune réponse une fois le délais terminé, la configuration reprend du début

### DHCPRELEASE

Quand le client DHCP libère manuellement l’adresse avant la fin du bail, le client émet un message DHCPRELEASE à destination du serveur DHCP pour l’avertir de la libération de l’adresse.

Le client émet le message DHCPRELEASE à partir de son adresse 192.168.0.10:68 00:0b:82:01:fc:42 xid à destination de 192.168.0.1:67 ( serveur ayant attribué l’ip ) 00:08:74:ad:f1:9b ( 255.255.255.255 FFFF:FFFF:FFFF si l’ip du DHCP est inconnu ) avec les champs " chaddr " et son adresse réseau. Le serveur met à jour son pool et marque l’adresse comme libre

### DHCPINFORMATION

Quand le client émet un DHCPINFORM, le client demande au serveur DHCP de mettre à jour ses paramètres de configuration comme le DNS,NTP,routeur etc… sans demander une adresse IP.

Le client émet une trame DHCPINFORM depuis 192.168.0.10:68 00:0b:82:01:fc:42 transaction ID à destination de 192.168.0.1:67 00:08:74:ad:f1:9b  ( ou 255.255.255.255 FFFF:FFFF:FFFF si l’ip du serveur DHCP n’est pas connu ). l’option 55 permet de lister les options à mettre à jour.

Le serveur DHCP répond avec un message DHCPACK qui inclut les options demandé mais ne fournis pas d’adresse IP

## Validité du bail

Lors de la configuration d'une adresse IP par un serveur DHCP, la validité est défini par 2 timers

- T1 : Correspond à 50% du temps de validité du lease
- T2 : Correspond à 87.5% du temps de validité du lease

## Etat du client

```bash
 --------                               -------
|        | +-------------------------->|       |<-------------------+
| INIT-  | |     +-------------------->| INIT  |                    |
| REBOOT |DHCPNAK/         +---------->|       |<---+               |
|        |Restart|         |            -------     |               |
 --------  |  DHCPNAK/     |               |                        |
    |      Discard offer   |      -/Send DHCPDISCOVER               |
-/Send DHCPREQUEST         |               |                        |
    |      |     |      DHCPACK            v        |               |
 -----------     |   (not accept.)/   -----------   |               |
|           |    |  Send DHCPDECLINE |           |                  |
| REBOOTING |    |         |         | SELECTING |<----+            |
|           |    |        /          |           |     |DHCPOFFER/  |
 -----------     |       /            -----------   |  |Collect     |
    |            |      /                  |   |       |  replies   |
DHCPACK/         |     /  +----------------+   +-------+            |
Record lease, set|    |   v   Select offer/                         |
timers T1, T2   ------------  send DHCPREQUEST      |               |
    |   +----->|            |             DHCPNAK, Lease expired/   |
    |   |      | REQUESTING |                  Halt network         |
    DHCPOFFER/ |            |                       |               |
    Discard     ------------                        |               |
    |   |        |        |                   -----------           |
    |   +--------+     DHCPACK/              |           |          |
    |              Record lease, set    -----| REBINDING |          |
    |                timers T1, T2     /     |           |          |
    |                     |        DHCPACK/   -----------           |
    |                     v     Record lease, set   ^               |
    +----------------> -------      /timers T1,T2   |               |
               +----->|       |<---+                |               |
               |      | BOUND |<---+                |               |
  DHCPOFFER, DHCPACK, |       |    |            T2 expires/   DHCPNAK/
   DHCPNAK/Discard     -------     |             Broadcast  Halt network
               |       | |         |            DHCPREQUEST         |
               +-------+ |        DHCPACK/          |               |
                    T1 expires/   Record lease, set |               |
                 Send DHCPREQUEST timers T1, T2     |               |
                 to leasing server |                |               |
                         |   ----------             |               |
                         |  |          |------------+               |
                         +->| RENEWING |                            |
                            |          |----------------------------+
                             ----------
```

### Initialisation et attribution d'une adresse réseau

Dans le cas d'une initialisation du client sans adresse réseau configuré, le client va suivre les étapes

- DISCOVER
- OFFER
- REQUEST
- ACKNOWLEDGE

### Initialisation avec une adresse réseau valide

Après un redémarrage du client qui possède, dans le pool DHCP, un bail actif encore valable

- REQUEST
- ACKNOWLEDGE

### Renew & Rebind

Dans le cas ou, un client utilise une adresse IP qui, arrive au stade d'expiration du timer T1 ou T2

- REQUEST
- ACKNOWLEDGE

### Release

Si le client n'a plus besoin de l'adresse réseau qui lui a été attribuée (par exemple, si le client est arrêté normalement), il envoie un message DHCPRELEASE au serveur

## Création pool DHCP

Pour créer un pool DHCP, en mode configuration global, par défaut, un pool DHCP sur du Cisco utilise l'ensemble des adresses IP de la plage, même l'adresse IP qui sera utilisé en gateway

```bash
ip dhcp pool LAN
network 192.168.0.0 255.255.255.0
default-routeur 192.168.0.1
domain-name lab.lan
dns-server 192.168.0.2
lease 7 12 0
exit
```

il peut être nécessaire, de choisir de limiter le nombre d'adresse IP attribuable, en excluant des adresses du pool DHCP

Il peut être intéressant, voir nécessaire, d’en exclure certaines du pool pour éviter qu'une adresse statique ou l'adresse de gateway soit réutilisé et entrer en conflit

Pour exclure une IP unique

```bash
ip dhcp excluded-address 192.168.0.1
```

Possibilité d'exclure en plage d'adresse IP

```bash
ip dhcp excluded-address 192.168.0.1 192.168.0.10
```

## Administration

### Maintenance

Vérifier les lease

```bash
show ip dhcp lease
```

Voir la configuration DHCP dans la running-config

```bash
show running-config | include dhcp
!
no ip dhcp use vrf connected
ip dhcp excluded-address 192.168.0.1
ip dhcp pool lan1
```

Voir les statistiques du serveur DHCP ( nombre de message DHCP / BOOTP)

```bash
show ip dhcp server statistics
Memory usage        24586
Address pools       1
Database agents     0
Automatic binding   3
Manual bindings     0
Expired bindings    0
Malformed messages  0
Secure arp entries  0
!
Message             Received
BOOTREQUEST         0
DHCPDISCOVER        6
DHCPREQUEST         3
DHCPDECLINE         0
DHCPRELEASE         0
DHCPINFORM          0

Message             Sent
BOOTREPLY           0
DHCPOFFER           6
DHCPACK             3
DHCPNAK             0
```

Voir les adresses ip attribué

```bash
show ip dhcp binding
Bindings from all pools not associated with VRF:
IP Address       Client-ID          Lease expiration         Type
192.168.0.2    0100.5079.6668.01   Mar 08 2002 12:25 PM    Automatic
192.168.0.3    0100.5079.6668.00   Mar 08 2002 12:25 PM    Automatic
192.168.0.4    0100.5079.6668.02   Mar 08 2002 12:25 PM    Automatic
```

Lister les pools DHCP, peut être appliquer sur un pool DHCP précis en ajoutant son nom

```bash
show ip dhcp pool
!
Pool Lan1:
Utilization mark (high/low) : 100/0
Subnet size (first/next)    : 0/0
Total addresses             : 254
Leased Adresses             : 3
Pending event               : none
1 subnet is currently in the pool :
Current index       IP address range            Leased adressess
192.168.0.5       192.168.0.1-192.168.0.254         3
```

Libérer une adresse IP

```bash
clear ip dhcp binding <ip>
```

### Sécurité

Exclure les adresses IP réservé et non utilisé

```bash
ip dhcp excluded-address 192.168.1.1 192.168.1.20
```

- Utilisation d'ACL pour bloquer le trafic non autorisé
- Utilisation de ip dhcp snooping
- Ajuster la taille lease selon le besoin
- Réserver des adresse pour les appareils critiques avec `ip dhcp excluded-address`
- Journalisation log et snmp
- Désactiver DHCP sur les interfaces non concernées `no ip dhcp server`
- Segmentation VLAN

### Dépannage

Voir les paquets, évenement DHCP

Attention, gourmand en CPU, penser à désactiver le debugging après utilisation

Afficher les paquets DHCP DORA

```bash
debug ip dhcp server packet
```

Affiche les évenements serveur: renouvellement, allocations, libérations

```bash
debug ip dhcp server events
```
