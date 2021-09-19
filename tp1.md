# TP1 - Mise en jambes

## I. Exploration locale en solo

### 1. Affichage d'informations sur la pile TCP/IP locale

**En ligne de commande**

**Affichez les infos des cartes réseau de votre PC :** 
* **Interface Wi-Fi :**
    Commande : `get-netadapter`
    `Name : Wi-Fi`
    `MacAddress : 74-D8-3E-0D-06-B0`
    Commande : `ipconfig`
    `Adresse IPv4 : 10.33.3.5`
* **Interface Ethernet :**
    Je n'ai pas de port Ethernet sur ma machine.
    
**Affichez votre gateway**

Commande : `ipconfig`
`Passerelle par défaut : 10.33.3.253`

**En graphique (GUI : Graphical User Interface)**

![](https://i.imgur.com/RpyHscF.png)

Questions : à quoi sert la gateway dans le réseau d'YNOV ?

La gateway dans le réseau d'YNOV sert à accéder aux autres réseaux.


### 2. Modifications des informations

#### A. Modification d'adresse IP (part 1)

![](https://i.imgur.com/AynEoaM.png)

En changeant son adresse IP il est possible de perdre l'accès à internet car on peut prendre une IP qui est déjà adressée ou alors prendre une IP qui n'est pas dans le réseau.


#### B. Table ARP

**Exploration de la table ARP :**

Pour afficher la table ARP en ligne de commande il faut faire la commande `arp -a`.

    Interface : 10.33.3.5 --- 0x12
      Adresse Internet      Adresse physique      Type
      10.33.0.242           74-4c-a1-51-1e-61     dynamique
      10.33.1.248           e8-84-a5-24-94-c9     dynamique
      10.33.2.56            c0-3c-59-a9-e0-75     dynamique
      10.33.3.80            3c-58-c2-9d-98-38     dynamique
      10.33.3.105           54-14-f3-b5-aa-36     dynamique
      10.33.3.123           38-fc-98-e5-ae-61     dynamique
      10.33.3.253           00-12-00-40-4c-bf     dynamique
      10.33.3.255           ff-ff-ff-ff-ff-ff     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      224.0.0.251           01-00-5e-00-00-fb     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique
      255.255.255.255       ff-ff-ff-ff-ff-ff     statique

      
Pour trouver la MAC de ma passerelle je cherche d'abord son IP avec un `ipconfig`. Ensuite je regarde dans ma table ARP l'adresse MAC qui correspond à l'IP de ma passerelle qui est `10.33.3.253`. Donc la MAC de ma passerelle est `00-12-00-40-4c-bf`.

**Remplissage de la table ARP :**

Après avoir ping d'autres IP on remarque qu'il y a des lignes en plus dans notre table ARP.

    Interface : 10.33.3.5 --- 0x12
      Adresse Internet      Adresse physique      Type
      10.33.0.119           18-56-80-70-9c-48     dynamique
      10.33.0.211           e8-d0-fc-ef-9e-af     dynamique
      10.33.0.242           74-4c-a1-51-1e-61     dynamique
      10.33.0.251           48-e7-da-41-bf-e1     dynamique
      10.33.1.243           34-7d-f6-5a-20-da     dynamique
      10.33.1.248           e8-84-a5-24-94-c9     dynamique
      10.33.2.56            c0-3c-59-a9-e0-75     dynamique
      10.33.3.10            a4-83-e7-59-ae-c2     dynamique
      10.33.3.80            3c-58-c2-9d-98-38     dynamique
      10.33.3.105           54-14-f3-b5-aa-36     dynamique
      10.33.3.123           38-fc-98-e5-ae-61     dynamique
      10.33.3.253           00-12-00-40-4c-bf     dynamique
      10.33.3.255           ff-ff-ff-ff-ff-ff     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      224.0.0.251           01-00-5e-00-00-fb     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique
      255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Lister les MAC des IP que j'ai ping :

    10.33.0.119           18-56-80-70-9c-48
    10.33.0.251           48-e7-da-41-bf-e1
    10.33.3.10            a4-83-e7-59-ae-c2
    
#### C. nmap

Après avoir fait un scan du réseau d'YNOV avec nmap on peut voir que ma table ARP a bien grandie.

    Interface : 10.33.3.5 --- 0x12
      Adresse Internet      Adresse physique      Type
      10.33.0.7             9c-bc-f0-b6-1b-ed     dynamique
      10.33.0.65            7c-04-d0-ce-8c-8a     dynamique
      10.33.0.96            ca-4f-f4-af-8f-0c     dynamique
      10.33.0.111           d2-41-f0-dc-6a-ed     dynamique
      10.33.0.117           8a-d8-25-ff-2e-a6     dynamique
      10.33.0.118           34-c9-3d-4d-a3-78     dynamique
      10.33.0.242           74-4c-a1-51-1e-61     dynamique
      10.33.1.70            30-57-14-94-de-fb     dynamique
      10.33.1.137           b0-7d-64-b1-98-d3     dynamique
      10.33.1.138           b0-7d-64-b1-98-d3     dynamique
      10.33.1.153           b0-7d-64-b1-98-d3     dynamique
      10.33.1.208           b2-dd-d8-d4-94-6c     dynamique
      10.33.1.243           34-7d-f6-5a-20-da     dynamique
      10.33.1.248           e8-84-a5-24-94-c9     dynamique
      10.33.2.58            ac-bc-32-88-1f-e7     dynamique
      10.33.2.198           24-ee-9a-e9-46-93     dynamique
      10.33.2.217           04-6c-59-0e-c4-91     dynamique
      10.33.3.59            02-47-cd-3d-d4-e9     dynamique
      10.33.3.123           38-fc-98-e5-ae-61     dynamique
      10.33.3.147           7c-04-d0-ce-8c-8a     dynamique
      10.33.3.152           b0-7d-64-b1-98-d3     dynamique
      10.33.3.154           b0-7d-64-b1-98-d3     dynamique
      10.33.3.168           12-88-c1-71-b7-a0     dynamique
      10.33.3.189           c2-6f-43-3d-c7-fa     dynamique
      10.33.3.253           00-12-00-40-4c-bf     dynamique
      10.33.3.255           ff-ff-ff-ff-ff-ff     statique
      224.0.0.22            01-00-5e-00-00-16     statique
      224.0.0.251           01-00-5e-00-00-fb     statique
      224.0.0.252           01-00-5e-00-00-fc     statique
      239.255.255.250       01-00-5e-7f-ff-fa     statique
      255.255.255.255       ff-ff-ff-ff-ff-ff     statique
      
      
#### D. Modification d'adresse IP (part 2)

Pour scanner le réseau d'YNOV on utilise la commande `nmap -sP 10.33.0.0/22`.

    Starting Nmap 7.92 ( https://nmap.org ) at 2021-09-16 10:17 Paris, Madrid (heure dÆÚtÚ)
    mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
    Nmap scan report for 10.33.0.5
    Host is up (0.025s latency).
    MAC Address: 1C:BF:C0:17:32:09 (Chongqing Fugui Electronics)
    Nmap scan report for 10.33.0.6
    Host is up (0.075s latency).
    MAC Address: 84:5C:F3:80:32:07 (Intel Corporate)
    Nmap scan report for 10.33.0.7
    Host is up (0.30s latency).
    MAC Address: 9C:BC:F0:B6:1B:ED (Xiaomi Communications)
    Nmap scan report for 10.33.0.19
    
    [...]
    
    MAC Address: 00:12:00:40:4C:BF (Cisco Systems)
    Nmap scan report for 10.33.3.254
    Host is up (0.0040s latency).
    MAC Address: 00:0E:C4:CD:74:F5 (Iskra Transmission d.d.)
    Nmap scan report for 10.33.1.142
    Host is up.
    Nmap done: 1024 IP addresses (139 hosts up) scanned in 23.53 seconds
    


![](https://i.imgur.com/HIq2pQN.png)

    Carte réseau sans fil Wi-Fi :

       Suffixe DNS propre à la connexion. . . :
       Adresse IPv6 de liaison locale. . . . .: fe80::d8f2:aebc:989a:c4ac%18
       Adresse IPv4. . . . . . . . . . . . . .: 10.33.1.142
       Masque de sous-réseau. . . . . . . . . : 255.255.252.0
       Passerelle par défaut. . . . . . . . . : 10.33.3.253
       
       
    PS C:\Users\hugoj> ping 8.8.8.8

    Envoi d’une requête 'Ping'  8.8.8.8 avec 32 octets de données :
    Réponse de 8.8.8.8 : octets=32 temps=25 ms TTL=115
    Réponse de 8.8.8.8 : octets=32 temps=17 ms TTL=115
    Réponse de 8.8.8.8 : octets=32 temps=18 ms TTL=115
    Réponse de 8.8.8.8 : octets=32 temps=19 ms TTL=115
    
Avec le ping 8.8.8.8 on voit que j'ai bien un accès internet.


## II. Exploration locale en duo

### Modification d'adresses IP

Avec un `ipconfig` on observe les changements qui ont été faits.
  
    Suffixe DNS propre à la connexion. . . : home
    Adresse IPv4. . . . . . . . . . . . . .: 192.168.10.1
    Masque de sous-réseau. . . . . . . . . : 255.255.255.252
    Passerelle par défaut. . . . . . . . . :
    
Le ping vers l'autre IP `192.168.10.2` fonctionne.

    PS C:\Users\baume> ping 192.168.10.2
    Envoi d’une requête 'Ping'  192.168.10.2 avec 32 octets de données :
    Réponse de 192.168.10.2 : octets=32 temps=4 ms TTL=128
    Réponse de 192.168.10.2 : octets=32 temps=2 ms TTL=128
    Réponse de 192.168.10.2 : octets=32 temps=2 ms TTL=128
    Réponse de 192.168.10.2 : octets=32 temps=2 ms TTL=128
    Statistiques Ping pour 192.168.10.2:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%)
    
Pour afficher la table arp on utilise `arp -a`.

    Interface : 192.168.10.1 --- 0xf
    Adresse Internet      Adresse physique      Type
    192.168.10.2          00-d8-61-87-1c-74     dynamique
    192.168.10.3          ff-ff-ff-ff-ff-ff     statique
    224.0.0.22            01-00-5e-00-00-16     statique
    224.0.0.251           01-00-5e-00-00-fb     statique
    224.0.0.252           01-00-5e-00-00-fc     statique
    239.255.255.250       01-00-5e-7f-ff-fa     statique
    255.255.255.255       ff-ff-ff-ff-ff-ff     statique
    
On constate qu'on retoruve l'IP de l'autre machine du réseau `192.168.10.2`.

### Utilisation d'un des deux comme gateway

Mon camarade arrive bien a ce connecter à internet avec son ping sur les serveurs dns de google

    PS C:\Users\LDLC> ping 8.8.8.8
    Envoi d’une requête 'Ping'  8.8.8.8 avec 32 octets de données :
    Réponse de 8.8.8.8 : octets=32 temps=20 ms TTL=114
    Réponse de 8.8.8.8 : octets=32 temps=20 ms TTL=114
    Réponse de 8.8.8.8 : octets=32 temps=20 ms TTL=114
    Statistiques Ping pour 8.8.8.8:
    Paquets : envoyés = 3, reçus = 3, perdus = 0 (perte 0%)
    
On vérifie aussi avec un ipconfig que mon camarade n'est pas connecté au wifi d'YNOV mais uniquement à moi

    Carte Ethernet Ethernet :
    Suffixe DNS propre à la connexion. . . :
    Adresse IPv6 de liaison locale. . . . .: fe80::88d8:25ec:2492:3bbf%10
    Adresse IPv4. . . . . . . . . . . . . .: 192.168.10.2
    Masque de sous-réseau. . . . . . . . . : 255.255.255.252
    Passerelle par défaut. . . . . . . . . : 192.168.10.1

    Carte réseau sans fil Wi-Fi :
    Statut du média. . . . . . . . . . . . : Média déconnecté
    Suffixe DNS propre à la connexion. . . : auvence.co
    

On utilise `tracert 8.8.8.8` pour vérifier que mon camarade passe bien par moi

    PS C:\Users\LDLC> tracert 8.8.8.8
    Détermination de l’itinéraire vers dns.google [8.8.8.8]
    avec un maximum de 30 sauts :
    1     1 ms     2 ms     1 ms  HUGO [192.168.10.1]
    


## III. Manipulations d'autres outils/protocoles côté client

### 1. DHCP

*J'ai fait cette partie chez moi donc j'ai l'IP du serveur DHCP de mon réseau Wi-Fi et non pas celui d'YNOV*

Pour afficher l'adresse IP du serveur DHCP il faut taper la commande `ipconfig /all`.

En cherchant on trouve l'IP du serveur DHCP de notre réseau.

    Serveur DHCP . . . . . . . . . . . . . : 192.168.1.1
    
Toujours avec la même commande on peut trouver la date d'expiration de notre bail DHCP.

    Bail expirant. . . . . . . . . . . . . : dimanche 19 septembre 2021 16:16:37
    
### 2. DNS

En faisant un `ipconfig` on peut trouver l'IP du serveur DNS que connaît notre ordinateur.

    Serveurs DNS. . .  . . . . . . . . . . : 192.168.1.1
   
Le *lookup* : 

* Pour google.com : 
    
    `Address: 142.250.179.78`

* Pour ynov.com : 

    `Address:  92.243.16.143`
    
Avec ces commandes on obtient les IP des noms de domaine google.com et ynov.com.

Grace à la commande `nslookup.exe google.com` et en faisant la même chose avec `ynov.com` on obtient la même adresse IPv6 du serveur.
    
    Serveur :   UnKnown
    Address:  fe80::6eba:b8ff:febc:81c0
    
Le *reverse lookup* : 

    PS C:\Users\hugoj> nslookup.exe 78.74.21.21
    Serveur :   UnKnown
    Address:  fe80::6eba:b8ff:febc:81c0

    Nom :    host-78-74-21-21.homerun.telia.com
    Address:  78.74.21.21

Le nom de domaine de `78.74.21.21` est `host-78-74-21-21.homerun.telia.com` .

    PS C:\Users\hugoj> nslookup.exe 92.146.54.88
    Serveur :   UnKnown
    Address:  fe80::6eba:b8ff:febc:81c0

    Nom :    apoitiers-654-1-167-88.w92-146.abo.wanadoo.fr
    Address:  92.146.54.88
    
Et le nom de domaine de `92.146.54.88` est `apoitiers-654-1-167-88.w92-146.abo.wanadoo.fr` .


## IV. Wireshark