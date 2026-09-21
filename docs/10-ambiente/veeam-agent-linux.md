<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Veeam Agent for Linux: come si allestisce, e i cinque punti dove ci si perde

> Scritta il 2026-09-16 dopo un allestimento che è costato molto più del previsto, e proprio per questo. Non è un manuale del prodotto, che esiste ed è altrove: è la sequenza delle decisioni e delle trappole incontrate su una macchina reale, con il sintomo di ciascuna e la sua causa. Vale per questo progetto e per `home-lab-cybersec-networking`, da cui viene l'esperienza precedente citata più sotto. Il racconto per intervento sta nei microstep da MS-117 a MS-122 di `docs/OPERATIONS-LOG.md`.

## Il punto che decide tutto, e va deciso prima di installare

Veeam Agent for Linux esiste in due pacchetti alternativi che installano lo stesso programma e differiscono in una cosa sola, cioè come ottengono l'istantanea[^1] del volume da copiare. La scelta fra i due non è una preferenza e non si corregge dopo: determina che tipo di backup la macchina potrà fare.

Il pacchetto `veeam` porta con sé un modulo del kernel, che viene compilato sulla macchina al momento dell'installazione tramite DKMS[^2] e ricompilato a ogni aggiornamento del kernel. Con esso il backup a livello di volume è sempre possibile, su qualunque tipo di disco, e produce una immagine da cui si può ripartire da zero. Il prezzo è la dipendenza dal kernel, che è reale e non teorica.

Il pacchetto `veeam-nosnap` non porta alcun modulo. Non rinuncia però all'istantanea: rinuncia a prenderla da sé, e si appoggia a quella di LVM[^3] dove i volumi sono logici. Ne discende la regola che quasi nessuno scrive e che fa perdere un pomeriggio: su una macchina con partizioni semplici, cioè senza LVM, la variante senza modulo non può fare backup a livello di volume, e resta soltanto il livello di file.

| Dischi della macchina | Variante | Livello di volume | Livello di file |
|---|---|---|---|
| partizioni semplici | `veeam` con modulo | sì | sì |
| partizioni semplici | `veeam-nosnap` | no | sì |
| volumi LVM | `veeam` con modulo | sì | sì |
| volumi LVM | `veeam-nosnap` | sì, tramite LVM | sì |

La differenza fra i due livelli non è di grado. Il livello di volume copia il volume come blocchi e permette il ripristino su ferro nudo, cioè avviare un supporto di ripristino e riscrivere il disco fino a riavere la macchina avviabile. Il livello di file copia i file con permessi e proprietà, quindi conserva tutto il contenuto, ma il ripristino passa da una installazione nuova del sistema seguita dal riversamento dei file.

## Le tre verifiche che costano cinque minuti e ne fanno risparmiare molti

