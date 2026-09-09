<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Installazione pulita di Ubuntu Studio 26.04 LTS, passo per passo

> Procedura operativa completa. Ogni passo dichiara che cosa si fa, perché si fa, il comando o l'azione esatta, e come si verifica che sia riuscito. La procedura è divisa in undici fasi e va eseguita nell'ordine: le fasi 0 e 1 raccolgono e mettono in salvo informazioni che dopo la fase 4 non sarebbero più recuperabili, quindi saltarle non è una scorciatoia ma una perdita di dati.
>
> Stato al 2026-09-08. La fase 0 è chiusa nella sostanza: la parte non privilegiata è eseguita, e delle tre voci privilegiate le due che potevano spostare una decisione sono state eseguite dall'utente, cioè lo stato di salute del disco e la verifica del Machine Identifier. La fase 1 è compiuta e verificata. Della fase 2 sono compiute la 2.1 e la 2.2, cioè immagine scaricata e verificata per somma e per firma; resta la 2.3. Le fasi da 3 in avanti non sono ancora state eseguite. Ogni fase viene spuntata nel registro dei microstep man mano che si compie, con l'esito reale accanto a quello atteso.
>
> Questa procedura è già stata corretta quattro volte dall'esito reale delle sue prime fasi, e vale saperlo prima di eseguirla: la premessa e il controllo del kernel della fase 6 dalla fase 0, le fasi 7 e 8 da ADR-016 sull'architettura a 32 bit, e la fase 2 dalla fonte ufficiale dell'immagine, che pubblica un point release che questa pagina non nominava. Chi eseguisse una versione precedente otterrebbe un ambiente in cui Akabak non parte.

## Premessa: perché una installazione e non un aggiornamento

La macchina è su Ubuntu Studio 25.04, una versione intermedia fuori supporto. La premessa originaria di questa procedura diceva anche che la LTS successiva non fosse raggiungibile con un salto singolo, e *quella parte è stata smentita* dalla fase 0: `do-release-upgrade` propone direttamente la 26.04.1 LTS. Il quadro reale è in [fotografia-macchina-2026-09-07.md](fotografia-macchina-2026-09-07.md).

Dei quattro motivi per cui questa strada era stata preferita all'aggiornamento in posto, registrati come ADR-006, il primo è caduto con quella smentita: non ci sono due aggiornamenti in cascata attraverso archivi storici da evitare, perché l'alternativa è un solo `do-release-upgrade` dopo aver applicato i 134 pacchetti pendenti. I tre restanti tengono, ed è ADR-011 a registrare la revisione. Il motivo dominante diventa quindi l'ambiente pulito, che la fotografia ha mostrato essere necessario in modo concreto: due repository WineHQ attivi per due rilasci diversi di Ubuntu, `wine-stable 3.0.1` del 2018 accanto a `wine 9.0`, l'architettura `i386` dichiarata, una sorgente `file:/cdrom/` residua e un prefix unico condiviso fra programmi. Restano validi anche il fatto che la licenza di Akabak non sia a rischio e che la 26.04 porti su una base supportata per cinque anni.

Si aggiunge un motivo che prima non c'era, perché richiedeva il dato: `/home` contiene 3,7 GB su 369 disponibili, quindi il costo del salvataggio è trascurabile e il rischio dell'operazione più basso di quanto si potesse stimare.

Poiché una delle quattro gambe era venuta meno, la decisione andava riconfermata dall'utente su questa base e non data per acquisita. *È stata riconfermata il 2026-09-07*, sui tre motivi che restano e con l'ambiente pulito come dominante, e la riconferma è registrata come ADR-013, che chiude PA-006.

Il fatto che rende l'operazione a basso rischio è il partizionamento scelto all'installazione originaria, con `/home` su una partizione separata. È la decisione che oggi paga il dividendo più alto, e va trattata con rispetto: l'unico modo di rovinare questa procedura è formattare `/home` per distrazione nella fase 4.

## Le regole non negoziabili

Tre, e valgono per tutta la procedura.

Non si formatta `/home`. In fase 4 la partizione `/home` va montata senza spuntare l'opzione di formattazione. È l'unico passo irreversibile dell'intera procedura.

Non si parte senza aver completato la fase 1. Il trasferimento dei materiali e la copia di sicurezza vanno chiusi e verificati prima di toccare il disco, non dopo.

Non si dà per verificato ciò che non si è letto. Ogni fase ha un controllo di uscita: se il controllo non dà l'esito atteso, si ferma e si capisce, non si prosegue sperando.

## Fase 0: fotografia completa della macchina attuale

> *Eseguita il 2026-09-07* nella sua parte non privilegiata. L'esito, con i dati reali e le tre ipotesi diagnostiche smentite, è in [fotografia-macchina-2026-09-07.md](fotografia-macchina-2026-09-07.md). Restano da fare le tre voci che richiedono `sudo` o l'interfaccia grafica, elencate in fondo a quel documento.

Obiettivo: registrare lo stato di ciò che esiste, per tre ragioni distinte. Confermare la diagnosi del blocco di aggiornamento. Raccogliere le informazioni che dopo la reinstallazione servono a ricostruire l'ambiente identico. E poter dimostrare, a lavoro finito, che cosa è cambiato e che cosa no.

Tutti i comandi di questa fase sono in sola lettura e non modificano nulla. Conviene raccogliere l'output in un file e portarlo via dalla macchina, perché è la fotografia che sopravvive alla formattazione.

### 0.1 Identità del sistema e conferma della diagnosi

```bash
lsb_release -a
cat /etc/os-release
uname -a
cat /etc/update-manager/release-upgrades
cat /etc/apt/sources.list.d/ubuntu.sources
ls -la /etc/apt/sources.list.d/
dpkg --print-foreign-architectures
do-release-upgrade -c
```

Esito reale, misurato il 2026-09-07: la 25.04 con nome in codice `plucky`, confermata; la direttiva `Prompt=normal` e *non* `lts`; sorgenti che puntano a `it.archive.ubuntu.com` sulle suite `plucky` e ancora vive, con risposta HTTP 200; l'architettura `i386` presente, confermata; e l'ultimo comando che risponde *New release '26.04.1 LTS' available*, cioè offre il salto diretto. Il confronto riga per riga fra questo esito e quello che era stato previsto è nella tabella di apertura di [fotografia-macchina-2026-09-07.md](fotografia-macchina-2026-09-07.md).

Su una macchina diversa, o su questa dopo un tempo lungo, l'esito può cambiare: la regola resta leggere i messaggi reali e non assumerli, che è precisamente la lezione di questa fase.

### 0.2 Disco, partizioni e spazio

```bash
lsblk -f
sudo fdisk -l
df -h
sudo blkid
cat /etc/fstab
```

Serve a registrare la mappa esatta del disco, con gli identificativi univoci delle partizioni, e a sapere quanto spazio occupa `/home` prima di decidere dove metterne una copia. Il contenuto di `/etc/fstab` è la fotografia di come le partizioni sono montate oggi, e in fase 5 permette di confrontare il risultato con il punto di partenza.

### 0.3 Stato di salute del disco

```bash
sudo apt list --installed 2>/dev/null | grep smartmontools
sudo smartctl -a /dev/nvme0n1
```

L'SSD era dato al 91 per cento di vita residua da una scansione fatta su Windows nel periodo dell'installazione originaria. Vale rileggerlo ora, prima di scriverci sopra un sistema nuovo: se il valore è crollato, la decisione da prendere non è più fra installazione e aggiornamento ma fra installazione e sostituzione del disco.

La verifica del 2026-09-07 ha accertato che `smartmontools` è *già installato*, alla versione `smartctl 7.4`, quindi l'avvertenza precedente su una possibile installazione da rete non serve. Ha accertato anche che non esiste una via non privilegiata per leggere i dati SMART, perché `/dev/nvme0` è `crw------- root root` e l'utente non appartiene al gruppo `disk`. Tre dati si ricavano comunque senza privilegi, e sono il modello reale del disco, il firmware e la temperatura del controller.

```bash
cat /sys/class/nvme/nvme0/model
cat /sys/class/nvme/nvme0/firmware_rev
cat /sys/class/nvme/nvme0/hwmon1/temp1_input
```

Il comando privilegiato va quindi eseguito da un terminale interattivo, e richiede l'opzione `-t` se lo si lancia via SSH, perché senza un terminale allocato `sudo` non ha dove chiedere la password. Il dettaglio, con le tre strade possibili e il costo di ciascuna, è in `fotografia-macchina-2026-09-07.md`.

### 0.4 Catena audio attuale

```bash
aplay -l
arecord -l
pactl info
systemctl --user status pipewire pipewire-pulse wireplumber
jack_control status
cat /etc/security/limits.d/*audio* 2>/dev/null
uname -r
```

Registra quale server audio è effettivamente in uso, quali dispositivi audio sono visti e con quale nome, e quali limiti di priorità in tempo reale sono configurati per i gruppi `audio` e `pipewire`. È la parte della fotografia che serve al progetto gemello di home recording tanto quanto a questo.

### 0.5 Prefix Wine e programmi installati

