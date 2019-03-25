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