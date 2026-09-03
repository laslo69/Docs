
EtherChannel est une technologie réseau qui permet de regrouper plusieurs liens physiques Ethernet entre deux appareils (comme des commutateurs ou des routeurs) pour former un seul lien logique

EtherChannel (ou Port Channel) est une méthode qui combine plusieurs ports physiques en un seul canal logique

Cela augmente la bande passante entre les appareils tout en offrant une redondance : si un lien tombe en panne, le trafic est automatiquement redirigé vers les autres liens actifs

## Protocole

Etherchannel utilise 2 protocoles de négociation pour créer le Lien:

- PagP ( Port Aggregation Protocol ) propriétaire Cisco, jusqu’à 8 interfaces
- LACP ( Link Aggregation Control Protocol ) Standard ouvert 802.3ad, jusqu’à 16 interfaces ( 8 actives + 8 backup )

Pour construire un lien , il faut définir les interfaces, le mode de négociation, le protocole de négociation et les liens en mode trunk, ils doivent être identique de chaque côté du lien avec la même configuration pour les interfaces.

Le mode de négociation doit utilisé le même protocole ( sinon ça ne marche pas ) ou du moins, il faut définir avec le même protocole, au minimum un côté en actif et un autre en passif propre au protocole qui doit obligatoirement être le même de chaque côté du lien etherchannel.

Il est possible de grouper jusqu’au maximum de 8 interfaces active + 8 passive en backup par groupe etherchannel avec le protocole LACP alors que PaGP n'en prend que 8 max

## Négociation

Pour pouvoir créer le lien etherchannel, chaque interface membre doit avoir un mode pour pouvoir négocier la formation du canal.

Chaque protocole à ses propres modes de négociation

|Protocole|Description|Mode|
|:-:|:-:|:-:|
|PagP|Propriétaire Cisco|Desirable ( Actif ), Initie la connexion|
|PAgP|Propriétaire Cisco|Auto ( Passif )|
|LACP|Standard IEEE 802.3ad|Active ( Actif ), Initie la connection|
|LACP|Standard IEEE 802.3ad|Passive ( Passif )|

Correspondance par type de négociation par protocole

|Protocole|Interface A|Interface B|Résultat|
|:-:|:-:|:-:|:-:|
|PaGP|Desirable(actif)|Desirable(actif)|Lien créer|
|PaGP|Desirable(actif)|Auto(passif)|Lien créer|
|PagP|Auto(passif)|Auto(passif)|Lien non créer|
|LACP|Active(actif)|Active(actif)|Lien créer|
|LACP|Active|Passive(passif)|Lien créer|
|LACP|Passive(passif)|Passive(passif)|Lien non créer|

Il existe un mode `on` qui forme un lien etherchannel, mais n'utilise aucun mode de négociation

## Load-Balancing

### Mode load-balancing

Il est possible de configurer, comment le load-balancing doit fonctionner en utilisant différente méthode d’équilibragr de charge basé sur :

- src-mac : Calcul du hash basé sur l’adresse MAC source
- dst-mac : Calcul du hash basé sur l’adresse MAC destination
- src-ip : Calcul du hash basé sur l’adresse IP source
- ip-dst : Calcul du hash basé sur l’adresse IP destination
- src-dst-mac : combine les adresses MAC sources et destination pour le hashage
- src-dst-ip : Combine les adresse IP sources et destination pour le hashage
- src-port : basé sur le port source
- dst-port : Basé sur le port de destination
- src-dst-port : Combine les ports sources et destination

### Décision load-balancing

Quand une trame est reçu par le commutateur, une fonction de hashage s’applique, le résultat passe par un modulo pour définir, selon le résultat, quel lien utilisé

Les exemples se basent sur 4 liens actifs en utilisant un hashage basé sur XOR simplifié, certains commutateur peuvent utiliser d’autres protocoles de hashage

#### src-mac / dst-mac

- Le commutateur extrait le champ cible, les 6 octets de l’adresse MAC et applique une fonction de hashage XOR bit à bit sur des sous ensembles d’octets, le résultat est modulo le nombre de lien actifs 

- Tout les paquest de la même MAC emprunteront le même lien, mais présente un risque de polarisation si une seule adresse MAC source ou destination est ciblé

- Utile si le Trafic provient ( pour src-mac ) de quelques machines comme des serveurs, ou dans le cas de dst-mac si le trafic est destiné à quelques machines

#### src-dst-mac

