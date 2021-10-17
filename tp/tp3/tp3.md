# TP3 : Progressons vers le réseau d'infrastructure

## I. (mini)Architecture réseau

| Nom du réseau | Adresse du réseau | Masque        | Nombre de clients possibles | Adresse passerelle | Adresse broadcast   |
|---------------|-------------------|---------------|-----------------------------|--------------------|---------------------|
| `server1`     | `10.3.1.0`        |`255.255.255.128` |    126                   | `10.3.1.126`       | `10.3.1.127`        |
| `client1`     | `10.3.1.128`      |`255.255.255.192` |   62                     | `10.3.1.190`       | `10.3.1.191`        |
| `server2`     | `10.3.1.192`      |`255.255.255.240` | 14                       | `10.3.1.206`       | `10.3.1.207`        |

Avec la commande `ip a` je prouve que mon routeur a bien une IP dans les 3 réseaux et que c'est l'IP de la passeerelle : 
```
[hugoj@routeur ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:67:40:6b brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85869sec preferred_lft 85869sec
    inet6 fe80::a00:27ff:fe67:406b/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:1b:4e:47 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.126/25 brd 10.3.1.127 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe1b:4e47/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e2:c7:3c brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.190/26 brd 10.3.1.191 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fee2:c73c/64 scope link
       valid_lft forever preferred_lft forever
5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a0:34:9c brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.206/28 brd 10.3.1.207 scope global noprefixroute enp0s10
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea0:349c/64 scope link
       valid_lft forever preferred_lft forever
```

Ensuite avec un `ping 8.8.8.8`, je prouve qu'il a un accès internet et avec un `dig ynov.com` je prouve qu'il a de la résolution de noms.

```
[hugoj@routeur ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=20.10 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=19.9 ms
```

```
[hugoj@routeur ~]$ dig ynov.com
 ANSWER SECTION:
ynov.com.               7594    IN      A       92.243.16.143

;; Query time: 5 msec
;; SERVER: 10.33.10.2#53(10.33.10.2)
;; WHEN: Mon Sep 27 12:32:31 CEST 2021
;; MSG SIZE  rcvd: 53
```

Je prouve avec `hostname` qu'il porte le bon nom.
```
[hugoj@routeur ~]$ hostname
routeur.tp3
```

Activation du routage : 
```
[hugoj@routeur ~]$ sudo firewall-cmd --list-all
[sudo] password for hugoj:
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s10 enp0s3 enp0s8 enp0s9
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[hugoj@routeur ~]$ sudo firewall-cmd --get-active-zone
public
  interfaces: enp0s10 enp0s3 enp0s8 enp0s9
[hugoj@routeur ~]$ sudo firewall-cmd --add-masquerade --zone=public
success
[hugoj@routeur ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```

| Nom machine         | Adresse IP `client1`     | Adresse IP `server1` | Adresse IP `server2` | Adresse de passerelle |
|---------------------|--------------------------|----------------------|----------------------|-----------------------|
| `router.tp3`        | `10.3.1.190/26`           | `10.3.1.126/25`      | `10.3.1.206/28`      | Carte NAT             |
| `dhcp.client1.tp3`  | `10.3.1.130/26`           | X                    | X                    | `10.3.1.190/26`        |
| `marcel.client1.tp3`| `10.3.1.129/26` (dynamique)| X                    | X                    | `10.3.1.190/26`        |
| `dns1.server1.tp3`  | X                        | `10.3.1.10/25`       | X                    | `10.3.1.126/25`       |
|                     |                      |                      |                      |                       |
|                     |                      |                      |                      |                       |



## II. Services d'infra

### 1. Serveur DHCP

Je mets en place une machine dans le réseau `client1`.
Elle porte le nom `dhcp.client1.tp3`.
```
[hugoj@dhcp ~]$ hostname
dhcp.client1.tp3
```

*Voir fichier `dhcp.conf`*

**Mise en place d'un client dans le réseau** `client1` : 

En faisant la commande `sudo systemctl status dhcp` sur la machine `dhcp.client1.tp3`, on peut voir que notre client récupère une adresse IP dynamiquement.
```
Sep 30 11:35:02 dhcp.client1.tp3 dhcpd[1481]: DHCPDISCOVER from 08:00:27:b7:31:24 via enp0s8
Sep 30 11:35:03 dhcp.client1.tp3 dhcpd[1481]: DHCPOFFER on 10.3.1.129 to 08:00:27:b7:31:24 (marcel) via enp0s8
Sep 30 11:35:03 dhcp.client1.tp3 dhcpd[1481]: DHCPREQUEST for 10.3.1.129 (10.3.1.130) from 08:00:27:b7:31:24 (marcel) via enp0s8
Sep 30 11:35:03 dhcp.client1.tp3 dhcpd[1481]: DHCPACK on 10.3.1.131 to 08:00:27:b7:31:24 (marcel) via enp0s8
```

Je prouve maintenant que le client à un accès internet, et une résolution de noms.

```
[hugoj@marcel ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=41.1 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=36.2 ms
```
```
[hugoj@marcel ~]$ dig ynov.com
ANSWER SECTION:
ynov.com.               8850    IN      A       92.243.16.143
```

Avec la commande `traceroute`, je prouve que `marcel.client1.tp3` passe par `routeur.tp3` pour sortir de son réseau : 
```
[hugoj@marcel ~]$ traceroute google.com
traceroute to google.com (142.250.75.238), 30 hops max, 60 byte packets
 1  _gateway (10.3.1.190)  0.538 ms  0.510 ms  0.497 ms
 2  10.0.2.2 (10.0.2.2)  0.487 ms  0.425 ms  0.406 ms
```

