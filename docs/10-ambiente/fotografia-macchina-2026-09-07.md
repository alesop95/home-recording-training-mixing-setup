<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Fotografia della macchina al 2026-09-07, e la diagnosi corretta

> Esito della fase 0 della procedura di installazione pulita, eseguita sulla macchina reale il 2026-09-07 dopo l'apertura dell'accesso SSH. È il primo documento di questo progetto costruito su dati letti dalla macchina invece che su ricostruzioni, e serve a due scopi opposti: registrare lo stato di ciò che esiste prima di azzerarlo, e correggere ciò che era stato scritto per ipotesi. Tre delle quattro cause che avevo attribuito al blocco di aggiornamento sono smentite dai dati, e la sezione dedicata lo dice per prima cosa.
>
> I ventitré file di output grezzo stanno sotto `_notes/fotografia-2026-09-07/`, ignorata da git, e sulla macchina sotto `~/fotografia-pre-reinstall/`.

## La diagnosi del blocco di aggiornamento era largamente sbagliata

Questa sezione viene prima di tutto perché è la ragione principale per cui la fase 0 esiste: una ipotesi non verificata va messa alla prova, e questa non ha superato la prova.

### Che cosa avevo ipotizzato, e che cosa dicono i dati

| Causa ipotizzata | Esito della verifica |
|---|---|
| Dalla 25.04 non esiste salto diretto alla LTS, serve passare dalla 25.10 | **Smentita.** `do-release-upgrade -c` risponde: *New release '26.04.1 LTS' available*. Il salto diretto è offerto. |
| La direttiva `Prompt=lts` impedisce di trovare il rilascio successivo | **Smentita due volte.** Il file contiene `Prompt=normal`. E il commento dello stesso file dichiara che con `lts` su un rilascio non-LTS l'aggiornatore assume `normal`, quindi anche con quel valore non avrebbe bloccato nulla. |
| Gli archivi della 25.04 sono stati spostati su `old-releases`, quindi `apt update` restituisce 404 | **Smentita.** Le sorgenti puntano a `it.archive.ubuntu.com` con le suite `plucky`, e le richieste HTTP rispondono 200 su archivio, security e mirror italiano. Su `old-releases` la stessa risorsa risponde 404, cioè il rilascio non è ancora stato spostato lì. |
| L'architettura `i386` e i repository di terze parti sono fattori di attrito | **Confermata, e più grave del previsto.** Si veda sotto. |

L'unica affermazione di quel documento che ha resistito è la più semplice: la 25.04 è fuori supporto. Lo dichiara l'aggiornatore stesso, con *Your Ubuntu release is not supported anymore*.

### Qual è la situazione reale

Non c'è alcun blocco tecnico all'aggiornamento. Il percorso verso la 26.04.1 LTS è aperto adesso, in un solo passo, e lo strumento lo propone.

Il dato che riorganizza tutto il quadro è un altro, ed è la cartella dei log: `/var/log/dist-upgrade/` è vuota, con la sola data del pacchetto che l'ha creata. Significa che **`do-release-upgrade` non è mai stato eseguito su questa macchina**. Non esiste un tentativo fallito da diagnosticare.

Il resto della cronologia conferma il quadro. L'ultima operazione registrata da apt è del 13 agosto 2025, e riguarda l'installazione di `winetricks`. Da allora nessun pacchetto è stato installato o aggiornato per via di apt. La simulazione `apt-get -s dist-upgrade` elenca 134 pacchetti da aggiornare, tutti provenienti da `plucky-updates`, e non segnala conflitti: sono tredici mesi di aggiornamenti ordinari mai applicati. Il sistema segnala inoltre `*** System restart required ***`, perché il kernel `6.14.0-37` è installato mentre quello in esecuzione è il `6.14.0-35`.

La conclusione onesta è quindi che la macchina non fa fatica ad aggiornarsi: **non è mai stata aggiornata**. Che cosa significasse concretamente la difficoltà riferita resta una domanda aperta per l'utente, e le ipotesi plausibili sono che l'avviso grafico non sia mai comparso, che un tentativo sia stato fatto da interfaccia grafica senza lasciare traccia nei log di `do-release-upgrade`, o che la difficoltà fosse attesa e non osservata.

### I due problemi reali, che restano

Il primo è il pasticcio dei repository WineHQ. Sotto `/etc/apt/sources.list.d/` convivono **due** file WineHQ attivi, `winehq-noble.sources` e `winehq-plucky.sources`, che puntano allo stesso URI dichiarando suite diverse, cioè quella della 24.04 e quella della 25.04. Entrambe rispondono 200, quindi entrambe forniscono pacchetti con gli stessi nomi e versioni diverse, per amd64 e i386. È esattamente la mescolanza di provenienze che la pagina di troubleshooting indicava come causa di incoerenze difficili da diagnosticare, e qui non è un rischio teorico ma una configurazione presente. Durante un aggiornamento di rilascio è una fonte classica di dipendenze insoddisfacibili.