Questa è la parte che colma le lacune dichiarate in `docs/90-riferimenti/timeline-akabak-vacs.md`, cioè in quale prefix stanno effettivamente i programmi e quale variante di VACS è installata.

```bash
wine --version
which wine wine64 wine32 winetricks
ls -la ~/ | grep -i wine
ls -la ~/wineprefixes/ 2>/dev/null
find ~ -maxdepth 2 -name "system.reg" -printf "%h\n" 2>/dev/null
ls -la ~/.wine/drive_c/ 2>/dev/null
ls -la "~/.wine/drive_c/Program Files" 2>/dev/null
dpkg -l | grep -i wine
```

Il comando con `find` è il modo affidabile di elencare i prefix esistenti, perché un prefix è riconoscibile dalla presenza del file di registro `system.reg` nella sua radice, indipendentemente da come è stato chiamato.

### 0.6 Verifica del Machine Identifier di Akabak, prima di azzerare

È un controllo manuale, non un comando, e va fatto adesso perché dopo la formattazione non sarebbe più possibile confrontare.

Si apre Akabak nel prefix in cui è installato, indicandolo esplicitamente, si va nel menu di aiuto alla voce del release code, e si legge il Machine Identifier mostrato. Va confrontato con quello registrato nella scheda riservata sotto `_notes/`, a cui il Release Code permanente è legato.

Se coincide, il codice esistente resterà valido dopo la reinstallazione e non serve fare altro. Se non coincide, qualcosa nell'hardware è cambiato dall'agosto 2025, e va ricontattato l'autore prima di procedere, non dopo.

Questa è anche l'occasione per catturare uno screenshot di quella finestra: è l'unica prova visiva dello stato di attivazione prima dell'azzeramento, e serve al confronto in fase 8.

### 0.7 Elenco dei pacchetti installati e delle configurazioni personali

```bash
dpkg --get-selections > ~/fotografia-pacchetti.txt
apt-mark showmanual > ~/fotografia-pacchetti-manuali.txt
ls -la ~/.config/
crontab -l
systemctl list-unit-files --state=enabled --type=service
```

Il secondo comando è il più utile dei cinque, perché elenca solo i pacchetti installati deliberatamente e non le loro dipendenze: è la lista da cui si ricostruisce l'ambiente sul sistema nuovo, invece di reinstallare a memoria.

### 0.8 Raccolta della fotografia in un unico file

```bash
mkdir -p ~/fotografia-pre-reinstall
cd ~/fotografia-pre-reinstall
lsb_release -a > 01-sistema.txt 2>&1
cat /etc/update-manager/release-upgrades > 02-release-upgrades.txt 2>&1
cat /etc/apt/sources.list.d/ubuntu.sources > 03-sorgenti-apt.txt 2>&1
lsblk -f > 04-dischi.txt 2>&1
sudo blkid > 05-blkid.txt 2>&1
cat /etc/fstab > 06-fstab.txt 2>&1
aplay -l > 07-audio-out.txt 2>&1
arecord -l > 08-audio-in.txt 2>&1
pactl info > 09-pactl.txt 2>&1
dpkg -l > 10-pacchetti-tutti.txt 2>&1
apt-mark showmanual > 11-pacchetti-manuali.txt 2>&1
wine --version > 12-wine.txt 2>&1
find ~ -maxdepth 2 -name "system.reg" -printf "%h\n" > 13-prefix-wine.txt 2>&1
uname -a > 14-kernel.txt 2>&1
```

Controllo di uscita della fase 0: la cartella contiene quattordici file, nessuno vuoto, e la cartella è stata copiata fuori dalla macchina. Il posto naturale dove copiarla è la stessa cartella del progetto sulla postazione di sviluppo, sotto `_notes/`, che è ignorata da git.

## Fase 1: trasferimento dei materiali e copia di sicurezza

Obiettivo: portare sulla macchina, sotto `/home`, tutto ciò che servirà a ricostruire, e portare fuori dalla macchina tutto ciò che non deve andare perduto.

### 1.1 Trasferimento dei materiali dalla postazione di sviluppo

Va eseguito ora e non dopo la reinstallazione, perché la destinazione è sotto `/home` e `/home` non viene formattata: i file arrivano prima e sopravvivono all'operazione.

```bash
bash tools/transfer-to-studio.sh --verifica
bash tools/transfer-to-studio.sh
```

Il primo comando esegue i soli controlli preliminari. Il secondo trasferisce gli otto file del manifest, per circa 158 MiB, e ricalcola le impronte SHA-256 sulla destinazione confrontandole con quelle di origine. Il dettaglio di che cosa viene trasferito e dove sta in `docs/TRANSFER-MANIFEST.md`.

Controllo di uscita: lo strumento dichiara che tutte le impronte coincidono.

### 1.2 Copia di sicurezza dei prefix Wine

Non serve a ricostruire, perché i prefix vanno rifatti puliti, ma serve come rete: se dopo la reinstallazione un programma non si comporta come prima, avere il prefix vecchio permette di confrontare configurazioni e librerie invece di indovinare.

```bash
cd ~
tar czf ~/backup-wineprefixes.tar.gz .wine wineprefixes 2>/dev/null
ls -lh ~/backup-wineprefixes.tar.gz
```

Il file resta sotto `/home` e quindi sopravvive. Se `/home` avesse poco spazio, va copiato fuori.

### 1.3 Copia di sicurezza di `/home` fuori dalla macchina

Questa è la vera assicurazione, e va fatta anche se la procedura non prevede di formattare `/home`, perché il rischio da coprire è precisamente l'errore umano nella fase 4.

La destinazione dipende da che cosa si ha a disposizione: un disco esterno è la scelta naturale. Il comando qui sotto copia verso un disco montato, preservando permessi e collegamenti.

```bash
sudo rsync -aHAX --info=progress2 /home/ /media/alesop95/DISCO-ESTERNO/backup-home/
```

Controllo di uscita della fase 1: la copia esiste, la sua dimensione è coerente con quella di `/home` letta in fase 0.2, e i materiali del manifest sono su `/home` con le impronte verificate. Solo a questo punto si può toccare il disco.

## Fase 2: preparazione del supporto di installazione

### 2.1 Scaricare l'immagine

La regola di questa fase è non dedurre il nome dell'immagine dal calendario dei rilasci ma prenderlo dalla fonte, e la sua ragione si è vista subito: la verifica del 2026-09-08 sull'archivio ufficiale ha mostrato che nella cartella del rilascio convivono *due* immagini, la 26.04 iniziale e la *26.04.1*, cioè il primo point release. È quest'ultima quella da prendere, perché incorpora le correzioni accumulate dopo il rilascio, fra cui quelle dell'installatore e del kernel, ed è anche la versione che `do-release-upgrade` offriva sulla macchina.

Il nome esatto è `ubuntustudio-26.04.1-desktop-amd64.iso`, pesa 6,6 GB, e sta sotto `https://cdimage.ubuntu.com/ubuntustudio/releases/26.04/release/`. Nella stessa cartella stanno il file delle somme di controllo e la sua firma, che si scaricano insieme all'immagine e non dopo, perché servono a decidere se l'immagine è buona prima di scriverla.

```bash
curl -O "https://cdimage.ubuntu.com/ubuntustudio/releases/26.04/release/SHA256SUMS" -O "https://cdimage.ubuntu.com/ubuntustudio/releases/26.04/release/SHA256SUMS.gpg"
curl -L -C - --retry 5 -O "https://cdimage.ubuntu.com/ubuntustudio/releases/26.04/release/ubuntustudio-26.04.1-desktop-amd64.iso"
```

L'opzione `-C -` merita una riga, perché su un file da 6,6 GB fa la differenza fra un intoppo e un lavoro da rifare: dice a `curl` di riprendere da dove si era interrotto invece di ricominciare, e va abbinata a `--retry` perché una interruzione di rete non richieda un intervento manuale.

### 2.2 Verificare l'immagine prima di scriverla

Questo passo si salta spesso e non va saltato: una immagine corrotta produce una installazione che sembra riuscita e fallisce settimane dopo in modo inspiegabile.

Le verifiche sono due e rispondono a domande diverse, il che è la ragione per cui non ne basta una. La prima chiede se il file scaricato è integro, cioè se coincide con quello che l'archivio dichiara. La seconda chiede se quella dichiarazione viene davvero da chi dice di essere, cioè se il file delle somme non è stato sostituito insieme all'immagine. Una somma di controllo confrontata con un file scaricato dallo stesso posto dell'immagine, da sola, non protegge da chi controlli quel posto.

```bash
sha256sum -c SHA256SUMS 2>&1 | grep -i ubuntustudio
gpg --keyid-format long --verify SHA256SUMS.gpg SHA256SUMS
```

Controllo di uscita del primo comando: la riga corrispondente all'immagine scaricata dice `OK`. Le righe delle immagini non scaricate dicono che il file non esiste, ed è normale, dato che il file delle somme elenca entrambe le immagini del rilascio. La somma attesa per il point release, registrata qui perché un valore verificato vale più di una istruzione da seguire, è `2b25d06203c8a2f60da23e20e3afd1fa400f8ff8104fd074c1b7e49fc3768084`.

