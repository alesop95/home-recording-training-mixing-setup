<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Architettura dell'ambiente Wine: che cosa è fatto di che cosa, e perché funziona

> Scritta il 2026-09-17, dopo la sostituzione di Wine documentata nei microstep da MS-137 a MS-140. Risponde a una domanda che le altre pagine del blocco non coprono: non come si installa e non che cosa è stato fatto, ma come è fatto l'ambiente che ne risulta, quali pezzi lo compongono e per quale ragione ciascuno esiste. Chi deve eseguire una procedura legga [veeam-agent-linux.md](veeam-agent-linux.md) o [wine-configurazione.md](wine-configurazione.md); chi deve capire perché quelle procedure sono fatte così legga questa.

## Le quattro cose che vanno tenute distinte, e che quasi tutti confondono

Quasi tutti i malintesi su Wine nascono dal trattare come una cosa sola quattro strati che sono separati, hanno cicli di vita diversi e si rompono in modi diversi. Vale enunciarli prima di qualunque dettaglio, perché ogni decisione presa in questo progetto si spiega guardando quale strato tocca.

```
   +------------------------------------------------------------------+
   |  4. programma Windows                                            |
   |     AKABAK.exe, VACS_32.exe, VituixCAD.exe, EASE Focus, ARTA      |
   |     e' un file PE, cioe' un eseguibile per Windows, invariato     |
   +------------------------------------------------------------------+
   |  3. prefix                                                       |
   |     ~/.wine, ~/wineprefixes/vituixcad64, easefocus64, arta64      |
   |     drive_c/  system.reg  user.reg  dosdevices/                   |
   |     e' una finta installazione di Windows, una per utente         |
   +------------------------------------------------------------------+
   |  2. Wine                                                         |
   |     /opt/wine-stable/bin/wine, wineserver, lib/wine/...           |
   |     traduce le chiamate Windows in chiamate Linux                 |
   +------------------------------------------------------------------+
   |  1. Linux                                                        |
   |     kernel 7.0, Wayland con XWayland, PipeWire, glibc a 64 bit    |
   +------------------------------------------------------------------+
```

Lo strato 1 è la macchina e non cambia per Wine. Lo strato 2 è un pacchetto di sistema, si installa e si disinstalla con il gestore dei pacchetti, ed è condiviso da tutti gli utenti e da tutti i prefix. Lo strato 3 è un albero di file dentro la home dell'utente, non è un programma e non si esegue. Lo strato 4 è il software del produttore, identico bit per bit a quello che girerebbe su Windows.

La distinzione fra 2 e 3 è quella che conta di più, perché è la sola che spiega come mai sostituire Wine non cancella i programmi installati e come mai, ciò nonostante, può impedire di eseguirli.

## Che cos'è un prefix, e perché non è una installazione di Wine

Un prefix è una cartella che contiene ciò che un programma Windows si aspetta di trovare attorno a sé: un disco `C:`, il registro di sistema, i caratteri, le librerie di sistema, le cartelle dei dati applicativi. Wine lo popola alla creazione e lo aggiorna quando serve; il programma che vi si installa scrive lì dentro i propri file, esattamente come farebbe su Windows.

Tre conseguenze pratiche discendono da questa forma, e sono la ragione di tre scelte del progetto.

La prima è che un prefix appartiene all'utente e non al sistema, quindi vive sotto `/home`. È il motivo per cui l'installazione pulita di settembre, che ha azzerato la radice conservando `/home`, ha lasciato intatti i prefix e con essi i programmi installati: il racconto è in MS-085, e quella sopravvivenza non è stata fortuna ma la conseguenza dello schema di partizionamento scelto in fase 2.