Il secondo è lo stato dei pacchetti Wine, che è la fotografia del pasticcio già raccontato dal documento sorgente. Convivono `wine 9.0~repack-4build3` dai repository Ubuntu e `wine-stable 3.0.1ubuntu1`, che è una versione del 2018. Il comando `wine --version` riporta 9.0, quindi il secondo è di fatto un residuo, ma resta installato. Sono presenti anche `libwine:i386` e `wine32:i386`, quindi l'architettura secondaria è in uso.

C'è infine una sorgente `file:/cdrom/` ancora attiva, residuo dell'installazione, e due file di backup delle sorgenti, `original.list.bak` e `ubuntu.sources.curtin.orig`.

### La cronologia apt conferma il racconto del documento sorgente

Questo è un riscontro che vale registrare, perché è la prima volta che una affermazione del documento sorgente viene confermata da una fonte indipendente sulla macchina stessa. La sezione di troubleshooting descriveva una sequenza di installazioni, purghe e reinstallazioni di Wine; la cronologia di apt la riporta con data e ora.

```
2025-08-13 12:06  apt install wine64 winetricks
2025-08-13 12:17  apt install wine64 wine32 winetricks
2025-08-13 13:18  apt remove --purge wine* -y
2025-08-13 13:19  apt autoremove -y
2025-08-13 13:31  apt install --install-recommends wine-stable
2025-08-13 16:37  apt install winetricks
```

È la sequenza del documento sorgente, negli stessi termini e nello stesso ordine, compresa la purga a metà pomeriggio dopo il fallimento e la reinstallazione con `--install-recommends`. Il sistema è stato installato il 5 agosto 2025 e l'ambiente Wine costruito il 13.

## Identità del sistema

Il nome della macchina è `i7-6700-16GBDDR4-500GBSSD`, che descrive l'hardware e conferma quanto documentato: processore i7-6700, 16 GB di RAM DDR4, SSD da 500 GB.

L'utente è `alesop95`, con identificativo numerico 1000, e appartiene fra gli altri ai gruppi `sudo`, `audio` e `plugdev`. L'appartenenza al gruppo `audio` è uno dei controlli previsti dalla fase 6 della procedura, e risulta già soddisfatto.

Il sistema è Ubuntu 25.04, nome in codice `plucky`, con kernel `6.14.0-35-generic`.

## Partizionamento reale, e uno scarto dal piano documentato

Lo schema è quello previsto, cioè quattro partizioni con `/home` separata, ma due dimensioni differiscono in modo rilevante da quanto il documento sorgente dichiarava.

| Partizione | Filesystem | Dimensione reale | Usato | Mount point | UUID |
|---|---|---|---|---|---|
| `nvme0n1p1` | vfat FAT32 | 1,1 GB | 6,2 MB, 1 per cento | `/boot/efi` | `9054-9070` |
| `nvme0n1p2` | ext4 | 73 GB | 24 GB, 35 per cento | `/` | `19351fc8-...` |
| `nvme0n1p3` | swap | - | - | `[SWAP]` | `50a4c66e-...` |
| `nvme0n1p4` | ext4 | 369 GB | 3,7 GB, 2 per cento | `/home` | `4aa5afff-...` |

Il primo scarto è la partizione EFI, che il documento sorgente indicava intorno ai 100 MB e che in realtà è di 1,1 GB, con 6,2 MB occupati. Non è un problema, è spazio sovradimensionato e inerte, ma va corretto nella documentazione perché una procedura di reinstallazione scritta su un valore sbagliato porterebbe a cercare una partizione che non corrisponde.

Il secondo scarto è più interessante per la decisione da prendere: `/home` è di 369 GB e ne usa 3,7, cioè il 2 per cento. La separazione di `/home` che rende la reinstallazione a basso rischio esiste ed è quella prevista, ma il volume di dati da proteggere è modesto. Su root ci sono 46 GB liberi su 73.

Il montaggio in `/etc/fstab` avviene per UUID e non per nome di dispositivo, che è la forma corretta e robusta.

## La catena audio, e la risposta a una domanda lasciata aperta

Il documento sull'aggiornamento elencava fra i punti da verificare il modo in cui la 26.04 fornisce il kernel a bassa latenza, e la fase 6 della procedura prescriveva di controllare che il kernel in esecuzione fosse un kernel a bassa latenza. Quella prescrizione era imprecisa, e la fotografia permette di correggerla.

Sulla macchina non è installato alcun `linux-image-lowlatency`: i kernel presenti sono `linux-image-6.14.0-35-generic`, `linux-image-6.14.0-37-generic` e il metapacchetto `linux-image-generic`. È installato invece `ubuntustudio-lowlatency-settings` alla versione `25.04.21`.

La riga di comando del kernel in esecuzione spiega come i due fatti convivano.

```
BOOT_IMAGE=/boot/vmlinuz-6.14.0-35-generic root=UUID=... ro quiet splash preempt=full threadirqs vt.handoff=7
```

