<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Setup della macchina, settembre 2026: cronologia e razionale

> Racconto continuo di come la macchina Ubuntu Studio è stata azzerata e ricostruita fra il 4 e il 9 settembre 2026, con i comandi eseguiti e la ragione di ciascuno. Esiste perché gli altri tre documenti dello stesso perimetro rispondono a domande diverse: [installazione-pulita-26-04.md](installazione-pulita-26-04.md) dice come si fa, [OPERATIONS-LOG.md](../OPERATIONS-LOG.md) dice che cosa è stato fatto un intervento per volta con la sua verifica, e [scheda-reinstallazione.md](scheda-reinstallazione.md) è il foglio che si porta accanto alla macchina. Nessuno dei tre racconta la sequenza nel tempo con il perché di ogni passo, e senza quel racconto la ricostruzione futura sarebbe una lista di comandi senza le decisioni che li hanno prodotti.
>
> Tutto ciò che segue è osservato, non ricostruito a memoria, e la distinzione è dichiarata dove serve: gli orari della giornata dell'installazione vengono dai log dell'installatore conservati in `/var/log/installer/`, i valori di stato da comandi eseguiti sulla macchina, e le poche affermazioni non verificate sono marcate come tali.

## Da dove si partiva, e perché non si è aggiornato

La macchina è un desktop del 2016, ASUSTeK H170-PRO con BIOS[^1] 3805, Intel i7-6700 e 16 GB di DDR4, con un NVMe[^2] da 500 GB. Il sistema installato era Ubuntu Studio 25.04, cioè un rilascio intermedio uscito dal supporto, e l'ultima operazione registrata nella cronologia di `apt` risaliva al 13 agosto 2025.

La prima ipotesi di lavoro era che un aggiornamento fosse stato tentato e fosse fallito, e su quella ipotesi era stata scritta una diagnosi elaborata. La fase 0 della procedura l'ha smentita: `/var/log/dist-upgrade/` era vuota, quindi l'aggiornamento non era mai stato tentato e non esisteva alcun blocco da diagnosticare. La lezione di metodo è registrata in MS-029 e vale oltre questo progetto, perché la premessa falsa non stava in un dato ma nella domanda: si era assunto che un tentativo ci fosse stato.

La scelta fra aggiornamento in posto e installazione pulita è stata quindi ripresa da zero e decisa per installazione pulita, con ADR-011 che rivede ADR-006 e ADR-013 che la riconferma. Le ragioni che hanno tenuto sono tre. L'ambiente aveva accumulato attrito reale, cioè due repository WineHQ attivi contemporaneamente per due rilasci diversi di Ubuntu, `wine-stable 3.0.1` del 2018 accanto a `wine 9.0`, una sorgente `file:/cdrom/` residua e un solo prefix Wine condiviso da tutto. Il salto da un rilascio intermedio fuori supporto a una LTS attraversa comunque una ricostruzione dell'ambiente. E il rischio dell'installazione pulita era circoscrivibile a un solo punto, la selezione delle partizioni, contro un rischio diffuso e non circoscrivibile nell'aggiornamento di un sistema in quello stato.

Da questa decisione discende la forma di tutto il resto del setup, ed è il punto in cui la ratio va capita invece che eseguita: *si azzera la radice e si conserva `/home`*. Non è una via di mezzo per risparmiare tempo, è la separazione che rende il rischio circoscrivibile. La radice contiene solo sistema e programmi, cioè cose riproducibili da un installatore e da `apt`; `/home` contiene i progetti, i materiali trasferiti e i prefix Wine, cioè cose che nessun comando ricostruisce. Tutte le cautele della procedura si concentrano perciò su una singola casella di spunta.

## Le decisioni che hanno dato forma al setup

Cinque decisioni registrate hanno determinato la configurazione finale, e conviene averle davanti prima della cronologia, perché i comandi eseguiti sono la loro conseguenza.

