<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Installazione di Ubuntu Studio

> Cronaca e razionale dell'installazione effettuata, con lo schema di partizionamento adottato e il motivo di ogni scelta. La procedura di installazione in sé è quella di qualsiasi Ubuntu; ciò che merita documentazione è il partizionamento, perché è la decisione che non si può correggere dopo senza rifare tutto.

## Requisiti e supporto di installazione

L'immagine si scarica dal sito ufficiale della distribuzione e si scrive su una chiavetta avviabile, su Windows con Rufus. La variante scelta è `ubuntustudio-25.04-desktop-amd64.iso`, cioè la distribuzione specializzata, e non `ubuntu-25.04-desktop-amd64.iso` seguita dall'installazione manuale del metapacchetto `ubuntustudio-installer`. Le due strade arrivano a un risultato simile ma non identico: la variante specializzata parte già con il kernel a bassa latenza e con la selezione audio coerente, mentre l'aggiunta a posteriori lascia il kernel generico a meno di intervenire.

I requisiti dichiarati dal progetto sono i seguenti, e la macchina li supera tutti con margine.

| Risorsa | Richiesto | Raccomandato |
|---|---|---|
| CPU | equivalente Intel Core 2 Duo | equivalente Intel Core i5 o superiore |
| RAM | 4 GB | 16 GB |
| Spazio su disco | 32 GB | 64 GB, di più per lavoro audio e video |

## Lo stato di partenza del disco

Il disco arrivava da una installazione Windows e portava le partizioni tipiche di quel sistema: una partizione NTFS di recovery da 471,86 MB, la partizione EFI[^1] in VFAT da 103,81 MB con il Windows Boot Manager, una partizione riservata Microsoft da 16,76 MB, la partizione principale NTFS da 498,95 GB, un frammento di spazio libero da 1,05 MB e una seconda partizione NTFS di recovery da 559,94 MB.

Di queste, dal punto di vista di Linux, solo la EFI ha un ruolo: le due partizioni di recovery e la partizione riservata Microsoft sono inutili, e la partizione principale NTFS è lo spazio da recuperare. Questo si verifica anche da dentro Windows prima di avviare l'installazione, ed è il controllo che conviene fare per non scoprire a metà procedura di aver frainteso la mappa del disco.

## Le opzioni del partizionamento manuale

Nella schermata di partizionamento manuale, che l'installatore chiama *Something else*, la colonna *Used as* indica il filesystem e quindi come Linux userà quella partizione. EXT4 è lo standard, robusto e compatibile con tutti gli strumenti. XFS è orientato ai server e non porta vantaggi concreti su un desktop audio. Btrfs offre snapshot e compressione al prezzo di una gestione più complessa. VFAT, cioè FAT32, serve alla partizione di avvio UEFI. SWAP è lo spazio usato come estensione virtuale della RAM. NTFS è il filesystem di Windows, che Ubuntu legge e scrive ma non può usare per il sistema operativo.

La colonna *Mount point* indica dove la partizione verrà innestata nell'albero di Linux. La radice è obbligatoria e contiene il sistema. La cartella `/home` contiene i file personali e le impostazioni utente, e si tiene separata proprio per conservare i dati attraverso una reinstallazione. La cartella `/boot/efi` è obbligatoria su UEFI e sta in FAT32. Una partizione senza punto di montaggio non viene usata dal sistema operativo.

## Lo schema adottato

Per eliminare Windows la strada più semplice e più pulita è creare una nuova tabella delle partizioni direttamente dalla procedura di installazione, con il comando *New partition table*, che cancella tutte le partizioni NTFS in un colpo. Con questa strada anche la EFI viene riformattata e l'opzione *Format* si può spuntare senza compromettere l'avvio, cosa verificata empiricamente su questa macchina.

| Partizione | Filesystem | Dimensione | Mount point | Formattata | Note |
|---|---|---|---|---|---|
| EFI | FAT32 (VFAT) | ~100 MB | `/boot/efi` | sì | riformattata dalla nuova tabella senza danno per l'avvio |
| root | EXT4 | ~80 GB | `/` | sì | dove vivono il sistema e i programmi |
| swap | - | ~16 GB | - | sì | pari alla RAM, così l'ibernazione resta possibile |
| home | EXT4 | tutto lo spazio restante | `/home` | sì | progetti, configurazioni, file personali |

La scelta di EXT4 per root e home è dettata dalla compatibilità: è il filesystem su cui JACK, PulseAudio e PipeWire sono più collaudati. La swap pari alla RAM ha due motivi distinti, e vale distinguerli perché portano allo stesso numero per ragioni diverse: uno è l'ibernazione, che richiede di poter scrivere l'intero contenuto della RAM su disco, l'altro è evitare il crash di un progetto audio o video che saturi la memoria.

La separazione di `/home` su una partizione propria è la decisione che oggi paga il dividendo più alto, perché rende una reinstallazione pulita del sistema una operazione a basso rischio: si riformatta root e si lascia home intatta. La pagina sull'aggiornamento alla LTS sfrutta esattamente questa proprietà.

## Un difetto dello schema documentato nel sorgente

Il documento sorgente, nell'elenco finale delle partizioni risultanti, riporta due partizioni distinte entrambe montate sulla radice, la seconda con il resto dello spazio. Questo non è uno schema valido, perché due filesystem non possono condividere lo stesso punto di montaggio: la quarta partizione è `/home`, come del resto dice la tabella dello stesso documento. Si tratta di un errore di trascrizione nel sorgente, corretto qui, e vale segnalarlo perché è il tipo di dettaglio che una reinstallazione fatta seguendo l'appunto sbagliato replicherebbe.

[^1]: *EFI*, Extensible Firmware Interface - interfaccia fra il firmware della macchina e il sistema operativo, che ha sostituito il BIOS tradizionale; la sua partizione dedicata contiene i caricatori di avvio e va formattata in FAT32.