La prima è il tipo di volumi, che decide la tabella qui sopra. Il comando dice subito se esiste uno strato LVM fra la partizione e il filesystem.

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
```

La seconda, necessaria solo se serve il livello di volume, è che il modulo sia compilabile. Servono gli header del kernel in uso e `dkms`, e serve soprattutto sapere che il modulo è software di terze parti che insegue il kernel: su una distribuzione appena uscita può non averlo ancora raggiunto. La verifica vera è la compilazione stessa, che avviene durante l'installazione ed è visibile.

```bash
uname -r
```

```bash
dpkg -l | grep linux-headers
```

La terza è lo spazio della destinazione, ricordando che un deposito posto sulla stessa macchina che si salva protegge dagli errori ma non dal guasto del disco, quindi è al più un luogo di transito.

## Installazione, nella sequenza che funziona

Il repository di Veeam non è legato a una versione di Ubuntu: dichiara una sola distribuzione chiamata `stable` con un solo componente chiamato `veeam`, quindi una distribuzione nuova non è di per sé un ostacolo. Lo si aggiunge con un pacchetto che porta anche la chiave di firma.

```bash
curl -sSL -o /tmp/veeam-release-deb.deb https://repository.veeam.com/backup/linux/agent/dpkg/debian/public/pool/veeam/v/veeam-release-deb/veeam-release-deb_1.0.11_amd64.deb
```

```bash
sudo dpkg -i /tmp/veeam-release-deb.deb
```

```bash
sudo apt update
```

Poi si installa la variante scelta, che è `veeam-nosnap` oppure `veeam`, e nel secondo caso va installato anche `dkms`.

```bash
sudo apt install -y veeam-nosnap
```

Una avvertenza sulle dipendenze dichiarate, che su una distribuzione recente sembrano mancanti e non lo sono. Il pacchetto nomina `libfuse2` e `libgcc1`, che su Ubuntu 26.04 non esistono più con quel nome: sono stati sostituiti da `libfuse2t64` e `libgcc-s1`, i quali però dichiarano di fornire i nomi vecchi, quindi `apt` risolve da sé. Leggere quei nomi nell'elenco delle dipendenze e concludere che il prodotto sia incompatibile è un errore facile.

## La prima trappola: il comando che sembra bloccato

Al primo uso di un sottocomando il programma chiede di accettare due contratti, prima quello del prodotto e poi le note sui componenti di terze parti, e attende la parola `yes` per ciascuno. Le accettazioni sono due e non una, il che spiega perché una prima risposta possa sembrare non aver avuto effetto.

La trappola non è la domanda ma il modo in cui la si può nascondere. Se il comando viene lanciato con l'uscita rediretta su un file, come si fa naturalmente per raccogliere un aiuto da leggere dopo, la domanda finisce nel file mentre la risposta è attesa dal terminale: a schermo non compare nulla e il cursore lampeggia. Il programma non è bloccato, sta aspettando.

Ne segue una regola che vale oltre Veeam. Un comando che redirige l'uscita non deve mai essere il primo contatto con un programma che non si conosce. Si esegue prima senza redirezione, si osserva che cosa chiede, e solo dopo lo si automatizza. Il testo dei contratti, se si vuole leggerlo prima, sta in `/usr/share/veeam/EULA` e in `/usr/share/veeam/3rdPartyNotices.txt`, entrambi leggibili senza privilegi.

Va inoltre saputo che tutti gli strumenti dell'agente, cioè `veeamconfig` per la riga di comando e `veeam` per l'interfaccia a caratteri, rifiutano di funzionare da utente non privilegiato. Ogni passo è quindi lavoro con `sudo`.

## Configurazione: il deposito e il lavoro

Il deposito è il posto dove gli archivi vengono scritti, e può essere locale, su NFS o su condivisione SMB.

```bash
sudo veeamconfig repository create --name repo-locale --location /percorso/del/deposito --type local
```

Il lavoro esiste nelle due forme già descritte, e la differenza si vede nel sottocomando.

```bash
sudo veeamconfig job create volumeLevel --name backup-sistema --repoName repo-locale --backupAllSystem --maxPoints 2
```

```bash
sudo veeamconfig job create fileLevel --name backup-file --repoName repo-locale --includeDirs / --maxPoints 2
```

Se la variante installata non può fare il livello di volume, il rifiuto è esplicito e nomina i dispositivi che non sa trattare, ed è il messaggio da riconoscere perché è il punto in cui ci si accorge di aver scelto la variante sbagliata.

```
Item cannot be backed up without veeam snapshot kernel module: /dev/nvme0n1p2, /dev/nvme0n1p4
```

Il lavoro non viene creato, e l'elenco dei lavori resta vuoto: non c'è nulla da correggere, va cambiata la variante oppure il livello.

## La seconda trappola: il modulo che non compila

Passando alla variante con modulo la compilazione può fallire, e il fallimento è visibile e non silenzioso: DKMS riporta un errore e rimanda al proprio registro di compilazione, che sta in `/var/lib/dkms/<modulo>/<versione>/build/make.log`. Quel file va letto, perché distingue due situazioni che richiedono risposte opposte.

Se gli errori riguardano un ambiente incompleto, per esempio header mancanti o un compilatore assente, si installa ciò che manca e si riprova. Se invece nominano funzioni del kernel dichiarate implicitamente, membri di strutture che non esistono, o funzioni chiamate con un numero di argomenti diverso da quello atteso, allora il modulo è scritto per una generazione di kernel precedente e non c'è nulla da aggiustare.

Sul caso osservato il 2026-09-16, con kernel `7.0.0-31-generic` e agente `6.3.2.1405`, entrambi i moduli disponibili hanno fallito in quel secondo modo. Il modulo storico `veeamsnap` chiama `blkdev_get_by_dev`, `blkdev_put` e `bio_set_op_attrs`, tutte rimosse, e passa tre argomenti a `bio_alloc_bioset` dove ora ne servono cinque. Il modulo nuovo `blksnap`, che è la riscrittura per i kernel moderni e si prova come alternativa perché il pacchetto `veeam` lo accetta al posto dell'altro, fallisce a sua volta perché cerca un'intestazione `linux/blk_snap.h` che non esiste, usa un membro `bd_inode` rimosso da `struct block_device` e ridefinisce una funzione che ora il kernel fornisce.

La conclusione, che vale come regola e non come resoconto, è che su una distribuzione molto recente il backup a livello di volume con Veeam può semplicemente non essere disponibile, e che questo si accerta in mezz'ora provando, non si prevede leggendo la matrice di compatibilità del produttore, che elenca le distribuzioni supportate e non quelle appena uscite.

## Come si esce dallo stato rotto

Un modulo che non compila lascia i pacchetti in uno stato intermedio, con l'agente scompattato e non configurato e il modulo in errore, e in quello stato non esiste un agente funzionante. La via di uscita è togliere tutto ciò che riguarda il modulo e reinstallare la variante che non ne ha bisogno.

```bash
sudo apt purge -y blksnap veeam veeamsnap
```

```bash
sudo apt install -y veeam-nosnap
```

Dopo l'installazione la variante senza modulo riabilita e riavvia il proprio servizio da sé, e lo si verifica prima di proseguire.

```bash
systemctl is-active veeamservice
```

Va notato che il passaggio alla variante con modulo trascina con sé l'intera catena di compilazione, cioè compilatore, librerie di sviluppo e `dkms`, per oltre duecento megabyte. Se il tentativo fallisce e non si intende ripeterlo, quel materiale resta installato e va rimosso deliberatamente, perché altrimenti finisce anche dentro il backup.

## Quando il livello di volume non è disponibile

Restano due vie e conviene conoscerle entrambe prima di scegliere.

La prima è il backup a livello di file di tutto il sistema, che la variante senza modulo esegue su qualunque disco. Conserva i file con permessi e proprietà, li comprime e li cataloga, e si fa interamente da remoto. Non produce una immagine avviabile, quindi il ripristino consiste nel reinstallare il sistema e riversare i file: per un progetto che ha la propria procedura di installazione documentata, il tempo perso è quello della reinstallazione e non quello della ricostruzione dell'ambiente, che è la parte cara.

La seconda è rinunciare a Veeam per l'immagine e usare uno strumento che lavora a macchina spenta, tipicamente Clonezilla avviato da chiavetta, che copia le partizioni così come sono senza bisogno di alcun modulo e senza dipendere dal kernel installato. Dà davvero il ripristino su ferro nudo, e costa la presenza fisica davanti alla macchina e un disco di destinazione collegato a lei.

Le due non si escludono, e la combinazione ragionevole è la prima come copia frequente e da remoto, la seconda come immagine presa nei momenti che contano, per esempio subito dopo aver finito di allestire un ambiente.

## Portabilità: che cosa cambia da una macchina all'altra e che cosa no

Questa pagina serve a rifare l'allestimento su una macchina Linux qualsiasi e non soltanto su quella per cui è stata scritta la prima volta. La sequenza dei comandi non cambia: cambiano alcuni valori e cambiano tre decisioni, e le une e le altre stanno qui, raccolte, così che chi esegue non debba estrarle leggendo la prosa.

I valori si dichiarano una volta sola in testa alla sessione, e tutti i comandi della sequenza li usano per nome. Vanno riletti e adattati, non copiati: il deposito e i percorsi inclusi dipendono da come è partizionata la macchina, che è la prima cosa da guardare.

```bash
MACCHINA=$(hostname)
UTENTE=$(id -un)
DEPOSITO=/home/veeam-repo
NOME_DEPOSITO=repo-locale
NOME_LAVORO=backup-file-sistema
INCLUDI=/,/home
ESCLUDI=/home/veeam-repo,/proc,/sys,/dev,/run,/tmp,/snap,/mnt,/media
PUNTI=2
```

Un avvertimento che non è pedanteria: quelle variabili vivono nella shell in cui sono state scritte e in nessun'altra. Chiudere il terminale o aprirne un secondo le fa sparire, e i comandi che seguono fallirebbero in modi che sembrano scollegati dalla causa. Se l'allestimento dura più di una sessione conviene scrivere il blocco in un file e caricarlo con `source` a ogni riapertura, invece di fidarsi della memoria della shell.

Le tre decisioni che cambiano davvero, in ordine di quanto pesano, sono le seguenti.

La prima è il tipo di volumi, che decide se il backup a livello di volume sia possibile e quindi se si otterrà una immagine avviabile o soltanto i file. Su volumi LVM entrambe le varianti del pacchetto lo consentono; su partizioni semplici lo consente la sola variante con modulo del kernel, e solo se quel modulo compila contro il kernel in uso. La tabella all'inizio di questa pagina riassume i quattro casi, e il comando che li distingue è il primo della fase 0.

La seconda è quali filesystem esistano, che decide il valore di `INCLUDI`. Il parametro `--includeDirs` non dichiara se l'attraversamento si fermi al confine del filesystem, quindi ogni punto di innesto che si vuole salvare va nominato esplicitamente: su una macchina con la sola radice basta `/`, su questa servono `/` e `/home`, su una con `/var` o `/srv` separati vanno aggiunti anche quelli. Il comando della fase 0 che elenca i filesystem montati è la fonte da cui si costruisce quel valore, e va letto invece che ricordato.

La terza è dove mettere il deposito, che decide il valore di `ESCLUDI`. Un deposito collocato dentro un percorso incluso nel backup fa sì che il lavoro copi sé stesso, quindi o sta fuori da ciò che si salva, tipicamente su un disco distinto, oppure sta dentro e va escluso esplicitamente. Qui sta dentro `/home` ed è escluso, scelta obbligata da una macchina con due sole partizioni, e resta comunque un luogo di transito: un deposito sullo stesso disco che si salva protegge dall'errore ma non dal guasto del disco.

Restano invariabili, e vale saperlo perché toglie decisioni inutili: il nome del gruppo `veeam` e i permessi che l'agente impone al proprio binario e al proprio deposito, la richiesta di accettare due contratti al primo uso, il fatto che ogni sottocomando richieda privilegi, e il punto di innesto predefinito `/mnt/backup` per il montaggio di un punto di ripristino.

## La sequenza operativa completa, dal nulla all'archivio riletto

Questa sezione è la procedura replicabile, cioè i soli comandi nell'ordine in cui vanno eseguiti, con una riga che dice che cosa ciascuno accerta. Le spiegazioni stanno nelle sezioni precedenti e nel troubleshooting più sotto: qui non si spiega, si esegue. Si presuppone che il blocco dei valori della sezione precedente sia stato eseguito nella stessa shell.

Una nota sui privilegi, perché cambia quanto la procedura sia comoda e perché la voce B del troubleshooting diceva il contrario fino al 2026-09-21. I comandi che seguono portano `sudo`, ed è la forma che funziona sempre; da una sessione aperta dopo l'installazione del pacchetto, però, `sudo` non serve, perché l'installazione aggiunge l'utente al gruppo `veeam` e il binario appartiene a quel gruppo. La differenza conta nella pratica, perché senza `sudo` l'intera sequenza, compresi l'avvio del lavoro e il montaggio del punto, si esegue da una sessione SSH non interattiva, quindi si automatizza; con `sudo` serve un terminale dove digitare una password.

### Fase 0, accertare i presupposti

Il tipo di volumi decide se il livello di volume sia possibile, e va letto prima di installare qualunque cosa.

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
```

