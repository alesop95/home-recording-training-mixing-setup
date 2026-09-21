<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Ambiente Linux: Ubuntu Studio e Wine

> Blocco documentale condiviso. Descrive la macchina di lavoro e lo strato di compatibilità Windows che ci gira sopra. Questo blocco esiste in due copie identiche, una in questo progetto e una nel progetto `home-recording-training-mixing-setup`, perché la stessa macchina serve sia alla progettazione elettroacustica sia all'home recording. La copia canonica è quella di `diy-2way-monitors-home`; la sincronizzazione è meccanica e si esegue con `tools/sync-ambiente.py`.

## La macchina

L'hardware è un vecchio desktop riconvertito: processore Intel i7-6700 a 3.40 GHz, 16 GB di RAM DDR4 e un SSD[^1] Crucial CT500P2SSD8 da 500 GB con firmware P2CR033, al 91 per cento di vita residua secondo SMART letto il 2026-09-07. È una macchina che Windows 11 avrebbe escluso per requisiti, e questo è il motivo per cui è stata riformattata su Linux invece di essere dismessa.

La scheda audio della macchina è quella integrata, `ALC887-VD`, osservata il 2026-09-09 come unica presente. Nessuna interfaccia audio esterna è collegata, e quale interfaccia servirà alla catena di misura è una decisione aperta e non un dato di fatto.

Su questo punto va registrato un ritiro, perché le versioni precedenti di questa pagina affermavano il contrario. Dichiaravano che la scheda audio fosse esterna, una Focusrite Scarlett 2i2 di seconda generazione con alimentazione phantom a 48 V disponibile, e ne traevano la conseguenza che un microfono XLR da misura fosse possibile e un microfono USB come l'UMIK-1 non obbligatorio. La fonte di quella affermazione era un file alla radice del repository che conteneva un solo indirizzo, la pagina di download dei driver Focusrite: un segnalibro promosso a inventario. L'utente ha confermato il 2026-09-09 di possedere quella interfaccia ma di non impiegarla in questo progetto, quindi il fatto hardware esiste e la sua appartenenza alla catena di questo progetto no. Il requisito resta, cioè un ingresso microfonico con alimentazione phantom per un microfono XLR da misura, e la sua soddisfazione è aperta. Il record dell'errore è in MS-079.

La distribuzione è Ubuntu Studio, non Ubuntu con i pacchetti audio aggiunti a mano. La differenza sta nella configurazione a bassa latenza predisposta e nella selezione di pacchetti già montata per registrazione, mixing, mastering e live processing. Sulla macchina quella configurazione non arriva da un kernel dedicato ma dal kernel generico con i parametri di avvio `preempt=full` e `threadirqs`, più i limiti di priorità in tempo reale per i gruppi `audio` e `pipewire`: il dettaglio è nella fotografia, e il controllo sul nome del kernel darebbe un falso negativo. Sulla 26.04 quei parametri arrivano da `/etc/default/grub.d/ubuntustudio.cfg` e non da `/etc/default/grub`, e i limiti sono concessi ai due gruppi ma non arrivano all'utente finché non gli si aggiungono, che è il difetto corretto in MS-077.

La versione installata è la *26.04.1 LTS*[^2], con kernel `7.0.0-31-generic`, installata l'8 settembre 2026 conservando `/home`. Fino a quella data la macchina portava la 25.04, un rilascio intermedio uscito dal supporto e mai aggiornato, con 134 pacchetti pendenti e l'ultimo intervento di `apt` datato 13 agosto 2025: quello stato è fotografato in [fotografia-macchina-2026-09-07.md](fotografia-macchina-2026-09-07.md), che resta valido come descrizione del punto di partenza e non del presente. La cronologia della sostituzione, con i comandi e le ragioni, è in [setup-macchina-2026-09.md](setup-macchina-2026-09.md).

## Indice del blocco

L'installazione del sistema, i requisiti e lo schema di partizionamento adottato stanno in [ubuntu-studio-installazione.md](ubuntu-studio-installazione.md).

La diagnosi dello stato di aggiornamento del sistema, con i dati letti dalla macchina e le due strade possibili, sta nella fotografia. La pagina che conteneva la ricostruzione fatta per ipotesi è stata rimossa perché tre delle sue quattro cause erano false: il record di quell'errore, con la ragione di ciascuna smentita, è in MS-029 del registro dei microstep.

La fotografia della macchina reale al 2026-09-07, con i dati letti sulla macchina e la diagnosi corretta che smentisce tre delle quattro cause ipotizzate per il blocco di aggiornamento, sta in [fotografia-macchina-2026-09-07.md](fotografia-macchina-2026-09-07.md). È il primo documento del blocco costruito su misure invece che su ricostruzioni, e va letto prima della pagina sulla diagnosi.

La procedura operativa completa di installazione pulita della 26.04 LTS, in undici fasi, con la fotografia dello stato attuale, i controlli di uscita di ogni fase e il piano di rientro, sta in [installazione-pulita-26-04.md](installazione-pulita-26-04.md). Include la configurazione della sospensione automatica e dell'accesso SSH, che sono i due motivi per cui la macchina risultava irraggiungibile.

