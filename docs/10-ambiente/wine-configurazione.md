<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Configurazione di Wine da zero

> Procedura di installazione pulita di Wine su Ubuntu Studio, con il razionale di ogni passo. È la sequenza da seguire dopo una reinstallazione del sistema, e la stessa che risolve un prefix corrotto. I comandi contengono operazioni distruttive segnalate come tali.

## Azzerare quello che c'è

Se sulla macchina esistono già prefix Wine, il primo passo è metterli da parte invece di cancellarli, così che un ambiente funzionante resti recuperabile.

```bash
mv ~/.wine ~/.wine.bak
mv ~/.wine32 ~/.wine32.bak
```

Il secondo comando serve solo se in passato si era provato a creare un prefix a 32 bit; se la cartella non esiste il comando fallisce senza conseguenze.

Poi si rimuovono le versioni di Wine eventualmente installate dai repository, con la loro configurazione.

```bash
sudo apt remove --purge wine* -y
sudo apt autoremove -y
```

## Aggiungere l'architettura a 32 bit

Wine ha bisogno di librerie a 32 bit anche su un sistema a 64 bit, e su Debian e derivate questo richiede di dichiarare esplicitamente l'architettura secondaria prima di installare qualsiasi cosa.

```bash
sudo dpkg --add-architecture i386
sudo apt update
```

Questo passo va fatto prima dell'installazione, non dopo: se si installa Wine e poi si aggiunge l'architettura, i pacchetti a 32 bit non vengono tirati dentro automaticamente e si finisce con un ambiente incompleto. È esattamente la sequenza sbagliata che sulla macchina ha prodotto il guasto descritto nella pagina di troubleshooting.

## Installare Wine

Ci sono due strade, e la differenza va capita perché produce ambienti diversi.

La prima installa il metapacchetto stabile con le dipendenze raccomandate.

```bash
sudo apt install --install-recommends wine-stable
```

L'opzione `--install-recommends` tira dentro le dipendenze consigliate ed è quella che rende l'installazione completa invece di minimale. Questa strada di norma fornisce Wine a 64 bit; non garantisce che tutto il necessario per eseguire applicazioni a 32 bit sia presente senza passare da `winetricks`.

La seconda installa esplicitamente i due binari e lo strumento di configurazione delle dipendenze.

```bash
sudo apt install wine64 wine32 winetricks
```

Questa strada copre subito entrambe le architetture e include `winetricks` da principio. È la scelta più prudente su una macchina dove si sa già che servirà almeno un programma a 32 bit, come WinISD.

Le verifiche dopo l'installazione sono tre, e ciascuna dice una cosa diversa.

```bash
wine --version
which wine
wine64 --version
wine32 --version
```

Il primo comando dà la versione, e su Ubuntu Studio conviene assicurarsi che sia almeno la 9.0. Il secondo mostra dove sta il binario, tipicamente sotto `/usr/bin`. Il terzo e il quarto dicono quali architetture sono effettivamente installate: se `wine32 --version` non risponde, il binario a 32 bit non c'è, ed è la spiegazione più frequente degli errori su `kernel32.dll` quando si tenta di lanciare un eseguibile a 32 bit.

Una nota sulla provenienza dei pacchetti. I repository di Ubuntu dividono Wine in pacchetti separati e non offrono sempre la versione più recente; il repository ufficiale di WineHQ è preferibile per stabilità, in particolare per il supporto a .NET, che è la dipendenza critica di VituixCAD e di EASE Focus, non di tutti i programmi del progetto. Su una installazione nuova conviene decidere subito quale delle due fonti si usa e non mescolarle, perché la mescolanza è una fonte classica di librerie incoerenti.

## Configurare il prefix

Lo strumento di configurazione si chiama `winecfg` e lavora sempre su un prefix: se non se ne indica uno, agisce su `~/.wine`, creandolo se non esiste.

```bash
winecfg
WINEARCH=win32 WINEPREFIX=~/wineprefixes/akabak32 winecfg
```

Le due impostazioni che contano per i programmi di questo progetto sono nella scheda delle applicazioni e in quella della grafica. Nella prima si imposta la versione di Windows su *Windows 10*. Nella seconda si attiva l'opzione che permette al window manager di decorare le finestre, che evita i problemi di rendering più comuni.

