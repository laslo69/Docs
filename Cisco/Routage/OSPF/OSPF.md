
OSPF (Open Shortest Path First) est un protocole de routage à état de liens (link-state), conçu pour permettre aux routeurs d'échanger dynamiquement les informations de topologie du réseau au sein d'un système autonome (AS)

Contrairement aux protocoles de routage par distance comme RIP, OSPF utilise l'algorithme de Dijkstra (SPF) pour calculer les chemins les plus courts et converge rapidement, tout en supportant le découpage en areas pour améliorer la scalabilité

OSPF n'utilise ni TCP ni UDP, il fonctionne directement au-dessus d'IP avec le protocole IP numéro 89, ce qui signifie qu'aucun port TCP/UDP ne lui est associé

Les échanges se font par multicast, notamment sur les adresses 224.0.0.5 ( AllSPFRouters, pour l'échange avec tous les routeurs OSPF) et 224.0.0.6 (AllDRouters, réservée aux routeurs désignés DR et BDR sur les réseaux multi-accès), ou en unicast pour certaines adjacences

OSPF intègre son propre mécanisme d'authentification (en clair, MD5 ou SHA selon la version) et une prise en charge native d'IPv6 via OSPFv3

## Election routeur

### Processus élection

Le processus d’élection du DR et du BDR suit un schéma précis, si le premier critère n’est pas validé, l’élection prendra le second critère et ainsi de suite

Il suit l'ordre suivant :

1) Priorité interface, valeur de la priorité d’une interface entre 0 et 255, 0 empèche l’élection alors que 255 le rend obligatoirement DR. Plus la priorité est haute, plus le routeur à de chance d’être DR
2) Le routeur ID doit être renseigner sous le format 1.1.1.1 ( valeur à personnalisé ). Plus le routeur ID est élevé, plus il à de chance d’être DR
3) Adresse ipv4 loopback active la plus élevé
4) Adresse ipv4 active la plus élevé

Cependant, si un routeur avec une priorité plus élevée est ajouté plus tard, le routeur ne deviendera pas automatiquement DR, il faut faire un reset de l’interface ou redémarrer le processus OSPF ou qu'un routeur tombe pour qu'une nouvelle élection ai lieu

Le processus d'élection se déroule comme suit et utilse des types de messages à chaque étape de l'élection :

Chaque routeur envoie des paquets Hello pour se découvrir entre eux,  les paquets Hello permettent de transmettre des informations ( Priorité, Router ID, liste des voisins connu, etc… )

Chaque routeur va alors comparé, ses caractéristique avec celles envoyé par les autres routeurs

Le routeur avec la plus haute priorité deviendra le DR, le routeur avec la deuxieme priorité la plus élevé deviens le BDR.

Une fois le DR et BDR élus, le DR envoi un message Hello à tous les autres routeurs pour qu’ils mettent à jour leur Link State Database et  devennir un DROther

### Messages & états

Durant la mise en place d’OSPF, les routeurs vont passer par différentes étapes et envoyés jusqu’a 5 type de messages avant d’atteindre un état de Full Adjency

Une adjacence est une relation entre des routeurs OSPF et le DR

Pour calculer le nombre d’adjacences d’une zone: n * ( n – 1 ) / 2

#### Down

L'état "Down" est le point de départ de tout processus de formation de voisinage OSPF

Dans cet état, il n'y a aucune communication entre les routeurs OSPF. Un routeur est considéré comme étant dans l'état "Down" s'il n'a pas reçu de Hello packet de son voisin depuis un certain temps. 

Ce délai par défaut est de 40 secondes pour la plupart des configurations. Cet état est crucial à comprendre lors du dépannage, car si un routeur reste dans l'état "Down", cela indique un problème de connectivité ou de configuration, tel qu'une mauvaise correspondance des timers ou une absence totale de communication entre les routeurs.

#### Init

Lorsque le routeur reçoit un paquet *Hello* d'un voisin, il passe à l'état "Init". Cet état signifie que le routeur a vu un autre routeur OSPF, mais il n'est pas encore sûr que ce dernier ait pris conscience de sa présence. L'état "Init" est essentiel car il marque le début de la formation d'une relation

