<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# La catena audio su PipeWire: verificare che esca suono, e le otto trappole che lo fanno sembrare guasto

> Pagina prescrittiva, in forma parametrica e riusabile su un'altra macchina Linux con PipeWire. Contiene la sequenza dei soli comandi nell'ordine in cui vanno eseguiti e, in coda, il troubleshooting dei casi realmente incontrati. Il racconto cronologico, con i ritiri e gli errori di conduzione, sta nei microstep MS-146, MS-154, MS-155 e MS-156 di `docs/OPERATIONS-LOG.md`, e non va cercato qui: questa pagina risponde a chi esegue, il registro a chi vuole sapere perché.

## Che cosa questa pagina verifica, e che cosa non può verificare

La domanda a cui la sequenza risponde è se la macchina sia capace di riprodurre suono in modo utilizzabile per un lavoro di elettroacustica, il che vuol dire quattro cose e non una: che la scheda esista per il kernel, che esista anche per il server audio, che l'uscita che serve sia quella attiva, e che i due canali arrivino nell'ordine giusto e con lo stesso livello. Le prime tre si leggono con comandi, la quarta richiede qualcuno che ascolti.

Il limite va dichiarato prima della sequenza, perché nessun comando lo rimuove. Ciò che si accerta è la simmetria della catena elettrica e digitale, non la simmetria acustica alle orecchie o ai microfoni: quella dipende anche dai trasduttori e dal loro accoppiamento, e si misura con un microfono di misura. Una asimmetria di trasduttore è possibile e questa sequenza non la escluderebbe.

## I tre assi che decidono se una misura audio valga qualcosa

Questa è la parte che rende la sequenza diversa da un elenco di comandi, e va letta prima di eseguirli, perché ognuno dei tre assi ha già prodotto una misura senza valore su una macchina reale.

Il primo asse è la sessione da cui si misura. L'accesso ai nodi di `/dev/snd/` non è dato da un permesso fisso ma da una lista di controllo di accesso che il gestore degli accessi assegna alla sessione attiva del posto, quindi una misura raccolta mentre il posto è tenuto da un'altra sessione, tipicamente quella dello schermo di accesso, riporta l'assenza di schede su una macchina perfettamente sana. La conseguenza pratica non è che si debba misurare dalla console: è che si deve leggere chi tenga il posto, e interpretare la misura di conseguenza. Con la sessione dell'utente attiva, una connessione SSH misura correttamente.

Il secondo asse è l'età del processo di gestione audio rispetto alle modifiche ai gruppi. Un processo riceve i propri gruppi quando la sessione che lo genera nasce e non li rilegge mai più, e il processo di gestione audio appartiene al gestore di servizi dell'utente, che resta vivo finché esiste almeno una sessione di quell'utente, comprese le connessioni SSH. Ne segue che un logout grafico non lo rinnova e che nemmeno riavviare il servizio serve, perché un servizio riavviato eredita i gruppi del gestore che lo lancia: l'unica via è il riavvio della macchina. La regola generale è quella enunciata in MS-150, cioè che dopo una installazione che tocca utenti, gruppi o variabili d'ambiente la prima verifica si fa da una sessione aperta dopo.

Il terzo asse è lo stato dei jack. I profili di una scheda integrata sono dichiarati disponibili o non disponibili in base al rilevamento della presenza sulle prese, quindi a jack liberi tutti i profili analogici risultano non disponibili e il server audio non può selezionarli: l'assenza di uscite analogiche descrive fedelmente una presa vuota e non un difetto. Una presa S/PDIF, che non ha rilevamento della presenza, dichiara la propria disponibilità come sconosciuta e non viene mai esclusa, ed è la ragione per cui su molte macchine l'uscita predefinita a freddo è quella digitale.

## La sequenza operativa completa

I comandi si eseguono sulla macchina di cui si verifica l'audio. Dove il lavoro si svolga da una postazione diversa, il primo comando del blocco è la connessione, e va incollato sulla postazione di partenza e non sulla macchina di destinazione.

```bash
ssh <alias della macchina>
```

### Fase 0, accertare da dove si misura

Il primo dato non riguarda l'audio ma la sessione, e senza di esso ogni misura successiva è ambigua. Si legge quale sessione tenga il posto e a quale utente appartenga.

```bash
loginctl list-sessions
```

```bash
loginctl show-seat seat0
```

