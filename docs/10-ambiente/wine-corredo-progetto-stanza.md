<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Il corredo "Progetto stanza": inventario verificato e piano Wine

> Piano di messa in opera sotto Wine del corredo software raccolto nella cartella `Progetto stanza (software)`. A differenza dell'inventario derivato dal file di testo, questa pagina nasce dall'ispezione diretta dei file: per ogni programma dichiara il formato reale dell'eseguibile, l'architettura, il prefix di destinazione, le dipendenze e lo stato di licenza accertato. Due delle informazioni qui raccolte modificano decisioni già prese altrove, e sono segnalate come tali. Una inferenza di una versione precedente di questa pagina è stata smentita da una verifica successiva ed è ritirata esplicitamente invece di essere cancellata in silenzio.

## Da dove viene questo corredo, e le quattro posizioni

Il materiale esiste in quattro luoghi, e distinguerli evita di cancellare la copia sbagliata.

La copia di lavoro, che è quella ispezionata per scrivere questa pagina, sta sul Desktop della postazione Windows, sotto `C:\Users\Utente\Desktop\Progetto stanza (software)`, e pesa 2,3 GB.

La copia ridondante sta su un SSD esterno, sotto `J:\Progetto stanza (software)`. La corrispondenza uno a uno con la copia sul Desktop, dichiarata dall'utente, è stata verificata il 2026-09-07 quando il disco è stato collegato, e risulta *confermata*: 650 file su entrambe le copie, gli stessi nomi, le stesse dimensioni, 2.380.021.546 byte in totale su ciascuna, e le impronte SHA-256 di tutti e 650 i file coincidenti, senza alcun file presente su una sola delle due. Va segnalato un tranello nel metodo: `du -sh` riportava dimensioni sensibilmente diverse fra le due copie, per esempio 14 MB contro 5,9 MB su LSPCad 5.25, e quella differenza non era reale ma dovuta alla dimensione dei cluster dei due filesystem, che arrotonda lo spazio occupato da ogni file. È la dimostrazione pratica del perché il criterio di confronto deve essere l'impronta del contenuto e non lo spazio occupato.

L'inventario testuale, il file `(SSD S7) DIY Loudspeaker Pack Softwares.txt` nella radice di questo progetto, descrive l'albero della copia su SSD. È il documento da cui era stato ricavato il primo inventario.

Va corretta qui una inferenza sbagliata di una versione precedente di questa pagina. Osservando che nella copia sul Desktop la versione 3.1.10 di EASE Focus era un collegamento e non una cartella, si era concluso che il suo contenuto vivesse presumibilmente sul solo SSD. È falso: il collegamento c'è su entrambe le copie, identico, e non è una divergenza fra loro. La lettura del collegamento ha rivelato dove punta davvero, ed è la scoperta della sezione seguente.

Esiste poi una quarta posizione, che nessuna delle tre precedenti dichiarava e che è emersa leggendo il contenuto del collegamento. Il materiale di EASE Focus 3.1.10 del workshop K-array risiede su un disco `G:`, al percorso `G:\LIBRARY\LOUDSPEAKERS & ELECTROACOUSTIC\K-ARRAY WORKSHOP\EASE Focus (k-array)`. Quel disco non era collegato al momento della verifica, quindi il contenuto non è stato ispezionato e non si sa che cosa contenga oltre a quanto il documento sorgente descriveva, cioè installer, alcuni GLL e i progetti di esempio del workshop. È tracciato come PA-004, e la priorità è bassa perché la 3.1.10 non è la versione da installare.

La destinazione finale è la macchina Ubuntu Studio, dove il sottoinsieme utile va installato sotto Wine. La cancellazione della copia su SSD è una azione differita e tracciata come PA-001 in `docs/PENDING-ACTIONS.md`: delle sue tre condizioni di sblocco, quella sulla corrispondenza fra le copie è ora soddisfatta, e resta il completamento verificato del trasferimento verso la macchina.

