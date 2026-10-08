# Appunti della Lezione: Reti di Sensori Senza Fili (WSN), Protocollo S-MAC e Routing

## 1. Introduzione alle Wireless Sensor Networks (WSNs)

### 1.1 Architettura Generale e Definizione di WSN

Una **Wireless Sensor Network (WSN)** è una rete di dispositivi elettronici distribuiti (nodi sensore) progettata per monitorare condizioni fisiche o ambientali e trasmettere i dati raccolti verso un punto di raccolta centralizzato, denominato **sink** (o pozzo). Poiché la portata di trasmissione delle radio a basso consumo dei singoli nodi è limitata, la comunicazione avviene generalmente su più salti (**multi-hop**), trasformando ciascun nodo sensore sia in un generatore di dati sia in un elemento di instradamento (router).

Il ciclo di vita dell'informazione e il flusso dati all'interno della rete si articolano in tre fasi operative fondamentali:

- **Sense (Rilevamento):** I nodi sensore posizionati nelle vicinanze di un evento fisico rilevano le grandezze ambientali d'interesse tramite i loro sensori a bordo.
- **Forward (Inoltro Multi-hop):** I dati raccolti vengono trasmessi lungo la rete hop-by-hop, integrando e rilanciando i pacchetti tra nodi adiacenti.
- **Deliver (Consegna al Sink):** Il sink raccoglie i flussi informativi e agisce da gateway verso l'infrastruttura di rete esterna (Internet) per la successiva elaborazione sui server centrali.

L'architettura complessiva del flusso informativo è rappresentata dalla seguente struttura concettuale:

- **Area Monitorata (Evento Fisico)**
    - **Nodi Sensore (S1, S2, ..., Sn)** _(Fase 1: Sense - Rilevamento locale)_
        - **Inoltro Multi-hop tra Nodi Vicini** _(Fase 2: Forward - Trasmissione hop-by-hop)_
            - **Nodo Sink / Gateway** _(Fase 3: Deliver - Raccolta centralizzata)_
                - **Rete Internet / IP Network**
                    - **Server Finale / Utenti e Applicazioni**

### 1.2 Anatomia del Nodo Sensore (Hardware e Software)

Un nodo sensore è un sistema embedded fortemente vincolato nell'hardware e coordinato da un'architettura software event-driven studiata per ridurre al minimo i consumi.

|   |   |
|---|---|
|Componente Hardware / Software|Descrizione e Ruolo Operativo|
|**Sensing Unit (Unità di Rilevamento)**|Integra i sensori fisici (es. temperatura, vibrazioni), i circuiti di condizionamento del segnale e un convertitore Analogico-Digitale (ADC). Un modello software converte le letture grezze in unità di misura fisiche.|
|**CPU e Memoria (Unità di Elaborazione)**|Microcontrollore a basso consumo energetico (MCU) accoppiato con risorse di memoria estremamente limitate: pochi Kilobyte (KB) di RAM e memoria Flash per il codice di programma e l'archiviazione dati temporanea.|
|**Transceiver (Ricetrasmettitore RF)**|Antenna e circuito radio a radiofrequenza (RF) a basso consumo. È il componente hardware critico controllato direttamente dal livello MAC per la gestione dello stato della radio.|
|**Power Unit (Unità di Alimentazione)**|Batterie a capacità limitata (talvolta integrate con moduli di _energy harvesting_). Nella maggior parte delle installazioni, la sostituzione della batteria è operativamente o economicamente impossibile.|
|**Piattaforma Hardware di Riferimento**|Il **Mica2 mote** rappresenta la piattaforma hardware di ricerca classica su cui è stato originariamente implementato e validato il protocollo S-MAC.|
|**Architettura Software**|Sistema operativo event-driven **TinyOS** (sviluppato presso UC Berkeley), programmato nel linguaggio modulare **nesC**, ottimizzato per gestire concorrenza ed eventi hardware in pochi byte di memoria.|

### 1.3 Confronto: WSN vs. Reti Wireless Convenzionali

Le WSN differiscono in modo sostanziale dalle reti wireless tradizionali (Wi-Fi, cellulari, ad-hoc generiche). Mentre le reti tradizionali perseguono la massimizzazione delle prestazioni di picco per utente, le WSN sono plasmate dal paradigma dell'**ottimizzazione energetica ad ogni costo (energy-first paradigm)**.

|   |   |   |
|---|---|---|
|Aspetto|Reti Convenzionali|Wireless Sensor Networks (WSN)|
|**Obiettivo di Progettazione**|_General purpose_ (supporto a molteplici applicazioni e utenti)|Specifico per un'applicazione unica o un set limitato di compiti|
|**Interesse Principale**|Massimizzare Throughput, minimizzare la Latenza, garantire la QoS|**Efficienza Energetica**: estensione della vita utile della rete|
|**Modello di Traffico**|Flussi continui e arbitrari tra coppie di nodi|Molti-a-uno (_Many-to-one_), basso bitrate, a raffiche (_bursty_)|
|**Ambiente di Deploy**|Controllato, moderato, con infrastruttura fissa|Non presidiato, severo, dinamico, del tutto privo di infrastruttura|
|**Accessibilità Fisica**|Agevole (dispositivi ricaricabili o facilmente sostituibili)|Difficile, costosa o del tutto impossibile dopo il rilascio|
|**Gestione e Controllo**|Conoscenza globale, controllo centralizzato o guidato dall'utente|Decisioni strettamente localizzate, assenza di entità centralizzata|
|**Scala (Numero Nodi)**|Da decine a centinaia di dispositivi per cella|Da centinaia a migliaia di nodi, spesso distribuiti ad alta densità|

