# Azioni differite

> Registro delle azioni che non si possono compiere adesso perché dipendono da una condizione esterna o da una decisione, con la condizione di sblocco di ciascuna e il criterio con cui si stabilisce che sono compiute. Un promemoria che vive solo in una conversazione è perduto; una voce qui sopravvive alla sessione e a un clone del repository.
>
> Ogni azione ha un identificativo nella forma `PA-NNN`. Una voce compiuta non si cancella: si marca come compiuta con la data e l'esito.

## PA-001 - Valutare l'acquisto di una scheda audio per l'home recording

Data di apertura: 2026-09-07. Stato: **aperta, valutazione scritta il 2026-09-30, scelta dell'utente attesa**. Il confronto dei candidati è in `docs/20-catena-di-ingresso.md`; la stessa voce nel progetto `diy-2way-monitors-home` è PA-012, che porta i requisiti dichiarati il 2026-09-21, cioè otto ingressi, almeno un ingresso ad alta impedenza e nessun tetto di spesa.

Che cosa va fatto. Una ricerca di mercato e una scelta di acquisto per l'interfaccia audio di questo progetto, con i criteri dichiarati prima dei modelli e non dopo.

Da dove nasce, e perché è una voce di questo progetto e non di quello dei monitor. La macchina Ubuntu Studio è condivisa fra due progetti: la progettazione elettroacustica dei monitor, che vive in `diy-2way-monitors-home`, e l'home recording, che vive qui. L'interfaccia oggi disponibile è una Focusrite Scarlett 2i2 di seconda generazione, con due ingressi e alimentazione phantom, e per il progetto dei monitor è sufficiente: lì serve un solo ingresso microfonico per il microfono di misura, e la Scarlett lo copre. Per l'home recording due ingressi sono invece il vincolo principale, quindi l'esigenza è di questo progetto e va decisa qui.

Va detto per chiarezza che al 2026-09-07 la Scarlett non risulta collegata alla macchina: la fotografia dello stato della macchina, nel blocco condiviso sull'ambiente, mostra come sole schede l'audio integrato `ALC887-VD` con le sue uscite HDMI. È una constatazione, non un problema: l'interfaccia va collegata quando serve.

I criteri da fissare prima di guardare i modelli, perché è l'ordine che evita di innamorarsi di una scheda e poi giustificarla.

Quanti ingressi contemporanei servono davvero, e di che tipo: microfonici con phantom, di linea, strumento a alta impedenza, digitali. È il criterio che ordina tutto il resto, perché è quello su cui la Scarlett attuale è insufficiente.

Se serva l'ascolto a latenza bassa con monitoraggio diretto, e se serva un secondo bus di uscita per una mandata separata, per esempio verso una coppia di monitor e una cuffia con mix diverso.

Il supporto su Linux, che non è un dettaglio: le interfacce conformi alla classe audio USB funzionano senza driver proprietari, mentre alcune richiedono software di configurazione che esiste solo per Windows e macOS, e in quel caso funzioni come il mixer interno o il routing restano inaccessibili. Il criterio operativo è preferire dispositivi che espongano tutto il necessario come conformi alla classe, e verificare la compatibilità sulle fonti della comunità Linux audio prima dell'acquisto e non dopo.

La qualità dei preamplificatori e il rumore, che contano ma meno di quanto il marketing suggerisca a questo livello di prezzo, e comunque meno del numero di ingressi se il numero di ingressi è il vincolo.

Il rapporto con l'esistente: se la nuova interfaccia sostituisce la Scarlett o la affianca. Affiancarla ha senso solo se si vuole aggregare gli ingressi, cosa che su Linux con PipeWire è possibile ma aggiunge una complicazione di sincronizzazione dei clock che va messa in conto.

Il criterio di completamento. Una decisione documentata con i criteri, i modelli confrontati, la verifica di compatibilità Linux e la motivazione della scelta, oppure la decisione motivata di non comprare nulla e restare sulla Scarlett.