Controllo di uscita del secondo comando: la firma risulta buona e appartiene alla chiave di firma delle immagini di Ubuntu. Alla prima esecuzione su una macchina nuova il comando dice che la chiave pubblica non è disponibile, che non è un fallimento della verifica ma la sua impossibilità: la chiave va importata prima, dal server di chiavi, e il suo identificativo si legge nel messaggio stesso.

```bash
gpg --keyserver keyserver.ubuntu.com --recv-keys 843938DF228D22F7B3742BC0D94AA3F0EFE21092
```

L'esito osservato il 2026-09-08, dopo l'importazione, è il seguente, e va riportato per intero perché contiene un avviso che si prende per un fallimento e non lo è.

```
gpg: Signature made Thu Aug 27 23:26:36 2026
gpg:                using RSA key 843938DF228D22F7B3742BC0D94AA3F0EFE21092
gpg: Good signature from "Ubuntu CD Image Automatic Signing Key (2012) <cdimage@ubuntu.com>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
```

La riga che conta è la terza, `Good signature`, e dice che il file delle somme è integro e firmato da quella chiave. L'avviso che segue non contraddice la riga precedente e non va letto come una verifica fallita: dice una cosa diversa, cioè che nel proprio anello di chiavi quella chiave non è stata firmata da nessuno di cui si sia dichiarata la fiducia. Sono due domande separate, l'autenticità della firma e la fiducia nella chiave, e `gpg` risponde alla prima con un sì e alla seconda con un non lo so. L'avviso sparisce solo dichiarando manualmente la fiducia in quella chiave, che è una scelta di chi la usa e non un requisito della verifica.

Va detto con precisione che cosa questa seconda verifica dimostra e che cosa no, perché è facile darle un valore che non ha. Dimostra che il file delle somme è stato firmato dalla chiave indicata. Non dimostra che quella chiave sia di Canonical, se la si è appena scaricata da un server di chiavi senza confrontarla con una fonte indipendente: la fiducia si ancora al fatto che l'impronta della chiave è pubblicata dalla distribuzione e che la stessa chiave firma i rilasci da anni. È comunque una difesa concreta, perché costringe un attacco a compromettere anche la chiave e non soltanto il sito.

Su Windows questi due comandi si eseguono nella shell POSIX che accompagna git, dove `sha256sum` e `gpg` esistono entrambi. Un dettaglio che altrimenti fa sospettare un file diverso: il file delle somme di Ubuntu scrive il nome precedendolo con un asterisco, nella forma `hash *nome`, che è la notazione della modalità binaria, e la stessa notazione la produce `sha256sum` nella shell di git, mentre su Linux la forma abituale è con due spazi. È la stessa trappola documentata in MS-039 di `docs/OPERATIONS-LOG.md`, dove un confronto fra impronte identiche risultava negativo per il solo separatore.

### 2.3 Scrivere la chiavetta

Il supporto deve essere di almeno 16 GB. Il vincolo non viene da un margine di prudenza ma dalla dimensione dell'immagine, 6,64 GiB: su una chiavetta da 8 GB, che di capacità utile ne offre circa 7,45 GiB, l'immagine entrerebbe per un soffio in modalità di scrittura diretta e non entrerebbe affatto se lo strumento dovesse costruire un filesystem con spazio di servizio. Una da 16 GB toglie la questione di mezzo.

Dalla postazione Windows lo strumento è Rufus, come per l'installazione originaria. Non richiede installazione: la variante il cui nome termina con `p` è portabile, cioè non scrive nel registro e non lascia niente sul sistema, e si scarica dalla pagina dei rilasci del progetto. La versione usata il 2026-09-08 è la 4.15.

La verifica dello strumento merita una riga, perché la strada corretta qui è diversa da quella usata per l'immagine. Per l'immagine si confronta una somma di controllo firmata; per un eseguibile Windows la verifica più forte è la firma Authenticode, che PowerShell controlla contro le radici di certificazione fidate del sistema invece che contro un valore pubblicato sullo stesso sito da cui si è scaricato il file.

```powershell
Get-AuthenticodeSignature "C:\Users\Utente\Desktop\_iso-ubuntu-studio\rufus-4.15p.exe" | Format-List Status, SignerCertificate
```

Controllo di uscita: `Status` vale `Valid` e il certificato è intestato a `Akeo Consulting`, che è l'autore di Rufus, con emittente una autorità di firma del codice. Un `NotSigned` o un `HashMismatch` a questo punto significa file manomesso o incompleto, e in quel caso si riscarica invece di procedere.

Le impostazioni da usare, con la ragione di ciascuna invece del solo elenco.

Il dispositivo è la chiavetta, e va controllato due volte perché Rufus la cancella per intero. La selezione di avvio è disco o immagine ISO, puntata al file verificato.

La scelta che conta davvero è la modalità di scrittura, e va detto *dove* si trova, perché non è dove la si cerca: non è un campo della finestra principale ma una finestra di dialogo che compare *dopo* aver premuto Avvia, quando Rufus riconosce che l'immagine è di tipo ibrido e offre modalità immagine ISO o modalità immagine DD. Cercarla fra le opzioni prima di avviare porta alla conclusione sbagliata che manchi qualcosa da configurare. *Va scelta DD.*

Che cosa distingue le due modalità. In modalità immagine ISO lo strumento crea sulla chiavetta una tabella delle partizioni, vi formatta un filesystem, estrae i file contenuti nell'immagine e li copia dentro, e installa un caricatore di avvio: la chiavetta è una *ricostruzione* del contenuto dell'immagine, dentro una struttura che lo strumento ha costruito. In modalità immagine DD l'immagine viene scritta sul dispositivo a partire dal settore zero, un byte dopo l'altro, senza essere interpretata: non si crea e non si formatta nulla, si sovrascrive tutto, tabella delle partizioni compresa, e la chiavetta diventa un *clone esatto* del file. Il nome viene dal comando Unix `dd`, che copia blocchi grezzi senza sapere che cosa contengano.

Le ragioni per cui in questa procedura si sceglie DD sono due, ed entrambe discendono dall'esattezza della copia. La prima è che la chiavetta risulta *verificabile a posteriori*, perché il suo contenuto coincide byte per byte con un file di cui si è già verificata l'impronta firmata; una chiavetta scritta in modalità ISO non è confrontabile con nulla, perché il suo contenuto è una struttura nuova che nessuna impronta descrive. La seconda è che l'avvio non dipende da un caricatore che lo strumento costruisce, ma è quello che l'immagine porta con sé, collaudato da chi l'ha pubblicata.

Il rovescio va detto, perché è ciò che lo strumento intende quando suggerisce l'altra modalità con la frase sull'avere pieno accesso all'unità dopo la scrittura: in DD la chiavetta è un installatore e nient'altro, non vi si possono aggiungere file e lo spazio residuo non è utilizzabile. Per riutilizzare il supporto in futuro occorre azzerarne la tabella delle partizioni, con `diskpart` e la sua operazione `clean` su Windows, e non basta una formattazione dall'interfaccia grafica.

Sul campo del sistema di file, che prima di premere Avvia mostra `Large FAT32`, va detto che si lascia così: è il valore che lo strumento propone presumendo la modalità ISO, e in DD diventa irrilevante perché nessun filesystem viene costruito. Vale però chiarire che cosa quella variante fa, perché il nome inganna: `Large FAT32` rimuove il limite di dimensione del *volume*, consentendo FAT32 su supporti oltre i 32 GB, e *non* rimuove il limite di 4 GiB meno un byte per singolo file, che è un vincolo diverso e appartiene al formato.

Su quest'ultimo punto va registrata una *inferenza ritirata*, perché era stata scritta qui come motivo principale della scelta di DD ed era falsa. Avevo affermato che dentro una immagine live di questa dimensione il filesystem compresso del sistema superasse i 4 GiB, rendendo la modalità ISO impraticabile. La misura sul contenuto dell'immagine dice il contrario: il file più grande è `casper/minimal.squashfs` con 4.112.433.152 byte, cioè 3,83 GiB, sotto il limite di 4.294.967.295. La modalità ISO avrebbe quindi funzionato, e il suggerimento dello strumento era corretto. La scelta di DD resta valida per le due ragioni dette sopra, che non dipendono da questa, ma il motivo sbagliato è stato rimosso invece di essere lasciato a reggere una conclusione giusta. Il margine è del quattro per cento, ed è la ragione per cui l'ipotesi era plausibile: su un'altra derivata, o su una versione futura, quel file può superare la soglia, e in quel caso l'argomento tornerebbe valido. Va misurato, non assunto, e la misura costa un comando.

```powershell
& "C:\Program Files\7-Zip\7z.exe" l "C:\Users\Utente\Desktop\_iso-ubuntu-studio\ubuntustudio-26.04.1-desktop-amd64.iso"
```