On voit bien qu'il passe par une gateway qui a la même IP que `routeur.tp3` donc il passe par le routeur.

### 2. Serveur DNS

Dans le fichier `/etc/resolv.conf` j'ajoute un serveur DNS connu.
```
# Generated by NetworkManager
search auvence.co
nameserver 10.33.10.2
nameserver 10.33.10.148
nameserver 10.33.10.155
nameserver 1.1.1.1
```
J'ai ajouté le serveur DNS 1.1.1.1

Dans le fichier `/etc/named.conf` j'ajoute les lignes
```
zone "server1.tp3" IN {
        type master;
        file "server1.tp3.forward";
        allow-update { none; };
};

zone "server2.tp3" IN {
        type master;
        file "server2.tp3.forward";
        allow-update { none; };
};
```

Et je crée les fichiers `/etc/named/server1.tp3.forward` et `/etc/named/server2.tp3.forward`.
Dans le premier fichier de conf j'ajoute les lignes
```
$TTL 86400
@   IN  SOA     dns1.server1.tp3. root.server1.tp3. (
         2021080804  ;Serial
         3600        ;Refresh
         1800        ;Retry
         604800      ;Expire
         86400       ;Minimum TTL
)
        ; Set your Name Servers here
@         IN  NS      dns1.server1.tp3.

; Set each IP address of a hostname. Sample A records.
dns1           IN  A       10.3.1.10
routeur        IN  A       10.3.1.126
```

Et je fais la même chose dans l'autre fichier de conf en faisant quelques modification.
```
$TTL 86400
@   IN  SOA     dns1.server1.tp3. root.server1.tp3. (
         2021080805  ;Serial
         3600        ;Refresh
         1800        ;Retry
         604800      ;Expire
         86400       ;Minimum TTL
)
        ; Set your Name Servers here
@         IN  NS      dns1.server2.tp3.

; Set each IP address of a hostname. Sample A records.
dns1           IN  A       10.3.1.10
routeur        IN  A       10.3.1.206
```




## III. Services métier

### 1. Serveur Web

Je crée une nouvelle machine `web1.server2.tp3` et j'installe NGINX avec `sudo dnf -y install nginx`.
Ensuite j'ouvre le port 443.
```
[hugoj@web1 ~]$ sudo firewall-cmd --add-port=443/tcp --permanent
success
```

Je test un curl sur `marcel.client1.tp3`.
```
[hugoj@marcel /]$ curl 10.3.1.200:443
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
```

### 2. Partage de fichiers

**Setup**

Je crée ma machine `nfs1.server2.tp3` et j'installe nfs-utils.
Ensuite dans le fichier `/etc/idmapd.conf` je modifie la ligne `#Domain =` en `Domain = nfs1.server2`.
Après ça, je crée le fichier `/etc/exports` et dans celui-ci je mets : 
```
/srv/nfs_share 10.3.1.192/28(rw,no_root_squash)
```
Je crée ensuite le dossier `/srv/nfs_share/`.

Sur ma machine `web1.server2.tp3` qui sera donc le client NFS, j'installe également nfs-utils.
De même que pour le serveur NFS, je me rends dans le fichier `/etc/idmapd.conf` et je fais la même modification.
Je crée le dossier `/srv/nfs/` et je fais `sudo mount -t nfs nfs1.server2.tp3:/srv/nfs_share /srv/nfs`.

**TEST**

Je teste que je peux lire et écrire dans `/srv/nfs/` depuis `web1.server2.tp3`.
```
[hugoj@web1 nfs]$ touch test
[hugoj@web1 nfs]$ ls
test
```

Je regarde ensuite sur `nfs1.server2.tp3` si je retrouve les modifications dans `/srv/nfs_share/`.
```
[hugoj@nfs1 nfs_share]$ ls
test
```

## IV. Un peu de théorie : TCP et UDP


## V. El final

![](https://i.imgur.com/wz53wcY.png)


Tableau des réseaux : 
| Nom du réseau | Adresse du réseau | Masque        | Nombre de clients possibles | Adresse passerelle | Adresse broadcast   |
|---------------|-------------------|---------------|-----------------------------|--------------------|---------------------|
| `server1`     | `10.3.1.0`        |`255.255.255.128` | 126                      | `10.3.1.126`       | `10.3.1.127`        |
| `client1`     | `10.3.1.128`      |`255.255.255.192` | 62                       | `10.3.1.190`       | `10.3.1.191`        |
| `server2`     | `10.3.1.192`      |`255.255.255.240` | 14                       | `10.3.1.206`       | `10.3.1.207`        |



Tableau d'adressage : 
| Nom machine         | Adresse IP `client1`       | Adresse IP `server1` | Adresse IP `server2` | Adresse de passerelle |
|---------------------|----------------------------|----------------------|----------------------|-----------------------|
| `router.tp3`        | `10.3.1.190/26`            | `10.3.1.126/25`      | `10.3.1.206/28`      | Carte NAT             |
| `dhcp.client1.tp3`  | `10.3.1.130/26`            | X                    | X                    | `10.3.1.190/26`       |
| `marcel.client1.tp3`| `10.3.1.129/26` (dynamique)| X                    | X                    | `10.3.1.190/26`       |
| `dns1.server1.tp3`  | X                          | `10.3.1.10/25`       | X                    | `10.3.1.126/25`       |
| `web1.server2.tp3`  | X                          | X                    | `10.3.1.200`         | `10.3.1.206`          |
| `nfs1.server2.tp3`  | X                          | X                    | `10.3.1.201`         | `10.3.1.206`          |