
HSRP (Hot Standby Routing Protocol ) est un protocole propriétaire de Cisco conçu pour assurer la redondance de la passerelle par défaut dans un réseau local

Il permet à plusieurs routeurs de partager une adresse IP virtuelle en créant un groupe de routeur. L’un d’eux est désigné comme actif, les autres sont en veille

Lorsqu’un hôte envoie des paquets, il transit toujours par l’adresse IP virtuelle. Si le routeur actif devient indisponible, l’un des routeurs en veille prend automatiquement le relais. Ce basculement est transparent pour les hôtes

HSRP fonctionne en attribuant une priorité à chaque routeur du groupe. Il est possible d’activer la préemption pour permettre à un routeur plus prioritaire de reprendre son rôle dès qu’il est de nouveau en ligne. Le protocole envoie des messages Hello toutes les 3 secondes et considère un routeur inactif après 10 secondes sans nouvelles

C’est une solution simple, fiable, mais limitée aux équipements Cisco et ne propose pas de répartition de charge

HSRP ipv4 ne peut pas être utilisé en même temps qu'un processus HSRP ipv6

HSRPv2 n'est pas interopérable avec HSRPv1

## Version

HSRP possède 2 versions avec ses propriétés:

HSRPv1:

- IPv4
- Adresse MAC utilisée : 0000.0C07.Acxx, ou xx est l’id du groupe en héxadécimal
- Utilise l’adresse multicast 224.0.0.2, pour envoyer des message hello de taille fixe
- Groupe 0 au groupe 255
- N’identifie pas explicitement les VLAN,peut poser problème en cas de mauvaises configuration
- Support authentification

HSRPv2
 
- IPv4 / IPv6
- Adresse MAC utilisée : 0000.0C9F.Fxxx, ou xx est l’id du groupe en héxadécimal
- Utilise l’adresse multicast 224.0.0.102, pour envoyer des message hello de taille variable, peut inclure des champs optionnels
- Groupe 0 au groupe 4095
- Introduit un champ pour le numéro de VLAN dans le paquet, ce qui permet d’éviter les ambiguïtés dans les configurations où plusieurs sous-interfaces d’un même routeur physique peuvent être des routeurs virtuels actifs pour différents groupes HSRP.
- Support authentification, supporte aussi MD5

Pour choisir la version

```bash
interface fa0/0
standby version 2
```

## Etat & rôle HSRP

### Rôle HSRP

Dans une configuration HSRP, chaque routeur remplie un rôle précis.

Le protocole HSRP est basé sur le modèle Actif / Passif. Cela veut dire qu’il n’y aura qu’un seul routeur en mode “Actif” et les autres routeurs en mode “Passif” :

- Mode Active (Actif) : Ce routeur portera l’adresse IP et l’adresse MAC virtuelle, envoie des Hello périodique
- Mode Standby (Passif) : Les autres routeurs attendent que le routeur en mode active soit indisponible pour prendre sa place, le 2ème routeur avec la priorité la plus élevé deviens le standby. envoie des Hello périodique, prêt à prendre la relève
- Mode Listen : Une fois le rôle active et standby attribuer, les autres routeurs restent en état listen en attente d’un changement de topologie, routeur passif, n’envoie pas de hello mais les écoute

### Etat HSRP

Chaque routeur d'un groupe HSRP passe par plusieurs états

- Initial : Etat basique du routeur au démarrage du processus HSRP, aucun message hello reçu ou envoyer
- Learn : Découverte du groupe HSRP, reçoit des Hello
- Learn : Découverte du groupe HSRP, reçoit des Hello - Speak : Le routeur participe à l’élection en envoyant des Hello
- Active : Le routeur ayant le rôle de router " active " sera dans l'état Active

Il ne peut y avoir qu’un seul routeur en mode active en simultané, pour définir quel routeur est “ active “, il va falloir passer par une élection

## Election routeur

### Processus

Pour procéder à l’élection, chaque routeur possède des propriétés

- Chaque routeur possède une priorité.
- Cette priorité est modifiable par l’administrateur.
- La priorité par défaut est à 100. Elle est comprise entre 0 et 255.
- Le routeur possédant la plus haute priorité devient le routeur “ Active “, en cas d’égalité, celui avec l’adresse IP la plus haute l’emporte. 0 annule l'élection, 255 le met d'office prioritaire

