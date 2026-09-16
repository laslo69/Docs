
LLDP (Link Layer Discovery Protocol) est un protocole libre et normé dans la publication IEEE 802.1ab.

C’est un protocole destiné à remplacer un bon nombre de protocoles propriétaires (Cisco CDP, Extreme EDP, etc...) utilisés dans la découverte des topologies réseau de proche en proche, il sert aussi à apporter des mécanismes d’échanges d’informations entre équipements réseaux et utilisateurs finaux

LLDP utilise l'adresse MAC `01:80:c2:00:00:0e`

## Interface

Par défaut, LLDP n'est pas activé sur les équipements Cisco, le protocole CDP étant déja actif, ce serait faire doublon pour le même travail

Toutefois, LLDP est un protocole qui permet une communication entre équipements de différents fabricants, l'activer peut être utilie dans un environnement multi-constructeur

Pour activer globalement CDP, active aussi pour chaque interface la réception et la transmission

```bash
SW1(config)# lldp run
```

## Transmit / Receive

Il est possible, dans le cas ou certaines interfaces, ne doivent pas recevoir et émettre de traffic LLDP, uniquement recevoir ou uniquement de émettre

La configuration peut se faire pour chaque interface indépendament mais doit impérativement, avoir LLDP actif

- Receive : Capacité à recevoir une trame LLDP
- Transmit : Capacité à envoyer une trame LLDP

En utilisant la commande " no " devant lldp transmit, la fonctionnalité d'envoie de trame LLDP est désactivé mais peut continuer de recevoir

Sur SW1

Désactiver la capacité de recevoir des trames LLDP

```bash
lldp run
int fa0/1
no lldp receive
```

SW2

Désactive la capacité à envoyer des trames LLDP

```bash
lldp run
int fa0/1
no lldp transmit
```

## Timers

Des timers permettent de, mettre à jour réguliérement ses voisins et, dans le cas ou le voisin ne répond plus pendant un laps de temps, son entré dans la table CDP est supprimé

Les timers:

- Message LLDP : Toutes les 30 secondes ( par défaut ), permet de détecter un voisin ou de maintenir son entré dans la table CDP
- Holdtime : durée de 120 secondes ( par défaut ), si aucune réponse durant ces 120 secondes, le voisin est considéré comme down

### Advertisement

LLDP utilise un message advertisements à intervalle de 30 secondes par défaut pour, transmettre ses informations aux voisins

Il est possible de modifier cette valeur

```bash
lldp timer < 5 - 65534 >
```

### Holdtime

Le holdtime, par défaut de 120 secondes, permet au processus LLDP de garder en mémoire les informations de son voisin dans le cas ou, ce dernier n'a pas retransmit dans le délai du LLDP advertisement ses informations pour le mettre à jour

Si le Holdtime arrive à 0, l'entrée du voisin est supprimé de la table LLDP

Pour modifier sa valeur

```bash
lldp holdtime < 0 - 65535 >
```

## Message LLDP

Lorsque, une trame est émise par le protocole LLDP, la trame est signalé avec l’adresse MAC source à destination de l’adresse MAC 0180:c200:000e ( multicast réservé pour LLDP ) encapsulé dans une trame EthernetII

Les messages LLDP sont appelé LLDPU

Un LLDPU est constitué de deux parties:

- Un entête et une fin de message fixe
- Un ensemble de TLV ( Type - Length - Value )

 les TLV sont une format qui permet de transmettre des informations avec une structure ordonée comme par exemple :

- Type : Nature de l'information ( nom, description port, etc...)
- Length : Taille du bloc de données
- Value : Valeur de l'information ( SW1, FastEthernet0/1, etc....)

LLDP va interpréter les TLV, il pourra:

- Le lire dans son intégralité et interprète tous les TLV qu’il peut interpéter
- Si un TLV ne peut pas être interpréter, il conserve le message tel quel et ne le prend pas en compte localement

Il retransmet ensuite le message originel en modifiant les TLV interprétés si il y’a besoin de les modifiers.

## Choix TLV

Il est possible, de pouvoir bloquer l'envoie de certaines informations dans les TLV

Avec la commande `no lldp tlv-select`

Par exemple, quand l'option `system-name` est bloqué, le nom de l'équipement n'est pas transmis aux voisins via les TLV, d'autres options sont disponibles ( voir documentation constructeur )

```bash
no lldp tlv-select system-name
```

## Administration

### Maintenance

Afficher la configuration LLDP

```rust
show lldp
```

Afficher les voisins LLDP, en ajoutant le paramètre `detail`, plus d'informations sont disponibles, similaire à `show lldp entry *`

```rust
Switch#sh lldp neighbors 
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
Switch              Fa0/1          120        B               Fa0/1
```

Voir le nombre de trames comptabilisé

```rust
show lldp traffic
!
LLDP traffic statistics:
    Total frames out: 4866
    Total entries aged: 0
    Total frames in: 310
    Total frames received in error: 0
    Totale frames discarded: 0
    Total TLVs discarded: 0
    Total TLVs unrecognized: 0
```

Afficher l'état LLDP sur une interface

```rust
show lldp interface fa0/0
!
FastEthernet0/0:
    Tx: enabled
    Rx: enabled
    Tx state: IDLE
    Rx state: WAIT FOR FRAME
```

### Sécurité

LLDP souffre du même probléme de sécurité que CDP, il n'y a pas d'authentification mis en place, ce qui laisse les informations en clair dans les paquets.

Désactiver LLDP sur les interfaces non utilisés ou concerné

```rust
switch(config-if)# no lldp transmit
switch(config-if)# no lldp receive
```

Désactiver les TLV sensibles

```rust
switch(config)# no lldp tlv <tlv_name>
```
