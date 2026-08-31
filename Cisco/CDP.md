
CDP est un protocole de couche 2, de découverte de voisin propriétaire cisco et activé par défaut, qui permet de renseigner sur l'appliance voisine de ses paramètres, activé par défaut

CDP peut être utile pour s'aider à se repérer dans le réseau mais reste une faille de sécurité car, utilisable pour faire du mapping réseau

Concrètement, il permet de voir :

- le nom de l’équipement voisin 
- son adresse IP 
- son type (switch, routeur, téléphone…) 
- l’interface utilisée 
- le modèle et version IOS 
- parfois des infos VLAN ou alimentation (PoE)

Sur le routeur, la sous-interface avec l’encapsulation VLAN/dot1q la plus faible est sélectionnée comme sous-interface préférée pour transporter les paquets CDP. 

Sur le commutateur, le trafic CDP est toujours préféré sur le VLAN le plus bas configuré. En d'autres termes, VLAN 1 toujours, qui ne peut pas être supprimé de la base de données VLAN.

CDP peut aussi notifier un VLAN mismatch

## CDP Version & Interface

### version

CDP advertise-v2 permet de, forcer le processus CDP à utiliser la version 2 du protocole.

Par défaut, cdp utilise déja la version 2 du protocole, pour forcer la version

```bash
(config)# cdp advertise-v2
```

### Interface

Par défaut, toutes les interfaces émettent des trames CDP.

Pour désactiver CDP sur une interface, le client n'émettera plus de trame CDP et n'en recevera plus via l'interface

```bash
int fa0/0
no cdp enable
```

## Sécurité

CDP est un protocole assez bavard qui donne des informations sur les appliances voisinses et la sienne, le problème est principalement que aucune sécurité n'est implanté avec CDP ce qui induit, que toutes les données transmisent, le sont en clair.

Protocole à désactiver dans le cas ou son utilisation n'est pas nécessaire, permet aussi d'économiser un peu de CPU et bande passante

## Timer

CDP est activé par défaut sur toutes les interfaces de l'appliance

Des timers permettent de, mettre à jour réguliérement ses voisins et, dans le cas ou le voisin ne répond plus pendant un laps de temps, son entré dans la table CDP est supprimé

Les timers:

- Message `CDP advertisement` : Toutes les 60 secondes ( par défaut ), permet de détecter un voisin ou de maintenir son entré dans la table CDP
- Holdtime : durée de 180 secondes ( par défaut ), si aucune réponse durant ces 180 secondes, le voisin est considéré comme down

### Advertise

Par défaut, le CDP Timer est de 60 secondes, pour le modifier:

```rust
(config)# cdp timer < 5 – 254 >
```

### Holdtime

Pour modifier le Holdtime:

```rust
(config)# cdp holdtime <10 – 255 >
```

## Message

Le contenu de la trame est identique pour chaque type de message CDP, seulement son contenu change.

CDP est encapsulé dans une trame 802.1q

Les messages CDP Request et CDP response sont 2 messages peu courant, du fait que chaque voisin annonce sa propre configuration

Les informations sont transmit dans un en tête `CDP TLV` qui correspond à un type : Type-Length-Valeur

C'est le message envoyé périodiquement par un équipement Cisco pour annoncer sa présence

### CDP Advertisement

Quand CDP émet une trame, il émet une trame Ethernet avec adresse MAC source, l’adresse MAC de l’interface utilisé pour émettre la trame à destination de l’adresse MAC de multicast 01:00:0c:cc:cc

Dans le message, des informations sont transmises pour annoncer la configuration et l’identité de l’expéditeur

Il contient des informations sur l'équipement émetteur, par exemple :

- Device ID : nom de l'équipement (hostname)
- Platform : modèle du matériel (ex. Catalyst 9300, ISR 4000)
- Capabilities : fonctions disponibles :
- - routeur
- - switch
- - téléphone IP
- - bridge…
- Port ID : interface utilisée pour la connexion
- IP Address : adresse IP de gestion
- Software Version : version IOS/IOS-XE
- Hold Time : durée pendant laquelle l'information reste valide