Akabak e VACS sono programmi a *32 bit*, non a 64, e questo rovescia una prescrizione che tre decisioni precedenti si erano passate l'una all'altra senza verificarla. È ADR-016, e la prova è stata `file` sull'eseguibile installato, che risponde `PE32 executable, Intel 80386`, con il prefix funzionante che dichiara `#arch=win32` e non contiene alcun `syswow64`. Ne segue che l'architettura `i386` va *dichiarata* sul sistema e non evitata come residuo, e che va dichiarata prima di installare Wine: farlo dopo non tira dentro i pacchetti a 32 bit, e il programma non parte.

Akabak e VACS non richiedono alcuna dipendenza aggiuntiva, cioè né .NET, né font Microsoft, né runtime Visual C++. Il prefix che funziona non ne ha nessuno, e la lista di dipendenze del documento sorgente descriveva i tentativi fatti durante un troubleshooting passato, non ciò che serviva davvero.

La licenza di Akabak è legata alla macchina e non al prefix né al sistema, quindi sopravvive a una reinstallazione. Verificato prima di azzerare, in MS-052: il Machine Identifier mostrato dal programma coincideva con quello a cui il Release Code è legato, e il programma dichiarava il codice valido. Il codice non compare in nessun file versionato e vive in `_notes/licenze-akabak-riservato.md`, che il `.gitignore` esclude; per ADR-017 non compare nemmeno nella scheda stampata, a meno che non si chieda espressamente.

La copia di sicurezza di `/home` va su una macchina diversa e in un archivio `tar` che conserva proprietari e permessi. È ADR-015, e la ragione è che una copia sullo stesso disco non protegge dall'errore che si vuole prevenire, cioè la formattazione di quel disco.

Il lavoro privilegiato resta manuale. `sudo` chiede la password per scelta dell'utente e non esistono regole `sudoers` di comodo, quindi ogni comando privilegiato viene preparato e poi eseguito a mano da chi ha la password. È una scelta che rallenta e che è stata mantenuta di proposito su una macchina che ospita l'unica copia di materiale di lavoro.

## Cronologia

### 4 settembre: impianto documentale, e la macchina irraggiungibile

Il progetto esisteva come impalcatura senza documentazione tecnica propria, e il materiale viveva come file di testo e un `.docx` alla radice. La giornata è servita a convertire il documento sorgente in un albero `docs/` navigabile con copertura verificata, ad allineare il progetto al template e a mettere in ordine il version control.

La macchina risultava di stato ignoto e non rispondeva. La prima diagnosi la dava per spenta o su un altro segmento di rete, e la lettura corretta l'ha fornita l'utente: si sospende da sola, e una macchina sospesa non risponde nemmeno alle richieste ARP[^3], quindi scompare dalla rete invece di rispondere male. Risvegliata, ha risposto al ping con TTL 64 e ha accettato la connessione sulla porta 22 rifiutando l'autenticazione. Questo fatto tornerà due volte, ed è la ragione per cui il 9 settembre la sospensione è stata disattivata come primo presidio e non come rifinitura.

### 7 settembre: accesso aperto, fase 0 eseguita, materiali trasferiti

L'utente ha installato una chiave SSH dedicata, sbloccando l'accesso non interattivo che era il vincolo delle due sessioni precedenti. Nel fornire i comandi per farlo ho commesso un errore che vale registrare qui perché ha una regola generale dietro: `ssh-copy-id` non esiste in PowerShell, essendo uno script POSIX presente su Windows solo dentro Git Bash. L'assunzione sbagliata era che due blocchi per due shell differissero solo nella sintassi, mentre qui differiva la disponibilità del comando. È MS-027.

Con l'accesso aperto è stata eseguita la fotografia della macchina, che è il documento di riferimento sullo stato di partenza e sta in [fotografia-macchina-2026-09-07.md](fotografia-macchina-2026-09-07.md). Tre suoi esiti hanno cambiato il piano. La catena audio a bassa latenza era già configurata bene ma il controllo prescritto, cioè il nome del kernel, avrebbe dato un falso negativo, perché il kernel è generico e le proprietà arrivano dai parametri di avvio. La partizione EFI era di 1,1 GB e non dei circa 100 MB che il documento sorgente dichiarava. E l'ispezione degli eseguibili ha prodotto ADR-016 sui 32 bit.

