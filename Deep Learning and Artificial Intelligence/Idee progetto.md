# Rapporto di Ricerca: Progettazione Architetturale e Applicazioni Avanzate per Deep Learning

La presente disamina tecnica formula un ecosistema di proposte progettuali destinate a soddisfare i requisiti avanzati della ricerca accademica nel campo dell'apprendimento profondo, strutturate specificamente per il paradigma BYOP (Bring Your Own Project). L'indagine è stata condotta escludendo rigorosamente i domini applicativi non ammessi (quali robotica, previsione di serie temporali, finanza, applicazioni biomediche, graph learning, neural rendering e gaussian splatting) e concentrandosi sulle sedici aree di ricerca esplicitamente autorizzate. L'impianto teorico di ciascuna proposta si fonda sui principi matematici dell'ottimizzazione stocastica, della geometria dei manifold e dell'analisi di Fourier , al fine di garantire un'elevata profondità tecnica. L'obiettivo primario risiede nell'adattamento di architetture e metodologie presentate in pubblicazioni recenti (2024-2026) verso domini applicativi alternativi, promuovendo un paradigma di innovazione trasversale (cross-domain) che massimizzi il contributo scientifico mantenendo i costi computazionali entro soglie accessibili per la ricerca accademica individuale.

## Training Dynamics

L'analisi delle dinamiche di addestramento investiga l'evoluzione temporale della topologia della funzione di perdita e i fenomeni emergenti durante l'ottimizzazione, distaccandosi dall'analisi asintotica classica per comprendere le transizioni di fase.

### PRJ-01: Analisi dell'Edge of Stability nei Modelli di Diffusione Acustica

L'indagine del fenomeno denominato "Edge of Stability" (EoS) ha rivelato che, durante l'addestramento tramite discesa del gradiente, la nitidezza del paesaggio di perdita, misurata attraverso l'autovalore massimo della matrice Hessiana, tende a convergere e oscillare attorno al valore $2/\eta$, dove $\eta$ rappresenta il tasso di apprendimento. Storicamente, questo comportamento è stato studiato in reti feed-forward standard o modelli linguistici su dati testuali o visivi. La proposta attuale prevede la trasposizione di questa indagine teorica al dominio dell'elaborazione audio, specificamente all'interno dei modelli di diffusione per la sintesi sonora. L'addestramento di modelli di diffusione audio comporta la previsione di rumore su spettrogrammi complessi, introducendo sfide di convergenza uniche legate all'analisi di Fourier e alla coerenza di fase. Il progetto richiede l'implementazione di un modello di diffusione leggero (ad esempio un framework basato su U-Net ridotta) per la generazione di segnali acustici ambientali, tracciando sistematicamente lo spettro dell'Hessiana a intervalli regolari. L'innovazione risiede nel dimostrare se l'EoS si manifesti in modo identico quando l'ottimizzazione è applicata a rappresentazioni tempo-frequenza, permettendo di derivare uno scheduling del learning rate teoricamente fondato che stabilizzi il training senza richiedere risorse hardware proibitive.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Training dynamics|
|**Riassunto**|Trasposizione dell'analisi "Edge of Stability" per quantificare le dinamiche dell'Hessiana nei modelli di diffusione acustica.|
|**Input / Output**|**Input:** Spettrogrammi di rumore e time-steps. **Output:** Spettrogrammi denoisati e vettori traccianti gli autovalori dell'Hessiana.|
|**Costo Computazionale**|Basso. L'uso di reti shallow e la stima dell'Hessiana tramite "power iteration" limitano il consumo di memoria.|
|**Dataset**|Sottoinsieme di ESC-50 o Speech Commands (ridotti a bassa risoluzione).|

### PRJ-02: Emergenza del Grokking in Reti Mamba per Musica Simbolica

Un secondo fenomeno emergente di estrema rilevanza è il "grokking", caratterizzato da una generalizzazione algoritmica che si verifica molto tempo dopo il raggiungimento dell'overfitting sui dati di addestramento. Finora, la letteratura ha documentato il grokking quasi esclusivamente all'interno di architetture Transformer applicate a operazioni aritmetiche o logiche discrete. Questa proposta estende il dominio di indagine alle architetture State-Space, in particolare Mamba, applicate alla generazione di sequenze musicali simboliche. I modelli Mamba, basati su operatori di convoluzione e scansioni selettive, offrono un'elaborazione in tempo lineare , ma la loro capacità di interiorizzare la struttura algoritmica della teoria musicale (come le progressioni armoniche) non è stata formalmente studiata. Sostituendo i token matematici con token MIDI che rappresentano altezze e durate, il progetto addestrerà un modello Mamba su un corpus limitato di progressioni di accordi. L'analisi si concentrerà sull'osservazione delle matrici di transizione di stato interne, cercando di identificare l'esatto momento di addestramento (fase di transizione) in cui la rete cessa di memorizzare i pattern e inizia a generalizzare le regole dell'armonia tonale.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Training dynamics|
|**Riassunto**|Esplorazione del fenomeno del grokking nei modelli Mamba applicati alla teoria musicale simbolica (MIDI).|
|**Input / Output**|**Input:** Sequenze di token MIDI. **Output:** Predizione autoregressiva del token successivo e tracciamento della loss di validazione.|
|**Costo Computazionale**|Basso. I modelli State-Space ridotti su vocabolari simbolici ristretti si addestrano rapidamente su singola GPU.|
|**Dataset**|JSB Chorales (generazione simbolica di musica).|

## Model Merging

L'integrazione di molteplici reti neurali, originariamente addestrate su compiti disparati, all'interno di un unico modello senza necessità di ulteriori calcoli di ottimizzazione intensivi rappresenta la frontiera del model merging.

### PRJ-03: Binarizzazione T-Switch per Fusione di Encoder Audio

Il concetto di "Task Vector Arithmetic" e l'uso di operatori di binarizzazione hanno recentemente dimostrato che i parametri ridondanti possono essere filtrati mantenendo solo gli scalari essenziali, un meccanismo introdotto originariamente per i Vision Transformers attraverso il framework T-Switch. La presente proposta trasferisce l'intuizione del T-Switch (che decompone i vettori in maschere di attivazione, polarità e manopole di scala) dal dominio della visione artificiale a quello della rappresentazione acustica. Gli encoder audio come HuBERT e MERT vengono spesso addestrati separatamente per il riconoscimento vocale e la classificazione musicale. Questo progetto isolerà i task vectors di due encoder audio fine-tuned, applicherà la binarizzazione per mitigare i conflitti parametrici distruttivi nel panorama della funzione di perdita , e li fonderà. Tale traduzione di dominio non solo valida la teoria dei vettori binari su distribuzioni di attivazione altamente non stazionarie tipiche del segnale vocale, ma fornisce anche un metodo per distribuire modelli audio universali con un ingombro in memoria ridotto in maniera radicale.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Model merging|
|**Riassunto**|Applicazione del T-Switch per la binarizzazione dei task vectors finalizzata alla fusione a basso costo di encoder audio (speech e musica).|
|**Input / Output**|**Input:** Matrici dei pesi di modelli audio fine-tuned. **Output:** Singolo modello unificato capace di processare sia parlato che musica.|
|**Costo Computazionale**|Molto Basso. Il merging opera nello spazio dei pesi senza retropropagazione o utilizzo di tensori di dati massivi.|
|**Dataset**|Nessun addestramento richiesto; validazione su Common Voice (speech) e GTZAN (music).|

### PRJ-04: Essential Subspace Merging (ESM) per U-Net Generative

