<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Wine, un emulatore e una macchina virtuale: tre cose diverse

> Pagina concettuale, e la più importante di questo blocco. Chiarisce che cosa fa Wine e in che modo differisce da un emulatore e da una macchina virtuale. Non è una digressione teorica: quasi tutte le decisioni pratiche dell'ambiente, dalla scelta di 32 o 64 bit alla sopravvivenza della licenza di Akabak dopo una reinstallazione, sono conseguenze dirette di questa distinzione. Chi la salta si trova poi a diagnosticare i guasti nella categoria sbagliata.

## Il nome dice già la risposta

Wine è un acronimo ricorsivo, *Wine Is Not An Emulator*, e la negazione è il punto. Wine non simula un computer e non esegue Windows: reimplementa, in spazio utente e su Linux, l'interfaccia di programmazione che i programmi Windows si aspettano di trovare.

## Che cosa succede davvero quando un programma parte

Vale seguire il percorso concreto, perché è qui che la differenza fra i tre approcci si vede meglio che in qualunque definizione.

Un eseguibile Windows è un file in formato PE[^1]. Non è codice che il kernel Linux sappia avviare da sé: Linux si aspetta il formato ELF. Il primo lavoro di Wine è quindi fare da caricatore. Legge le intestazioni del PE, capisce di quali librerie il programma ha bisogno, alloca la memoria negli indirizzi che il file dichiara, e mette in esecuzione il codice macchina che vi trova.

Quel codice macchina non viene toccato. È il punto che sorprende più spesso, e va detto con precisione: un eseguibile Windows a 64 bit contiene istruzioni x86-64, e il processore Intel i7-6700 di questa macchina esegue istruzioni x86-64. Sono le stesse istruzioni. Girano direttamente sul silicio, senza interprete, senza traduzione e senza ricompilazione. Una moltiplicazione dentro Akabak costa esattamente quello che costerebbe se Akabak girasse su Windows sulla stessa macchina.

La traduzione avviene a un livello del tutto diverso: non sulle istruzioni, ma sulle chiamate ai servizi del sistema. Quando il programma vuole aprire un file, non emette una istruzione speciale: chiama una funzione di libreria, per esempio `CreateFileW` di `kernel32.dll`. Su Windows quella funzione starebbe in una libreria Microsoft che, dopo qualche passaggio, entrerebbe nel kernel Windows. Su Linux, Wine fornisce la propria implementazione di `kernel32.dll`, e quella implementazione traduce la richiesta in ciò che Linux capisce, cioè una `open` sul filesystem reale, con il percorso `C:\...` convertito nel percorso Linux corrispondente.

Il rapporto numerico fra i due livelli è ciò che spiega le prestazioni. Il codice di calcolo, che è la quasi totalità del tempo di una simulazione, non paga nulla. Le chiamate di sistema pagano un piccolo sovrapprezzo di traduzione, e sono relativamente poche. È per questo che un simulatore acustico sotto Wine gira a velocità paragonabile a quella nativa, mentre lo stesso programma sotto un emulatore di CPU sarebbe lento in modo evidente.

## Le tre conseguenze che contano

Dalla struttura appena descritta discendono tre proprietà, e conviene enunciarle separatamente perché sono i tre punti su cui l'intuizione comune sbaglia.

### Non esiste un kernel Windows, quindi "Windows 10" è solo una stringa

Wine implementa le librerie di sistema, non il nucleo del sistema operativo, non i driver e non i servizi. Non c'è `ntoskrnl` in esecuzione, non c'è un registro di sistema Windows vero, non c'è il modello dei driver.

Ne segue una conseguenza che nel documento sorgente era stata intuita correttamente e che merita di essere resa esplicita e spiegata fino in fondo. Impostare *Windows 10* in `winecfg` non installa Windows 10. Quella impostazione scrive un valore in un registro finto, un file dentro il prefix, e i programmi lo leggono chiamando una funzione come `GetVersionEx` per decidere quali API considerare disponibili. È una dichiarazione, non una installazione: cambiarla non aggiunge e non toglie una riga di codice al sistema.

