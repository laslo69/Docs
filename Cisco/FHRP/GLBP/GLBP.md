
GLBP est un protocole propriétaire Cisco qui assure la redondance tout en permettant la répartition de charge entre plusieurs routeurs

Un routeur est désigné comme AVG (Active Virtual Gateway) et attribue différentes adresses MAC virtuelles aux clients, les autres routeurs agissent en tant qu’AVF (Active Virtual Forwarders) et partagent la charge selon une méthode définie

Contrairement à HSRP et VRRP, tous les routeurs peuvent être actifs en même temps, cela améliore l’utilisation des ressources et les performances du réseau et permet surtout, de prendre en charge nativement le load-balancing GLBP est idéal pour les architectures critiques mais reste limité à un environnement Cisco

GLBP utilise le socket multicast 224.0.0.102:UDP/3222

## Etat et rôle GLBP

### Etat GLBP

1) Initial

- État au démarrage de GLBP
- Le routeur n'a pas encore participé au processus d'élection

2) Listen

- Le routeur écoute les messages Hello envoyés par les autres membres du groupe
- Il connaît l'existence de l'AVG mais ne participe pas activement à la gestion de la passerelle virtuelle
- Si l'AVG disparaît, il peut participer à une nouvelle élection

3) Speak

- Le routeur envoie et reçoit des messages Hello
- Il participe activement à l'élection de l'AVG
- Si sa priorité est la plus élevée (ou, à priorité égale, son adresse IP est la plus élevée), il devient l'AVG

4) Active

- Le routeur est élu Active Virtual Gateway (AVG)
- Il répond aux requêtes ARP des clients avec les adresses MAC virtuelles des AVF (Active Virtual Forwarders)
- Il gère l'attribution des forwarders et supervise le groupe

### Rôle GLBP

GLBP possède 2 types de routeur:

- AVG ( active virtual gateway )
- AVF ( active virtual forwarding, max 4 actif )

AVG:

- Gére l’adresse IP virtuelle
- Répond aux ARP des clients
- Attribue des adresses MAC virtuelles aux clients
- Distribue les rôles AVF

AVF:
 
- Tous les routeurs du groupe GLBP peuvent devenir AVF
- Le routeur AVG choisi qui va être AVF et lui affecte une adresse MAC virtuelle

## Election routeur

Pour procéder à l’élection, chaque routeur possède des propriétés

- Chaque routeur possède une priorité
- Cette priorité est modifiable par l’administrateur
- La priorité par défaut est à 100. Elle est comprise entre 0 et 255
- Le routeur possédant la plus haute priorité devient le routeur “ Active “, en cas d’égalité, celui avec l’adresse IP la plus haute l’emporte

Au démarrage, tous les routeur sont en état “ Init “ et passent en “ Learn “ pour écouter les messages “ Hello “

Dans le protocole GLBP, les paquets “Hello” jouent un rôle crucial dans la communication entre les routeurs membres d’un groupe GLBP, voici les points essentiels à connaître à propos des paquets Hello dans GLBP :

- Les paquets Hello sont utilisés pour annoncer la présence d’un routeur au sein d’un groupe HSRP et pour établir et maintenir la communication entre les routeurs actifs et de secours
- Intervalle et temporisation : Les paquets Hello sont envoyés périodiquement. L’intervalle par défaut entre les paquets Hello est généralement de 3 secondes, et le délai avant de considérer un routeur comme défaillant (Hold Time) est typiquement de 10 secondes

Les paquets Hello sont utilisés lors du processus d’élection du routeur actif et de secours. Les routeurs comparent les informations reçues via les paquets Hello pour décider quel routeur doit être actif ou de secours

Un paquet Hello contient plusieurs informations importantes, telles que l’adresse IP virtuelle du groupe GLBP, la priorité du routeur qui envoie le paquet, l’état actuel du routeur (actif, en attente, etc.), et le numéro de groupe GLBP

Si un routeur ne reçoit pas de paquet Hello d’un routeur actif dans le délai spécifié (Hold Time), il supposera que le routeur actif a échoué. Cela peut déclencher une réélection pour choisir un nouveau routeur actif. Si notre routeur en mode Active n’est plus en état de fonctionner, une nouvelle élection à lieu

## Groupe GLBP & MAC

### Groupe GLBP

Configuration d’un groupe GLBP

```rust
R1(config)#interface fa0/0
R1(config-if)#glbp 1 ip 10.0.0.254
R1(config-if)#glbp 1 preempt
R1(config-if)#glbp 1 priority 140
```

### Adresse MAC

Lors de la configuration d'un groupe GLBP, le route AVG distribue une adresse MAC virtuelle à 4 routeurs AVF

L'adresse MAC se présente sous la forme :

0007.b400.0101

0007.b4 permet d'identifier une adresse MAC appartenant au processus GLBP.

Les autres sections de l'adresse MAC servent à identifier le groupe GLBP et le numéro de routeurs du groupe

- 00.01 : identifie le groupe GLBP, 0 à 1023
- 01 : Identifie le numéro du routeur dans le processus GLBP, support jusqu'à 1024 routeurs

### Modifier la priorité:

Pour influencer l'élection du routeur qui aura le rôle AVG, il est possible de modifier la priorité pour prendre la décision avant que le processus d'élection se fasse

```rust
glbp 1 priority < 0 - 255 >
```

Plus la priorité est élevé, plus il est probable que le routeur devient l'AVG.