Nella stessa giornata sono stati trasferiti i materiali sotto `~/electroacoustics`, 728 MB e 281 file, ed è stata fatta la copia di sicurezza di `/home`, 4,4 GB, verificata per numero di file e permessi. I due numeri del trasferimento diventeranno il criterio con cui il 9 settembre si è verificato che `/home` fosse sopravvissuto.

### 8 settembre, mattina e primo pomeriggio: il supporto e la scheda

L'immagine `ubuntustudio-26.04.1-desktop-amd64.iso` è stata scaricata e verificata due volte, per somma di controllo e per firma del file delle somme, e accanto a essa è stato messo Rufus 4.15 portabile verificato per firma Authenticode. Il point release 26.04.1 e non la 26.04 iniziale, perché è quello che porta le correzioni accumulate.

La chiavetta è stata scritta in modalità DD, in otto minuti e sette secondi. La modalità DD è stata scelta perché produce un clone byte per byte, e la struttura risultante lo conferma a occhio: tabella GPT[^4] con tre partizioni, il volume principale di circa 6792 MB più una partizione di sistema EFI da 5 MB e una ausiliaria da 0,3 MB, che sono le partizioni dell'immagine ibrida e non una struttura costruita dallo strumento di scrittura, che in modalità ISO ne avrebbe creata una sola.

Su questa fase è stato commesso e ritirato un errore di giudizio che vale più del suo esito tecnico, e sta in MS-069. Avevo prescritto di verificare la chiavetta confrontando l'impronta dei primi byte del dispositivo grezzo con quella dell'immagine, e il confronto ha dato impronte diverse su una chiavetta perfettamente valida. Il presupposto è falso su Windows per tre ragioni che sono tutte scritture legittime del sistema: una tabella GPT dimensionata sull'immagine viene riparata quando il supporto è più grande, la zona da cui la copia di sicurezza della tabella è stata spostata cambia a sua volta, e le partizioni che il sistema monta con lettera propria ricevono le cartelle di servizio che Windows crea al primo accesso. La regola che ne discende: un confronto byte per byte fra immagine e supporto ha senso solo su un sistema che non monta i volumi da sé e non ripara le tabelle delle partizioni.

Nel primo pomeriggio è stata costruita la scheda da stampare, e il suo `.docx` alle 15:10. Alle 15:12 la fase 4.2 della procedura ha guadagnato l'avvertenza sulla casella di formattazione che si attiva da sé, e alle 15:14 la scheda sorgente. Poi la sessione è stata interrotta da un crash, portando via il codice che aveva generato il `.docx`: la carta era rimasta indietro di due minuti rispetto alla documentazione e non era riproducibile. La riparazione, il 9 settembre, è stato lo strumento `tools/make-scheda-docx.py`, ed è MS-070.

### 8 settembre, 17:28 - 17:50: l'installazione, secondo i suoi log

Gli orari che seguono vengono dai log conservati in `/var/log/installer/` sul sistema installato. Una avvertenza sulla loro lettura, perché altrimenti i numeri sembrano incoerenti: i timestamp dentro i file di log sono in UTC, mentre le date di modifica dei file sono in ora locale CEST, cioè due ore avanti. Le ore qui sono locali.

Il supporto è stato verificato, e non da un'azione dell'operatore. La voce `Check disc for defects` che la scheda prescriveva *non esisteva* nel menu di avvio, e la sua assenza non è un difetto: sulle immagini recenti il controllo di integrità è automatico. L'esito è in `/var/log/installer/casper-md5check.json` e vale `{"checksum_missmatch": [], "result": "pass"}`, cioè supporto integro, nessuna somma discordante. La prescrizione era quindi sbagliata due volte, perché indicava un comando inesistente per una verifica che avviene comunque, e la sua forma corretta non è un'azione prima ma una lettura dopo.