La configurazione corretta, verificata su Rufus 4.15 il 2026-09-08, ha questo aspetto: dispositivo la chiavetta e non un altro disco USB collegato, tipo boot sistema l'immagine con la spunta verde di riconoscimento, dimensione partizione persistente a zero perché si sta preparando un installatore e non un sistema live con memoria, schema GPT, sistema destinazione UEFI senza CSM. Con questi valori lo stato in fondo alla finestra dice pronto, e l'unica cosa che resta è premere Avvia.

Una avvertenza sul dispositivo, che è il punto in cui un errore costa caro, va data insieme alla sua mitigazione, altrimenti descrive un rischio più grande di quello reale. La casella `Elenco unità disco USB` deve restare *deselezionata*, ed è il presidio che tiene i dischi rigidi e gli SSD esterni fuori dall'elenco dei dispositivi, lasciandovi solo le unità rimovibili: con quella casella spenta un disco di lavoro collegato non è nemmeno selezionabile, e la barra di stato lo conferma dichiarando quanti dispositivi sono stati rilevati. Se per qualche ragione la si attiva, allora sì, il disco esterno compare accanto alla chiavetta e Rufus cancella per intero ciò che gli si indica: in quel caso la distinzione si fa sulla capacità dichiarata accanto al nome, non sulla posizione nell'elenco, che cambia. La conclusione operativa è che quella casella non si attiva senza un motivo preciso.

Le tre opzioni avanzate di formattazione, cioè formattazione rapida, creazione dell'etichetta estesa con i file icona, e test dei blocchi errati, si lasciano ai valori predefiniti. Le prime due appartengono alla modalità ISO, dove un filesystem viene costruito, quindi in modalità DD non hanno oggetto. Il test dei blocchi errati va lasciato spento: su un supporto nuovo aggiunge una lettura completa dell'intera capacità senza dire nulla che l'esito della scrittura non dica già, e su un supporto sospetto la domanda a cui rispondere non è se la chiavetta abbia blocchi difettosi ma se valga la pena usarla.

Lo schema di partizione è GPT e il sistema di destinazione è UEFI senza compatibilità CSM, perché l'installazione esistente su questa macchina è UEFI e la sua partizione EFI va riusata, come stabilisce la fase 3.

Una avvertenza sul dopo, che è il punto in cui si rovina una chiavetta appena fatta. Terminata la scrittura in modalità DD, Windows vede sulla chiavetta uno spazio non allocato oltre le partizioni dell'immagine e propone di formattarlo, a volte con un avviso che il disco va inizializzato. *Va rifiutato.* Quell'operazione riscrive la tabella delle partizioni e rende il supporto non avviabile, e il fatto che l'avviso sembri una richiesta di manutenzione ordinaria è esattamente ciò che la rende insidiosa.

### 2.4 Verificare la chiavetta scritta

La verifica giusta è quella che il supporto fa su se stesso, ed è nel menu di avvio della chiavetta alla voce `Check disc for defects`. Richiede un paio di minuti, controlla le somme che l'immagine porta al proprio interno, e risponde esattamente alla domanda utile, cioè se il supporto rilegge integro ciò che vi è stato scritto. Va fatta al primo avvio, prima di entrare nell'installatore.

Questa sottofase, nella sua prima stesura, prescriveva invece di confrontare l'impronta dei primi byte del dispositivo grezzo con quella dell'immagine, sul presupposto che una chiavetta scritta in modalità DD debba restare identica al file. *Quel presupposto è falso su Windows*, e la sua smentita è documentata in MS-069: il confronto è stato eseguito e ha dato impronte diverse su una chiavetta perfettamente valida.

Le ragioni per cui è falso sono tre e sono tutte legittime scritture del sistema operativo, non guasti. Una immagine ibrida porta una tabella GPT dimensionata sull'immagine, con la copia di sicurezza alla propria fine; scritta su un supporto molto più grande quella copia si trova a metà disco invece che in fondo, e il sistema la ripara spostandola e riscrivendo l'intestazione primaria, il campo che punta alla copia e il codice di controllo. La zona da cui la copia è stata rimossa cambia a sua volta. E le partizioni scrivibili che il sistema monta con una lettera propria, fra cui la partizione di sistema EFI che è formattata FAT, ricevono le cartelle di servizio che Windows crea al primo accesso.

Ne segue la regola, che vale oltre questo caso: un confronto byte per byte fra una immagine e il supporto su cui è stata scritta ha senso soltanto su un sistema che non monta automaticamente i volumi e non ripara le tabelle delle partizioni. Su Windows la differenza è la norma e non l'eccezione, quindi un confronto integrale non è una verifica ma un generatore di falsi allarmi.

Lo strumento `tools/verify-usb-dd.ps1` resta nel progetto ma *non appartiene più a questa procedura*: il suo uso legittimo è confrontare un'immagine con una copia grezza che nessun sistema ha montato, per esempio un archivio di un supporto. Il suo docstring lo dichiara.

Riferimento dell'esecuzione del 2026-09-08, utile per riconoscere una scrittura riuscita senza verifiche aggiuntive: la scrittura ha richiesto otto minuti e sette secondi, e a fine scrittura la chiavetta presenta tre partizioni in tabella GPT, cioè il volume principale dell'immagine di circa 6792 MB, una partizione di sistema EFI di 5 MB e una partizione ausiliaria di 0,3 MB, le ultime due delle quali Windows monta con lettere proprie. Sono le partizioni dell'immagine ibrida e non una struttura costruita dallo strumento di scrittura, che in modalità immagine ISO ne avrebbe creata una sola: è quindi la conferma leggibile a occhio che la modalità DD è stata effettivamente usata.

### 2.5 Scrivere la chiavetta dalla macchina Linux, in alternativa

Dalla macchina Linux, se ancora funzionante, lo strumento equivalente è il seguente, dove il dispositivo di destinazione va identificato con certezza prima di lanciarlo.

```bash
lsblk -f
sudo dd if=ubuntustudio-26.04.1-desktop-amd64.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

Attenzione al nome del dispositivo: `dd` scrive dove gli si dice senza chiedere conferma, e indicare per errore il disco di sistema invece della chiavetta lo distrugge. Il primo comando serve a identificare la chiavetta dalla sua dimensione e dalla sua etichetta, e va eseguito con la chiavetta inserita e poi rimossa, per confrontare i due elenchi.

## Fase 3: controlli del firmware

Obiettivo: assicurarsi che la macchina si avvii dalla chiavetta e che il sistema nuovo trovi il firmware nello stato in cui lo si aspetta.

Si entra nel setup UEFI all'avvio. Quattro voci da verificare.

L'avvio in modalità UEFI e non in compatibilità BIOS, perché l'installazione originaria è UEFI e la partizione EFI esistente va riusata.

Il Secure Boot. Ubuntu Studio si installa anche con Secure Boot attivo, ma se in futuro serviranno moduli del kernel firmati a mano la cosa si complica; per una macchina audio domestica la scelta più semplice è lasciarlo come è e occuparsene solo se un driver lo richiede.

L'ordine di avvio, per mettere la chiavetta prima del disco interno, oppure il menu di avvio temporaneo, che è la strada preferibile perché non lascia modifiche permanenti.

Il risparmio energetico e il Wake-on-LAN. Quest'ultimo va abilitato adesso, e la ragione è la sezione 10.2: la macchina si sospende e sparisce dalla rete, e senza Wake-on-LAN va risvegliata a mano.

## Fase 4: l'installazione

È la fase irreversibile. Va eseguita con la fase 1 completata e verificata.

### 4.1 Avvio dell'installatore

Si avvia dalla chiavetta e si scegli di provare o installare. Conviene, prima di lanciare l'installazione, aprire un terminale nell'ambiente live e ricontrollare la mappa del disco, perché i nomi dei dispositivi nell'ambiente live possono differire da quelli del sistema installato.

```bash
lsblk -f
sudo blkid
```

Controllo: la mappa coincide con quella registrata in fase 0.2. In particolare si identificano con certezza quale partizione è la EFI, quale è root, quale è la swap e quale è `/home`.

### 4.2 Partizionamento manuale

Si sceglie il partizionamento manuale, cioè la voce che l'installatore chiama *Something else* o *Partizionamento manuale*. Non si sceglie in nessun caso la cancellazione del disco né l'installazione guidata, perché entrambe rifarebbero la tabella delle partizioni e porterebbero via `/home`. La schermata in cui si decide compare dopo lingua, tastiera e rete, ed è il primo dei tre soli momenti in cui questa fase si può sbagliare.

Le quattro partizioni vanno configurate così.

| Partizione | Dimensione reale | Mount point | Formattare | Nota |
|---|---|---|---|---|
| `nvme0n1p1`, FAT32 | 1 GB, 6,2 MB occupati | `/boot/efi` | **no** | si riusa quella esistente; formattarla non è necessario e sarebbe un rischio inutile. Il documento sorgente la dichiarava intorno ai 100 MB |
| `nvme0n1p2`, ext4 | 73 GB, 32 per cento usato | `/` | **sì**, ext4 | è la sola partizione da azzerare, contiene solo sistema e programmi |
| `nvme0n1p3`, swap | | nessuno | sì | si riusa così come è |
| `nvme0n1p4`, ext4 | **346 GB**, 1 per cento usato | `/home` | **no** | qui vivono progetti, materiali trasferiti e prefix Wine |

Il punto di attenzione assoluto è l'ultima riga, e la trappola concreta va nominata perché non è la casella in sé: nell'installatore la casella di formattazione *si attiva da sola* quando si seleziona un filesystem nel menu della riga. Su `/home` il filesystem non si tocca affatto, si imposta soltanto il punto di montaggio, ed è il secondo dei tre momenti in cui questa fase si può sbagliare.

Il terzo momento è la schermata di riepilogo che l'installatore mostra prima di scrivere, ed è l'unico controllo che conta davvero, perché elenca le operazioni che verranno eseguite e perché fino a quel pulsante nulla è stato scritto sul disco. Deve comparire una formattazione *soltanto* per `nvme0n1p2`. Se la parola compare accanto a `nvme0n1p4` o a `nvme0n1p1`, si torna indietro.

Se l'installatore lo permette, conviene catturare uno screenshot della schermata di riepilogo prima di confermare: è la prova di che cosa è stato chiesto, e in caso di esito inatteso è l'unico documento che dice se l'errore era nella richiesta o nell'esecuzione.

### 4.3 Le altre scelte dell'installatore

Il nome utente va mantenuto identico a quello attuale, cioè `alesop95`. Questa non è una preferenza estetica: i file sotto `/home/alesop95` appartengono a un identificativo utente numerico, e creare un utente con nome diverso, o con un identificativo numerico diverso, lascerebbe tutta la home appartenente a un utente inesistente, con permessi da riparare a mano. Se l'installatore assegna comunque un identificativo diverso, la riparazione è nella fase 5.

La cifratura del disco non va attivata, perché cifrerebbe solo la partizione nuova e complicherebbe l'accesso a `/home` esistente.

L'installazione di codec e driver di terze parti conviene accettarla, perché su una macchina audio e video serve.

Controllo di uscita della fase 4: l'installazione termina senza errori e la macchina si riavvia dal disco interno.

## Fase 5: primo avvio e verifica dell'integrità

Obiettivo: accertarsi che il sistema nuovo veda `/home` come prima e che nulla sia andato perduto, prima di installare qualsiasi cosa.

### 5.1 Verifica del montaggio e dei permessi

```bash
lsblk -f
df -h
cat /etc/fstab
ls -la /home/
ls -la /home/alesop95/ | head -30
id
stat -c "%U %G %n" /home/alesop95
```

Esito atteso: `/home` è montata sulla sua partizione, i file di prima ci sono, e la proprietà dei file corrisponde all'utente corrente. Se `stat` mostrasse un identificativo numerico invece di un nome utente, significa che l'identificativo assegnato dall'installatore è diverso da quello di prima, e la riparazione è la seguente.

```bash
sudo chown -R alesop95:alesop95 /home/alesop95
```

### 5.2 Verifica dei materiali trasferiti

```bash
cd ~/electroacoustics
find . -type f -exec sha256sum {} + | sort -k2
ls -lh installers examples licenze sorgenti
```

Le impronte vanno confrontate con quelle in `docs/TRANSFER-MANIFEST.md`. Coincidono se la partizione non è stata toccata, ed è la prova che la fase 4 è andata come doveva.

### 5.3 Aggiornamento della base

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt autoremove -y
lsb_release -a
uname -r
```