Nota di priorità. Non blocca nulla del progetto dei monitor, che sulla Scarlett attuale ha tutto ciò che gli serve. Blocca invece l'home recording multitraccia, cioè lo scopo di questo progetto, quindi qui è la prima voce.

## PA-002 - Allineare la copia del template al progetto principale

Data di apertura: 2026-09-07. Stato: aperta, priorità bassa, è una decisione dell'utente su materiale di sistema.

Che cosa va fatto. Cinque modelli sotto `.claude/templates/` di questo progetto non rispettano la convenzione Markdown della riga sorgente unica, mentre le copie corrispondenti in `diy-2way-monitors-home` sì. Sono `CLAUDE.local.md` e, sotto `_notes/`, `DIARIO.md`, `RESOCONTO.md`, `RESUME-PROMPT.md` e `TEST-CHECKLIST.md`.

Da dove viene. La verifica `python tools/md-unwrap.py --check .` eseguita in questo progetto il 2026-09-07 segnala quei cinque file, per un totale di quattordici righe da unire. La stessa verifica nel progetto principale non segnala nulla, e il confronto dei file mostra che sono diversi: la copia del template qui è più vecchia di una propagazione. Non è quindi un difetto nuovo ma un ritardo di allineamento.

Perché non è urgente. Sono modelli, cioè file che si copiano quando si istanzia qualcosa, non documentazione attiva di questo progetto. La non conformità non produce nessun effetto finché uno di essi non viene usato, e in quel caso lo strumento la corregge sul file istanziato.

Il criterio di completamento. La verifica `--check` pulita anche in questo progetto, oppure la decisione motivata di non tenere allineata la copia del template qui, dato che il progetto principale ne conserva una corretta.

Nota sulla relazione con l'altro progetto. Questa voce è parente di PA-003 del progetto principale, che riguarda quattro correzioni da propagare all'indietro verso `template-claude-developing`. Conviene trattarle insieme, perché la sorgente comune è la stessa e correggerla lì risolve entrambe le copie invece di rincorrerle una per una.

## PA-003 - SoundCraft come DAW pilotabile dall'agente, e se serva un fork per Ubuntu Studio

Data di apertura: 2026-10-26, da una sessione aperta nel progetto `home-lab-cybersec-networking`. Stato: aperta, in attesa di riallineamento con l'utente. La stessa voce, con le fonti per esteso, è PA-034 di `diy-2way-monitors-home`, nella cui pagina `docs/90-riferimenti/fonti.md` stanno i riferimenti, tutti letti il 2026-10-26.

Che cosa va fatto. L'utente ha chiesto di fare un fork di SoundCraft, la DAW della suite ArtCraft, e di personalizzarlo per Ubuntu Studio. Poi ha indicato questo progetto e quello dei monitor come i diretti interessati, e ha chiesto una spiegazione prima del dettaglio della personalizzazione, che considera lo sviluppo più interessante.

Che cosa si sa. Per usare il programma il fork non serve, perché upstream pubblica per Linux deb, tarball, AppImage e Flatpak, e il programma gira passando da ALSA e quindi da PipeWire. Serve per cambiare il codice. Upstream dichiara come blocco dell'alpha la mancanza di JACK e della verifica su Linux, il mix gira a blocchi fissi di 512 campioni, non c'è hosting LV2 e l'interscambio di sessioni AAF e OMF manca. Il README stima il programma a circa il 40% di un sostituto reale di Pro Tools. Per registrare, quindi, Ardour resta lo strumento maturo della macchina. Il valore di SoundCraft in questo progetto è altro: la CLI 0.10.0 espone 480 comandi e 27 plugin interni, e con la riga di comando e il server MCP l'agente può costruire sessioni di esercizio, ispezionarle, inserire plugin e fare i bounce. Per un progetto di formazione al missaggio è una capacità che Ardour non offre all'agente. Se si fa il fork, avrà un nome neutro, come ha scelto l'utente.

