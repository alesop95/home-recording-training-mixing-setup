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

## Il criterio che chiude, e che quasi nessuno applica

Un backup non è tale finché non è stato riletto. La prova può essere parziale, cioè estrarre alcuni file dall'archivio e confrontarli con gli originali, e già questo distingue una copia da una speranza; la prova completa è ripristinare davvero, su una macchina di prova o dentro una macchina virtuale, e avviare il risultato.

Nell'esperienza precedente citata in apertura quella prova era stata fatta per davvero, con le macchine ripristinate avviate e verificate dall'interno, ed è la ragione per cui quell'esperienza vale più di una procedura scritta. Va ripetuta qui, e finché non è fatta la voce corrispondente del registro delle azioni differite resta aperta.

[^1]: *istantanea*, in inglese snapshot - una vista congelata del contenuto di un volume in un dato momento, che permette di copiarlo mentre il sistema continua a scrivere. Senza di essa una copia presa a caldo può contenere file in stato intermedio.

[^2]: *DKMS*, Dynamic Kernel Module Support - il meccanismo con cui Linux ricompila automaticamente i moduli di terze parti quando il kernel cambia, così che non vadano reinstallati a mano a ogni aggiornamento.

[^3]: *LVM*, Logical Volume Manager - lo strato che su Linux permette di costruire volumi logici sopra una o più partizioni fisiche, con la possibilità di ridimensionarli e di prenderne istantanee. È la disposizione predefinita di molte installazioni, ma non di tutte.