## L'inventario verificato

La tabella riporta il formato reale di ciascun installer, letto con `file`, e non quello dedotto dal nome. La colonna dell'architettura è quella dell'installer, che non sempre coincide con quella dell'applicazione installata: è una distinzione che confonde spesso, e la sezione successiva la spiega.

| Programma | Installer | Formato reale | Dimensione | Stato di licenza |
|---|---|---|---|---|
| VituixCAD 2 | `VituixCAD_setup.exe` | PE32 i386, 32 bit | 796 KB | gratuito per l'uso previsto |
| ARTA 1.7.1 | `ArtaSetup171.exe` | PE32 i386, 32 bit | 6,3 MB | shareware, modalità dimostrativa senza registrazione |
| EASE Focus 3.1.260 | `setup.exe` con `EASE Focus 3.msi` e `Data1.cab` | PE32 i386, InstallShield | 58 MB | gratuito, licenza inclusa |
| EASE Focus 3.0.18 | `setup.exe` analogo | PE32 i386, InstallShield | 45 MB | gratuito, superato dalla 3.1.260 |
| Database GLL 2016 | dati, nessun installer | 221 file, di cui 174 `.gll` | 451 MB | i GLL sono licenziati dai costruttori |
| Ramsete 27b | `setup.exe` con `Ramsete27b.CAB` e `SETUP.LST` | PE32 i386, toolkit Visual Basic 6 | 9,7 MB | da verificare, si veda sotto |
| FineCone 2.1 | `InstFC.exe` | PE32 i386, 32 bit | 66 MB | protezione rimossa |
| FineMotor 2.5 | `setup.exe` | PE32 i386, 32 bit | 21 MB | protezione rimossa |
| LSPCad 5.25 | `SETUP.EXE` | **NE, 16 bit, Windows 3.1** | 5,9 MB | protezione rimossa |
| LSPCad 6.32 | `install.exe` | PE32 i386, compresso UPX | 15 MB | protezione rimossa |
| Grenander Loudspeaker Lab 3.13 | `LspLAB313.exe` | PE32 i386, installer NSIS | 2,4 MB | protezione rimossa |
| AmpliTube 3 | `Install AmpliTube.exe` con sbloccatore separato | PE32 i386 | 531 MB | protezione rimossa |
| Guitar Rig 5 Pro | pacchetto dichiarato sbloccato | non ispezionato | 813 MB | protezione rimossa |
| POD Farm 1.11 | `PODFarmv1.11.0Installer.exe` | PE32 i386 | 251 MB | provenienza non lecita, richiede licenza Line 6 |

## L'architettura dell'installer non è quella dell'applicazione

Questo è il punto tecnico più utile della pagina, perché è la causa più comune di una scelta di prefix sbagliata.

Tutti gli installer del corredo sono a 32 bit, `VituixCAD_setup.exe` compreso. Eppure VituixCAD 2 è una applicazione .NET a 64 bit, e la pagina dei programmi Windows prescrive per essa un prefix a 64 bit. Le due cose non sono in contraddizione: un installer a 32 bit gira senza problemi in un prefix a 64 bit, perché Windows, e Wine con esso, mantiene uno strato di compatibilità che permette a un processo a 32 bit di girare su un sistema a 64. È lo stesso meccanismo per cui su Windows reale si installano ancora programmi con installer datati.

La regola corretta è quindi guardare l'architettura dell'applicazione installata, non quella del suo installer, e l'architettura dell'applicazione si desume dalla documentazione del produttore o si legge sull'eseguibile dopo l'installazione.

```bash
file ~/wineprefixes/vituixcad64/drive_c/Program\ Files/VituixCAD2/VituixCAD.exe
```