Un routeur peut rester bloqué dans cet état si les paquets *Hello* ne contiennent pas l'ID du routeur local dans leur liste de voisins, ce qui peut être dû à une mauvaise configuration des interfaces OSPF ou à des problèmes de réseau plus profonds

Lorsqu’un routeur reçoit un paquet Hello d’un voisin, il doit indiquer l’ID du routeur expéditeur dans son paquet Hello en tant qu’accusé de réception d’un paquet Hello valide.

#### Two-way

L'état "2-Way" est l'un des plus importants dans OSPF. Il est atteint lorsque les deux routeurs échangent des paquets *Hello* et que chacun voit son propre ID de routeur dans la liste de voisins de l'autre

Cet état indique que la communication bidirectionnelle a été établie et qu'une relation de voisinage stable est possible. Cependant, tous les voisins OSPF ne progressent pas au-delà de cet état. Par exemple, sur un réseau multi-accès comme Ethernet, seuls les routeurs élus DR (Designated Router) et BDR (Backup Designated Router) passeront à l'état suivant

Pour le dépannage, un routeur bloqué en "2-Way" peut indiquer une configuration incorrecte du DR/BDR ou une absence de progression vers une relation plus approfondie

#### Exstart

L'état "ExStart" marque le début du processus d'échange de bases de données entre deux routeurs

Ici, les routeurs déterminent lequel d'entre eux deviendra le maître de l'échange et lequel sera l'esclave

Le maître commencera à envoyer des *DD* (DataBase Description) packets, et l'esclave répondra. Les routeurs doivent convenir d'une valeur de MTU compatible, et des problèmes à ce niveau peuvent causer des échecs dans le processus. Cet état est crucial pour le dépannage, car un routeur bloqué dans "ExStart" peut indiquer des problèmes de compatibilité MTU, de synchronisation de l'état de la base de données, ou de connectivité.

#### Exchange

Une fois l'état "ExStart" réussi, les routeurs passent à l'état "Exchange"

Dans cet état, ils échangent des *DD* packets, qui contiennent des résumés des LSA (Link-State Advertisements) que chaque routeur connaît. Le but de cet état est de s'assurer que chaque routeur a une vue à jour et cohérente de la topologie du réseau

Si un routeur reste bloqué dans l'état "Exchange", cela peut indiquer des problèmes avec les paquets *DD*, tels qu'une incompatibilité de MTU, une perte de paquets, ou des problèmes de traitement des LSA.

Dans le cas ou, aucune différence est constaté, l’échange ne continu pas.

#### Loading

Dans l'état "Loading", les routeurs demandent les LSA manquantes qu'ils ont découvertes dans l'état précédent mais qu'ils ne possèdent pas encore. Ils envoient des paquets *LSR* (Link State Request) pour obtenir ces LSA manquantes

Cet état est crucial car il complète le processus de synchronisation de la base de données. Si un routeur est bloqué dans l'état "Loading", cela peut signaler des problèmes avec la réception ou l'envoi des LSA, tels que des erreurs de transmission, des paquets corrompus, ou des ressources insuffisantes sur le routeur pour traiter les LSA reçues.

#### Full adjency

L'état "Full" est l'état final dans le processus de formation d'une relation OSPF

Ici, les bases de données des routeurs voisins sont complètement synchronisées, et ils sont prêts à participer pleinement à l'élection du chemin le plus court à travers le réseau. Cet état signifie que le processus de voisinage a été correctement complété, et que les routeurs peuvent échanger des informations de routage de manière fiable

Si un routeur n'atteint jamais l'état "Full", il est impératif de diagnostiquer les problèmes rencontrés dans les états précédents, car l'absence de cet état indique un échec dans la synchronisation des bases de données ou dans la communication avec les voisins

## Cout OSPF

### Algorithme

OSPF utilise l’algorithme de Dijkstra, Shortest Path First ( SPF ) pour calculer le coût d’un lien et déterminer la route la plus rapide vers une destination. Un coup plus faible sera un chemin préféré.

Le coût est un chiffre entier compris entre 1 et 255

La bande passante de référence, par défaut correspond à 10^8 ( 100 000 000 = 100 mbps )

Calcul :

Coût = Bande passante de référence / bande passante de l’interface