Lo schema di partizionamento applicato è quello deciso, con le dimensioni come l'installatore le mostra, cioè in GB decimali. `nvme0n1p1` da 1,13 GB riusata come `/boot/efi`[^5] senza formattare, perché formattarla non serve e sarebbe un rischio inutile. `nvme0n1p2` da 80,00 GB formattata `ext4` e montata su `/`, unica partizione azzerata. `nvme0n1p3` da 16,00 GB come swap. `nvme0n1p4` da 402,98 GB montata su `/home` *senza formattare*.

Sul passo irreversibile la trappola concreta si è manifestata come previsto e va nominata perché non è la casella in sé: nell'installatore la casella di formattazione *si attiva da sola* quando si seleziona un filesystem nel menu della riga. Su `/home` il menu del filesystem non si tocca affatto, si imposta soltanto il punto di montaggio. Il controllo che decide è la schermata di riepilogo, dove la parola `Formatted` deve comparire soltanto per `nvme0n1p2`: nel riepilogo reale diceva `Formatted as ext4 used for /` su `nvme0n1p2`, `Used for /boot/efi` su `nvme0n1p1`, `Used for /home` su `nvme0n1p4` e `Unchanged` su `nvme0n1p3` e su tutte le partizioni della chiavetta. Fino a quel pulsante nulla era stato scritto sul disco.

Un passo della sequenza si era fermato a metà e ha bloccato il pulsante di avanzamento: `nvme0n1p2` era stata marcata per la formattazione ma non aveva ricevuto il punto di montaggio `/`. L'installatore non consente di proseguire senza una radice, e il pulsante grigio non era un guasto ma un requisito non soddisfatto. È il genere di stallo che si legge male, perché l'attenzione va al pulsante invece che alla colonna vuota.

L'ultima schermata ha creato l'utente con lo stesso nome, `alesop95`. Era una prescrizione della scheda e la sua ragione operativa si è verificata: l'utente nuovo ha ricevuto identificativo numerico `1000`, identico al vecchio, quindi `/home/alesop95` è rimasto di `1000:1000` e non è stato necessario alcun `chown` ricorsivo su 402 GB.

Alle 17:50 i log registrano il completamento. `curtin`, che è il componente che partiziona, copia il sistema e installa il boot loader, chiude con `install-grub: SUCCESS`, `configuring-bootloader: SUCCESS` e infine `curtin: Installation finished.`. Alle 17:50:38 la configurazione finale del sistema riporta successo, con un solo avvertimento, `failed to rmdir /target/cdrom: [Errno 16] Device or resource busy`. Alle 17:50:40 lo stato dell'installatore passa a `DONE`.

### 8 settembre, dalle 17:50 alle 10:12 del giorno dopo: sedici ore di blocco apparente

L'interfaccia è rimasta sulla schermata `Setting up the system` per sedici ore, e l'utente ha riavviato togliendo la chiavetta. Il sistema si è avviato correttamente.

La conclusione, sostenuta dai log e non dedotta dall'esito: *l'installazione era finita alle 17:50:40* e quello che restava appeso era la fase conclusiva, cioè la copia dei log nel sistema installato e la conseguente chiusura. Lo stato dichiarato dall'installatore era `DONE` prima che il blocco cominciasse, quindi la schermata che l'utente guardava non descriveva più il lavoro in corso.

Un limite di questa evidenza va dichiarato perché è strutturale e non un difetto della raccolta. La copia del log che si legge nel sistema installato termina nell'istante in cui viene copiata, dato che sta copiando se stessa: per costruzione non può contenere ciò che è successo dopo. Le ultime righe mostrano l'avvio dell'`rsync` verso `/target/var/log/installer` e la cattura del journal, entrambi conclusi con successo. Restano quindi due anomalie registrate proprio in coda, `/target/cdrom` che risulta occupato al momento di rimuoverlo e un errore di scrittura del rapporto di telemetria, ed entrambe sono compatibili con uno stallo nella chiusura ma *nessuna delle due è dimostrata come causa*.

