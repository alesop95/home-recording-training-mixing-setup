<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Fotografia finale della macchina, e il confronto con quella di partenza

> Scritta il 2026-09-21, eseguendo la fase 11 della procedura di [installazione pulita](installazione-pulita-26-04.md) a fase 8 chiusa. Contiene tre cose che di solito stanno in tre posti diversi, e stanno insieme perché una sola delle tre si legge da sola: la sequenza dei comandi da rieseguire su qualunque macchina, il confronto voce per voce fra lo stato di oggi e quello del 2026-09-07, e le voci di troubleshooting nate dai casi incontrati durante l'esecuzione. Il racconto per intervento sta nei microstep da MS-144 a MS-146 di [OPERATIONS-LOG.md](../OPERATIONS-LOG.md). I file grezzi delle due fotografie stanno sulla macchina in `~/fotografia-pre-reinstall` e `~/fotografia-post-reinstall`, e una copia della seconda sta sulla postazione sotto `_notes/`, che è ignorata da git.

## A che cosa serve una fotografia finale, e perché non è una formalità

Il controllo di uscita di una reinstallazione non è che il sistema si avvii, perché quello si sa il primo giorno. È che lo stato raggiunto sia quello voluto in ogni voce che era stata registrata prima di azzerare, e che gli scarti siano tutti spiegati. La fotografia iniziale esiste proprio per rendere possibile questo confronto, e senza il confronto sarebbe stata una raccolta di file mai riletti.

Il valore pratico si vede in una asimmetria. Uno scarto atteso, per esempio la versione del sistema che passa dalla 25.04 alla 26.04, non insegna niente perché era il motivo del lavoro. Uno scarto non atteso è invece l'unica cosa che questa fase produce e che nessun altro controllo avrebbe prodotto, perché è un pezzo di stato che nessuno stava guardando. In questa esecuzione gli scarti non attesi sono stati tre, e nessuno dei tre era nel piano della giornata.

Vale anche il verso opposto, ed è la ragione per cui alcune voci si registrano pur sapendo già la risposta. Una coincidenza attesa che si verifica non è rumore: la partizione `/home` che porta lo stesso identificativo univoco[^1] di prima è la prova che la conservazione dei dati è riuscita, e senza quella riga la si crederebbe soltanto.

## La sequenza operativa, replicabile

La forma che segue è parametrica e vale su qualunque macchina Linux con `apt`, non solo su questa. Si esegue dalla postazione via SSH, oppure sulla macchina stessa togliendo il `ssh <alias>` iniziale. Nessun comando richiede privilegi, e nessuno modifica alcunché tranne la creazione della cartella di destinazione.

Il presupposto da verificare prima di cominciare è che esista la fotografia di partenza con cui confrontare, perché senza di essa questa fase produce dati e non un confronto.

```bash
ssh studio "ls ~/fotografia-pre-reinstall | wc -l"
```

La raccolta si scrive come uno script e non come una sequenza di comandi incollati, per una ragione che si paga alla seconda esecuzione: uno script si rilancia identico, e due raccolte fatte con comandi leggermente diversi non sono confrontabili. Lo script che segue riproduce di proposito gli stessi nomi di file della fotografia di partenza, così che il confronto sia un `diff` file per file.

```bash
ssh studio "cat > ~/fase11.sh && chmod +x ~/fase11.sh" < fase11.sh
```

```bash
ssh studio "bash ~/fase11.sh"
```

Il contenuto dello script, nella forma usata qui, raccoglie ventinove voci. Le prime ventitre portano i nomi della fotografia di partenza e servono al confronto; le ultime sei sono nuove e descrivono lo stato che allora non esisteva, cioè i limiti realtime effettivi, i gruppi dell'utente, i target di sospensione mascherati, i comandi di Wine presenti nel PATH e l'elenco dei prefix.

