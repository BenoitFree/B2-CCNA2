# b2-net-tp1

## I. Exploration du réseau d'une machine CentOS

### 1. Mise en place

### combien y a-t-il d'adresses disponibles dans un  `/24`  ?

Il y a 256 adresses disponibles dans un /24

### combien y a-t-il d'adresses disponibles dans un  `/30`  ? Son utilité ?

Il y a 2 adresses disponibles dans un /30

On sait qu'il n-y à que 2 machines, donc c'est plus sécurisé.

### Route statique
```
TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes
IPADDR=10.1.1.2
NETMASK=255.255.255.0
ZONE=public
```
_BOOTPROTO=static_  Idem pour la enp0s9

### Connexion SSH

Je me connecte en ssh (Powershell) :

```
ssh centos@10.1.1.2
```

-> Connexion réussi

### Définition nom de domaine
```
echo 'client1.tp1.b2' | sudo tee /etc/hostname
```
-> Changement réussi

#### Changement Hosts
```
10.1.1.1 client1 client1.tp1.b2
10.1.2.1 client1 client1.tp1.b1
```
### Fonctionnement des 3 cartes

**Nat**
```
curl google.com
```
nous renvoie une réponse  **HTTP 301**

Le code 301 permet une redirection permanente vers une nouvelle page.

**Net1 et net2**  >  Ping
```
Envoi d’une requête 'Ping'  10.1.1.1 avec 32 octets de données :
Réponse de 10.1.1.1 : octets=32 temps<1ms TTL=128
Réponse de 10.1.1.1 : octets=32 temps<1ms TTL=128
Réponse de 10.1.1.1 : octets=32 temps<1ms TTL=128
```
```
Envoi d’une requête 'Ping'  10.1.2.1 avec 32 octets de données :
Réponse de 10.1.2.1 : octets=32 temps<1ms TTL=128
Réponse de 10.1.2.1 : octets=32 temps<1ms TTL=128
Réponse de 10.1.2.1 : octets=32 temps<1ms TTL=128
```
## [](https://github.com/BenoitYnov/b2-net-tp1/blob/master/tp/tp1.md#2-basics)2. Basics

• Supprimer

On supprime la route par defaut pour que le TP marche ^^
```
ip route del default
```
• Afficher
```
[root@client1 network-scripts]# ip route show
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 101
10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 102
```
L'affichage de la table de routage permet de vérifier que tout s'est bien passé :

• Je supprime une route
```
ip route del 10.1.2.0/30
```
• Je vérifie qu'elle a bien été suprimée
```
Délai d’attente de la demande dépassé.
Statistiques Ping pour 10.1.2.0:
    Paquets : envoyés = 1, reçus = 0, perdus = 1 (perte 100%),
```
• Je remets la roude supprimée.
```
systemctl restart network
ip route show
10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 102
- ou alors : 
sudo ip route add 10.1.2.0/30 via 10.1.2.2 dev enp0s9
```
### Table ARP
```
[root@client1 network-scripts]# ip neigh show
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:47 DELAY
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
```
Delay : Expirée
Stale : Arrive bientôt à expiration


•  Je vide la table ARP :
```
[root@client1 network-scripts]# ip neigh flush all
[root@client1 network-scripts]# ip neigh show
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:47 REACHABLE
```

• J'effectue une requête : 
```
ping 10.1.2.1
```
• J'affiche la table ARP
```
[root@client1 network-scripts]# ip neigh show
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:47 DELAY
10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:4b REACHABLE
```
l'adresse 10.1.2.1 ping récément apparait dans la table ARP avec :  _REACHABLE_

Elle a été joignable récemment.

### Capture Réseau

_(faut desactiver parefeu windows sinon ça marche pas mdr)_

```
Sur ssh 1
 sudo tcpdump -i enp0s8 -w ping.pcap`
Sur ssh2
 ping 10.1.2.1 
```
• Je quitte le ssh, je fais :
```
C:\Users\benoi> scp centos@10.1.1.2:/home/centos/ping.pcap .\Downloads\
```
Je récupère le ping.pcap et je l'ouvre avec Wireshark et j'observe :)

## II. Communication simple entre deux machines

### UDP

```
[centos@client1 ~]$ sudo firewall-cmd --add-port=8888/udp --permanent
[centos@client1 ~]$ sudo firewall-cmd --reload
[centos@client1 ~]$ nc -u -l 8888

[centos@client2 ~]$ nc -u 10.1.1.2 8888
```
Je peux ensuite communiquer entre les 2 VM

```bash
[centos@client1 ~]$ ss -unp
Recv-Q Send-Q                    Local Address:Port                                   Peer Address:Port
0      0                              10.1.1.2:8888                                       10.1.1.3:59629               users:(("nc",pid=4456,fd=4))

[centos@client1 ~]$ sudo tcpdump -i enp0s8 -w nc-udp.pcap
Je récupère la capture pour l'analyser avec Wireshark
PS C:\Users\benoi> scp centos@10.1.1.2:/home/centos/nc-udp.pcap .\Downloads\
> 100% 1327   265.5KB/s
```

### TCP

```
[centos@client1 ~]$ nc -l 8888
[centos@client2 ~]$ nc 10.1.1.2 8888
```

### Firewall

```bash
Client 1 : 
[centos@client1 ~]$ sudo firewall-cmd --remove-port=8888/udp --permanent
[sudo] Mot de passe de centos :
success
[centos@client1 ~]$ sudo firewall-cmd --reload
success
[centos@client1 ~]$ nc -u -l 8888
Client 2 :
[centos@client2 ~]$ nc 10.1.1.2 8888

Second client 2 : 
C:\Users\benoi> scp centos@10.1.1.3:/home/centos/capture4.pcap .\Downloads\
centos@10.1.1.3's password:
capture4.pcap
```

_La destination n'est pas joignable car le port UDP 8888 a été fermé._

![firewall](https://github.com/BenoitYnov/b2-net-tp1/blob/master/img/capture4.PNG?raw=true)

## III. Routage statique simple
```
Client 1 : 
[centos@client1 ~]$ sudo sysctl -w net.ipv4.ip_forward=1
[sudo] Mot de passe de centos :
net.ipv4.ip_forward=1
```

```
Client 2 : 
[centos@client2 ~]$  sudo ip route add 10.1.2.0/30 via 10.1.1.2 dev enp0s9
[centos@client2 ~]$ ping 10.1.2.1
PING 10.1.2.1 (10.1.2.1) 56(84) bytes of data.
64 bytes from 10.1.2.1: icmp_seq=1 ttl=127 time=0.517 ms
64 bytes from 10.1.2.1: icmp_seq=2 ttl=127 time=0.677 ms
64 bytes from 10.1.2.1: icmp_seq=3 ttl=127 time=0.683 ms
^C
--- 10.1.2.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2095ms
rtt min/avg/max/mdev = 0.517/0.625/0.683/0.082 ms

[centos@client2 ~]$ traceroute 10.1.2.1
 1  gateway (10.0.2.2)  0.247 ms  0.167 ms  0.198 ms
 2  10.1.2.1 (10.1.2.1)  0.647 ms  0.581 ms  0.517 ms
```