Ne discende la regola operativa, che è il vero guadagno di questa vicenda. La schermata di avanzamento non è l'autorità sullo stato dell'installazione: l'autorità è lo stato dichiarato dall'installatore, ed è consultabile dal vivo aprendo il registro dall'icona a forma di terminale in basso a destra nella finestra. Sedici ore di attesa sono state il costo di non aver guardato lì. In hindsight il riavvio forzato è stato l'azione corretta, e attendere di più non avrebbe cambiato nulla.

Tre osservazioni indipendenti hanno poi confermato che il riavvio non aveva rotto niente, e nessuna delle tre basterebbe da sola: il gestore dei pacchetti pulito, `fstab` completo con radice, EFI e `/home` tutti per UUID[^6], e i codec proprietari installati, cioè la scelta fatta nell'installatore effettivamente onorata.

### 9 settembre, mattina: verifica dell'integrità e ripristino dell'accesso

La prova che `/home` è sopravvissuto non è il riepilogo dell'installatore, che è una dichiarazione di intenti, ma il contenuto reale del disco: la scrivania si è ripresentata con i propri file e `~/electroacoustics` conta esattamente 281 file per 728 MB, cioè i due numeri registrati al momento del trasferimento. Anche il prefix Wine `~/.wine` è sopravvissuto con il suo `drive_c`.

Il ripristino dell'accesso SSH ha avuto una scorciatoia che vale capire, perché nasce dalla stessa separazione su cui poggia tutto il setup. La radice azzerata ha portato via `openssh-server` e le chiavi d'identità della macchina, ma `authorized_keys` vive in `/home` ed è sopravvissuto, con la data originale del 7 settembre e i permessi `600` su una cartella `700`, cioè già conformi a quanto il servizio pretende. È bastato installare il server.

```bash
sudo apt update && sudo apt install -y openssh-server
```

```bash
id && ls -ln /home/ && ls -la ~/.ssh/ && stat -c '%U %G %a %n' ~/.ssh ~/.ssh/authorized_keys
```

Il secondo comando serve a decidere se procedere, e non è un controllo di forma. Se l'identificativo numerico non fosse `1000`, oppure se `ls -ln` mostrasse un numero invece del nome come proprietario, l'utente nuovo avrebbe ricevuto un identificativo diverso dal vecchio e i permessi andrebbero corretti su tutto `/home` prima di qualunque altra cosa.

```bash
sudo systemctl enable --now ssh && systemctl is-active ssh && ip -br a
```

Dal lato della postazione Windows è servito rimuovere la voce vecchia dall'archivio delle chiavi note, perché l'identità della macchina è cambiata legittimamente con la reinstallazione e un cambio di chiave host è indistinguibile da un attacco per chi legge solo il messaggio d'errore.

```powershell
ssh-keygen -R 192.168.10.204
```

La connessione è stata poi verificata in modo non interattivo, che è la proprietà che serve al progetto: con `BatchMode` attivo nessuna richiesta di password è possibile, quindi una risposta positiva dimostra che l'autenticazione è passata per chiave.

```bash
ssh -o IdentitiesOnly=yes -o BatchMode=yes -i ~/.ssh/id_ed25519_studio alesop95@192.168.10.204 "echo CONNESSO; hostname; uname -r"
```

Per accorciare i comandi successivi è stato aggiunto un alias nel file di configurazione SSH della postazione, che l'agente non può scrivere perché le regole di permesso del progetto negano l'accesso ai percorsi delle chiavi, e che è stato quindi creato a mano dall'utente. Da quel momento la macchina si raggiunge come `ssh studio`.

```
Host studio
    HostName 192.168.10.204
    User alesop95
    IdentityFile ~/.ssh/id_ed25519_studio
    IdentitiesOnly yes
    ServerAliveInterval 30
```

L'ultima direttiva non è cosmetica: manda un segnale ogni trenta secondi, così una sessione aperta non muore in silenzio se la macchina si addormenta ma lo dichiara subito. È un presidio contro il difetto che aveva bloccato due sessioni intere.

### 9 settembre, 13:29 - 13:31: le tre correzioni, con la ragione di ciascuna

Le verifiche da remoto avevano trovato tre difetti, tutti reversibili e nessuno causato dal blocco. Sono stati corretti in una sola tornata, con la parte in sola lettura eseguita per prima di proposito, così che un eventuale problema nelle correzioni non portasse via le prove.

