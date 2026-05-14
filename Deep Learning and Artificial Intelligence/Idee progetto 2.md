# Rapporto di Ricerca: Progettazione Architetturale e Applicazioni Avanzate per Deep Learning

La presente disamina tecnica formula un ecosistema di proposte progettuali destinate a soddisfare i requisiti avanzati della ricerca accademica nel campo dell'apprendimento profondo, strutturate specificamente per il paradigma BYOP (Bring Your Own Project). L'indagine è stata condotta escludendo rigorosamente i domini applicativi non ammessi e concentrandosi sulle sedici aree di ricerca esplicitamente autorizzate.

A differenza degli approcci iper-specializzati, questo documento propone progetti **trasversali (cross-area)**. L'impianto si fonda sui principi matematici trattati nel corso, quali l'ottimizzazione stocastica, le divergenze informazionali (es. divergenza di Kullback-Leibler) e l'analisi sul manifold , con un'enfasi sulla rielaborazione di architetture consolidate (es. VAE, Transformer, CNN) applicate in combinazioni innovative.

## Proposte Progettuali Interdisciplinari

### PRJ-01: VQ-VAE a posteriori con Quantizzazione Geodetica

Questo progetto rivisita la classica pipeline del Vector Quantized Variational Autoencoder (VQ-VAE). Invece di apprendere congiuntamente uno spazio latente discreto, si propone di addestrare inizialmente un VAE continuo standard (su immagini o spettrogrammi audio). Successivamente, si applica una quantizzazione vettoriale a posteriori utilizzando le distanze geodetiche nel manifold latente (calcolate tramite cammini minimi su un k-NN graph o approssimazioni riemanniane) invece delle consuete distanze euclidee. Infine, un modello autoregressivo viene addestrato sui codici discreti risultanti. Il progetto sfida a riflettere criticamente sulla geometria intrinseca degli spazi latenti appresi.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Quantization, Representation learning, Deep learning theory|
|**Riassunto**|Sostituzione della quantizzazione euclidea end-to-end con una quantizzazione a posteriori basata sulla distanza geodetica nel manifold latente di un VAE.|
|**Input / Output**|**Input:** Dati continui (es. immagini o audio). **Output:** Ricostruzione generativa basata su codebook discreto e topologicamente coerente.|
|**Costo Computazionale**|Medio. L'addestramento del VAE continuo è standard; il calcolo dei cammini geodetici post-hoc richiede risorse computazionali CPU/RAM aggiuntive.|
|**Dataset**|MNIST (per prototipazione) esteso poi a spettrogrammi audio.|

### PRJ-02: Generazione Audio Autoregressiva nel Dominio della Frequenza