L'elenco dei filesystem montati è la fonte da cui si costruisce il valore di `INCLUDI`, e serve anche a scegliere dove collocare il deposito.

```bash
df -hT -x tmpfs -x devtmpfs
```

Solo se si intende usare la variante con modulo del kernel, i due comandi che dicono se esistano i presupposti per compilarlo.

```bash
uname -r
```

```bash
dpkg -l | grep linux-headers
```

### Fase 1, installare

Il repository di Veeam non è legato a una versione di Ubuntu e si aggiunge con un pacchetto che porta anche la chiave di firma. La versione del pacchetto di repository va verificata sul sito del produttore, perché è l'unico valore di questa sequenza che invecchia da solo.

```bash
curl -sSL -o /tmp/veeam-release-deb.deb https://repository.veeam.com/backup/linux/agent/dpkg/debian/public/pool/veeam/v/veeam-release-deb/veeam-release-deb_1.0.11_amd64.deb
```

```bash
sudo dpkg -i /tmp/veeam-release-deb.deb
```

```bash
sudo apt update
```

```bash
sudo apt install -y veeam-nosnap
```

```bash
systemctl is-active veeamservice
```

### Fase 2, accettare i due contratti

Il primo comando privilegiato dopo l'installazione chiede l'accettazione di due contratti e attende `yes` per ciascuno. Va eseguito senza redirezione, altrimenti le domande finiscono nel file e il terminale sembra bloccato: è la voce A del troubleshooting. Qualunque sottocomando va bene, e conviene usare quello che serve comunque subito dopo.