```bash
lsb_release -a                                   > 01-sistema.txt              2>&1
cat /etc/os-release                              > 01b-os-release.txt          2>&1
cat /etc/update-manager/release-upgrades         > 02-release-upgrades.txt     2>&1
cat /etc/apt/sources.list.d/ubuntu.sources       > 03-sorgenti-apt.txt         2>&1
ls -la /etc/apt/sources.list.d/                  > 03b-sorgenti-elenco.txt     2>&1
cat /etc/apt/sources.list.d/winehq*.sources      > 03c-winehq.txt              2>&1
lsblk -f                                         > 04-dischi.txt               2>&1
df -h                                            > 04b-spazio.txt              2>&1
cat /etc/fstab                                   > 06-fstab.txt                2>&1
aplay -l                                         > 07-audio-out.txt            2>&1
arecord -l                                       > 08-audio-in.txt             2>&1
cat /proc/cmdline                                > 08b-cmdline.txt             2>&1
cat /etc/security/limits.d/*audio*               > 08c-limits.txt              2>&1
dpkg -l                                          > 10-pacchetti-tutti.txt      2>&1
apt-mark showmanual                              > 11-pacchetti-manuali.txt    2>&1
wine --version                                   > 12-wine.txt                 2>&1
dpkg -l | grep -i wine                           > 12b-wine-pacchetti.txt      2>&1
find "$HOME" -maxdepth 3 -name system.reg -printf '%h\n' | sort > 13-prefix-wine.txt 2>&1
uname -a                                         > 14-kernel.txt               2>&1
dpkg --print-foreign-architectures               > 15-architetture.txt         2>&1
do-release-upgrade -c                            > 16-release-upgrade-check.txt 2>&1
apt-get -s dist-upgrade                          > 17-simulazione-upgrade.txt  2>&1
cat /var/log/apt/history.log                     > 18-cronologia-apt.txt       2>&1
pactl info                                       > 19-pactl.txt                2>&1
ulimit -r -l                                     > 20-limiti-effettivi.txt     2>&1
id                                               > 21-gruppi.txt               2>&1
systemctl list-unit-files --state=masked         > 22-target-mascherati.txt    2>&1
command -v wine wine64 wine32 winetricks xdotool Xvfb > 23-comandi-wine.txt    2>&1
ls -la "$HOME/wineprefixes"                      > 24-prefix-elenco.txt        2>&1
```

Il controllo che la raccolta sia completa si fa cercando i file vuoti, perché un comando assente produce un file vuoto e non un errore, e un file vuoto in mezzo a ventotto pieni non si nota.

```bash
ssh studio "find ~/fotografia-post-reinstall -maxdepth 1 -type f -empty -printf '%f\n'"
```

Il confronto vero si fa poi sulla macchina, un file alla volta, perché è là che stanno entrambe le fotografie e perché trasferire per confrontare aggiunge un passaggio che può sbagliare.

```bash
ssh studio "cd ~ && for f in 01-sistema 02-release-upgrades 06-fstab 12-wine 14-kernel 15-architetture; do diff -u fotografia-pre-reinstall/\$f.txt fotografia-post-reinstall/\$f.txt; done"
```

L'ultimo passo porta la fotografia fuori dalla macchina, che è la parte senza la quale tutto il resto è fragile quanto il disco su cui sta.

```bash
scp -r studio:~/fotografia-post-reinstall/. "_notes/fotografia-post-reinstall-<data>/"
```

La copia si verifica per impronta alle due estremità e non per dimensione, applicando lo stesso criterio che la procedura di backup impone all'archivio.

```bash
ssh studio "cd ~/fotografia-post-reinstall && md5sum * | awk '{print \$1}' | sort"
```

```bash
cd "_notes/fotografia-post-reinstall-<data>" && md5sum * | awk '{print $1}' | sort
```

## Il confronto, voce per voce

La tabella riporta le voci in cui lo stato è cambiato o in cui l'invarianza è essa stessa il risultato. Le voci non elencate sono identiche e non dicono nulla.

