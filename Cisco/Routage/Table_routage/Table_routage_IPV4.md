
Une table de routage est une base de données stockée dans un routeur ou un hôte, qui contient les informations nécessaires pour acheminer les paquets vers leur destination en enregistrant des routes par des protocoles dynamique ou statique

## Composition table de routage

Elle est composée de plusieurs colonnes, chacune représentant des composant clé

Exemple d'une table de routage extrait d'un routeur Cisco, la description de chaque partie sera fait en partant de la gauche

```bash
Router# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 192.168.1.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 192.168.1.1
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.1.1.0/24 is directly connected, GigabitEthernet0/0
L        10.1.1.1/32 is directly connected, GigabitEthernet0/0
O        192.168.2.0/24 [110/2] via 10.1.1.2, 00:00:10, GigabitEthernet0/0
S        192.168.3.0/24 [1/0] via 10.1.1.3
```

### Code protocole

Un protocole de routage est une technologie qui permet de, selon l’infrastructure, de choisir par ou un paquet doit transiter.

Chaque protocole de routage, dynamique comme statique, est représenter dans une table de routage par une lettre ou symbole.

Liste non-exhaustive de code de protocole de routage

| Code |     Protocole     |                      Description                       |
| :--: | :---------------: | :----------------------------------------------------: |
|  C   |     Connected     | Réseau directement connecté à une interface du routeur |
|  L   |       Local       |            Adresse IP locale de l’interface            |
|  S   |      Static       |              Route configuré manuellement              |
|  O   |       OSPF        |                 Route apprise via OSPF                 |
|  D   |       EIGRP       |                Route apprise via EIGRP                 |
|  R   |        RIP        |                 Route apprise via RIP                  |
|  B   |        BGP        |                 Route apprise via BGP                  |
|  *   | Candidate Default |               Route par défaut candidate               |
Exemple : 

```bash
O 192.168.2.0/24 [110/2] via 10.1.1.2 → Route apprise via OSPF. 
S 192.168.3.0/24 [1/0] via 10.1.1.3 → Route statique.
```

### Prefix

Le préfixe représente,la partie réseau représenté dans l'adresse IPv4.

```rust
192.168.2.0/24 → Le préfixe est 192.168.2.0. 
10.1.1.0/24 → Le préfixe est 10.1.1.0.
```

### Netmask

Détermine le nombre de bits utilisé pour définir la taille du réseau et le nombre d'adresse IP utilisable sur le segment défini.

```rust
192.168.2.0/24 → Masque = 255.255.255.0
10.1.1.0/30 → Masque = 255.255.255.252
```

### Administrative Distance

La distance administrative est représenté par un entier entre 0 et 255. Ce chiffre permet de donne un niveau de confiance sur un routage vers la destination, de la plus faible ( confiance plus élevé ) à la plus haute ( confiance plus faible ).

Si plusieurs routes vont à la même destination mais avec des distances administrative égales, la distance administrative la plus basse est prioritaire.

Une route statique à par défaut, une distance administrative de 1, un lien local ( Connected ) à une distance administrative de 0 obligatoirement.

|     Protocole      | Distance Administrative |
| :----------------: | :---------------------: |
|     Connected      |            0            |
|       Static       |            1            |
| EIGRP ( Summary )  |            5            |
|  BGP( External )   |           20            |
| EIGRP( Internal )  |           90            |
|        OSPF        |           110           |
|        RIP         |           120           |
| EIGRP ( External ) |           170           |
|  BGP ( Internal )  |           200           |

```bash
O 192.168.2.0/24 [110/2] via 10.1.1.2 -> Indique une AD de 110
S 192.168.3.0 [ 1/0] via 10.1.1.3 -> Indique une AD de 1
```

### Metric

Une métrique est un coût ou poids associé à une route, utilisé par les protocoles de routages dynamique pour choisir le meilleur chemin parmis plusieurs routes avec la même distance administratives.

Si un routeur apprend deux routes OSPF vers le même réseau avec la même distance administrative, il choisira celle avec la métrique la plus basse car plus fiable.

```bash
O 192.168.2.0/24 [110/2] via 10.1.1.2 → Métrique = 2 ( coût OSPF )
D 192.168.4.0/24 [90/30720] via 10.1.1.4 → Métrique EIGRP = 30720
```

### Next-Hop

Désigne l’adresse IP du prochain routeur vers lequel, envoyer un paquet pour atteindre la destination.

