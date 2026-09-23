
![etherchannel1](Cisco/Etherchannel/illustration/etherchannel1.png)

## Configuration L2

### PC0

```bash
ip 192.168.10.10/24
vlan 10
```

### PC1

```bash
ip 192.168.10.11/24
vlan 10
```

### SW1

- Création des VLANs 10,20
- Attribution accès VLAN
- Passage interfaces en trunk
- Autoriser VLAN sur le trunk
- Déclaration VLAN natif
- Création du lien Etherchannel 1
- Déclaration mode négociation Etherchannel
- Autorisation des VLANs sur port-channel
- Déclaration VLAN natif sur port-channel

```bash
en
conf t
hostname SW1
vlan 10
vlan 20
exit
int fa0/4
switchport mode access
switchport access vlan 10
int range fa0/1-3
switchport mode trunk
switchport trunk allowed vlan 1,10,20
switchport trunk native vlan 1
channel-protocol pagp
channel-group 1 mode desirable
exit
int po1
switchport mode trunk
switchport trunk allowed vlan 1,10,20
switchport trunk native vlan 1
end
wr
```

### SW2

- Création des VLANs 10,20
- Attribution accès VLAN
- Passage interfaces en trunk
- Autoriser VLAN sur le trunk
- Déclaration VLAN natif
- Création du lien Etherchannel 1
- Déclaration mode négociation Etherchannel
- Autorisation des VLANs sur port-channel
- Déclaration VLAN natif sur port-channel

```bash
en
conf t
hostname SW2
vlan 10
vlan 20
exit
switchport mode access
switchport access vlan 10
int range fa0/1-3
switchport mode trunk
switchport trunk allowed vlan 1,10,20
switchport trunk native vlan 1
channel-protocol pagp
channel-group 1 mode desirable
exit
int po1
switchport mode trunk
switchport trunk allowed vlan 1,10,20
switchport trunk native vlan 1
end
wr
```

## Configuration L3

![etherchannel2](Cisco/Etherchannel/illustration/etherchannel1.png)

### PC0

```bash
ip 192.168.10.10/24
gateway 192.168.10.1
```

### PC1

```bash
ip 192.168.20.10/24
gateway 192.168.20.1
```

### SW1

- interface fa0/4
	- Désactivation capacité switching
	- Adressage IP
- Interface fa0/1-3
	- Désactivation capacité switching
	- Création port-channel 1
	- Désignation protocole négociation
- Adressage IP port-channel 1
- Désignation mode load-balancing
- Activation routage
- Création route pour le réseau distant

```bash
int fa0/4
no switchport
ip address 192.168.10.1 255.255.255.0
int range fa0/1-3
no switchport
channel-protocole pagp
channel-group 1 mode desirable
int po1
ip address 192.168.0.1 255.255.255.0
exit
port-channel load-balance src-dst-ip
ip routing
ip route 192.168.20.0 255.255.255.0 192.168.0.2
exit
wr
```

### SW2

- interface fa0/4
	- Désactivation capacité switching
	- Adressage IP
- Interface fa0/1-3
	- Désactivation capacité switching
	- Création port-channel 1
	- Désignation protocole négociation
- Adressage IP port-channel 1
- Désignation mode load-balancing
- Activation routage
- Création route pour le réseau distant

```bash
int fa0/4
no switchport
ip address 192.168.20.1 255.255.255.0
int range fa0/1-3
no switchport
channel-protocole pagp
channel-group 1 mode desirable
int po1
ip address 192.168.0.2 255.255.255.0
exit
ip routing
port-channel load-balance src-dst-ip
ip route 192.168.10.0 255.255.255.0 192.168.0.1
end
wr
```
