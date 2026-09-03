
Une route statique est une route, défini manuellement par l'administrateur réseau pour pouvoir contrôler finement, vers qu'elle destination, un routeur peut forward des paquets en fonction de l'adresse IP source et de destination

## Default Route

Une route par défaut (0.0.0.0/0) est utilisée lorsque aucune route plus spécifique ne correspond à la destination du paquet. Elle est souvent appelée "gateway of last resort" (passerelle de dernier recours)

Elle permet, dans le cas ou un routeur n'est pas en mesure de, forward un paquet vers l'IP demandé, de le transférer vers une destination qui va servir à drop tous les paquets inconnus

Utile dans des infrastructure avec peu de routeurs

Mise en place de la route par défaut pour envoyer tout le trafic non local vers un routeur FAI

```bash
R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.3.2
R2(config)#ip route 0.0.0.0 0.0.0.0 192.168.3.1
```

## Network Route

Une Network Route, est une route statique configuré pour atteindre un réseau distant normalement non accessible avec la table de routage de base.

La configuration de la route doit se faire sur chaque routeur pour pouvoir, dans un premier temps émettre, mais aussi recevoir la réponse

Généralement, il faut configurer les routes sur les deux routeurs pour que, le routeur destinataire sache comment atteindre le routeur source

Mise en place de la Network Route

```bash
R1(config)#ip route 192.168.2.0 255.255.255.0 192.168.3.2
R2(config)#ip route 192.168.1.0 255.255.255.0 192.168.3.1
```

## Host route

Une route hôte est une route vers une adresse IP spécifique (masque /32). Elle est utilisée pour atteindre un hôte particulier (ex. un serveur, un routeur distant)

Permet de diriger le trafic vers une machine critique ou pour tester la connectivité vers un hôte précis

Mise en place de la Host route

```bash
Router(config)# ip route 192.168.1.100 255.255.255.255 10.0.0.3
```

## Floating Route

Une route flottante est une route de secours avec une distance administrative plus élevée que la route principale. Elle est utilisée pour assurer la redondance en cas de panne du lien principal

Dans le cas ou, le lien principal tombe en panne, la route statique défini va permettre, à vitesse égale ou en service dégradé, de garantir une continuité de service

Une distance administrative doit être saisie manuellement à la fin de la route flottante. Il peut être intéressant de mettre une route statique avec une distance administrative élevé, dans le cas ou un protocole de routage dynamique crée une route similaire, la route statique flottante peut prendre le relais en cas de panne du processus

Mise en place de Floating Route

```bash
R1(config)#no ip route 192.168.2.0 255.255.255.0 192.168.3.2
R1(config)#ip route 192.168.2.0 255.255.255.0 192.168.3.2 130
R2(config)#no ip route 192.168.1.0 255.255.255.0 192.168.3.1
R2(config)#ip route 192.168.1.0 255.255.255.0 192.168.3.1 130
```

## Configuration

Configuration avec 3 routeurs en ligne

```rust
R1 <----> R2 <-----> R3
```

### R1

```bash
int e0/0
ip address 192.168.0.2 255.255.255.0
no shutdown
#### Route statique
ip route 192.168.1.0 255.255.255.0 192.168.0.1
#### Default route
ip route 0.0.0.0 0.0.0.0 192.168.0.1
```

### R2

```bash
int e0/0
ip address 192.168.0.1 255.255.255.0
no shutdown
int e0/1
ip address 192.168.1.1 255.255.255.0
int e0/2
ip address 10.0.0.1 255.255.0.0
no shutdown
#### Route flottante, administrative distance 120
ip route 10.0.0.0 255.255.0.0 10.0.0.2 120
no shutdown
```

### R3

```bash
int e0/0
ip address 192.168.1.2 255.255.255.0
no shutdown
#### Route statique
ip route 192.168.0.0 255.255.255.0 192.168.1.1
#### Default route
ip route 0.0.0.0 0.0.0.0 192.168.1.1
```

## Administration

### Sécurité

- Sécuriser les accès physique et distant pour ne pas avoir accès à la table de routage
- ACL pour filtrer le trafic pour empécher tout clients non-autorisé
- Vérifier que les routes ne pointent pas vers des réseaux non-fiables
- Utiliser des routes flottantes pour la redondance
- Utiliser des mots de passe pour les protocoles de routage dynamique

### Dépannage

voir la table de routage, possible de voir seulement un type de route en fonction du paramètre `[ospf | static | RIP | ...]`

```bash
show ip route
Codes:  C - connected,S - static,R - RIP,M - mobile,B - BGP
        D - EIGRP,EX - EIGRP external,O - OSPF,IA - OSPF inter area
        N1 - OSPF NSSA external type1,N2 - OSPF NSSA external type2
        E1 - OSPF external type1,E2 - OSPF external type2
        i - IS-IS,su - IS-IS summary,L1 - IS-IS level1,L2 - IS-IS level2
        ia - IS-IS inter are,* - candidate dafault,U - per-user static route
        o - ODR,P - periodic downloaded static route

gateway of last resort is 192.168.0.1 to network 0.0.0.0

C  192.168.0.0/24 is directly connected, FastEthernet0/0
S  192.168.1.0/24 [120/0] via 192.168.0.1
S* 0.0.0.0/0 [1/0] via 192.168.0.1
```

Contrôler une route précise

```bash
show ip route 192.168.1.0
Routing entry for 192.168.1.0/24
    Know via "static", distance 120, metric 0
    Routing Descriptor Blocks:
    * 192.168.0.1
        Route metric is 0, traffic share count is 1
```

Lister un sommaire du nombre de routes avec son type

```bash
show ip route summary
IP routing table name is Default-IP-Routing-Table(0)
IP routing table maximum-paths is 16
Route Source    Networks    Subnets     Overhead    Memory(bytes)
Connected           1           0           72          128
Static              2           0           144         256
Total               3           0           216         384
```

Voir les routes enregistré manuellement, dans la running-config

```bash
show running-config | include ip route
ip route 0.0.0.0 0.0.0.0 192.168.0.1
ip route 192.168.1.0 255.255.255.0 192.168.0.1 120
```

Tester une route avec les commandes:
- Ping
- Traceroute

Vérifier si le voisin est détecté via CDP ou LLDP

```bash
show cdp neighbors
show lldp neighbors
```
