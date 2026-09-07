<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# I programmi Windows del progetto sotto Wine

> Una scheda per programma, con architettura, dipendenze, prefix di destinazione e sequenza di installazione verificata. Le sequenze presuppongono che Wine e winetricks siano già installati come descritto nella pagina di configurazione.

## Akabak 3

Akabak è un simulatore di sistemi elettro-meccanico-acustici che combina il metodo degli elementi al contorno, cioè BEM[^1], e il modello a elementi concentrati, cioè LEM[^2]. È lo strumento su cui il progetto converge, perché è l'unico gratuito che permette di simulare in un solo passaggio la parte elettroacustica del diffusore e l'acustica dell'ambiente, e quindi di prevedere l'effetto dei monitor nella stanza invece di dedurlo.

Va chiarito un punto su cui il documento sorgente è internamente contraddittorio, perché è il tipo di ambiguità che manda fuori strada. Esistono due famiglie di Akabak: la 1.x storica, a 16 bit e testuale, ancora usata in ambito DIY per i modelli a tromba, e le 2.x e 3.x moderne, a 64 bit, con interfaccia grafica e scripting. Il sorgente in un punto descrive Akabak come software a 16 bit e in un altro sceglie correttamente la 3.x a 64 bit. La versione in uso in questo progetto è la 3.x professionale, a 64 bit, quindi le affermazioni sui 16 bit non le si applicano.

La versione scaricabile dal sito dell'autore non è una demo limitata nel tempo o nelle funzioni: è la versione completa, che richiede licenza per uso commerciale ed è gratuita per uso privato, hobbistico o di ricerca senza scopo di lucro, previa registrazione. La licenza di questo progetto è una student license concessa dall'autore, e il dettaglio della procedura sta nella pagina delle licenze.

L'architettura è 64 bit senza build alternativa, i requisiti sono Windows 10 o 11 a 64 bit con .NET 4.8 e librerie grafiche e audio moderne, quindi serve un prefix a 64 bit con profilo Windows 10.

```bash
WINEARCH=win64 WINEPREFIX=~/wineprefixes/akabak64 winecfg
WINEPREFIX=~/wineprefixes/akabak64 winetricks -q dotnet48 vcrun2019 corefonts
cd ~/Downloads
WINEPREFIX=~/wineprefixes/akabak64 wine setup_akabak3_x64.exe
```

All'installer si può accettare il percorso predefinito. Il programma finisce sotto `Program Files`, in una cartella dell'autore, che sul filesystem Linux corrisponde a un percorso dentro `drive_c` del prefix. La verifica che l'installazione sia riuscita è aprire la finestra principale senza errori, caricare uno dei file di esempio e lanciare una simulazione, controllando che i grafici di risposta in frequenza si disegnino.

Gli esempi si scaricano a parte dal sito dell'autore e si scompattano nella cartella degli esempi, che si può creare da dentro l'applicazione. Il pacchetto degli esempi è il file più pesante dell'intero corredo, ed è uno dei motivi per cui il materiale binario di questo progetto va spostato sulla macchina di destinazione invece di restare sul disco di sviluppo.

Non serve installare anche Wine a 32 bit per Akabak. Con il solo Wine a 64 bit e `dotnet48` il programma funziona; l'installazione di `wine32` copre eventuali librerie o plugin a 32 bit che Akabak non usa direttamente, quindi per questa applicazione specifica non è necessaria.

## VACS

VACS è lo strumento di visualizzazione e analisi dei dati che l'autore di Akabak distribuisce insieme al simulatore, e la student license concessa copre entrambi con un solo Release Code. La versione dichiarata dall'autore è la 2.1.3 build 33, coerente con i nomi degli installer conservati. È distribuito in due varianti, a 32 e a 64 bit, e sulla macchina è disponibile in entrambe.

La variante da installare è quella a 64 bit, nello stesso prefix di Akabak, perché i due programmi si usano in sequenza e condividono le dipendenze. La variante a 32 bit resta come riserva per il caso in cui la prima dia problemi, e in quel caso va in un prefix a 32 bit separato, non nello stesso.