C'è una sola eccezione, ed è netta: un eseguibile a 16 bit non gira in un prefix a 64 bit. Su Wine il supporto ai 16 bit esiste soltanto nei prefix a 32 bit, perché deriva dallo stesso strato di compatibilità che i sistemi a 64 bit hanno rimosso. Nel corredo c'è esattamente un caso, ed è `SETUP.EXE` di LSPCad 5.25, riconoscibile dal formato NE dichiarato per Windows 3.1.

Vale segnalare una coincidenza che getta luce sull'incoerenza 2 del documento sorgente, dove Akabak era descritto come software a 16 bit. Nel corredo un programma genuinamente a 16 bit esiste, ed è LSPCad 5.25. È plausibile che l'attributo sia migrato da un programma all'altro durante la stesura degli appunti. Resta una spiegazione plausibile e non una causa accertata, e va letta come tale.

## Gruppo A: i programmi da installare

Sono quattro, più i dati. Per ciascuno il prefix di destinazione segue la regola di ADR-004, cioè un prefix per programma, con l'emendamento discusso nella sezione sui 32 bit.

### VituixCAD 2

Già previsto dalla procedura di installazione pulita, fase 8.5. Prefix `~/wineprefixes/vituixcad64` a 64 bit, con `dotnet48` e `corefonts`.

```bash
cd ~/electroacoustics/progetto-stanza/diy
WINEPREFIX=~/wineprefixes/vituixcad64 wine VituixCAD_setup.exe
```

### ARTA 1.7.1

ARTA non era nel piano originale e va aggiunto, perché ha un ruolo preciso nel workflow: è la via più diretta per produrre un file GLL da un diffusore misurato, quindi diventa pertinente alla fase 8 del progetto, quando il monitor autocostruito esiste e va caratterizzato. Fino a quel momento la misura la fa REW, nativo, e ARTA resta inutilizzato.

È shareware. Senza registrazione funziona in modalità dimostrativa, con limitazioni che vanno verificate al momento dell'uso e che non sono state accertate qui. La cartella contiene soltanto l'installer ufficiale, senza modifiche.

Prefix dedicato a 64 bit. Le dipendenze sono i runtime Visual C++ e i font di base; ARTA è un programma nativo Windows non gestito, quindi non richiede .NET.

```bash
WINEARCH=win64 WINEPREFIX=~/wineprefixes/arta64 winecfg
WINEPREFIX=~/wineprefixes/arta64 winetricks -q vcrun2019 corefonts
cd ~/electroacoustics/progetto-stanza/diy/Arta
WINEPREFIX=~/wineprefixes/arta64 wine ArtaSetup171.exe
```

C'è un punto da verificare sul campo e non da assumere: ARTA è un programma di misura, quindi vuole accedere alla scheda audio. Sotto Wine l'accesso passa dal driver audio di Wine verso PipeWire, e la latenza e la stabilità che ne risultano non sono quelle di un programma nativo. Per la produzione di un GLL da misure già acquisite il problema non si pone, perché si lavora su file; per una misura dal vivo conviene misurare in REW, che è nativo, ed esportare.

### EASE Focus 3.1.260 e il servizio di database AFMG

Qui l'ispezione ha corretto un errore che era passato nella procedura: il documento sorgente indicava un installer chiamato `EASE_Focus_Setup_v3.1.260.exe`, e quel file non esiste. La cartella contiene un installer InstallShield composto da `setup.exe`, `EASE Focus 3.msi`, `Data1.cab`, `ISSetup.dll` e `Setup.ini`. Ne segue che l'installazione va lanciata da dentro quella cartella e non copiando il solo eseguibile altrove, perché `setup.exe` cerca gli altri file accanto a sé.

L'ispezione ha anche rivelato un passo di installazione che il piano non prevedeva. Accanto all'installer principale ci sono due cartelle, `AFMGDatabaseService` e `AFMGDatabaseService_x64`, ciascuna con il proprio installer MSI. È il servizio di database introdotto con la linea 3.1, quello che nella tabella del changelog era valutato di impatto alto perché evita di scaricare a mano ogni GLL dal sito del costruttore. Va installato, e nella variante a 64 bit dato che il prefix è a 64 bit.