exemple: 

Bande passante de référence = 100 Mbps, Lien à 10 Mbps
Coût = 100/10 => 10

une bande passante d’interface de 10Go/s
Coût = 100 000 / 10 000 000 000  => 1 ( ne peut pas être inférieur à 1 )

Quand OSPF calcule le coût des chemins, il va utiliser son routeur comme Rooted Tree, à partir de ce Rooted Tree, le calcul des chemins vers les routeurs voisins va permettre de constituer un SPF Tree qui sert à cartographié, pour chaque lien le coût associé

### Modifier cout de reference

Il est possible de modifier le coût OSPF d’un lien pour favoriser ou forcer un chemin.

Par exemple, sur un switch qui possède un lien 1Go/S et un lien 100Go/S , les deux liens ont un côut de 1 ce qui fait que, OSPF ne fait pas le différence entre les deux lien. En modifiant le coût manuellement, il est possible par exemple de favoriser le lienb 100Go/S pour lui donner une priorité et faire en sorte que le lien 1Go/S soit utilisé en secondaire ou en lien de backup

exemple:

Ici, l’auto-cost se compte en mbits ( 100 par défaut ). En modifiant l’auto-cost à 10 000, la référence passe de 100Mb à 10 000Mbit ( soit 10Gb ), mon lien à 10 Go/S aura un coût égale à 1 alors que mon lien à 1Go/S aura un coût égale à 10.

Le lien 100Go/s sera favorisé pour le transport de paquet du fait de son faible coût.

Cependant, si la valeur de l’auto-cost est modifié, tous les routeurs du domaine OSPF doit avoir la même valeur. Une valeur différente peut emmené à des disfonctionnement ou des irrégularité du routage.

```bash
router ospf 1
auto-cost reference bandwith 10000
```

### Modifier cout interface

Il est possible de modifier le coût directement sur un interface en spécifiant une valeur défini.

Par exemple, je choisis de mettre un coût à 5 alors qu’en temps normal, le coût serait bien supérieur. Permet d’influencer les liens utilisé

L'interet de modifier le cout d'une interface, permet de favoriser ou défavoriser volontairement des liens, la modification est locale et n'a pas besoin d'être établi sur d'autre routeur de la zone ou du domaine OSPF

```bash
interface gigabitEthernet0/0
ip ospf cost 5
```

## Authentication

OSPF prend en charge plusieurs mécanisme d’authentification, l’objectif est de sécuriser les échanges en les rendant illisibles pour toutes personnes extérieur ou dans le cas d’un routeur rogue OSPF ajouté et  garantir l’intégrité des messages OSPF

OSPF prend en charge

- Aucune authentification: Aucun mot de passe pour sécurisé les échanges
- Plain text: un mot de passe configuré à la main mais apparait en clair lors de capture de trame, ce qui rend sa sécurité de très faible à inexistante
- Hachage MD5 ou SHA: Mot de passe partagé et un id de clé son configuré sur chaque routeur, le routeur émetteur calcul un hash pour chaque message, ce hash est ajouté dans l’en-tête du paquet. Le routeur destinataire va recalculé le hash avec sa clé local et comparer les valeurs. Le mot de passe ne circule jamais sur le réseau et permet d’éviter les attaques par packet crafting ou une alteration des paquets.

Il peut être intéressant d’utiliser une authentification pour chaque zone et définir un mot de passe par interface et de préféré le SHA-256 en premier, si pas de possibilité prendre le MD5

## Interface passive

Mettre une interface en passive interface, permet de dire au routeur qu’il ne faut pas envoyer et recevoir de messages OSPF sur cette interface pour éviter de flooder un réseau inutilement

Par exemple, si l’interface est connecté directement sur un LAN utilisateur, envoyé des messages OSPF ne servirais à rien car les périphériques client ne seront pas en mesure de traiter ces messages

Il peut être intéressant de spécifier explicitement les interfaces à mettre en actif, la première commande permet de mettre en passif tout les interfaces et après, retirer sur un interface précis le statut de passif

```bash
passive-interface default
no passive-interface gigabitEthernet 0/0/0
```

- Passive-interface default : Met toutes les interfaces en état `passive`
