
Lors de la réception d'une trame DHCP par le relais DHCP, il va contrôler que le paquet reçu ne possède pas une valeur nulle pour le champ giaddr, dans le cas contraire, le paquet est drop.

Une fois le contrôle effectué, le relais DHCP va ajouter l'option 82, l'adresse source de la trame sera modifié par l'ajout de l'adresse du relais DHCP, ce qui permet de transmettre les trames DHCP, en unicast mais aussi de pouvoir traverser des routeurs qui bloque les broadcast ( dont les requêtes DHCP ).

Le relais DHCP va créer un circuit-id pour permettre de définir dans l'option 82, ses sous-options, de quel port provient la trame localement pour permettre d'identifié l'origine de la demande et, ou transmettre la réponse du serveur DHCP

Les messages DHCP (comme DHCPDISCOVER et DHCPREQUEST) sont des diffusions (broadcasts) et ne traversent pas les routeurs par défaut, car les routeurs ne relayent pas les broadcasts entre sous-réseaux.

Les messages DHCP OFFER et ACKNOWLEDGE sont retransmis par le relais DHCP via des "circuit-id".

L’option 82 utilise:

- Circuit ID : Identifie le circuit ( interface ou vlan ) sur lequel la demande à été reçu. L’ID de circuit contient le nom de l’interace et le vlan séparé par deux points
- Remote ID : Identifie le host distant
- Vendor ID : Identifie le fournisseur de l’hôte

Par défaut, ip helper-address relaie les broadcasts pour 8 services UDP :

- DHCP/BOOTP (port 67) 
- TFTP (port 69) 
- DNS (port 53) 
- Time (port 37) 
- TACACS (port 49) 
- NetBIOS Name Service (port 137) 
- NetBIOS Datagram Service (port 138)

Il est possible d'ajouter des protocoles à autoriser dans le forward via DHCP relay, via la commande `ip forward-protocol udp <port>`

Structure de l'option 82 est une sorte de "conteneur" constitué de plusieurs sous-options, ou `i` représente une sous-option, conformément à la RFC 3046:

```rust
 Code   Len    Agent  Information-Field
+------+------+------+------+------+------+--...-+------+
|  82  |   N  |  i1  |  i2  |  i3  |  i4  |      |  iN  |
+------+------+------+------+------+------+--...-+------+
```

Le champ Agent-information est constitué de tuples en sous-options, ou "S" et "I" représente les valeurs pour chaque sous-options, les constructeurs peuvent potentiellement ajouter des octets mais la logique reste la même:

```rust
DHCP Agent          Sub-Option Description

Sub-option Code
+-------------------------------------------------+
| 1                   Agent Circuit ID Sub-option |
| 2                   Agent Remote ID Sub-option  |
+-------------------------------------------------+

SubOpt  Len     Sub-option Value
+------+------+------+------+------+------+------+------+
|  1   |   N  |  s1  |  s2  |  s3  |  s4  |      |  sN  |
+------+------+------+------+------+------+------+------+
 SubOpt  Len     Sub-option Value
+------+------+------+------+------+------+--...-+------+
|  2   |   N  |  i1  |  i2  |  i3  |  i4  |      |  iN  |
+------+------+------+------+------+------+--...-+------+
```

En signalant dans la configuration, une option de relais DHCP, les messages DHCP sont converti en unicast pour les envoyés vers l’adresse IP désigné dans le ip helper-address ( option 82 : relay agent information ) est ajouté pour indiquer le sous réseau d’origine du client.

L’adresse IP multicast ( 255.255.255.255 ) est remplacé par l’adresse IP désigné dans l’ip helper-address, permettant au paquet de ne pas être bloqué si il doit traverser un routeur et l’adresse ip source devient celle du relais DHCP

## Circuit-ID && Format

### Circuit-ID

L'agent-relais utilise cette sous-option pour savoir sur quel circuit renvoyer les réponses du serveur DHCP (ex: OFFER, ACK, NAK).

Si un client envoie un DHCP DISCOVER depuis le port 5 d'un switch, l'agent-relais ajoute le Circuit ID correspondant (ex: Port-5) dans le paquet. Quand le serveur DHCP répond, l'agent-relais utilise ce Circuit ID pour envoyer la réponse uniquement sur le port 5, évitant ainsi une diffusion inutiles sur tous les ports.

Un serveur DHCP peut utiliser le Circuit ID pour :

- Réserver des plages d'IP par circuit (ex: les clients du port 1 obtiennent des IP dans 192.168.1.0/24, ceux du port 2 dans 192.168.2.0/24).
- Appliquer des règles de sécurité (ex: limiter le nombre d'IP par circuit pour éviter l'épuisement des adresses).
- Associer des paramètres spécifiques (ex: routeur par défaut, DNS) en fonction du circuit.

|Type de circuit|Exemple de valeur (s1, s2...)|Description|
|:-:|:-:|:-:|
|Numéro d'interface routeur|0x00 0x05|Interface eth0/5 du routeur|
|Port d'un switch/Hub|0x00 0x03|Port 3 d'un switch Ethernet|
|Port d'un RAS (Remote Access Server)|0x00 0x0A|Port 10 d'un serveur d'accès distant|
|DLCI (Frame Relay)|0x00 0x2B|DLCI 43 en hexadécimal|
|VPI/VCI (ATM)|0x00 0x01 0x00 0x32|Circuit virtuel ATM 1.50|
|Circuit ID (Cable Data)|0x43 0x61 0x62 0x6C 0x65 0x31|Chaîne ASCII "Cable1"|