| Voce | 2026-09-07 | 2026-09-21 | Lettura |
|---|---|---|---|
| sistema | Ubuntu 25.04 `plucky` | Ubuntu 26.04.1 LTS `resolute` | atteso, è lo scopo del lavoro |
| kernel | `6.14.0-35-generic` | `7.0.0-31-generic` | atteso |
| nome della macchina | `i7-6700-16GBDDR4-500GBSSD` | `alessio-ubuntustudio` | atteso, scelto durante l'installazione |
| direttiva di aggiornamento | `Prompt=normal` | `Prompt=lts` | atteso, è la riga che impedisce il ripetersi del blocco |
| controllo di nuova versione | offre il salto alla 26.04.1 | nessuna LTS di sviluppo disponibile | atteso, ed è la conferma che la direttiva funziona |
| parametri di avvio | `preempt=full threadirqs` | `preempt=full threadirqs rcu_nocbs=all` | migliorato, il terzo parametro prima non c'era |
| radice | UUID `19351fc8` | UUID `bdae6be9` | atteso, la radice è stata riformattata |
| `/home` | UUID `4aa5afff` | UUID `4aa5afff` | invariato, ed è la prova che `/home` è sopravvissuta |
| partizione EFI | UUID `9054-9070` | UUID `9054-9070` | invariato, non è stata formattata per scelta |
| swap | UUID `50a4c66e` | stesso UUID con `nofail` | invariato nella sostanza |
| occupazione di `/home` | 3,7 GB | 66 GB | atteso, sono i prefix, i materiali e due residui |
| uscite audio | cinque dispositivi su `ALC887-VD` | identiche | invariato |
| ingressi audio | un dispositivo di cattura | identico | invariato |
| architetture straniere | `i386` | `i386` | invariato, e contraddice il confronto atteso scritto nella procedura |
| Wine | `wine-9.0` | `wine-10.0` della distribuzione | atteso |
| pacchetti Wine | sette, con `wine-stable` transitorio | sette, con `wine` e `wine64` espliciti | atteso |
| prefix Wine | uno solo, `~/.wine` | quattro in uso, più tre residui | atteso, è la mappa di ADR-019 |
| sorgenti apt | `plucky` più due file WineHQ | `resolute` più un file WineHQ e uno Veeam | uno scarto non atteso, si veda più sotto |
| pacchetti manuali | quarantotto | cinquantuno | atteso |

Sei voci meritano più di una riga di tabella.

La conservazione di `/home` è l'unica affermazione dell'intera procedura che non poteva essere verificata mentre la si faceva, perché una formattazione accidentale si scopre dopo. L'identificativo univoco della partizione è lo stesso di quattordici giorni prima, e poiché formattare un filesystem ne genera uno nuovo, l'uguaglianza è una prova e non un indizio. La stessa lettura vale al contrario per la radice, il cui identificativo è cambiato: era esattamente ciò che si voleva.

L'architettura straniera `i386` ancora dichiarata è il punto in cui il confronto atteso scritto nella procedura è sbagliato, e va corretto nel documento e non nei fatti. La fase 11 prevedeva un elenco vuoto, perché fu scritta prima di ADR-016, quando si credeva che l'architettura a 32 bit fosse un residuo da non ricreare. ADR-016 ha poi stabilito il contrario per misura, cioè che Akabak è a 32 bit e che l'architettura va dichiarata sul sistema, e MS-141 lo ha rafforzato di una ragione in più. La riga della procedura è quindi una previsione superata, e questa pagina la ritira.

Il terzo parametro di avvio, `rcu_nocbs=all`, non c'era sulla macchina precedente e c'è su questa. Non è stato aggiunto da nessuno: arriva dal pacchetto di impostazioni a bassa latenza della 26.04, che lo scrive insieme agli altri due. Sposta il lavoro differito del meccanismo di lettura concorrente del kernel fuori dai processori che eseguono il carico, che è precisamente ciò che serve a una macchina audio, quindi è un miglioramento e non una deriva.

I limiti realtime meritano una nota perché la voce raccolta sembra raddoppiata e non lo è, ed è una imprecisione di questa pagina corretta il 2026-09-21 dopo averla misurata. Il comando della raccolta legge `/etc/security/limits.d/*audio*`, e quel modello incontra due file: `30-ubuntustudio-audio.conf`, che è quello del pacchetto di impostazioni di Ubuntu Studio, e `audio.conf.disabled`, che è quello del pacchetto `jackd`. Il secondo porta le stesse due righe ma non è in vigore, perché PAM legge i soli file con estensione `.conf` e quel nome termina altrimenti: qualcuno lo ha disattivato di proposito rinominandolo. La voce raccolta mostra quindi due dichiarazioni dove una sola agisce, ed è un caso da manuale del motivo per cui i limiti si verificano dove sono in vigore e non dove sono scritti. La verifica in vigore è la voce nuova numero venti, che riporta `rtprio 95` e `memlock unlimited`, ed è la stessa forma che MS-077 aveva imposto dopo aver scoperto che i limiti scritti non erano attivi.