```bash
sudo veeamconfig repository list
```

### Fase 3, creare il deposito

Il comando crea anche la cartella se non esiste, e la assegna a `root:veeam`.

```bash
sudo veeamconfig repository create --name "$NOME_DEPOSITO" --location "$DEPOSITO" --type local
```

```bash
sudo veeamconfig repository list
```

Il secondo comando non è una formalità: è l'unica misura che distingue un deposito registrato da una cartella con il nome giusto, e la differenza conta dopo ogni ciclo di purga e reinstallazione, come dice la voce E del troubleshooting.

### Fase 4, creare il lavoro

Un lavoro a livello di file su tutto il sistema, senza pianificazione, quindi manuale. I percorsi inclusi sono quelli decisi nella sezione sulla portabilità, e vanno nominati tutti.

```bash
sudo veeamconfig job create fileLevel --name "$NOME_LAVORO" --repoName "$NOME_DEPOSITO" --includeDirs "$INCLUDI" --excludeDirs "$ESCLUDI" --nosnap --maxPoints "$PUNTI"
```

```bash
sudo veeamconfig job list
```

### Fase 5, eseguire

Il comando restituisce subito l'identificativo di sessione e il percorso dei registri, e il lavoro prosegue in secondo piano.

```bash
sudo veeamconfig job start --name "$NOME_LAVORO"
```

