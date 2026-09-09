<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Scheda operativa di reinstallazione, da stampare

> Sorgente tracciato della scheda che si porta accanto alla macchina durante la reinstallazione. Non sostituisce [installazione-pulita-26-04.md](installazione-pulita-26-04.md), che resta la procedura completa con il razionale di ogni passo: questa è la sua riduzione a ciò che serve avere sotto gli occhi quando non si è davanti alla postazione di sviluppo. Se le due divergono, ha ragione la procedura.
>
> Il `.docx` stampabile si genera con `python tools/make-scheda-docx.py`, che lo scrive sotto `_notes/`, escluso dal versionamento, oppure dove indica `--out`. Il documento generato non è la conversione di questo file: la carta ha vincoli che il Markdown non ha, cioè due pagine di spazio, caselle da spuntare a penna e riquadri colorati sul passo irreversibile, quindi il suo contenuto vive nello strumento come dati e i due vanno tenuti allineati a mano. È il prezzo dichiarato di avere un documento di stampa curato, e se le due divergono ha ragione la procedura completa.
>
> I dati di licenza non compaiono in questo file perché il repository è pubblico, e non compaiono nemmeno nel `.docx` a meno che non si passi `--con-licenza`, che li legge da `_notes/licenze-akabak-riservato.md`. Il valore predefinito è la loro assenza per una ragione operativa e non solo di riservatezza: alla reinstallazione non servono, perché il codice resta valido e si reinserisce dal file riservato al momento di riaprire AKABAK, mentre stamparli metterebbe un codice di attivazione permanente su un foglio che si porta in giro. Con `--con-licenza` il foglio stampato è materiale riservato.

## Dati della macchina

| Voce | Valore |
|---|---|
| Scheda madre | ASUSTeK H170-PRO, BIOS 3805 |
| Processore e memoria | Intel i7-6700, 16 GB DDR4 |
| Disco | NVMe 500 GB, `nvme0n1` |
| Interfaccia di rete | `enp3s0`, MAC `2c:4d:54:53:a4:fb` |
| Indirizzo attuale | `192.168.10.204` |
| Utente | `alesop95` |
| Sistema da installare | Ubuntu Studio 26.04.1 LTS |

## BIOS, prima di installare

Si entra con `Del` all'accensione. Quattro voci, e le prime due servono entrambe perché la seconda annulla la prima se resta come è.

| Percorso | Voce | Valore |
|---|---|---|
| `Advanced` / `APM Configuration` | `Power On By PCI-E/PCI` | `Enabled`, è il Wake-on-LAN |
| `Advanced` / `APM Configuration` | `ErP Ready` | `Disabled`, altrimenti la rete resta senza alimentazione a macchina spenta |
| `Boot` / `CSM` | `Launch CSM` | `Disabled`, la chiavetta è GPT/UEFI senza CSM |
| `Boot` | `Secure Boot` | lasciare come è, Ubuntu Studio si installa comunque |

Si salva con `F10`. Il menu di avvio temporaneo si apre con `F8` all'accensione, ed è preferibile a cambiare l'ordine di avvio perché non lascia modifiche permanenti.

## Avvio dalla chiavetta

Si sceglie `Try or Install Ubuntu Studio`. *Non si cerca la voce `Check disc for defects`: su questa immagine non esiste più*, e la sua assenza non è un difetto del supporto né una svista. Il controllo di integrità viene eseguito automaticamente all'avvio, e il suo esito resta leggibile a installazione finita in `/var/log/installer/casper-md5check.json`, dove `"result": "pass"` senza somme discordanti significa che il supporto era integro. La verifica quindi non si fa prima con un'azione, si legge dopo con un comando. La versione precedente di questa scheda prescriveva quella voce di menu ed è stata smentita dall'uso reale il 2026-09-08, come racconta MS-073.

## Partizionamento: il passo irreversibile

Si sceglie *`Partizionamento manuale`*. Non si sceglie in nessun caso la cancellazione del disco.

| Partizione | Dimensione | Mount point | Formattare |
|---|---|---|---|
| `nvme0n1p1` FAT32 | 1,13 GB | `/boot/efi` | **no** |
| `nvme0n1p2` ext4 | 80,00 GB | `/` | **sì**, ext4 |
| `nvme0n1p3` swap | 16,00 GB | nessuno | sì |
| `nvme0n1p4` ext4 | **402,98 GB** | `/home` | **NO** |