Esito atteso: `apt update` non produce errori 404, che è la differenza più visibile rispetto al punto di partenza, e `lsb_release` dichiara la 26.04.

## Fase 6: verifica della catena audio

Obiettivo: accertarsi che la configurazione a bassa latenza sia in vigore e che il sottosistema audio funzioni, prima di costruire l'ambiente Wine sopra. Il controllo non riguarda una interfaccia esterna specifica: al 2026-09-09 la macchina ha la sola scheda integrata `ALC887-VD`, e quale interfaccia servirà alla misura è una decisione aperta, si veda MS-079.

```bash
uname -r
aplay -l
arecord -l
pactl info
systemctl --user status pipewire pipewire-pulse wireplumber
groups
```

*Attenzione: il controllo sul nome del kernel è sbagliato e darebbe un falso negativo.* Lo diceva una versione precedente di questa fase, e la fotografia del 2026-09-07 lo ha smentito. Su Ubuntu Studio 25.04 non è installato alcun `linux-image-lowlatency`: il kernel è generico, e la configurazione a bassa latenza è ottenuta tramite parametri di avvio. Chi cercasse un kernel chiamato lowlatency concluderebbe che manchi, e installerebbe un pacchetto che non serve.

Il controllo corretto è sulla riga di comando del kernel e sui limiti di priorità in tempo reale.

```bash
cat /proc/cmdline
cat /etc/security/limits.d/30-ubuntustudio-audio.conf
dpkg -l | grep -i lowlatency
systemctl --user is-active pipewire pipewire-pulse wireplumber
```

I due parametri che contano in `/proc/cmdline` sono `preempt=full`, che abilita la prelazione completa del kernel, e `threadirqs`, che sposta la gestione degli interrupt in thread schedulabili: sono le proprietà per cui esisteva un kernel separato. Sulla 26.04 osservata il 2026-09-09 se ne aggiunge un terzo, `rcu_nocbs=all`, e soprattutto va corretta la loro provenienza: non stanno in `/etc/default/grub`, che contiene soltanto `quiet splash`, ma in `/etc/default/grub.d/ubuntustudio.cfg`, un file drop-in che il sistema installa da sé ed estende la variabile invece di sostituirla. Aggiungerli a mano al file principale, come una versione precedente di questa fase prescriveva, li duplicherebbe sulla riga di comando del kernel. Si verifica e basta; solo se mancassero si interviene, e in quel caso nel drop-in. I limiti attesi sono `rtprio 95` e `memlock unlimited` per i gruppi `audio` e `pipewire`, e li configura il pacchetto `ubuntustudio-lowlatency-settings`, che è un pacchetto di impostazioni e non un kernel. I tre servizi devono risultare tutti attivi.

Se sulla 26.04 questo schema fosse cambiato, la fonte è la documentazione ufficiale di Ubuntu Studio per quel rilascio, non l'assunzione: il confronto va fatto contro i valori registrati nella fotografia del 2026-09-07, che sono lo stato noto e funzionante di partenza.

Il comando `groups` deve mostrare l'appartenenza ai gruppi `audio` e `pipewire`, che è ciò che abilita le priorità in tempo reale. Il 2026-09-09 mancavano entrambi su una installazione appena fatta, ed è un difetto che l'installatore introduce di serie: si veda MS-074.

Su questo controllo va segnalata una trappola, perché la verifica ingenua produce un falso positivo. Leggere i valori dentro `/etc/security/limits.d/` non dimostra nulla, dato che quei valori sono giusti anche quando non arrivano a nessuno: concedono `rtprio 95` e `memlock unlimited` ai due gruppi, e se l'utente non vi appartiene i limiti restano scritti e non in vigore. La prova sta nel confronto con ciò che la sessione riporta davvero, cioè `ulimit -r -l`, che in quel caso risponde `0` e `8192`. È la stessa differenza fra una regola scritta e una regola applicata che ha già prodotto il falso negativo sul nome del kernel poche righe sopra.

Se i gruppi mancano si aggiungono, tenendo presente che l'appartenenza si applica al login e non alla sessione in corso: la verifica va quindi fatta in una sessione di login nuova, oppure dopo un riaccesso.

```bash
sudo usermod -aG audio,pipewire alesop95 && id -nG alesop95
```

Controllo di uscita: i dispositivi audio presenti compaiono in ingresso e in uscita e una riproduzione di prova si sente, con i parametri di avvio confermati in `/proc/cmdline` e i limiti realtime confermati da `ulimit -r -l` e non dalla sola lettura dei file di configurazione.

Sul perché questo controllo non nomina più una interfaccia esterna va registrata la correzione. Al 2026-09-07 e di nuovo al 2026-09-09 `lsusb` non riportava alcun dispositivo Focusrite, e le sole schede viste erano l'audio integrato `ALC887-VD` con le sue uscite HDMI. La versione precedente di questa fase ne concludeva che la Scarlett 2i2 fosse presente ma staccata, perché altre pagine la dichiaravano come hardware della macchina; l'utente ha chiarito il 2026-09-09 di possederla ma di non impiegarla in questo progetto. Il controllo si esegue quindi sui dispositivi effettivamente presenti, e il requisito di un ingresso con alimentazione phantom resta aperto. Si veda MS-079.

## Fase 7: ricostruzione dell'ambiente Wine

Obiettivo: un ambiente pulito, con un prefix per programma, senza l'architettura a 32 bit e senza la sedimentazione che aveva prodotto i guasti su `kernel32.dll`.

La procedura completa, con il razionale di ogni passo, sta in `docs/10-ambiente/wine-configurazione.md`. Qui la sequenza nell'ordine di questa installazione, con le due differenze deliberate rispetto al passato.