L'avanzamento si segue da qualunque sessione, anche non privilegiata, con due comandi. Il primo dice quanto è stato scritto, il secondo dice se il lavoro sia ancora in corso: quando non stampa alcuna riga, è finito.

```bash
df -h "$DEPOSITO"
```

```bash
ps -eo etime,cmd | grep -E "veeamjobman|veeamagent" | grep -v grep
```

A corsa finita l'esito dichiarato si legge così, e va letto: un lavoro che finisce non è un lavoro che riesce, perché lo stato può essere anche `Warning` o `Failed`.

```bash
sudo veeamconfig session list
```

### Fase 6, rileggere l'archivio, che è la parte che rende il backup tale

Si prende l'identificativo del backup, poi quello del punto di ripristino, poi si monta il punto. Il montaggio espone il contenuto come filesystem percorribile e non scrive nulla sul sistema vivo; `restore` ed `export` non servono a questo scopo e su una macchina funzionante il primo è da evitare.

```bash
sudo veeamconfig backup list
```

```bash
sudo veeamconfig point list --backupId {IDENTIFICATIVO-DEL-BACKUP}
```

```bash
sudo veeamconfig point mount --id {IDENTIFICATIVO-DEL-PUNTO}
```

Il punto viene montato su `/mnt/backup`, che risulta `drwxr-xr-x` di `root:root`: il montaggio richiede privilegi, l'ispezione no, quindi tutto ciò che segue si può fare da una sessione ordinaria, anche remota.

```bash
ls /mnt/backup
```

Le tre verifiche che contano, in ordine di forza crescente. La prima è che ci sia ciò che si teme manchi, cioè il contenuto di ogni filesystem distinto dalla radice che si è scelto di includere.

```bash
ls "/mnt/backup/home/$UTENTE"
```

La seconda è il confronto per impronta fra sistema vivo e archivio, su file scelti in modo da coprire tutti i filesystem inclusi e più ordini di grandezza.

```bash
for f in /etc/fstab "/home/$UTENTE/.bashrc"; do sha256sum "$f" "/mnt/backup$f"; done
```

La terza è il conteggio, che misura una cosa che le impronte non misurano: un confronto per impronta prova che i file scelti sono identici e non dice nulla dei file non scelti, quindi da solo non distingue una copia integrale da una copia parziale fatta bene. Si conta ciò che costa di più ricostruire.

```bash
echo "vivo=$(find /home/$UTENTE | wc -l) backup=$(find /mnt/backup/home/$UTENTE | wc -l)"
```

Va infine verificato che il deposito non sia finito dentro sé stesso, e si verifica per assenza e non deducendolo dalla dimensione dell'archivio.

```bash
ls -A "/mnt/backup$DEPOSITO"
```

Il punto non va lasciato montato. Lo smontaggio è una chiusura di sessione e usa l'identificativo restituito dal montaggio, non quello del punto.

```bash
sudo veeamconfig session stop --id {IDENTIFICATIVO-DELLA-SESSIONE-DI-MONTAGGIO}
```

```bash
ls -A /mnt/backup
```

### Fase 7, portare l'archivio fuori dalla macchina

Un deposito sullo stesso disco che si salva protegge dall'errore e non dal guasto del disco, quindi questa fase non è facoltativa. Servono i due file, cioè l'archivio `.vbk` e il descrittore `.vbm`, perché senza il secondo il primo è un archivio che Veeam non sa più collocare.

L'utente non appartiene al gruppo `veeam` e quindi non può leggere il deposito. Lo si aggiunge una volta sola, ricordando che l'appartenenza a un gruppo si legge al login e non ha effetto sulle sessioni già aperte.

```bash
sudo sh -c "usermod -aG veeam $UTENTE; id $UTENTE"
```

