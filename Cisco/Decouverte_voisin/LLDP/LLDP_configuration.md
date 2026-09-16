
![lldp1](Cisco/Decouverte_voisin/LLDP/illustration/lldp1.png)

## SW1

- Activation de LLDP
- Désactiver Transmit/Receive sur les interfaces fa0/2 à fa0/24
- Activation explicite de Transmit/Receive sur fa0/1

```rust
SW1(config)# lldp run
SW1(config)# lldp timer 10
SW1(config)# lldp holdtime 60
SW1(config)# int range fa0/2-24
SW1(config-if-range)# no lldp receive
SW1(config-if-range)# no lldp transmit
SW1(config-if-range)# exit
SW1(config)# int fa0/1
SW1(config-if)# lldp transmit
SW1(config-if)# lldp receive
SW1(config-if)#exit
```

## SW2

- Activation de LLDP
- Désactiver Transmit/Receive sur les interfaces fa0/2 à fa0/24
- Activation explicite de Transmit/Receive sur fa0/1 et fa0/2

```rust
SW2(config)# lldp run
SW2(config)# lldp timer 10
SW2(config)# lldp holdtime 60
SW2(config)# int range fa0/3-24
SW2(config-if-range)# no lldp receive
SW2(config-if-range)# no lldp transmit
SW2(config-if-range)# exit
SW2(config)# int fa0/1
SW2(config-if)# lldp transmit
SW2(config-if)# lldp receive
SW2(config-if)#exit
SW2(config)# int fa0/2
SW2(config-if)# lldp transmit
SW2(config-if)# lldp receive
SW2(config-if)#exit
```

## SW3

- Activation de LLDP
- Désactiver Transmit/Receive sur les interfaces fa0/2 à fa0/24
- Activation explicite de Transmit sur fa0/1
- Désactivation receive sur fa0/1

```rust
SW3(config)# lldp run
SW3(config)# lldp timer 10
SW3(config)# lldp holdtime 60
SW3(config)# int range fa0/2-24
SW3(config-if)# no lldp receive
SW3(config-if)# no lldp transmit
SW3(config-if)# exit
SW3(config)# int fa0/1
SW3(config-if)# lldp transmit
SW3(config-if)# no lldp receive
SW3(config-if)#exit
```
