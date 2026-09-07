<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Installazione pulita di Ubuntu Studio 26.04 LTS, passo per passo

> Procedura operativa completa. Ogni passo dichiara che cosa si fa, perché si fa, il comando o l'azione esatta, e come si verifica che sia riuscito. La procedura è divisa in undici fasi e va eseguita nell'ordine: le fasi 0 e 1 raccolgono e mettono in salvo informazioni che dopo la fase 4 non sarebbero più recuperabili, quindi saltarle non è una scorciatoia ma una perdita di dati.
>
> Stato: procedura scritta e non ancora eseguita. Le fasi da 0 in avanti verranno spuntate nel registro dei microstep man mano che si compiono, con l'esito reale accanto a quello atteso.

## Premessa: perché una installazione e non un aggiornamento

La macchina è su Ubuntu Studio 25.04, una versione intermedia fuori supporto, e la LTS successiva non è raggiungibile con un salto singolo. La ricostruzione della causa sta in `docs/10-ambiente/ubuntu-lts-upgrade.md`, ed è una ipotesi che la fase 0 di questa procedura conferma o smentisce prima di procedere.

I quattro motivi per cui questa strada è preferibile all'aggiornamento in posto sono argomentati in quel documento e registrati come ADR-006. In sintesi: è una operazione invece di due aggiornamenti in cascata attraverso archivi storici; coincide con l'obiettivo di un ambiente pulito e rimuove la sedimentazione che ha prodotto i guasti su `kernel32.dll`; non mette a rischio la licenza di Akabak, che è legata all'hardware e non all'installazione; e porta su una base supportata per cinque anni.

Il fatto che rende l'operazione a basso rischio è il partizionamento scelto all'installazione originaria, con `/home` su una partizione separata. È la decisione che oggi paga il dividendo più alto, e va trattata con rispetto: l'unico modo di rovinare questa procedura è formattare `/home` per distrazione nella fase 4.

## Le regole non negoziabili

Tre, e valgono per tutta la procedura.

Non si formatta `/home`. In fase 4 la partizione `/home` va montata senza spuntare l'opzione di formattazione. È l'unico passo irreversibile dell'intera procedura.

Non si parte senza aver completato la fase 1. Il trasferimento dei materiali e la copia di sicurezza vanno chiusi e verificati prima di toccare il disco, non dopo.

Non si dà per verificato ciò che non si è letto. Ogni fase ha un controllo di uscita: se il controllo non dà l'esito atteso, si ferma e si capisce, non si prosegue sperando.

## Fase 0: fotografia completa della macchina attuale

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

Esito atteso se la ricostruzione è corretta: la 25.04 con nome in codice `plucky`, la direttiva `Prompt=lts`, sorgenti che puntano ancora ad `archive.ubuntu.com`, l'architettura `i386` fra quelle straniere, e il messaggio *No new release found* dall'ultimo comando. Se invece l'ultimo comando propone la 25.10, il blocco è altrove e la diagnosi va rifatta sui messaggi reali prima di proseguire.

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

L'SSD era dato al 91 per cento di vita residua da una scansione fatta su Windows nel periodo dell'installazione originaria. Vale rileggerlo ora, prima di scriverci sopra un sistema nuovo: se il valore è crollato, la decisione da prendere non è più fra installazione e aggiornamento ma fra installazione e sostituzione del disco. Se `smartctl` non è presente si installa con `sudo apt install smartmontools`, ma su una 25.04 fuori supporto l'installazione da rete potrebbe non funzionare: in quel caso il controllo si rimanda al primo avvio del sistema nuovo, dove è comunque utile.

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

Registra quale server audio è effettivamente in uso, quali dispositivi sono visti, se la Scarlett 2i2 è riconosciuta e con quale nome, e quali limiti di priorità in tempo reale sono configurati per il gruppo audio. È la parte della fotografia che serve al progetto gemello di home recording tanto quanto a questo.

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

L'immagine di Ubuntu Studio 26.04 LTS si scarica dal sito ufficiale del progetto. Va verificata l'esistenza e la denominazione esatta della versione dal sito, e non dedotta dal calendario dei rilasci: la cadenza di Ubuntu è regolare e le derivate ufficiali seguono la stessa numerazione, ma la conferma va presa dalla fonte.

Insieme all'immagine si scaricano i file delle somme di controllo e la firma.

### 2.2 Verificare l'immagine prima di scriverla

Questo passo si salta spesso e non va saltato: una immagine corrotta produce una installazione che sembra riuscita e fallisce settimane dopo in modo inspiegabile.

```bash
sha256sum -c SHA256SUMS 2>&1 | grep -i ubuntustudio
```

Controllo di uscita: la riga corrispondente all'immagine scaricata dice `OK`.