*Attenzione: questa fase è stata corretta il 2026-09-07 e la versione precedente era sbagliata.* Prescriveva quattro prefix a 64 bit con `dotnet48` per tutti, sulla base dell'affermazione del documento sorgente secondo cui Akabak 3 sarebbe a 64 bit. L'ispezione del prefix funzionante sulla macchina l'ha smentita: si veda ADR-016.

I fatti misurati. Il prefix in uso è `~/.wine` e il suo registro dichiara `#arch=win32`, cioè è a *32 bit*. `AKABAK.exe` installato è `PE32 executable, Intel 80386`, cioè *a 32 bit*, come `VACS_32.exe`. Nel prefix non c'è alcun `winetricks.log`, non c'è `Microsoft.NET/Framework/v4` e non ci sono font Microsoft di base: Akabak e VACS funzionano *senza nessuna delle dipendenze che il documento sorgente prescriveva*.

Ne segue la prima differenza rispetto alla versione precedente di questa fase: *l'architettura `i386` va dichiarata*, perché è necessaria e non residua. Senza di essa il solo software del progetto che oggi funziona non funzionerebbe.

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install --install-recommends wine winetricks
wine --version
```

Attenzione al nome del pacchetto, corretto il 2026-09-09 dopo averlo verificato sulla macchina: su Ubuntu 26.04 il pacchetto `wine-stable` non esiste e `apt-cache policy` risponde `Candidate: (none)`. Quel nome appartiene ai repository WineHQ, che il sistema precedente aveva attivi e che questa installazione non usa; il pacchetto dell'archivio Ubuntu si chiama `wine`, versione `10.0~repack-12ubuntu1`. Va notato inoltre che `wine32` risulta indisponibile finché l'architettura `i386` non è dichiarata, il che è la conferma pratica dell'ordine imposto da ADR-016. Il racconto è in MS-081.

La seconda differenza resta valida: si decide una sola provenienza dei pacchetti e non si mescola, perché la fotografia ha trovato due repository WineHQ attivi per due rilasci diversi di Ubuntu.

Un passo che la procedura non conteneva e senza cui la decisione sui 32 bit resta inapplicabile. Su Ubuntu il comando `wine` è un collegamento che porta a uno script, il quale usa il caricatore a 64 bit se `wine64` è eseguibile e ripiega su quello a 32 bit soltanto se manca: con entrambi i rami installati, come questa fase impone, il caricatore a 32 bit non viene mai scelto, e su un prefix `win32` l'avvio fallisce con `is a 32-bit installation, it cannot support 64-bit applications`. Ogni comando rivolto al prefix a 32 bit va quindi dato con `wine32`, a cui `WINEPREFIX` va sempre passato esplicitamente perché altrimenti ne assume uno proprio. La diagnosi è in MS-084.

```bash
WINEPREFIX=~/wineprefixes/akabak32 wine32 "C:/Program Files/RDTeam/AKABAK/AKABAK.exe"
```

Poi i prefix, quattro, con *architetture diverse* e non tutte a 64 bit.

```bash
WINEARCH=win32 WINEPREFIX=~/wineprefixes/akabak32 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/vituixcad64 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/easefocus64 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/arta64 winecfg
```

Le dipendenze si installano soltanto dove servono, e per Akabak e VACS *non ne servono*: la configurazione funzionante non ne ha nessuna, e aggiungerne sarebbe riprodurre i tentativi del troubleshooting invece della soluzione.

```bash
WINEPREFIX=~/wineprefixes/vituixcad64 winetricks -q dotnet48 corefonts
WINEPREFIX=~/wineprefixes/easefocus64 winetricks -q dotnet48 corefonts vcrun2013 vcrun2019
WINEPREFIX=~/wineprefixes/arta64 winetricks -q vcrun2019 corefonts
```

Per VituixCAD ed EASE Focus i requisiti restano quelli dichiarati dai produttori, perché su questa macchina non sono mai stati installati: non c'è nulla da riprodurre e nulla da smentire, quindi si parte da quanto documentato e si corregge sull'esito.

In ciascun prefix, dalla scheda delle applicazioni di `winecfg` si imposta la versione di Windows su Windows 10, e dalla scheda della grafica si attiva la decorazione delle finestre da parte del window manager.

Controllo di uscita della fase 7: i quattro prefix esistono, ciascuno contiene il proprio `system.reg`, e `winecfg` si apre in ciascuno senza errori su `kernel32.dll`. Il controllo che vale più degli altri è l'architettura dichiarata, perché è il punto su cui la versione precedente di questa fase sbagliava.

```bash
find ~/wineprefixes -maxdepth 2 -name system.reg -exec grep -H -m1 "#arch" {} +
```

Esito atteso: `win32` per `akabak32` e `win64` per gli altri tre. La mappa completa dei prefix, con i programmi e le dipendenze di ciascuno, è in `wine-corredo-progetto-stanza.md`.

```bash
find ~/wineprefixes -maxdepth 2 -name "system.reg" -printf "%h\n"
```

## Fase 8: reinstallazione dei programmi e riattivazione della licenza

### 8.1 Akabak e VACS

Gli installer sono già sulla macchina, portati dalla fase 1.

```bash
cd ~/electroacoustics/installers
WINEPREFIX=~/wineprefixes/akabak32 wine AKABAK_Pro_v324b126.exe
WINEPREFIX=~/wineprefixes/akabak32 wine VACS_32_v213b33.exe
```

Due correzioni rispetto alla versione precedente di questo passo, entrambe da ADR-016. Il prefix è a *32 bit* e non a 64, perché l'eseguibile installato è PE32 i386. E la variante di VACS è la *32 bit*, non la 64: è quella che il lanciatore sulla scrivania della macchina invoca, cioè `VACS_32.exe` in `C:\Program Files\RDTeam\VACS2`.

Quest'ultimo dato chiude una lacuna che lo storico di Akabak e VACS aveva dichiarato aperta, cioè quale variante fosse installata e come fosse stato risolto il fallimento iniziale di VACS. La risposta è che fu risolto usando la build a 32 bit, che era esattamente l'ipotesi formulata nella corrispondenza del 13 agosto 2025: era corretta.

I due programmi vanno nello stesso prefix, perché si usano in sequenza e perché così è la configurazione che funziona sulla macchina. La variante a 64 bit di VACS resta come riserva non usata.

### 8.2 Inserimento del Release Code

È il momento che chiude il cerchio con la fase 0.6 e che verifica praticamente l'affermazione su cui poggia tutta la decisione: che una licenza legata alla macchina sopravvive alla reinstallazione del sistema.

Si apre Akabak nel prefix corretto, si va nel menu di aiuto alla voce del release code, si controlla che il Machine Identifier mostrato sia lo stesso letto in fase 0.6, e si inserisce il codice permanente. I due valori sono nella scheda riservata sotto `_notes/`, non nel repository.

```bash
WINEPREFIX=~/wineprefixes/akabak32 wine "C:/Program Files/RDTeam/AKABAK/AKABAK.exe"
```

Esito atteso: il Machine Identifier è invariato, il codice viene accettato, e all'avvio successivo di VACS non viene richiesto un secondo codice, perché un solo codice copre entrambi i programmi.

Se il Machine Identifier fosse cambiato, non si insiste: si contatta l'autore. Il fatto che sia cambiato sarebbe una informazione tecnica interessante di per sé, perché indicherebbe che l'identificativo dipende da qualcosa che la reinstallazione tocca, e in quel caso l'affermazione registrata in ADR-003 andrebbe corretta con una voce nuova nel registro delle decisioni.

### 8.3 Il limite delle pipeline COM, da configurare subito

L'autore del software ha dichiarato che su Linux il trasferimento dei dati fra AKABAK e VACS avviene attraverso gli appunti di sistema e non tramite COM. L'impostazione relativa sta nelle preferenze di AKABAK, e conviene verificarla adesso invece di scoprirla alla prima iterazione della fase 5 del progetto. Il dettaglio sta in `docs/90-riferimenti/timeline-akabak-vacs.md`.

### 8.4 Gli esempi di Akabak

```bash
cd ~/electroacoustics/examples
unzip AKABAK-Examples.zip -d ~/AkabakProjects/esempi
ls ~/AkabakProjects/esempi | head
```

Il pacchetto degli esempi è materiale di lavoro e non documentazione accessoria: la fase 5 del progetto comincia adattando un esempio di sistema a due vie.

### 8.5 VituixCAD

L'installer è a 32 bit ma l'applicazione è .NET a 64 bit, quindi va nel prefix a 64 bit. È la distinzione fra architettura dell'installer e architettura dell'applicazione, spiegata in `wine-corredo-progetto-stanza.md`.

```bash
cd ~/electroacoustics/progetto-stanza/diy
WINEPREFIX=~/wineprefixes/vituixcad64 wine VituixCAD_setup.exe
```

### 8.6 EASE Focus 3.1.260 e il servizio di database AFMG

Qui la procedura corregge due errori del documento sorgente, entrambi scoperti ispezionando i file e documentati come incoerenza 6 in `docs/90-riferimenti/incoerenze-sorgente.md`.

Il primo è il nome dell'installer. Il sorgente indicava `EASE_Focus_Setup_v3.1.260.exe`, e quel file non esiste: il pacchetto è un InstallShield composto da `setup.exe`, `EASE Focus 3.msi`, `Data1.cab` e `ISSetup.dll`. Ne segue che va lanciato da dentro la sua cartella, perché `setup.exe` cerca gli altri file accanto a sé, e copiare il solo eseguibile altrove produce un errore che non nomina la causa vera.

Il secondo è un passo che mancava del tutto. Accanto all'installer principale ci sono `AFMGDatabaseService` e `AFMGDatabaseService_x64`, due installer MSI distinti: è il servizio di database che la tabella del changelog nello stesso documento valutava di impatto alto, perché evita di scaricare a mano ogni GLL dal sito del costruttore. Va installato, nella variante a 64 bit dato che il prefix è a 64 bit.

```bash
cd ~/electroacoustics/progetto-stanza/room/EASE_Focus_v3.1.260
WINEPREFIX=~/wineprefixes/easefocus64 wine setup.exe
cd AFMGDatabaseService_x64
WINEPREFIX=~/wineprefixes/easefocus64 wine setup.exe
```

Sulle dipendenze, i requisiti reali sono più larghi di quanto la documentazione ufficiale lasci intendere, perché alcuni moduli GLL portano DLL proprie compilate con compilatori diversi. Se un GLL specifico non si carica, un redistributable Visual C++ mancante è il primo sospetto.

Poi il database dei GLL, che sono dati e non un programma: 221 file, di cui 174 con estensione `.gll`, per 451 MB. La copia deve essere dell'intera cartella e non selettiva sui soli `.gll`, e la ragione è precisamente la differenza fra quei due numeri: alcuni costruttori distribuiscono un `.gll` accompagnato da file `.dll` e `.bin`, e i tre devono restare nella stessa cartella o il modulo non si carica.

```bash
mkdir -p ~/wineprefixes/easefocus64/drive_c/users/$USER/Documents/EASE_Focus_3_GLL
cp -r ~/electroacoustics/progetto-stanza/room/EASE_Focus_3_GLL_Database_2016_10_11/. ~/wineprefixes/easefocus64/drive_c/users/$USER/Documents/EASE_Focus_3_GLL/
ls ~/wineprefixes/easefocus64/drive_c/users/$USER/Documents/EASE_Focus_3_GLL | wc -l
```

Il percorso esatto in cui il programma cerca i GLL va confermato dalle sue preferenze al primo avvio, perché dipende dalla versione: la cartella creata qui è una destinazione di comodo, e se il programma ne indica un'altra si sposta il contenuto invece di duplicarlo.

Di EASE Focus si installa soltanto la 3.1.260. La 3.0.18 e la 3.1.10 restano materiale d'archivio: tre versioni dello stesso programma in tre prefix sono manutenzione senza ritorno, e la retrocompatibilità dei GLL rende la più recente sufficiente. Della 3.1.10, sulla copia di lavoro del corredo, esiste soltanto un collegamento e non la cartella: il contenuto vive probabilmente sul solo SSD esterno.

Un punto da provare e non da assumere: il servizio di database è un servizio Windows, e sotto Wine non gira come servizio di sistema ma come processo dentro il prefix, dipendente da `wineserver`. Se il programma lamenta l'assenza del database, la verifica è che il servizio sia stato installato in quel prefix e non in un altro.

### 8.7 ARTA 1.7.1

ARTA non era nel piano originale e si aggiunge, perché è la via più diretta per produrre un file GLL da un diffusore misurato, quindi serve alla fase 8 del progetto, quando il monitor autocostruito esiste e va caratterizzato. Fino a quel momento resta inutilizzato: la misura la fa REW, che è nativo.

È shareware, e senza registrazione funziona in modalità dimostrativa con limitazioni da verificare al momento dell'uso, tracciate come punto aperto.

```bash
cd ~/electroacoustics/progetto-stanza/diy/Arta
WINEPREFIX=~/wineprefixes/arta64 wine ArtaSetup171.exe
```

Un avvertimento sull'uso: ARTA è un programma di misura e vuole accedere alla scheda audio, e sotto Wine quell'accesso passa dal driver audio di Wine verso PipeWire, con latenza e stabilità che non sono quelle di un programma nativo. Per produrre un GLL da misure già acquisite il problema non si pone, perché si lavora su file. Per una misura dal vivo si usa REW, che è nativo.

### 8.8 Ramsete 27b, solo se la licenza lo consente

Da non installare in questa fase. Ramsete è una applicazione Visual Basic 6, quindi a 32 bit e con bisogno del runtime `vb6run`, e richiederebbe un prefix a 32 bit più la dichiarazione dell'architettura `i386` sul sistema, che è uno dei fattori di attrito degli aggiornamenti di rilascio da cui questa procedura sta uscendo.

Soprattutto, il suo stato di licenza non è accertato: la cartella non mostra indizi di manomissione, ma Ramsete è un prodotto commerciale e non è stato verificato se questa sia una versione dimostrativa liberamente distribuibile. La verifica è tracciata come PA-002 in `docs/PENDING-ACTIONS.md`, e va fatta prima di installare, non dopo.

Il consiglio operativo è quindi di non dichiarare l'architettura `i386` durante questa installazione, e di aggiungerla soltanto se e quando Ramsete supera la verifica. Rimandare quel passo costa un comando; anticiparlo costa attrito permanente. La procedura, se si arriverà a eseguirla, è in `wine-corredo-progetto-stanza.md`.

Va aggiunto un giudizio di priorità che rende la cosa meno urgente di quanto sembri: il ruolo di Ramsete sarebbe l'acustica architettonica, già coperta da Akabak, che fa elettroacustica e acustica ambientale in un solo passaggio ed è licenziato e funzionante. È un supplemento facoltativo, non un tassello mancante.

### 8.9 Che cosa non si installa, e perché non manca nulla

Otto voci del corredo, per circa 1,7 GB, non entrano nel piano perché portano protezioni rimosse o provenienza non lecita. L'elenco e la motivazione voce per voce sono in `wine-corredo-progetto-stanza.md`.

Il punto che conta è che nessuna serve. FineCone e FineMotor simulano cono e motore magnetico di un altoparlante, quindi servono a chi costruisce gli altoparlanti e non a chi li assembla in un sistema partendo dai parametri Thiele/Small. LSPCad e Grenander Loudspeaker Lab progettano crossover e casse, e il loro ruolo è coperto da VituixCAD, gratuito, molto più recente e con gestione della direttività. AmpliTube, Guitar Rig e POD Farm sono emulatori di amplificatori per chitarra, senza alcuna relazione con la progettazione elettroacustica; se servissero per il progetto gemello di home recording, gli equivalenti nativi liberi su Linux sono Guitarix e Rakarrack.

Controllo di uscita della fase 8: Akabak, VACS, VituixCAD, EASE Focus 3.1.260 e ARTA aprono la propria finestra; Akabak è attivato con il codice esistente e VACS non ne chiede un secondo; EASE Focus carica almeno un GLL dal database; i prefix sono quattro e non cinque, perché Ramsete resta sospeso.

## Fase 9: strumenti nativi del progetto

```bash
sudo apt install rew 2>/dev/null || echo "REW si installa dal sito ufficiale, non dai repository"
sudo apt install blender freecad octave -y
octave --eval "disp(version)"
blender --version
freecad --version
```

REW non è nei repository di Ubuntu e si installa dal sito del suo autore. MATAA è un insieme di funzioni per Octave e si installa aggiungendone il percorso, non come pacchetto di sistema.

## Fase 10: igiene post-installazione

Sono i quattro interventi che rendono la macchina comoda da usare e che, se saltati, riproducono i problemi da cui si è partiti.

### 10.1 Configurare la politica di aggiornamento per il futuro

```bash
cat /etc/update-manager/release-upgrades
sudo sed -i 's/^Prompt=.*/Prompt=lts/' /etc/update-manager/release-upgrades
cat /etc/update-manager/release-upgrades
```

Su una LTS la direttiva `Prompt=lts` è quella corretta: propone il passaggio soltanto alla LTS successiva, cioè la 28.04, e ignora le versioni intermedie. È la stessa direttiva che sulla 25.04 contribuiva al blocco, e non è una contraddizione: era sbagliata là perché la macchina non era su una LTS, ed è giusta qui perché lo è. Questa singola riga è ciò che impedisce alla situazione di ripetersi.

### 10.2 Sospensione automatica e accesso remoto

Questo intervento nasce da una osservazione fatta sul campo durante questa sessione, e vale raccontarla perché è istruttiva.

La macchina risultava irraggiungibile: nessuna risposta al ping, nessuna voce nella tabella ARP della postazione, connessione alla porta 22 in timeout. La lettura corretta l'ha data l'utente: la macchina si sospende da sola dopo un periodo di inattività. Una macchina sospesa non risponde alle richieste ARP, quindi non solo non risponde: scompare del tutto dal punto di vista della rete, ed è esattamente il quadro osservato, con gli indirizzi vicini presenti nella tabella e il suo assente. Alla ripresa dell'attività l'indirizzo è ricomparso e il ping ha risposto con TTL 64.

Ci sono due modi di risolvere, e conviene applicarli entrambi perché coprono casi diversi.

Il primo è impedire la sospensione quando la macchina deve restare disponibile. Su un desktop che serve anche da postazione raggiungibile in rete, la sospensione automatica è una impostazione da spegnere.

```bash
gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type
gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
systemctl status sleep.target suspend.target
```

Su Ubuntu Studio l'ambiente grafico è KDE Plasma e non GNOME, quindi il comando `gsettings` potrebbe non essere quello pertinente: in quel caso la sospensione si disattiva dalle impostazioni di risparmio energetico dell'ambiente, e il controllo a livello di sistema è il seguente.

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
systemctl status sleep.target
```