La raccolta diagnostica ha copiato in `~/diag/` i log che richiedono privilegi, in modo che fossero poi leggibili da remoto senza elevazione. La redirezione è eseguita dalla shell dell'utente e non da `sudo`, quindi i file risultanti appartengono all'utente: è la ragione per cui questa forma funziona dove un `sudo cp` avrebbe prodotto file di `root`.

```bash
sudo tail -n 300 /var/log/installer/subiquity-server-debug.log > ~/diag/subiquity-tail.txt && wc -l ~/diag/subiquity-tail.txt
```

Primo difetto, i limiti realtime. I file in `/etc/security/limits.d/` concedono `rtprio 95` e `memlock unlimited` ai gruppi `audio` e `pipewire`, e Ubuntu Studio li installa da sé in `30-ubuntustudio-audio.conf`. L'utente creato dall'installatore però non apparteneva a nessuno dei due, quindi `ulimit -r` rispondeva `0` e la memoria bloccabile era 8192 kB: i limiti erano scritti e non in vigore.

```bash
sudo usermod -aG audio,pipewire alesop95 && id -nG alesop95
```

La trappola di questo difetto merita di stare a verbale perché è generale. La verifica prescritta dalla procedura era leggere i valori nei file di configurazione, e quei valori erano giusti: chi si fermasse lì concluderebbe che la catena è configurata. Solo il confronto fra ciò che il file concede e ciò che `ulimit` riporta davvero mostra che il permesso non arriva all'utente. È la stessa differenza fra una regola scritta e una regola in vigore che aveva già prodotto un falso negativo sul nome del kernel.

```bash
sudo -u alesop95 -i bash -c 'id -nG; echo "rtprio: $(ulimit -r)"; echo "memlock: $(ulimit -l)"'
```

Questo comando esiste per una ragione precisa: l'appartenenza a un gruppo si applica al login e non subito, quindi la sessione in corso non la vede. Avviare una sessione di login nuova permette di verificare senza riavviare. L'esito è stato `rtprio: 95` e `memlock: unlimited`, quindi i limiti risultano in vigore, ed è un esito migliore dell'atteso perché non tutte le configurazioni applicano i limiti a una sessione aperta in questo modo.

Secondo difetto, la swap. L'installatore ha ignorato la partizione `nvme0n1p3`, che esiste, è formattata e porta un UUID valido, e ha creato al suo posto un file `/swap.img` da 4 GB sulla radice: è la spiegazione della riga `Unchanged` accanto a quella partizione nella schermata di riepilogo, che al momento era sembrata innocua. Le conseguenze erano quasi 15 GB di disco inutilizzati e una swap troppo piccola per l'ibernazione, che con 16 GB di RAM ne richiede altrettanti, e che era la ragione per cui quella partizione era stata dimensionata pari alla memoria.

```bash
sudo cp /etc/fstab /etc/fstab.bak-$(date +%F-%H%M) && ls -la /etc/fstab.bak-*
```

```bash
sudo sed -i 's|^/swap.img.*|UUID=50a4c66e-60a4-41a5-8587-e6768b87b7ac none swap sw,nofail 0 0|' /etc/fstab && grep -n swap /etc/fstab
```

Due scelte in questo comando vanno spiegate. La swap è indicata per UUID e non per nome di dispositivo, perché un nome come `/dev/nvme0n1p3` dipende dall'ordine di enumerazione e un UUID no, che è la stessa ragione per cui l'installatore scrive tutte le altre voci di `fstab` in quella forma. E le opzioni portano `nofail` come rete di sicurezza: un errore in quella riga senza `nofail` costerebbe un avvio bloccato in attesa di un dispositivo inesistente, mentre con `nofail` il sistema si avvia senza swap e lo si scopre a freddo. Era una precauzione necessaria perché la modifica è stata applicata poco prima di lasciare la macchina non presidiata.

```bash
sudo swapoff /swap.img && sudo swapon -a && swapon --show && free -h | head -3
```