- L’adresse MAC source et destination sont concaténé pour atteindre un valeur de 12 octets, ensuite le hashage s’effectue sur ces 12 octets
- Tous les paquets entre la même paire de MAC emprunteront ce même lien
- Utile dans un réseau LAN classique, meilleur répartition que src-mac ou dst-mac

#### src-ip

- Le hashage va se produit sur l’adresse source ( 4 octets pour IPv4, 16 pour IPv6 ).
- Tous les paquets de la même source IP emprunteront le même lien, risque de polarisation si une seule IP source
- Utile lors de trafic entre sous-réseaux

#### dst-ip:

- Le champ d’adresse IP de destination sera utilisé pour l’algorithme de hashage.
- Tous les paquets vers une même IP de destination emprunteront le même lien
- Utilise pour du trafic vers des serveurs

#### src-dst-ip:

- Les champs adresse IP source et destination sont concaténé ( 8 octets pour IPv4, 32 octets pour IPv6 ), ensuite le hashage est appliqué sur le résultat de la concatenation
- Tous les paquets entre la même paire d’IP emprunteront le même lien
- Utile pour une meilleure répartition du trafic IP

#### src-port / dst-port:

- Le port source ou destination est utilisé ( 2 octets, 80 pour HTTP ), le hashage s’effectue sur les 2 octets.
- Tous les paquets avec le même port source ou destination selon le choix, emprunteront le même lien.
- Utile lors de trafic avec plusieurs connexions ou différents protocole utilisés, mais peu efficace si peu de ports sont utilisés

#### src-dst-port:

- Les ports source et destination sont concaténé puis hasher ( 4 octets )
  Tous les paquets entre la même paire de ports emprunteront le même lien-local
- Idéal pour les flux TCP/UDP multiples, dans les environnements haute performance et datacenters

## Etats Etherchannel

Les ports Etherchannel passent par différent status durant l’envoi d’information, la collecte, contrôle et synchronisation des informations pour créer le lien port-channel

- Individual : Port non agrégé (état initial)
- Negotiating : Échange de PDU PAgP/LACP en cours
- Bundle : Port ajouté au Port-Channel (canal formé)
- Hot Standby : Port de secours (si le nombre maximal de liens est atteint)
- Suspended : Port inactif (incompatibilité de configuration)

## Etherchannel L2

Etherchannel permet de laisser passer du traffic de couche 2, cependant il n'est pas capable nativement, de transmettre des trames 802.1q avec tag.

Dans le cas d’utilisation de VLAN avec un port-channel, il faut dans un premier temps mettre les interfaces membres du port-channel en trunk et autoriser les VLANs voulu à transmettre dessus

Ensuite, mettre le port channel en mode trunk et autoriser les VLANs voulu à transmettre dessus

## Etherchannel L3

Un port-channel peut être configuré en tant que lien couche 3, permet de faire du routage IP entre réseaux, contrairement à un port-channel L2 qui transporte du trafic de couche 2 et des trames de VLAN.

Un port-channel configué en L3 va servir principalement dans un routage IP sur un port agrégé, pour permettre d’augmenter la rapidité de flux, la redondance.

Pour configurer un port-channel en L3, il faut mettre les interfaces membres du port-channel en tant que port routable mais ne peut pas gérer de VLAN

## Message Etherchannel

### Message LACP

Pour les messages LACP, l’adresse MAC multicast utilisé est 0180.c200.0002, tous les message LACP sont envoyé en multicast. Un port en mode passive ne lancera pas de négociation mais répondera aux LACPDU reçu

Contrairement à PagP, LACP envoie toutes les informations dans des message LACPDU à intervalle de 1 seconde ( fast, par défaut ) ou 30 secondes ( slow )

### Processus négociation LACP

Le processus de négociation LACP procéde comme:

Chaque commutateur envoie des LACPDU en multicast  0180.c200.0002 contenant ses informations locales ( Actor ) et les informations apprise du partenaire ( Partner ), les LACPDU sont échangés toutes les secondes par défaut

Ensuite, les deux commutateurs comparent les informations reçues, si les paramètres sont compatibles, les ports passent à l’état synchroniser

Une fois synchronisé, les ports sont ajoutés au port-channel pour former le port-channel, les LACPDU continent d’être échanger pour maintenir l’état du canal.

Lors de l’ajout d’un lien au port-channel, un LACP Marker DU est envoyé pour annoncer l’ajout de l’interface ou la modification de sa configuration, au port-channel et synchroniser les trames. Le switch voisins doit répondre avec un LACP Marker Response pour confirmer la synchronisation

Si un changement survient, les LACPDU vont permettre de mettre à jour les tables MAC et STP

Les Marker PDU et Marker Response PDU on une structure similaire.