I tre soli momenti in cui questa fase si può sbagliare.

Primo: la schermata del tipo di installazione, dove va scelto il partizionamento manuale e non la cancellazione del disco.

Secondo: la riga di `nvme0n1p4`, dove si imposta soltanto il punto di montaggio e *non si tocca il menu del filesystem*, perché selezionare un filesystem attiva la formattazione da sé.

Terzo: la schermata di riepilogo prima di scrivere. Deve comparire una formattazione *soltanto* per `nvme0n1p2`. Fino a quel pulsante nulla è stato scritto sul disco.

Nell'ultima schermata l'utente va creato con lo stesso nome, cioè `alesop95`, altrimenti il `/home` conservato risulta di proprietà di un altro identificativo numerico e i permessi vanno corretti a mano.

## Primo avvio, tre verifiche

```bash
ls -la /home/alesop95/ && df -h /home
ls ~/electroacoustics && ls -la ~/Desktop
lsb_release -a && uname -r
```

Il `/home` deve contenere i materiali e i prefix Wine, e `~/electroacoustics` deve esserci con i suoi 281 file. Se `/home` risulta vuoto la formattazione è avvenuta, e a quel punto si ripristina l'archivio di backup.

## Catena audio a bassa latenza

Non serve un kernel diverso e, su Ubuntu Studio 26.04, non serve nemmeno configurarla: i parametri di avvio sono già attivi, e non arrivano da `/etc/default/grub`, che contiene soltanto `quiet splash`, ma da `/etc/default/grub.d/ubuntustudio.cfg`, che il sistema installa da sé. Aggiungerli a mano li duplicherebbe. Si verifica e basta; solo se mancassero si interviene, e in quel caso nel file drop-in e non in `/etc/default/grub`.

```
preempt=full threadirqs
```

I limiti per il gruppo audio stanno in un file sotto `/etc/security/limits.d/`, tipicamente `audio.conf`, e vanno verificati più che scritti da zero, perché Ubuntu Studio li installa da sé.

```
@audio   -  rtprio      95
@audio   -  memlock     unlimited
```

Verifiche: `cat /proc/cmdline` deve contenere i parametri, `ulimit -r -l` deve riportare `95` e `unlimited`, e `aplay -l` deve vedere i dispositivi audio presenti. Attenzione: i limiti si leggono con `ulimit` e non nei file di `/etc/security/limits.d/`, perché quei file sono giusti anche quando l'utente non appartiene ai gruppi `audio` e `pipewire` e quindi i limiti non sono in vigore.

## Wine: la parte dove un errore costa il programma

