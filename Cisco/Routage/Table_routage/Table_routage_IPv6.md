
Une table de routage est une base de données stockée dans un routeur ou un hôte, qui contient les informations nécessaires pour acheminer les paquets vers leur destination en enregistrant des routes par des protocoles dynamique ou statique

Elle est composée de plusieurs colonnes, chacune représentant des composant clé

## Composition table de routage

Exemple d'une table de routage extrait d'un routeur Cisco

La description de chaque partie sera fait en partant de la gauche

```bash
Router#sh ipv6 route 
IPv6 Routing Table - 7 entries
Codes: C - Connected, L - Local, S - Static, R - RIP, B - BGP
       U - Per-user Static route, M - MIPv6
       I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF intra, OI - OSPF inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       D - EIGRP, EX - EIGRP external
C   2000:DB8:0:1::/64 [0/0]
     via GigabitEthernet0/0, directly connected
L   2000:DB8:0:1::2/128 [0/0]
     via GigabitEthernet0/0, receive
O   2000:ACAD:0:1::/64 [110/1]
     via 
O   2001:DB8:ACAD:1::/64 [110/3]
     via FE80::2, GigabitEthernet0/0
O   2001:DB8:ACAD:2::/64 [110/2]
     via FE80::2, GigabitEthernet0/0
O   2001:DB8:ACAD:3::/64 [110/2]
     via FE80::2, GigabitEthernet0/0
L   FF00::/8 [0/0]
     via Null0, receive
```

### Code protocole

Un protocole de routage est une technologie qui permet de, selon l’infrastructure, de choisir par ou un paquet doit transiter.

Chaque protocole de routage, dynamique comme statique, est représenter dans une table de routage par une lettre ou symbole.

Liste non-exhaustive de code de protocole de routage

|Code|Protocole|Description|
|:-:|:-:|:-:|
|C|Connected|Réseau directement connecté à une interface du routeur|
|L|Local|Adresse IP locale de l’interface|
|S|Static|Route configuré manuellement|
|O|OSPF|Route apprise via OSPF|
|D|EIGRP|Route apprise via EIGRP|
|R|RIP|Route apprise via RIP|
|B|BGP|Route apprise via BGP|
|*|Candidate Default|Route par défaut candidate|

```bash
C   2000:DB8:0:1::/64 [0/0]
     via GigabitEthernet0/0, directly connected ! Route connecté en local
L   2000:DB8:0:1::2/128 [0/0]
     via GigabitEthernet0/0, receive ! Route apprise en local
O   2001:DB8:ACAD:1::/64 [110/3]
     via FE80::2, GigabitEthernet0/0 ! Route apprise par OSPFv3
```

### Prefix

Le préfixe représente,la partie réseau représenté dans l'adresse IPv6


```bash
2001:DB8:ACAD:1::/64 -> Le préfixe est 2001:db8:acad:1
```

### Netmask

Détermine le nombre de bits utilisé pour définir la taille du réseau et le nombre d'adresse IP utilisable sur le segment défini.

```bash
2001:DB8:ACAD:1::/64 -> Masque = /64
2000:DB8:0:1::2/128 -> Masque = /128
```

### Administrative Distance

La distance administrative est représenté par un entier entre 0 et 255. Ce chiffre permet de donne un niveau de confiance sur un routage vers la destination, de la plus faible ( confiance plus élevé ) à la plus haute ( confiance plus faible ).

Si plusieurs routes vont à la même destination mais avec des distances administrative égales, la distance administrative la plus basse est prioritaire.

Une route statique à par défaut, une distance administrative de 1, un lien local ( Connected ) à une distance administrative de 0 obligatoirement.

|Protocole|Distance Administrative|
|:-:|:-:|
|Connected|0|
|Static|1|
|EIGRP ( Summary )|5|
|BGP( External )|20|
|EIGRP( Internal )|90|
|OSPF|110|
|RIP|120|
|EIGRP ( External )|170|
|BGP ( Internal )|200|

```bash
C   2000:DB8:0:1::/64 [0/0]
     via GigabitEthernet0/0, directly connected ! AD de 0
O   2001:DB8:ACAD:3::/64 [110/2]
     via FE80::2, GigabitEthernet0/0 ! AD de 110
```

Dans la table de routage, une entrée `Connected` ou `local` à toujours une distance administrative égale à 0

### Metric

Une métrique est un coût ou poids associé à une route, utilisé par les protocoles de routages dynamique pour choisir le meilleur chemin parmis plusieurs routes avec la même distance administratives.

Si un routeur apprend deux routes OSPF vers le même réseau avec la même distance administrative, il choisira celle avec la métrique la plus basse car plus fiable.

