
OSPF est organisé en différent rôle au sein d'un AS et inter-AS mais aussi de rôle propre dans chaque area défini.

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
