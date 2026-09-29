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

Norme caricate su richiesta, una riga per situazione con le parole con cui si presenta, così che il caricamento non dipenda dal ricordare che la norma esista.

- `git worktree list` mostra più di un albero, se ne crea o se ne rimuove uno, si deve decidere da dove leggere la memoria versionata: skill `alberi-di-lavoro`.
- Un recupero web fallisce con 403 o con una pagina di verifica anti-bot, la fonte sta su Reddit o su Discord, serve la trascrizione di un video, si sta per annotare una fonte non letta: skill `fonti-non-recuperabili`.
- Si scrive o si valuta una prova automatica, si chiude un difetto, una verifica manuale smentisce una suite verde, si sta per dichiarare completo un intervento il cui scopo era un effetto misurabile: skill `prove-che-misurano`.
- Si inizializza o si allinea il progetto, oppure cambia il modo in cui si prova e si rilascia, e va deciso come separare test e produzione: skill `separazione-ambienti`.