Condizione di sblocco. Il riallineamento con l'utente su che cosa SoundCraft debba fare qui: esercizi di missaggio generati e corretti dall'agente, oppure una DAW di lavoro.

Criterio di completamento. Una decisione documentata fra rilascio ufficiale e fork. Se si sceglie il fork, servono anche la prima milestone, che per le fonti lette è l'host JACK, e la verifica sulla macchina.

## PA-004 - Sottoprogetto: analizzatore di spettro con Raspberry Pi Pico

Data di apertura: 2026-10-26. Stato: proposto dall'utente come sottoprogetto, con la valutazione che segue, e in attesa della sua decisione.

Che cosa è. Un articolo di Martin Ossmann pubblicato da Futuranet il 2026-10-15 e letto per intero il 2026-10-26 (`https://futuranet.it/progetti/2026/10/15/raspberry-pi-pico-come-analizzatore-di-spettro/`). Descrive un analizzatore di spettro costruito sull'ADC interno dell'RP2040, a 12 bit e fino a 500 kS/s. L'ingresso è portato a metà della tensione di alimentazione da due resistenze, con un condensatore che toglie la componente continua. Il segnale passa per la finestra di Blackman e la FFT, con N da 1024 a 16384. I dati vanno al PC in seriale a 115200 baud, con funzioni di media e MAXHOLD; un display LCD lo rende autonomo, e con un ADC esterno si arriva a 50 MS/s. Il software si scarica dalla pagina Elektor dell'articolo, `https://www.elektormagazine.com/magazine/elektor-312/62150`, non ancora letta, quindi licenza e linguaggio non sono verificati. Gli esempi dell'articolo sono segnali radio a onde lunghe, non audio.

Se ha senso qui. Sì, come progetto di elettronica e di formazione, non come strumento di misura audio, e la distinzione decide come lo si costruisce.

Come strumento di misura perde contro ciò che la macchina ha già. Un ADC a 12 bit ha un rapporto segnale rumore teorico di quantizzazione di circa 74 dB, calcolato con 6,02 per 12 più 1,76 e non misurato: è molto sotto la Scarlett 2i2 a 24 bit. Per la stanza e i diffusori, poi, REW con il microfono di misura del progetto dei monitor resta lo strumento giusto.

Ha però tre usi che nessuna scheda audio copre. Vede fino a 250 kHz, cioè residui ultrasonici che una scheda audio taglia: la portante di commutazione di un amplificatore in classe D, come quelli valutati per i monitor attivi, oppure un'oscillazione parassita in un circuito analogico. È proprio lo strumento da banco che servirebbe alla pedaliera analogica di PA-030 del progetto dei monitor. Insegna poi in modo concreto finestre, risoluzione in frequenza, aliasing e filtro anti-aliasing, cioè i concetti che stanno dietro ogni analizzatore di una DAW, e in un progetto di training sono contenuto e non contorno. Infine, con il display diventa un analizzatore visivo autonomo da tenere in studio.

Due avvertenze prima di collegarlo a una catena audio. La prima riguarda il livello. Con il punto medio a metà dei 3,3 V la dinamica è di circa ±1,65 V, mentre un'uscita professionale a +4 dBu ha picchi di circa 1,74 V; un'uscita per altoparlante è molto più alta. Serve quindi un partitore o uno stadio di adattamento, e il valore dei 3,3 V è una deduzione dallo schema, non letta nell'articolo. La seconda riguarda il collegamento seriale: a 115200 baud passano circa 11 500 byte al secondo, quindi il PC riceve qualche migliaio di campioni al secondo e il refresh resta lento. Va bene per guardare, non per un uso dal vivo. Anche questo è un calcolo, non una misura.

Condizione di sblocco. La decisione dell'utente di aprirlo. Il primo passo è leggere la pagina Elektor del software, per licenza, linguaggio e formato dei dati.

Criterio di completamento. Il sottoprogetto aperto con il proprio perimetro, cioè strumento da banco per la pedaliera, didattica o display da studio, oppure la decisione motivata di non farlo.
