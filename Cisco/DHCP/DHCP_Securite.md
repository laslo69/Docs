
## DHCP Snooping

DHCP Snooping est une technologie de sécurité de couche 2 intégrée dans le système d’exploitation d’un switch qui connecte les clients aux serveurs DHCP et supprime le trafic DHCP provenant d'un serveur non autorisé. Il empêche les serveurs DHCP non autorisés de fournir des adresses IP aux clients.

La fonction DHCP Snooping permet d’effectuer les activités suivantes :

- Tout les ports sont mit par défaut, en non-sur ( Untrusted ), permet de définir un port considéré comme sur ( Trusted ) qui sera autorisé à laisser passer des trames DHCP OFFER et DHCP ACK provenant d'un serveur DHCP
- Les ports Untrusted qui recoivent des DHCPOFFER et DHCPACK vont automatiquement drop les paquets
- Construit et maintient la base de données de liaison DHCP Snooping, qui contient les informations sur les hôtes avec des adresses IP louées par un DHCP sur un port trusted
- Utilise la base de données de liaison DHCP Snooping pour valider les requêtes ultérieures des hôtes non fiables

Une base de donnée DHCP snooping sera crée et contiendera, l’adresse MAC de l’hôte, l’adresse IP louée, la durée de location, le type de liaison, le numéro VLAN et les informations d’interface associées à l’hôte.

Cette base de donnée crée par DHCP snooping peut aussi servir pour le service, Dynamic Arp Inspection ( DAI )

Par défaut, DHCP Snooping n'est pas activé

Pour mettre en place dhcp snooping, il faut d’abord activer le services

```bash
switch(config)# ip dhcp snooping
```

Il faut ensuite, activer DHCP snooping pour que les VLAN soient pris en charge. Par défaut, les VLANs ne sont pas pris en charge dans le DHCP snooping, il faut donc spécifier quel vlan participe au processus

```bash
switch(config)# ip dhcp snooping vlan <vlan_id>
```

Il est possible, de mettre en place l’option 82 sans avoir à configurer de relay DHCP,  en autorisant le DHCP snooping à ajouter et supprimer l’information contenu dans l’option 82 du DHCP

```bash
switch(config)# ip dhcp snooping information option
```

## DHCP Snooping trusted & untrusted

Pour que IP DHCP Snooping soit mis en place et utile, il faut spécifier quels ports sont fiables ou non. Par défaut, tous les ports sont marqués comme non-fiable ( untrusted ).

Un port considéré comme fiable par un administrateur est, un port qui est connecté directement ou est un chemin alternatif vers un serveur DHCP de l'infrastructure qui est considéré comme sur car connu et géré localement

Une fois ce port défini, il faut se placer dessus et mettre le port en tant que " trusted "

```bash
switch(config)#  interface fa0/0
switch(config-if)# ip dhcp snooping trust
```

Un port placer en trusted peut véhiculer des messages DHCP offer et acknowledge, des ports untrusted peuvent seulement véhiculer des messages discover et request

## DHCP Limit-Rate

Avec IP DHCP snooping, il est possible de limiter le nombre de trame par secondes, pour réduire l’impacte sur la stabilité du réseau, mais aussi que si le port dépasse le nombre de requête DHCP par seconde, les paquets en excedents seront drop, sur certaines appliances ( ex: Catalyst CBS220 ), le port peut passer en `error-disabled`.

Valeurs recommandés selon les conditions:

- Clients finaux ( PC, imprimantes, etc. → 5 à 10 )
- Equipements réseau → 20 à 50
- Environnement sensible ( data centers ) → 1 à 5
- Ports dans environnements haute densité → 10 à 30

Pour le configurer, il faut se placer sur l’interface

```bash
switch(config)# interface fa0/2
switch(config)# ip dhcp snooping limit rate 10
```

## DAI - Dynamic Arp Inspection

L’inspection dynamique d’ARP est une fonction de sécurité qui valide les paquets ARP dans un réseau.

Il intercepte, enregistre et rejette les paquets ARP ayant des liaisons d’adresses IP à MAC non valides. Cette fonction protège le réseau contre certaines attaques MITM.

L’inspection dynamique d’ARP garantit que seules les requêtes et les réponses d’ARP valides sont relayées. Le commutateur effectue les activités suivantes:

