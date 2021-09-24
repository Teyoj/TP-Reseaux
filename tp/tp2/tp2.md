# TP2 : On va router des trucs

## I. ARP

### 1. Echange ARP

Je fais un ping d'une machine à l'autre : 

    [hugoj@node1 ~]$ ping 10.2.1.12
    PING 10.2.1.12 (10.2.1.12) 56(84) bytes of data.
    64 bytes from 10.2.1.12: icmp_seq=1 ttl=64 time=1.05 ms
    64 bytes from 10.2.1.12: icmp_seq=2 ttl=64 time=1.20 ms
    64 bytes from 10.2.1.12: icmp_seq=3 ttl=64 time=1.19 ms
    64 bytes from 10.2.1.12: icmp_seq=4 ttl=64 time=1.17 ms

Observation des tables ARP des deux machines : 

Table ARP de la machine 1

    [hugoj@node1 ~]$ arp -a
    ? (10.2.1.1) at 0a:00:27:00:00:4b [ether] on enp0s8
    _gateway (10.0.2.2) at 52:54:00:12:35:02 [ether] on enp0s3
    ? (10.2.1.12) at 08:00:27:37:74:47 [ether] on enp0s8

Table ARP de la machine 2

    [hugoj@node2 ~]$ arp -a
    ? (10.2.1.11) at 08:00:27:78:f7:55 [ether] on enp0s8
    ? (10.2.1.1) at 0a:00:27:00:00:4b [ether] on enp0s8
    _gateway (10.0.2.2) at 52:54:00:12:35:02 [ether] on enp0s3
    
En regardant dans la table ARP de node2, on repère l'IP de node1 et on trouve la MAC de node1 qui est `08:00:27:78:f7:55`.
Et dans la table ARP de node1 on fait la même chose et on trouve la MAC de node2 qui est `08:00:27:37:74:47`.

Prouver que l'info est correcte : 

En faisant un `ip a` avec node2 on trouve son adresse MAC qui est `08:00:27:37:74:47`. On compare avec la table ARP de node1 un peu plus haut et on voit que la MAC est la même donc l'information est correcte.


### 2. Analyse de trames

En analysant le fichier tp2_arp.pcap on peut faire ce tableau avec les trames ARP

| ordre | type trame  | source                      | destination                   |
|-------|-------------|-----------------------------|-------------------------------|
| 1     | Requête ARP | `node2` `08:00:27:37:74:47` | Broadcast `FF:FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | `node1` `08:00:27:78:f7:55` | `node2` `08:00:27:37:74:47`   |
| 3     | Requête ARP | `node1` `08:00:27:78:f7:55` | `node2` `08:00:27:37:74:47`   |
| 4     | Réponse ARP | `node2` `08:00:27:37:74:47` | `node1` `08:00:27:78:f7:55`   |

## II. Routage

### 1. Mise en place du routage

Pour ajouter les routes statiques afin que `marcel` et `node1` puissent se ping, on utilise la commande `sudo ip route add 10.2.2.0/24 via 10.2.1.254 dev enp0s3` sur `node1`.

Ensuite sur notre VM marcel on tape la commande `sudo ip route add 10.2.1.0/24 via 10.2.2.254 dev enp0s3`.

On vérifie avec un ping de `node1` vers `marcel` si les routes statiques ont bien été ajoutées : 

```
[hugoj@node1 ~]$ ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=1.10 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=2.00 ms
64 bytes from 10.2.2.12: icmp_seq=3 ttl=63 time=2.10 ms
64 bytes from 10.2.2.12: icmp_seq=4 ttl=63 time=2.25 ms
```

### 2. Analyse de trames

| ordre | type trame  | IP source | MAC source                   | IP destination | MAC destination               |
|-------|-------------|-----------|------------------------------|----------------|-------------------------------|
| 1     | Requête ARP | 10.2.1.11 | `node1` `08:00:27:cb:ff:ce`  | 10.2.1.254     | Broadcast `FF:FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | 10.2.1.254| `routeur` `08:00:27:b5:16:b0`| 10.2.1.11  | `node1` `AA:BB:CC:DD:EE`      |
| 3     | Requête ARP | 10.2.2.254| `routeur` `08:00:27:62:ae:78`| 10.2.2.12  | Broadcast `FF:FF:FF:FF:FF:FF` |
| 4     | Réponse ARP | 10.2.2.12 | `marcel` `08:00:27:e5:a3:b6` | 10.2.2.254 | `routeur` `08:00:27:62:ae:78` |
| 5     | Ping        | 10.2.1.11 | `node1` `08:00:27:cb:ff:ce`  | 10.2.2.12  | `marcel` `08:00:27:e5:a3:b6`  |
| 6     | Pong        | 10.2.2.12 | `marcel` `08:00:27:e5:a3:b6` | 10.2.1.11  | `node1` `08:00:27:cb:ff:ce`   |

### 3. Accès internet

Pour donner un accès internet à nos machines il faut ajouter une route par défaut vers notre routeur.
Pour cela on tape la commande `sudo ip route add default via 10.2.1.254 dev enp0s3`.

On vérifie avec un ping si on a bien un accès internet.
```
[hugoj@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=40.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=21.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=22.4 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=113 time=19.9 ms
```

Après avoir ajouté le serveur DNS on fait un `dig` et un `ping` pour vérifiez que j'ai une résolution de noms qui fonctionne.

```
[hugoj@node1 ~]$ dig ynov.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4009
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               4584    IN      A       92.243.16.143

;; Query time: 4 msec
;; SERVER: 10.33.10.2#53(10.33.10.2)
;; WHEN: Thu Sep 23 12:43:21 CEST 2021
;; MSG SIZE  rcvd: 53

```