La seconda è che le librerie di sistema dentro il prefix sono copie, prodotte dalla versione di Wine che ha creato il prefix. Non sono collegamenti alle librerie installate: sono file veri dentro `drive_c/windows`. Quando si cambia la versione di Wine quelle copie restano quelle vecchie, e vanno aggiornate. Wine lo fa da sé al primo avvio, oppure lo si chiede esplicitamente con `wineboot -u`, che è preferibile perché così l'aggiornamento accade in un momento osservabile invece che dentro l'avvio di un programma, dove un suo fallimento si confonderebbe con un difetto del programma.

La terza è che un prefix dichiara la propria architettura alla creazione e non la cambia mai più. La dichiarazione sta nella prima riga di `system.reg`, sotto forma di `#arch=win32` oppure `#arch=win64`, e si riconosce anche dalla presenza o assenza della cartella `drive_c/windows/syswow64`. Non esiste alcun comando che converta un prefix da una architettura all'altra: se serve l'altra, si crea un prefix nuovo e si reinstalla.

## I due modi in cui Wine esegue un programma a 32 bit

Questo è il punto tecnico su cui si è deciso il tentativo del 2026-09-17 e su cui è fallito, e senza di esso il passaggio ai pacchetti di WineHQ sembra un aggiornamento di versione mentre è un cambio di meccanismo.

Un programma Windows a 32 bit deve poter chiamare le proprie librerie di sistema a 32 bit. Wine può soddisfare quella richiesta in due modi, e la differenza sta tutta in quanti bit ha il lato Unix, cioè la parte di Wine che parla davvero con il kernel Linux.

```
   WoW64 classico, cioe' Wine 10 dei pacchetti della distribuzione

     AKABAK.exe                          eseguibile PE a 32 bit
          |
     lib/wine/i386-windows/*.dll         librerie Windows a 32 bit
          |
     lib/wine/i386-unix/*.so             lato Unix a 32 bit        <-- esiste
          |
     glibc e librerie di sistema a 32 bit                          <-- esistono
     (pacchetto wine32:i386, 263 pacchetti, 1,1 GB)
          |
     kernel Linux a 64 bit


   WoW64 nuovo, cioe' WineHQ 11.0

     AKABAK.exe                          eseguibile PE a 32 bit
          |
     lib/wine/i386-windows/*.dll         librerie Windows a 32 bit
          |
     passaggio da 32 a 64 bit dentro lo stesso processo
          |
     lib/wine/x86_64-unix/*.so           lato Unix solo a 64 bit
          |
     glibc a 64 bit
          |
     kernel Linux a 64 bit
```

Nel primo modo il sistema deve portare un intero albero di librerie a 32 bit parallelo a quello a 64 bit, dalla libreria C fino a Mesa, GTK e GStreamer: è il motivo per cui installare `wine32:i386` tirava dentro 263 pacchetti, ed è la ragione per cui ADR-016 prescriveva di dichiarare l'architettura `i386` sulla macchina.

Nel secondo modo quell'albero non serve più, perché il passaggio da 32 a 64 bit avviene dentro il processo di Wine e il lato Unix è solo a 64 bit. In teoria i programmi a 32 bit continuano a funzionare con un sistema ospitante interamente a 64 bit; su questa macchina non è accaduto, e il perché sta più sotto.

I numeri che distinguono i due modi si leggono dentro il pacchetto, e sono stati misurati in MS-139 elencando il contenuto del `.deb` senza installarlo: il pacchetto di WineHQ porta 1071 file sotto `lib/wine/i386-windows/`, cioè le librerie Windows a 32 bit, e zero file sotto `lib/wine/i386-unix/`, cioè nessun lato Unix a 32 bit. Sotto `x86_64-unix/` ce ne sono 288. Non esiste alcun `bin/wine64` perché non serve: c'è un solo `bin/wine`, ed è a 64 bit.

## Perché il secondo modo, su questa macchina, non ha funzionato

Due ostacoli si sono presentati uno dopo l'altro, e vanno distinti perché il primo è una proprietà del meccanismo e il secondo è un difetto osservato.

