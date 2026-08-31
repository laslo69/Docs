
Un VLAN est une solution qui permet de diviser un réseau Ethernet physique en plusieurs sous-réseaux logiquement distincts

Malgré leur partage d’une infrastructure physique unique, ces sous-réseaux agissent comme s’ils étaient séparés, offrant une isolation entre eux

Quand du traffic provient d'un VLAN autre que le natif, un tag est ajouté dans la trame pour identifier le VLAN source

Les VLAN peuvent apportés plusieurs avantages

- Réduction de la surcharge CPU : les VLANs réduisent le nombre d’appareils qui reçoivent des trames de diffusion (broadcast), diminuant la charge de travail du CPU de chaque appareil.
- Amélioration de la sécurité : les VLANs limitent la diffusion de trames à un sous-ensemble d’appareils ; moins d’hôtes reçoivent des trames potentiellement nuisibles ou non pertinentes.
- Politiques de sécurité distinctes : vous pouvez appliquer des politiques différentes à chaque VLAN selon les exigences de chaque service.
- Conception plus flexible : regroupez les utilisateurs par département, fonction ou projet plutôt que par emplacement physique.
- Résolution de problèmes plus rapide : un incident est souvent limité au VLAN concerné, ce qui facilite le diagnostic.
- Réduction de la charge pour STP : en limitant un VLAN à un seul switch d’accès, vous réduisez la complexité du Spanning Tree Protocol.
- Crée un nouveau domaine de broadcast et de collision

## Defaut VLAN

Le VLAN par défaut est le VLAN 1 et ne peut pas être changé, contient toutes les interfaces du switch

il est recommandé de changer les interfaces de VLAN pour éviter de, laisser un potentiel accès pour un host non autorisé, quite à créer un " black hole VLAN "

De nombreuses fonctions de gestion (CDP, VTP…) utilisent également le VLAN 1. Pour des raisons de sécurité, il est recommandé de ne pas utiliser le VLAN 1 pour le trafic des utilisateurs

## VLAN Natif

Le VLAN natif est un VLAN ou, le VLAN tagging ne s’applique pas

Par défaut, tout les switchs possèdent un VLAN par défaut qui est aussi le VLAN natif par défaut, ( VLAN ID 1 ), où tout les ports sont en access sur ce VLAN.

Dans le cas ou, le VLAN natif est modifié, il est important que chaque switchs possède le même VLAN natif pour éviter les `VLAN-mismatch` mais surtout, une trame qui passe d’un VLAN natif égale à 10 par exemple, vers un switch où le VLAN natif est le VLAN 1, la trame va être forwarded dans le VLAN 10 même si non natif sur l’autre switch ce qui peut casser le cloisonement par VLAN

Il est très fortement recommandé de créer un nouveau VLAN et de le définir comme nouveau VLAN Natif avec le même numéro d'ID.

Par soucis de sécurité, ce VLAN est une cible d’attaque car le VLAN natif par défaut est connu

### Modifier VLAN natif

Création d’un VLAN et modification du VLAN natif dans le cas d'utilisation avec un trunk

```bash
switch(config)# vlan 99
switch(config)# name NATIF
switch(config)# exit
switch(config)# interface GigabitEthernet 0/2
switch(config)# switchport mode trunk
switch(config)# switchport trunk native vlan 99 
```


## VLAN Tagging

les PC ou autres dispositifs finaux (imprimantes, serveurs) ne sont généralement pas au courant de leur appartenance à un VLAN

Pour eux, ils sont simplement connectés à un réseau, e VLAN est transparent pour ces dispositifs et reste une abstraction au niveau du switch pour organiser et contrôler le trafic.

lorsque deux PC du même VLAN communiquent entre eux et sont connectés au même switch, la trame n’a pas besoin d’être taguée, le switch sait déjà à quel VLAN chaque port appartient.

C’est seulement lorsque la trame doit quitter le switch pour aller vers un autre switch à travers un lien trunk que le tagging devient nécessaire

## Type VLAN

### VLAN Data

Le VLAN Data est un VLAN qui va servir pour faire transiter des données entre clients ( PC, serveurs, imprimantes, etc….) En leur donnant un accès.

Dans le cas d'un VLAN qui transite par un lien trunk, l'utilisation d'un VLAN par un client, va ajouter un tag dans l'en-tête, qui va permettre de définir la source du message

### VLAN Voice

Un VLAN  voice est un VLAN dédié à la transmission de trafic vocal et vidéo.

Un VLAN voice doit fournis une latence inférieur à 150ms, il peut possède un tagging 802.1p pour la priorisation.

Une interface peut être sur un VLAN data mais aussi un VLAN voice, c’est la seule exception de ce type.

Lors de l’utilisation d’un VLAN Voice, le protocole CDP ou LLDP peut permettre de configurer le périphérique client ( souvent un téléphone ), ce qui a pour action d’automatiquement insérer le tag vlan.

Dans le cas ou, CDP / LLDP est désactivé, le vlan tagging doit être configuré manuellement sur le périphérique client, il peut aussi être ignoré sur le switch et être traité de manière normal

### VLAN Management

Les VLANs management sont des VLANs avec une adresse IP attribué ( SVI - Switch Virtual Interface ) pour permettre la configuration du switch à distance via SSH.

Il est recommandé d'utiliser des ACL pour limiter les appareils ayant le droits de se connecter dessus

## Plage VLAN

Il existe différentes plages de VLAN

