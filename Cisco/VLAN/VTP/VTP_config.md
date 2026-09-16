
## Configuration

### Configration VTPv2

#### SW1 - VTP Master

```rust
vlan 10
vlan 20
vlan 30
vlan 99
interface e0/0
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
switchport trunk native vlan 99
exit
vtp version 2
vtp domain lab.lan
vtp mode server
vtp password lab hidden
vtp pruning
```

#### SW2 - VTP Slave

```rust
interface e0/0
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
switchport trunk native vlan 99
exit
vtp version 2
vtp domain lab.lan
vtp mode client
vtp password lab hidden
```

### Configuration VTPv3

#### SW1 - VTP Server Primary

```rust
vlan 10
vlan 20
vlan 30
vlan 99
interface e0/0
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
switchport trunk native vlan 99
exit
router(config)# vtp domain
router(config)# vtp version 3
router(config)# vtp password lab hidden
router(config)# vtp mode server
router(config)# vtp pruning
router# vtp primary
```

#### SW2 - VTP Server secondary

```rust
interface e0/0
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99
switchport trunk native vlan 99
exit
router(config)# vtp domain
router(config)# vtp version 3
router(config)# vtp password lab hidden
router(config)# vtp mode server
```