La ragione per cui *Windows 10* è la scelta corretta, e non un compromesso, è che i programmi di questo progetto sono collaudati su quel profilo e chiedono API recenti come .NET Framework 4.8, le librerie WinMM per l'audio e WASAPI. Un profilo Windows 7 o inferiore farebbe loro credere che quelle API non ci siano. Wine sa dichiararsi anche Windows 11, ma il profilo Windows 10 ha compatibilità migliore e meno difetti noti.

Ne segue anche la risposta alla domanda sulla sicurezza. Le vulnerabilità di Windows riguardano codice Windows in esecuzione, e qui non ce n'è: nessuna patch di Microsoft è pertinente a questa macchina. Le vulnerabilità di cui preoccuparsi sono quelle di Wine, corrette dal progetto WineHQ, e il perimetro è quello di un programma in spazio utente con i privilegi dell'utente che lo lancia, non quello di un sistema operativo intero.

### Il filesystem è quello vero, quindi un prefix è una cartella

Un *prefix*, cioè quello che a volte viene chiamato *bottle*, è una cartella normale nella home dell'utente Linux. Contiene un albero `drive_c` con `Program Files` e `Windows/System32`, una cartella `dosdevices` che mappa le lettere di unità su percorsi Linux, e i file del registro finto.

Un programma Windows che scrive nella cartella `C:\Program Files\RD Team\AKABAK` sta scrivendo dentro `~/wineprefixes/akabak64/drive_c/Program Files/RD Team/AKABAK`, e quella cartella si elenca con `ls`, si cerca con `grep`, si copia, si archivia e si mette in un backup come qualsiasi altra. È il motivo per cui salvare un ambiente Windows configurato, su Linux, è una singola copia ricorsiva di directory, e trasferirlo su un'altra macchina Linux è la stessa copia.

Una macchina virtuale, per contrasto, tiene tutto dentro un file immagine di disco. Da fuori è opaco: per leggere un file dell'ospite si accende l'ospite, oppure si monta l'immagine con strumenti appositi. La differenza pratica, su un progetto in cui si vuole ispezionare e versionare ciò che i programmi producono, è grande.

### L'hardware visto dal programma è quello reale, e qui si decide la licenza

Questo è il punto che ha la conseguenza economica più concreta, e vale seguirlo su un esempio reale invece che in astratto.

Akabak protegge la licenza con un identificativo di macchina. Quando il programma lo calcola, chiama funzioni Windows che interrogano l'hardware: il tipo e il numero di serie del processore, l'indirizzo MAC di una interfaccia di rete, l'identificativo del disco, o una combinazione di questi. Wine implementa quelle funzioni leggendo l'hardware vero attraverso Linux. Il risultato è che l'identificativo calcolato è quello del i7-6700 di questa macchina.

Il fatto verificato che ne segue, documentato in `docs/90-riferimenti/timeline-akabak-vacs.md`, è che il Release Code ottenuto dall'autore nell'agosto 2025 è permanente e legato a quell'identificativo. Poiché l'identificativo non dipende dal prefix, né dall'installazione di Wine, né dall'installazione di Ubuntu, tutte queste cose si possono azzerare e ricostruire: al programma reinstallato si reinserisce lo stesso codice e funziona.

In una macchina virtuale l'esito sarebbe opposto. Lo strato di virtualizzazione presenta all'ospite un hardware virtuale, con un MAC generato, un disco virtuale con un proprio identificativo e un processore mascherato. L'identificativo calcolato sarebbe quello della macchina virtuale, diverso da quello reale, e il codice esistente non sarebbe valido. Peggio, cambierebbe di nuovo se si ricreasse la macchina virtuale con parametri diversi.

I due casi che invaliderebbero la licenza restano quindi soltanto due: un cambio significativo di hardware, per esempio una nuova scheda madre, e lo spostamento del programma dentro una macchina virtuale.