- Marker subtype : indique si il s’agit d’un Marker PDU ou d’un Marker Response
- Requester’s Port number : Numéro du port de l’expéditeur
- Marker Transaction ID : identifiant de la transaction
- Marker Response time : Temps de réponse attendu pour le Marker Response

### Message PaGP

Tous les messages PagP sont envoyés sur l’adresse MAC 0100.0ccc.cccc à intervalle de 30 secondes une fois que les synchronisation sont faites

Il y’a 4 type de message PagP:

- PagP Offer
- PagP Response
- Pagp Success
- PagP flush

### Processus négociation PaGP


Pagp va émettre un message PagP Offer à partir de chaque interface membres du groupe etherchannel à son voisin, en lui fournissant certaines informations à partir l’adresse MAC de multicast 0100.0ccc.cccc

Le voisin va répondre en émettant une réponse PagP Response sur la MAC 0100.0ccc.cccc en remplissant les champs demandé par le voisin pour vérifier si, les réglages des interfaces sont identiques ou non et pouvoir former une adjacence ou non.

Une fois la configuration contrôler, l’émetteur du PagP Offer va émettre un message PagP Success pour confirmer la création du port-channel et intégre les liens dans le port-channel.

Un message PagP Success contient des informations contenant les ID des Device local et partner, les port, group information etc…. Un state flags est ajouté pour indiqué que le lien est prêt à être ajouté au port-channel

Lorsque un lien est ajouté, retiré ou modifié, un message PagP Flush est émis pour mettre à jour les tables MAC et STP, un PagP Flush contient les local et partner Device ID, Port etc… et un flush request indiquant que les tables doivent être mises à jour

## Configuration Cisco

### Création Etherchannel

Pour créer un lien etherchannel, il faut d'abord sélectionner les interfaces voulu

```bash
interface range fastethernet 0/1-4
```

### Protocole Etherchannel

Pour appliquer un protocole

```bash
channel-protocole lacp | pagp
```

### Formation groupe

Pour créer le lien

```bash
channel-groupe 1 mode [active|desirable|passive|auto]
```

### Etherchannel L2

Pour former un lien etherchannel L2 avec des VLANs, à faire sur les 2 switchs

- Création des VLANs
- Appliquer le mode trunk sur des interfaces
- Autoriser les VLANs
- Définir VLAN natif
- se placer sur l'interface po1 ( etherchannel groupe 1)
- Appliquer mode trunk
- Autoriser les VLANs comme sur les interfaces
- Définir le VLAN natif

```bash
vlan 10
name admin
vlan 20
name server
interface range fastethernet 0/1-4
switchport mode trunk
switchport trunk allowed vlan 10,20
switchport trunk native vlan 20
interface po1
switchport mode trunk
switchport trunk allowed vlan 10,20
switchport trunk native vlan 20
```

### Etherchannel L3

Former un lien etherchannel en L3

- Sélection des interfaces
- enlever l'adressage ip si existant
- Enlever la capacité de switching ( port commuté -> port routable )
- Sélection protocole
- Création lien etherchannel
- Ajout adresse IP sur lien etherchannel

```bash
interface range fastethernet 0/1-4
no ip address
no switchport
channel-protocol lacp
channel-group 1 mode active
port-channel 1
ip address 192.168.20.1 255.255.255.0
```

## Administration

### Sécurité

• Desactiver DTP
• Storm control pour limiter multicast
• L2, autoriser uniquement les VLAN nécessaire
### Dépannage

• show etherchannel summary : Affiche le résumé de tous les liens etherchannels et leurs états
• show etherchannel X detail : Détails sur un etherchannel spécifique ( protocole,ports,statut)
• show etherchannel port-channel : liste des interfaces logiques et leurs membres
• show etherchannel load-balance : Affiche l’algorithme de load-balancing

diagnostic ports :

• show interfaces GigabitEthernet 0/0 etherchannel : voir si le port est membre d’un etherchannel
• show interfaces GigabitEthernet 0/0 switchport : Voir les paramètres de switchport du lien

Vérification protocole

• show lacp/pagp neighbor : Voir les voisins LACP/PagP et leur statut
• show lacp/pagp [x] internal : Détails internes sur la négociation PagP
• show lacp counters : statistiques des paquets LACP

dépannage des erreurs

• show interfaces [ x ] errors : affiche les erreurs CRC, runts, glants d’une interface logique ou physique
• show logging : affiche les logs systeme
• debug lacp/pagp : active le débogage lacp/pagp
• clear counters : reinitialise le compteurs d’erreurs sur les interfaces