I pacchetti installati manualmente passano da quarantotto a cinquantuno, e il contenuto della differenza è più interessante del conteggio. Entrano i due pacchetti di Veeam, che sono la traccia di PA-011, e i due strumenti `xdotool` e `xvfb`, che sono la capacità acquisita in MS-111 di lavorare in interfaccia da remoto. Esce `fonts-wine`, ed entra la coppia `wine` e `wine64` al posto del transitorio `wine-stable`. Il resto della differenza non riguarda questo progetto ed è il rimescolamento degli strumenti di base della 26.04, che affianca ai `coreutils` storici la loro riscrittura in Rust.

L'elenco dei prefix è la voce che ha richiesto una correzione del comando e non solo una lettura, e il motivo sta nella sezione seguente.

## I tre accertamenti non attesi

### Il repository di WineHQ è rimasto, e dice perché il tentativo del 17 settembre non poteva riuscire

Il file `/etc/apt/sources.list.d/winehq-resolute.sources` esiste ancora sulla macchina, con data del 2026-09-17, insieme alla chiave in `/etc/apt/keyrings/`. È il residuo del tentativo di sostituzione di Wine che MS-140 aveva eseguito e MS-142 aveva annullato: i pacchetti sono stati rimossi con purga, il repository no. Da solo non fa danno, perché nessun pacchetto di quella provenienza è installato e `apt upgrade` non installa ciò che non c'è, ma è uno stato dichiarato che nessuno ha deciso di lasciare, ed è esattamente il genere di cosa che una fotografia finale esiste per trovare.

Il suo contenuto, però, dice molto di più della propria presenza. La riga `Architectures: amd64` significa che da quel repository apt non scarica affatto l'indice dei pacchetti a 32 bit, e la cronologia di apt conferma che il 2026-09-17 furono installati soltanto `winehq-stable:amd64` e `wine-stable:amd64`, senza alcuna controparte `i386`. La domanda naturale, cioè se quella riga fosse un errore di configurazione che ha invalidato la prova, ha una risposta misurabile e la risposta è no: il file `Release` pubblicato da WineHQ per la suite `resolute` dichiara esso stesso `Architectures: amd64`, e l'indice `binary-i386` risponde con un codice 404.

La misura estesa alle quattro suite vicine mostra dove sta la cesura, e la cesura non è di questa macchina.

| Suite WineHQ | Versione di Ubuntu | Architetture pubblicate |
|---|---|---|
| `noble` | 24.04 LTS | `amd64 i386` |
| `plucky` | 25.04 | `amd64 i386` |
| `questing` | 25.10 | `amd64` |
| `resolute` | 26.04 LTS | `amd64` |

Ne segue la lettura, e va enunciata con precisione perché cambia il valore di una conclusione già scritta senza cambiarne l'esito. MS-141 aveva misurato che i pacchetti di WineHQ non popolano il lato a 32 bit dei prefix su questa macchina, e aveva dichiarato apertamente di non avere isolato la causa prima. La causa prima ora si conosce: per Ubuntu 26.04 WineHQ non pubblica alcuna metà a 32 bit, quindi il caricatore ELF[^2] a 32 bit non esisteva da installare e l'unica strada disponibile era il WoW64[^3] nuovo, che su questa macchina non ha inizializzato `syswow64`. L'esclusione ottava di MS-143 resta quindi valida e diventa più forte, perché non è più un comportamento osservato su una macchina ma una proprietà dichiarata del repository, verificabile da chiunque senza installare nulla.