Un'ulteriore prospettiva di ricerca nel merging riguarda l'ottimizzazione del sottospazio e l'allineamento dei modelli prima della loro interpolazione, per evitare la perdita di connettività lineare dei modi. Un metodo di frontiera, Essential Subspace Merging (ESM), sfrutta l'Analisi delle Componenti Principali (PCA) per proiettare gli spostamenti dei pesi indotti dall'addestramento e combinare i modelli solo nei sottospazi ortogonali principali. Sebbene nato per i Large Language Models (LLM), questo progetto mira ad applicare la proiezione ortogonale ai modelli generativi convoluzionali per la sintesi visiva. Utilizzando reti di denoising (U-Net) addestrate su stili artistici differenti, il progetto calcolerà i baricentri sferici limitatamente ai sottospazi derivati dalla PCA. Il processo matematico garantisce che la fusione non distrugga le rappresentazioni spaziali di basso livello (edge detection, palette di colori), verificando se i principi algebrici che preservano la semantica testuale si applichino analogamente alla coerenza visiva dei manifold generativi.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Model merging|
|**Riassunto**|Utilizzo dell'Essential Subspace Merging (tramite PCA) per combinare modelli U-Net di diffusione addestrati su stili visivi differenti.|
|**Input / Output**|**Input:** Pesi di modelli U-Net pre-addestrati. **Output:** Modello fuso capace di generare immagini con un'interpolazione stilistica fluida.|
|**Costo Computazionale**|Basso. La decomposizione ai valori singolari (SVD/PCA) sui tensori dei pesi richiede memoria RAM, ma nessun ciclo di training GPU.|
|**Dataset**|Pesi pre-addestrati scaricati da HuggingFace (nessun dataset crudo necessario per il merging).|

## Multitask Learning

L'apprendimento multi-task si concentra sullo sfruttamento di rappresentazioni condivise per migliorare le prestazioni su compiti correlati, attenuando il fenomeno dell'interferenza e ottimizzando l'efficienza parametrica.

### PRJ-05: Apprendimento Multi-Task Senza Replay tramite Metrica LwP

Un avanzamento significativo nel settore è l'introduzione di tecniche come il Learning with Preserving (LwP), originariamente progettato per serie temporali multivariate, che impiega una funzione di perdita di conservazione della distanza ponderata dinamicamente per mitigare l'oblio catastrofico nei sistemi di apprendimento continuo multi-task senza fare affidamento su buffer di replay. Il progetto proposto intende estrarre questa metrica di conservazione della distanza e applicarla all'analisi di spettrogrammi audio. Il dominio di applicazione sarà il riconoscimento di eventi sonori ambientali sovrapposti (es. classificazione simultanea di sirene, parlato e traffico). La rete convoluzionale profonda apprenderà i task in sequenza, e la perdita LwP penalizzerà le deviazioni nelle rappresentazioni intermedie del manifold latente rispetto ai compiti già appresi. Questa traduzione architetturale verifica la scalabilità della metrica di preservazione dinamica su rappresentazioni spaziali 2D (spettrogrammi) , fornendo un framework multi-task estremamente efficiente per l'edge computing.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Multitask learning|
|**Riassunto**|Trasposizione della metrica LwP (Learning with Preserving) per addestrare classificatori di eventi audio multi-task senza utilizzare buffer di memoria storica.|
|**Input / Output**|**Input:** Spettrogrammi audio sequenziali. **Output:** Etichette multiple simultanee per gli eventi acustici rilevati.|
|**Costo Computazionale**|Basso. L'assenza di buffer di replay riduce drasticamente l'impronta di memoria durante il fine-tuning sequenziale.|
|**Dataset**|UrbanSound8K o ESC-50, partizionato per l'apprendimento incrementale.|

### PRJ-06: Routing Condizionato in Mixture of Experts Multimodali

Le architetture Mixture of Experts (MoE) rappresentano una frontiera per gestire l'eterogeneità dei gradienti nel multi-task learning, impiegando router differenziabili per instradare le attivazioni verso reti esperte specifiche. Una ricerca del 2024 ha proposto di migliorare il routing attraverso la modulazione dell'attributo duale in contesti linguistici. Questa idea può essere innovativamente tradotta nel dominio visivo-linguistico (Multimodal Multi-task Learning). Il progetto mira a costruire un Transformer ridotto dove i token derivanti da testo e da immagini sono elaborati in modo congiunto. Il modulo di routing valuterà non solo il contenuto semantico del token, ma anche la modalità di origine, indirizzando i flussi verso esperti specializzati nella comprensione visiva, nella sintassi o in compiti di fusione astratta. Si cercherà di dimostrare che un routing esplicitamente condizionato dalla modalità risolve la competizione dei gradienti molto più efficacemente rispetto ai modelli a densità fissa.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Multitask learning|
|**Riassunto**|Sviluppo di un Transformer MoE multimodale con meccanismo di routing condizionato per disaccoppiare task visivi e testuali.|
|**Input / Output**|**Input:** Coppie immagine-testo. **Output:** Risoluzione simultanea di Image Captioning e Visual Question Answering.|
|**Costo Computazionale**|Medio. L'impiego di una strategia sparse-MoE attiva solo una frazione dei parametri per ciascun forward pass.|
|**Dataset**|VQA v2 o sottoinsiemi ridotti di MS-COCO.|

## Artificial Life

La vita artificiale studia l'emergenza di dinamiche complesse e di auto-organizzazione tramite simulazioni differenziabili, tipicamente implementate attraverso architetture quali gli Automi Cellulari Neurali (NCA).

### PRJ-07: Evoluzione Audio-Reattiva in Automi Cellulari Multiagente (PD-NCA)

Il framework Automated Search for Artificial Life (ASAL) sfrutta i Vision-Language Models (VLM) pre-addestrati come funzione di fitness semantica per scoprire modelli di vita artificiale emergenti negli NCA, liberando la disciplina dalla necessità di regole scritte a mano. Il progetto in esame intende applicare il paradigma ASAL a un sistema Petri Dish NCA (PD-NCA), un substrato multagente in cui diverse popolazioni di automi coesistono e competono. L'innovazione si focalizza sull'estensione dell'ambiente verso l'interazione audio-reattiva. Le reti convoluzionali locali degli NCA saranno influenzate dinamicamente dall'ampiezza e dalla fase di un flusso audio di ingresso, mentre il VLM fornirà feedback linguistici (sotto forma di gradienti surrogati o metriche di selezione evolutiva) per premiare l'emergenza di strutture visive che "danzano" o si organizzano coerentemente con il ritmo musicale. Questo trasforma la vita artificiale in uno strumento per l'espressività artistica algoritmica.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Artificial life|
|**Riassunto**|Integrazione di modelli linguistico-visivi (VLM) per guidare l'evoluzione di automi cellulari PD-NCA condizionati da segnali audio in tempo reale.|
|**Input / Output**|**Input:** Flusso audio e stati iniziali stocastici delle celle. **Output:** Griglia di pixel (video) che esibisce morfogenesi auto-organizzante a ritmo di musica.|
|**Costo Computazionale**|Medio. La simulazione NCA è altamente parallelizzabile. Il VLM può essere interrogato a bassa frequenza tramite API o modelli quantizzati.|
|**Dataset**|File audio generici per il condizionamento; nessun dataset di training convenzionale necessario (apprendimento evolutivo guidato da VLM).|

### PRJ-08: Compressione di Spettrogrammi tramite Differentiable Logic CA