```bash
WINEARCH=win64 WINEPREFIX=~/wineprefixes/easefocus64 winecfg
WINEPREFIX=~/wineprefixes/easefocus64 winetricks -q dotnet48 corefonts vcrun2013 vcrun2019
cd ~/electroacoustics/progetto-stanza/room/EASE_Focus_v3.1.260
WINEPREFIX=~/wineprefixes/easefocus64 wine setup.exe
cd AFMGDatabaseService_x64
WINEPREFIX=~/wineprefixes/easefocus64 wine setup.exe
```

Sulle dipendenze, il documento sorgente indicava requisiti più larghi di quanto la documentazione ufficiale lasci intendere, cioè .NET Framework 4.0 o superiore e i redistributable di Visual C++ almeno nelle versioni 2010, 2013 e 2015, perché alcuni moduli GLL portano DLL proprie compilate con quei compilatori. I verbi di winetricks sopra coprono il caso; se un GLL specifico non si carica, il redistributable mancante è il primo sospetto.

Un avvertimento sul servizio di database. Un servizio Windows, sotto Wine, non gira come servizio di sistema ma come processo dentro il prefix, e la sua esecuzione dipende da `wineserver`. Se il programma lamenta l'assenza del database, la verifica è che il servizio sia stato installato in quel prefix e non in un altro. È un punto che va provato sul campo e che al momento non è verificato.

### Il database GLL del 2016

Sono dati e non un programma: 221 file, di cui 174 con estensione `.gll`, per 451 MB. Vanno copiati nella cartella che EASE Focus usa per cercarli, che dentro il prefix corrisponde a un percorso sotto i documenti dell'utente.

```bash
mkdir -p ~/wineprefixes/easefocus64/drive_c/users/$USER/Documents/EASE\ Focus\ 3/GLL
cp -r ~/electroacoustics/progetto-stanza/room/EASE_Focus_3_GLL_Database_2016_10_11/* ~/wineprefixes/easefocus64/drive_c/users/$USER/Documents/EASE\ Focus\ 3/GLL/
ls ~/wineprefixes/easefocus64/drive_c/users/$USER/Documents/EASE\ Focus\ 3/GLL | wc -l
```

Il dettaglio che fa perdere tempo se non lo si sa, già registrato nella pagina dei programmi: alcuni costruttori distribuiscono un `.gll` accompagnato da file `.dll` e `.bin`, e i tre devono restare nella stessa cartella o il modulo non si carica. Il conteggio di 221 file contro 174 GLL è precisamente la misura di quanti file di accompagnamento ci sono, e spiega perché la copia deve essere dell'intera cartella e non selettiva sui soli `.gll`.

### EASE Focus 3.0.18

Non si installa. È superata dalla 3.1.260, i GLL sono retrocompatibili, e tre versioni dello stesso programma in tre prefix sono manutenzione senza ritorno. Resta materiale d'archivio.

### Ramsete 27b: da verificare prima di installare

Ramsete è il programma di acustica architettonica che il documento sorgente annotava come alternativa e mai valutava, lasciando la sezione vuota. L'ispezione dei file dice qualcosa in più, e va detto con precisione ciò che dice e ciò che non dice.

Il formato dell'installazione è il toolkit di distribuzione di Visual Basic 6, riconoscibile dalla struttura del file `SETUP.LST`, che dichiara un `CabFile`, l'avvio di un `Setup1.exe` e fra le dipendenze di bootstrap `MSVCRT40.DLL` e `OLEPRO32.DLL`. Ne segue che Ramsete 27b è una applicazione Visual Basic 6, quindi a 32 bit, e che richiede il runtime di Visual Basic 6.