Quand un paquet passe d’un routeur à un autre, le mécanisme de forward d’un paquet entre reouteur s’appel : Packet rewriting

```bash
O 192.168.2.0/24 [110/2] via 10.1.1.2 -> Métrique = 2 ( coût OSPF ). 
D 192.168.4.0/24 [90/30720] via 10.1.1.4 -> Métrique EIGRP = 30720.
via 10.1.1.2 -> Le prochain saut est 10.1.1.2 car plus faible
```

### Gateway of Last Resort

La passerelle de dernier recours, sert dans le cas ou un paquet doit joindre un réseau et que aucune route spécifique dans la table de routage ne correspond à la destination, à définir une adresse IP de sortie du réseau

Sans route par défaut, les paquets destinés à des réseaux inconnus sont jetés. Elle est essentielle pour l’accès à Internet ou aux réseaux externes

Souvent, l’adresse IP du routeur FAI est défini

```bash
Gateway of last resort is 192.168.1.1 to network 0.0.0.0
S*    0.0.0.0/0 [1/0] via 192.168.1.1
```

- 0.0.0.0/0 représente la route par défaut
- via 192.168.1.1 est l’adresse du prochain saut

## Decision routage

Lorsqu’un routeur reçoit un paquet, il doit décider par quelle interface l’envoyer pour atteindre la destination. Ce processus se déroule en étapes dans l'ordre suivant:

1) Longest Prefix Match : Trouve la route la plus spécifique, en prenant celle avec le masque réseau le plus long. 
2) Administrative Distance : Choisis entre plusieurs routes vers la même destination, la route avec la distance administrative la plus faible est sélectionné.
3) Routing Protocol Metric : Sélectionner le meilleur chemin parmi les routes avec la même distance administrative, la métrique la plus basse est sélectionné. Si plusieurs routes ont la même distance administrative et métrique, le trafic est réparti. ( ECMP – Equal Cost Multi-Path )

### Longest prefix match

Le Longest Prefix Match est, le processus dans lequel le routeur compare l’adresse IP de destination du paquet avec toutes les routes de sa table de routage.

Il sélectionne ensuite, la route avec le masque le plus long, c’est à dire le préfixe le plus spécifique. ( comprendre par masque le plus long, en partant du premier bit )

La sélection de cette route à pour raison, si plusieurs routes menent à la même déstination, la route la plus spécifique, et donc plus précise sera sélectionné.

Exemple : Un paquet destiné à 192.168.1.50 peut correspondre à deux routes :
```bash
192.168.1.0/24 (masque : 255.255.255.0) 
192.168.1.32/27 (masque : 255.255.255.224)
```

Le routeur choisira 192.168.1.32/27 car son masque est plus long (/27 > /24)

### Administrative distance

Si plusieurs routes correspondent exactement à la même destination (même préfixe), le routeur utilise la distance administrative (AD) pour choisir.

La Distance Administrative la plus basse est prioritaire

Modifier la distance administrative peut causer des boucles de routages et anomalies réseau

Paquet destiné à 192.168.2.10
```bash
S        192.168.2.0/24 [1/0] via 10.0.0.2
O        192.168.2.0/24 [110/2] via 10.0.0.3
R        192.168.2.0/24 [120/1] via 10.0.0.4
```

La route statique sera sélectionné car, son AD est la plus basse 1 < 110 < 120

### Routing protocol metric

Si plusieurs routes ont la même distance administrative, le routeur utilise la métrique pour choisir le meilleur chemin.

Plus la métrique est basse, mieux c’est

La métrique est calculée différemment selon le protocole : 
- RIP : Nombre de sauts (hop count)
- OSPF : Coût basé sur la bande passante et la coût cumulé pour atteindre le destinataire
- EIGRP : Formule complexe (bande passante, délai, fiabilité, charge)

Paquet destiné à 192.168.3.50 : 

```bash
O        192.168.3.0/24 [110/10] via 10.0.0.5, 00:01:10, GigabitEthernet0/1
O        192.168.3.0/24 [110/20] via 10.0.0.6, 00:01:05, GigabitEthernet0/2
```

Les deux routes ont la même AD (110), 10.0.0.5 est choisie car sa métrique (10) est plus basse que 20

## Route appris dynamiquement

Quand une route est appris dynamiquement par un protocole de routage, les routes sont ajoutés dans la table de routage du routeur voisin

La route installé dans le routeur voisin sera uniquement celle avec la distance administrative la plus basse

Cisco privilégie toujours le protocole avec la distance administrative la plus basse