Au démarrage, tous les routeur sont en état “ Init “ et passent en “ Learn “ pour écouter les messages “ Hello “

Élection du routeur actif : Les paquets Hello sont utilisés lors du processus d’élection du routeur actif et de secours. Les routeurs comparent les informations reçues via les paquets Hello pour décider quel routeur doit être actif ou de secours

### Message Hello

Dans le protocole HSRP (Hot Standby Router Protocol), les paquets “Hello” jouent un rôle crucial dans la communication entre les routeurs membres d’un groupe HSRP. Voici les points essentiels à connaître à propos des paquets Hello dans HSRP :

- Les paquets Hello sont utilisés pour annoncer la présence d’un routeur au sein d’un groupe HSRP et pour établir et maintenir la communication entre les routeurs actifs et de secours
- Intervalle et temporisation : Les paquets Hello sont envoyés périodiquement. L’intervalle par défaut entre les paquets Hello est généralement de 3 secondes, et le délai avant de considérer un routeur comme défaillant (Hold Time) est typiquement de 10 secondes

Détection de défaillance : Si un routeur ne reçoit pas de paquet Hello d’un routeur actif dans le délai spécifié (Hold Time), il supposera que le routeur actif est en échec

Cela peut déclencher une réélection pour choisir un nouveau routeur actif. Si notre routeur en mode Active n’est plus en état de fonctionner, une nouvelle élection à lieu.

Le routeur en mode Standby va donc passer en mode Active. Si notre routeur hors ligne est de nouveau en ligne, il n’y aura pas de nouvelle élection ! (il ne récupèrera donc pas son rôle avant la prochaine panne du nouveau routeur en mode Active, sauf si la preemtion est activé )

## Contenu Message Hello

Un paquet Hello contient plusieurs informations importantes, telles que : 

- l’adresse IP virtuelle du groupe HSRP
- la priorité du routeur qui envoie le paquet
- l’état actuel du routeur (actif, en attente, etc.)
- le numéro de groupe HSRP

## Groupe HSRP & MAC

### Groupe HSRP

Lors de la configuration des numéros de groupe pour HSRPv2 et HSRP, vous devez utiliser des numéros de groupe appartenant à des plages qui sont des multiples de 256. Les plages valides sont les suivantes : de 0 à 255, de 256 à 511, de 512 à 767, de 3 840 à 4 095, et ainsi de suite

Il est nécessaire de créer un groupe, lui attribuer une IP virtuel et éventuellement lui donner un nom

sur le 1er routeur

```bash
ip addresse 192.168.10.1
standby 1 name HSRP-LAN
standby 1 ip 192.168.10.254
standby 1 version 2s
standby 1 preempt
standby 1 priority 110
```

sur le 2ème routeur

```bash
ip addresse <192.168.10.2>
standby 1 name HSRP-LAN
standby 1 ip 192.168.10.254
standby 1 version 2s
standby 1 preempt
standby 1 priority 110
```

### MAC

Un groupe HSRP utilise une adresse MAC virtuelle pour identifier le routeur actif, différente selon la version du protocole

HSRP version 1:

- IPv4
- Adresse MAC utilisée : 0000.0C07.ACxx
- Utilise l’adresse multicast 224.0.0.2
- Groupe 0 au groupe 255

HSRP version 2

- IPv4
- Adresse MAC utilisée : 0000.0C9F.Fxxx
- Utilise l’adresse multicast 224.0.0.102
- Groupe 0 au groupe 4095

`xx` identifie le numéro de groupe HSRP

### Nom groupe

Il est possible de donner un nom au groupe HSRP pour faciliter son identification

```bash
standby 1 name HSRP_LAN0
```

### Modifier la priorité

Pour modifier la priorité pour influencer l'élection du routeur actif

```bash
standby 1 priority < 0 - 255 >
```

### Timers

Par défaut, un message hello est envoyé toutes les 3 secondes et le holdtime est de 10 secondes, il est possible de modifier ces valeurs

la première valeur est le temps entre chaque message hello, valeur entre 1 et 255 secondes ou spécifié une valeur en millisecondes, l’autre valeur est le holdtime compris entre 6-255 secondes

```bash
interface fa0/0
standby 1 timers 5 30
```

