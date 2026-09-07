# Documentazione tecnica

> Livello documentale tracciato di questo progetto. Al momento contiene un solo blocco, quello sull'ambiente della macchina di lavoro, che arriva sincronizzato da un progetto gemello.

## Il blocco sull'ambiente

La cartella [10-ambiente/](10-ambiente/README.md) descrive la macchina Ubuntu Studio usata per l'home recording e lo strato di compatibilità Wine che ci gira sopra: l'installazione del sistema con il suo partizionamento, la diagnosi del blocco di aggiornamento verso la LTS successiva, la differenza fra Wine, un emulatore e una macchina virtuale, la configurazione di Wine da zero, l'installazione dei programmi Windows e i guasti già incontrati.

Il blocco arriva sincronizzato da `diy-2way-monitors-home`, che ne è la copia canonica, perché la stessa macchina serve a entrambi i progetti: alla progettazione elettroacustica dei monitor in quello, e all'home recording in questo. Ogni file porta in testa una intestazione che dichiara la provenienza. Non va modificato qui: le modifiche si fanno nella copia canonica e si propagano con `python tools/sync-ambiente.py` eseguito da quel progetto.

## Perché condiviso e non duplicato

Le informazioni sulla macchina sono le stesse per i due usi, e tenerne due copie modificate a mano garantisce che divergano in silenzio. Le alternative valutate erano un submodule git, che aggiungerebbe una dipendenza fra due repository personali per otto file di testo, e una sincronizzazione bidirezionale, che richiederebbe una risoluzione dei conflitti che a due copie non vale il costo. La decisione è registrata come ADR-005 nel progetto canonico.

## Che cosa in questo blocco riguarda l'home recording

Tre parti sono direttamente pertinenti a questo progetto e non soltanto a quello dei monitor.

La descrizione della macchina e della catena audio, cioè il kernel a bassa latenza di Ubuntu Studio, la gestione di JACK, PulseAudio e PipeWire, e l'interfaccia Focusrite Scarlett 2i2 di seconda generazione con la sua alimentazione phantom.

Lo schema di partizionamento, in particolare la scelta di tenere `/home` su una partizione separata e la swap pari alla RAM, che è ciò che rende una reinstallazione del sistema una operazione a basso rischio per i progetti audio già presenti.

La diagnosi del blocco di aggiornamento e la raccomandazione di una installazione pulita di Ubuntu Studio 26.04 LTS, che è una decisione sulla macchina e quindi riguarda entrambi i progetti allo stesso modo.

La parte su Wine è meno pertinente qui, perché i programmi Windows che richiede sono simulatori acustici e non strumenti di registrazione. Resta comunque parte del blocco, sia perché descrive lo stato reale della macchina, sia perché la pagina sulla differenza fra Wine, un emulatore e una macchina virtuale è la spiegazione da leggere se un domani servisse far girare qui un plugin o uno strumento solo-Windows.

## Azioni differite

Le decisioni e gli impegni di questo progetto che dipendono da una condizione esterna stanno in [PENDING-ACTIONS.md](PENDING-ACTIONS.md). La prima voce, e al momento l'unica, è la valutazione dell'acquisto di una interfaccia audio: la Focusrite Scarlett 2i2 attualmente disponibile ha due ingressi, che bastano al progetto dei monitor ma sono il vincolo principale per la registrazione multitraccia, che è lo scopo di questo progetto. La voce fissa i criteri da stabilire prima di guardare i modelli, fra cui il supporto su Linux, che a questo scopo non è un dettaglio.

## Che cosa manca a questo progetto

Tutto il resto. Non c'è ancora una catena del segnale documentata, nessun progetto in una workstation audio digitale, nessuna lista di attrezzatura, nessuna nota di mixing. Il repository resta un contenitore intenzionalmente minimale, e questo blocco è il primo contenuto tecnico reale che vi entra.
