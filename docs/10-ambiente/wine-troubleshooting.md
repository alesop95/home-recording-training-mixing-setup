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

## Errore: no driver could be loaded, lanciando da una sessione SSH

Il sintomo è una coppia di righe che compare subito dopo l'avvio e prima che nulla appaia a schermo, seguita da una eccezione non gestita che apre il debugger.

```
err:winediag:nodrv_CreateWindow Application tried to create a window, but no driver could be loaded.
err:winediag:nodrv_CreateWindow L"The explorer process failed to start."
```

Il messaggio va letto per quello che dice e non per quello che sembra. Non dice che manchi un driver grafico sul sistema, e non dice che il prefix sia rotto: dice che Wine, dovendo creare una finestra, non ha trovato alcun driver che sapesse dove disegnarla. La differenza è fra una assenza e una indicazione mancante, ed è la stessa distinzione per cui un programma non trova un file quando il percorso è vuoto invece che quando il file è stato cancellato.

La diagnosi va fatta su quattro fatti, e conviene raccoglierli tutti prima di toccare qualcosa, perché tre di essi escludono le cause che verrebbero in mente per prime. I driver esistono, e per l'architettura giusta: `find /usr/lib -name "winex11.drv*" -o -name "winewayland.drv*"` li trova sotto `i386-windows` e non solo sotto `x86_64-windows`, quindi il ramo a 32 bit è completo. La sessione grafica è viva: `loginctl list-sessions` mostra una sessione di tipo `wayland` su `seat0`, attiva. I socket ci sono entrambi, cioè `wayland-0` dentro `/run/user/1000` e `X0` dentro `/tmp/.X11-unix`. E il registro del prefix non dichiara alcun driver grafico, quindi non c'è una scelta sbagliata scritta da qualche parte: la chiave `Software\Wine\Drivers` contiene le sole voci di `winepulse.drv`, che riguardano l'audio.

Il quinto fatto è la causa. In una sessione aperta via SSH le variabili `DISPLAY` e `WAYLAND_DISPLAY` sono entrambe vuote, mentre `XDG_RUNTIME_DIR` vale `/run/user/1000` e `XDG_SESSION_TYPE` vale `tty`. Wine non ha quindi alcun indirizzo a cui mandare la finestra, e lo dichiara nel solo modo che conosce.

*Una spiegazione precedente, qui ritirata.* Il 2026-09-09 questo progetto aveva registrato che una sessione SSH dispone comunque di un display, perché la libreria di Wayland ricadrebbe sul socket predefinito dentro `XDG_RUNTIME_DIR`, che la sessione remota eredita. La prova del 2026-09-14 smentisce quella spiegazione: `XDG_RUNTIME_DIR` è impostata, il socket `wayland-0` esiste, e l'avvio fallisce lo stesso. Anche impostare a mano `WAYLAND_DISPLAY=wayland-0` non basta, e l'errore resta identico. Perché la via Wayland non funzioni su questa installazione non è accertato e non va supposto; il modo di accertarlo, se un giorno servisse, è dichiarare esplicitamente il driver nel registro del prefix e osservare che cosa cambia. Non serve adesso, perché la via X11 funziona.

La causa dell'errore di allora non era dunque quella scritta. La spiegazione era plausibile e si adattava a ciò che si era visto, cioè una finestra comparsa; non era però stata messa alla prova, e alla prima occasione in cui avrebbe dovuto predire un esito ha predetto quello sbagliato. La lezione generale è che una spiegazione che si adatta a una osservazione non è verificata finché non ne predice una seconda, e che il momento di metterla alla prova è quello in cui la si scrive, non quello in cui fallisce.

*La forma che funziona.* Servono due variabili insieme, e nessuna delle due da sola basta. La prima è `DISPLAY`, che dice a quale server X mandare la finestra, e su questa macchina è `:0`, cioè XWayland dentro la sessione Plasma. La seconda è `XAUTHORITY`, che dice dove sta il biscotto di autorizzazione senza il quale quel server rifiuta la connessione: la sessione lo scrive in `/run/user/1000/` con un nome che contiene una parte casuale, quindi va ricavato e non trascritto.

```bash
WINEPREFIX=$HOME/.wine DISPLAY=:0 XAUTHORITY=$(ls -1 /run/user/$(id -u)/xauth_* | head -1) wine32 "C:/Program Files/RDTeam/AKABAK/AKABAK.exe"
```

La parte che ricava il file di autorizzazione non è pignoleria: quel nome cambia a ogni nuovo accesso grafico, quindi un comando che lo trascrive funziona oggi e fallisce dopo il primo riavvio, e fallisce con un errore diverso, cioè un rifiuto di connessione al server X invece che l'assenza di driver, il che manderebbe la diagnosi su una pista sbagliata.

*Quando questo problema non si presenta.* Lanciando dal terminale della macchina dentro la sessione grafica, oppure cliccando i lanciatori sulla scrivania, le due variabili sono già impostate dalla sessione e il comando nudo funziona. Il guasto riguarda quindi il solo lavoro da remoto, che è però il modo in cui questo progetto amministra la macchina.

## Errore: un programma .NET fallisce dentro System.Drawing prima di mostrare una finestra