La maggior parte dei modelli generativi audio opera nel dominio del tempo (sintesi di forme d'onda) o su spettrogrammi di magnitudine (recuperando la fase tramite algoritmi come Griffin-Lim). Questo progetto mira a progettare un modello autoregressivo (es. Transformer) che genera audio direttamente nel dominio della frequenza, predicendo i coefficienti spettrali complessi della Trasformata di Fourier a Breve Termine (STFT) sia in ampiezza che in fase.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Deep learning for audio, Image/video generation (tecniche trasposte)|
|**Riassunto**|Creazione di un modello generativo autoregressivo in grado di operare direttamente sui coefficienti complessi (ampiezza e fase) della STFT.|
|**Input / Output**|**Input:** Prompt di condizionamento o rumore. **Output:** Sequenza di coefficienti spettrali complessi convertibili in audio ad alta fedeltà.|
|**Costo Computazionale**|Alto. La previsione congiunta di ampiezza e fase introduce un'elevata dimensionalità e richiede funzioni di perdita specifiche per i numeri complessi.|
|**Dataset**|FMA (Free Music Archive) o Speech Commands.|

### PRJ-03: Fusione Multi-Modello tramite Baricentri Sferici (SLERP per $n>2$)

Le tecniche di "Model Merging" interpolano tipicamente tra due checkpoint neurali usando la Spherical Linear Interpolation (SLERP) per evitare interferenze distruttive. Questo progetto estende il concetto al caso di $n>2$ modelli. L'obiettivo è formulare e implementare un approccio basato sui baricentri sferici per trovare una media ponderata di molteplici reti sull'ipersfera, esplorando modi per garantire la "cycle consistency" (la fusione lungo percorsi diversi nello spazio dei modelli produce risultati coerenti).

| **Parametro**            | **Descrizione Dettagliata**                                                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Aree di Afferenza**    | Model merging, Deep learning theory, Optimization for deep learning                                                                                           |
| **Riassunto**            | Generalizzazione dell'interpolazione sferica (SLERP) per combinare simultaneamente i pesi di oltre due modelli senza perdere le capacità originali.           |
| **Input / Output**       | **Input:** Matrici dei pesi di $n$ reti neurali addestrate indipendentemente. **Output:** Un singolo modello fuso e ottimizzato sull'ipersfera.               |
| **Costo Computazionale** | Basso. Le operazioni matriciali per il calcolo del baricentro sferico si eseguono offline sui pesi senza richiedere cicli di addestramento e backpropagation. |
| **Dataset**              | Checkpoint pre-addestrati scaricati liberamente. Nessun dato grezzo necessario.                                                                               |

### PRJ-04: Re-Basin Ortogonale Oltre le Permutazioni Discrete

Il framework "Git Re-Basin" dimostra che modelli addestrati in modo indipendente possono essere allineati permutando le unità nascoste per consentire una fusione senza barriere. Questo progetto rilassa il vincolo della permutazione (che è un piccolo sottogruppo del gruppo ortogonale) ed esplora trasformazioni ortogonali generali (es. rotazioni tramite decomposizione spettrale) per allineare le rappresentazioni interne. L'esperimento quantificherà l'errore di disallineamento residuo dovuto alla non-commutatività con funzioni non lineari come la ReLU.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Model merging, Deep learning theory|
|**Riassunto**|Allineamento dei pesi di modelli disparati utilizzando trasformazioni ortogonali continue al posto di semplici matrici di permutazione discreta.|
|**Input / Output**|**Input:** Pesi di due modelli omologhi. **Output:** Modello fuso con rappresentazioni allineate e misurazione analitica dell'errore di barriera.|
|**Costo Computazionale**|Medio. Ottimizzare le trasformazioni ortogonali sulla varietà di Stiefel richiede calcoli algebrici moderati.|
|**Dataset**|Reti giocattolo addestrate su CIFAR-10.|

### PRJ-05: Estensione Multimodale VMSST con BERT e DINO

Il paper "Beyond Contrastive Learning" introduce VMSST, un modello generativo variazionale per l'embedding testuale multilingua che separa la semantica condivisa dal rumore senza affidarsi agli hard negatives del contrastive learning. Questo progetto porta VMSST in un ambiente multimodale. Utilizzando BERT per l'encoding semantico e DINO (o un altro ViT) per le feature visive, l'obiettivo è costruire un modello che allinei e districhi la semantica condivisa attraverso le modalità (testo e immagine) tramite priori gaussiani, risultando estremamente utile per il retrieval zero-shot.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Multimodal learning, Representation learning, Language models|
|**Riassunto**|Adattamento di un approccio variazionale generativo per districare (disentangle) la semantica condivisa tra vision e language senza apprendimento contrastivo.|
|**Input / Output**|**Input:** Coppie testo-immagine. **Output:** Spazi latenti multimodali condivisi e componenti latenti specifiche della singola modalità.|
|**Costo Computazionale**|Medio-Alto. L'inferenza variazionale su estrattori multimodali profondi richiede un setup GPU capace.|
|**Dataset**|MS-COCO o Flickr30k.|

### PRJ-06: Allineamento Spettrale ResiDual per Modelli Audio

Il metodo "ResiDual" re-pesa le componenti principali nel flusso residuo dei Transformer, sfruttando la specializzazione delle attention heads per migliorare le prestazioni zero-shot. Il progetto adatterà questa tecnica al dominio dell'audio elaborando modelli come CLAP (Contrastive Language-Audio Pretraining). L'idea è condurre un'Analisi delle Componenti Principali (PCA) sugli output delle head per verificare se un sottoinsieme di queste catturi feature audio rilevanti per il task, per poi implementare un livello di re-weighting spettrale.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Deep learning for audio, Representation learning, Multimodal learning|
|**Riassunto**|Esplorazione della specializzazione del flusso residuo nei Transformer audio-testuali e ri-ponderazione tramite PCA per migliorare l'allineamento cross-modale.|
|**Input / Output**|**Input:** Output residui di un Transformer audio. **Output:** Vettori di attivazione pesati spettralmente per un migliore audio caption retrieval.|
|**Costo Computazionale**|Basso. La tecnica opera sul flusso residuo e non richiede il fine-tuning completo del modello CLAP.|
|**Dataset**|AudioCaps per la valutazione.|

### PRJ-07: APiT - Audio-Predictive Instruction Tuning

Ispirato a MetaMorph, che introduce il Visual-Predictive Instruction Tuning per i LLM, questo progetto propone l'Audio-Predictive Instruction Tuning (APiT). L'obiettivo è sostituire l'encoder visivo con uno acustico e addestrare un LLM pre-addestrato affinché accetti testo e audio in input, predicendo sia token testuali discreti che token audio continui. Questo approccio unifica la comprensione e la generazione autoregressiva multimodale, abilitando l'istruzione mista (es. "Cosa succede dopo in questa scena sonora?").

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Language models, Deep learning for audio, Multimodal learning|
|**Riassunto**|Sintonizzazione delle istruzioni di un LLM per predire token audio continui e testo simultaneamente, permettendo compiti di Text-to-Audio e Audio-QA misti.|
|**Input / Output**|**Input:** Prompt misti testo/audio. **Output:** Risposta testuale e/o decodifica autoregressiva di token audio (forma d'onda via decodificatore).|
|**Costo Computazionale**|Alto. Richiede il fine-tuning di un LLM e l'addestramento dell'allineamento dei token audio.|
|**Dataset**|Clotho o prompt sintetici generati ad hoc.|

### PRJ-08: AWOL per Audio: Generazione Suono tramite Sintesi Parametrica

AWOL (Analysis Without synthesis using Language) mappa lo spazio latente CLIP ai parametri fisici per generare forme 3D. Questo progetto traspone il framework dalla geometria tridimensionale al dominio acustico. Anziché produrre spettrogrammi, l'obiettivo è apprendere una mappatura da un modello audio-linguaggio (es. CLAP) allo spazio dei parametri di un sintetizzatore del suono (es. FM synth, o un modello differenziabile come DDSP). Ciò permette di generare suoni parametrizzati e strutturati interpolando nello spazio semantico del linguaggio.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Program synthesis, Deep learning for audio, Multimodal learning|
|**Riassunto**|Mappatura di embedding linguistici (CLAP) a parametri di controllo di sintetizzatori audio procedurali per la generazione controllata del suono.|
|**Input / Output**|**Input:** Descrizione linguistica di un suono. **Output:** Parametri di sintesi numerica utilizzabili da un DSP per suonare l'audio descritto.|
|**Costo Computazionale**|Medio. Evita i costosi processi di diffusione stocastica concentrandosi su reti di regressione per la previsione dei parametri.|
|**Dataset**|Dataset personalizzato di suoni generati sistematicamente da un sintetizzatore software con relative etichette dei parametri.|

### PRJ-09: Dinamiche di Apprendimento ed "Early Stopping" in Assenza di Replay

La transizione dalle normali dinamiche di apprendimento asintotico alla generalizzazione multi-task solleva il problema del catastrophic forgetting. Basandosi sulle metriche di "Learning with Preserving", si progetterà una rete convoluzionale standard (MLP / CNN) senza buffer storici. La particolarità sta nello studiare la traiettoria della funzione di perdita (Training Dynamics) mentre si applicano perturbazioni al momentum stocastico della discesa del gradiente ogni volta che il dominio della task cambia sequenzialmente. Si confronterà questo andamento con l'approccio dell'Early Stopping analizzando le curve a forma di "U".

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Training dynamics, Multitask learning, Optimization for deep learning|
|**Riassunto**|Analisi microscopica della derivata prima e del momentum durante il cambio sequenziale di domini (multi-task) senza memoria storica, interfacciato con strategie di early stopping.|
|**Input / Output**|**Input:** Dati sequenziali multimodali eterogenei. **Output:** Dinamiche tracciate dei pesi di rete e metrica di penalizzazione dinamica della rappresentazione.|
|**Costo Computazionale**|Basso. Si appoggia su CNN classiche, eliminando il carico della gestione di enormi replay buffer in RAM.|
|**Dataset**|Modelli standard iterati sequenzialmente (es. CIFAR-100 partizionato).|

### PRJ-10: Compressione di Spettrogrammi tramite DiffLogic CA (Automi Cellulari)

Invece di affidarsi a complessi autoencoder per la compressione di serie storiche o immagini , recenti studi applicano i "Differentiable Logic Cellular Automata" (Automi Cellulari a logica booleana). Questo progetto multidisciplinare propone l'utilizzo degli automi cellulari per espandere e ripristinare spettrogrammi audio da un minuscolo "seme" compresso nello spazio latente, simulando le meccaniche della "Vita Artificiale" in contesti di ricostruzione del segnale.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Artificial life, Quantization, Deep learning for audio|
|**Riassunto**|Utilizzo della morfogenesi tipica degli automi cellulari (modellati con operatori logici discreti) per la decompressione ultraleggera di spettrogrammi ambientali.|
|**Input / Output**|**Input:** Seme latente discretizzato e step iterativi. **Output:** Spettrogramma audio spazialmente espanso tramite regole di auto-organizzazione.|
|**Costo Computazionale**|Basso. Le operazioni puramente booleane e le logiche dei cellular automata permettono esecuzioni rapidissime e parallele.|
|**Dataset**|Sottoinsieme isolato di FreeSound o ESC-50.|

### PRJ-11: Apprendimento Transduttivo per l'Ispezione di Anomalie Temporali

Mentre l'approccio induttivo classico generalizza dai dati di training, l'approccio di "Transductive Program Synthesis" utilizza direttamente gli input del test-set per generare euristiche transitorie ed eliminare ipotesi inconsistenti. Guidato da un LLM, il progetto svilupperà programmi in codice per ispezionare e isolare pattern anomali in registrazioni continue di dati (es. anomalie nell'audio biologico o in comandi generici), unendo il ragionamento dinamico a tempo di test all'analisi sequenziale.

|**Parametro**|**Descrizione Dettagliata**|
|---|---|
|**Aree di Afferenza**|Reasoning, Program synthesis, Multitask learning|
|**Riassunto**|Sviluppo di un sistema guidato da LLM che impiega il ragionamento transduttivo a tempo di test per generare codice di filtro su dati sequenziali.|
|**Input / Output**|**Input:** Dati anomali o parzialmente oscurati. **Output:** Codice Python testato internamente per la deduzione e il ripristino dell'integrità del dato.|
|**Costo Computazionale**|Alto nell'inferenza (richiede iterazioni e campionamenti attivi sul LLM).|
|**Dataset**|Test set personalizzati su astrazioni logiche o sequenze temporali sintetiche.|

---

## Conclusioni

Questa selezione di progetti è stata ristrutturata per integrare le pietre miliari teoriche (quali le varietà, il contrastive learning e le meccaniche di ottimizzazione) con i domini di frontiera del _Model Merging_ e del _Multimodal Learning_ espressi nelle guidelines. Essi offrono la flessibilità di modulare le risorse computazionali agendo sulle topologie pre-esistenti, enfatizzando il rigore dell'indagine scientifica rispetto al mero consumo architetturale.