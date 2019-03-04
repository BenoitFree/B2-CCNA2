

[TOC]

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

> Modification du fichier de config chrony

```
[centos@router2 network-scripts]$ cat /etc/chrony.conf | grep "server"
# Use public servers from the pool.ntp.org project.
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```

> Ajout du port/udp 123

```
[centos@router1 ~]$ sudo firewall-cmd --add-port=123/udp --permanent
success	
```

> Vérification de l'état de la synchronisation NTP

```
[centos@router1 ~]$ sudo systemctl start chronyd
chronyc> sources
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^- powered.by.root24.eu          2   8   373   100  +4628us[+4500us] +/-   61ms
^- web01.webhd.nl                3   9   377   816  +6843us[+6547us] +/-   79ms
^* regar42.fr                    3   9   377    38   -314us[ -445us] +/- 9837us
^- clients14.arcanite.ch         2   9   377   334  +7209us[+7097us] +/-   65ms

chronyc> tracking
Reference ID    : 3ED2F492 (regar42.fr)
Stratum         : 4
Ref time (UTC)  : Mon Mar 04 15:55:12 2019
System time     : 422.776092529 seconds slow of NTP time
Last offset     : -0.000131207 seconds
RMS offset      : 53.378540039 seconds
Frequency       : 6.897 ppm fast
Residual freq   : -0.012 ppm
Skew            : 0.332 ppm
Root delay      : 0.019673342 seconds
Root dispersion : 0.000057056 seconds
Update interval : 475.6 seconds
Leap status     : Normal
```

On remarque qu'il y à **1 heure** de moins.



### 4. Web server

> Ajout de la carte NAT sur server1 pour avoir une connexion à internet
>
> Installation de nginx  + ouverture du port/tcp 80

```
[centos@server1 ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
```

> Vérification

![server](https://github.com/BenoitYnov/Tp-ccna2/blob/master/img/server.PNG?raw=true)