Va detto che questo secondo comando è drastico: impedisce la sospensione anche quando la si vuole. Su una macchina che sta accesa in ufficio e serve raggiungibile è la scelta giusta; su una che si vuole poter sospendere a mano, si preferisce la sola disattivazione della sospensione automatica.

Il secondo modo è il Wake-on-LAN, che permette di risvegliare la macchina dalla rete invece di andare a premere un tasto. Va abilitato sia nel firmware, come indicato in fase 3, sia nell'interfaccia di rete.

```bash
sudo apt install ethtool -y
ip -o link show
sudo ethtool enp3s0 | grep -i wake
sudo ethtool -s enp3s0 wol g
```

Il nome dell'interfaccia va letto dal secondo comando e non assunto. L'impostazione di `ethtool` non è persistente al riavvio e va resa permanente con un servizio o una regola di rete, secondo quanto la 26.04 prevede.

Dalla postazione Windows il risveglio si invia poi con un pacchetto magico all'indirizzo MAC della macchina, che è `2c:4d:54:53:a4:fb`, rilevato dalla tabella ARP durante questa sessione.

### 10.3 Chiave SSH per l'accesso senza password

Durante questa sessione la macchina, una volta sveglia, ha risposto sulla porta 22 con `Permission denied (publickey,password)`. Il significato è preciso e vale saperlo leggere: il servizio SSH è attivo e raggiungibile, l'utente esiste, e l'autenticazione è fallita. Nessuna delle due chiavi presenti sulla postazione era autorizzata sulla macchina, e la diagnostica verbosa ha mostrato perché: non esiste una voce in `~/.ssh/config` per questo host, quindi `ssh` prova solo i nomi di chiave predefiniti, che su quella postazione non esistono, dato che le chiavi si chiamano `id_ed25519_personal` e `id_ed25519_corp`.

