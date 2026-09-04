<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Wine, un emulatore e una macchina virtuale: tre cose diverse

> Pagina concettuale. Chiarisce che cosa fa Wine e in che modo differisce da un emulatore e da una macchina virtuale. Non è una digressione teorica: quasi tutte le decisioni pratiche dell'ambiente, dalla scelta di 32 o 64 bit alla sopravvivenza della licenza di Akabak dopo una reinstallazione, sono conseguenze dirette di questa distinzione.

## Il nome dice già la risposta

Wine è un acronimo ricorsivo, *Wine Is Not An Emulator*, e la negazione è il punto. Wine non simula un computer e non esegue Windows: reimplementa in spazio utente, su Linux, l'interfaccia di programmazione che i programmi Windows si aspettano di trovare. Quando un eseguibile chiama una funzione di `kernel32.dll` per aprire un file, Wine non passa quella chiamata a Windows, perché Windows non c'è: la traduce nella chiamata di sistema Linux equivalente, la esegue e restituisce al programma un risultato nella forma che il programma si aspetta.

Da questo discendono tre proprietà che vale enunciare separatamente, perché sono i tre punti su cui l'intuizione comune sbaglia.

La prima è che le istruzioni della CPU non vengono tradotte. Un eseguibile Windows a 64 bit contiene istruzioni x86-64, e il processore Intel i7-6700 di questa macchina esegue istruzioni x86-64: sono le stesse istruzioni, girano direttamente sul silicio, senza interprete nel mezzo. È per questo che le prestazioni sono vicine a quelle native e non c'è la penalità tipica dell'emulazione.

La seconda è che non esiste un kernel Windows. Wine implementa le librerie di sistema, non il nucleo del sistema operativo, non i driver e non i servizi. Ne segue una conseguenza che nel documento sorgente era stata intuita correttamente e che merita di essere resa esplicita: impostare *Windows 10* in `winecfg` non installa Windows 10 e non eredita le sue vulnerabilità. Quella impostazione scrive un numero di versione in un registro finto, che i programmi leggono per decidere quali API usare. Le vulnerabilità di cui preoccuparsi sono quelle di Wine, che le corregge il progetto WineHQ, non quelle di Windows, che su questa macchina non ha alcun codice in esecuzione.

La terza è che il filesystem è quello vero. Un *prefix*, cioè quello che a volte viene chiamato *bottle*, è una cartella normale nella home dell'utente Linux che contiene un albero `drive_c` con `Program Files` e `Windows/System32`, più un file di registro. Un programma Windows che scrive nella cartella `C:\Program Files\RD Team\AKABAK` sta scrivendo dentro `~/.wine/drive_c/Program Files/RD Team/AKABAK`, e quella cartella si copia, si archivia e si versiona come qualsiasi altra. È il motivo per cui il backup di un ambiente Windows intero, su Linux, è una singola copia ricorsiva di directory.

## Che cosa fa invece un emulatore

Un emulatore riproduce in software una macchina diversa da quella su cui gira. QEMU che esegue un sistema ARM su un PC x86, o DOSBox che ricrea un PC 8086 completo con la sua scheda video e la sua Sound Blaster, non traducono chiamate di libreria: interpretano o ricompilano dinamicamente le istruzioni della CPU emulata e simulano i dispositivi hardware. Sopra quella macchina finta gira un sistema operativo vero, con il suo kernel e i suoi driver.

Il prezzo è la velocità, perché ogni istruzione della macchina ospite costa molte istruzioni della macchina ospitante, e il guadagno è la generalità, perché si può eseguire software compilato per una architettura che il processore reale non conosce.

C'è un caso limite che illumina esattamente dove passa il confine, e vale citarlo perché è il controesempio che rende la definizione precisa. Su un computer con processore ARM, per esempio un Mac recente o un Raspberry Pi, Wine da solo non basta a eseguire un programma Windows compilato per x86: le istruzioni sono diverse e qualcuno le deve tradurre. In quel caso si affianca a Wine un emulatore di istruzioni come FEX-Emu o box64, e la pila diventa emulatore più strato di compatibilità. Wine resta non-emulatore anche lì: è che l'emulatore serve, e sta sotto.

## Che cosa fa una macchina virtuale

