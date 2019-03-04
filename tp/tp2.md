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

![Capture 2](C:\Users\benoi\AppData\Roaming\Typora\typora-user-images\1551706312205.png)