La soluzione pulita è una chiave dedicata a questo host, separata da quelle usate per GitHub, con una voce di configurazione che le dia un alias. La separazione non è pedanteria: una chiave per host limita il danno di una chiave compromessa, e l'alias rende il comando corto e ripetibile.

I comandi vanno eseguiti dalla postazione Windows, e quello che installa la chiave chiede una volta la password dell'utente sulla macchina.

Va premesso un avvertimento che nasce da un errore realmente commesso in questa documentazione, registrato come MS-027. Il comando `ssh-copy-id` non è un eseguibile: è uno script di shell POSIX, distribuito con OpenSSH sui sistemi Unix e presente su Windows dentro Git Bash, ma non fra i comandi che il client OpenSSH di Windows installa. In PowerShell non esiste, e una prima versione di questa pagina lo proponeva lì dentro, producendo `Termine 'ssh-copy-id' non riconosciuto come nome di cmdlet`. La regola generale che ne discende: quando si forniscono due blocchi equivalenti per due shell, l'equivalenza va verificata sulla disponibilità dei comandi e non solo sulla loro sintassi, perché il linter dei comandi del progetto controlla la forma delle righe e non l'esistenza dei binari.

La generazione della chiave, che è identica nelle due shell a meno della forma del percorso e dell'escaping della passphrase vuota.

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\id_ed25519_studio" -C "postazione-windows -> ubuntu-studio" -N '""'
```

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_studio -C "postazione-windows -> ubuntu-studio" -N ""
```

Se la chiave esiste già il comando chiede di sovrascriverla, e la risposta è no: si passa direttamente al passo successivo, perché una chiave rigenerata invaliderebbe quella eventualmente già installata sulla macchina.

L'installazione della chiave sulla macchina. In PowerShell si fa a mano ciò che `ssh-copy-id` automatizza, cioè si legge la chiave pubblica e la si accoda al file delle chiavi autorizzate, creando la cartella con i permessi che il servizio SSH pretende. In Git Bash si usa `ssh-copy-id`, che esiste.

```powershell
type "$env:USERPROFILE\.ssh\id_ed25519_studio.pub" | ssh alesop95@192.168.10.204 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_studio.pub alesop95@192.168.10.204
```

I permessi non sono un dettaglio estetico: il servizio SSH rifiuta di usare un file di chiavi autorizzate accessibile ad altri utenti, e senza `chmod` l'autenticazione continuerebbe a fallire senza spiegare perché. È il motivo per cui il comando in una riga li imposta esplicitamente.

La verifica che l'installazione sia riuscita, da eseguire subito e prima di modificare la configurazione del servizio.

```bash
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519_studio alesop95@192.168.10.204 "echo CONNESSO; hostname"
```

Poi si aggiunge a `~/.ssh/config` della postazione una voce come la seguente, che dà l'alias `studio` e fissa quale chiave usare.

```
Host studio
    HostName 192.168.10.204
    User alesop95
    IdentityFile ~/.ssh/id_ed25519_studio
    IdentitiesOnly yes
```

Da quel momento la connessione è `ssh studio`, e gli strumenti di trasferimento si possono invocare con `STUDIO_HOST=studio`.

Sulla macchina, per igiene, conviene poi disattivare l'autenticazione per password una volta che la chiave funziona, così che l'unico modo di entrare sia la chiave.

```bash
sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sshd -t
sudo systemctl restart ssh
```

Il comando intermedio verifica la sintassi della configurazione prima del riavvio del servizio, e va eseguito sempre: riavviare `sshd` con una configurazione non valida su una macchina a cui si accede solo in rete è il modo classico di chiudersi fuori.

### 10.4 Indirizzo stabile

L'indirizzo `192.168.10.204` arriva da DHCP e potrebbe cambiare. Su una macchina che si raggiunge per nome dagli strumenti conviene fissarlo, con una prenotazione sul router basata sull'indirizzo MAC, che è la strada preferibile perché resta gestita in un punto solo, oppure con un indirizzo statico configurato sulla macchina.

## Fase 11: verifica finale e chiusura

Il controllo di uscita dell'intera procedura è la ripetizione della fotografia della fase 0 sul sistema nuovo, e il confronto delle due.

```bash
mkdir -p ~/fotografia-post-reinstall
cd ~/fotografia-post-reinstall
lsb_release -a > 01-sistema.txt 2>&1
cat /etc/update-manager/release-upgrades > 02-release-upgrades.txt 2>&1
lsblk -f > 04-dischi.txt 2>&1
cat /etc/fstab > 06-fstab.txt 2>&1
aplay -l > 07-audio-out.txt 2>&1
arecord -l > 08-audio-in.txt 2>&1
apt-mark showmanual > 11-pacchetti-manuali.txt 2>&1
wine --version > 12-wine.txt 2>&1
find ~/wineprefixes -maxdepth 2 -name "system.reg" -printf "%h\n" > 13-prefix-wine.txt 2>&1
uname -a > 14-kernel.txt 2>&1
dpkg --print-foreign-architectures > 15-architetture.txt 2>&1
```

Il confronto atteso, voce per voce. Il sistema passa da 25.04 a 26.04. La direttiva di aggiornamento resta `Prompt=lts`, ma ora su una base che ne trae vantaggio. Le partizioni sono le stesse quattro, con `/home` invariata negli identificativi univoci. I dispositivi audio sono visti come prima. I prefix Wine sono quattro e nuovi, invece del prefix di default sedimentato. L'elenco delle architetture straniere è vuoto, mentre prima conteneva `i386`. I pacchetti installati manualmente sono meno di prima, perché l'ambiente è ricostruito con l'essenziale.

E il controllo che conta più di tutti: Akabak si apre, accetta lo stesso Release Code, e VACS non ne chiede un secondo.

## Se qualcosa va storto

Tre scenari e la risposta a ciascuno.

L'installazione non parte o si interrompe. Nulla è perduto perché `/home` non è stata toccata e la copia della fase 1.3 esiste. Si riprova, eventualmente riscrivendo la chiavetta dopo aver riverificato la somma di controllo dell'immagine.

Il sistema si installa ma non avvia. È tipicamente un problema di avvio UEFI. Si riavvia dalla chiavetta in modalità live e si ripara il caricatore; la partizione EFI esistente e non formattata è un vantaggio in questo scenario, perché contiene ancora la voce di avvio precedente.

`/home` è stata formattata per errore. È l'unico scenario davvero grave, e l'unica risposta è il ripristino dalla copia della fase 1.3. È anche la ragione per cui quella fase non è opzionale.