## Che cosa fa invece un emulatore

Un emulatore riproduce in software una macchina diversa da quella su cui gira. Il suo lavoro non è tradurre chiamate di libreria ma eseguire istruzioni che il processore reale non conosce.

QEMU che esegue un sistema ARM su un PC x86 legge le istruzioni ARM una per una e, per ciascuna, esegue la sequenza di istruzioni x86 che ne riproduce l'effetto sui registri e sulla memoria simulati. DOSBox ricrea un PC 8086 completo, con la sua scheda video, il suo timer e la sua Sound Blaster, perché i programmi DOS parlavano direttamente con quei dispositivi invece di passare da un sistema operativo.

Sopra quella macchina finta gira un sistema operativo vero, con il suo kernel e i suoi driver, che non sa di essere emulato.

Il prezzo è la velocità, perché ogni istruzione della macchina ospite costa molte istruzioni della macchina ospitante, anche con la ricompilazione dinamica che riduce il divario traducendo blocchi interi invece di singole istruzioni. Il guadagno è la generalità, perché si può eseguire software compilato per una architettura che il processore reale non conosce affatto.

C'è un caso limite che illumina esattamente dove passa il confine, e vale citarlo perché è il controesempio che rende la definizione precisa. Su un computer con processore ARM, per esempio un Mac recente o un Raspberry Pi, Wine da solo non basta a eseguire un programma Windows compilato per x86: le istruzioni sono diverse e qualcuno le deve tradurre. In quel caso si affianca a Wine un emulatore di istruzioni come FEX-Emu o box64, e la pila diventa emulatore più strato di compatibilità. Wine resta non-emulatore anche lì: è che l'emulatore serve, e sta sotto. Su questa macchina, che ha un processore x86-64 e programmi x86-64, quel livello non c'è affatto.

## Che cosa fa una macchina virtuale

Una macchina virtuale, con VirtualBox, VMware o KVM, esegue il vero kernel Windows sullo stesso processore fisico, sfruttando le estensioni di virtualizzazione della CPU. Non c'è traduzione di istruzioni, quindi le prestazioni di calcolo sono buone e vicine al nativo: da questo punto di vista somiglia a Wine più che a un emulatore.

La differenza sta in tutto il resto. C'è un sistema operativo ospite completo e separato, con il suo kernel: serve una licenza Windows, servono i suoi aggiornamenti di sicurezza, e la superficie di attacco è quella di un Windows intero. I dispositivi sono modelli virtuali con i loro driver, e il passaggio di un dispositivo reale all'ospite, per esempio una interfaccia audio USB, aggiunge uno strato che si paga in latenza e in stabilità. Il filesystem dell'ospite è un file immagine opaco. E l'identità hardware è quella virtuale.

## Il confronto, in una tabella

| Aspetto | Wine | Emulatore | Macchina virtuale |
|---|---|---|---|
| Cosa riproduce | l'interfaccia di programmazione di Windows | l'hardware e la CPU di un'altra macchina | l'hardware di un PC, con il kernel ospite vero |
| Livello a cui traduce | chiamate di libreria e di sistema | singole istruzioni della CPU | nessuno per la CPU, i dispositivi sono modelli |
| Istruzioni CPU del programma | eseguite direttamente sul silicio | interpretate o ricompilate | eseguite direttamente, con assistenza hardware |
| Formato eseguibile | PE caricato da Wine | PE caricato dall'OS ospite | PE caricato dall'OS ospite |
| Kernel Windows | assente | presente, dentro l'ospite | presente |
| Licenza Windows | non serve | serve, se si installa Windows | serve |
| Patch di sicurezza Windows | non pertinenti | pertinenti all'ospite | pertinenti all'ospite |
| Prestazioni di calcolo | vicine al nativo | sensibilmente inferiori | vicine al nativo |
| Filesystem visto da Linux | cartelle normali in home | immagine disco opaca | immagine disco opaca |
| Identità hardware vista dal programma | quella della macchina reale | quella emulata, arbitraria | quella virtuale, diversa dalla reale |
| Catena audio | quella nativa di Linux | simulata | virtualizzata, con latenza aggiuntiva |
| Comunicazione fra processi via COM | parziale, vedi il limite di Akabak e VACS | completa dentro l'ospite | completa dentro l'ospite |
| Forma tipica dei guasti | libreria mancante, API non implementata | lentezza, dispositivo non emulato | driver, configurazione dell'ospite |

