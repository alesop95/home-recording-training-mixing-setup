<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Prefix, architettura e licenze in Wine

> Modello mentale di Wine e sue conseguenze operative. Spiega che cosa è un prefix, perché conviene averne più di uno, come si sceglie fra 32 e 64 bit e perché una licenza legata alla macchina sopravvive alla cancellazione di tutto.

## Wine è uno, i prefix sono molti

La confusione più frequente è pensare che installare un programma Windows sotto Wine significhi installarlo nel sistema operativo. Non è così, e la distinzione fra il motore e i contenitori è la struttura da tenere in mente.

Wine è il motore, e di quello ce n'è una sola installazione: il binario in `/usr/bin/wine` e le librerie sotto `/usr/lib`. Un prefix è un contenitore, cioè una cartella di Windows virtuale con il suo `C:`, i suoi `Program Files`, il suo `Windows/System32` e il suo registro. Di prefix se ne possono avere quanti si vuole, ciascuno con la propria versione di Windows dichiarata, le proprie librerie e i propri programmi installati.

Il prefix di default è `~/.wine`. Per crearne altri si valorizza la variabile `WINEPREFIX` davanti al comando, e la prima esecuzione di uno strumento di Wine in una cartella inesistente la crea e la inizializza.

```bash
WINEARCH=win32 WINEPREFIX=~/wineprefixes/akabak32 wine32 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/vituixcad64 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/easefocus64 winecfg
```

Il vantaggio dell'isolamento è concreto e non teorico. Le dipendenze installate con `winetricks` finiscono soltanto nel prefix in cui si lavora, quindi un programma che pretende una versione particolare di .NET o di un runtime Visual C++ non può rompere gli altri. Il costo è spazio su disco e qualche variabile in più da scrivere nei comandi.

Il backup di un ambiente configurato è la copia della sua cartella, e questa è la proprietà che rende Wine comodo da amministrare.

```bash
cp -r ~/wineprefixes/akabak32 ~/wineprefixes/akabak32-backup
```

## La scelta fra 32 e 64 bit

Alla creazione di un prefix Wine deve decidere se simulare un ambiente Windows a 32 o a 64 bit, e la decisione non è reversibile senza ricreare il prefix. Determina come sono strutturate le cartelle `System32` e `SysWOW64`, quali API sono disponibili e se Wine userà librerie e runtime a 32 o a 64 bit. Il default nelle versioni moderne è 64 bit; per un prefix a 32 bit si valorizza anche `WINEARCH`.

```bash
WINEARCH=win32 WINEPREFIX=~/wineprefixes/akabak32 wine32 winecfg
WINEARCH=win64 WINEPREFIX=~/wineprefixes/vituixcad64 winecfg
```

Il prefix a 32 bit conviene al software legacy. I programmi vecchi, quelli pensati per Windows XP, Vista o 7 a 32 bit, ci funzionano meglio, e molti pacchetti di `winetricks` si aspettano proprio quell'ambiente e vi si installano senza intoppi. Il limite è lo stesso che avrebbero su Windows reale: un processo a 32 bit non può allocare più di 4 GB di memoria.

Il prefix a 64 bit è obbligatorio per il software moderno, perché una applicazione compilata solo per x64 non parte in un ambiente a 32 bit, e permette a un singolo processo di usare più di 4 GB, cosa che per un simulatore acustico su modelli complessi conta davvero. In cambio, alcune librerie vecchie, per esempio i runtime di Visual Basic 6 o versioni datate di DirectX, non ci girano bene, e per quelle serve un prefix a 32.

La regola operativa che ne deriva è semplice. Un programma dichiaratamente a 32 bit, o vecchio, va in un prefix a 32 bit, che è più stabile per quel caso. Un programma moderno che chiede Windows 10 o 11 a 64 bit va in un prefix a 64. E soprattutto non si mescola tutto in un prefix unico: tenere separati i prefix a 32 e a 64 evita i conflitti fra runtime .NET e Visual C++, e permette di fare il backup di ciascun ambiente per conto proprio.

La tabella riassume la scelta per i programmi di questo progetto.

| Programma | Architettura | Prefix | Motivo |
|---|---|---|---|
| Akabak 3 | **32 bit** | `~/wineprefixes/akabak32` | l'eseguibile installato è PE32 i386 e il prefix funzionante è `win32`: si veda ADR-016, che smentisce l'affermazione del documento sorgente sui 64 bit |
| VACS 2.1.3 | **32 bit** | `~/wineprefixes/akabak32` | variante `VACS_32.exe`, nello stesso prefix di Akabak perché si usano in sequenza |
| VituixCAD 2 | 64 bit | `~/wineprefixes/vituixcad64` | nativo a 64 bit per dichiarazione del produttore, non verificato su questa macchina perché mai installato |
| WinISD | 32 bit | `~/wineprefixes/winisd32` | distribuito solo a 32 bit, con runtime datati |
| EASE Focus 3 | 64 bit | `~/wineprefixes/wine-ease` | basato su .NET 4.x, e alcuni moduli GLL portano DLL proprie |

## Perché una licenza legata alla macchina sopravvive

Le licenze che un programma Windows può usare ricadono in due categorie, e la differenza determina che cosa si perde ricostruendo l'ambiente.

Una licenza legata al prefix è scritta nel registro di Wine o in un file dentro il prefix. Ricreando il prefix da zero l'attivazione si perde, e va reinserito il codice.

Una licenza legata alla macchina è calcolata da un identificativo hardware, tipicamente derivato da processore, indirizzo MAC o numero di serie del disco. Poiché Wine espone l'hardware reale invece di simularne uno, quell'identificativo non cambia quando si cancellano i prefix, né quando si reinstalla Wine, né quando si reinstalla il sistema operativo. Dal punto di vista del programma si è sempre sulla stessa macchina.

Akabak ricade nel secondo caso. La licenza è machine-based e si ottiene inviando all'autore il *Machine Identifier* che il programma mostra nel menu di aiuto; il *Release Code* che torna vale esclusivamente per quella macchina. Ne segue che si può azzerare Ubuntu Studio, reinstallare Wine, rifare i prefix e reinstallare Akabak, e sarà sufficiente reinserire lo stesso release code. L'unico caso che richiede di ricontattare l'autore è un cambio significativo di hardware, per esempio una nuova scheda madre, oppure lo spostamento del programma dentro una macchina virtuale, dove l'identificativo sarebbe quello virtuale e quindi diverso.