Per preservare l'essenza discreta della computazione negli ecosistemi artificiali, recenti lavori hanno introdotto i Differentiable Logic Cellular Automata (DiffLogic CA), che sostituiscono i pesi continui fluttuanti con porte logiche booleane differenziabili, creando modelli capaci di replicare esattamente il "Game of Life". Questo progetto propone di trasferire la topologia dei DiffLogic CA per eseguire compiti di compressione e restauro di spettrogrammi audio. Gli automi cellulari hanno già dimostrato efficacia nella compressione ultraleggera di immagini ; qui, l'obiettivo è istruire un sistema puramente booleano a evolvere nel tempo partendo da un "seme" compresso per rigenerare strutture bidimensionali che corrispondano alla Trasformata di Fourier a Breve Termine (STFT) di suoni ambientali. L'approccio promette un sistema di ricostruzione acustica che richiede risorse di esecuzione prossime allo zero, essendo basato esclusivamente su operazioni logiche discrete.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Artificial life|
|**Riassunto**|Sfruttamento di Differentiable Logic Cellular Automata per la modellazione e la ricostruzione generativa ultra-compressa di spettrogrammi audio.|
|**Input / Output**|**Input:** Vettore latente "seed" e regole logiche discrete. **Output:** Spettrogramma acustico espanso tramite l'iterazione delle regole locali.|
|**Costo Computazionale**|Basso. I modelli logici discreti aggirano il costo delle moltiplicazioni matriciali in virgola mobile.|
|**Dataset**|Un piccolo set di spettrogrammi derivati da FreeSound.|

## Optimization for Deep Learning

La ricerca nel campo dell'ottimizzazione mira a sviluppare algoritmi in grado di navigare in paesaggi di perdita complessi con maggiore stabilità, trasferibilità del tasso di apprendimento ed efficienza computazionale.

### PRJ-09: Ottimizzazione Whitening (SPlus) in Reti Convoluzionali

L'algoritmo SPlus (Stable Whitening Optimizer) si è distinto nel 2025 per aver risolto i problemi di divergenza del precondizionamento "Shampoo", impiegando un'aggiornamento limitato combinato con un'autoscalatura dipendente dalla forma della matrice e l'averaging delle iterazioni, garantendo un trasferimento lineare del learning rate indipendentemente dalla larghezza della rete. Il progetto propone di trasferire i vantaggi teorici dell'ottimizzazione SPlus nell'addestramento di architetture convoluzionali classiche adattate per compiti di classificazione di segnali biologici o ambientali. L'analisi matematica dei gradienti servirà a dimostrare se il precondizionamento di whitening mantiene i propri benefici in termini di tempo di clock e step stocastici in regimi dove i tensori dei pesi differiscono strutturalmente dalle matrici di proiezione dei Transformer, collaudando i limiti empirici della convergenza non-euclidea.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Optimization for deep learning|
|**Riassunto**|Applicazione e analisi dell'ottimizzatore SPlus (whitening preconditioning) in reti convoluzionali per valutare la stabilità e la scalabilità trasversale alle dimensioni.|
|**Input / Output**|**Input:** Tensori di attivazione in passaggi forward e backward. **Output:** Matrici di aggiornamento precondizionate per i pesi della rete, curve di discesa della loss.|
|**Costo Computazionale**|Basso. SPlus è progettato specificamente per ridurre il sovraccarico computazionale dei metodi del secondo ordine.|
|**Dataset**|CIFAR-100 o dataset analoghi per immagini/spettrogrammi.|

### PRJ-10: Stabilizzazione Minimax di GAN tramite Ortogonalizzazione del Gradiente

Una tecnica emergente definita "Trust-Region Optimization tramite Ortogonalizzazione Matriciale del Gradiente" impone che il gradiente venga proiettato ortogonalmente per arginare il degrado del segnale nelle reti molto profonde, sfruttando la norma spettrale per definire la regione di affidabilità dell'aggiornamento. Il presente progetto indaga l'applicabilità di questo approccio per la stabilizzazione delle Generative Adversarial Networks (GAN), notoriamente inclini al collasso dei modi e a gradienti instabili. Modificando l'ottimizzatore del discriminatore con l'ortogonalizzazione del gradiente matriciale, si mira a limitare le fluttuazioni asimmetriche durante il minimax game tra generatore e discriminatore. L'output teorico e pratico verificherà se la stabilizzazione della norma spettrale del gradiente riesca a produrre campioni generati (ad esempio frammenti audio o immagini sintetiche) di qualità superiore rispetto a varianti regolarizzate tramite tecniche di gradient penalty convenzionali.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Optimization for deep learning|
|**Riassunto**|Integrazione dell'ottimizzazione trust-region tramite ortogonalizzazione del gradiente per risolvere i problemi di stabilità minimax nelle GAN.|
|**Input / Output**|**Input:** Aggiornamenti del gradiente calcolati sulla loss avversaria. **Output:** Paesaggio di ottimizzazione stabilizzato che previene il collasso dei modi.|
|**Costo Computazionale**|Medio. Il calcolo della norma spettrale introduce operazioni algebriche aggiuntive rispetto all'SGD o Adam, facilmente ammortizzabili.|
|**Dataset**|CelebA (bassa risoluzione) o dataset di segnali audio 1D brevi.|

## Gaming and AI

L'integrazione di architetture intelligenti nei contesti videoludici va oltre la logica degli NPC, toccando ambiti di accessibilità, percezione spaziale e generazione procedurale dinamica basata sull'interpretazione degli input.

### PRJ-11: Feedback Acustico Spaziale per Accessibilità Videoludica (GamerAstra)

Il framework "GamerAstra" ha recentemente dimostrato come i modelli linguistico-visivi (VLM) e i sistemi multi-agente possano analizzare gli elementi visivi dei giochi in tempo reale per fornire feedback descrittivi, mentali e sonori a giocatori non vedenti (Blind and Low-Vision, BLV). Questa proposta progetta un'architettura derivata che bypassa il pesante ragionamento linguistico dei VLM, utilizzando invece un encoder visivo (come DINOv2) accoppiato a una politica di apprendimento per rinforzo (RL) e algoritmi di "sonification" generativa. L'agente neurale opererà nello spazio latente estraendo feature topologiche e convertendole, mediante layer di regressione non lineare , in coordinate spaziali per un motore audio 3D. Ciò permetterà una mappatura istantanea, senza latenza semantica, dell'ambiente circostante, traducendo dinamicamente ostacoli e pathfinding in cues uditivi continui.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Gaming and AI|
|**Riassunto**|Sostituzione dei VLM con reti RL guidate da encoder visivi nello spazio latente per generare feedback acustico spazializzato a latenza ultra-bassa per l'accessibilità.|
|**Input / Output**|**Input:** Frame visivi crudi catturati dal gameplay. **Output:** Coordinate continue mappate a un motore di sintesi audio posizionale.|
|**Costo Computazionale**|Medio. L'inferenza di un encoder visivo leggero e di una policy RL richiede modeste capacità computazionali, adattabili per girare in parallelo al motore grafico.|
|**Dataset**|Ambienti simulati Gym (es. MiniGrid o Atari).|

### PRJ-12: Modulazione del Movimento Disaccoppiato per Animazione Procedurale (DiT)