Il trasferimento si fa poi dalla macchina di destinazione, con una sessione nuova. Il percorso del deposito contiene uno spazio, perché l'agente nomina la propria cartella unendo il nome della macchina e quello del lavoro, e su come proteggerlo vale la voce J del troubleshooting: un solo livello di virgolette, quelle della shell locale, e nessuna virgoletta in più per una shell remota che non esiste.

```bash
scp "studio:/home/veeam-repo/<NOME-CARTELLA-DEL-DEPOSITO>/<NOME-FILE>" "<DESTINAZIONE>"
```

La copia si verifica per impronta e non per dimensione: un trasferimento interrotto può lasciare un file della dimensione giusta, e su un archivio di backup la differenza fra una copia integra e una plausibile è tutto ciò che conta.

```bash
ssh studio "sha256sum '/home/veeam-repo/<NOME-CARTELLA-DEL-DEPOSITO>/<NOME-FILE>'"
```

```bash
sha256sum "<DESTINAZIONE>/<NOME-FILE>"
```

### Fase 8, la corsa successiva, e la sola decisione che porta con sé

Dalla seconda corsa in avanti le fasi da 0 a 4 non si rifanno, perché deposito e lavoro esistono già. Restano la fase 5 per eseguire, la 6 per rileggere e la 7 per portare fuori, identiche. Cambia una cosa sola, ed è una decisione che conviene prendere invece di ereditarla dal comando.

Senza opzioni, il lavoro produce un punto incrementale che dipende dal pieno precedente. Con l'opzione seguente produce un pieno nuovo e indipendente, che costa altrettanto spazio del primo e non eredita alcuna dipendenza.

```bash
veeamconfig job start --name "$NOME_LAVORO" --activeFull
```

Il criterio per scegliere non è lo spazio ma che cosa fotografa il punto precedente. Se il primo pieno ritrae uno stato che qualcuno vorrebbe davvero ripristinare, l'incrementale è la scelta economica e corretta. Se ritrae uno stato intermedio che nessuno vuole più, far dipendere il punto utile da quello inutile aggiunge una superficie di guasto senza aggiungere valore, e allora il pieno indipendente è la scelta giusta. Va inoltre ricordato che un punto incrementale non si porta fuori da solo: senza il pieno da cui dipende non ripristina nulla, quindi la copia esterna resta una catena e non un file.

Il numero di punti conservati è quello dichiarato alla creazione del lavoro con `--maxPoints`, e vale anche qui: con due punti e un pieno nuovo, il pieno precedente resta finché una terza corsa non lo espelle.

Un controllo che chiude la corsa e costa un comando: i punti si elencano con il proprio tipo, e un pieno indipendente compare come `Full` e non come `Increment`.

```bash
veeamconfig point list --backupId {IDENTIFICATIVO-DEL-BACKUP}
```

## Troubleshooting: sintomo, causa, rimedio

Ogni voce nasce da un caso osservato su una macchina reale e non da una previsione. L'ordine è quello in cui i casi si incontrano percorrendo la sequenza.

### A, il comando resta appeso senza stampare nulla

Sintomo: il primo sottocomando privilegiato dopo l'installazione non restituisce il prompt e non mostra niente. Causa: l'agente chiede l'accettazione di due contratti, prima quello del prodotto e poi le note sui componenti di terze parti, e attende `yes` per ciascuno; se il comando redirige l'uscita su un file, la domanda finisce nel file mentre la risposta è attesa dal terminale. Rimedio: rispondere `yes` due volte al buio, oppure interrompere e rieseguire senza redirezione.

La diagnosi, se non si sa che cosa stia aspettando, è leggere il file di destinazione da un'altra sessione: le ultime righe contengono la domanda. Regola generale che ne discende: un comando che redirige l'uscita non deve mai essere il primo contatto con un programma che non si conosce.

Va saputo che l'accettazione non sopravvive a una purga dei pacchetti, quindi questo sintomo può ricomparire molto dopo l'installazione, ed è il segnale della voce E.

### B, `veeamconfig: Permission denied` da utente normale

Sintomo: qualunque sottocomando, compreso `--help`, risponde con un rifiuto di permesso. Causa: `veeamconfig` e `veeam` sono collegamenti simbolici allo stesso binario `/usr/sbin/veeamworker`, che ha permessi `-rwxr-x---` e appartiene a `root:veeam`; il rifiuto arriva dal filesystem prima che il programma parta, ed è per questo che il messaggio è quello della shell.