Due cose che questo accertamento non dimostra, e vanno dette perché la tentazione di estenderlo è forte. Non dimostra che il WoW64 nuovo sia difettoso in generale: dimostra che su questa macchina non ha inizializzato il lato a 32 bit, e la causa prima di quel fallimento resta non isolata come MS-141 dichiarava. E non dimostra che WineHQ sia inutilizzabile in assoluto, perché sulle due versioni precedenti di Ubuntu la metà a 32 bit c'è: dice che non è utilizzabile per programmi a 32 bit su questa versione di Ubuntu, che è l'affermazione che serve a questo progetto.

La conseguenza pratica è una sola e semplice. Finché il corredo di questo progetto è fatto di programmi a 32 bit, e lo è tutto, i pacchetti della distribuzione sono l'unica fornitura possibile su Ubuntu 26.04, e ADR-016 ne esce confermata per la terza volta e per una ragione nuova. Che fare del repository rimasto era invece una decisione e non una copia, ed è stata presa lo stesso giorno: il file di sorgenti e la sua chiave sono stati rimossi, perché una configurazione conservata vale per il poterla riusare e la misura dice che su questa versione di Ubuntu non è riusabile. L'esecuzione, con lo stato verificato dopo, è in MS-151, e PA-018 è compiuta.

### La catena audio non è osservabile da una sessione SSH quando il posto è tenuto dallo schermo di accesso

La voce nuova numero diciannove, cioè `pactl info`, ha riportato `Default Sink: auto_null`, che è il dispositivo fittizio che PipeWire[^4] espone quando non trova alcuna scheda. L'elenco delle schede è vuoto e l'unica uscita è una `Dummy Output`, mentre ALSA[^5], cioè lo strato sottostante, vede regolarmente la scheda integrata con i suoi cinque dispositivi di riproduzione. Due strati adiacenti danno quindi risposte opposte sulla stessa domanda.

La causa è stata isolata per misura, e sono due cause indipendenti che agiscono insieme, il che è la ragione per cui è utile scriverle entrambe invece della prima che si trova.

La prima è che i nodi in `/dev/snd/` appartengono a `root:audio` con permessi `crw-rw----` e portano una lista di controllo di accesso[^6] che `systemd-logind` assegna alla sola sessione attiva del posto. In questo momento la sessione attiva del posto `seat0` è quella dello schermo di accesso, e infatti la lista riporta `user:sddm:rw-` e nessuna voce per l'utente: la sessione grafica dell'utente esiste ancora, risulta `Active=no` e `State=online`, e ha perso l'accesso ai dispositivi quando lo schermo di accesso ha preso il posto.

La seconda è che il processo `wireplumber` di quella sessione gira con l'insieme di gruppi `4 24 27 30 46 100 111 115 1000`, dove mancano il gruppo `audio`, che è il 29, e il gruppo `pipewire`, che è il 982. L'utente vi appartiene, e lo conferma la voce nuova numero ventuno, ma un processo riceve i gruppi quando la sessione nasce e non li rilegge mai più: quella sessione grafica è iniziata il 2026-09-09 alle 12:57, cioè prima che MS-077 aggiungesse l'utente ai due gruppi. È la stessa forma della quarta causa della regola sul contesto di shell, cioè un processo che porta con sé l'ambiente del momento in cui è partito, e qui l'ambiente sono i gruppi.

Ciò che questo accertamento dimostra e ciò che non dimostra vanno separati, perché confonderli produrrebbe un allarme falso. Dimostra che la catena audio non si verifica da una sessione SSH mentre il posto è tenuto dallo schermo di accesso, e che una misura fatta così riporta l'assenza di schede anche su una macchina perfettamente funzionante. Non dimostra che la catena audio sia guasta: nessuna delle due cause riguarda la configurazione del sistema, ed entrambe cadono con un accesso nuovo alla console, che riattiva la sessione e le assegna i gruppi correnti. La verifica vera resta da fare in quella forma, ed è tracciata come PA-019.

La lezione generale è la stessa di MS-093, dove si scoprì che Wine non trova un display da una sessione SSH, e conviene enunciarla una volta per tutte in forma che copra entrambi i casi. Una sessione remota non è una finestra sulla macchina: è una sessione diversa, con un proprio ambiente, propri gruppi e nessun posto assegnato, e tutto ciò che dipende dalla sessione grafica va misurato da quella e non da questa.

### Un comando di censimento copiato alla lettera misurava meno di quanto dichiarasse

