# Azioni differite

> Registro delle azioni che non si possono compiere adesso perché dipendono da una condizione esterna o da una decisione, con la condizione di sblocco di ciascuna e il criterio con cui si stabilisce che sono compiute. Un promemoria che vive solo in una conversazione è perduto; una voce qui sopravvive alla sessione e a un clone del repository.
>
> Ogni azione ha un identificativo nella forma `PA-NNN`. Una voce compiuta non si cancella: si marca come compiuta con la data e l'esito.

## PA-001 - Valutare l'acquisto di una scheda audio per l'home recording

Data di apertura: 2026-09-07. Stato: **aperta, da valutare**.

Che cosa va fatto. Una ricerca di mercato e una scelta di acquisto per l'interfaccia audio di questo progetto, con i criteri dichiarati prima dei modelli e non dopo.

Da dove nasce, e perché è una voce di questo progetto e non di quello dei monitor. La macchina Ubuntu Studio è condivisa fra due progetti: la progettazione elettroacustica dei monitor, che vive in `diy-2way-monitors-home`, e l'home recording, che vive qui. L'interfaccia oggi disponibile è una Focusrite Scarlett 2i2 di seconda generazione, con due ingressi e alimentazione phantom, e per il progetto dei monitor è sufficiente: lì serve un solo ingresso microfonico per il microfono di misura, e la Scarlett lo copre. Per l'home recording due ingressi sono invece il vincolo principale, quindi l'esigenza è di questo progetto e va decisa qui.

Va detto per chiarezza che al 2026-09-07 la Scarlett non risulta collegata alla macchina: la fotografia dello stato della macchina, nel blocco condiviso sull'ambiente, mostra come sole schede l'audio integrato `ALC887-VD` con le sue uscite HDMI. È una constatazione, non un problema: l'interfaccia va collegata quando serve.

I criteri da fissare prima di guardare i modelli, perché è l'ordine che evita di innamorarsi di una scheda e poi giustificarla.

Quanti ingressi contemporanei servono davvero, e di che tipo: microfonici con phantom, di linea, strumento a alta impedenza, digitali. È il criterio che ordina tutto il resto, perché è quello su cui la Scarlett attuale è insufficiente.

Se serva l'ascolto a latenza bassa con monitoraggio diretto, e se serva un secondo bus di uscita per una mandata separata, per esempio verso una coppia di monitor e una cuffia con mix diverso.

Il supporto su Linux, che non è un dettaglio: le interfacce conformi alla classe audio USB funzionano senza driver proprietari, mentre alcune richiedono software di configurazione che esiste solo per Windows e macOS, e in quel caso funzioni come il mixer interno o il routing restano inaccessibili. Il criterio operativo è preferire dispositivi che espongano tutto il necessario come conformi alla classe, e verificare la compatibilità sulle fonti della comunità Linux audio prima dell'acquisto e non dopo.

La qualità dei preamplificatori e il rumore, che contano ma meno di quanto il marketing suggerisca a questo livello di prezzo, e comunque meno del numero di ingressi se il numero di ingressi è il vincolo.

Il rapporto con l'esistente: se la nuova interfaccia sostituisce la Scarlett o la affianca. Affiancarla ha senso solo se si vuole aggregare gli ingressi, cosa che su Linux con PipeWire è possibile ma aggiunge una complicazione di sincronizzazione dei clock che va messa in conto.

Il criterio di completamento. Una decisione documentata con i criteri, i modelli confrontati, la verifica di compatibilità Linux e la motivazione della scelta, oppure la decisione motivata di non comprare nulla e restare sulla Scarlett.

Nota di priorità. Non blocca nulla del progetto dei monitor, che sulla Scarlett attuale ha tutto ciò che gli serve. Blocca invece l'home recording multitraccia, cioè lo scopo di questo progetto, quindi qui è la prima voce.