```bash
Device ID: SW-CORE
Entry address(es):
  IP address: 192.168.1.10
Platform: Cisco WS-C9300
Capabilities: Switch
Interface: GigabitEthernet1/0/24
Port ID: GigabitEthernet0/1
Holdtime: 153 sec
```

### CDP Request

Le CDP Request est un message de demande d'informations

Il permet à un équipement de demander à un voisin des informations spécifiques

Contrairement à l'Advertisement, il n'est généralement pas utilisé dans le fonctionnement normal de CDP

Il peut être utilisé dans certains scénarios internes du protocole pour demander des informations supplémentaires

### CDP Response

Le CDP Response est la réponse envoyée après une demande CDP Request

Il fournit les informations demandées par le voisin

## Header CDP

### Header

```bash
+--------------------------------------+
| Version(8b) | TTL (8b) | Checksum(8b)|
+--------------------------------------+
```

- VER : Version du protocole
- TTL : Durée de vie du message
- Checksum : contrôle de l’intégrité du message

### Header TLV

```bash
+--------------------------------------+
	| Version(16b) |     TTL (16b)     |
+--------------------------------------+
			  Value(32b)               |
+--------------------------------------+
```

- Type : type du TLV
- Length : longueur selon type TLV
- Value : valeur selon type TLV

## Configuration

Sur une topologie avec 3 switch en ligne

```bash
SW1 -------------- SW2 -------------- SW3
   f0/1        f0/1   f0/2        f0/1
```

### SW1

```rust
sw1(config)# int range fa0/2-24
sw1(config-if)# no cdp enable
sw1(config-if)# exit
sw1(config)# cdp timer 30
sw1(config)# cdp holdtime 90
sw1(config)# cdp advertise-v2
```

## SW2

```rust
sw1(config)# int range fa3/2-24
sw1(config-if)# no cdp enable
sw1(config-if)# exit
sw1(config)# cdp timer 30
sw1(config)# cdp holdtime 90
sw1(config)# cdp advertise-v2
```

### SW3

```rust
sw1(config)# int range fa0/2-24
sw1(config-if)# no cdp enable
sw1(config-if)# exit
sw1(config)# cdp timer 30
sw1(config)# cdp holdtime 90
sw1(config)# cdp advertise-v2
```

## Administration

### Maintenance

Afficher la configuration CDP

```bash
Switch#sh cdp 
Global CDP information:
    Sending CDP packets every 60 seconds
    Sending a holdtime value of 180 seconds
    Sending CDPv2 advertisements is enabled
```

Afficher les voisins CDP, il est possible de choisir une interface précise

```bash
Switch#sh cdp neighbors 
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
Switch       Fas 0/1          136            S       2960        Fas 0/1
```

Voir les voisins avec plus de détail ( utilisation des TLV), même sortie que `show cdp entry *`

```bash
Switch#sh cdp neighbors detail

Device ID: Switch
Entry address(es): 
Platform: cisco 2960, Capabilities: Switch
Interface: FastEthernet0/1, Port ID (outgoing port): FastEthernet0/1
Holdtime: 134

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen

advertisement version: 2
Duplex: full
```

### Sécurité

CDP n'est pas un protocole sécurisé, aucun mot de passe ou encryption n'existe, recommandé de désactiver si son utilisation n'est pas nécessaire

```bash
router(config)# no cdp enable
```

Désactiver sur les interfaces qui n'a pas besoin de transmettre de trame CDP ( router edge, lien vers un client final, ...)

```bash
router(config)# int fa0/0
router(config-if)# no cdp enable
```

### Dépannage

Contrôler la configuratio

```bash
show cdp
```

Effacer la table CDP pour la remettre à 0

```bash
clear cdp table
```

Remettre à 0 les compteurs CDP

```bash
clear cdp counters
```