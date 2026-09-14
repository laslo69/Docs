
## Configuration

```bash
Switch-L3
vlan 10
interface fa0/1
switchport mode trunk
switchport trunk allowed vlan 1,10
switchport trunk native vlan 1
switchport port-security maximum 4
switchport port-security violation shutdown
switchport port-security mac-address sticky
switchport port-security
```

```bash
Switch-L2
vlan 10
interface fa0/1
switchport trunk allowed vlan 1,10
switchport trunk native vlan 1
interface fa0/2
switchport mode access
switchport access vlan 10
interface fa0/3
switchport mode access
switchport access vlan 10
```