Il rimedio, corretto il 2026-09-21 dopo averlo misurato. Una versione precedente di questa voce diceva che rimedio non ce n'era e che ogni passo fosse lavoro privilegiato: è falso, e va ritirato. Il gruppo `veeam` esiste proprio per questo, l'installazione del pacchetto vi aggiunge l'utente, e da una sessione che lo porta il binario si esegue senza `sudo`. Misurato su `alessio-ubuntustudio` da una sessione SSH ordinaria: l'elenco dei depositi, quello dei lavori, l'avvio del lavoro con `--activeFull`, il montaggio del punto di ripristino e la chiusura della sessione di montaggio riescono tutti con stato zero e senza privilegi.

Il motivo per cui la voce era stata scritta così merita di restare, perché è lo stesso della quarta causa della regola sul contesto di shell e della prima causa di MS-146. Il gruppo viene aggiunto dall'installazione del pacchetto, ma un processo riceve i propri gruppi quando la sessione nasce e non li rilegge mai più: la sessione da cui si installa non li ha, e continua a non averli finché resta aperta. Non è quindi vero che serva `sudo`, è vero che serve una sessione nuova. Dove non si voglia o non si possa aprirla, `sudo` resta la via che funziona subito, ed è la ragione per cui i comandi della sequenza qui sopra lo portano.

Un modo di misurare male questo caso merita una riga, perché costa tempo. Passando l'uscita a un altro comando, per esempio `head`, lo stato restituito è quello dell'ultimo elemento della pipeline e non quello di `veeamconfig`, quindi `$?` risponde zero mentre il comando è fallito. Eseguito da solo, lo stato è `126`, che in una shell POSIX significa esattamente comando trovato ma non eseguibile.

### C, `Item cannot be backed up without veeam snapshot kernel module`

Sintomo: la creazione di un lavoro a livello di volume fallisce nominando i dispositivi, e l'elenco dei lavori resta vuoto. Causa: la variante `veeam-nosnap` non prende istantanee da sé e si appoggia a quelle di LVM, che su partizioni semplici non esistono. Rimedio: o si passa alla variante con modulo del kernel, o si rinuncia al livello di volume e si usa quello di file. Non c'è nulla da correggere nella configurazione.

### D, il modulo del kernel non compila

Sintomo: l'installazione della variante con modulo termina con un errore di DKMS. Causa: da distinguere leggendo `/var/lib/dkms/<modulo>/<versione>/build/make.log`. Rimedio: se gli errori riguardano un ambiente incompleto si installa ciò che manca e si riprova; se nominano funzioni dichiarate implicitamente, membri di strutture inesistenti o funzioni chiamate con un numero di argomenti diverso, il modulo è scritto per una generazione di kernel precedente e non c'è nulla da aggiustare. In quel caso si esce dallo stato rotto con la purga e la reinstallazione della variante senza modulo, come descritto più sopra, sapendo che quella purga innesca la voce E.

### E, dopo una purga e reinstallazione il deposito e i lavori non ci sono più

Sintomo: i contratti vengono richiesti di nuovo, e `veeamconfig repository list` e `veeamconfig job list` stampano la sola riga di intestazione, pur essendo stati creati in precedenza. Causa: le due varianti del pacchetto condividono le cartelle di stato `/etc/veeam` e `/var/lib/veeam`, quindi la purga dell'una cancella la configurazione dell'altra, anche quando l'altra è stata soltanto rimossa e mai purgata. Rimedio: ricreare deposito e lavoro, cioè rieseguire le fasi 3 e 4.

La trappola vera di questa voce non è la perdita ma il modo in cui si nasconde. La purga rimuove la configurazione e non tocca la cartella dei dati, quindi resta sul disco una directory con il nome giusto, i permessi giusti e nessun deposito dietro: un `ls` risponde che c'è. La verifica corretta è chiedere al programma e non al filesystem, e costa quanto un `ls`.

Regola generale: dopo ogni ciclo di purga e reinstallazione la configurazione va riverificata con i comandi del programma, perché nulla nell'esito dell'installazione segnala la perdita.

### F, l'archivio cresce e non si ferma

Sintomo: lo spazio occupato dalla destinazione aumenta ben oltre la dimensione della sorgente. Causa: il deposito si trova dentro un percorso incluso nel backup, quindi il lavoro sta copiando sé stesso. Rimedio: escludere il percorso del deposito con `--excludeDirs`. La verifica non è che l'archivio smetta di crescere, che è solo un indizio, ma che il percorso del deposito sia assente dentro il punto montato.

### G, un filesystem distinto potrebbe non entrare

