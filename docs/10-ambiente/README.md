<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Ambiente Linux: Ubuntu Studio e Wine

> Blocco documentale condiviso. Descrive la macchina di lavoro e lo strato di compatibilità Windows che ci gira sopra. Questo blocco esiste in due copie identiche, una in questo progetto e una nel progetto `home-recording-training-mixing-setup`, perché la stessa macchina serve sia alla progettazione elettroacustica sia all'home recording. La copia canonica è quella di `diy-2way-monitors-home`; la sincronizzazione è meccanica e si esegue con `tools/sync-ambiente.py`.

## La macchina

L'hardware è un vecchio desktop riconvertito: processore Intel i7-6700 a 3.40 GHz, 16 GB di RAM DDR4 e un SSD[^1] Crucial CT500P25SD8 da 500 GB, dato al 91 per cento di vita residua da una scansione con CrystalDiskInfo. È una macchina che Windows 11 avrebbe escluso per requisiti, e questo è il motivo per cui è stata riformattata su Linux invece di essere dismessa.

La scheda audio è esterna, una Focusrite Scarlett 2i2 di seconda generazione, con alimentazione phantom a 48 V disponibile. Questa scelta ha una conseguenza diretta sulla catena di misura, discussa nella pagina della fase di misura: rende possibile un microfono XLR da misura, e quindi rende non obbligatorio un microfono USB come l'UMIK-1.

La distribuzione è Ubuntu Studio, non Ubuntu con i pacchetti audio aggiunti a mano. La differenza sta nel kernel a bassa latenza preconfigurato e nella selezione di pacchetti già montata per registrazione, mixing, mastering e live processing. La versione installata è la 25.04, e la pagina sull'aggiornamento spiega perché questa scelta oggi è il problema principale della macchina.

## Indice del blocco

L'installazione del sistema, i requisiti e lo schema di partizionamento adottato stanno in [ubuntu-studio-installazione.md](ubuntu-studio-installazione.md).

La diagnosi del blocco di aggiornamento verso la LTS[^2] successiva, con la procedura di verifica e le due strade possibili, sta in [ubuntu-lts-upgrade.md](ubuntu-lts-upgrade.md).

La differenza concettuale fra Wine, un emulatore e una macchina virtuale, che è il punto da cui dipende tutto il resto della configurazione, sta in [wine-vs-emulatore.md](wine-vs-emulatore.md).

Il modello dei prefix, la scelta fra 32 e 64 bit e il comportamento delle licenze legate alla macchina stanno in [wine-prefix-e-dipendenze.md](wine-prefix-e-dipendenze.md).

La procedura di pulizia, installazione e configurazione di Wine sta in [wine-configurazione.md](wine-configurazione.md).

L'installazione dei singoli programmi Windows, cioè Akabak, VituixCAD, WinISD ed EASE Focus, sta in [wine-programmi-windows.md](wine-programmi-windows.md).

I guasti già incontrati e la loro risoluzione stanno in [wine-troubleshooting.md](wine-troubleshooting.md).

[^1]: *SSD*, Solid State Drive - unità di memorizzazione a stato solido, senza parti in movimento, con tempi di accesso molto inferiori a quelli di un disco meccanico.

[^2]: *LTS*, Long Term Support - designazione delle versioni di Ubuntu con supporto esteso, rilasciate ogni due anni ad aprile, contrapposte alle versioni intermedie a supporto breve.
