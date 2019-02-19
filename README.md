Welcome file
Welcome file
# b2-net-tp1	
## I. Exploration du réseau d'une machine CentOS
### 1. Mise en place
### combien y a-t-il d'adresses disponibles dans un  `/24`  ?
Il y a 256 adresses disponibles dans un /24

### combien y a-t-il d'adresses disponibles dans un  `/30`  ? Son utilité ?
• Il y a 2 adresses disponibles dans un /30

• On sait qu'il n-y à que 2 machines, donc c'est plus sécurisé.

### Route statique
```bash
TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes
IPADDR=10.1.1.2
NETMASK=255.255.255.0
ZONE=public
```
_BOOTPROTO=static_
Idem pour la enp0s9

### Connexion SSH
• Je me connecte en ssh (Powershell) : 
```
centos@10.1.1.2
```
-> Connexion réussi

### Définition nom de domaine 
```bash
echo 'client1.tp1.b2' | sudo tee /etc/hostname
```
-> Changement réussi

### Changement Hosts
```bash
10.1.1.1 client1 client1.tp1.b2
10.1.2.1 client1 client1.tp1.b1
```

### Fonctionnement des 3 cartes
__Nat__ 
```bash
curl google.com
```
nous renvoie une réponse __HTTP 301__
Le code 301 permet une redirection permanente vers une nouvelle page. 

__Net1 et net2__
Ping
```bash
Envoi d’une requête 'Ping'  10.1.1.1 avec 32 octets de données :
Réponse de 10.1.1.1 : octets=32 temps<1ms TTL=128
Réponse de 10.1.1.1 : octets=32 temps<1ms TTL=128
Réponse de 10.1.1.1 : octets=32 temps<1ms TTL=128
```
```bash
Envoi d’une requête 'Ping'  10.1.2.1 avec 32 octets de données :
Réponse de 10.1.2.1 : octets=32 temps<1ms TTL=128
Réponse de 10.1.2.1 : octets=32 temps<1ms TTL=128
Réponse de 10.1.2.1 : octets=32 temps<1ms TTL=128
```


## 2. Basics

• Supprimer 
```bash
#On supprime la route par defaut pour que le TP marche ^^
ip route del default
```
• Afficher
```bash
[root@client1 network-scripts]# ip route show
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 101
10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 102
```
L'affichage de la table de routage permet de vérifier que tout s'est bien passé :

```bash
#Je supprime une route
ip route del 10.1.2.0/30

#Je vérifie qu'elle a bien été suprimée
Délai d’attente de la demande dépassé.
Statistiques Ping pour 10.1.2.0:
    Paquets : envoyés = 1, reçus = 0, perdus = 1 (perte 100%),
    
#Je remets la roude supprimée.
systemctl restart network
ip route show
10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 102
```

### Table ARP
```bash
[root@client1 network-scripts]# ip neigh show
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:47 DELAY
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
```
Delay : Expirée
Stale : Arrive bientôt à expiration 
```bash
# Je vide la table ARP : ip neigh flush all*[root@client1 network-scripts]# ip neigh show
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:47 REACHABLE

#J'effectue une requête : ping 10.1.2.1
#J'affiche la table ARP
[root@client1 network-scripts]# ip neigh show
10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:47 DELAY
10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:4b REACHABLE
```
l'adresse 10.1.2.1 ping récément apparait dans la table ARP avec : _REACHABLE_
Elle a été joignable récemment.

### Capture Réseau
*(faut desactiver parefeu windows sinon ça marche pas mdr)*
```bash
#Sur ssh 1
sudo tcpdump -i enp0s8 -w ping.pcap`
#Sur ssh2
ping 10.1.2.1 

#Je quitte le ssh, je fais :
PS C:\Users\benoi> scp centos@10.1.1.2:/home/centos/ping.pcap .\Downloads\
#Je récupère le ping.pcap et je l'ouvre avec Wireshark et j'observe :)
```
## II. Communication simple entre deux machines

### 1. Mise en place

Markdown 3527 bytes 534 words 135 lines Ln 8, Col 43 HTML 2764 characters 493 words 90 paragraphs