I due parametri che contano sono `preempt=full`, che abilita la prelazione completa del kernel, e `threadirqs`, che sposta la gestione degli interrupt in thread schedulabili. Sono precisamente le proprietà per cui esisteva un kernel `lowlatency` separato, ottenute qui sul kernel generico tramite parametri di avvio. È l'approccio che Ubuntu Studio ha adottato, e il pacchetto di impostazioni è ciò che li configura.

Il pacchetto configura anche i limiti di priorità in tempo reale, che sono l'altra metà della catena.

```
@audio    - rtprio 95
@audio    - memlock unlimited
@pipewire - rtprio 95
@pipewire - memlock unlimited
```

I tre servizi audio, cioè `pipewire`, `pipewire-pulse` e `wireplumber`, risultano tutti attivi.

Ne segue la correzione alla fase 6: il controllo corretto non è il nome del kernel ma la presenza di `preempt=full` e `threadirqs` in `/proc/cmdline`, insieme ai limiti `rtprio` per il gruppo `audio`. Un controllo sul nome del kernel su questa macchina darebbe un falso negativo, concludendo che la configurazione a bassa latenza manchi quando invece è attiva.

Va infine registrato che **la Scarlett 2i2 non è collegata**. L'elenco USB non riporta alcun dispositivo Focusrite, e le sole schede viste sono l'audio integrato `ALC887-VD` con le sue uscite HDMI. Il controllo di uscita della fase 6, che chiede di vedere la Scarlett in ingresso e in uscita, non è quindi eseguibile finché l'interfaccia non viene collegata.

## L'ambiente Wine reale

C'è **un solo prefix**, ed è quello di default: `/home/alesop95/.wine`. Non esistono prefix separati per programma.

Questo chiude una delle lacune dichiarate nello storico di Akabak e VACS, cioè in quale prefix i due programmi siano installati: sono nel prefix condiviso, secondo l'approccio che il documento sorgente chiamava fare come per Akabak e di cui riconosceva il rischio. La seconda lacuna, cioè quale variante di VACS sia installata e come fu risolto il suo fallimento iniziale, richiede di guardare dentro il prefix e resta aperta.

La versione di Wine è `wine-9.0 (Ubuntu 9.0~repack-4build3)`, cioè quella dei repository Ubuntu e non di WineHQ, nonostante entrambi i repository WineHQ siano configurati. Il binario è `/usr/bin/wine`; non esistono `wine64` né `wine32` come comandi separati. I pacchetti installati sono `wine`, `wine-stable`, `wine32:i386`, `libwine`, `libwine:i386`, `fonts-wine` e `winetricks`.

## Che cosa questa fotografia cambia nelle decisioni

Due decisioni vanno riviste, e vale distinguere quanto.

La decisione ADR-006, cioè l'installazione pulita invece dell'aggiornamento in posto, poggiava su quattro motivi. Il primo era che l'installazione fosse più corta e più prevedibile di due aggiornamenti in cascata attraverso archivi storici: **quel motivo è venuto meno**, perché di aggiornamenti in cascata non ce n'è bisogno e di archivi storici non se ne attraversa nessuno. Il percorso alternativo è un solo `do-release-upgrade` verso la 26.04.1 LTS, offerto adesso.

Gli altri tre motivi restano validi e non sono stati toccati dalla verifica. L'ambiente pulito elimina la sedimentazione, che la fotografia ha mostrato essere reale e concreta: due repository WineHQ contemporaneamente attivi, una versione di `wine-stable` del 2018 accanto a una del 2024, l'architettura `i386` dichiarata, una sorgente `file:/cdrom/` residua e un prefix unico condiviso fra programmi. La licenza di Akabak non è a rischio in nessuno dei due scenari. E la 26.04 LTS porta su una base supportata fino al 2031.

Si aggiunge un argomento nuovo che la fotografia rende disponibile e che prima non c'era: `/home` contiene 3,7 GB, quindi il costo del salvataggio dei dati è trascurabile e il rischio dell'operazione è più basso di quanto si potesse stimare senza il dato.

La decisione resta quindi difendibile, ma su tre motivi invece di quattro, e con un'alternativa che si è rivelata molto meno onerosa di come era stata descritta. È una decisione che va riconfermata dall'utente sapendo questo, non data per acquisita: la registrazione della revisione è ADR-011.

## Che cosa resta da fare in fase 0

Tre voci, di cui due richiedono privilegi che l'accesso via chiave non concede in modo non interattivo, perché `sudo` su questa macchina chiede la password.

Lo stato di salute dell'SSD con `smartctl`, che è il controllo il cui esito potrebbe cambiare la decisione da installazione a sostituzione del disco. Richiede `sudo` e, probabilmente, l'installazione di `smartmontools`.

L'esito reale di `sudo apt update`, che le prove HTTP rendono prevedibile ma non certo.

La verifica del Machine Identifier di Akabak, che è un controllo manuale in interfaccia grafica e va fatto davanti alla macchina, con la cattura di uno screenshot della finestra del release code.