```bash
swapon --show | grep -q /dev/nvme0n1p3 && sudo rm -f /swap.img && echo "FILE RIMOSSO, swap sulla partizione" || echo "NON RIMOSSO: la partizione non risulta attiva, fermati e dimmelo"
```

L'ordine dei due comandi è il presidio. La cancellazione del file è condizionata al fatto che la partizione risulti effettivamente attiva, verificato sull'output di `swapon --show` e non sull'assenza di errori: cancellare prima avrebbe potuto lasciare il sistema senza alcuna swap. L'esito è stato `/dev/nvme0n1p3`, tipo `partition`, 14,9 GiB, e il file è stato rimosso.

Terzo difetto, la sospensione automatica, tornata attiva perché il sistema è nuovo. È il difetto che aveva reso la macchina invisibile alla rete in due sessioni precedenti, e la sua conseguenza non è un fastidio ma la perdita dell'accesso remoto mentre nessuno è davanti alla macchina.

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

Questa forma è più radicale dell'impostazione grafica, perché impedisce anche la sospensione manuale, e la scelta è deliberata su una macchina che deve restare raggiungibile e che non ha ragioni di risparmio energetico da rispettare. È reversibile con `unmask` sugli stessi quattro bersagli. Il comando di verifica risponde `masked` quattro volte e con codice di uscita diverso da zero, che è la risposta giusta e non un errore.

## Stato verificato al termine del setup

Tutti i valori qui sotto sono stati letti sulla macchina il 9 settembre 2026, non desunti.

| Voce | Valore osservato |
|---|---|
| Sistema | `Ubuntu 26.04.1 LTS`, supporto `Resolute Raccoon` build 20260826 |
| Kernel | `7.0.0-31-generic`, generico e non lowlatency |
| Desktop | Plasma 6.6.6, Frameworks 6.24.0, Qt 6.10.2, su Wayland |
| Parametri di avvio | `preempt=full threadirqs rcu_nocbs=all`, da `/etc/default/grub.d/ubuntustudio.cfg` |
| Limiti realtime | `rtprio 95`, `memlock unlimited`, verificati con `ulimit` in sessione di login |
| Gruppi utente | `audio` e `pipewire` presenti, oltre a `sudo`, `adm`, `plugdev`, `lpadmin`, `lxd` |
| Radice | `nvme0n1p2`, 73 GiB, 27 per cento usato |
| `/home` | `nvme0n1p4`, 369 GiB, 4,5 GiB usati, *conservato* |
| EFI | `nvme0n1p1`, 1,1 GiB, riusata senza formattare |
| Swap | `nvme0n1p3`, 14,9 GiB, partizione, per UUID con `nofail` |
| Materiali | `~/electroacoustics`, 281 file, 728 MB |
| Prefix Wine vecchio | `~/.wine` presente con `drive_c`, non ancora usato |
| Wine | non installato, architettura `i386` non dichiarata |
| Politica di aggiornamento | `Prompt=lts`, già impostata di serie |
| Codec proprietari | `ubuntu-restricted-addons` in stato `ii` |
| Accesso remoto | `ssh studio` per chiave, non interattivo, sospensione disattivata |
| Sospensione | quattro target `masked` |

Sulle unità di misura vale un'avvertenza, perché la stessa partizione compare in questo documento con tre numeri diversi e la discrepanza non è un errore. L'installatore misura in GB decimali e mostra `402,98 GB`; `lsblk` misura in GiB e mostra `375,3G`; `df` misura il filesystem in GiB al netto dello spazio riservato e mostra `369G` di dimensione con `346G` disponibili. Le partizioni si identificano dal nome e dal punto di montaggio, non dalla dimensione, e confrontare grandezze in unità diverse davanti al passo irreversibile è precisamente il dubbio che non deve nascere.

## Che cosa resta, e perché in quest'ordine

Restano le fasi da 6 a 9 della procedura, e il loro ordine non è arbitrario.

