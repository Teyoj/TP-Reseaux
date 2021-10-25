# TP4 : Vers un réseau d'entreprise

## I. Dumb switch

### 1. Topologie 1

![](https://i.imgur.com/4xVkkmC.png)

### 2. Adressage topologie 1

Je change les IP des deux VPCS pour que le PC1 ait l'IP `10.1.1.1/24` et le PC2 ait l'IP `10.1.1.2/24`.

```
PC1> ip 10.1.1.1/24
Checking for duplicate address...
PC1 : 10.1.1.1 255.255.255.0
```

```
PC2> ip 10.1.1.2/24
Checking for duplicate address...
PC2 : 10.1.1.2 255.255.255.0
### 2. Adressage topologie 2
```

Ensuite j'effectue un ping de pc1 vers pc2.
```
PC1> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=26.079 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=38.987 ms
```

## II. VLAN

### 1. Topologie 2

![](https://i.imgur.com/BSXHcZG.png)

### 2. Adressage topologie 2

Je change l'IP de mon pc3 en `10.1.1.3/24`.
Je vérifie que tout le monde peut se ping.
```
PC3> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=34.813 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=28.600 ms
84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=38.749 ms
84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=30.007 ms
84 bytes from 10.1.1.2 icmp_seq=5 ttl=64 time=37.170 ms


PC3> ping 10.1.1.1

84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=26.640 ms
84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=35.535 ms
84 bytes from 10.1.1.1 icmp_seq=3 ttl=64 time=31.651 ms
```

**Configuration des VLANs**

Je définis d'abord les vlans à utiliser.
```
Switch>enable
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#vlan 10
Switch(config-vlan)#name vlan10
Switch(config-vlan)#exit
Switch(config)#vlan 20
Switch(config-vlan)#name vlan20
Switch(config-vlan)#exit
```

Ensuite j'attribue à chaque port du switch un vlan à utiliser.
```
Switch(config)#interface GigabitEthernet 0/0
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config)#interface GigabitEthernet 0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config)#interface GigabitEthernet 0/2
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 20
Switch(config-if)#no shutdown
Switch(config-if)#exit
```

Je vérifie ensuite que pc1 et pc2 peuvent se ping et que pc3 ne peut plus ping personne.
```
PC1> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=33.971 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=15.274 ms
^C
PC1> ping 10.1.1.3

host (10.1.1.3) not reachable
```


## III. Routing

### 1. Topologie 3

![](https://i.imgur.com/xNVx5hp.png)

**Setup**

Je commence par définir les IPs statiques sur mes machines.
`PC1` est en `10.1.1.1/24`
`PC2` est en `10.1.1.2/24`
`adm1` est en `10.2.2.1/24`
`web1` est en `10.3.3.1/24`

Ensuite je déclare les VLANs sur le switch.

```
Switch>enable
Switch#conf t
Switch(config)#vlan 11
Switch(config-vlan)#name vlan_11
Switch(config-vlan)#exit
Switch(config)#vlan 12
Switch(config-vlan)#name vlan_12
Switch(config-vlan)#exit
Switch(config)#vlan 13
Switch(config-vlan)#name vlan_13
Switch(config-vlan)#exit
```

Après ça j'ajoute les ports du switch dans les bons VLANs, c'est à dire `PC1` et `PC2` dans le VLAN 11, `adm1` dans le VLAN 12 et `web1` dans le VLAN 13.

```
Switch(config)#interface GigabitEthernet 0/0
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 11
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config)#interface GigabitEthernet 0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 11
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config-if)#interface GigabitEthernet 0/2
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 12
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config)#interface GigabitEthernet 0/3
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 13
Switch(config-if)#no shutdown
Switch(config-if)#exit
```

Ensuite j'ajoute le port qui pointe vers le routeur (le GigabitEthernet 1/0) et je le met en mode trunk.

```
Switch(config)#interface GigabitEthernet 1/0
Switch(config-if)#switchport trunk encapsulation dot1q
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk allowed vlan add 11,12,13
Switch(config-if)#no shutdown
Switch(config-if)#exit
```

Au niveau du routeur j'ajoute plusieurs IPs sur l'interface qui pointe vers le switch et j'indique à chaque "sous-interface" à quel VLAN elle appartient.
```
R1(config)#interface fastEthernet 0/0.10
R1(config-subif)#encapsulation dot1q 11
R1(config-subif)#ip addr 10.1.1.254 255.255.255.0
R1(config-subif)#exit
R1(config)#interface fastEthernet 0/0.20
R1(config-subif)#encapsulation dot1q 12
R1(config-subif)#ip addr 10.2.2.254 255.255.255.0
R1(config-subif)#exit
R1(config)#interface fastEthernet 0/0.30
R1(config-subif)#encapsulation dot1q 13
R1(config-subif)#ip addr 10.3.3.254 255.255.255.0
R1(config-subif)#exit
```

**Vérif**

Je vérifie que les machines peuvent ping leur passerelle.
```
PC1> ping 10.1.1.254

10.1.1.254 icmp_seq=1 timeout
84 bytes from 10.1.1.254 icmp_seq=2 ttl=255 time=11.355 ms
84 bytes from 10.1.1.254 icmp_seq=3 ttl=255 time=11.283 ms
84 bytes from 10.1.1.254 icmp_seq=4 ttl=255 time=6.661 m
```

```
PC2> ping 10.1.1.254

84 bytes from 10.1.1.254 icmp_seq=1 ttl=255 time=18.370 ms
84 bytes from 10.1.1.254 icmp_seq=2 ttl=255 time=20.121 ms
84 bytes from 10.1.1.254 icmp_seq=3 ttl=255 time=6.012 ms
```

```
adm1> ping 10.2.2.254

84 bytes from 10.2.2.254 icmp_seq=1 ttl=255 time=20.116 ms
84 bytes from 10.2.2.254 icmp_seq=2 ttl=255 time=12.523 ms
84 bytes from 10.2.2.254 icmp_seq=3 ttl=255 time=7.052 ms
```


## IV. NAT

### 1. Topologie 4

![](https://i.imgur.com/KyhFETj.png)

**Setup**

Je configure l'interface du routeur qui est branchée au cloud et je la mets en DHCP
```
R1(config)#interface fastEthernet 1/0
R1(config-if)#ip address dhcp
R1(config-if)#no shutdown
R1(config-if)#exit
```

Je vérifie ensuite avec `show ip int br` que mon interface est bien configurée en DHCP.
```
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up
FastEthernet0/0.10         10.1.1.254      YES manual up                    up
FastEthernet0/0.20         10.2.2.254      YES manual up                    up
FastEthernet0/0.30         10.3.3.254      YES manual up                    up
FastEthernet1/0            10.0.3.16       YES DHCP   up                    up
FastEthernet2/0            unassigned      YES unset  administratively down down
FastEthernet3/0            unassigned      YES unset  administratively down down
```

Je ping ensuite `1.1.1.1`
```
R1#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 48/61/68 ms
```

Configuration du NAT : 
```
R1(config)#interface fastEthernet 1/0
R1(config-if)#ip nat outside
R1(config-if)#exit
R1(config)#interface fastEthernet 0/0
R1(config-if)#ip nat inside
R1(config-if)#exit
R1(config)#access-list 1 permit any
R1(config)#ip nat inside source list 1 interface fastEthernet 1/0 overload
```


## V. Add a building

### 1. Topologie 5

![](https://i.imgur.com/c3rMG1C.png)
