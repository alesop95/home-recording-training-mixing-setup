# home-recording-training-mixing-setup

> Istruzioni di progetto, versionate. Le preferenze personali vivono in `CLAUDE.local.md` (ignorato).

## Cos'e questo progetto

Setup di home recording, training e mixing: note di configurazione. Il progetto non è ancora avviato come codice; la struttura standard e predisposta.

## Materiali e dati

Il materiale scritto a mano vive alla radice ed è escluso dal versionamento dal gitignore per tipo di file. Puoi raccoglierlo in una cartella `_notes` (anch'essa ignorata).

## Sviluppo e identità

git locale, identità `alesop95`, alias SSH `github-personal`. Remoto da collegare. Commit e push manuali.

## Standard

Struttura standard `.claude/PROJECT-SYSTEM.md` predisposta. Man mano che si lavora si decide quali parti del template servono davvero.

## Documentazione dell'ambiente: copia sincronizzata, non modificare qui

La cartella `docs/10-ambiente/` descrive la macchina Ubuntu Studio e lo strato Wine che ci gira sopra, e arriva sincronizzata da `diy-2way-monitors-home`, che ne è la copia canonica: la stessa macchina serve alla progettazione elettroacustica dei monitor e all'home recording. Ogni file porta in testa una intestazione che dichiara la provenienza.

Le modifiche a quel blocco si fanno nella copia canonica e si propagano con `python tools/sync-ambiente.py` eseguito da quel progetto. Una modifica fatta qui verrebbe sovrascritta alla propagazione successiva, e lo strumento di controllo la segnalerebbe come deriva. L'indice della documentazione di questo progetto è `docs/README.md`.