Il primo, che era prevedibile e previsto, riguarda i prefix a 32 bit. Un prefix dichiarato `#arch=win32` non contiene alcuna parte a 64 bit: il suo `drive_c/windows/system32` ospita librerie a 32 bit e la cartella `syswow64` non esiste. Per eseguirlo serve un processo Wine interamente a 32 bit, cioè esattamente il caricatore che il WoW64 nuovo non ha. Il prefix `~/.wine`, che ospita AKABAK e VACS, è di questo tipo, quindi il passaggio avrebbe richiesto di ricrearlo a 64 bit e di reinstallarvi i due programmi, portandosi dietro il file della licenza.

Il secondo non era prevedibile e si è scoperto misurando. Sotto i pacchetti di WineHQ, in questa macchina, anche un prefix a 64 bit creato da zero resta senza lato a 32 bit: `syswow64` viene creata e lasciata vuota, `system32` si popola regolarmente, e l'inizializzazione si interrompe con un fallimento di marshalling OLE e con il servizio di chiamata di procedura remota che non parte. Un programma a 64 bit gira, uno a 32 bit non ha librerie su cui girare. La misura è stata ripetuta sei volte sul ramo stabile e una sul ramo staging, con e senza display, e l'esito non cambia: è MS-141 e MS-142.

Poiché tutti i programmi Windows di questo progetto sono a 32 bit, il secondo ostacolo da solo rende i pacchetti di WineHQ inutilizzabili qui, indipendentemente dal primo. Il tentativo è stato annullato e la macchina è tornata ai pacchetti della distribuzione, dove il WoW64 classico funziona e tutti e quattro i programmi si aprono.

## Perché un prefix per programma

La scelta è di ADR-004 e sopravvive a tutte le revisioni successive. La ragione è che le dipendenze installate in un prefix sono globali a quel prefix: un componente installato per un programma è visibile a tutti gli altri che vivono lì dentro, e un componente che ne rompe uno li rompe tutti. Tenerli separati rende ogni guasto locale e ogni esperimento reversibile cancellando una cartella.

Il progetto ha quindi quattro prefix e non uno, e la loro composizione riflette esattamente le dipendenze misurate e non quelle dichiarate dai produttori.

```
   ~/.wine                    AKABAK 3.2.4 + VACS 2.1.3
                              nessuna dipendenza installata
                              e' l'eccezione dichiarata: due programmi,
                              un prefix, perche' si usano in sequenza

   ~/wineprefixes/vituixcad64 VituixCAD
                              dotnet48

   ~/wineprefixes/easefocus64 EASE Focus 3.1.260 + servizio di database AFMG
                              dotnet48, gdiplus

   ~/wineprefixes/arta64      ARTA 1.7.1
                              nessuna dipendenza aggiuntiva
```

Le dipendenze di EASE Focus sono due e non le quattro che la documentazione del produttore elenca: la misura è in MS-110, e le due che non servono sono state escluse provandole, non ipotizzandole.

## Dove vive ogni cosa sul disco

```
   /opt/wine-stable/                     Wine, strato 2
       bin/wine                          il solo eseguibile di avvio
       bin/wineserver                    il processo che tiene lo stato
       lib/wine/x86_64-unix/             lato Unix a 64 bit, 288 file
       lib/wine/i386-windows/            librerie Windows a 32 bit, 1071 file
       lib/wine/x86_64-windows/          librerie Windows a 64 bit

   /usr/bin/wine                         collegamento creato dal metapacchetto
   /usr/local/bin/winetricks             script preso dalla sorgente ufficiale

   /home/alesop95/
       .wine/                            prefix, strato 3
           system.reg                    prima riga: #arch=win32 o win64
           user.reg
           dosdevices/                   c: -> ../drive_c, z: -> /
           drive_c/
               windows/                  la finta installazione di Windows
               Program Files/            i programmi, strato 4
               ProgramData/RDTeam/       Akabak.ini, con la licenza
               users/alesop95/           AppData, Temp, documenti
       wineprefixes/                     gli altri tre prefix
       electroacoustics/                 gli installer del corredo
```