Il valore che conta è `ActiveSession`. Se corrisponde a una sessione dell'utente che deve usare l'audio, la misura che segue è informativa qualunque sia il canale, SSH compreso. Se corrisponde alla sessione del gestore dello schermo di accesso, la misura non è informativa in nessuna direzione e la sequenza si ferma qui: va fatto un accesso alla sessione grafica, e non basta sbloccare uno schermo.

Dove si voglia la prova diretta di chi possieda l'accesso ai dispositivi, si legge la lista di controllo di accesso su un nodo di controllo della scheda.

```bash
getfacl -p /dev/snd/controlC0
```

### Fase 1, i gruppi del processo di gestione audio

Si confrontano i gruppi dell'utente con quelli che il processo di gestione audio porta davvero, che non sono gli stessi se il processo è nato prima di una modifica ai gruppi.

```bash
id
```

```bash
pgrep -x wireplumber
```

```bash
grep -E "^Groups" /proc/<pid>/status
```

```bash
ps -o lstart= -p <pid>
```

L'ora di avvio serve a datare il processo rispetto alla modifica dei gruppi, e nella pratica dice subito se il processo sia nato con la macchina oppure sia sopravvissuto a giorni di attività. Su questa macchina i gruppi che contano sono `audio` e `pipewire`, che vanno ricavati per numero e non assunti.

```bash
getent group audio pipewire
```

Questo criterio è igiene e non un requisito di funzionamento, e la prova che sia ridondante è che la catena funziona senza di esso, perché l'accesso ai dispositivi arriva dalla lista di controllo di accesso della fase 0. Va soddisfatto comunque, perché una ridondanza che manca è una ridondanza che non c'è il giorno in cui la lista di controllo di accesso non basta.

### Fase 2, la scheda a livello di kernel

Questa fase e la successiva vanno tenute distinte, perché è precisamente fra le due che passa la trappola più comune: una scheda vista dal kernel e non dal server audio non è un difetto della scheda.

```bash
cat /proc/asound/cards
```

```bash
aplay -l
```

### Fase 3, la scheda a livello di server audio

```bash
pactl info
```

```bash
pactl list short cards
```

```bash
pactl list short sinks
```

```bash
wpctl status
```

Il valore che decide è `Default Sink` nell'uscita del primo comando. Un `auto_null` significa che il server audio non ha alcuna scheda a disposizione, e a quel punto la causa sta nella fase 0 o nella fase 1 e non nella scheda.

### Fase 4, i profili e le porte, cioè perché l'uscita che serve non c'è

È la fase che spiega il novanta per cento dei casi in cui il suono non esce da dove dovrebbe, ed è quella che di solito non si guarda.

```bash
pactl list cards
```

Di quella uscita, lunga, si leggono tre cose. La riga `Active Profile`, che dice quale profilo è in uso. La disponibilità dichiarata accanto a ogni profilo, dove `available: no` su tutti i profili analogici significa jack liberi e non guasto. E la disponibilità dichiarata accanto a ogni porta, dove una presa digitale dichiara `availability unknown` perché non ha rilevamento della presenza.

Le priorità dei profili, scritte sulla stessa riga, spiegano la selezione automatica: a parità di disponibilità vince la priorità più alta, e su una scheda integrata il duplex analogico ha priorità maggiore del digitale. Ne segue che collegare un jack analogico fa passare il server audio all'analogico da sé, senza che nessuno scelga nulla, e che scollegarlo lo fa tornare al digitale.

```bash
pactl list sinks
```

Di questa uscita servono la porta attiva, lo stato di silenziamento e i volumi per canale, che sono le tre cose che rendono muta una uscita perfettamente configurata.

### Fase 5, la prova di ascolto, che richiede che chi ascolta sia dove esce il suono

Prima di ogni prova si accerta che non ne sia già in corso un'altra, perché due riproduzioni sovrapposte producono un esito che sembra informativo e non lo è.

```bash
pgrep speaker-test
```

```bash
pactl list short sink-inputs
```

La prima prova è a due canali con un segnale parlato, che dichiara il proprio nome e non richiede di conoscere l'istante iniziale.

```bash
speaker-test -c 2 -t wav
```

La seconda è un tono puro, che serve a giudicare la pulizia dell'uscita e non l'ordine dei canali.

```bash
speaker-test -c 2 -t sine -f 440
```