### 1.4 Pattern di Comunicazione, Opzioni di Deploy e Requisiti di Progetto

#### Pattern di Comunicazione

- **Convergecast (Many-to-one):** Flusso dati dominante generato dalla moltitudine di nodi sensore e indirizzato verso l'unico nodo sink per raccogliere letture sensoriali e parametri di stato.
- **Dissemination (One-to-all):** Flusso di controllo generato dal sink e diffuso verso tutti i nodi della rete per l'invio di query, riconfigurazioni o aggiornamenti del codice software.
- _Implicazioni per il MAC:_ Il traffico nelle WSN si presenta tipicamente **leggero, a raffiche (bursty) e temporalmente/spazialmente correlato**. Per la maggior parte del tempo la rete è inattiva. Inoltre, i nodi posizionati geograficamente più vicini al sink devono inoltrare un volume di traffico molto superiore rispetto ai nodi di periferia (_sink bottleneck_).

#### Opzioni di Deployment

- **Random (Casuale):** Nodi distribuiti in modo incontrollato (es. aviolancio da velivoli); si assume solitamente una distribuzione uniforme sull'area da monitorare.
- **Regular (Pianificato/Griglia):** Disposizione pianificata e deterministica, spesso modellata come una griglia geometrica regolare.
- **Mobile (Mobile):** Nodi dotati di movimento passivo (trascinati da vento o correnti d'acqua) o attivo (montati su droni o robot) per colmare buchi di copertura o esplorare aree di interesse.

#### Requisiti Chiave e Principi di Progettazione

- **Scalabilità e Densità:** Il protocollo deve operare in modo efficiente con un numero variabile di nodi, da topologie rade a scenari estremamente densi.
- **Tolleranza ai Guasti (Fault Tolerance):** Capacità della rete di sopravvivere al cedimento di singoli nodi per esaurimento della batteria o danno fisico.
- **Topologia Dinamica:** Gestione continua dei cambiamenti topologici causati da guasti, cicli di _sleep/wake_ e mobilità.
- **In-network Processing:** Elaborazione locale ed aggregazione dei dati lungo il percorso multi-hop per ridurre al minimo i bit trasmessi via radio.
- **Data-Centric Networking:** La comunicazione si concentra sull'interrogazione dei _dati_ associati a una regione geografica o evento, indipendentemente dagli ID univoci dei singoli nodi.
- **Principi di Località:** Risoluzione dei problemi di coordinamento e instradamento tramite interazioni esclusive tra nodi adiacenti (vicinato locale).

### 1.5 Caso di Studio: Golden Gate Bridge

**Caso di Studio Realizzativo (Stanford, 2005):** Installazione sperimentale di una WSN composta da **64 nodi sensore** disposti lungo un percorso multi-hop esteso fino a **46 hop** per il monitoraggio sincrono e continuo delle vibrazioni strutturali ambientali del ponte. Il deployment è stato completato ed eseguito **senza interrompere l'operatività o il traffico stradale della struttura**.

## 2. Vincoli Energetici e Sprechi Energetici al Livello MAC

### 2.1 Profilo dei Consumi del Radio Transceiver (Chip CC2420)

Il livello di collegamento dati (MAC) controlla direttamente lo stato operativo del modulo radio. Analizzando i dati di assorbimento reale del ricetrasmettitore **Texas Instruments CC2420** (utilizzato su nodi TelosB e MICAz), si evince la struttura dei consumi nelle varie modalità operative:

|   |   |   |
|---|---|---|
|Modalità Operativa|Assorbimento di Corrente (mA)|Descrizione dello Stato Hardware|
|**Transmit (****0\text{ dBm}****)**|**17.4\text{ mA}**|Il radio invia dati sul canale wireless alla potenza di 0\text{ dBm}.|
|**Receive / Listen**|**18.8\text{ mA}**|Il radio ascolta il canale, **anche in totale assenza di traffico**.|
|**Idle**|**0.43\text{ mA}**|Circuiteria RF disattivata, oscillatore di clock mantenuto acceso.|
|**Sleep**|**0.02\text{ mA}**|Modulo radio in power-down completo (circuiteria e clock spenti).|

**La Regola d'Oro del Risparmio Energetico nelle WSN:** **Ascoltare il canale (****18.8\text{ mA}****) costa quanto ricevere e più che trasmettere (****17.4\text{ mA}****). L'UNICO MODO PER RISPARMIARE ENERGIA È SPEGNERE LA RADIO.** Esistono circa tre ordini di grandezza di differenza nell'assorbimento tra lo stato di _Listen_ e lo stato di _Sleep_.

### 2.2 Le Cinque Fonti di Spreco Energetico nel MAC

1. **Idle Listening (Ascolto Inoperoso):** È **la causa dominante in assoluto** di spreco energetico. Consiste nel mantenere il radio in ascolto su un canale vuoto in attesa di eventuale traffico. In scenari a basso carico, consuma la quasi totalità del budget energetico.
2. **Collisions (Collisioni):** Si verificano quando due o più trasmissioni sovrapposte interferiscono sul ricevitore, corrompendo i pacchetti. Richiedono la scarto e la ritrasmissione del frame, sprecando l'energia sia dell'invio fallito sia di quello successivo.
3. **Overhearing (Ascolto Indebito):** Si verifica quando un nodo riceve e decodifica integralmente pacchetti dati destinati ad altri nodi appartenenti al suo vicinato.
4. **Control Overhead (Overhead di Controllo):** Invio di frame di controllo (RTS, CTS, ACK, SYNC) che non trasportano payload utile. Nelle WSN, l'overhead dei pacchetti di controllo è spesso pari o superiore alla dimensione dei dati sensoriali.
5. **Overemitting (Emissione Eccessiva):** Tentativo di trasmissione eseguito quando il destinatario non è pronto a ricevere (ad esempio perché si trova nello stato di sleep o impegnato in un altro compito).

## 3. Il Protocollo CSMA/CA (IEEE 802.11 DCF) e Limiti per le WSN

### 3.1 Meccanismo DCF e Spazi Inter-Frame (IFS)

Il meccanismo **Distributed Coordination Function (DCF)** di IEEE 802.11 si basa sull'accesso al canale tramite _Carrier Sense Multiple Access with Collision Avoidance_ (CSMA/CA).

#### Vincolo Fisico del Canale Wireless

A differenza delle reti cablate (es. Ethernet CSMA/CD), **un ricetrasmettitore wireless non può rilevare le collisioni durante la trasmissione** (_Collision Detection_ impossibile). La potenza del segnale emesso dalla propria antenna sopraffà e satura il circuito di ricezione locale, rendendo invisibile qualsiasi altro segnale interferente. Per questo motivo, il protocollo deve operare per _prevenire_ preventivamente le collisioni (_Collision Avoidance_).

#### Funzionamento del DCF

- **Canale Libero:** Se il canale viene rilevato continuamente libero per un intervallo temporale pari a **DIFS**, il nodo trasmette immediatamente.
- **Canale Occupato:** Se il canale è occupato, il nodo rinvia la trasmissione (_deferral_), attende la fine dell'occupazione corrente, attende un intervallo DIFS ed estrae un valore di **backoff casuale** misurato in slot time.
- **ACK e SIFS:** Se la trasmissione ha successo, il ricevitore invia un riscontro positivo (**ACK**) dopo un intervallo breve **SIFS**.

Le priorità di accesso al canale sono determinate dalla lunghezza degli intervalli **Inter-Frame Spaces (IFS)**:

|   |   |   |
|---|---|---|
|Inter-Frame Space (IFS)|Durata Relativa|Utilizzo Primario e Priorità|
|**SIFS** (Short IFS)|Il più breve|Riservato per ACK, CTS e risposte immediate. Garantisce priorità assoluta per completare la transazione in corso.|
|**PIFS** (PCF IFS)|SIFS + 1 slot time|Utilizzato dal Point Coordinator nell'accesso centralizzato ad opzione polling.|
|**DIFS** (DCF IFS)|SIFS + 2 slot time|Utilizzato per le trasmissioni dati asincrone ordinarie in contesa.|

_Regola della Priorità:_ Un intervallo IFS più breve garantisce un accesso prioritario al canale, impedendo ad altri nodi di intercettarlo durante uno scambio critico (es. DATA-ACK).

### 3.2 Carrier Sensing (CCA), Vulnerable Window e Backoff Esponenziale

#### Clear Channel Assessment (CCA)

La procedura di **Clear Channel Assessment (CCA)** determina se il canale wireless è libero o occupato basandosi su tre criteri fisici specifici definiti a livello PHY:

1. **Rilevamento di Energia:** Verificare se la potenza RF misurata sul canale supera una soglia energetica prefissata (E > E_{th}).
2. **Rilevamento del Preambolo:** Identificare la presenza di una struttura di preambolo radio valida corrispondente ad un frame compatibile.
3. **Combinazione di Entrambi:** Verificare la contemporanea presenza di energia sopra soglia e di un preambolo valido.

#### Vulnerable Window e Collisioni

Nonostante l'esecuzione del CCA, le collisioni si verificano comunque per due motivi:

1. Due o più stazioni estraggono casualmente lo stesso identico valore di backoff.
2. **Vulnerable Window (Finestra di Vulnerabilità):** Ha la durata di **1\text{ slot time}** (intervallo che copre il tempo di rilevamento CCA, la commutazione della radio da Rx a Tx e il tempo di propagazione). Un frame iniziato da meno di uno slot time non è ancora rilevabile dagli altri nodi, che vedono il canale ingannevolmente libero e iniziano a trasmettere, collidendo.

#### Binary Exponential Backoff (BEB)

1. Per ogni pacchetto da trasmettere, il nodo estrae un intero casuale nell'intervallo [0, CW-1] slot time, dove inizialmente CW = CW_{min}.
2. Il contatore viene decrementato ad ogni slot time in cui il canale rimane libero; viene **congelato** se il canale diventa occupato e ripreso al termine del successivo DIFS.
3. Se l'ACK non perviene (collisione), la finestra di contesa raddoppia (CW_{nuova} = 2 \times CW_{vecchia}), fino al valore massimo CW_{max}.
4. A seguito di una trasmissione completata con successo, CW viene resettata al valore minimo CW_{min}.

### 3.3 Problemi dei Terminali Nascosti ed Esposti e Meccanismo RTS/CTS (NAV)

#### Analisi Comparativa delle Anomalie Topologiche

```text
[ Terminale Nascosto ]
  (A) ──────► (B) ◄────── (C)        (D)
  - A sta trasmettendo a B.
  - C non sente A (fuori dalla portata radio di A).
  - C esegue il carrier sense, valuta il canale libero ed avvia la trasmissione verso B.
  - ESITO: Collisione catastrofica sul nodo B.

[ Terminale Esposto ]
  (A) ◄────── (B)         (C) ──────► (D)
  - B sta trasmettendo ad A.
  - C sente la trasmissione di B tramite il carrier sense.
  - C rinviasse la sua trasmissione verso D pensando che il canale sia occupato.
  - ESITO: Differimento inutile; C potrebbe trasmettere a D senza interferire con A.
```

#### Soluzione RTS/CTS e Virtual Carrier Sensing

- **Handshake RTS/CTS:** Il mittente invia un breve frame **RTS** (_Request to Send_); il destinatario risponde con un breve frame **CTS** (_Clear to Send_).
    - _Risoluzione del Terminale Nascosto:_ Il nodo C sente il CTS emesso da B (che contiene la durata della trasmissione) e blocca i propri tentativi di invio per quel lasso di tempo.
    - _Risoluzione del Terminale Esposto:_ Il nodo C sente l'RTS inviato da B ma non il CTS inviato da A; deduce di poter trasmettere verso D senza generare interferenze su A.
- **Virtual Carrier Sensing e NAV:** Ogni frame (RTS, CTS, DATA) trasporta un campo _Duration_. I nodi adiacenti leggono tale valore e impostano il proprio **Network Allocation Vector (NAV)**, un timer interno di occupazione virtuale del canale. Finché NAV > 0, il canale viene considerato occupato anche se il CCA non rileva energia radio.

### 3.4 Inadeguatezza del CSMA/CA Tradizionale per i Nodi Sensore

L'applicazione diretta del CSMA/CA di 802.11 sulle WSN genera una perdita di efficienza inaccettabile. Oltre al costo in byte dei frame di controllo rispetto ai piccolissimi payload dei sensori, **ogni frame di controllo richiede una grave penalizzazione temporale ed energetica dovuta all'invio del preambolo PHY e ai tempi di commutazione della radio (****t_{SIFS}****)**.

#### Calcolo Numerico dell'Efficienza di Trasmissione (Byte sull'aria)

Assumendo frame standard 802.11 (RTS = 20 B, CTS = 14 B, Header MAC + FCS = 28 B, ACK = 14 B, Payload sensore = 30 B):

- **Basic Access (Senza RTS/CTS):** \text{Byte Totali} = \text{Header (28 B)} + \text{Payload (30 B)} + \text{ACK (14 B)} = 72\text{ Byte} \text{Efficienza del Payload} = \frac{30\text{ B}}{72\text{ B}} \approx \mathbf{42\%}
- **Accesso con RTS/CTS Handshake:** \text{Byte Totali} = \text{RTS (20 B)} + \text{CTS (14 B)} + \text{Header (28 B)} + \text{Payload (30 B)} + \text{ACK (14 B)} = 106\text{ Byte} \text{Efficienza del Payload} = \frac{30\text{ B}}{106\text{ B}} \approx \mathbf{28\%}

#### Quattro Motivi per cui CSMA/CA è Inadatto alle WSN

1. **Radio Sempre In Ascolto (Idle Listening):** I nodi mantengono la radio accesa tra una trasmissione e l'altra, prosciugando la batteria anche senza traffico.
2. **Overhearing Sistematico:** Non essendoci meccanismi di spegnimento coordinato, tutti i nodi adiacenti decodificano il traffico diretto altrui.
3. **Elevato Control Overhead:** RTS, CTS e ACK introducono ritardi fisici (t_{SIFS}, preamboli PHY) ed un numero di byte paragonabile all'intero payload.
4. **Obiettivi di Progetto Errati:** 802.11 è ottimizzato per garantire _throughput_ ed _equità per nodo (fairness)_, concetti secondari nelle WSN dove l'unica metrica critica è l'estensione della vita della rete.

## 4. Analisi Dettagliata del Protocollo S-MAC

### 4.1 Filosofia di Progetto e Duty Cycling

Il protocollo **S-MAC (Sensor-MAC)** (Ye, Heidemann, Estrin - INFOCOM 2002 / IEEE/ACM ToN 2004) introduce un'architettura progettata per abbattere le fonti di spreco energetico.

**Filosofia Fondamentale di S-MAC:** Scambiare parte della latenza e dell'equità (fairness) per ottenere la massima **efficienza energetica** ed estendere la vita della rete.

Il pilastro concettuale di S-MAC è il **Duty Cycling**: la radio viene spenta periodicamente ed accesa soltanto in brevi finestre temporali concordate tra nodi adiacenti.

\text{Frame S-MAC } (T_{frame}) = \text{Intervallo Listen } (T_{listen}) + \text{Intervallo Sleep } (T_{sleep})

\text{Duty Cycle} = \frac{T_{listen}}{T_{frame}}

- _Esempio:_ Con T_{listen} = 200\text{ ms} e T_{sleep} = 2\text{ s}, il duty cycle è circa il 10\%, consentendo di spegnere la radio per il 90\% del tempo operativo.

### 4.2 Selezione e Diffusione degli Schedule e Tabella degli Schedule

Per consentire la comunicazione tra nodi in duty cycling, i vicini geografici sincronizzano i propri orari di ascolto formando cluster virtuali basati sulla condivisione dello **Schedule**.

#### Procedura per la Selezione e Diffusione dello Schedule

1. **Ascolto Iniziale:** Un nuovo nodo si attiva e rimane in ascolto per un intervallo temporale continuo per rilevare eventuali pacchetti di sincronizzazione (**SYNC**).
2. **Creazione dello Schedule (Synchronizer):** Se non sente alcun SYNC, il nodo sceglie uno schedule casuale, stabilisce il proprio orario e trasmette un pacchetto SYNC.
3. **Adozione dello Schedule (Follower):** Se riceve un SYNC prima di aver scelto, adotta lo schedule letto e lo trasmette a sua volta per estenderlo.
4. **Adozione Multipla (Border Node):** Se successivamente sente un secondo schedule differente inviato da un altro vicino, adotta **entrambi** gli schedule. Il nodo diventa un **Border Node** e deve rimanere sveglio negli intervalli _Listen_ di entrambi gli schedule, trasmettendo i propri pacchetti broadcast due volte.

#### Estratto della Schedule Table di un Nodo

Ogni nodo registra gli orari d'ascolto dei vicini all'interno di una tabella locale:

|   |   |   |
|---|---|---|
|Nodo Vicino|Schedule Adottato|Stato del Nodo Locale|
|**Nodo A**|Schedule 1|Follower di Schedule 1|
|**Nodo B**|Schedule 1|Follower di Schedule 1|
|**Nodo E**|Schedule 2|**Border Node** (Registra molteplici schedule attivi)|

### 4.3 Meccanismo dei Minislot

Se tutti i nodi di uno stesso cluster si svegliassero ed emettessero un frame RTS contemporaneamente all'inizio dell'intervallo di _Listen_, si verificherebbero collisioni sistematiche. Per evitare questo fenomeno, la fase di contesa all'interno dell'intervallo di _Listen_ è suddivisa in una sequenza di **minislots**.

**Aspetto Fondamentale:** Il meccanismo dei minislot viene utilizzato **due volte all'interno di ogni frame S-MAC**:

1. Una prima volta durante la sottofase **SYNC** (per la trasmissione dei pacchetti di sincronizzazione).
2. Una seconda volta durante la sottofase **RTS/CTS** (per la contesa e la prenotazione del canale dati).

```text
Struttura dell'Intervallo Listen (Fase RTS/CTS)
┌───────────┬───────────┬───────────┬───────────┬───────────┬───────────┐
│ Minislot 1│ Minislot 2│ Minislot 3│ Minislot 4│ Minislot 5│ Minislot 6│ ...
└───────────┴───────────┴───────────┴───────────┴───────────┴───────────┘
```

#### Algoritmo dei Minislot

1. Un nodo che intende trasmettere estrae casualmente un indice di minislot (es. Nodo A sceglie il Minislot 2, Nodo B sceglie il Minislot 5) e rimane in solo ascolto durante i minislot precedenti.
2. **Il Valore Minore Vince la Contesa:**
    - Al Minislot 2, il Nodo A rileva il canale libero ed emette il proprio frame RTS.
    - Al Minislot 5, il Nodo B esegue il carrier sensing, rileva che il canale è già stato occupato dall'RTS di A: **desiste immediatamente, rinuncia ad inviare e si spegne (sleep)**, rimandando il tentativo al frame successivo.

### 4.4 Evitamento dell'Overhearing e NAV Sleep Timers

S-MAC converte il parametro logico NAV di 802.11 in un vero e proprio **Sleep Timer hardware**:

- **Regola dell'Overhearing:** Qualsiasi nodo adiacente che intercetta un pacchetto RTS o CTS **non indirizzato a se stesso** legge il campo _Duration_ contenuto nel frame ed entra immediatamente in modalità **Sleep** per tutta la durata specificata dal NAV.

#### Stato dei Nodi durante uno Scambio Unicast

|   |   |   |
|---|---|---|
|Identificatore Nodo|Ruolo Operativo|Stato della Radio durante la Trasmissione|
|**Nodo A**|Mittente Unicast|**Sveglio:** Invia RTS, riceve CTS, invia DATA, riceve ACK.|
|**Nodo B**|Destinatario Unicast|**Sveglio:** Riceve RTS, invia CTS, riceve DATA, invia ACK.|
|**Nodo C**|Adiacente al mittente A (sente RTS)|**Sleep:** Spegne la radio per tutta la durata del NAV estratto dall'RTS.|
|**Nodo D**|Adiacente al destinatario B (sente CTS)|**Sleep:** Spegne la radio per tutta la durata del NAV estratto dal CTS.|
|**Nodi E, F**|Nodi fuori portata radio|**Liberi:** Proseguono secondo i propri cicli ordinari.|

### 4.5 Message Passing per Messaggi Lunghi

La trasmissione di un blocco dati esteso tramite piccoli frammenti tradizionali genererebbe un sovrappiù di contese energetiche ed overhead se preceduta da RTS/CTS singoli per ogni frammento. S-MAC risolve il problema adottando il meccanismo del **Message Passing**:

1. Una singola stretta di mano RTS/CTS riserva il canale wireless per un'intera **raffica (burst)** di frammenti di dati.
2. Il messaggio viene suddiviso in piccoli frammenti gestiti individualmente: se un frammento viene corrotto da un errore di canale, viene ritrasmesso solo quel singolo elemento tramite il suo ACK mancante.
3. **Campo Duration Progressivo:** Ogni frammento e ciascun ACK intermedio trasportano la **durata residua** dell'intera trasmissione. In questo modo, eventuali nodi che dovessero svegliarsi temporaneamente aggiornano correttamente il proprio NAV e tornano subito a dormire.

```text
[Mittente]   ──RTS──►        ──Frag1──►        ──Frag2──►        ──Frag3──►
[Ricevitore] ◄──CTS──        ◄──ACK1───        ◄──ACK2───        ◄──ACK3───
             |<───────────── Unica Prenotazione NAV Complessiva ───────────>|
```

### 4.6 Estensione: Adaptive Listening per la Riduzione della Latenza

S-MAC con schedule fisso introduce un ritardo accumulato significativo: ogni salto multi-hop richiede tipicamente l'attesa di un intero frame S-MAC (T_{frame}). Per mitigare tale vincolo, Ye et al. hanno introdotto l'estensione **Adaptive Listening**:

- **Meccanismo:** Un nodo che intercetta un RTS o CTS destinato ad altri entra in sleep, ma programma un risveglio temporaneo **esattamente al termine della trasmissione corrente**. Se il pacchetto appena concluso era diretto a lui come hop successivo, il nodo è già sveglio ed in grado di riceverlo immediatamente senza dover attendere il periodo _Listen_ del frame successivo.
- **Trade-off / Costo Energetico:** L'Adaptive Listening introduce un piccolo costo marginale: **i nodi vicini che non si trovano sul percorso di instradamento si svegliano inutilmente per un breve intervallo al termine della trasmissione (**_**a little extra listening**_**)**.

#### Analisi della Latenza Multi-Hop (N Hop)

- **Standard Always-On (802.11):** D(N) \approx N \cdot (t_{cs} + t_{tx})
- **S-MAC Scheda Fissa (Senza Adaptive Listening):** D(N) \approx N \cdot T_{frame}
- **S-MAC con Adaptive Listening:** D(N) \approx N \cdot \frac{T_{frame}}{2}

_Valutazione nel Caso del Golden Gate Bridge (46 Hop con_ _T_{frame} = 1\text{ s}__):_

- Latenza S-MAC fisso: \approx 46\text{ secondi}
- Latenza S-MAC con Adaptive Listening: \approx 23\text{ secondi}

### 4.7 Valutazione Prestazionale e Risultati Sperimentali

#### Configurazione del Test di Laboratorio

- **Topologia:** Rete multi-hop a 2 salti composta da 2 sorgenti (A, B), 2 sink (D, E) ed 1 nodo relay centrale (C).
- **Volume Dati:** 200 pacchetti complessivi (10 messaggi composti da 10 frammenti di 40 byte per ciascuna sorgente).
- **Variabile Indipendente:** Periodo di inter-arrivo dei messaggi da 1\text{ s} (carico pesante) a 10\text{ s} (carico leggero).

#### Risultati ed Evidenze Sperimentali

- **A Basso Carico (Messaggi radi, inter-arrivo largo):** Il meccanismo di **Periodic Sleep** è determinante. S-MAC riduce il consumo energetico fino a **3 volte** rispetto ad un protocollo CSMA/CA senza cicli di sleep.
- **Ad Alto Carico (Messaggi frequenti):** L'idle listening è ridotto. I risparmi energetici di S-MAC derivano principalmente dall'**Overhearing Avoidance** e dal **Message Passing**. In ogni regime di carico, S-MAC garantisce un'efficienza energetica nettamente superiore a 802.11.

## 5. Protocolli di Routing nelle WSN

### 5.1 Tassonomia delle Strategie di Routing

I protocolli di instradamento nelle WSN si classificano in base al momento in cui le rotte vengono calcolate, alla modalità di identificazione dei destinatari ed all'organizzazione della topologia di rete.

```text
TASSONOMIA DEL ROUTING NELLE WSN
├── Tempistica di Calcolo della Rotta
│   ├── Proattivo (Table-driven) -> Esempio: DSDV
│   ├── Reattivo (On-demand) -> Esempi: Flooding, Gossiping, DSR, AODV
│   ├── Ibrido -> Esempio: ZRP
│   └── Geografico (Position-based) -> Esempi: Most Forward within r, Geocasting
├── Organizzazione della Topologia
│   ├── Piatta (Flat) -> Tutti i nodi svolgono identiche funzioni
│   └── Gerarchica -> Presenza di coordinatori (Cluster Head)
└── Identificazione dei Nodi
    ├── Basata su Indirizzo Univoco (ID-based)
    └── Basata su Posizione / Dati (Position-based / Data-centric)
```

|   |   |   |
|---|---|---|
|Categoria Routing|Meccanismo Principale|Esempi Principali|
|**Proattivo (Table-driven)**|Mantiene rotte sempre aggiornate in tabelle prima che siano necessarie.|DSDV|
|**Reattivo (On-demand)**|Trova un percorso soltanto al momento dell'invio di un pacchetto.|Flooding, Gossiping, DSR, AODV|
|**Ibrido**|Combina comportamenti proattivi a breve raggio e reattivi a lungo raggio.|ZRP|
|**Geografico (Position-based)**|Inoltro basato esclusivamente sulle coordinate spaziali dei nodi.|Most Forward within r, Geocasting|

### 5.2 Routing Proattivo Table-Driven: Protocollo DSDV

Il protocollo **DSDV (Destination-Sequenced Distance Vector)** è un algoritmo proattivo basato sull'equazione di Bellman-Ford distribuita.

- **Sequence Numbers:** Ogni voce della tabella contiene un numero di sequenza univoco generato direttamente dalla destinazione. Un numero di sequenza **più alto** indica una rotta più recente (fresca), garantendo l'eliminazione dei loop di routing (_loop-free property_).

#### Traccia Didattica della Convergenza DSDV (Rete a 4 Nodi: A-B-C-D)

La topologia è costituita da 4 nodi e 3 link con costo unitario equalizzato (1\text{ hop}): A-B, A-C, C-D.

```text
  (B)
   │ (costo 1)
   │
  (A) ──── (C) ──── (D)
    (costo 1)  (costo 1)
```

##### Step 1: Stato Iniziale (Conoscenza locale dei soli vicini diretti a distanza 1)

- **Tabella di A:** `B: 1`, `C: 1`, `D: ∞`
- **Tabella di B:** `A: 1`, `C: ∞`, `D: ∞`
- **Tabella di C:** `A: 1`, `B: ∞`, `D: 1`
- **Tabella di D:** `A: ∞`, `B: ∞`, `C: 1`

##### Step 2: A diffonde il proprio vettore di distanza ai vicini B e C

- B e C ricevono il vettore da A (`B:1, C:1, D:∞`) ed incrementano le distanze di 1.
- **Tabella di B (aggiornata):** `A: 1`, `**C: 2**` (via A), `D: ∞`
- **Tabella di C (aggiornata):** `A: 1`, `**B: 2**` (via A), `D: 1`

##### Step 3: C diffonde il proprio vettore aggiornato ad A e D

- A e D ricevono il vettore da C (`A:1, B:2, D:1`) ed incrementano le distanze di 1.
- **Tabella di A (aggiornata):** `B: 1`, `C: 1`, `**D: 2**` (via C)
- **Tabella di D (aggiornata):** `**A: 2**` (via C), `**B: 3**` (via C), `C: 1`

##### Step 4: Stato di Convergenza delle Tabelle di Routing

|   |   |   |   |   |
|---|---|---|---|---|
|Nodo Sorgente|Distanza e Hop per A|Distanza e Hop per B|Distanza e Hop per C|Distanza e Hop per D|
|**Tabella di A**|—|1 (Diretto)|1 (Diretto)|**2** (via C)|
|**Tabella di B**|1 (Diretto)|—|**2** (via A)|**3** (via A)|
|**Tabella di C**|1 (Diretto)|**2** (via A)|—|1 (Diretto)|
|**Tabella di D**|**2** (via C)|**3** (via C)|1 (Diretto)|—|

### 5.3 Routing Reattivo On-Demand: Flooding, Gossiping, DSR e AODV

#### Critica alle Strategie di Inoltro Semplici (Flooding e Gossiping)

1. **Implosion (Implosione):** Nodi vicini inviano copie duplicate dello stesso pacchetto al medesimo destinatario, intasando la rete.
2. **Overlap (Sovrapposizione):** Nodi adiacenti che coprono la medesima regione geografica inviano dati identici.
3. **Resource Blindness (Incuria delle Risorse):** I protocolli ignorano l'energia residua dei nodi, esaurendo i nodi critici.

#### DSR (Dynamic Source Routing)

- **Route Discovery:** Quando un nodo sorgente deve trasmettere verso una destinazione priva di rotta, emette in broadcast un pacchetto **RREQ (Route Request)** con ID unico. Ogni nodo intermedio **aggiunge il proprio indirizzo nell'header** del pacchetto RREQ e lo rilancia.
- **Route Reply (RREP):** Il nodo destinatario riceve l'RREQ ed invia un pacchetto **RREP** contenente il percorso completo registrato, sfruttando il _source routing_.
- **Route Maintenance (Manutenzione della Rotta):** Se un link si interrompe durante la trasmissione, il nodo rileva l'errore e tenta di utilizzare **un'altra rotta alternativa memorizzata nella propria** _**Route Cache**_ o riavvia la procedura di discovery. Il mantenimento di molteplici rotte memorizzate velocizza il recupero dai guasti.

#### AODV (Ad hoc On-Demand Distance Vector)

Alternativa reattiva a DSR. Anziché trasportare l'intero percorso nell'header di ogni pacchetto dati, AODV stabilisce puntatori di inoltro _next-hop_ direttamente nelle tabelle dei nodi intermedi durante la fase RREQ/RREP.

#### Tabella Comparativa dei Protocolli Reattivi

|   |   |   |   |
|---|---|---|---|
|Algoritmo|Meccanismo di Inoltro|Vantaggio Principale|Svantaggio Critico|
|**Flooding**|Re-inoltro indiscriminato su tutte le interfacce fuorché quella di provenienza.|Garanzia di consegna se esiste un percorso; zero overhead di calcolo tabelle.|Implosione, Overlap e consumo energetico insostenibile.|
|**DSR**|Discovery on-demand (RREQ/RREP), _Source Routing_ nell'header e _Route Cache_.|Nessun overhead a riposo; _Route Maintenance_ rapida grazie a rotte multiple in cache.|Overhead dell'header crescente con il numero di hop.|
|**AODV**|Discovery on-demand con stato e tabelle _Next-Hop_ mantenute nei nodi intermedi.|Overhead dei dati fisso (header compatto senza catena di indirizzi).|Latenza iniziale di scoperta (_Discovery Delay_).|

### 5.4 Routing Geografico e Position-Based

Il **Geographic Routing** elimina la necessità di mantenere tabelle di instradamento globali. Le decisioni di inoltro vengono prese localmente confrontando la posizione geografica del nodo, dei suoi vicini e della destinazione.

#### Distinzione Concettuale: Geocasting vs. Position-Based Routing

- **Position-Based Routing:** Protocollo progettato per indirizzare ed instradare pacchetti ad un **singolo nodo destinazione specifico**, individuato dalle sue coordinate spaziali precise.
- **Geocasting:** Protocollo progettato per consegnare un messaggio a **qualsiasi nodo (o a tutti i nodi)** che si trovi all'interno di una **regione geografica definita** (es. inviare un'allerta a tutti i nodi nella zona X, Y).