### 2.3 Scrivere la chiavetta

Dalla postazione Windows si usa Rufus, come per l'installazione originaria, in modalità di scrittura diretta dell'immagine. Dalla macchina Linux, se ancora funzionante, lo strumento equivalente è il seguente, dove il dispositivo di destinazione va identificato con certezza prima di lanciarlo.

```bash
lsblk -f
sudo dd if=ubuntustudio-26.04-desktop-amd64.iso of=/dev/sdX bs=4M status=progress oflag=sync
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

Si scegli il partizionamento manuale, cioè la voce che l'installatore chiama *Something else* o equivalente. Non si scegli in nessun caso la cancellazione del disco né l'installazione guidata, perché entrambe rifarebbero la tabella delle partizioni e porterebbero via `/home`.

Le quattro partizioni vanno configurate così.

| Partizione | Filesystem | Mount point | Formattare | Nota |
|---|---|---|---|---|
| EFI, circa 100 MB, FAT32 | non cambiare | `/boot/efi` | no | si riusa quella esistente; formattarla non è necessario e sarebbe un rischio inutile |
| root, circa 80 GB, EXT4 | EXT4 | `/` | sì | è la partizione da azzerare, contiene solo sistema e programmi |
| swap, circa 16 GB | swap | nessuno | sì | pari alla RAM, per tenere possibile l'ibernazione |
| home, il resto, EXT4 | EXT4 | `/home` | **no** | qui vivono progetti, materiali trasferiti e prefix Wine |

Il punto di attenzione assoluto è l'ultima riga. La casella di formattazione della partizione `/home` deve restare vuota. Prima di confermare, conviene rileggere la schermata di riepilogo che l'installatore mostra e verificare che fra le operazioni previste non compaia una formattazione di quella partizione.

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

Obiettivo: accertarsi che il kernel a bassa latenza e la Scarlett 2i2 funzionino, prima di costruire l'ambiente Wine sopra.

```bash
uname -r
aplay -l
arecord -l
pactl info
systemctl --user status pipewire pipewire-pulse wireplumber
groups
```

Il primo comando deve mostrare un kernel a bassa latenza. Il modo in cui Ubuntu Studio fornisce quel kernel è cambiato fra i rilasci, quindi va verificato dalla documentazione ufficiale della 26.04 e non assunto: è uno dei punti dichiarati come da verificare in `docs/10-ambiente/ubuntu-lts-upgrade.md`. Se il kernel installato è quello generico, si valuta l'installazione del pacchetto a bassa latenza secondo quanto indica la documentazione del rilascio.

Il comando `groups` deve mostrare l'appartenenza al gruppo `audio`, che è ciò che abilita le priorità in tempo reale. Se manca, si aggiunge e si riavvia la sessione.

```bash
sudo usermod -aG audio alesop95
```

Controllo di uscita: la Scarlett 2i2 compare in ingresso e in uscita, e una riproduzione di prova si sente.

## Fase 7: ricostruzione dell'ambiente Wine

Obiettivo: un ambiente pulito, con un prefix per programma, senza l'architettura a 32 bit e senza la sedimentazione che aveva prodotto i guasti su `kernel32.dll`.

La procedura completa, con il razionale di ogni passo, sta in `docs/10-ambiente/wine-configurazione.md`. Qui la sequenza nell'ordine di questa installazione, con le due differenze deliberate rispetto al passato.

La prima differenza: non si aggiunge l'architettura `i386`. Serviva a WinISD, che per la decisione registrata come ADR-004 non viene installato perché ridondante rispetto a VituixCAD e Akabak. Non aggiungerla elimina un prefix, una architettura da mantenere e una classe di guasti, e rimuove anche uno dei fattori di attrito dei futuri aggiornamenti di rilascio.

La seconda differenza: si decide una sola provenienza dei pacchetti e non si mescola. Il repository ufficiale di WineHQ è preferibile per il supporto a .NET, che è la dipendenza critica di tutti e quattro i programmi.

```bash
sudo apt install --install-recommends wine-stable winetricks
wine --version
which wine
```

Esito atteso: versione non inferiore alla 9.0.

Poi i prefix, uno per programma, tutti a 64 bit, tutti sotto una cartella dedicata così che si vedano insieme e nessuno finisca dentro un altro.

```bash
WINEARCH=win64 WINEPREFIX=~/wineprefixes/akabak64 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/vituixcad64 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/easefocus64 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/arta64 winecfg
```

In ciascuno, dalla scheda delle applicazioni si imposta la versione di Windows su Windows 10, e dalla scheda della grafica si attiva la decorazione delle finestre da parte del window manager.

Poi le dipendenze, nel prefix in cui servono e mai globalmente.

```bash
WINEPREFIX=~/wineprefixes/akabak64 winetricks -q dotnet48 vcrun2019 corefonts
WINEPREFIX=~/wineprefixes/vituixcad64 winetricks -q dotnet48 corefonts
WINEPREFIX=~/wineprefixes/easefocus64 winetricks -q dotnet48 corefonts vcrun2013 vcrun2019
WINEPREFIX=~/wineprefixes/arta64 winetricks -q vcrun2019 corefonts
```

Controllo di uscita della fase 7: i quattro prefix esistono, ciascuno contiene il proprio `system.reg`, e `winecfg` si apre in ciascuno senza errori su `kernel32.dll`. La mappa completa dei prefix, con i programmi e le dipendenze di ciascuno, è in `wine-corredo-progetto-stanza.md`.

```bash
find ~/wineprefixes -maxdepth 2 -name "system.reg" -printf "%h\n"
```

## Fase 8: reinstallazione dei programmi e riattivazione della licenza

### 8.1 Akabak e VACS

Gli installer sono già sulla macchina, portati dalla fase 1.

```bash
cd ~/electroacoustics/installers
WINEPREFIX=~/wineprefixes/akabak64 wine AKABAK_Pro_v324b126.exe
WINEPREFIX=~/wineprefixes/akabak64 wine VACS_64_v213b33.exe
```

I due programmi vanno nello stesso prefix, perché si usano in sequenza e condividono le dipendenze. La variante a 32 bit di VACS resta come riserva e, se servisse, andrebbe in un prefix a 32 bit separato e non in questo.

### 8.2 Inserimento del Release Code

È il momento che chiude il cerchio con la fase 0.6 e che verifica praticamente l'affermazione su cui poggia tutta la decisione: che una licenza legata alla macchina sopravvive alla reinstallazione del sistema.

Si apre Akabak nel prefix corretto, si va nel menu di aiuto alla voce del release code, si controlla che il Machine Identifier mostrato sia lo stesso letto in fase 0.6, e si inserisce il codice permanente. I due valori sono nella scheda riservata sotto `_notes/`, non nel repository.

```bash
WINEPREFIX=~/wineprefixes/akabak64 wine "C:/Program Files/RD Team/AKABAK/Akabak.exe"
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