Due dettagli di questo albero spiegano cose che altrimenti sorprendono. Il collegamento `dosdevices/z:` punta alla radice del filesystem Linux, ed è il motivo per cui un programma Windows dentro il prefix vede tutto il disco come unità `Z:`: è così che EASE Focus raggiunge il database dei GLL senza che nulla sia stato copiato, come MS-110 ha accertato. E la cartella `Temp` dentro `users` può contenere file sparsi molto grandi, cioè file che dichiarano una lunghezza e non occupano spazio: MS-138 ne ha trovato uno da 20 479 MiB con zero blocchi allocati, che rende `du` e `tar` in disaccordo di un fattore quattro.

## Dove vive la licenza, e perché sopravvive a quasi tutto

La licenza di AKABAK non sta dove si presume. Non è nel registro del prefix, che è il posto dove una applicazione Windows di solito la mette, e la ricerca che partiva da lì ha perso tempo su una assunzione ragionevole e sbagliata. Sta in un file di testo, `C:\ProgramData\RDTeam\Akabak.ini`, nella sezione `[Security]` alla voce `SecCode`, ed è il racconto di MS-085.

La collocazione spiega il comportamento. `ProgramData` è la cartella dei dati a livello di macchina e non di utente, e sta sotto `drive_c` del prefix, cioè fra i dati e non fra le parti di sistema. L'aggiornamento di un prefix riscrive il registro e le librerie di sistema e non tocca i dati, quindi la licenza sopravvive alla migrazione da una versione di Wine a un'altra. Sopravvive anche alla reinstallazione del sistema operativo, purché `/home` resti intatta, perché il prefix sta lì.

Non sopravvive invece alla creazione di un prefix nuovo, che è appunto nuovo e vuoto. In quel caso la licenza si porta copiando quel file dal prefix vecchio, che nessuna delle operazioni descritte qui cancella, oppure reinserendo il Release Code permanente, che è conservato fuori dal repository in `_notes/licenze-akabak-riservato.md` perché un codice di attivazione non va pubblicato.

Il codice è legato al Machine Identifier, cioè a un valore derivato dall'hardware, e non al prefix né all'installazione: è la ragione per cui una reinstallazione non lo invalida, ed è anche la ragione per cui un cambio sostanziale di hardware lo invaliderebbe.

## Che cosa cambia e che cosa no quando si sostituisce Wine

Vale riassumerlo come tabella, perché è la domanda che si pone ogni volta e la risposta è sempre la stessa.

| Elemento | Sostituendo Wine |
|---|---|
| programmi installati nei prefix | restano, sono file dentro `drive_c` |
| licenza di AKABAK | resta, è un file dentro `drive_c/ProgramData` |
| dipendenze installate con winetricks | restano, sono file dentro il prefix |
| librerie di sistema dentro il prefix | vanno aggiornate, con `wineboot -u` |
| prefix a 64 bit | sopravvivono all'aggiornamento |
| prefix a 32 bit sotto WoW64 nuovo | non sono più eseguibili, vanno rifatti |
| `winetricks` da pacchetto | segue il pacchetto Wine da cui dipende |

L'ultima riga è la ragione per cui `winetricks` è stato preso dalla sua sorgente ufficiale e messo in `/usr/local/bin` prima di rimuovere qualunque cosa: il pacchetto della distribuzione dipende dal pacchetto `wine` della distribuzione, quindi sarebbe stato rimosso insieme a esso, e senza di lui un prefix non si ricostruisce.

## L'architettura stabile, e come si verifica che sia quella

Al 2026-09-17, dopo il tentativo di sostituzione e il suo annullamento, l'ambiente è tornato a essere questo, ed è stabile in tutti e quattro gli strati.