Due avvertenze di conduzione, entrambe nate da prove sprecate e non da prudenza teorica. Chi ascolta deve essere fisicamente dove esce il suono, il che su un lavoro condotto da una seconda macchina non è implicito e va detto prima di lanciare. E una riproduzione avviata con un limite di tempo non è finita perché il tempo sembra passato: lo stato si legge con i due comandi qui sopra, non si deduce.

### Fase 6, l'ordine dei canali, che un tono alternato non dice

Un tono puro non dichiara la propria identità, quindi una prova alternata accerta che i canali siano due e distinti ma non quale sia quale: l'informazione sull'ordine sta nell'istante iniziale, che l'ascoltatore perde se arriva a riproduzione avviata, e un cablaggio invertito produrrebbe la stessa osservazione. Le due forme che risolvono il problema sono il segnale parlato, indipendente dal momento in cui si ascolta, e una sequenza con una pausa iniziale dichiarata, che riporta l'istante iniziale sotto il controllo di chi ascolta.

La forma usata su `alessio-ubuntustudio` il 2026-09-21 è la seconda, su preferenza dell'utente per il tono puro: trenta secondi di silenzio dichiarati, venti secondi sul solo canale sinistro, quattro secondi di silenzio, venti secondi sul solo canale destro. Della sua invocazione esatta il registro conserva la struttura e l'esito e non il testo del comando, quindi la forma che segue è quella che produce quella struttura e va confermata alla prima riesecuzione invece di essere data per verificata: `speaker-test` seleziona il canale singolo con l'opzione `-s`, dove `1` è il sinistro e `2` il destro.

Il criterio è che il primo tono si senta a sinistra e il secondo a destra. Una inversione va accertata qui e non dopo, perché si propaga a ogni misura stereo successiva ed è invisibile all'ascolto ordinario.

### Fase 7, la simmetria fra i canali

Si legge su due livelli, perché uno sbilanciamento può vivere nel server audio o nel mixer della scheda, e i due non si vedono a vicenda.

```bash
wpctl get-volume @DEFAULT_AUDIO_SINK@
```

```bash
amixer -c 0 scontents
```

Del secondo si guardano i soli controlli stereo di riproduzione, verificando che i due canali riportino lo stesso valore su ciascuno. Un controllo a zero e disattivato non è un guasto e non va scambiato per tale: è una uscita silenziata a livello di mixer, e sulla macchina di riferimento è il caso del controllo del pannello frontale mentre si usa l'uscita di linea posteriore.

### Fase 8, lo stato conservato su disco, che dice se una scelta esista

Questa fase serve a distinguere una preferenza registrata da una selezione automatica, e senza di essa le due si confondono a ogni riavvio.

```bash
ls -la ~/.local/state/wireplumber/
```

```bash
cat ~/.local/state/wireplumber/default-routes
```

Se non esiste alcun file `default-profile`, nessuna preferenza di profilo è mai stata registrata e ciò che si osserva è selezione automatica per disponibilità e priorità. Ne segue che non c'è nulla da rendere persistente e che un ritorno al profilo digitale dopo un riavvio non è una impostazione perduta. La data di questi file va guardata: su una macchina reinstallata conservando `/home` possono essere più vecchi del sistema, e il loro contenuto descrive allora una configurazione che non esiste più.

## Portabilità: che cosa cambia da una macchina all'altra e che cosa no

Non cambiano i tre assi, che sono proprietà del gestore degli accessi e del server audio e non di una macchina, né l'ordine delle otto fasi, che va dal contesto verso il dettaglio proprio perché una misura raccolta nel contesto sbagliato non si distingue da una corretta.

Cambiano i valori. Il nome della scheda e quello dei sink dipendono dal percorso sul bus, quindi vanno letti e non copiati da qui. I numeri dei gruppi `audio` e `pipewire` sono assegnati alla creazione e differiscono fra installazioni, e si ricavano con `getent group`. Il nome del posto è `seat0` su una macchina con un solo posto e va riletto dove ce ne sia più di uno. L'indice della scheda passato ad `amixer` è zero solo se la scheda da verificare è la prima, che è vero su una macchina con la sola integrata e falso appena se ne aggiunge una esterna.

