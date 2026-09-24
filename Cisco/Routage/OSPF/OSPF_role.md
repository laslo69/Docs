
OSPF (Open Shortest Path First) est un protocole de routage à état de liens (link-state), conçu pour permettre aux routeurs d'échanger dynamiquement les informations de topologie du réseau au sein d'un système autonome (AS)

Contrairement aux protocoles de routage par distance comme RIP, OSPF utilise l'algorithme de Dijkstra (SPF) pour calculer les chemins les plus courts et converge rapidement, tout en supportant le découpage en areas pour améliorer la scalabilité

OSPF n'utilise ni TCP ni UDP, il fonctionne directement au-dessus d'IP avec le protocole IP numéro 89, ce qui signifie qu'aucun port TCP/UDP ne lui est associé

Les échanges se font par multicast, notamment sur les adresses 224.0.0.5 ( AllSPFRouters, pour l'échange avec tous les routeurs OSPF) et 224.0.0.6 (AllDRouters, réservée aux routeurs désignés DR et BDR sur les réseaux multi-accès), ou en unicast pour certaines adjacences

OSPF intègre son propre mécanisme d'authentification (en clair, MD5 ou SHA selon la version) et une prise en charge native d'IPv6 via OSPFv3

## Role dans l'Autonomous System

### Backbone Router ( BR )

- Routeur qui à une interface connecté à l’area  0 ( Backbone ), zone principale et obligatoire, sert à transmettre le traffic dans les area OSPF.
- Participe à la diffusion des informations de routage entre les area

### Internal Routeur ( IR )

- Un IR remplit des fonctions au sein d’une zone (area) uniquement, autre que la zone Backbone
- Sa fonction primordiale est d’entretenir à jour avec tous les réseaux de son area, sa link-state database qui est identique sur chaque IR
- Un IR va maintenir sa Link State Database à jour pour son area autre que la backbone uniquement et va calculer le côut des routes en utilisant l’algorithme SPF
- Pour transmettre les informations à une autre zone, il est nécessaire de passer par un ABR
- Un IR est un routeur membre d'une zone, autre que l'area 0
- Il renvoie toute information aux autres routeurs de son area, le routage ou Le flooding des autres zones requiert L’intervention d’un ABR

### Area Border Router ( ABR )

- Un ABR est un routeur qui connecte au moins 2 zones, dont l’area 0, il possède et maintient à jour autant de Link State Database qu’il y’a de zones.
- Il peut injecter une route par défaut dans les area autre que la Backbone si nécessaire et permet la communication entre les aires en annonçant les routes inter-area
- Chacune de ces bases de données contient la topologie entière de l'area connectée et peut donc être “summarizée”, c’est-à-dire agrégée en une seule route IP.
- Ces informations peuvent être transmises à la zone de backbone pour la distribution
- Un élément clé est qu’un ABR est l’endroit où l’agrégation doit être configurée pour réduire la taille des mises à jour de routage qui doivent être envoyées ailleurs

### Autonomous System Border Routeur ( ASBR )

- Un ASBR assure le rôle de ASBR et peut aussi être un ABR.
- Il permet de connecter plusieurs area OSPF et redistribuer des routes externes. Il permet de faire la liaison entre différentes area mais aussi avec d’autre AS
- OSPF est un IGP (Interior Gateway Protocol), autrement dit il devra être connecté au reste de l’Internet par d’autres AS.
- Ce type de routeur fera en quelque sorte office de passerelle vers un ou plusieurs AS. L’échange d’information entre un AS OSPF et d’autres AS est le rôle d’un ASBR.
- Les informations qu’il reçoit de l’extérieur seront redistribuées au sein de l’AS OSPF

## Rôle dans la zone

Les rôles de routeurs décrit sont dans le cas ou, les routeurs font parti du même segment réseau. Dans le cas ou, plusieurs segments sont présent, il est possible que des routeurs aient plusieurs rôles en simultané, ces rôles sont chacun pour un segment différent

### Designated Routeur ( DR )

- Le DR est le routeur élu pour représenter le segment réseau
- Au sein de son area, il va centraliser la diffusion des LSA et la redistribution des LSA aux autres routeurs pour diminuer le traffic en multicast, la centralisation va permettre de maintenir une adjacence complète avec tous les routeurs du segments
- Le DR génère des LSA réseau ( LSA Type 2 )

### Backup Designated Router ( BDR )

- Le BDR est le routeur de secours de la zone et prend le relais en cas de non réponse de la part du DR
- Il va surveiller le DR par le biais d’un message Hello régulier, si au bout d’un temps donné le DR ne répond pas, le BDR deviens DR et une nouvelle élection se fera pour élire un nouveau BDR
- le BDR va écouter les LSA de la zone mais, ne va pas les redistribuer tant que le DR est actif

### Designated Routeur Other ( DROTHER )

- Tous les routeurs autre que le DR et le BDR prendront le rôle de DROther
- Leurs fonction est d’établir une adjacence complète avec le DR et le BDR, ils vont aussi envoyé leurs LSA au DR pour établir sur le DR, la Link State Database et reçoivent les LSA du DR pour mettre à jour leur Link State Database

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

Si un routeur n'atteint jamais l'état "Full", il est impératif de diagnostiquer les problèmes rencontrés dans les états précédents, car l'absence de cet état indique un échec dans la synchronisation des bases de données ou dans la communication avec les voisins.