La verifica della catena audio è incompleta, ma non per la ragione che era stata scritta. Non manca una interfaccia staccata da collegare: manca una decisione. Al 2026-09-09 la macchina ha la sola scheda integrata `ALC887-VD` e nessuna interfaccia esterna, e l'utente ha chiarito che la Focusrite Scarlett 2i2 che i documenti dichiaravano come hardware della macchina esiste ma non è impiegata in questo progetto. Il requisito della fase 8 resta, cioè un ingresso microfonico con alimentazione phantom per un microfono XLR calibrato, e la sua soddisfazione è aperta con due vincoli fissati dall'utente: dispositivo di classe audio che il kernel veda senza driver proprietari, e capacità di reggere anche le misure e non solo la registrazione. Il record del ritiro è in MS-079.

La ricostruzione dell'ambiente Wine ha un ordine obbligato per ADR-016, cioè prima `dpkg --add-architecture i386`, poi l'installazione di Wine, e solo dopo la creazione o l'apertura di un prefix a 32 bit. Invertire i primi due passi produce un ambiente in cui Akabak non parte, ed è l'errore che tre decisioni precedenti avrebbero fatto commettere.

Su questa fase esiste una possibilità che vale mettere alla prova prima di ricostruire da zero, e va dichiarata come ipotesi e non come piano: il prefix `~/.wine` è sopravvissuto con AKABAK installato dentro e la sua attivazione nel registro, e il Machine Identifier della macchina non è cambiato. Se quel prefix risultasse utilizzabile dopo la migrazione automatica che Wine applica ai prefix creati da versioni molto precedenti, salterebbe sia la reinstallazione sia la riattivazione. Se la migrazione lo rompesse, il piano resta quello scritto nella fase 7 e nulla è perduto, perché gli installer sono in `~/electroacoustics/installers/` e il codice di licenza è nel file riservato.

Prima di cancellare qualunque copia esterna di materiale personale va compiuta la verifica di PA-010. I due controlli fatti finora, cioè la scrivania che si ripresenta e i numeri di `~/electroacoustics`, coprono un perimetro molto più stretto di quello che serve, e il confronto va fatto per impronta del contenuto e non per dimensione occupata, dato che il tranello del conteggio dello spazio occupato è già costato una inferenza sbagliata a questo progetto.

A ricostruzione compiuta e verificata va preso il backup completo della macchina di PA-011, che è cosa diversa dall'archivio di `/home` già esistente: quello contiene i dati e non il sistema, e lo scopo del secondo è rendere ripetibile in poche ore un risultato che è costato più di un giorno.

[^1]: *BIOS*, Basic Input/Output System - il firmware della scheda madre, che su questa macchina espone una interfaccia UEFI, cioè Unified Extensible Firmware Interface, il successore del BIOS classico che avvia il sistema leggendo una partizione dedicata invece di un settore di avvio.

[^2]: *NVMe*, Non-Volatile Memory Express - il protocollo con cui un disco a stato solido dialoga direttamente con il bus PCI Express, da cui il nome di dispositivo `nvme0n1` invece di `sda`.

[^3]: *ARP*, Address Resolution Protocol - il protocollo con cui una macchina sulla rete locale chiede quale indirizzo hardware corrisponde a un indirizzo IP; una macchina sospesa non risponde nemmeno a questa domanda, quindi scompare del tutto invece di risultare irraggiungibile.

[^4]: *GPT*, GUID Partition Table - lo schema di partizionamento dei dischi usato dai sistemi UEFI, che conserva una copia di sicurezza della propria tabella alla fine del supporto; è questa copia che, su un supporto più grande dell'immagine scritta, il sistema sposta e riscrive.

[^5]: *EFI*, Extensible Firmware Interface - qui indica la partizione di sistema EFI, formattata FAT32, dove il firmware cerca i caricatori di avvio dei sistemi installati; è condivisa fra tutti i sistemi del disco, ed è la ragione per cui si riusa invece di formattarla.

[^6]: *UUID*, Universally Unique Identifier - l'identificativo che un filesystem porta al proprio interno, indipendente dal nome di dispositivo che il kernel assegna in base all'ordine di enumerazione; è la forma corretta per riferirsi a una partizione in `/etc/fstab`.