Va registrato un precedente utile, perché è il tipo di guasto che si ripresenta. Al primo impianto, nell'agosto 2025, Akabak partì e VACS no, e l'ipotesi formulata sul momento fu che dipendesse dall'aver usato la variante a 64 bit invece di quella a 32. Il problema fu risolto entro il giorno successivo, ma la corrispondenza non registra quale intervento lo abbia risolto, quindi non lo si sa: potrebbe essere stata l'installazione della variante a 32 bit, una dipendenza aggiunta con winetricks, o la ricreazione del prefix. È una lacuna dichiarata e non riempita per ipotesi, e si chiude soltanto ispezionando la macchina attuale, come previsto dalla fase 0.5 di `installazione-pulita-26-04.md`.

## Il limite delle pipeline COM fra Akabak e VACS

Questo non è un guasto da risolvere ma una proprietà dell'ambiente, dichiarata dall'autore del software, e va conosciuta prima di iniziare a lavorare invece di scoprirla alla prima iterazione.

Su Windows i due programmi si scambiano i dati attraverso COM, l'infrastruttura con cui due processi distinti espongono e invocano oggetti fra loro: Akabak consegna i risultati e VACS li riceve senza che l'utente faccia nulla. Wine implementa COM soltanto in parte, e l'autore ha constatato che su Linux quel canale non funziona per i suoi programmi. La via che indica è quella degli appunti di sistema: si copia da Akabak e si incolla in VACS, con l'impostazione relativa nelle preferenze di Akabak.

La conseguenza pratica è un passo manuale a ogni passaggio di dati, e le iterazioni della fase 5 del progetto sono molte, perché il ciclo consiste nel modificare parametri e materiali finché la risposta non rientra nella tolleranza dichiarata. Conviene quindi adottare una disciplina di denominazione dei risultati e verificare a ogni incollaggio che il dato sia quello atteso, perché un passo manuale ripetuto è il punto in cui si incolla per errore il risultato della simulazione precedente.

Lo storico completo di questa constatazione, con la cronologia della corrispondenza, sta in `docs/90-riferimenti/timeline-akabak-vacs.md`.

## VituixCAD 2

VituixCAD è lo strumento di progettazione del crossover e della direttività, e nel workflow occupa la fase 4a. È nativo a 64 bit, richiede .NET Framework 4.8 e librerie Windows moderne, cioè WinForms, GDI+ e WinMM per l'audio, e gira bene soltanto in un prefix a 64 bit. Il vincolo sui 64 bit non è una preferenza: alcune versioni di .NET 4.8 non si installano correttamente nei prefix a 32 bit, quindi la scelta è obbligata.

```bash
WINEARCH=win64 WINEPREFIX=~/wineprefixes/vituixcad64 winecfg
WINEPREFIX=~/wineprefixes/vituixcad64 winetricks -q dotnet48 corefonts
cd ~/Downloads
WINEPREFIX=~/wineprefixes/vituixcad64 wine VituixCAD_setup.exe
```

Il pacchetto `vcrun2019` si aggiunge solo se il programma, al primo avvio, segnala dipendenze mancanti: installarlo preventivamente non fa danno ma non serve.

## WinISD

WinISD serve all'ottimizzazione del volume e dell'accordo bass reflex, e nel workflow occupa la fase 4c. Va detto subito che è in buona parte ridondante: se si usano già VituixCAD per il box e il crossover e Akabak per la verifica finale, WinISD non aggiunge nulla di essenziale, ed è per questo che nel workflow è marcato come opzionale.

Se si vuole comunque averlo, l'applicazione è vecchia e distribuita solo a 32 bit, quindi va in un prefix a 32 bit, dove le librerie datate e i runtime ridotti si installano senza problemi. Metterlo in un prefix a 64 bit fa correre complicazioni inutili senza alcun vantaggio.

```bash
WINEARCH=win32 WINEPREFIX=~/wineprefixes/winisd32 winecfg
WINEPREFIX=~/wineprefixes/winisd32 winetricks -q vcrun6 corefonts
```

