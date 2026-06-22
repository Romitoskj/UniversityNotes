
Firewall:
Listare tutte le regole della Table FILTER:
iptables -L

Comandi iptables:
-t table : specifica la Table
-j target: specifica l'azione (ACCEPT, DROP o REFUSE)
-A chain: appende la regola alla specifica Chain
-F : resetta la chain
-L : lista tutte le regole
-L -n : lista le regole facendo vedere gli indirizzi ip
-t nat -L : lista le regole del NAT
-P policy: cambia la policy (quindi il pacchetto non matcha nessuna regola della chain gli viene applicata l'azione di default che può essere ACCEPT, DROP o REFUSE)
esempi:
iptables -P INPUT DROP (scarta tutti i pacchetti diretti al proprio host)
-p protocol: matcha il protocollo in questione (tcp,udp,icmp)
-s ip-address : matcha l'ip sorgente (può essere o un ip o un network)
-d ip-address: matcha l'ip destinazione 
-p tcp --sport port: matcha la porta sorgente
-p tcp --dport port: matcha la porta destinazione
chiaramente al posto di tcp possiamo avere anche udp
-i interface: matcha l'interfaccia di input
-o interface: matcha l'interfaccia di output
--icmp-type tipo: matcha lo specifico tipo di pacchetto icmp (echo-reply o echo-request)
-m module: usa un modulo estensivo
specificare regole che considerano lo stato della connessione:
-m state --state s: matcha un pacchetto che è in uno specifico stato che può essere:
NEW: pacchetto che è l'inizio di una connessione
ESTABLISHED: pacchetto che è parte di una connessione stabilita
RELATED: pacchetto che è parte di una connessione related
INVALID: il pacchetto non può essere identificato
-m multiport: specifica più porte
--syn: matcha i pacchetti che iniziano una connessione TCP come SYN,ACK SYC
-m mac --mac-source: matcha l'indirizzo mac
-m conntrack --ctstate: abilita connection tracking

esempio:
iptables -A FORWARD -s 0/0 -i eth0 -d 192.168.1.58 -o eth1 -p TCP \ --sport 1024:65535 -m multiport --dport 80,443 -j ACCEPT

bloccare gli icmp request
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

I pacchetti ICMP possono essere anche di tipo RELATED , questo quando dobbiamo notificare ad esempio che la destinazione non è raggiungibile

Tabelle:
FILTER: filtra i pacchetti
MANGLE: Manipola campi dell'header dei pacchetti
NAT: Manipola gli indirizzi ip o le porte
RAW: Si occupa del tracciamento delle connessioni

MANGLE:
Manipola campi come TTL, tipo di servizio ecc. ma non può essere usato per filtrare o nat
ha 5 chains:
PREROUTING: viene valutato prima che vengono prese le decisioni di routing
POSTROUTING: viene valutato prima che vengono prese le decisioni di routing
INPUT: pacchetti destinati alla macchina in sè
OUTPUT: pacchetti uscenti generati dalla macchina
FORWARD: pacchetti in input non destinati alla macchina in sè

FILTER:
usato per filtrare i pacchetti
ha 3 chains:
INPUT
OUTPUT
FORWARD

NAT:
non è usata per filtrare i pacchetti ma per la traduzione degli indirizzi quindi per manipolare l'indirizzo sorgente o quello di destinazione ma anche per manipolare la porta sorgente o di destinazione
Il Nat ha i seguenti Target:
1) SNAT (source nat)
2) DNAT (destination nat)
3) MASQUERADE (dynamic nat)
4) REDIRECT 
ha 3 chains:
POSTROUTING
PREROUTING
OUTPUT

RAW:
Si utilizza per la connection tracking
ha 2 chains:
PREROUTING
OUTPUT

SNAT (Source address translation):
È quando un pc della Lan manda un pacchetto all'esterno, il router prende il pacchetto e al posto dell'indirizzo sorgente del pc ci inserisce il suo ip
Viene utilizzato nel POSTROUTING

DNAT (Destination Nat):
Quando il router riceve un paccheto dall'esterno cambia la destinazione e al posto del suo ip ci mette quello del computer di destinazione
Viene utilizzato nel PREROUTING, perchè ovviamente l'indirizzo di destinazione va cambiato prima che venga presa la decisione di routing

MASQUERADE:
È un caso speciale dell'SNAT chiamato anche dynamic nat, invece di avere un assegnamento statico dell'indirizzo ve ne è uno dinamico.
Utile quando l'interfaccia che è tra il router e la WAN cambia ip frequentemente

REDIRECT:
Ridireziona il pacchetto a se stesso (alla macchina in sè)

Priorità delle Tables:
RAW > MANGLE > NAT > FILTER

Priorità delle Chain:
PREROUTING < INPUT
PREROUTING < FORWARD < POSTROUTING
OUTPUT < POSTROUTING

PREROUTING: MANGLE, NAT
POSTROUTING: MANGLE, NAT
FORWARD: MANGLE, FILTER
INPUT: MANGLE, FILTER
OUTPUT: MANGLE, NAT, FILTER

esempi:
iptables -t NAT -o eth1 -A POSTROUTING -j MASQUERADE
iptables -t NAT -A POSTROUTING -j SNAT --to 192.168.0.1 (cambia ip sorgente del pacchetto con ip del router)
iptables -t NAT -A PREROUTING -j DNAT --to 100.1.1.1 (cambia ip destinazione del pacchetto con 100.1.1.1)
iptables -P INPUT DROP (Cambia la policy di default della chain input della tabella filter in DROP, ricorda che quando si cambia la policy non serve il -j)
iptables -t nat -A PREROUTING -p udp --dport 514 -j REDIRECT --to-ports 5140