Il sintomo è un programma che non apre nulla e termina con una eccezione la cui catena di chiamate finisce dentro una classe grafica della libreria standard di .NET, tipicamente il costruttore di `System.Drawing.Icon` o `System.Drawing.Bitmap`. Il messaggio cambia a seconda del runtime installato nel prefix, e i due che si incontrano sono questi.

```
System.ArgumentException: A null reference or invalid value was found [GDI+ status: InvalidParameter]
System.Runtime.InteropServices.ExternalException: A generic error occurred in GDI+
```

La prima forma la produce Wine Mono, la seconda il .NET Framework di Microsoft installato con `winetricks -q dotnet48`. Vedere cambiare il messaggio dopo avere installato il framework è un dato utile e va letto per quello che è: dimostra che il runtime è davvero cambiato, quindi che l'installazione è riuscita, e insieme che il framework non era la causa del guasto. La catena delle chiamate, se si confrontano le due eccezioni riga per riga, resta identica fino all'ultimo elemento.

La causa è lo strato sottostante. GDI+[^gdi] è il sottosistema grafico di Windows su cui `System.Drawing` poggia, e nessuno dei due runtime .NET ne porta uno proprio: entrambi si appoggiano a quello del sistema, che sotto Wine è la reimplementazione di Wine. Un programma che esca dai sentieri battuti, per esempio caricando una icona da un vettore di byte invece che da un file, incontra allora uno scarto fra la reimplementazione e l'originale, e quello scarto si manifesta come un errore generico perché è tutto ciò che la funzione chiamante sa dire.

Il rimedio è sostituire la libreria di Wine con quella originale di Windows.

```bash
WINEPREFIX=~/wineprefixes/<nome del prefix> winetricks -q gdiplus
```

Due cose vanno sapute prima di lanciarlo, perché altrimenti sorprendono. La prima è il costo: il verbo non scarica una libreria ma i due pacchetti di aggiornamento di Windows 7 SP1, uno per architettura, per circa 1,8 GB complessivi, e da ciascuno estrae il solo `gdiplus.dll`, che pesa fra 1,5 e 2,1 MB. Lo scaricamento finisce nella cache di `winetricks` dentro la cartella dell'utente, quindi si paga una volta sola e i prefix successivi lo riusano. La seconda è che le architetture installate sono due, cioè il file a 32 bit in `C:\windows\syswow64` e quello a 64 bit in `C:\windows\system32`, e questo è precisamente ciò che serve in un prefix a 64 bit che ospiti un programma a 32, che è il caso più comune fra i programmi di elettroacustica.

La verifica è il riavvio del programma, e conviene farla su tre prove invece che su una, perché da remoto la prima e la seconda ingannano. L'elenco delle finestre con `wmctrl -l` dice che una finestra esiste, ma può elencare anche finestre di crash rimaste aperte da tentativi precedenti, quindi va letta insieme al processo a cui appartengono. La cattura della finestra con `import` mostra che l'interfaccia è disegnata, ed è la prova visiva. La terza è la più forte e va cercata sempre: molti programmi .NET scrivono un proprio registro degli errori sotto `AppData\Local`, e un avvio riuscito è esattamente un avvio che in quel file non aggiunge nulla.

Il caso reale da cui questa scheda nasce è EASE Focus 3.1.260, dove il framework era necessario e non sufficiente, ed è raccontato in MS-108, MS-109 e MS-110 del registro dei microstep.

[^gdi]: *GDI+*, Graphics Device Interface Plus - il sottosistema grafico di Windows per il disegno bidimensionale, le immagini e i caratteri; Wine ne fornisce una reimplementazione libera, che è sufficiente per la gran parte dei programmi e non per tutti.

## Provenienza dei pacchetti e stabilità

I repository di Ubuntu dividono Wine in pacchetti separati e non sempre offrono la versione più recente. Per stabilità, in particolare sul supporto a .NET che è la dipendenza critica di tutti i programmi di questo progetto, il repository ufficiale di WineHQ è preferibile. Su Ubuntu Studio conviene assicurarsi di essere almeno alla versione 9.0 di Wine; la versione osservata sulla macchina era `wine-9.0 (Ubuntu 9.0~repack-4build3)`, quindi al limite inferiore accettabile e proveniente dai repository della distribuzione.

Su una installazione nuova la decisione va presa una volta e non mescolata, perché avere librerie provenienti da due fonti diverse è una causa di incoerenza difficile da diagnosticare a posteriori.

## Nota sul rapporto con l'aggiornamento del sistema

I guasti di questa pagina hanno un legame diretto con la pagina sull'aggiornamento alla LTS, e riconoscerlo cambia l'ordine con cui conviene affrontare le cose. Un ambiente Wine sedimentato, con architettura `i386` aggiunta a mano, pacchetti da due provenienze e prefix creati in momenti diversi, è sia la fonte di questi errori sia uno dei fattori che rendono fragile un aggiornamento di rilascio. Ricostruire l'ambiente Wine da zero su un sistema appena installato risolve i due problemi in una volta, ed è la ragione principale per cui la strada della installazione pulita è quella raccomandata.
