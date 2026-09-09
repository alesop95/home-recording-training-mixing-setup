<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Ambiente Linux: Ubuntu Studio e Wine

> Blocco documentale condiviso. Descrive la macchina di lavoro e lo strato di compatibilità Windows che ci gira sopra. Questo blocco esiste in due copie identiche, una in questo progetto e una nel progetto `home-recording-training-mixing-setup`, perché la stessa macchina serve sia alla progettazione elettroacustica sia all'home recording. La copia canonica è quella di `diy-2way-monitors-home`; la sincronizzazione è meccanica e si esegue con `tools/sync-ambiente.py`.

## La macchina

L'hardware è un vecchio desktop riconvertito: processore Intel i7-6700 a 3.40 GHz, 16 GB di RAM DDR4 e un SSD[^1] Crucial CT500P2SSD8 da 500 GB con firmware P2CR033, al 91 per cento di vita residua secondo SMART letto il 2026-09-07. È una macchina che Windows 11 avrebbe escluso per requisiti, e questo è il motivo per cui è stata riformattata su Linux invece di essere dismessa.

La scheda audio è esterna, una Focusrite Scarlett 2i2 di seconda generazione, con alimentazione phantom a 48 V disponibile. Questa scelta ha una conseguenza diretta sulla catena di misura, discussa nella pagina della fase di misura: rende possibile un microfono XLR da misura, e quindi rende non obbligatorio un microfono USB come l'UMIK-1.

La distribuzione è Ubuntu Studio, non Ubuntu con i pacchetti audio aggiunti a mano. La differenza sta nella configurazione a bassa latenza predisposta e nella selezione di pacchetti già montata per registrazione, mixing, mastering e live processing. Sulla macchina quella configurazione non arriva da un kernel dedicato ma dal kernel generico con i parametri di avvio `preempt=full` e `threadirqs`, più i limiti di priorità in tempo reale per i gruppi `audio` e `pipewire`: il dettaglio è nella fotografia, e il controllo sul nome del kernel darebbe un falso negativo. Sulla 26.04 quei parametri arrivano da `/etc/default/grub.d/ubuntustudio.cfg` e non da `/etc/default/grub`, e i limiti sono concessi ai due gruppi ma non arrivano all'utente finché non gli si aggiungono, che è il difetto corretto in MS-077.

La versione installata è la *26.04.1 LTS*[^2], con kernel `7.0.0-31-generic`, installata l'8 settembre 2026 conservando `/home`. Fino a quella data la macchina portava la 25.04, un rilascio intermedio uscito dal supporto e mai aggiornato, con 134 pacchetti pendenti e l'ultimo intervento di `apt` datato 13 agosto 2025: quello stato è fotografato in [fotografia-macchina-2026-09-07.md](fotografia-macchina-2026-09-07.md), che resta valido come descrizione del punto di partenza e non del presente. La cronologia della sostituzione, con i comandi e le ragioni, è in [setup-macchina-2026-09.md](setup-macchina-2026-09.md).

## Indice del blocco

L'installazione del sistema, i requisiti e lo schema di partizionamento adottato stanno in [ubuntu-studio-installazione.md](ubuntu-studio-installazione.md).

La diagnosi dello stato di aggiornamento del sistema, con i dati letti dalla macchina e le due strade possibili, sta nella fotografia. La pagina che conteneva la ricostruzione fatta per ipotesi è stata rimossa perché tre delle sue quattro cause erano false: il record di quell'errore, con la ragione di ciascuna smentita, è in MS-029 del registro dei microstep.

La fotografia della macchina reale al 2026-09-07, con i dati letti sulla macchina e la diagnosi corretta che smentisce tre delle quattro cause ipotizzate per il blocco di aggiornamento, sta in [fotografia-macchina-2026-09-07.md](fotografia-macchina-2026-09-07.md). È il primo documento del blocco costruito su misure invece che su ricostruzioni, e va letto prima della pagina sulla diagnosi.

La procedura operativa completa di installazione pulita della 26.04 LTS, in undici fasi, con la fotografia dello stato attuale, i controlli di uscita di ogni fase e il piano di rientro, sta in [installazione-pulita-26-04.md](installazione-pulita-26-04.md). Include la configurazione della sospensione automatica e dell'accesso SSH, che sono i due motivi per cui la macchina risultava irraggiungibile.

Il racconto continuo di come la macchina è stata azzerata e ricostruita fra il 4 e il 9 settembre 2026, con i comandi eseguiti, gli orari ricavati dai log dell'installatore e la ragione di ogni passo, sta in [setup-macchina-2026-09.md](setup-macchina-2026-09.md). È il documento da leggere per capire perché la configurazione è quella che è, e non come si esegue la procedura: la sequenza nel tempo e le decisioni che l'hanno prodotta non stanno né nella procedura, che è prescrittiva, né nel registro dei microstep, che è per intervento.

La riduzione di quella procedura a ciò che serve avere sotto gli occhi davanti alla macchina, cioè la scheda che si stampa e si porta accanto al computer da azzerare, sta in [scheda-reinstallazione.md](scheda-reinstallazione.md). Non sostituisce la procedura e non ne è un riassunto fedele: è una selezione fatta per un lettore che non ha la postazione di sviluppo davanti, quindi dove le due divergono ha ragione la procedura. Il `.docx` stampabile si genera da lì con `python tools/make-scheda-docx.py`.

La differenza concettuale fra Wine, un emulatore e una macchina virtuale, che è il punto da cui dipende tutto il resto della configurazione, sta in [wine-vs-emulatore.md](wine-vs-emulatore.md).

Il modello dei prefix, la scelta fra 32 e 64 bit e il comportamento delle licenze legate alla macchina stanno in [wine-prefix-e-dipendenze.md](wine-prefix-e-dipendenze.md).

La procedura di pulizia, installazione e configurazione di Wine sta in [wine-configurazione.md](wine-configurazione.md).

L'installazione dei singoli programmi Windows, cioè Akabak, VituixCAD, WinISD ed EASE Focus, sta in [wine-programmi-windows.md](wine-programmi-windows.md).

L'inventario verificato del corredo software `Progetto stanza`, con il formato reale di ogni installer, il prefix di destinazione, le dipendenze e lo stato di licenza accertato di ciascuna voce, sta in [wine-corredo-progetto-stanza.md](wine-corredo-progetto-stanza.md). È anche la pagina che spiega perché l'architettura di un installer non è quella dell'applicazione che installa.

I guasti già incontrati e la loro risoluzione stanno in [wine-troubleshooting.md](wine-troubleshooting.md).

[^1]: *SSD*, Solid State Drive - unità di memorizzazione a stato solido, senza parti in movimento, con tempi di accesso molto inferiori a quelli di un disco meccanico.

[^2]: *LTS*, Long Term Support - designazione delle versioni di Ubuntu con supporto esteso, rilasciate ogni due anni ad aprile, contrapposte alle versioni intermedie a supporto breve.