### Preempt

La préemption est une fonctionnalité de HSRP qui permet, que si un routeur avec une priorité supérieur est ajouté au groupe HSRP, il peut prendre la place du routeur actif si preempt est activé ( non activé par défaut ) ou si le routeur actif vient à ne plus être joignable, un autre routeur peut récupérer le rôle

Dans le cas ou un routeur tombe en panne, le changement de rôle s’effectue à partir du holdtime qui est trop long

```bash
interface gigabitEthernet0/0
standby 1 preempt
```

Lorsque, un routeur avec une priorité supérieur rejoins le groupe HSRP, il va prendre le rôle de routeur actif, après avoir envoyer un message hello pour donner sa priorité, il va envoyer un message “ coup “ pour prendre le rôle. Le routeur ayant le rôle actif va passer en état “ speak ” et envoyer un message “ resign ” pour annoncer qu’il libère le rôle pour un routeur avec une meilleur priorité

## Authentification

HSRP prend en charge l’authentification, une interception du traffic peut permettre de découvrir le numéro de groupe, l’adresse IP virtuel mais aussi le mot de passe si il est en clair, ce qui peut permettre à un attaquant de connaitre la configuration puis ajouter un routeur rogue

Une authentication en plain-text ( mot de passe clair ) n’apporte aucune sécurité si ce n'est que d'empécher un routeur d'être ajouté à la volé

```bash
Switch(config-if)# standby 1 authentication test
```

Le mieux serait à minima, de mettre une encryption MD5

```bash
Switch(config-if)# standby 1 authentication md5 key-string test
```

La méthode la plus sure, serait d’utiliser une key chain

```bash
Switch(config)# key chain HSRP_LAN0
Switch(config-keychain)# key 1
Switch(config-keychain-key)# key-string test
Switch(config)# interface fa0/0
Switch(config-if)# standby 1 authentication md5 key-chain HSRP_LAN0
```

## Configuration

Infrastructure avec 3 routeurs connecter ensemble par un switch

```bash
R1\
R2----SW1
R3/
```

### R1

```bash
key chian HSRP_LAN0
key 1
key-string test
ip address 192.168.0.1
no shutdown
standby 1 name HSRP_LAN0
standby 1 authentication md5 key-string
standby 1 ip 192.168.0.254
standby 1 preempt
standby 1 priority
```

### R2

```bash
key chian HSRP_LAN0
key 1
key-string test
ip address
no shutdown
standby 1 name HSRP_LAN0
standby 1 authentication md5 key-string
standby 1 ip 192.168.0.254
standby 1 preempt
standby 1 priority
```

### R3

```bash
key chian HSRP_LAN0
key 1
key-string test
ip address
no shutdown
standby 1 name HSRP_LAN0
standby 1 authentication md5 key-string
standby 1 ip 192.168.0.254
standby 1 preempt
standby 1 priority
```

## Administration

### Maintenance

Voir la configuration de HSRP, ajouter le nom d'interface permet de voir la configuration uniquement pour l'interface en question

```bash
show standby
FastEthernet0/0 - Group 1
    State is Active
        2 state changes, last changes 00:00:46
    Virtual IP Address is 192.168.0.254
    Active virtual MAC address is 0000.0c9f.f001
    Local virtual MAC address is 0000.0c9f.f001 (V2)
Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.508 secs
Preemption enabled
Active router is local
Standy router is 192.168.0.2, priority 100 (expires in 9.900sec)
Priority 100 (default 100)
Group name is "hsrp-Fa0/0-1" (default)
```

Affiche un résumé des groupes

```bash
show standby brief
                    P indicates configured to preempt
Interfaces Grp Pri P State   Active Standby      Virtual IP
Fa0/0       1  100 P Active  Local  192.168.0.2  192.168.0.254
```

Voir la configuration dans la running-config

```bash
show running-config | section standby
standby 1 ip 192.168.0.254
standby 1 preempt
```

Voir les voisins HSRP

```bash
show standby neighbors
HSRP neighbors on FastEthernet0/0
    192.168.0.2
        No active groups
        Standy groups: 1
```

### Sécurité

Utiliser une authentification, minimum MD5, préférer une key-string

```bash
key chian HSRP_LAN0
key 1
key-string test
standby 1 authentication md5 key-string
```