```
[hugoj@node1 ~]$ ping google.com
PING google.com (216.58.206.238) 56(84) bytes of data.
64 bytes from par10s34-in-f14.1e100.net (216.58.206.238): icmp_seq=1 ttl=112 time=20.5 ms
64 bytes from par10s34-in-f14.1e100.net (216.58.206.238): icmp_seq=2 ttl=112 time=22.2 ms
64 bytes from par10s34-in-f14.1e100.net (216.58.206.238): icmp_seq=3 ttl=112 time=22.1 ms
64 bytes from par10s34-in-f14.1e100.net (216.58.206.238): icmp_seq=4 ttl=112 time=22.6 ms
```

| ordre | type trame | IP source           | MAC source               | IP destination | MAC destination      |
|-------|------------|---------------------|--------------------------|----------------|----------------------|
| 1     | ping       | `node1` `10.2.1.11` | `node1` `08:00:27:cb:ff:ce`| `8.8.8.8`    |`routeur` `08:00:27:b5:16:b0`|
| 2     | pong       | `8.8.8.8`           | `routeur` `08:00:27:b5:16:b0`|`node1` `10.2.1.11` |`node1` `08:00:27:cb:ff:ce`|

## III. DHCP

### 1. Mise en place du serveur DHCP

Installation du serveur : 
Il faut créer le fichier `/etc/dhcp/dhcpd.conf`.
Ensuite dans ce fichier on configure notre serveur dhcp : 
```
default-lease-time 900;
max-lease-time 10800;
authoritative;
subnet 10.2.1.0 netmask 255.255.255.0 {
    range dynamic-bootp 10.2.1.20 10.2.1.200;
    option broadcast-address 10.2.1.255;
}
```

Récupérer une IP en DHCP avec node2 : 
`[hugoj@node2 ~]$ sudo nmcli connection modify enp0s8 ipv4.method auto`
`[hugoj@node2 ~]$ sudo nmcli connection down enp0s8; sudo nmcli connection up enp0s8`


**Améliorer la configuration du DHCP**

Pour que notre serveur DHCP ait une route par défaut et un serveur DNS à utiliser il faut ajouter ces lignes dans notre fichier `/etc/dhcp/dhcpd.conf`

```
option routers 10.2.1.254;
option domain-name-servers 1.1.1.1;
```

Avec `ip a` on voit que `node2` a récupéré une IP qui est `10.2.1.20`
```
[hugoj@node2 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:58:cf:07 brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.20/24 brd 10.2.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 897sec preferred_lft 897sec
    inet6 fe80::a00:27ff:fe58:cf07/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

On test avec `ping 10.2.1.254` si il peut ping la passerelle.
```
[hugoj@node2 ~]$ ping 10.2.1.254
PING 10.2.1.254 (10.2.1.254) 56(84) bytes of data.
64 bytes from 10.2.1.254: icmp_seq=1 ttl=64 time=0.907 ms
64 bytes from 10.2.1.254: icmp_seq=2 ttl=64 time=1.15 ms
64 bytes from 10.2.1.254: icmp_seq=3 ttl=64 time=1.22 ms
```

Avec la commande `ip route show` on peut voir qu'il a bien une route par défaut.
```
[hugoj@node2 ~]$ ip route show
default via 10.2.1.254 dev enp0s8 proto dhcp metric 100
10.2.1.0/24 dev enp0s8 proto kernel scope link src 10.2.1.20 metric 100
```

```
[hugoj@node2 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=21.9 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=23.2 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=21.3 ms
```

La route fonctionne bien.

On vérifie avec `dig ynov.com` que la résolution de noms fonctionne.
```
[hugoj@node2 ~]$ dig ynov.com

; <<>> DiG 9.11.26-RedHat-9.11.26-4.el8_4 <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54975
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 85d8f76d0c7e900751554d64614e090c93bd08c9e4c1e2b7 (good)
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               10800   IN      A       92.243.16.143

;; Query time: 44 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Fri Sep 24 19:21:14 CEST 2021
;; MSG SIZE  rcvd: 81
```

Et on a vu avec le `ping 8.8.8.8` que ça fonctionne aussi.


### 2.Analyse de trames


| ordre | type trame | IP source           | MAC source               | IP destination | MAC destination      |
|-------|------------|---------------------|--------------------------|----------------|----------------------|
| 1     | Requête ARP| `node2` `10.2.1.20` | `node2` `08:00:27:58:cf:07`|`node1` `10.2.1.11`|Broadcast `ff:ff:ff:ff:ff:ff`|
| 2     | Réponse ARP| `node1` `10.2.1.11` | `node1` `08:00:27:d4:09:bb`|`node2` `10.2.1.20`|`node2` `08:00:27:58:cf:07`|
| 3     | Discover   | `0.0.0.0`           | `node2` `08:00:27:58:cf:07`|`255.255.255.255`|Broadcast `ff:ff:ff:ff:ff:ff`|
| 4     | Requête ARP| `node1` `10.2.1.11` | `node1` `08:00:27:cb:ff:ce`|`10.2.1.21`  |Broadcast `ff:ff:ff:ff:ff:ff`|
| 5     | Offer      | `node1` `10.2.1.11` | `node1` `08:00:27:cb:ff:ce`|`10.2.1.21`  |`node2` `08:00:27:58:cf:07`|
| 6     | Request    | `0.0.0.0`           | `node2` `08:00:27:58:cf:07`|`255.255.255.255`|Broadcast `ff:ff:ff:ff:ff:ff`|
| 7     | Acknowledge| `node1` `10.2.1.11` | `node1` `08:00:27:cb:ff:ce`|`node2` `10.2.1.21`|`node2` `08:00:27:58:cf:07`|

