<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Guasti di Wine incontrati e loro risoluzione

> Registro dei problemi realmente osservati sulla macchina, con la causa accertata e la cura. Non è un elenco di problemi possibili: è la cronaca di quelli capitati, che è il materiale più utile quando lo stesso guasto si ripresenta.

## Lo stato che ha generato i guasti

La configurazione da cui provengono i problemi registrati era mista, e la mescolanza è la causa comune di tutto ciò che segue. Era stata aggiunta l'architettura `i386`, erano stati installati `wine64` e `wine32` dai repository di Ubuntu, ed era stato lanciato `winetricks` per .NET 4.8 e i font di base. Il prefix esistente `~/.wine`, però, era stato creato prima di questa sequenza, quindi risultava incompatibile o corrotto rispetto all'installazione nuova, e `winecfg` falliva nel caricare `kernel32.dll`.

Questo è il pattern da riconoscere: non un pacchetto mancante, ma un prefix creato con una configurazione di Wine diversa da quella attualmente installata. È anche il motivo per cui la pagina di configurazione insiste sull'ordine dei passi, con l'architettura aggiunta prima dell'installazione e i prefix creati dopo.

## Errore: could not load kernel32.dll, status c0000135

La libreria `kernel32.dll` è una delle librerie di base di Windows, e Wine la fornisce dentro il prefix. Quando non riesce a caricarla il problema non è quasi mai la libreria in sé: è che l'installazione di Wine è incompleta oppure che il prefix non è coerente con essa.

Due cause distinte, che vale separare perché hanno cure diverse.

La prima causa è un prefix non inizializzato o incoerente. Anche avendo installato `wine32`, il prefix esistente può essere stato creato quando quel binario non c'era, e non contiene le DLL di base per l'architettura richiesta. La cura è creare un prefix pulito, che alla prima esecuzione genera tutte le DLL di base, `kernel32.dll` compresa, e apre la finestra di configurazione senza errori.

```bash
WINEARCH=win32 WINEPREFIX=~/wineprefixes/wine32 wine32 winecfg
WINE=wine32 WINEPREFIX=~/wineprefixes/wine32 winetricks dotnet48 corefonts
WINEPREFIX=~/wineprefixes/wine32 wine32 /percorso/al/programma.exe
```

Da quel momento tutti i programmi a 32 bit vanno lanciati indicando quel prefix, e questa è la ragione pratica per cui conviene fissare da subito una convenzione di percorsi dei prefix invece di improvvisarli.

Un avvertimento sulla forma dei percorsi. Il documento sorgente proponeva in un punto `WINEPREFIX=~/.wine/wine32`, cioè un prefix annidato dentro un altro prefix. Va evitato: un prefix è la radice di un albero `drive_c`, e metterne uno dentro l'albero di un altro confonde entrambi. La forma corretta è una cartella sorella, per esempio sotto `~/wineprefixes/`, ed è quella usata in tutta questa documentazione.

La seconda causa sono dipendenze di sistema mancanti, in particolare le librerie a 32 bit su un sistema a 64 bit.

```bash
sudo apt install libwine libwine:i386 fonts-wine
```

Il suffisso `:i386` è la sintassi con cui si chiede la variante a 32 bit di un pacchetto su un sistema multiarchitettura, e funziona soltanto se l'architettura è stata dichiarata prima con `dpkg --add-architecture i386`. Se quel passo manca, il pacchetto risulta introvabile e l'errore che si legge parla di pacchetto inesistente, non di architettura mancante, il che rende la diagnosi meno ovvia di quanto sia.

Vale segnalare che la disponibilità dei nomi `libwine` e `libwine:i386` dipende da come la distribuzione confeziona Wine in quel rilascio, e cambia fra le versioni di Ubuntu. Su una installazione nuova conviene verificare i nomi effettivi con una ricerca nei pacchetti disponibili invece di assumerli.

## Errore generico: l'eseguibile a 32 bit non parte con Wine a 64 bit

La maggior parte dei problemi su `kernel32.dll` deriva dal caso più semplice: l'eseguibile è a 32 bit e il Wine predefinito del sistema è a 64. La verifica è immediata.

```bash
wine64 --version
wine32 --version
file /percorso/al/programma.exe
```

Se il secondo comando non risponde, il binario a 32 bit non è installato. Se il terzo dichiara un eseguibile PE a 32 bit mentre solo `wine64` è presente, la causa è quella. La cura è installare i pacchetti mancanti e creare un prefix a 32 bit pulito con i runtime corretti, che risolve quasi sempre.

## Provenienza dei pacchetti e stabilità

I repository di Ubuntu dividono Wine in pacchetti separati e non sempre offrono la versione più recente. Per stabilità, in particolare sul supporto a .NET che è la dipendenza critica di tutti i programmi di questo progetto, il repository ufficiale di WineHQ è preferibile. Su Ubuntu Studio conviene assicurarsi di essere almeno alla versione 9.0 di Wine; la versione osservata sulla macchina era `wine-9.0 (Ubuntu 9.0~repack-4build3)`, quindi al limite inferiore accettabile e proveniente dai repository della distribuzione.

Su una installazione nuova la decisione va presa una volta e non mescolata, perché avere librerie provenienti da due fonti diverse è una causa di incoerenza difficile da diagnosticare a posteriori.

## Nota sul rapporto con l'aggiornamento del sistema

I guasti di questa pagina hanno un legame diretto con la pagina sull'aggiornamento alla LTS, e riconoscerlo cambia l'ordine con cui conviene affrontare le cose. Un ambiente Wine sedimentato, con architettura `i386` aggiunta a mano, pacchetti da due provenienze e prefix creati in momenti diversi, è sia la fonte di questi errori sia uno dei fattori che rendono fragile un aggiornamento di rilascio. Ricostruire l'ambiente Wine da zero su un sistema appena installato risolve i due problemi in una volta, ed è la ragione principale per cui la strada della installazione pulita è quella raccomandata.