#### Componenti e Meccanismi

- **Location Service:** Servizio di rete preposto a mappare l'ID di un nodo con le sue coordinate spaziali correnti.
- **Strategia Greedy (es. Most Forward within r):** Il nodo seleziona come prossimo hop il vicino geografico che riduce maggiormente la distanza euclidea verso la destinazione finale.
- **Il Problema dei Dead Ends (Vuoti Topologici):** Si verifica quando un nodo non possiede alcun vicino più vicino alla destinazione rispetto a se stesso (presenza di un buco di copertura o di un ostacolo), provocando il fallimento dell'inoltro greedy.

### 5.5 Analisi Comparativa Tradizionale delle Strategie di Routing

|   |   |   |
|---|---|---|
|Approccio|Punti di Forza (Strengths)|Costi e Limiti (Costs & Limits)|
|**Proattivo (Table-Driven)**|**Veloce:** Le rotte sono immediatamente disponibili senza ritardi di scoperta.|**Alto overhead costante:** Gli aggiornamenti periodici consumano banda e batterie anche in assenza di traffico.|
|**Reattivo (On-Demand)**|**Basso overhead a riposo:** Nessun aggiornamento trasmesso se non ci sono dati da inviare.|**Latenza iniziale:** Ritardo di accumulo (_Discovery Delay_) al momento del primo invio verso una destinazione.|
|**Geografico (Position-Based)**|**Massima scalabilità e risparmio energetico:** Eliminazione delle tabelle globali.|**Richiede dispositivi di localizzazione** (hardware/software) e gestione complessa dei _Dead Ends_.|