Sintomo: nessuno, ed è il problema. L'aiuto di `--includeDirs` descrive il parametro come elenco di directory separate da virgola e non dichiara se l'attraversamento si fermi al confine del filesystem. Causa potenziale: su una macchina con `/home` o `/var` su partizione separata, `--includeDirs /` da solo potrebbe non includerli, e il backup riuscirebbe dichiarando successo. Rimedio: nominare esplicitamente ogni punto di innesto da salvare. Se l'attraversamento avviene i percorsi in più sono ridondanti e innocui, se non avviene sono indispensabili, quindi la forma esplicita è corretta in entrambi i casi. Verifica: cercare dentro il punto montato un percorso che esiste solo sulla partizione in questione.

### H, i permessi dell'archivio e la destinazione esterna

Sintomo: nessuno finché l'archivio resta dov'è. Il file `.vbk` ha permessi `-rw-rw-rw-`, cioè leggibile e scrivibile da chiunque, e appartiene a `root:veeam`; la protezione effettiva viene dalla cartella che lo contiene, che è `drwxrws---` e nega l'accesso a chi non è nel gruppo `veeam`. Causa del problema: copiando l'archivio su un volume esterno con un filesystem che non conserva i permessi POSIX, quella protezione sparisce, e l'archivio contiene l'intero sistema, quindi anche `/etc/shadow`. Rimedio: decidere della cifratura prima di creare il lavoro, con `--setEncryption`, perché si cifra il lavoro e non l'archivio, quindi attivarla dopo significa rifare il backup.

### I, l'utente non riesce a leggere l'archivio per trasferirlo

Sintomo: `scp` o una copia da un'altra macchina falliscono per permesso negato sul file `.vbk`. Causa: la cartella del deposito è `drwxrws---` di `root:veeam` e l'utente non appartiene al gruppo. Rimedio: o si aggiunge l'utente al gruppo `veeam`, sapendo che l'appartenenza si legge al login e richiede quindi una sessione nuova per avere effetto, oppure si copia l'archivio in una cartella dell'utente con `sudo` e la si cancella dopo il trasferimento, al prezzo di occupare temporaneamente lo spazio una seconda volta.

### J, `scp` dice che il file non esiste, e il file esiste

Sintomo: `scp` verso un percorso remoto che contiene uno spazio risponde `No such file or directory`, mentre lo stesso percorso elencato con `ls` o `find` sulla macchina remota c'è ed è leggibile. Causa: dalla versione 9.0 OpenSSH usa per `scp` il protocollo SFTP invece del vecchio trasferimento su shell remota, quindi il percorso non viene più interpretato da alcuna shell sulla macchina remota. La ricetta tradizionale, cioè proteggere lo spazio con un secondo livello di virgolette destinato alla shell remota, produce ora un nome di file che contiene davvero i caratteri di virgoletta, e quel file non esiste.

Rimedio: un solo livello di virgolette, quello della shell locale, che passa il percorso come argomento unico. Se serve il comportamento vecchio, per esempio per usare un carattere jolly che qualcuno deve espandere, l'opzione `-O` forza il protocollo precedente e allora la shell remota torna a esistere e le due virgolette tornano necessarie.

Il modo di accorgersene senza perdere tempo è che il messaggio nomina il percorso comprese le virgolette: leggerlo per intero, invece di fermarsi alle ultime parole, dice da solo qual è il problema.

## Il criterio che chiude, e che quasi nessuno applica

Un backup non è tale finché non è stato riletto. La prova può essere parziale, cioè estrarre alcuni file dall'archivio e confrontarli con gli originali, e già questo distingue una copia da una speranza; la prova completa è ripristinare davvero, su una macchina di prova o dentro una macchina virtuale, e avviare il risultato.

Nell'esperienza precedente citata in apertura quella prova era stata fatta per davvero, con le macchine ripristinate avviate e verificate dall'interno, ed è la ragione per cui quell'esperienza vale più di una procedura scritta. Va ripetuta qui, e finché non è fatta la voce corrispondente del registro delle azioni differite resta aperta.

[^1]: *istantanea*, in inglese snapshot - una vista congelata del contenuto di un volume in un dato momento, che permette di copiarlo mentre il sistema continua a scrivere. Senza di essa una copia presa a caldo può contenere file in stato intermedio.

[^2]: *DKMS*, Dynamic Kernel Module Support - il meccanismo con cui Linux ricompila automaticamente i moduli di terze parti quando il kernel cambia, così che non vadano reinstallati a mano a ogni aggiornamento.

[^3]: *LVM*, Logical Volume Manager - lo strato che su Linux permette di costruire volumi logici sopra una o più partizioni fisiche, con la possibilità di ridimensionarli e di prenderne istantanee. È la disposizione predefinita di molte installazioni, ma non di tutte.
