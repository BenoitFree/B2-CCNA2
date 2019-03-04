------

# I. Mise en place du lab

### 1. Création des VMs et adressage IP

### 2. Routage statique 

```
Activation de l'IPv4 Forwarding
[centos@router1 ~]$ sudo sysctl -w net.ipv4.conf.all.forwarding=1
[sudo] Mot de passe de centos : 
net.ipv4.conf.all.forwarding = 1
[centos@router2 ~]$ sudo sysctl -w net.ipv4.conf.all.forwarding=1
[sudo] Mot de passe de centos : 
net.ipv4.conf.all.forwarding = 1
Le client est capable de ping le server1.

[centos@client1 ~]$ ping 10.2.2.10
PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
64 bytes from 10.2.2.10: icmp_seq=1 ttl=62 time=1.90 ms
64 bytes from 10.2.2.10: icmp_seq=2 ttl=62 time=1.22 ms
64 bytes from 10.2.2.10: icmp_seq=3 ttl=62 time=1.04 ms
64 bytes from 10.2.2.10: icmp_seq=4 ttl=62 time=1.10 ms
c64 bytes from 10.2.2.10: icmp_seq=5 ttl=62 time=1.89 ms
64 bytes from 10.2.2.10: icmp_seq=6 ttl=62 time=1.43 ms
^C
--- 10.2.2.10 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5029ms
rtt min/avg/max/mdev = 1.048/1.434/1.906/0.353 ms
```

**Expliquer pourquoi depuis router1 on n'arrive pas à ping server1 Server1 ne connait pas l'adresse de réseau de router 1 donc il ne pas communiqué avec ce dernier.**
De la manière que le router2 ne pas ping client1, puique la route retour n'existe pas.

### 3. Visualisation du routage avec Wireshark

![Capture 1](https://github.com/BenoitYnov/Tp-ccna2/blob/master/img/tp2-1.PNG?raw=true)

![Capture 2](https://github.com/BenoitYnov/Tp-ccna2/blob/master/img/tp2-2.PNG?raw=true)



# II. NAT et services d'infra

### 1. Mise en place du NAT

> Après avoir curl vers google.com je modifie les fichiers de config de router1.

```
[root@router1 network-scripts]# cat ifcfg-enp0s3 | grep "public"
ZONE=public

[root@router1 network-scripts]# cat ifcfg-enp0s8 | grep "public"
ZONE=public

[root@router1 network-scripts]# cat ifcfg-enp0s9 | grep "internal"
ZONE=internal
```

> Activation de NAT dans la zone public

```
[centos@router1 ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
[sudo] Mot de passe de centos : 
success
[centos@router1 ~]$ sudo firewall-cmd --reload
success
```

> Sur router2, j'ajoute une route vers router1 puis je test l'accès à internet

```
[centos@router2 network-scripts]$ cat route-enp0s9
default via 10.2.12.2 dev enp0s9
10.2.1.0/24 via 10.2.12.2 dev enp0s9

[centos@router2 network-scripts]$ curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

> Idem pour client1

```
[root@client1 network-scripts]# cat route-enp0s8
default via 10.2.1.254 dev enp0s8
10.2.2.0/24 via 10.2.1.254 dev enp0s8

[root@client1 network-scripts]# curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

> Je vérifie que server1 n'ait pas accès à internet

```
[centos@server1 ~]$ curl google.com
curl: (6) Could not resolve host: google.com; Erreur inconnue
```



### 2. DHCP server

```
1. sudo yum install -y dhcp
2. sudo systemctl start dhcpd
3. Modification du dhcpd.conf
```

```
[root@client2 network-scripts]# cat ifcfg-enp0s8
TYPE=Ethernet
BOOTPROTO=dhcp
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes
ZONE=public

> dhclient
```



### 3. NTP server

```
[centos@router1 ~]$ sudo systemctl restart chronyd
```

```
chronyc> sources
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* mica.ad-notam.net             2   6    17    57  +2617us[ +703us] +/-   47ms
^+ herbrand.noumicek.cz          2   6    17    57   +194us[-1720us] +/-   49ms
^+ web01.webhd.nl                3   6    17    57  -2820us[-4734us] +/-   77ms
^+ ntp2.omdc.pl                  2   6    17    57   +108us[-1806us] +/-   78ms
```

```
chronyc> tracking
Reference ID    : 25BB682C (herbrand.noumicek.cz)
Stratum         : 3
Ref time (UTC)  : Mon Mar 04 15:39:23 2019
System time     : 0.000314301 seconds slow of NTP time
Last offset     : -0.001164005 seconds
RMS offset      : 0.001164005 seconds
Frequency       : 6.969 ppm fast
Residual freq   : -13.407 ppm
Skew            : 0.750 ppm
Root delay      : 0.044139553 seconds
Root dispersion : 0.027277250 seconds
Update interval : 63.9 seconds
Leap status     : Normal
```