Il comando che elenca i prefix di Wine cerca il file di registro `system.reg`, che è ciò che rende un prefix riconoscibile qualunque nome porti, e nella fotografia di partenza aveva la forma `find ~ -maxdepth 2`. Riprodotto identico oggi, per avere due file confrontabili, ha riportato due prefix invece di sette.

La ragione non è un difetto del comando ma un cambiamento del terreno che il comando misura. Nel 2026-09-07 esisteva il solo `~/.wine`, che sta a profondità uno; oggi i prefix del corredo stanno sotto `~/wineprefixes/`, quindi a profondità due, e il loro `system.reg` a profondità tre. Il limite di profondità che era generoso allora è diventato stretto adesso, e il comando ha continuato a rispondere senza errori, riportando un sottoinsieme.

Corretto a `-maxdepth 3`, il censimento riporta sette prefix, che si leggono in tre gruppi: i quattro in uso, cioè `~/.wine` per Akabak e VACS più i tre sotto `~/wineprefixes/`; i tre danneggiati sotto `~/wineprefixes.rotto-winehq/`, che sono il residuo del tentativo annullato in MS-142 e comprendono anche un prefix `teststaging` creato durante le prove; e `~/.wine-prima-di-wine10`, che è la copia di sicurezza presa prima della migrazione di MS-085.

La regola che ne discende vale oltre questo caso. Un comando di censimento riprodotto alla lettera da una raccolta precedente misura ciò che era vero allora, e un limite di ampiezza scritto in quel comando è una assunzione sul terreno, non una parte della domanda. Un censimento che restituisce meno del previsto va sempre riletto guardando i suoi limiti prima delle sue risposte, perché non produce un errore: produce una risposta più corta.

## Troubleshooting: sintomo, causa, rimedio

### A, la raccolta produce un file vuoto e nessun errore

Sintomo: uno dei file della cartella ha dimensione zero. Causa: il comando corrispondente non esiste sulla macchina, oppure il percorso che legge non esiste, e la redirezione con `2>&1` cattura l'errore invece di mostrarlo. Rimedio: il controllo con `find -type f -empty` va eseguito sempre subito dopo la raccolta, e il comando che ha fallito si esegue a mano per leggerne il messaggio.

### B, il file delle sorgenti di WineHQ risulta assente

Sintomo: la voce corrispondente della raccolta è vuota o riporta un percorso non trovato. Causa: su una macchina che non ha mai avuto quel repository è il comportamento corretto e non un difetto. Rimedio: nessuno, ma la voce va conservata nella raccolta, perché l'assenza di un repository di terze parti è un dato quanto la sua presenza.

### C, il censimento dei prefix di Wine ne riporta meno di quanti se ne conoscono

Sintomo: `find` con un limite di profondità risponde senza errori elencandone un sottoinsieme. Causa: i prefix stanno più in profondità del limite scritto nel comando, tipicamente perché sono stati raccolti sotto una cartella comune dopo che il comando fu scritto. Rimedio: portare il limite a `-maxdepth 3`, che copre sia i prefix nella radice della home sia quelli sotto una cartella. Caso reale su `alessio-ubuntustudio` il 2026-09-21, con due prefix riportati su sette.

### D, `pactl info` riporta `auto_null` e l'elenco delle schede è vuoto

Sintomo: PipeWire non vede alcuna scheda mentre `aplay -l` la vede. Causa: la sessione grafica dell'utente non è la sessione attiva del posto, quindi `systemd-logind` ha assegnato la lista di controllo di accesso sui nodi di `/dev/snd/` allo schermo di accesso; concorre, se la sessione è vecchia, la mancanza dei gruppi `audio` e `pipewire` nel processo `wireplumber`, che li avrebbe ricevuti solo se la sessione fosse nata dopo l'aggiunta. Rimedio: la verifica della catena audio si esegue da una sessione grafica attiva sulla console, non da SSH. Le due cause si distinguono con `getfacl /dev/snd/controlC0` e leggendo la riga `Groups` in `/proc/<pid di wireplumber>/status`. Caso reale su `alessio-ubuntustudio` il 2026-09-21.

### E, il confronto delle impronte fra Windows e Linux segnala tutte le righe diverse