La fotografia finale della macchina al 2026-09-21, con il confronto voce per voce contro quella di partenza, la sequenza replicabile che la produce e le voci di troubleshooting nate eseguendola, sta in [fotografia-macchina-post-reinstall.md](fotografia-macchina-post-reinstall.md). È il controllo di uscita della procedura di installazione pulita, e va letta accanto alla fotografia del 2026-09-07 perché il suo contenuto è una differenza e non uno stato: da sola direbbe com'è la macchina, insieme all'altra dice che cosa il lavoro ha prodotto e quali affermazioni sono ora verificate invece che credute. I tre scarti non attesi che ha trovato sono il residuo del repository WineHQ, il limite di osservabilità della catena audio da una sessione remota e un comando di censimento che misurava meno di quanto dichiarasse.

Il racconto continuo di come la macchina è stata azzerata e ricostruita fra il 4 e il 9 settembre 2026, con i comandi eseguiti, gli orari ricavati dai log dell'installatore e la ragione di ogni passo, sta in [setup-macchina-2026-09.md](setup-macchina-2026-09.md). È il documento da leggere per capire perché la configurazione è quella che è, e non come si esegue la procedura: la sequenza nel tempo e le decisioni che l'hanno prodotta non stanno né nella procedura, che è prescrittiva, né nel registro dei microstep, che è per intervento.

La riduzione di quella procedura a ciò che serve avere sotto gli occhi davanti alla macchina, cioè la scheda che si stampa e si porta accanto al computer da azzerare, sta in [scheda-reinstallazione.md](scheda-reinstallazione.md). Non sostituisce la procedura e non ne è un riassunto fedele: è una selezione fatta per un lettore che non ha la postazione di sviluppo davanti, quindi dove le due divergono ha ragione la procedura. Il `.docx` stampabile si genera da lì con `python tools/make-scheda-docx.py`.

La differenza concettuale fra Wine, un emulatore e una macchina virtuale, che è il punto da cui dipende tutto il resto della configurazione, sta in [wine-vs-emulatore.md](wine-vs-emulatore.md).

La forma dell'ambiente che ne risulta, cioè quali strati lo compongono, che cosa sia davvero un prefix, in quali due modi Wine esegue un programma a 32 bit e dove vive ogni pezzo sul disco, sta in [architettura-ambiente-wine.md](architettura-ambiente-wine.md). È la pagina da leggere per capire perché l'ambiente funziona, con i diagrammi dei quattro strati e dei due modi di esecuzione, la mappa dei percorsi e i quattro comandi con cui si verifica che l'architettura sia davvero quella descritta. Non è una procedura e non sostituisce quelle che seguono.

Il modello dei prefix, la scelta fra 32 e 64 bit e il comportamento delle licenze legate alla macchina stanno in [wine-prefix-e-dipendenze.md](wine-prefix-e-dipendenze.md).

La procedura di pulizia, installazione e configurazione di Wine sta in [wine-configurazione.md](wine-configurazione.md).

L'installazione dei singoli programmi Windows, cioè Akabak, VituixCAD, WinISD ed EASE Focus, sta in [wine-programmi-windows.md](wine-programmi-windows.md).

L'inventario verificato del corredo software `Progetto stanza`, con il formato reale di ogni installer, il prefix di destinazione, le dipendenze e lo stato di licenza accertato di ciascuna voce, sta in [wine-corredo-progetto-stanza.md](wine-corredo-progetto-stanza.md). È anche la pagina che spiega perché l'architettura di un installer non è quella dell'applicazione che installa.

I guasti già incontrati e la loro risoluzione stanno in [wine-troubleshooting.md](wine-troubleshooting.md).

L'allestimento di Veeam Agent for Linux, con la scelta fra le due varianti che decide che tipo di backup la macchina potrà fare, le verifiche preliminari, le trappole incontrate e le alternative quando il backup a livello di volume non è disponibile, sta in [veeam-agent-linux.md](veeam-agent-linux.md). È una pagina prescrittiva, estratta dai microstep da MS-117 a MS-122 perché rifare la cosa non richieda di rileggerne il racconto, ed è scritta per essere riusabile anche fuori da questo progetto.

La verifica della catena audio, cioè come si stabilisce che la macchina sia capace di riprodurre suono in modo utilizzabile per un lavoro di elettroacustica, sta in [catena-audio-pipewire.md](catena-audio-pipewire.md). È una pagina prescrittiva in otto fasi, estratta dai microstep MS-146, MS-154, MS-155 e MS-156, e la sua parte che vale oltre questa macchina sono i tre assi che decidono se una misura audio abbia valore, cioè quale sessione tenga il posto, quanto sia vecchio il processo di gestione audio rispetto alle modifiche ai gruppi, e se i jack siano occupati. Tutte e tre hanno già prodotto qui una misura vacua, cioè un esito che non informava né in positivo né in negativo mentre sembrava farlo.

[^1]: *SSD*, Solid State Drive - unità di memorizzazione a stato solido, senza parti in movimento, con tempi di accesso molto inferiori a quelli di un disco meccanico.

[^2]: *LTS*, Long Term Support - designazione delle versioni di Ubuntu con supporto esteso, rilasciate ogni due anni ad aprile, contrapposte alle versioni intermedie a supporto breve.