```
   Linux        Ubuntu Studio 26.04.1 LTS, kernel 7.0.0-31-generic
                sessione Wayland, con XWayland per le finestre di Wine
                server X privato :9 via Xvfb per il lavoro da remoto

   Wine         10.0 dai pacchetti della distribuzione, componente
                universe, WoW64 classico con il ramo i386 installato
                wine, wine64, wine32:i386, wine-common, libwine x2

   winetricks   script dalla sorgente ufficiale, in /usr/local/bin
                versione 20260125-next, indipendente da ogni pacchetto

   prefix       quattro, tutti sotto /home
                ~/.wine a 32 bit per AKABAK e VACS, con la licenza
                tre a 64 bit per VituixCAD, EASE Focus e ARTA
```

Il tentativo di passare ai pacchetti ufficiali di WineHQ è stato fatto e annullato nella stessa giornata, e va registrato qui perché è la domanda che chiunque si porrà guardando questa configurazione: perché la distribuzione e non l'upstream, che la documentazione del progetto raccomandava. La risposta è misurata e sta in MS-141 e MS-142. I pacchetti WineHQ per Ubuntu 26.04 esistono in tre rami e usano tutti il WoW64 nuovo, senza alcun ramo `i386`; su questa macchina non popolano il lato a 32 bit dei prefix, cioè creano `syswow64` e lo lasciano vuoto, quindi nessun programma a 32 bit può girare. La prova è stata fatta sei volte sul ramo stabile e una sul ramo staging, con e senza display, su prefix migrati e su prefix nuovi, e l'esito è sempre lo stesso. Poiché tutti i programmi Windows di questo progetto sono a 32 bit, quei pacchetti non sono utilizzabili qui.

Ne segue che ADR-016 resta in vigore e non va revisionata: l'architettura `i386` dichiarata sul sistema è necessaria, e lo è oggi per una ragione in più rispetto a quando fu scritta, cioè che l'alternativa upstream è stata provata e non funziona.

La verifica che l'ambiente sia davvero questo, e non solo dichiarato tale, sta in quattro comandi che vanno eseguiti in quest'ordine e letti per quello che escludono.

```bash
wine --version
```

```bash
command -v wine winetricks
```

```bash
apt-cache policy wine-stable | head -4
```

```bash
for p in ~/.wine ~/wineprefixes/*; do echo "$p: $(head -1 $p/system.reg 2>/dev/null; grep -m1 '^#arch' $p/system.reg 2>/dev/null)"; done
```

Il primo deve rispondere `wine-11.0`: se rispondesse `wine-10.0` significherebbe che un residuo dei pacchetti della distribuzione vince nel PATH, che è il modo tipico in cui una sostituzione sembra fatta e non lo è. Il secondo deve rispondere `/usr/bin/wine` e `/usr/local/bin/winetricks`, cioè il collegamento del metapacchetto nuovo e lo script indipendente, non quello di un pacchetto. Il terzo deve nominare `dl.winehq.org` come origine e non l'archivio Ubuntu. Il quarto deve riportare `#arch=win64` per ogni prefix: un `win32` residuo sarebbe un prefix che non si può più aprire, e conviene scoprirlo con questo comando invece che lanciando il programma.

## Che cosa questa architettura non dà

Va detto perché una pagina che descrive una architettura tende a farla sembrare completa.

Non dà una immagine avviabile della macchina: il backup in essere è a livello di file, per i motivi accertati in MS-120, MS-121 e MS-122, quindi un ripristino totale passa da una installazione nuova seguita dal riversamento. L'immagine avviabile, se servirà, si prende con Clonezilla a macchina spenta.

Non dà alcuna garanzia sui programmi a 32 bit sotto il WoW64 nuovo oltre a quella che la misura ha dato: le 1071 librerie Windows a 32 bit dicono che l'ambiente per eseguirli esiste, non che ogni programma vi giri senza difetti. La prova è l'esecuzione, non l'elenco dei file.

E non dà una catena di misura: la fase 6 resta aperta perché la macchina ha la sola scheda audio integrata, e quale interfaccia acquisire è la decisione tracciata in PA-012.