- Il intercepte toutes les requêtes et les réponses ARP sur les ports non sécurisés. 
- Vérifie que chacun de ces paquets interceptés a une liaison d'adresse IP à MAC valide avant de mettre à jour le cache ARP local ou avant de transférer le paquet à la destination appropriée 
- Il abandonne les paquets ARP non valides. 

L’inspection dynamique d’ARP détermine la validité d’un paquet ARP en fonction des liaisons d’adresses IP à MAC valides stockées dans une base de données sécurisée, la base de données de liaison d’espionnage DHCP. 

Cette base de données est créée par DHCP Snooping si ce dernier est activé sur les réseaux VLAN et le commutateur. Si le paquet ARP entre sur une interface sécurisée, le commutateur le transfère sans vérification, Sur les interfaces non sécurisées, le commutateur transmet le paquet uniquement s’il est valide.

Il est possible, dans le cas d’une adresse IP configuré manuellement ou, d’une saisie statique dans la table MAC, de placer un port en trusted pour que, l’inspection DAI ne s’applique pas sur cet interfaces

```bash
switch(config)# interface <interface_id>
switch(config-if)# ip arp inspection trust
```

## ISPG - IP Source Guard

IP Source Guard utilise la base de données de DHCP Snooping pour vérifier que l'adresse IP source d'un paquet correspond à l'adresse IP attribuée au port en utilisant la table de correspondance crée par `ip dhcp snooping`.

L’utilisation de IP source Guard empêche l’usurpation d’adresse IP en vérifiant que le trafic provient bien de l’adresse ip attribué par le serveur DHCP, doit être configuré sur les interfaces considéré comme non fiables ( untrusted )

Si un paquet provient d’une adresse IP non autorisé, il est bloqué

```bash
enable
configure terminal
interface <interface_ID>
 ip verify source <source>
end
```

## Administration

### Maintenance

Vérifier l'état du DHCP snooping, permet de controller :

- DHCP Snooping activé ou non
- VLAN protégés
- Ports de confiance (Trusted)
- Option 82
- Base de données (Database Agent)

```bash
switch# show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
1
DHCP snooping is operational on following VLANs:
1
Smartlog is configured on following VLANs:
none
Smartlog is operational on following VLANs:
none
DHCP snooping is configured on the following L3 Interfaces:

Insertion of option 82 is enabled
   circuit-id default format: vlan-mod-port
   remote-id: 0001.C709.2061 (MAC)
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Verification of giaddr field is enabled
DHCP snooping trust/rate is configured on the following Interfaces:

Interface                  Trusted    Allow option    Rate limit (pps)
-----------------------    -------    ------------    ----------------
FastEthernet0/1            yes        yes             25              
  Custom circuit-ids:
```

Voir les VLANs protégé

```bash
show running-config | include dhcp snooping
ip dhcp snooping vlan 1
ip dhcp snooping
 ip dhcp snooping trust
 ip dhcp snooping limit rate 25
```

Vérifier les interfaces trusted/untrusted

```bash
show running-config interface GigabitEthernet1/0/24
show ip dhcp snooping interface GigabitEthernet1/0/24
```

Vérifier la table des binding, table utilisé par Dynamic ARP inspection et IP source guard

```bash
Switch#show ip dhcp snooping binding 
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  -----------------
00:60:47:BC:6D:14   192.168.1.1      86400       dhcp-snooping  1     FastEthernet0/2
Total number of bindings: 1
```

Vérifier la base de donnéee

```bash
Switch#show ip dhcp snooping database 
Agent URL : flash:/snooping.dat
Write delay Timer : 300 seconds
Abort Timer : 

Agent Running : No
Delay Timer Expiry : Not Running
Abort Timer Expiry : Not Running

Last Succeded Time : None
Last Failed Time : None
Last Failed Reason : No failure recorded.

Total Attempts       :        5   Startup Failures :        0
Successful Transfers :        0   Failed Transfers :        0
Successful Reads     :        0   Failed Reads     :        0
Successful Writes    :        5   Failed Writes    :        0
Media Failures       :        0
```

Vérifier le limit rate

```bash
show ip dhcp snooping limit rate
```

### Dépannage

Commande de débug, augmentation charge CPU

```bash
debug ip dhcp snooping events
debug ip dhcp snooping packets
```

Vérifier les statistiques

```bash
show ip dhcp snooping statistics
ou
show ip dhcp snooping
```