Questo è precisamente il caso che il documento sorgente segnalava come problematico nei prefix a 64 bit: i runtime di Visual Basic 6 sono fra le librerie vecchie che non ci girano bene, e in quei casi serve un prefix a 32 bit. È la ragione dell'emendamento alla decisione sui prefix discusso nella sezione seguente.

Sullo stato di licenza, l'onestà impone di dichiarare che non si sa. La cartella contiene soltanto i tre file dell'installazione originale, senza cartelle di modifica e senza file di gruppi di distribuzione illecita, quindi non ci sono indizi di manomissione. Ma Ramsete è un prodotto commerciale, e non è stato accertato se questa sia una versione dimostrativa liberamente distribuibile o una copia completa. La verifica va fatta prima di installarlo, non dopo, e la fonte è il sito del produttore.

Se la verifica dà esito positivo, la procedura è la seguente.

```bash
WINEARCH=win32 WINEPREFIX=~/wineprefixes/ramsete32 winecfg
WINEPREFIX=~/wineprefixes/ramsete32 winetricks -q vb6run corefonts
cd ~/electroacoustics/progetto-stanza/room/Ramsete27b
WINEPREFIX=~/wineprefixes/ramsete32 wine setup.exe
```

Va aggiunto un giudizio di priorità, per non spendere tempo su una cosa che il progetto non usa. Il ruolo di Ramsete nel workflow sarebbe l'acustica architettonica, ed è coperto da Akabak, che fa elettroacustica e acustica ambientale in un solo passaggio ed è già licenziato e funzionante. Ramsete è quindi un supplemento facoltativo, non un tassello mancante, e la sua installazione sta in fondo alla lista.

## Gruppo B: i programmi che non si installano

Otto voci, per circa 1,7 GB, cioè quasi tre quarti del peso del corredo. Non entrano nel piano perché portano protezioni rimosse: cartelle di modifica dichiarate nel nome, un emulatore di chiave hardware, sbloccatori separati, file descrittivi di gruppi di distribuzione illecita, e in un caso il file di provenienza da un servizio di condivisione.

Nel dettaglio: FineCone 2.1 con una cartella di modifica e una cartella `HASP`, che è l'emulazione della chiave hardware; FineMotor 2.5 con una cartella di modifica; LSPCad 5.25 e 6.32, la seconda con il file descrittivo di un gruppo; Grenander Loudspeaker Lab 3.13, con lo stesso tipo di file; AmpliTube 3 con uno sbloccatore separato accanto all'installer; Guitar Rig 5 Pro dichiarato sbloccato nel nome; e POD Farm, che accanto all'installer porta il file di provenienza da un servizio di condivisione e che in ogni caso richiede una licenza Line 6.

Il punto che conta per il progetto, e che vale più di qualunque considerazione generale, è che non serve nessuno di questi.

FineCone e FineMotor simulano il cono e il motore magnetico di un altoparlante a elementi finiti. Sono strumenti per chi costruisce gli altoparlanti, non per chi li assembla in un sistema: questo progetto compra woofer e tweeter finiti e ne usa i parametri Thiele/Small, quindi lavora a valle di dove quei programmi operano.

LSPCad progetta crossover e casse. Il suo ruolo è coperto da VituixCAD, che è gratuito, molto più recente, gestisce la direttività e la risposta in potenza, ed è già nel piano. La versione 5.25 è inoltre a 16 bit, quindi richiederebbe un prefix a 32 bit dedicato solo a sé.

Grenander Loudspeaker Lab è un altro strumento di progettazione di diffusori della stessa generazione, e vale lo stesso ragionamento.

AmpliTube, Guitar Rig e POD Farm sono emulatori di amplificatori per chitarra. Non hanno alcuna relazione con la progettazione elettroacustica: sono materiale di home recording finito nella stessa cartella. Se in futuro servissero per quel progetto, gli equivalenti liberi su Linux sono Guitarix e Rakarrack, nativi e senza problemi di licenza, ed esistono pacchetti di impulsi di cabinet liberamente distribuibili.

