## I. Manipulation de switches et de VLAN

### 1. Mise en place du lab

####  Vérification

- [x] [Nom de domaines](https://github.com/It4lik/B2-Reseau-2018/blob/master/cours/procedures.md#changer-son-nom-de-domaine) sur toutes les machines
- [x] Toutes les machines doivent pouvoir se `ping`



##  2. Configuration des VLANs

```bash
SW1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/3, Et1/0, Et1/1, Et1/2
                                                Et1/3, Et2/0, Et2/1, Et2/2
                                                Et2/3, Et3/0, Et3/1, Et3/2
                                                Et3/3
10   client1-sw1                      active    Et0/1
20   sw1-client2                      active    Et0/0
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

```

On ne peut donc plus ping client2 depuis client1 	

```bash
[centos@client]# ping 10.1.1.2
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
^C
--- 10.1.1.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2000ms

[centos@client]# ping 10.1.1.3
PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=1,63 ms
64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=1,56 ms
64 bytes from 10.1.1.3: icmp_seq=3 ttl=64 time=1,66 ms
^C
--- 10.1.1.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2092ms
```



## II. Manipulation simple de routeurs

### 1. Mise en place du lab

Configuration de **router1** et **router2**

> Exemple : Configuration du router 1

```bash
# conf t
R1(config)#interface fastEthernet 1/0
R1(config-if)#ip address 10.2.1.254 255.255.255.0
R1(config-if)#no shut
R1(config-if)#exit
R1(config)#exit
# show ip int br
```

> Server1 ping vers router 2

```bash
[server1@client]# ping 10.2.2.254
PING 10.2.2.254 (10.2.2.254) 56(84) bytes of data.
64 bytes from 10.2.2.254: icmp_seq=1 ttl=64 time=28,4 ms
64 bytes from 10.2.2.254: icmp_seq=2 ttl=64 time=3,18 ms
64 bytes from 10.2.2.254: icmp_seq=3 ttl=64 time=7,16 ms
^C
--- 10.2.2.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2535ms
```

> Mettre R1 et R2 dans le même réseau  (même principe sur R2)

```bash
R1(config)#interface fastEthernet 2/0
R1(config-if)#ip address 10.2.12.1 255.255.255.252
R1(config-if)#no shut
R1(config-if)#exit
R1(config)#exit
```



### 2. Configuration du routage statique

Configuration des routes 

> Exemple : Configuration server1

```bash
[server1@client]# sudo ip route add 10.2.1.0/24 via 10.2.2.254 dev enp0s3
```

> Configuration client2

```bash
[client2@client]# sudo ip route add 10.2.2.0/24 via 10.2.1.254 dev enp0s3
```

> Client2 ping vers server1

```bash
[client2@client]# ping 10.2.2.10
PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
64 bytes from 10.2.2.10: icmp_seq=1 ttl=62 time=37,9 ms
64 bytes from 10.2.2.10: icmp_seq=2 ttl=62 time=27,1 ms
64 bytes from 10.2.2.10: icmp_seq=3 ttl=62 time=30,0 ms
^C
--- 10.2.2.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 4185ms
```

> Server1 ping vers client2

```bash
[server1@client]# ping 10.2.1.11
PING 10.2.1.11 (10.2.2.11) 56(84) bytes of data.
64 bytes from 10.2.1.11: icmp_seq=1 ttl=62 time=37,5 ms
64 bytes from 10.2.1.11: icmp_seq=2 ttl=62 time=38,4 ms
64 bytes from 10.2.1.11: icmp_seq=3 ttl=62 time=34,5 ms
^C
--- 10.2.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2260ms
```



# III. Mise en place d'OSPF

### 1. Mise en place du lab

> Configuration des routeurs

```bash
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.3.100.1      YES manual up                    up
FastEthernet1/0            10.3.102.254    YES manual up                    up
FastEthernet2/0            10.3.100.22     YES manual up                    up
```

```bash
R2#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.3.100.2      YES manual up                    up
FastEthernet1/0            10.3.100.5      YES manual up                    up

```

```bash
R3#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.3.100.6      YES manual up                    up
FastEthernet1/0            10.3.100.9      YES manual up                    up
```

```bash
R4#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.3.100.13     YES manual up                    up
FastEthernet1/0            10.3.100.10     YES manual up                    up
FastEthernet2/0            10.3.101.254    YES manual up                    up

```

```bash
R5#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.3.100.17     YES manual up                    up
FastEthernet1/0            10.3.100.14     YES manual up                    up

```

```bash
R6#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.3.100.21     YES manual up                    up
FastEthernet1/0            10.3.100.18     YES manual up                    up

```

> Chaque routeur peut ping son voisin, exemple avec R2

```bash
R2#ping 10.3.100.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.100.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
R2#ping 10.3.100.5

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.100.5, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/4 ms

```

### 2. Configuration de OSPF

> Exemple : Configuration OSPF de router 2 

```bash
R2(config)#router ospf 1
R2(config-router)#router
R2(config-router)#router-id 2.2.2.2
R2(config-router)#network 10.3.100.2 0.0.0.3 area 0

R2(config-router)#network 10.3.100.5 0.0.0.3 area 0
```

> ... Il faut le faire pour chaque router

```bash
R4(config)#router ospf 1
R4(config-router)#router-id 4.4.4.4
R4(config-router)#network 10.3.101.254 0.0.0.255 area 1

R4(config-router)#network 10.3.100.10 0.0.0.3 area 0

R4(config-router)#network 10.3.100.13 0.0.0.3 area 0
```



# IV. Lab Final

**Topologie** 

![](https://raw.githubusercontent.com/BenoitYnov/B2-CCNA2/master/img/topo.png)

|      Hosts       | 10.3.100.0/30 | 10.3.100.4/30 | 10.3.101.0/24 | 10.3.102.0/24 |
| :--------------: | :-----------: | :-----------: | :-----------: | :-----------: |
| client1.lab4.tp3 |       x       |       x       |  10.3.101.11  |       x       |
| client2.lab4.tp3 |       x       |       x       |  10.3.101.12  |       x       |
| server1.lab4.tp3 |       x       |       x       |       x       |  10.3.102.10  |
| server2.lab4.tp3 |       x       |       x       |       x       |  10.3.102.11  |
| router1.lab4.tp3 |  10.3.100.1   |  10.3.100.5   |       x       |       x       |
| router2.lab4.tp3 |  10.3.100.2   |  10.3.100.6   |       x       |       x       |

