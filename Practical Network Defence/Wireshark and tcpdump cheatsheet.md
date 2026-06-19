***************
comandi associazioni ip-mac:
```
ip neigh
ip neigh show
arp -a
ip n
``` 
*************

Display Filters:

Cercare indirizzo ip sia sorgente che destinazione:
`ip.addr==10.0.0.1

Cercare indirizzo ip sorgente:
`ip.src==10.10.0.0

Cercare indirizzo ip destinazione:
`ip.dst==10.10.3.1

Cercare due indirizzi ip:
`ip.addr==10.0.0.1 || ip.addr==100.1.1.1

Cercare indirizzo ip sorgente e con porta SSH
`ip.src==10.0.0.1 && ssh

Cercare indirizzo ip con porta tcp sorgente 2220
`ip.src==10.0.0.3 && tcp.srcport==2220

Cercare indirizzo ip con porta tcp destinazione 2220
`ip.src==10.0.0.1 && tcp.dstport==2220

Cercare indirizzo ip con porta sorgente tcp o udp 2220
`ip.src==10.0.0.2 && (tcp.srcport==2220 || udp.srcport==2220)

Cercare indirizzo sorgente ipv6:
`ipv6.src==2001::1

Filtrare pacchetti http:
`http

Si può utilizzare eq al posto di ==

Cercare indirizzo ip sia sorgente che destinazione:
`ip.addr eq 10.10.1.1

Cercare indirizzo ip sorgente in un range di indirizzi
`ip.src > 10.0.0.0 and ip.src < 255.0.0.0

Escludere un indirizzo ip
`!(ip.src == 10.0.0.0)

Cercare indirizzo mac sorgente:
`eth.src == 08:00:02:02:0d:04

Cercare indirizzi con time to live = 2
`ip.ttl == 2

***********************

Capture Filters:

Catturare indirizzo ip sia sorgente che destinazione:
`host 10.0.1.15

Catturare indirizzo ip sorgente:
`src host 10.0.2.1
oppure
`ip src 10.0.2.1

Catturare indirizzo ip destinazione:
`dst host 10.0.0.1
oppure
`ip src 10.0.2.2

Catturare solo la porta 54:
`port 53

Catturare porta tcp 10:
`tcp port 10

Catturare porta sorgente tcp 10:
`tcp src port 10

Catturare porta destinazione tcp 20
`tcp dst port 20

Catturare host con porta ssh:
`host 10.0.0.1 and ssh

Catturare host con porta 22:
`host 10.0.0.2 and port 22

Berkley Filter:
sintassi: protocol , direction type
protocol: tcp, udp, arp, ip, ip6 ecc.
direction: src, dst
type: host, port, net,portrange, (nel caso di ip host nel caso di tcp port ecc.)
In wireshark quando si vuole selezionare porta o indirizzo si deve scrivere il type ovvero host o port altrimenti non funziona

Catturare host sorgente con ip 10.0.0.2:
`ip src host 10.0.0.2

Catturare porta sorgente 22:
`tcp src port 22

Catturare indirizzo ipv6 sorgente:
`ipv6 src host 2001::1

Cercare indirizzi sorgenti con porta sorgente tcp o udp 22:
`ip src host 10.0.0.2 and (tcp src port 22 || udp src port 22)

***********************

tcpdump

opzioni:
-i: specifica interfaccia dal quale ascoltare
-w: salva l'output in un file .pcap
-n: non converte gli indirizzi in nomi
-t: non mostra i timestamp
-v: versione verbosa
-vvv: incrementa la verbosità 

host: filtra l'indirizzo ip o l'indirizzo web (sorgente e destinazione)
port: filtra la porta (sorgente e destinazione)
proto \\protocollo: cattura il traffico del protocollo specificato ma funziona solo con icmp tcp e udp


Catturare pacchetti dall'interfaccia eth0:
`tcpdump -i eth0

Catturare pacchetti dall'interfaccia eth0 e salvarli in file.pcap
`tcpdump -i eth0 -w file.pcap

Catturare pacchetti riguardanti l'host 10.10.2.1
tcpdump host 10.10.2.1

Catturare pacchetti provenienti dalla subnet 10.0.2.2/24
tcpdump net 10.0.2.2/24

Catturare pacchetti che hanno come sorgente 10.10.2.1
tcpdump src 10.10.2.1

Catturare pacchetti che hanno come destinazione 10.10.2.1
tcpdump dst 10.10.2.1

Catturare i pacchetti che hanno come sorgente o destinazione la porta 443
tcpdump port 443

Catturare pacchetti che hanno come porta sorgente 22
tcpdump src port 22

Catturare pacchetti che hanno come porta destinazione 22
tcpdump dst port 22

Catturare pacchetti che hanno come porta tcp 43:
tcpdump tcp port 43

Catturare pacchetti che coinvolgono www.google.com
tcpdump host "www.google.com"

Catturare i pacchetti che hanno come sorgente www.google.com
tcpdump src "www.google.com"

Catturare i pacchetti che hanno come destinazione www.google.com
tcpdump dst "www.google.com"

Catturare traffico ip6
tcpdump ip6

Catturare traffico icmp
tcpdump icmp

Catturare traffico udp
tcpdump udp

Catturare traffico tcp:
tcpdump proto \\tcp