Une valeur à 0 l'empèche de devenir AVG, une valeur de 255 le fait passer d'office

### Preempt AVG

La préemption est une fonctionnalité de GLBP qui permet, que si un routeur avec une priorité supérieur est ajouté au groupe GLBP, il peut prendre la place du routeur AVG.

```rust
interface gigabitEthernet0/0
glbp 1 preempt
```

Lorsque, un routeur avec une priorité supérieur rejoins le groupe GLBP, il va prendre le rôle de routeur AVG, après avoir envoyer un message hello pour donner sa priorité, il va envoyer un message “ coup “ pour prendre le rôle. Le routeur ayant le rôle actif va passer en état “ speak ” et envoyer un message “ resign ” pour annoncer qu’il libère le rôle pour un routeur avec une meilleur priorité

Dans le cas ou un routeur tombe en panne, le changement de rôle s’effectue à partir du holdtime qui est trop long

Dans le cas ou, ont souhaite définir un routeur comme étant le principale de facto, preempt lui permet qu’en cas de retour après une panne ou redémarrage, de reprendre son rôle de routeur actif

### Preempt AVF

Délai pour que, un nouveau routeur puisse prendre le rôle d'un AVF

```rust
interface gigabitEthernet0/0
glbp 1 forwarder preempt
```

### Timers Holdtime

Les messages “ Hello “ sont envoyé toutes les 3 secondes et un holdtime de 10 secondes, il est possible de modifier les timers mais, uniquement sur l’AVG, les autres routeurs du groupe GLBP vont apprendre les timers à partir des “ Hello “ du AVG

Pour modifier les timers “ Hello “ et “ Holdtime “ dans l’ordre

```rust
glbp 1 timers <1-60> <6-180>
```

### Timers Redirect time

Si un routeur AVF est considéré comme “Dead” le routeur AVG continue d’envoyer l’adresse MAC virtuelle de ce routeur pendant un certain temps.

Ce laps de temps est le « redirect time », redirect Time par défaut = 600s

Si le routeur en question ne revient pas en ligne, cette adresse MAC virtuelle ne sera plus distribué.

Si un routeur AVF est considéré comme “Dead”, un autre AVF va supporter l’adresse MAC du routeur “dead”, il va garder cette adresse MAC pendant un certain temps.

Ce temps correspond au « Secondary Holdtime », secondary Holdtime par défaut = 14400s

Pour modifier les “ Redirect Time “, dans l'ordre : Redirect time, secondary holdtime

```rust
glbp 1 timers redirect <0-3600> <720-64800>
```

## Load-Balance

Le load-balancing est pris en charge nativement avec GLBP en distribuant le traffic sur les différentes adresse MAC virtuels des AVF du groupe GLBP

Lorsque plusieurs PC demandent la passerelle, tous utilisent la même IP virtuelle, mais leurs trames Ethernet sont envoyées vers différents routeurs, cela répartit naturellement le trafic

GLBP propose plusieurs algorithmes d’équilibrage 

### Round Robin

Les MAC virtuelles sont distribuées tour à tour aux nouveaux clients, exemple avec un groupe GLBP de 3 routeurs, en assumant que R1 est l'AVG

- Client 1 → R1
- Client 2 → R2
- Client 3 → R3
- Client 4 → R1
- ...

```rust
R1(config-if)#glbp 1 load-balancing round-robin
```

### Host-dependent

Un même client obtient toujours la même MAC virtuelle, ce qui peut être utile pour certaines applications en empéchant le switching de routeur en cours d'utilisation

```rust
interface gigabitEthernet0/0
glbp 1 load-balancing host-dependent
```

### Weighted

La répartition dépend d'un poids attribué à chaque routeur.

Le système de Weighting se base sur le nombre de clients (plus précisément, sur le ratio de requêtes ARP résolues), et non sur le volume de trafic réel ou le nombre de paquets

Plus le weight est élevé, plus le routeur peut recevoir du traffic, si le weight descend en dessous de la valeur “ lower “, le routeur arrête de forward et quitte son rôle AVF

si le weight dépasse le “ upper “, le routeur reprend le rôle de AVF

Exemple :

- R1 : weight 100 
- R2 : weight 50 

R1 recevra environ deux fois plus de trafic.

Pour configurer le weighting

```rust
track 1 gigabitEthernet0/0 ip routing
interface gigabitEthernet0/0
glbp 1 weighting track 1 110 lower 95 upper 105
glbp 1 weighting track 1 decrement 5
```

## Authentification

GLBP prend en charge l’authentification, comme les paquets sont transmis en multicast, une interception du traffic peut permettre de découvrir le numéro de groupe, l’adresse IP virtuel mais aussi le mot de passe si il est en clair, ce qui peut permettre à un attaquant de connaitre la configuration pour ajouter un routeur rogue

Une authentication en plain-text ( mot de passe clair ) n’apporte aucune sécurité

```rust
Switch(config-if)# glbp 1 authentication Finger
```

Le mieux serait à minima, de mettre une encryption MD5

```rust
Switch(config-if)# glbp 1 authentication md5 key-string Finger
```

La méthode la plus sure, serait d’utiliser une key chain

```rust
Switch(config)# key chain Finger-chain
Switch(config-keychain)# key 1
Switch(config-keychain-key)# key-string [ 0 | 7 ] Finger
Switch(config)# interface vlan 10
Switch(config-if)# glbp 1 authentication md5 key-chain Finger-chain
```