## Perché la distinzione conta in questo progetto

Cinque conseguenze pratiche, tutte già rilevanti per il lavoro fatto o da fare su questa macchina.

La licenza di Akabak sopravvive a qualunque azzeramento, per il meccanismo spiegato sopra. È la ragione per cui la strada della reinstallazione pulita, descritta in `docs/10-ambiente/installazione-pulita-26-04.md`, non mette a rischio nulla, ed è la ragione per cui la decisione registrata come ADR-003 non si riapre.

La catena audio resta quella nativa. REW, Blender, Octave e la Scarlett 2i2 parlano con PipeWire e JACK direttamente, con la latenza del kernel a bassa latenza di Ubuntu Studio. Se la parte di progettazione girasse in una macchina virtuale Windows, la latenza della catena e il passaggio USB attraverso lo strato di virtualizzazione diventerebbero un problema misurabile, e proprio nella fase in cui la precisione temporale è il dato che si sta cercando.

Il backup e il ripristino sono banali, perché salvare l'intero ambiente Akabak configurato è una copia ricorsiva della cartella del prefix.

I guasti hanno una forma diversa, e riconoscerla accorcia la diagnosi. In una macchina virtuale un programma che non parte è tipicamente un problema di driver o di configurazione dell'ospite. In Wine è quasi sempre una libreria mancante o una API non implementata: è la ragione per cui gli errori documentati in questo progetto sono del tipo *could not load kernel32.dll*, e la cura è installare il runtime giusto nel prefix giusto, non riparare un sistema operativo.

Il prezzo da pagare esiste, ed è uno solo, noto e circoscritto. Wine implementa COM soltanto in parte, e l'autore di Akabak ha dichiarato che per questo il passaggio dei dati fra Akabak e VACS su Linux avviene attraverso gli appunti di sistema invece che via COM. È un passo manuale in più a ogni iterazione della fase 5. Il confronto corretto è fra quel passo manuale e la somma di licenza Windows, latenza sulla misura e licenza Akabak da rifare: il primo costo è chiaramente il minore. Il dettaglio sta in `docs/90-riferimenti/timeline-akabak-vacs.md`.

## Il limite generale

Wine non copre tutto, e va detto invece di scoprirlo. Un programma che richiede un driver in kernel space, una chiave hardware con driver proprietario, o un sistema anti-manomissione che lavora sotto il livello delle API utente, non funziona sotto Wine per costruzione, perché tutte queste cose vivono in un livello che Wine non ha. In quei casi la macchina virtuale è la risposta corretta.

Per i programmi di questo progetto il caso non si presenta: Akabak, VACS, VituixCAD ed EASE Focus sono applicazioni desktop che chiedono .NET Framework[^2], i runtime Visual C++ e i font di sistema, e tutte e quattro rientrano nel dominio in cui Wine funziona bene. Che funzionino non è una previsione: Akabak e VACS girano su questa macchina, licenziati e verificati, dal 3 settembre 2025.

[^1]: *PE*, Portable Executable - formato dei file eseguibili e delle librerie di Windows, diverso dal formato ELF usato da Linux; Wine ne implementa il caricatore, che è il primo pezzo di lavoro necessario per avviare un programma Windows senza Windows.

[^2]: *.NET Framework* - insieme di librerie e ambiente di esecuzione Microsoft su cui sono compilate molte applicazioni desktop Windows; sotto Wine si installa nel singolo prefix tramite winetricks, e la versione richiesta dai programmi di questo progetto è la 4.8.