- VLANs normaux : Portée de 1 à 1001
- VLANs réservés : Portée 1002 à 1005 ( Token ring, FDDI ), ne doivent pas être utilisés, ne peuvent pas être supprimé
- VLANs étendus : 1006 à 4094

## VLAN Trunk

Un trunk est un lien entre deux équipements réseaux et permet de transporter le trafic de plusieurs VLAN sur un seul lien physique

Les paquets transitant par un port trunk, auront leurs trames modifié pour insérer un tag vlan, pour identifier la destination

Lorsque le paquet quitte le VLAN, le VLAN Tagging lui est rajouté, lorsqu’il rejoint le prochain switch avant se destination, l’étiquette est enlevé et la trame inséré dans le bon VLAN

Il est important ( et obligatoire ) de mettre les interfaces qui connectent les switchs entre eux, en mode trunk pour pouvoir autoriser le trafic de trame avec un tag VLAN

## Gestion VLAN

### Création VLAN

Pour créer un VLAN, en configuration globale

A répéter pour le nombre de VLAN voulu

```bash
switch(config)# vlan 10
switch(config-vlan)# name commerce
```

### Renommer VLAN

Il est possible de renommer un VLAN, soit le renommer, soit ajouter un nom si il à été créer sans nom

```bash
switch(config)#vlan 10
switch(config)#name <nouveau nom>
```

### Suppression VLAN

Pour supprimer un VLAN

```bash
switch(config)# no vlan <id_vlan>
```

## Accès VLAN

L'accès au VLAN se fait sur la configuration d'une interface, via la commande `switchport mode access` pour désigner son mode de fonctionne suivi de `switchport access <vlan id>` pour désigner le VLAN d'appartenance.

### VLAN Data

1 port ne peut être membre que d’un seul VLAN

switch(config)# interface GigabitEthernet 0/1
switch(config)# Switchport mode Access
switch(config)# Switchport Access VLAN 10

### VLAN Voice

Un interface peut, seulement dans le cas du VLAN voice, être membre d'un VLAN data ET d'un VLAN voice.

```bash
switch(config)# interface GigabitEthernet 0/1
switch(config)# switchport mode access
switch(config)# switchport access vlan 10
switch(config)# switchport voice vlan 20
```

Il est possible de prioriser le trafic voix en fonction des tags COS ( Class Of Service ) en utilisant un tag 802.1p

```bash
switch(config-if)# switchport vlan voice dot1p
```

## Trunk

Sur un lien trunk, il faut autoriser un ou plusieurs VLAN à pouvoir transmettre des trames d’un VLAN à un autre

```bash
interface GigabitEthernet0/1
switchport encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30
switchport trunk native vlan 99
```

Il y’a plusieurs commandes pour autoriser / refuser / supprimer / ajouter des VLAN sur un lien trunk

`switchport trunk allowed vlan` :

- `add <vlan id>` : Ajoute le VLAN à la liste des VLAN déja autorisé sur le lien trunk
- `all` : autorise tout les vlans ( déconseillé )
- `except <vlan_id>` : autoriser tout les vlans SAUF celui indiqué
- `none` : N’autorise aucun VLAN, supprime aussi tout les vlan autorisé
- `remove <vlan_id>` : Supprime le VLAN indiqué de la liste des vlans autorisé sur le lien

## Administration

Lister les VLANs

```bash
Switch# show vlan 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    
10   modif                            active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0   
1003 tr    101003     1500  -      -      -        -    -        0      0   
1004 fdnet 101004     1500  -      -      -        ieee -        0      0   
1005 trnet 101005     1500  -      -      -        ibm  -        0      0   

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------

Remote SPAN VLANs
------------------------------------------------------------------------------

Primary Secondary Type              Ports
------- --------- ----------------- -----------------------------------------
```

pour avoir un résumé assez bref

```bash
Switch# show vlan brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    
10   modif                            active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active 
```

il est possible d'afficher les informations pour un VLAN précis en fonction de son ID ou de son nom

```bash
Switch# sh vlan id 10

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
10   modif                            active    

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
10   enet  100010     1500  -      -      -        -    -        0      0
```

Fournis le nombre de VLAN existant

```bash
switch# show vlan summary

Number of existing VLANs                :6
    Number of existing VTP VLANs        :6
    Number of existing extended VLANs   :6
```

Voir le type de switchport, peux être appliqué pour toutes les interfaces ou une interface précise

```bash
Switch# show interfaces switchport ! show interfaces switchport Fa0/1
Name: Fa0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: down
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Voice VLAN: none
Administrative private-vlan host-association: none
Administrative private-vlan mapping: none
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk private VLANs: none
Operational private-vlan: none
Trunking VLANs Enabled: All
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL
Protected: false
Appliance trust: none
!
! Truncated, more interface below
```

Affiche l'état d'une interface

```bash
Switch# show interfaces status 
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        connected    trunk      a-full  a-100 10/100BaseTX
Fa1/1                        notconnect   1          auto    auto  10/100BaseTX
Fa2/1                        notconnect   1          auto    auto  10/100BaseTX
Fa3/1                        notconnect   1          auto    auto  10/100BaseTX
Fa4/1                        notconnect   1          auto    auto  10/100BaseTX
Fa5/1                        notconnect   1          auto    auto  10/100BaseTX
```

Consulter la table MAC du VLAN


```bash
switch# show mac address-table vlan 1

Vlan     Mac Address      Type     Ports
----    --------------   -------   -----
  1     aabb.cc00.0200   DYNAMIC   Et0/0
Total Mac Addresses for this creterion: 1
```