Il vantaggio secondario, non trascurabile, è di peso: escludendoli il trasferimento verso la macchina passa da 2,3 GB a circa 524 MiB, e il numero di prefix Wine da mantenere resta cinque invece di dieci.

## L'emendamento alla decisione sui prefix

La decisione ADR-004 stabiliva un prefix per programma, tutti a 64 bit, e nessun prefix a 32 bit, con la motivazione che l'unico candidato a richiederlo era WinISD, escluso perché ridondante. L'ispezione di questo corredo cambia il quadro, e la decisione va emendata invece di essere applicata alla lettera.

Il fatto nuovo è che Ramsete 27b è una applicazione Visual Basic 6, quindi richiede un prefix a 32 bit con il runtime `vb6run`. Il fatto resta condizionato alla verifica di licenza, quindi l'emendamento è condizionato a sua volta.

La forma dell'emendamento è registrata come ADR-009. In sintesi: la regola resta un prefix per programma e la preferenza resta per i 64 bit, ma il divieto assoluto di un prefix a 32 bit decade, perché era motivato da un solo candidato escluso e non da un principio. Se e quando Ramsete verrà installato, avrà il suo prefix a 32 bit, isolato dagli altri, e l'architettura `i386` dovrà essere dichiarata sul sistema. Quest'ultimo punto ha un costo che va messo in conto: l'architettura secondaria è uno dei fattori di attrito degli aggiornamenti di rilascio, ed è uno dei motivi per cui la macchina si trova nella situazione da cui questa documentazione parte.

Il consiglio operativo che ne deriva è di non dichiarare l'architettura `i386` durante l'installazione pulita, e di aggiungerla soltanto se e quando Ramsete supera la verifica di licenza e si decide di installarlo. Rimandare quel passo costa un comando; anticiparlo costa attrito permanente.

## La mappa dei prefix risultante

| Prefix | Architettura | Programmi | Dipendenze |
|---|---|---|---|
| `~/wineprefixes/akabak32` | **32 bit** | Akabak 3, VACS a 32 bit | nessuna: la configurazione funzionante non ha winetricks, .NET né corefonts |
| `~/wineprefixes/vituixcad64` | 64 bit | VituixCAD 2 | `dotnet48`, `corefonts` |
| `~/wineprefixes/easefocus64` | 64 bit | EASE Focus 3.1.260, servizio database AFMG | `dotnet48`, `corefonts`, `vcrun2013`, `vcrun2019` |
| `~/wineprefixes/arta64` | 64 bit | ARTA 1.7.1 | `vcrun2019`, `corefonts` |
| `~/wineprefixes/ramsete32` | 32 bit, condizionato | Ramsete 27b | `vb6run`, `corefonts` |

## Che cosa resta da verificare

Cinque punti, tutti dichiarati come non accertati e nessuno da promuovere a fatto senza una prova.

Lo stato di licenza di Ramsete 27b, dal sito del produttore. È la condizione che sblocca o chiude il quinto prefix e l'emendamento ad ADR-004.

Le limitazioni della modalità dimostrativa di ARTA senza registrazione, e se siano compatibili con la produzione di un file GLL. Vanno lette dalla documentazione del programma.

Il comportamento del servizio di database AFMG sotto Wine, che è un servizio Windows e sotto Wine non gira come servizio di sistema. Si prova solo installandolo.

L'accesso alla scheda audio da parte di ARTA sotto Wine, con la sua latenza e la sua stabilità. Per il progetto è secondario, perché la misura dal vivo la fa REW nativo.

Il contenuto del disco `G:`, dove risiede il materiale di EASE Focus 3.1.10 del workshop K-array a cui punta il collegamento presente in entrambe le copie. Il disco non era collegato, quindi non è stato ispezionato. Tracciato come PA-004, priorità bassa perché la versione da installare è la 3.1.260.