Un avvertimento sull'uso: ARTA è un programma di misura e vuole accedere alla scheda audio, e sotto Wine quell'accesso passa dal driver audio di Wine verso PipeWire, con latenza e stabilità che non sono quelle di un programma nativo. Per produrre un GLL da misure già acquisite il problema non si pone, perché si lavora su file. Per una misura dal vivo con la Scarlett 2i2 si usa REW.

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

I comandi vanno eseguiti dalla postazione Windows, e il secondo chiede una volta la password dell'utente sulla macchina.

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\id_ed25519_studio" -C "postazione-windows -> ubuntu-studio" -N '""'
ssh-copy-id -i "$env:USERPROFILE\.ssh\id_ed25519_studio.pub" alesop95@192.168.10.204
```

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_studio -C "postazione-windows -> ubuntu-studio" -N ""
ssh-copy-id -i ~/.ssh/id_ed25519_studio.pub alesop95@192.168.10.204
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

Il confronto atteso, voce per voce. Il sistema passa da 25.04 a 26.04. La direttiva di aggiornamento resta `Prompt=lts`, ma ora su una base che ne trae vantaggio. Le partizioni sono le stesse quattro, con `/home` invariata negli identificativi univoci. La Scarlett 2i2 è vista come prima. I prefix Wine sono quattro e nuovi, invece del prefix di default sedimentato. L'elenco delle architetture straniere è vuoto, mentre prima conteneva `i386`. I pacchetti installati manualmente sono meno di prima, perché l'ambiente è ricostruito con l'essenziale.

E il controllo che conta più di tutti: Akabak si apre, accetta lo stesso Release Code, e VACS non ne chiede un secondo.

## Se qualcosa va storto

Tre scenari e la risposta a ciascuno.

L'installazione non parte o si interrompe. Nulla è perduto perché `/home` non è stata toccata e la copia della fase 1.3 esiste. Si riprova, eventualmente riscrivendo la chiavetta dopo aver riverificato la somma di controllo dell'immagine.

Il sistema si installa ma non avvia. È tipicamente un problema di avvio UEFI. Si riavvia dalla chiavetta in modalità live e si ripara il caricatore; la partizione EFI esistente e non formattata è un vantaggio in questo scenario, perché contiene ancora la voce di avvio precedente.

`/home` è stata formattata per errore. È l'unico scenario davvero grave, e l'unica risposta è il ripristino dalla copia della fase 1.3. È anche la ragione per cui quella fase non è opzionale.