Un caso che cambia la sequenza e non i soli valori merita di essere previsto qui, perché è quello che questa macchina incontrerà: con due schede presenti la selezione automatica per priorità non basta più, perché la priorità non sa quale delle due serva al lavoro, e il dispositivo predefinito diventa una scelta da dichiarare. I comandi che la dichiarano non compaiono in questa sequenza perché su questa macchina non sono stati eseguiti, e non vanno scritti qui prima di averlo fatto.

## Troubleshooting: sintomo, causa, rimedio

Ogni voce nasce da un caso osservato su una macchina reale fra il 20 e il 21 settembre 2026 e non da una previsione. L'ordine è quello in cui i casi si incontrano percorrendo la sequenza.

### A, `Default Sink: auto_null` mentre il kernel vede la scheda

Sintomo: `pactl info` riporta `auto_null`, `pactl list short cards` non riporta alcuna scheda e `wpctl status` mostra la sola uscita fittizia, mentre `/proc/asound/cards` e `aplay -l` riportano regolarmente la scheda integrata. Causa: la sessione attiva del posto non è quella dell'utente, tipicamente perché lo schermo di accesso la tiene, quindi la lista di controllo di accesso sui nodi di `/dev/snd/` è assegnata a quell'altra sessione. Rimedio: fare un accesso alla sessione grafica dell'utente, non sbloccare uno schermo, e ripetere la misura.

La parte da non sbagliare è l'interpretazione prima del rimedio. Questa misura non dice che la catena audio sia guasta e non dice che funzioni: è vacua, e trattarla come un esito è il difetto che la regola sulle prove che misurano descrive. Osservato su `alessio-ubuntustudio` il 2026-09-20 dentro la fotografia della fase 11, dove per giunta compariva accanto a decine di voci corrette, che è il modo in cui una misura senza valore passa inosservata.

### B, la stessa misura da SSH che prima era cieca e ora non lo è

Sintomo: una pagina o un microstep dichiarano che la misura raccolta da SSH non sia informativa, e la misura raccolta da SSH è invece corretta. Causa: quella dichiarazione era contingente sullo stato del posto e non sul canale, e lo stato è cambiato, tipicamente per un riavvio o per un accesso alla console. Rimedio: riprovare il canale invece di darlo per cieco, e correggere la portata dell'affermazione dove è scritta.

Vale come regola oltre l'audio, ed è la stessa che governa le etichette di indisponibilità delle fonti: una etichetta che sopravvive alla propria causa scoraggia dal misurare, e costa più della sua assenza. Osservato il 2026-09-21 su `alessio-ubuntustudio`, ed è la correzione di portata registrata in MS-156.

### C, l'utente appartiene ai gruppi e il processo audio no

Sintomo: `id` elenca i gruppi `audio` e `pipewire`, mentre la riga `Groups` del processo di gestione audio non li contiene. Causa: quel processo è nato prima che i gruppi fossero assegnati, e un processo non rilegge mai i propri gruppi; appartenendo al gestore di servizi dell'utente, sopravvive al logout grafico finché esiste una qualunque sessione di quell'utente, connessioni SSH comprese. Rimedio: riavviare la macchina. Riavviare il servizio non serve, perché eredita i gruppi del gestore che lo lancia.

Osservato su `alessio-ubuntustudio` il 2026-09-20, con un processo nato il 2026-09-09 alle 12:57, cioè prima che l'utente fosse aggiunto ai due gruppi lo stesso giorno più tardi, e chiuso dal riavvio del 2026-09-21 alle 16:04. È la terza occorrenza in dodici giorni della stessa forma, dopo il PATH non rilette da un terminale aperto e il gruppo `veeam` mancante in una sessione nata prima dell'installazione del pacchetto.

### D, tutti i profili analogici dichiarati non disponibili

Sintomo: `pactl list cards` riporta `available: no` su ogni profilo analogico e le porte di uscita analogica come non disponibili, mentre i profili digitali sono disponibili. Causa: il rilevamento della presenza sui jack funziona e le prese sono libere. Rimedio: collegare qualcosa al jack, dopo il che i profili passano a disponibili e il server audio seleziona da sé l'analogico, che ha priorità maggiore.

Non c'è nulla da configurare, ed è il punto in cui si perde tempo cercando un driver. Osservato su `alessio-ubuntustudio` il 2026-09-21 in entrambi i versi, cioè il passaggio a disponibile collegando le cuffie e il ritorno a non disponibile dopo il riavvio a prese libere.