Exemple:
```bash
C   2000:DB8:0:1::/64 [0/0]
     via GigabitEthernet0/0, directly connected ! Metrique égale à 0
O   2001:DB8:ACAD:3::/64 [110/2]
     via FE80::2, GigabitEthernet0/0 ! métrique égale à 110
```

### Next-Hop

Désigne l’adresse IP du prochain routeur vers lequel, envoyer un paquet pour atteindre la destination.

Quand un paquet passe d’un routeur à un autre, le mécanisme de forward d’un paquet entre reouteur s’appel : Packet rewriting

Avec IPv6, une route, qu'elle soit statique ou dynamique, doit stipuler l'interface de sortie et l'adresse link-local du voisin

```bash
O   2001:DB8:ACAD:1::/64 [110/3]
     via FE80::2, GigabitEthernet0/0
```

### Gateway of Last Resort

La passerelle par défaut sert ( ou route par défaut ), dans le cas ou un paquet doit joindre un réseau et que aucune route spécifique dans la table de routage ne correspond à la destination, à définir une adresse IP de sortie du réseau.

Sans route par défaut, les paquets destinés à des réseaux inconnus sont jetés. Elle est essentielle pour l’accès à Internet ou aux réseaux externes.

Souvent, l’adresse IP du routeur FAI est défini

```rust
S ::/0 [1/0]
via ::, Serial2/0
via ::, Serial3/0
```

- ::0/0 représente la route par défaut
- via ::, Serial2/0 est l’interface de sortie du routeur pour la route par défaut

## Decision routage

Lorsqu’un routeur reçoit un paquet, il doit décider par quelle interface l’envoyer pour atteindre la destination. Ce processus se déroule en étapes dans l'ordre suivant:

1) Longest Prefix Match :  Trouve la route la plus spécifique, en prenant celle avec le masque réseau le plus long. 
2) Administrative Distance : Choisis entre plusieurs routes vers la même destination, la route avec la distance administrative la plus faible est sélectionné.
3) Routing Protocol Metric : Sélectionner le meilleur chemin parmi les routes avec la même distance administrative, la métrique la plus basse est sélectionné. Si plusieurs routes ont la même distance administrative et métrique, le trafic est réparti. (ECMP – Equal Cost Multi-Path )

### Longest prefix match

Le Longest Prefix Match est, le processus dans lequel le routeur compare l’adresse IP de destination du paquet avec toutes les routes de sa table de routage.

Il sélectionne ensuite, la route avec le masque le plus long, c’est à dire le préfixe le plus spécifique. ( comprendre par masque le plus long, en partant du premier bit )

La sélection de cette route à pour raison, si plusieurs routes menent à la même déstination, la route la plus spécifique, et donc plus précise sera sélectionné.

Exemple : Un paquet destiné à 2001:db8:1:12/64 peut correspondre à deux routes :

```bash
2001:db8:1::/48
2001:db8:1:2::/64
```

Le routeur choisira 2001:db8:1:2::/64 car son masque est plus long

### Administrative distance

Si plusieurs routes correspondent exactement à la même destination (même préfixe), le routeur utilise la distance administrative (AD) pour choisir.

La Distance Administrative la plus basse est prioritaire

Paquet destiné à 2001:db8:1:12/64

```bash
O   2001:DB8:ACAD:1::/64 [110/3]
     via FE80::2, GigabitEthernet0/0
R   2001:DB8:ACAD:1::/64 [120/3]
     via FE80::2, GigabitEthernet0/0
```

La route OSPF sera sélectionné car, son AD est la plus basse, 110 < 120

### Routing protocol metric

Si plusieurs routes ont la même distance administrative, le routeur utilise la métrique pour choisir le meilleur chemin.

Plus la métrique est basse, mieux c’est.

La métrique est calculée différemment selon le protocole : 
- RIP : Nombre de sauts (hop count)
- OSPF : Coût basé sur la bande passante et la coût cumulé pour atteindre le destinataire.
- EIGRP : Formule complexe (bande passante, délai, fiabilité, charge)

Paquet destiné à 2001:db8:1:12/64 : 

```bash
O   2001:DB8:ACAD:1::/64 [110/3]
     via FE80::2, GigabitEthernet0/0
O   2001:DB8:ACAD:1::/64 [110/2]
     via FE80::2, GigabitEthernet0/1
```

Les deux routes ont la même AD (110), 2001:DB8:ACAD:1::/64 [110/2] via FE80::2, GigabitEthernet0/1 est choisie car sa métrique (2) est plus basse que l'autre

## Route appris dynamiquement

Quand une route est appris dynamiquement par un protocole de routage, les routes sont ajoutés dans la table de routage du routeur voisin

La route installé dans le routeur voisin sera uniquement celle avec la distance administrative la plus basse

Cisco privilégie toujours le protocole avec la distance administrative la plus basse