Sintomo: `diff` fra l'uscita di `md5sum` eseguito sulla postazione Windows e quella eseguita sulla macchina Linux riporta ogni riga come differente, pur essendo le impronte identiche. Causa: `md5sum` di Git Bash stampa un asterisco prima del nome del file, a indicare la lettura in modo binario, mentre quello di Linux stampa due spazi. Rimedio: confrontare le sole impronte estraendo il primo campo prima dell'ordinamento. Caso reale il 2026-09-21, e vale registrarlo perché per un istante sembra che la copia sia andata male.

### F, `do-release-upgrade -c` risponde che non c'è alcuna LTS di sviluppo

Sintomo: il comando non offre alcun aggiornamento e nomina una versione di sviluppo. Causa: è la risposta corretta su una LTS con la direttiva `Prompt=lts`, che per costruzione propone soltanto la LTS successiva, cioè in questo caso la 28.04 quando uscirà. Rimedio: nessuno. Il messaggio è la prova che la direttiva è in vigore, ed è l'esito atteso della voce corrispondente del confronto.

### G, l'occupazione di `/home` è molto maggiore di quella attesa

Sintomo: lo spazio occupato è cresciuto di decine di gigabyte rispetto alla fotografia di partenza. Causa: oltre ai materiali e ai prefix, possono esserci residui di lavori precedenti, cioè archivi di sicurezza e copie di cartelle messe da parte. Rimedio: elencare i file più grandi della home prima di concludere che la crescita sia normale, perché un residuo dimenticato entra in ogni backup successivo. Su questa macchina i residui noti al 2026-09-21 sono due e sono tracciati nel prerequisito di PA-017.

## Il criterio che chiude la fase

La fase 11 è compiuta quando valgono quattro cose insieme, e la quarta è quella che di solito si salta. La cartella della fotografia finale esiste e nessuno dei suoi file è vuoto. Ogni voce che risulta diversa dalla fotografia di partenza è spiegata, e le spiegazioni sono scritte dove qualcuno le rileggerà, cioè qui e non in una conversazione. Ogni scarto non atteso ha prodotto o una correzione o una voce nelle azioni differite, perché uno scarto osservato e non trattato è peggio di uno non osservato, dato che qualcuno l'ha già pagato e nessuno ne ricaverà nulla. E la fotografia sta anche fuori dalla macchina, verificata per impronta alle due estremità.

Su questa macchina il criterio è soddisfatto al 2026-09-21, con tre scarti non attesi che hanno prodotto una correzione a questa procedura, due azioni differite nuove e una causa finalmente identificata per una conclusione che era stata raggiunta senza.

[^1]: *UUID*, Universally Unique Identifier - identificativo di 128 bit che un filesystem riceve quando viene creato; formattare una partizione ne genera uno nuovo, quindi la sua invarianza prova che il filesystem non è stato ricreato.

[^2]: *ELF*, Executable and Linkable Format - il formato degli eseguibili nativi su Linux; il caricatore a 32 bit di Wine è un eseguibile ELF a 32 bit, distinto dalle librerie Windows a 32 bit che il prefix contiene.

[^3]: *WoW64*, Windows on Windows 64-bit - lo strato che permette a un processo a 32 bit di girare dentro un ambiente a 64 bit; nella sua forma nuova Wine lo realizza senza alcun codice Unix a 32 bit, e nella forma classica usa il caricatore ELF a 32 bit.

[^4]: *PipeWire* - il server audio e video che sulla 26.04 sostituisce PulseAudio e JACK esponendo le interfacce di entrambi; `wireplumber` è il gestore di sessione che gli dice quali dispositivi usare.

[^5]: *ALSA*, Advanced Linux Sound Architecture - lo strato del kernel che espone le schede audio come dispositivi in `/dev/snd/`; sta sotto PipeWire e può vedere una scheda che PipeWire non vede.

[^6]: *ACL*, Access Control List - insieme di permessi aggiuntivi oltre ai tre classici di utente, gruppo e altri; `systemd-logind` la usa per dare all'utente della sessione attiva l'accesso ai dispositivi del posto, e la revoca quando la sessione smette di essere attiva.