### E, l'uscita torna digitale dopo ogni riavvio

Sintomo: il sink predefinito è quello S/PDIF a ogni avvio, anche dopo aver usato l'uscita analogica. Causa: nessuna preferenza di profilo è registrata, perché il passaggio all'analogico era una selezione automatica e non una scelta, e a prese libere il profilo digitale è il più prioritario fra quelli disponibili, dato che una presa S/PDIF non ha rilevamento della presenza e dichiara la propria disponibilità come sconosciuta invece di dichiararsi vuota. Rimedio: nessuno, se il jack analogico verrà occupato, perché l'analogico torna da sé; dove serva una uscita fissa indipendente dai jack, la preferenza va dichiarata esplicitamente.

La diagnosi che separa questo caso da una impostazione perduta è la fase 8: l'assenza del file `default-profile` dimostra che non c'era alcuna scelta da perdere. Osservato su `alessio-ubuntustudio` il 2026-09-21.

### F, una prova di ascolto dà un esito che non corrisponde ad alcuna prova

Sintomo: l'ascoltatore riferisce una sequenza che nessuna delle prove lanciate produce, per esempio un canale ripetuto e poi entrambi. Causa: due riproduzioni sovrapposte sullo stesso dispositivo, perché la precedente non era terminata; il suo limite di tempo era più lungo di quanto si credesse e una riproduzione non lascia traccia visibile a chi la lancia da remoto. Rimedio: fermare tutto, verificare il silenzio con `pgrep` sul processo di prova e con il conteggio dei flussi verso l'uscita, e rifare la prova.

Osservato il 2026-09-21 su `alessio-ubuntustudio`, ed è un errore di conduzione mio e non della catena. La regola che ne discende sta nella fase 5: lo stato di una riproduzione si legge, non si deduce dal tempo trascorso.

### G, la prova conferma i canali ma non il loro ordine

Sintomo: l'ascoltatore riferisce che i canali si alternano correttamente, e resta impossibile dire se il sinistro sia il sinistro. Causa: un tono alternato porta l'informazione sull'ordine nel solo istante iniziale, che si perde sedendosi a riproduzione avviata; un cablaggio invertito produce la stessa osservazione. Rimedio: usare un segnale parlato, che dichiara il proprio nome, oppure una sequenza con pausa iniziale dichiarata, come nella fase 6.

Osservato due volte di fila il 2026-09-21 su `alessio-ubuntustudio`, ed è la ragione per cui la fase 6 esiste come fase separata dalla 5.

### H, l'uscita HDMI muta con il volume a zero

Sintomo: selezionando l'uscita HDMI non si sente nulla, con lo stato di silenziamento disattivato, che è la forma in cui una configurazione si maschera da guasto. Causa: lo stato conservato da WirePlumber registra per quella porta i volumi dei due canali a zero. Rimedio: alzare il volume di quella porta, oppure rimuovere la voce dallo stato conservato, dopo averne letto il contenuto.

Su `alessio-ubuntustudio` la voce esiste, porta la data del 14 novembre 2025 ed è un residuo del sistema precedente, sopravvissuto dentro `/home` alla reinstallazione dell'8 settembre 2026. Oggi è inerte perché l'HDMI non è in uso, e sta qui perché si manifesterebbe molto dopo, quando nessuno collegherebbe più il sintomo alla sua causa. Di passaggio è una conferma indipendente che `/home` sia sopravvissuto: un file di stato di undici mesi prima non potrebbe esistere su un filesystem creato a settembre.

## Il criterio che chiude

La verifica è compiuta quando quattro cose sono vere insieme e ciascuna è stata osservata e non dedotta: il posto è tenuto dalla sessione dell'utente, la scheda esiste per il kernel e per il server audio, la porta attiva è quella che serve al lavoro, e una prova con l'ordine dei canali accertato è stata udita da una persona che si trovava dove esce il suono. Il criterio sui gruppi del processo di gestione audio si aggiunge come igiene, non come requisito.

Una prova verde su tre criteri su quattro non è una verifica parziale ma una verifica non fatta, perché il criterio che manca è sempre quello che decide: la catena può essere perfetta e riprodurre nel posto sbagliato, o riprodurre nel posto giusto con i canali invertiti, e in entrambi i casi ogni misura successiva eredita il difetto senza dichiararlo.