Nell'ambito della generazione dinamica, l'architettura Diffusion Transformer (DiT), ampiamente affermata nella creazione di video ad alta risoluzione , può rivoluzionare l'esperienza dei giochi in 2D. Il progetto "Decoupled Motion Modulation" si basa sulle tecniche recenti di modulazione vettoriale che separano le direttive cinetiche dall'aspetto visivo nei modelli di diffusione. L'idea è applicare un piccolo modello DiT condizionato all'interno di un loop di rendering videoludico per generare animazioni di sprite procedurali (effetti di magia, esplosioni, alterazioni atmosferiche) dipendenti dai parametri fisici istantanei del gioco. Il sistema accetterà in input vettori rappresentanti direzione, forza e contesto, e modificherà iterativamente il rumore per restituire una sequenza di frame pixel-art, testando l'affidabilità delle reti generative come middleware grafici real-time.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Gaming and AI|
|**Riassunto**|Uso di Diffusion Transformers leggeri e condizionati per la sintesi procedurale in tempo reale di sprite 2D e animazioni particellari nei videogiochi.|
|**Input / Output**|**Input:** Vettori di stato del gioco (variabili fisiche) e prompt latenti. **Output:** Sequenze di frame 2D animati coerenti con le dinamiche in-game.|
|**Costo Computazionale**|Alto in fase di training, ma l'impiego di tecniche di step-distillation renderà l'inferenza compatibile con scenari semi-realtime.|
|**Dataset**|Dataset personalizzato derivante dall'estrazione di sprite sheet da repository open-source.|

## Deep Learning Theory

La teoria del Deep Learning si preoccupa di formulare i fondamenti logico-matematici che giustificano il successo, o spiegano i fallimenti, delle metodologie empiriche adottate nella progettazione e integrazione delle reti neurali.

### PRJ-13: Risoluzione della Vanishing Feature nel Merging Acustico

Il fenomeno della "Vanishing Feature", identificato recentemente nella ricerca sul model merging, descrive come le rappresentazioni interne (le feature) indotte dagli input tendano a diminuire progressivamente propagandosi lungo i livelli di un modello fuso, portando a un drastico collasso della varianza e al conseguente crollo delle prestazioni. Il progetto in questione esplorerà questo paradigma matematico estendendolo ai modelli di elaborazione audio convoluzionale. Attraverso il teorema di approssimazione universale e l'analisi del prodotto scalare tra le attivazioni interne di due reti acustiche pre-addestrate su domini fonetici separati , l'esperimento cercherà di quantificare la degradazione dell'energia spettrale retropropagata. L'innovazione consisterà nel formulare un layer di "normalizzazione post-pruning" in grado di riscalare le attivazioni basandosi sulla topologia originaria dei manifold, invertendo analiticamente il decadimento delle feature.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Deep learning theory|
|**Riassunto**|Analisi formale e mitigazione del fenomeno "Vanishing Feature" che affligge le rappresentazioni interne durante il merging di reti convoluzionali per l'audio.|
|**Input / Output**|**Input:** Pesi di modelli indipendenti e tensori di feature intermedie. **Output:** Dimostrazioni teoriche sulle varianze e un'architettura fusa priva di collasso dimensionale.|
|**Costo Computazionale**|Basso. L'analisi richiede principalmente tracciamenti statistici e manipolazioni algebriche sui pesi, eludendo la necessità di addestramenti massivi.|
|**Dataset**|Modelli giocattolo pre-addestrati su Speech Commands.|

### PRJ-14: Bound Statistici per Modelli di Diffusione Discreta

La stima dell'errore statistico indipendente dalle dimensioni dello stato per i "Discrete Diffusion Models" costituisce una problematica teorica aperta e critica. Nei modelli di diffusione discreta, le perturbazioni avvengono non su variabili gaussiane continue ma su token categorici, il che rende i bound di generalizzazione classici spesso non applicabili. Questo studio teorico applica le divergenze informative (come la divergenza KL trattata nel corso ) per definire nuovi limiti superiori sull'errore di approssimazione quando si modellano spazi distribuzionali categorici, quali i file MIDI o partiture musicali. Generando una catena di Markov di corruzione discreta , lo studente deriverà formulazioni matematiche per correlare il numero ottimale di step di denoising rispetto all'entropia intrinseca del brano musicale, verificando le previsioni teoriche tramite addestramento su piccole matrici di transizione.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Deep learning theory|
|**Riassunto**|Derivazione di bound di errore statistico per i modelli di diffusione discreta applicati alla generazione di sequenze categoriche (musica simbolica).|
|**Input / Output**|**Input:** Matrici di transizione di Markov e token MIDI. **Output:** Formule analitiche e modelli giocattolo che validano sperimentalmente i teoremi proposti.|
|**Costo Computazionale**|Molto Basso. Si basa prevalentemente su derivazioni matematiche supportate da simulazioni stocastiche di basso livello.|
|**Dataset**|Dati puramente sintetici generati da processi stocastici noti.|

## Distributed Learning

Il framework del distributed learning affronta l'addestramento collaborativo di reti neurali attraverso nodi e infrastrutture remote, dovendo gestire rigorosi vincoli di privacy, overhead di comunicazione e disomogeneità statistica (non-IID) dei dati.

### PRJ-15: Aggregazione Latente Entangled (FedRE) per Bioacustica

La condivisione dei parametri in scenari di Federated Learning si rivela impraticabile quando i nodi periferici possiedono architetture neurali eterogenee (Model-Heterogeneous). Il paradigma recente "FedRE" (Federated Representation Entanglement) introduce la possibilità di condividere esclusivamente rappresentazioni latenti offuscate ed "entangled". Il progetto in questione modella la traduzione di questo framework per le interfacce BCI (Brain-Computer Interfaces) o per sensori bioacustici. I nodi client elaboreranno i segnali sonori localmente con reti asimmetriche, estraendo codici latenti che saranno poi aggrovigliati (entangled) tramite permutazioni controllate. Il server globale non apprenderà una rete, ma uno spazio metrico globale ottimizzando i prototipi latenti. Si analizzerà come il meccanismo di entanglement preservi la differenziabilità garantendo simultaneamente difese crittografiche intrinseche contro gli attacchi avversari di ricostruzione.

| **Parametro**            | **Descrizione Dettagliata**                                                                                                                                           |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Area di Afferenza**    | Distributed learning                                                                                                                                                  |
| **Riassunto**            | Trasferimento del framework FedRE su architetture eterogenee per l'elaborazione bioacustica, permettendo aggregazione latente senza scambio di pesi.                  |
| **Input / Output**       | **Input:** Codici latenti provenienti da estrattori locali asimmetrici. **Output:** Prototipi di classificazione distribuiti e resistenti alle inversioni avversarie. |
| **Costo Computazionale** | Basso per il server (operante solo su vettori latenti corti), variabile e scalabile per i singoli nodi client.                                                        |
| **Dataset**              | Dataset pubblici di bioacustica partizionati artificialmente con skew elevati.                                                                                        |

### PRJ-16: Stabilizzazione Proximal (PEARL-Prox) per Apprendimento Distribuito Quantizzato

Le oscillazioni estreme e la deriva dei giocatori (player drift) sono criticità assolute nel training decentralizzato. Una tecnica recente, il "PEARL-Prox" (Proximal Algorithm for Resolving Player Drift), interviene nell'ottimizzazione tramite l'imposizione di vincoli di prossimalità. La presente proposta rielabora questo algoritmo di regolarizzazione prossimale adattandolo alla compressione quantizzata dei gradienti. Immaginando un setup di addestramento su dispositivi mobili per modelli linguistici conversazionali (sotto forma di Low-Rank Adaptation - LoRA ), i gradienti locali subiscono forti compressioni che aggravano il drift. Integrando la metrica PEARL-Prox nel processo di calcolo dell'errore, si punta a forzare le traiettorie dell'apprendimento verso le proiezioni ortogonali ideali , dimostrando un miglioramento misurabile nella convergenza asintotica in condizioni di rete frammentata.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Distributed learning|
|**Riassunto**|Utilizzo dell'algoritmo PEARL-Prox per stabilizzare l'addestramento distribuito di parametri LoRA soggetti a quantizzazione distruttiva.|
|**Input / Output**|**Input:** Aggiornamenti LoRA quantizzati dai client. **Output:** Modello centrale aggregato con mitigazione algoritmica del drift.|
|**Costo Computazionale**|Medio. Il calcolo prossimale introduce un lieve overhead matematico, ma compensa ampiamente le instabilità risparmiando iterazioni complessive.|
|**Dataset**|Sottoinsiemi di dataset di istruzioni NLP (es. Alpaca ridotto).|