L'architettura a 32 bit va *dichiarata*, e va fatto prima di installare Wine e non dopo, altrimenti i pacchetti a 32 bit non vengono tirati dentro.

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install --install-recommends wine-stable winetricks
```

| Programma | Prefix | Architettura | Dipendenze |
|---|---|---|---|
| Akabak 3 e VACS | `~/wineprefixes/akabak32` | **win32** | **nessuna** |
| VituixCAD 2 | `~/wineprefixes/vituixcad64` | win64 | `dotnet48 corefonts` |
| WinISD | `~/wineprefixes/winisd32` | win32 | `vcrun6 corefonts` |
| EASE Focus 3.1.260 | `~/wineprefixes/easefocus64` | win64 | `dotnet48 corefonts` |

```bash
WINEARCH=win32 WINEPREFIX=~/wineprefixes/akabak32 winecfg
```

Nella scheda delle applicazioni si imposta la versione di Windows su *Windows 10*, che è quella dichiarata dal prefix funzionante.

Akabak e VACS *non* richiedono .NET, font Microsoft o runtime Visual C++: il prefix che funziona non ne ha nessuno, e la lista di dipendenze del documento sorgente descriveva i tentativi del troubleshooting e non ciò che serviva.

## Reinstallazione dei programmi e licenza

Gli installer sono sulla macchina, in `~/electroacoustics/installers/`, e sopravvivono alla reinstallazione perché stanno dentro `/home`.

```bash
WINEPREFIX=~/wineprefixes/akabak32 wine ~/electroacoustics/installers/AKABAK_Pro_v324b126.exe
WINEPREFIX=~/wineprefixes/akabak32 wine ~/electroacoustics/installers/VACS_32_v213b33.exe
```

La licenza è legata alla macchina e non al prefix, quindi il codice esistente resta valido. Si inserisce *una volta sola*, da AKABAK, e VACS non ne chiede un secondo.

```
Machine Identifier   {{MACHINE_IDENTIFIER}}
Release Code         {{RELEASE_CODE}}
```

Si apre AKABAK indicando esplicitamente il prefix, si va al menu di aiuto alla voce del release code, si incolla il codice e si conferma. L'edizione che si ottiene è Standard, non professionale, malgrado il nome dell'installer.

Il trasferimento dei dati fra AKABAK e VACS su Linux passa dagli appunti di sistema e non dalle pipeline COM, che Wine non implementa per la comunicazione fra processi. Va impostato nelle preferenze di AKABAK.

## Igiene dopo l'installazione

```bash
sudo sed -i 's/^Prompt=.*/Prompt=lts/' /etc/update-manager/release-upgrades
sudo ethtool -s enp3s0 wol g
```

La prima riga fa proporre in futuro le sole versioni LTS invece delle intermedie, che è la ragione per cui questa macchina si è trovata su un rilascio fuori supporto; su una installazione LTS pulita `Prompt=lts` è *già impostato*, quindi il comando è una non-operazione e serve solo a verificare. La seconda abilita il Wake-on-LAN sul lato sistema, che va aggiunta all'impostazione del BIOS e non la sostituisce.

Restano da rifare la disattivazione della sospensione automatica, la chiave SSH dedicata con la disattivazione dell'autenticazione per password, e la prenotazione dell'indirizzo sul router. La fase 10 della procedura le descrive per intero.

## Due cose che l'installatore sbaglia, e vanno corrette dopo

Osservate sull'installazione reale del 2026-09-08 e registrate in MS-074. Nessuna delle due impedisce di lavorare, entrambe si correggono a freddo.

La partizione di swap viene ignorata. L'installatore lascia `nvme0n1p3` intatta, e nella schermata di riepilogo la marca `Unchanged`, poi crea al suo posto un file `/swap.img` da 4 GB sulla radice. Il risultato è che quasi 15 GB di disco restano inutilizzati e la swap non basta più all'ibernazione, che con 16 GB di RAM ne richiede altrettanti. Si corregge sostituendo la voce del file con quella della partizione in `/etc/fstab`, per UUID.

I limiti realtime non arrivano all'utente. I file in `/etc/security/limits.d/` concedono `rtprio 95` e `memlock unlimited` ai gruppi `audio` e `pipewire`, che esistono, ma l'utente creato dall'installatore non appartiene a nessuno dei due. La trappola è che leggere quei file non rivela niente, perché il loro contenuto è giusto: il difetto si vede solo confrontandolo con `ulimit -r -l`, che risponde `0` e `8192`. Si corregge aggiungendo l'utente ai due gruppi e riaccedendo, perché l'appartenenza a un gruppo si applica al login e non subito.

## Dove sono le cose

| Cosa | Dove |
|---|---|
| Procedura completa in undici fasi | `docs/10-ambiente/installazione-pulita-26-04.md` |
| Fotografia della macchina prima dell'azzeramento | `docs/10-ambiente/fotografia-macchina-2026-09-07.md` |
| Registro degli interventi | `docs/OPERATIONS-LOG.md` |
| Sequenza numerata delle azioni | in testa a `docs/PENDING-ACTIONS.md` |
| Repository | `github.com/alesop95/diy-2way-monitors-home` |
| Backup di `/home`, 4,4 GB | `C:\Users\Utente\Desktop\_backup-ubuntu-studio\home-alesop95-2026-09-07.tar` |
| Immagine e Rufus | `C:\Users\Utente\Desktop\_iso-ubuntu-studio\` |

Il ripristino del backup, se servisse, si fa in streaming attraverso `ssh` e va lanciato dalla postazione Windows.

```bash
ssh alesop95@192.168.10.204 "cd / && sudo tar xpf - --numeric-owner" < home-alesop95-2026-09-07.tar
```