Alcune build sperimentali richiedono .NET 2.0 invece dei soli runtime Visual C++, e in quel caso si aggiunge `dotnet20` nello stesso prefix.

## EASE Focus 3

EASE Focus è lo strumento di simulazione della copertura acustica dei diffusori, e usa i file GLL[^3], che contengono il modello acustico fornito dal costruttore con dati di direttività, SPL in funzione della frequenza ed equalizzazioni preimpostate. Il progetto ne ha tre versioni sul disco, 3.0.18, 3.1.10 e 3.1.260, con un database di GLL di dimensioni notevoli, e la pagina della fase di modellazione discute quale ha senso usare e per cosa.

Dal punto di vista dell'ambiente il programma è basato su .NET 4.x, e i requisiti reali sotto Wine sono più larghi di quanto la documentazione ufficiale lasci intendere: serve .NET Framework 4.0 o superiore e servono i redistributable di Visual C++ almeno nelle versioni 2010, 2013 e 2015, perché alcuni moduli GLL portano DLL proprie compilate con quei compilatori.

Ci sono due modi di procedere, e la differenza è di manutenzione, non di funzionamento.

Il primo è installarlo nel prefix principale già attrezzato, come è stato fatto per Akabak. Se `~/.wine` ha già `dotnet48`, `corefonts`, `vcrun6` e `vcrun2015` non c'è alcun blocco tecnico. Il rischio è quello di ogni prefix condiviso: un programma futuro che chieda una versione diversa di .NET o di una DLL può rompere tutti gli altri.

Il secondo è un prefix dedicato, che costa spazio su disco e qualche variabile in più nei comandi ma isola il programma.

```bash
WINEPREFIX=~/wine-ease wineboot --init
WINEPREFIX=~/wine-ease winetricks -q dotnet48 corefonts
cd ~/Downloads
WINEPREFIX=~/wine-ease wine EASE_Focus_Setup_v3.1.260.exe
WINEPREFIX=~/wine-ease wine "C:/Program Files/EASE Focus 3/EASE Focus 3.exe"
```

La buona pratica generale resta il prefix separato per ogni programma. Che con Akabak sia andata liscia nel prefix condiviso dipende dal fatto che usa dipendenze simili a quelle già presenti e non porta librerie audio o video particolari; non è una garanzia che valga per il programma successivo.

Il database dei GLL va copiato nella cartella che il programma usa per cercarli, che dentro il prefix corrisponde a un percorso sotto i documenti dell'utente, nella cartella del programma. Un dettaglio che fa perdere tempo se non lo si sa: alcuni costruttori distribuiscono un `.gll` accompagnato da file `.dll` e `.bin`, e tutti e tre devono restare nella stessa cartella, altrimenti il modulo non si carica.

Se il rendering della finestra risulta lento si può abilitare la traduzione Direct3D verso Vulkan con `winetricks dxvk`, ma per il disegno bidimensionale di questo programma di norma non serve. Sul kernel a bassa latenza di Ubuntu Studio il calcolo SPL non ha dato problemi.

## Da verificare

Un punto resta aperto dal documento sorgente e va tenuto come tale: quali altri programmi del corredo richiedono le stesse DLL e le stesse versioni di .NET, e quindi potrebbero condividere un prefix invece di averne uno per uno. La risposta si ottiene solo provando sulla macchina, e finché non è provata la scelta prudente resta un prefix per programma.

[^1]: *BEM*, Boundary Element Method - metodo numerico che risolve il campo acustico discretizzando le sole superfici del modello invece del volume, adatto alla diffrazione sul pannello frontale e alla dispersione.

[^2]: *LEM*, Lumped Element Model - rappresentazione di un sistema acustico come circuito equivalente a componenti concentrati, adatta al comportamento a bassa frequenza e alla simulazione dei filtri.

[^3]: *GLL*, Generic Loudspeaker Library - formato di AFMG che incapsula il modello acustico completo di un diffusore, distribuito dai costruttori e utilizzabile senza attivazione ulteriore.