Una macchina virtuale, con VirtualBox, VMware o KVM, esegue il vero kernel Windows sullo stesso processore fisico, sfruttando le estensioni di virtualizzazione della CPU. Non c'è traduzione di istruzioni, quindi le prestazioni di calcolo sono buone, ma c'è un sistema operativo ospite completo e separato: serve una licenza Windows, servono i suoi aggiornamenti di sicurezza, i dispositivi sono modelli virtuali con i loro driver, e il filesystem dell'ospite è un file immagine opaco visto da Linux.

La tabella riassume i tre approcci sui punti che contano per questo progetto.

| Aspetto | Wine | Emulatore | Macchina virtuale |
|---|---|---|---|
| Cosa riproduce | l'interfaccia di programmazione di Windows | l'hardware e la CPU di un'altra macchina | l'hardware di un PC, con il kernel ospite vero |
| Istruzioni CPU | eseguite direttamente | interpretate o ricompilate | eseguite direttamente, con assistenza hardware |
| Kernel Windows | assente | presente, dentro l'ospite | presente |
| Licenza Windows | non serve | serve, se si installa Windows | serve |
| Patch di sicurezza Windows | non pertinenti | pertinenti all'ospite | pertinenti all'ospite |
| Prestazioni | vicine al nativo | sensibilmente inferiori | vicine al nativo per la CPU |
| Filesystem visto da Linux | cartelle normali in home | immagine disco opaca | immagine disco opaca |
| Identità hardware vista dal programma | quella della macchina reale | quella emulata, arbitraria | quella virtuale, diversa dalla reale |
| Catena audio | quella nativa di Linux | simulata | virtualizzata, con latenza aggiuntiva |

## Perché la distinzione conta in questo progetto

Quattro conseguenze pratiche, tutte già rilevanti per il lavoro fatto o da fare su questa macchina.

La licenza di Akabak è legata all'identificativo hardware che il programma legge dalla macchina. Poiché Wine espone l'hardware vero, quell'identificativo è quello del i7-6700 e resta lo stesso attraverso qualunque cancellazione e ricreazione dei prefix, o anche una reinstallazione completa di Ubuntu Studio. Su una macchina virtuale l'identificativo sarebbe quello virtuale, diverso, e la licenza andrebbe richiesta di nuovo. Questo è il motivo per cui la strada della reinstallazione pulita, discussa nella pagina sull'aggiornamento, non mette a rischio la licenza.

La catena audio resta quella nativa. REW, Blender, Octave e la Scarlett 2i2 parlano con PipeWire e JACK direttamente, con la latenza del kernel a bassa latenza di Ubuntu Studio. Se la parte di progettazione girasse in una macchina virtuale Windows, la latenza della catena e il passaggio USB attraverso lo strato di virtualizzazione diventerebbero un problema misurabile, e proprio nella fase in cui la precisione temporale è il dato che si sta cercando.

Il backup e il ripristino sono banali, perché salvare l'intero ambiente Akabak configurato è una copia ricorsiva della cartella del prefix, e trasferirlo su un'altra macchina Linux è la stessa copia. È una proprietà che una immagine disco virtuale non ha.

I guasti hanno una forma diversa, e riconoscerla accorcia la diagnosi. In una macchina virtuale un programma che non parte è tipicamente un problema di driver o di configurazione dell'ospite. In Wine è quasi sempre una libreria mancante o una API non implementata: è la ragione per cui gli errori documentati in questo progetto sono del tipo *could not load kernel32.dll*, e la cura è installare il runtime giusto nel prefix giusto, non riparare un sistema operativo.

## Il limite

Wine non copre tutto, e va detto invece di scoprirlo. Un programma che richiede un driver in kernel space, una chiave hardware con driver proprietario, o un sistema anti-manomissione che lavora sotto il livello delle API utente, non funziona sotto Wine per costruzione, e in quel caso la macchina virtuale è la risposta corretta. Per i programmi di questo progetto, cioè Akabak, VituixCAD, WinISD ed EASE Focus, il caso non si presenta: sono applicazioni desktop che chiedono .NET Framework[^1], i runtime Visual C++ e i font di sistema, e tutte e quattro rientrano nel dominio in cui Wine funziona bene.

[^1]: *.NET Framework* - insieme di librerie e ambiente di esecuzione Microsoft su cui sono compilate molte applicazioni desktop Windows; sotto Wine si installa nel singolo prefix tramite winetricks, e la versione richiesta dai programmi di questo progetto è la 4.8.