La scelta di Windows 10 merita una riga di spiegazione, perché è controintuitiva rispetto all'idea che una versione più vecchia sia più compatibile. I programmi audio e di simulazione moderni, VituixCAD ed EASE Focus fra questi, sono collaudati solo su profili Windows 10 e 11 e chiedono API recenti come .NET 4.8, le librerie WinMM e WASAPI, che i profili Windows 7 o inferiori non espongono. Akabak non è fra questi, per ADR-016: nel prefix funzionante non c'è .NET, quindi il profilo non gli serve per una dipendenza. Va però detto che il prefix funzionante quel profilo lo ha davvero, perché Akabak riporta nelle sue informazioni il sistema come `NT 10.0 (Build 19043)`, cioè Windows 10: impostarlo è quindi riprodurre la configurazione misurata, non un atto di uniformità e nemmeno una scelta priva di riscontro. Wine sa dichiararsi anche Windows 11, ma il profilo Windows 10 ha compatibilità migliore e meno difetti noti, quindi è la scelta corretta e non un compromesso.

## Installare le dipendenze con winetricks

Le dipendenze si installano nel prefix in cui servono, mai globalmente, ed è questo il senso di avere prefix separati.

Va detto subito, perché rovescia quanto il documento sorgente prescriveva: per Akabak e VACS **non serve alcuna dipendenza**. Il prefix funzionante sulla macchina non ha `winetricks.log`, non ha .NET e non ha font Microsoft di base, e i due programmi girano. La lista di dipendenze del sorgente descriveva i tentativi del troubleshooting, non ciò che serviva. Il dettaglio è in ADR-016.

```bash
sudo apt install winetricks
WINEPREFIX=~/wineprefixes/vituixcad64 winetricks -q dotnet48 corefonts
WINEPREFIX=~/wineprefixes/winisd32 winetricks -q vcrun6 corefonts
```

L'opzione `-q` esegue in modalità non interattiva, cioè accetta automaticamente le finestre di installazione dei pacchetti Microsoft, e su una installazione da zero risparmia una quantità notevole di clic.

Il significato dei pacchetti usati in questo progetto è il seguente. Il pacchetto `corefonts` installa i font Microsoft di base, cioè Arial, Times New Roman e Verdana, che molte applicazioni si aspettano di trovare e in assenza dei quali le finestre si disegnano male o non si disegnano. Il pacchetto `dotnet48` installa .NET Framework 4.8, richiesto da VituixCAD ed EASE Focus per dichiarazione dei rispettivi produttori. **Non** da Akabak, contrariamente a quanto il documento sorgente affermava: nel prefix funzionante non è installato. Il pacchetto `vcrun6` installa i runtime di Visual C++ 6.0, richiesti dai programmi legacy compilati con quel compilatore, fra cui WinISD. Il pacchetto `vcrun2015` installa i runtime di Visual C++ 2015, cioè `MSVCP140.dll` e `VCRUNTIME140.dll`, indispensabili alle applicazioni compilate con Visual Studio 2015. Il pacchetto `vcrun2019` copre l'equivalente per le versioni più recenti, e su Akabak e VituixCAD serve nei casi in cui il programma segnala dipendenze mancanti. Il pacchetto `dotnet20` serve soltanto ad alcune build sperimentali di WinISD. Il pacchetto `dxvk` traduce le chiamate Direct3D in Vulkan e si aggiunge solo se il rendering di una finestra risulta lento, che per il disegno bidimensionale di questi programmi di norma non è il caso.

## Installare e lanciare un programma

L'installazione di un eseguibile Windows si fa posizionandosi nella cartella dove sta l'installer e passandolo a Wine.

```bash
cd ~/Downloads
WINEPREFIX=~/wineprefixes/akabak32 wine AKABAK_Pro_v324b126.exe
```

Il lancio successivo punta all'eseguibile installato dentro l'albero del prefix, ricordando che il percorso Windows che il programma dichiara corrisponde a un percorso Linux dentro `drive_c`.

```bash
WINEPREFIX=~/wineprefixes/akabak32 wine "C:/Program Files/RDTeam/AKABAK/AKABAK.exe"
```

La forma con il percorso Windows fra apici doppi è preferibile alla forma con il percorso Linux, perché evita di dover proteggere gli spazi nei nomi di cartella come `Program Files`.

Per l'uso quotidiano conviene creare un lanciatore sul desktop, che nell'ambiente grafico di Ubuntu Studio si ottiene creando un collegamento e scegliendo di crearne un lanciatore con il nome del programma.