### Format Circuit-ID

Les Sub-Options suivent le format

- SubOpt = 1 : Code de la sous-option Circuit ID
- Len = n : Longueur (en octets) de la valeur du Circuit ID (ex: n=2 pour un numéro de port sur 2 octets)
- s1, s2... : Valeur brute du Circuit ID (ex: 0x00 0x05 pour le port 5)

```rust
SubOpt   Len     Circuit ID
+------+------+------+------+------+------+------+------+--
|  1   |   n  |  s1  |  s2  |  s3  |  s4  |  s5  |  s6  | ...
+------+------+------+------+------+------+------+------+--
```

Exemple de Circuit-ID

```rust
Circuit ID (Suboption 1)

Format : [1][Length][Data]
Exemple : [1][5]["eth0/1"] (5 octets pour l'identifiant du port en ASCII).
```

## Remote-ID && Format

### Remote-ID

Le Remote-ID est une option non-obligatoire qui peut être utilisé pour, identifié l'hôte distant pour des cas particuliers .

Le champ Remote-ID peut être:

- a "caller ID" telephone number for dial-up connection
- a "user name" prompted for by a Remote Access Server
- a remote caller ATM address
- a "modem ID" of a cable data modem
- the remote IP address of a point-to-point link
- a remote X.25 address for X.25 connections

### Format Remote-ID

La structure est sensiblement la même que le "circuit-id", la valeur SubOpt est égale à 2, et R1,Rn correspond à la valeur du remote-id

```rust
 SubOpt  Len     Agent Remote ID
+------+------+------+------+------+------+------+------+----+
|  2   |   n  |  r1  |  r2  |  r3  |  r4  |  r5  |  r6  | rN |
+------+------+------+------+------+------+------+------+----+
```

Exemple de Circuit-ID

```rust
Remote ID (Suboption 2)

Format : [2][Length][Data]
Exemple : [2][6][00:11:22:33:44:55] (6 octets pour une MAC).
```

## Configurer Relais DHCP

Par défaut, le relay DHCP est activé, si ce n’est pas le cas, en configuration globale :

```bash
switch(config)# ip dhcp relay
```

Ensuite, il faut définir l'adresse IP du ou des serveurs DHCP via la commande " ip helper-address ". L'interface peut aussi être un VLAN

```bash
interface GigabitEthernet0/1
ip address 192.168.1.1 255.255.255.0
ip dhcp relay information option
ip helper-address 10.0.0.2  # Adresse du serveur DHCP
```

## Forcer réponse en unicast

Il est possible, de forcer la réponse en unicast en désactivant la réponse en broadcast, quand un mesage du processus DORA passe par un relais DHCP.

L’intéret de forcer la réponse en unicast serait dans un premier temps de réduire le trafic réseau en supprimant des broadcast inutile.

Dans un second temps, la prévention des boucles de diffusion, dans le cas ou plusieurs relais dhcp sont configuré dans le même sous réseau

Permet aussi de réduire la visibilité des réponses DHCP aux seuls client concernés, réduisant les risque de sniffing

```bash
switch(config)# interface gigabitEthernet 0/0
switch(config-if)# ip helper-address 192.168.1.100
switch(config-if)# no ip dhcp relay subnet-broadcast
```

## Administration

### Maintenance

Voir la configuration du DHCP relay sur une interface

```bash
show running-config interface GigabitEthernet0/1
!
interface GigabitEthernet0/1
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 10.1.1.20
```

la commande `show running-config | section interface` donne un rendu similaire mais plus généraliste

Voir tout les DHCP relay configuré

```bash
show running-config | include helper
```

Voir les statistiques du service relay dhcp

```bash
show ip dhcp relay information
show ip dhcp server statistics
```

Voir la configuration complète de l'interface, une ligne stipule si un DHCP relay est configuré ou non

```bash
show ip interface GigabitEthernet0/1
```

Vérifier le routage avec:

- ping
- traceroute

### Sécurité

Désactiver les protocoles UDP forwarded qui sont inutiles

```bash
no ip forward-protocol udp tftp
no ip forward-protocol udp netbios-ns
no ip forward-protocol udp netbios-dgm
```

Utiliser DHCP snooping sur les switchs pour autoriser certains switch à transmettre des messages DHCP BOOTPREPLAY ( type DHCP offer et DHCP acknowledge )

Mettre une limite du nombre de requête par secondes sur les ports utilisateurs

### Dépannage

Voir les évenements DHCP ( augmentation de la charge CPU )

```bash
debug ip dhcp server events
debug ip dhcp server packet
```

Vérifier les ACLs, que les ports ou le protocole DHCP soit autorisé

```bash
show access-lists
```

Voir ou les ACLs sont appliquées

```bash
show ip interface GigabitEthernet0/1
```

Vérifier les ports relayé

```bash
show running-config | include forward-protocol
```

Doit contenir:

- ip forward-protocol udp bootps
- ip forward-protocol udp bootpc