## Representation Learning

Il dominio del representation learning si prefigge l'obiettivo di distillare l'essenza dell'informazione cruda in vettori compatti (embedding), rimuovendo il rumore, separando i fattori di variazione e costruendo manifold latenti dotati di significato algebrico e geometrico.

### PRJ-17: Allineamento Audio-Testo Dinamico tramite Condizionamento Relazionale (Rcml)

Mentre l'apprendimento contrastivo classico (es. CLIP) produce per ogni campione un embedding monolitico, ignorando la ricchezza relazionale, il framework "Relation-Conditioned Multimodal Learning" (Rcml) suggerisce che i vettori latenti dovrebbero variare in base alle direttive semantiche (query o relazioni descrittive). Questa proposta porta il paradigma Rcml nel delicato spazio dell'Audio-Text alignment. Tradizionalmente, uno spettrogramma di una giungla viene mappato in un punto fisso. Integrando un modulo di condizionamento relazionale, il modello produrrà rappresentazioni dinamiche: se la relazione testuale è "suoni acuti", il vettore latente audio si spostersposterà verso le coordinate del verso degli uccelli, ignorando il fruscio del vento. La complessità consiste nell'implementare un obiettivo contrastivo unificato che modelli contemporaneamente l'allineamento cross-modale e l'induzione dinamica imposta dalle condizioni, fornendo metriche di retrieval di gran lunga superiori sui database sonori generici.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Representation learning|
|**Riassunto**|Condizionamento dinamico delle rappresentazioni acustiche contrastive (tramite Rcml) per enfatizzare feature sonore specifiche in base alla relazione richiesta.|
|**Input / Output**|**Input:** Clip audio e una descrizione della relazione semantica target. **Output:** Embedding latente riposizionato dinamicamente in funzione del contesto.|
|**Costo Computazionale**|Alto (richiede l'aggiornamento congiunto di encoder multimodali complessi e pesanti calcoli contrastivi).|
|**Dataset**|AudioCaps o Clotho.|

### PRJ-18: Prevenzione del Collasso Dimensionale in Audio SSL tramite SSOLE

Le strategie Self-Supervised Learning (SSL) si imbattono sovente nel collasso dimensionale, problema in cui l'energia della rappresentazione si concentra su un ristretto numero di dimensioni del sottospazio. Il meccanismo "Orthogonal Low-rank Embedding" (SSOLE) affronta questo ostacolo per la visione artificiale minimizzando la ridondanza informativa forzando l'ortogonalità tra le matrici di embedding. Il progetto mira a trasporre la loss SSOLE sui modelli di auto-supervisione per il parlato (come i derivati di wav2vec o HuBERT). Piuttosto che dipendere da complessi meccanismi di mascheratura e quantizzazione, l'inserimento di una penalità di ortogonalità estrarrà feature vocali intrinsecamente indipendenti e disaccoppiate (ad esempio, separando l'identità del parlatore dall'informazione fonetica), promuovendo un robusto apprendimento dei manifold senza supervisione linguistica.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Representation learning|
|**Riassunto**|Applicazione della penalità di embedding ortogonale a basso rango (SSOLE) per prevenire il collasso dimensionale nei modelli self-supervised vocali.|
|**Input / Output**|**Input:** Tracce vocali non etichettate. **Output:** Rappresentazioni latenti distribuite in componenti ortogonali, pronte per il fine-tuning a valle.|
|**Costo Computazionale**|Medio. Il framework SSL richiede risorse per il training esplorativo, abbattibili ricorrendo a modelli "student" alleggeriti.|
|**Dataset**|LibriSpeech (frazioni da 100 ore).|

## Program Synthesis

La generazione, l'interpretazione e la deduzione strutturata di codice informatico tramite reti neurali che combinano pattern recognition profondo con l'affidabilità rigorosa dei paradigmi simbolici e sintattici (Neurosymbolic AI).

### PRJ-19: Sintesi di Partiture Musicali tramite Latent Program Networks (LPN)

L'affidamento su meccanismi generativi testuali puri porta i modelli a soffrire il collasso logico di fronte a vincoli stringenti. Le "Latent Program Networks" (LPN) risolvono questo problema eseguendo algoritmi di ricerca (come gradient-based optimization) direttamente nello spazio dei "programmi impliciti" latenti, prima ancora di emettere l'output discreto testuale. Questo progetto applica l'innovativa architettura LPN al dominio del sequenziamento musicale (Music Information Retrieval). Il modello apprenderà uno spazio latente corrispondente alla notazione musicale (come ABC Notation o LilyPond). L'input sarà una registrazione audio polifonica ; anziché decodificare immediatamente le note, il modello navigherà lo spazio dei programmi latenti per ottimizzare la coerenza globale (rispettando vincoli di ritmo e polifonia imposti come penalizzazioni sul gradiente differenziabile), decodificando infine lo script sorgente impeccabile. Tale connubio garantisce che il "codice" generato sia non solo sintatticamente corretto, ma musicalmente sensato.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Program synthesis|
|**Riassunto**|Trascrizione da audio a partitura musicale (tramite linguaggi simbolici come ABC notation) sfruttando le ricerche ottimizzate nei Latent Program Networks.|
|**Input / Output**|**Input:** Tracce audio grezze. **Output:** Sintesi del codice testuale compilabile per la ricostruzione accurata della partitura e dell'arrangiamento.|
|**Costo Computazionale**|Alto in fase di inferenza a causa dell'adattamento test-time, compensato da encoder acustici leggeri in fase di training.|
|**Dataset**|MAPS (MIDI Aligned Piano Sounds) accompagnato dalla sua trasposizione sintattica testuale.|

### PRJ-20: Sintesi Transduttiva di Shader Grafici con World Models

Recenti avanzamenti nei LLM per la codifica si focalizzano sull'incorporazione di "World Models" che permettono alla rete di simulare passaggi di esecuzione Python all'interno dello spazio latente (Code Generation with World Models - CWM). Espandendo questo paradigma alle tecniche di "Transductive Program Synthesis", in cui gli input del test-set orientano attivamente l'eliminazione delle ipotesi incoerenti , il progetto propone la sintesi di shader grafici (GLSL o frammenti WebGL) partendo da specifiche in linguaggio naturale. L'LLM genererà molteplici sintassi candidate, simulando i passaggi vettoriali matematici nel suo "world model" e calcolando una loss di aderenza rispetto a input transduttivi generati per valutare la validità visiva. Il risultato unisce l'ispezione algoritmica profonda alla creatività visiva, offrendo un compilatore di shader differenziabile end-to-end.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Program synthesis|
|**Riassunto**|Sintesi di shader grafici complessi tramite World Models simulativi e valutazione transduttiva test-time per scartare codici con errori logico-visivi.|
|**Input / Output**|**Input:** Descrizione dell'effetto visivo desiderato. **Output:** Codice GLSL testato e validato internamente dal world model simulato.|
|**Costo Computazionale**|Molto Alto, richiede contesti estesi (fino a 100k+ token) per supportare la simulazione latente prolungata delle istruzioni.|
|**Dataset**|Dataset derivato dal web-scraping strutturato di repository specializzati in shader grafici.|

## Multimodal Learning

Lo sviluppo di framework computazionali capaci di elaborare simultaneamente input di differente natura fisica (es. spettrogrammi continui, pixel, token discreti), ricavando un tessuto semantico unificato e interrelato.

### PRJ-21: Risoluzione del Modality-Gap (InfoNCE Disaccoppiata) per Segnali EEG

L'addestramento multimodale tramite obiettivi puramente contrastivi (InfoNCE) introduce gap di distribuzione che inibiscono l'uso congiunto in scenari generativi e discriminativi. Una soluzione teorica all'avanguardia disaccoppia esplicitamente l'allineamento delle coppie positive dalla repulsione uniforme delle coppie negative, instaurando una divergenza di Hölder per garantire la convergenza distributiva. Questa tesi verrà convertita e applicata all'allineamento incrociato tra tracciati elettroencefalografici (EEG) trasformati in formato immagine e descrizioni testuali neuro-cognitive. Pur trattandosi di segnali, visualizzarli come spettrogrammi temporali li espone alle regole del multimodal learning visivo. Il disaccoppiamento InfoNCE forzerà il modello a rispettare sottili similarità topologiche nei flussi energetici senza farsi sovrastare dalla repulsione globale, migliorando esponenzialmente il retrieval testuale-EEG rispetto ad approcci naïf.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Multimodal learning|
|**Riassunto**|Risoluzione del modality-gap tramite perdita InfoNCE disaccoppiata (divergenza di Hölder) per l'allineamento fine tra spettrogrammi derivati e testo.|
|**Input / Output**|**Input:** Spettrogrammi (o mappe di features) e relative annotazioni semantiche. **Output:** Spazio latente incrociato armonizzato e privo di distribuzioni conflittuali.|
|**Costo Computazionale**|Medio. La complessità computazionale si sposta sulla struttura delle loss e della divergenza matematica, senza necessità di scalare l'architettura.|
|**Dataset**|Dataset di spettrogrammi pubblici open-domain etichettati.|

### PRJ-22: Ottimizzazione DPO Gerarchica per Localizzazione Audiovisiva

I paradigmi di fusione multimodale ("early, intermediate, late") falliscono tipicamente in compiti altamente granulari per la difficoltà di estrarre mappe semantiche esplicite da dati spettrali. Architetture come CHiP (Cross-modal Hierarchical Direct Preference Optimization) hanno introdotto l'ottimizzazione delle preferenze dirette (DPO) all'interno degli LLM multimodali per allinearli ai valori desiderati. Questo progetto adotterà l'allineamento delle preferenze DPO strutturandolo gerarchicamente per la classificazione e segmentazione di eventi audio nei flussi video (Audio-Visual Event Localization). Sfruttando un'architettura ibrida e l'insegnamento per preferenze latenti (fornendo coppie di allineamenti video-audio "buoni" vs "scarsi"), il sistema affinerà i confini decisionali in modo non supervisionato. Si dedurrà se le ottimizzazioni basate su feedback umani diretti, consuete nei text-generator, traslino con successo nel grounding temporale audiovisivo.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Multimodal learning|
|**Riassunto**|Implementazione del framework CHiP basato su Direct Preference Optimization gerarchica per la sintonizzazione della localizzazione fine di eventi nei video acustici.|
|**Input / Output**|**Input:** Flussi continui multimodali (Audio/Video). **Output:** Segmentazione temporale annotata ed estratta secondo le preferenze allineate dal DPO.|
|**Costo Computazionale**|Alto (le tecniche DPO richiedono passaggi inferenziali continui attraverso modelli teacher e logit di riferimento durante l'aggiornamento).|
|**Dataset**|VGGSound o subset specializzati estratti da YouTube.|

## Quantization

Lo studio sistematico dei compromessi tra fedeltà rappresentazionale ed efficienza esecutiva, esplorando algoritmi per confinare i parametri in spazi numerici fortemente discretizzati o ternari.

### PRJ-23: Quantizzazione Ternaria (BitLinear 1.58) per Architetture Conformer ASR

Le architetture "BitNet b1.58", pioniere nell'imposizione di pesi ristretti all'alfabeto ternario $\{-1, 0, 1\}$, hanno provato che la Quantization-Aware Training (QAT) elude le inefficienze moltiplicative della virgola mobile, riducendo esponenzialmente la memoria pur mantenendo le metriche di entropia e perplexity quasi intatte rispetto all'FP16. Questo progetto adatta il framework BitLinear al dominio del processamento acustico, tipicamente dipendente da segnali continui ad alta fedeltà. Si sostituiranno i layer standard all'interno di un modello Conformer per il riconoscimento vocale (ASR) con le varianti ternarie, implementando tecniche di approssimazione del gradiente come la Straight-Through Estimator per aggirare la non-differenziabilità dei pesi discreti. Questo esperimento determinerà l'impatto qualitativo della sottocampionatura spinta sui coefficienti Cepstrali di Mel (MFCCs) e il relativo balzo in avanti nelle velocità di inferenza su CPU e NPU.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Quantization|
|**Riassunto**|Traduzione e addestramento del costrutto ternario BitLinear (1.58-bit) per architetture Conformer destinate all'analisi dei segnali vocali continui.|
|**Input / Output**|**Input:** Forme d'onda acustiche crudi (o spettrogrammi Mel). **Output:** Embedding quantizzati o decodifica testuale con matrice di pesi ternarizzata.|
|**Costo Computazionale**|Medio-Alto nel QAT training, ma l'inferenza annulla quasi totalmente l'assorbimento VRAM e le operazioni moltiplicative costose (solo addition).|
|**Dataset**|Sottoinsiemi di LibriSpeech o TIMIT.|

### PRJ-24: Quantizzazione Sigma-Delta per Diffusion Transformers (DiT)

I modelli generativi probabilistici affrontano ostacoli immensi quando soggetti alla Post-Training Quantization (PTQ), a causa della sensibilità intrinseca al rumore nel processo iterativo di denoising. Metodologie emergenti come SDQ-LLM impiegano il "Sigma-Delta Quantization", mutuato dalle logiche dei convertitori analogico-digitali, avvalendosi del noise shaping per iniettare l'errore di discretizzazione in bande di frequenza irrilevanti. La presente ricerca mira ad infondere la tecnica Sigma-Delta nella quantizzazione PTQ di Diffusion Transformers (DiT) operanti su segnali ambientali. Questo stratagemma spingerà i difetti della rappresentazione numerica verso le frequenze impercettibili del rumore latente , producendo per la prima volta un modello di diffusione generativo a singolo bit o a bassa precisione non soggetto alla tipica disintegrazione del sampling qualitativo.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Quantization|
|**Riassunto**|Sfruttamento della quantizzazione Sigma-Delta per trasferire l'errore di PTQ nelle bande non influenti, preservando la coerenza nei modelli di diffusione audio/visivi.|
|**Input / Output**|**Input:** Tensori dei pesi FP32 di un DiT e sample di attivazione. **Output:** Reticolo parametrico altamente compresso resistente all'errore iterativo di denoising.|
|**Costo Computazionale**|Basso (tecnica applicata post-training che richiede principalmente cicli inferenziali di calibrazione e ottimizzazione del noise shaping).|
|**Dataset**|Nessun addestramento richiesto; dataset ristretto per la calibrazione delle attivazioni (es. sottoinsieme FSD50K).|

## Language Models

Esplorazione, ottimizzazione logica e decostruzione meccanicistica dei flussi cognitivi all'interno di Modelli Linguistici (LLM), concentrandosi su paradigmi che affrancano le reti dai vincoli del contesto e dai costi inferenziali diretti.

### PRJ-25: Pianificazione Latente (PLaT) per Ragionamento Armonico Simbolico

L'efficienza inferenziale del ragionamento "Chain-of-Thought" (CoT) è fatalmente frenata dall'esplicita elaborazione step-by-step nello spazio discreto dei token. L'innovativo framework PLaT (Planning with Latent Thoughts) affronta questo ostacolo formulando il ragionamento testuale come una pura traiettoria deterministica in uno spazio di pianificazione latente. Questa ricerca integrerà i principi del PLaT nel contesto della sintesi di espressioni aritmetico-musicali o del sequencing MIDI. Sganciando l'intricato calcolo armonico e temporale dall'obbligo di generare costantemente output discreti (spesso propensi alle allucinazioni ), un Transformer dedicato navigherà i pesi dei vettori latenti e determinerà in tempo reale quando il "pensiero acustico" risulta convergente e pronto per essere decodificato nel blocco terminale, scalando dinamicamente le diversità rappresentazionali senza appesantire la complessità computazionale a test-time.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Language models|
|**Riassunto**|Sostituzione del ragionamento CoT esplicito con traiettorie latenti dinamiche (PLaT) per risolvere compiti di astrazione strutturale (armonie MIDI) in modo ultra-efficiente.|
|**Input / Output**|**Input:** Prompt contenenti vincoli teorici stringenti. **Output:** Traiettoria logico-musicale valutata latente e successivamente tradotta nel dominio formale finale.|
|**Costo Computazionale**|Medio. Abbassa vertiginosamente la latenza di decoding inferenziale precludendo generazioni token-wise non essenziali.|
|**Dataset**|Dataset di progressioni formali simboliche (es. JSB Chorales).|

### PRJ-26: Analisi di Repository tramite Collaborazione Multi-Agente (Chain-of-Agents)

Le reti LLM contemporanee palesano enormi limitazioni di contesto a fronte di task a vasto respiro; l'architettura Chain-of-Agents (CoA) bypassa l'ingombrante calcolo dell'attenzione quadratica orchestrando agenti "worker" indipendenti che frammentano il contesto per ricomporlo verso un agente manageriale. La proposta implementa il framework CoA per analizzare giganteschi repository di configurazione grafica e fisica in ambienti simulativi (es. script procedurali Unreal/Unity). Ogni agente processerà segmenti testuali di codice e scambierà "embeddings" di sintesi vettoriali derivate con il nodo centrale, testando l'ipotesi che la collaborazione in linguaggio naturale aumentata dalla diffusione vettoriale superi strutturalmente gli approcci Retrieval-Augmented Generation (RAG) nel mantenere il senso di coesione globale in task prolungati e fortemente ingegnerizzati.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Language models|
|**Riassunto**|Integrazione della frammentazione asincrona (Chain-of-Agents) arricchita dalla propagazione di embedding testuali per superare la contest length nei modelli linguistici.|
|**Input / Output**|**Input:** Blocchi massivi di codice o documentazione non sintetizzabile. **Output:** Insight coerente, sintesi di intere directory tramite inferenza collaborativa.|
|**Costo Computazionale**|Alto (parallelizzazione di passaggi computazionali per vari sub-agenti, gestibile allocando modelli più piccoli come i 2B-3B parameters).|
|**Dataset**|GitHub repositories testuali o dataset QA documentali sintetizzati.|

## Image/Video Generation

Architetture ed innovazioni trasformative volte al superamento dei colli di bottiglia computazionali e concettuali nella sintesi coerente ad alta fedeltà dello spazio visivo e temporale.

### PRJ-27: Modulazione dell'Intensità Cinetica (MotionStone) Audio-Reattiva

La predominanza dei Diffusion Transformers (DiT) fatica a disaccoppiare l'estetica formale dall'intensità cinetica nei flussi generati. La ricerca su "MotionStone" (Decoupled Motion Intensity Modulation) ha esposto la possibilità di modulare dinamicamente tali intensità isolando i fattori di attenzione. La presente indagine adatta questa segregazione modale alla sintesi generativa in risposta a input di controllo astratti. Anziché intervenire sulla semplice velocità , il modello implementerà blocchi di condizionamento basati sui principi della classificazione di regressione non-lineare per pilotare la gravità vettoriale, la propagazione particellare o le transizioni di scena in base a input acustici ambientali integrati come vettori direzionali supplementari. Il risultato disporrà di leve di intensità indipendenti dal dominio estetico per l'animazione algoritmica.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Image/video generation|
|**Riassunto**|Separazione dei vettori legati all'intensità del movimento nello spazio latente dei Diffusion Transformer per un controllo cinematico fine basato su direttive astratte.|
|**Input / Output**|**Input:** Scalari di intensità dinamica associati a vettori di base. **Output:** Clip video o array di sprite animati in cui l'intensità reattiva varia indipendentemente dalla texture.|
|**Costo Computazionale**|Medio-Alto. Il training su dataset video esige efficienza; l'impiego di micro-batch temporali alleggerisce il gradiente.|
|**Dataset**|UCF-101 o database curati per azioni specifiche di piccola entità temporale.|

### PRJ-28: Adattamento Temporale Lineare (MoMa) per Generazione Video Condizionata

Un approccio concorrente, denominato "MoMa" (Modulating Mamba), elude interamente le pesanti strutture di Self-Attention spazio-temporali iniettando nei Foundation Model di immagini la selettività lineare degli State Space Models. Tale integrazione permette adattamenti Parameter-Efficient (PEFT). Il progetto declina la metodologia "SeqMod" (Divide-and-Modulate) verso la generazione visiva controllata da spettrogrammi audio in sequenza lineare. Lo "State Space Duality" dei blocchi Mamba integrati nell'encoder visivo consentirà al generatore di apprendere e mantenere le dipendenze dinamiche del suono (es. le vibrazioni dei bassi costanti) nel tempo $O(N)$ senza distruggere i feature maps della pre-titolazione d'immagine originale, costituendo un breakthrough per l'efficienza della generazione Image-to-Video condizionata acusticamente.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Image/video generation|
|**Riassunto**|Adattamento PEFT (Parameter-Efficient) di modelli d'immagine tramite iniezioni di moduli Mamba per l'evoluzione temporale fluida e a basso costo condizionata da audio.|
|**Input / Output**|**Input:** Pesi di un Foundation Model visivo e sequenze temporali estese. **Output:** Modello incrementato dalla comprensione video senza calcoli d'attenzione quadratici.|
|**Costo Computazionale**|Basso. Sfrutta il congelamento (freezing) del backbone originale, imparando esclusivamente le connessioni dei parametri Mamba iniettati.|
|**Dataset**|Sottoinsieme coerente tratto da WebVid-10M.|

## Deep Learning for Audio

Approcci che oltrepassano il semplice processamento del segnale per decostruire, generare, fondere o denoizzare il dominio sonoro avvalendosi di dinamiche differenziabili allo stato dell'arte.

### PRJ-29: Restaurazione Orchestrale Estrema tramite Universal Speech Enhancement Mamba

Mentre l'architettura Mamba e l'audio transformer dominano gli scenari esplorativi , un problema cruciale risiede nella manipolazione diretta del rumore stocastico senza corrodere le strutture armoniche in scenari estremi. L'approccio "USEMamba" (Universal Speech Enhancement) unifica i meccanismi state-space con la regressione time-frequency. Invece di limitarsi allo speech, questo progetto esplora la regressione differenziabile guidata dai blocchi Mamba per ricostruire la larghezza di banda e i transitori spettrali persi o corrotti (come nella perdita di pacchetti) della registrazione degli strumenti musicali orchestrali (Audio Restoration). L'addestramento verificherà come le connessioni bidirezionali assimilino la coerenza compositiva, superando intrinsecamente il blurring tipico del denoising autoencoder-based operante nel dominio continuo.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Deep learning for audio|
|**Riassunto**|Estensione del sistema USEMamba per la regressione e ricostruzione generativa dell'integrità ad ampio spettro in strumenti musicali soggetti a degradazioni estreme.|
|**Input / Output**|**Input:** Registrazioni di segnali audio distorte, rumorose o con perdite di frequenza. **Output:** Tracce acustiche interamente rigenerate con coerenza di fase e spettro.|
|**Costo Computazionale**|Basso (inferenza); Medio (training). La natura lineare dell'architettura riduce drasticamente l'assorbimento computazionale classico dei Transformer.|
|**Dataset**|FMA (Free Music Archive) opportunamente deteriorato sinteticamente o dataset orchestrali.|

### PRJ-30: Controllo Differenziabile di Sintetizzatori DSP tramite EzAudio-DiT

La sintesi parametrica del suono affronta il paradosso per cui le architetture moderne come DiT offrono coerenza formidabile ma lentezze insostenibili, come ovviato parzialmente da sistemi quali "EzAudio-DiT" che ricalibrano la Classifier-Free Guidance (CFG) e usano layer AdaLN altamente snelliti. Il presente lavoro preleva l'architettura matematica ottimizzata per il testo-a-suono e la dirotta all'analisi invertita dell'elaborazione di feature ambientali ("AWOL per Audio" via modelli di flusso e embeddings linguistici). Sostituendo la pura decodifica latente con parametri di sintesi procedurale dedotti empiricamente [1, 1], l'EzAudio-DiT modificato mapperà le stringhe latenti testuali non al rumore finale, bensì agli oscillatori e inviluppi dei sintetizzatori fisici modulabili, connettendo il machine learning puro al sound design DSP (Digital Signal Processing) manipolabile umanamente.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Deep learning for audio|
|**Riassunto**|Deviazione della sintesi Text-to-Audio (EzAudio-DiT) verso la generazione differenziabile di parametri di controllo manipolabili per sintetizzatori DSP tradizionali.|
|**Input / Output**|**Input:** Prompt testuali ed embeddings CLAP/CLIP. **Output:** Mappature esatte dei parametri fisici (knobs, LFO, macro) del synth al fine di generare il suono target.|
|**Costo Computazionale**|Alto (richiede l'integrazione di renderer differenziabili o cicli di rinforzo surrogati per stimare il gradiente delle librerie audio).|
|**Dataset**|Pairwise testuali estratti da AudioCaps connessi a banchi di preset per sintetizzatori (es. Dexed).|

## Reasoning

Investigazione della complessa alchimia che porta l'intelligenza artificiale dalla semplice deduzione stocastica dei token alla costruzione deliberata, gerarchica e causale di conclusioni complesse, sbrogliando l'incapacità o la generalizzazione degli algoritmi fondazionali.

### PRJ-31: Activation Patching per l'Isolamento del Grokking Fisico-Matematico

La comprensione dell'effettiva meccanica dietro il ragionamento logico nei modelli è oscurata dall'architettura a scatola nera ("black-box"). Le analisi di "Activation Patching" o di "Attribution Graphs" hanno scoperchiato circuiti localizzabili in cui le connessioni formano gerarchie vere e proprie di elaborazione. Lo studio teorico sul "Grokking of Implicit Reasoning" ha inoltre delineato come la risoluzione logica multi-hop (a passaggi continui e ricorsivi) necessiti di periodi estesi affinché le euristiche cedano il passo ai modelli algebrici formali. Il progetto propone un esperimento causale: isolare il grokking delle regole fisiche di collisione o di deduzione matematica intervenendo nei transformer pesati manipolando e sostituendo vettori specifici per forzare generalizzazioni "a comando". Attraverso perturbazioni mirate dell'Hessiana sui tensori interni, l'esperimento accerterà quanto la memorizzazione sia scollegata o unita alla capacità induttiva intrinseca della rete.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Reasoning|
|**Riassunto**|Analisi causale tramite Activation Patching e Attribution Graphs per tracciare e alterare la formazione dell'intelligenza multi-hop all'interno dei layer latenti.|
|**Input / Output**|**Input:** Sillogismi logici o regole di induzione fisica per un modello pre-addestrato. **Output:** Rappresentazioni strutturali (Attribution Graphs) che indicano la genesi delle deduzioni.|
|**Costo Computazionale**|Basso. Si appoggia all'analisi retrospettiva e differenziale delle matrici di modelli di medie dimensioni (es. <3B parametri) senza necessità di training esaustivi.|
|**Dataset**|Set dedicati al ragionamento logico multi-hop o versioni semplificate del benchmark ARC-AGI.|

### PRJ-32: Sintesi di Programmi Transduttivi per l'Ispezione di Serie Temporali

Parallelamente, l'approccio di "Transductive Program Synthesis" rovescia il problema algoritmico del ragionamento. Mentre le soluzioni normali imparano induttivamente dall'addestramento per poi testare passivamente, il framework transduttivo tratta i problemi di verifica (come lo smistamento in matrici geometriche del benchmark ARC-AGI ) sfruttando attivamente l'input del test-set per generare euristiche transitorie di sbarramento o falsificazione delle ipotesi sbagliate tramite algoritmi avidi maximin. Questo progetto adatta il ragionamento transduttivo a un dominio esotico: il recupero e la manipolazione controllata di segnali temporali rumorosi. Guidato da un LLM compatto (es. Llama o derivati quantizzati), il modello produrrà codici inferenziali in grado di interpolare e ripulire anomalie caotiche analizzando l'incoerenza tra il programma ipotetico predetto e la coerenza analitica osservata nello spettro temporale di test.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Area di Afferenza**|Reasoning|
|**Riassunto**|Integrazione dell'algoritmica transduttiva test-time per forzare la deduzione algoritmica di programmi sintattici focalizzati sulla pulizia e l'ispezione di dati temporali anomali.|
|**Input / Output**|**Input:** Dati di test parzialmente oscurati o logicamente sfidanti. **Output:** Sintesi dinamica di codice e generazione attiva di regole logiche esclusive valutate in real-time.|
|**Costo Computazionale**|Alto nell'inferenza interattiva (test-time search e chiamate cicliche ai pesi dell'LLM).|
|**Dataset**|Dataset di compiti logico-matematici 1D o serie temporali convertite a task di astrazione geometrica (ARC-AGI 1D).|

---

## Conclusioni e Prospettive Metodologiche

Il compendio dei progetti sopra delineato assolve rigorosamente la richiesta di innovazione accademica per la ricerca magistrale, bilanciando in ogni occasione il rigore matematico con le necessità di esecuzione dettate da budget computazionali ristretti. L'approccio sistematico della traslazione di dominio – ad esempio la commutazione delle reti T-Switch , Mamba o delle divergenze multi-modali al contesto dell'elaborazione acustica – assicura l'indipendenza delle idee rispetto all'ordinario. Gli studenti, integrando procedure valutative solide basate sulle loss curve e la convalida stocastica , forniranno elaborati rispondenti agli esigenti standard pubblicazionali di conferenze leader (come NeurIPS e ICML), concretizzando la visione d'indagine esplorativa che anima le direttive del corso.