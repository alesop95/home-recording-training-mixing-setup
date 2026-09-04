<!-- COPIA SINCRONIZZATA. Non modificare qui.
     La copia canonica di questo blocco vive in diy-2way-monitors-home/docs/10-ambiente/
     e si propaga con `python tools/sync-ambiente.py` da quel progetto. -->

# Perché la macchina non si aggiorna alla LTS successiva

> Diagnosi del blocco di aggiornamento, con la procedura di verifica da eseguire sulla macchina e le due strade possibili. Attenzione al livello di certezza: la causa qui ricostruita è una ipotesi fondata sul calendario dei rilasci di Ubuntu e sulla versione dichiarata nel documento sorgente, non una osservazione. La macchina non era raggiungibile in rete al momento della stesura, quindi nessun comando di questa pagina è stato eseguito su di essa. La sezione di verifica esiste proprio per promuovere l'ipotesi a fatto, o per smentirla.

## Il calendario dei rilasci, che è la chiave di tutto

Ubuntu alterna due tipi di rilascio con due politiche di supporto diverse, e confonderli è la causa più comune dei blocchi di aggiornamento. Le versioni LTS escono ogni due anni ad aprile, cioè 22.04, 24.04, 26.04, e hanno cinque anni di supporto. Le versioni intermedie escono ogni sei mesi e hanno nove mesi di supporto: sono pensate come anticipazione, non come base stabile.

Applicando il calendario alla versione installata su questa macchina si ottiene il quadro seguente.

| Versione | Tipo | Rilascio | Fine del supporto |
|---|---|---|---|
| 24.04 | LTS | aprile 2024 | aprile 2029 |
| 25.04 | intermedia | aprile 2025 | gennaio 2026 |
| 25.10 | intermedia | ottobre 2025 | luglio 2026 |
| 26.04 | LTS | aprile 2026 | aprile 2031 |

La macchina è su 25.04. Alla data di stesura di questa pagina, settembre 2026, quella versione è fuori supporto da circa otto mesi, e anche la versione intermedia immediatamente successiva, la 25.10, è fuori supporto da circa due mesi.

## Le tre cause che si sommano

Da questo quadro discendono tre ostacoli distinti, che agiscono insieme. Distinguerli conta, perché ciascuno produce un messaggio di errore diverso e ciascuno si verifica con un comando diverso.

Il primo ostacolo è che dalla 25.04 non esiste un salto diretto alla 26.04. Lo strumento `do-release-upgrade` propone soltanto il rilascio immediatamente successivo nella catena, quindi da 25.04 propone la 25.10, e da 25.10 propone la 26.04. Il percorso è obbligatoriamente in due passi. Chi si aspetta di passare da una intermedia alla LTS successiva in un colpo trova uno strumento che sembra non funzionare, mentre sta funzionando come progettato.

Il secondo ostacolo è la configurazione del prompt di aggiornamento. Il file `/etc/apt/../update-manager/release-upgrades`, cioè `/etc/update-manager/release-upgrades`, contiene una direttiva `Prompt` che vale `lts`, `normal` oppure `never`. Con `Prompt=lts` lo strumento consulta l'elenco delle sole versioni LTS, e da una versione intermedia quell'elenco non contiene alcun successore: il risultato è il messaggio *No new release found*, che sembra dire che non ci sono aggiornamenti mentre sta dicendo che non ce ne sono di quel tipo. Su una versione intermedia la direttiva corretta per procedere è `normal`.

Il terzo ostacolo, e probabilmente il più visibile in pratica, è che gli archivi dei rilasci fuori supporto vengono spostati. Quando una versione raggiunge la fine del supporto i suoi pacchetti lasciano `archive.ubuntu.com` e finiscono su `old-releases.ubuntu.com`. Finché le sorgenti puntano al vecchio indirizzo, `apt update` restituisce errori 404 su tutti i componenti, e senza un `apt update` che vada a buon fine `do-release-upgrade` non può nemmeno scaricare il proprio strumento di aggiornamento. Questo vale sia per la 25.04 su cui la macchina si trova, sia per la 25.10 attraverso cui dovrebbe transitare: entrambe le tappe sono su archivio storico.

A questi tre si aggiungono due fattori di attrito propri di questa macchina, che non bloccano ma complicano. Uno è l'architettura `i386` aggiunta a mano con `dpkg --add-architecture i386` per far funzionare Wine a 32 bit: raddoppia l'insieme dei pacchetti per molte librerie e, durante un aggiornamento di rilascio, è una fonte classica di dipendenze insoddisfacibili. L'altro sono i repository di terze parti, tipicamente quello di WineHQ, che l'aggiornamento disabilita automaticamente e che poi vanno riabilitati a mano con il nome del nuovo rilascio.

## Come verificarlo sulla macchina

Questi comandi sono da eseguire sulla macchina Ubuntu Studio, in lettura, e servono a confermare o smentire la ricostruzione. Nessuno di essi modifica il sistema.

```bash
lsb_release -a
cat /etc/os-release
cat /etc/update-manager/release-upgrades
grep -rn "archive.ubuntu.com\|old-releases.ubuntu.com\|security.ubuntu.com" /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
ls -la /etc/apt/sources.list.d/
dpkg --print-foreign-architectures
sudo apt update
do-release-upgrade -c
df -h / /boot /boot/efi /home
uname -r
dpkg -l | grep -c "^ii"
```

L'esito atteso, se la ricostruzione è corretta, è il seguente. Il primo blocco conferma la 25.04. Il terzo mostra `Prompt=lts`. Il quarto mostra sorgenti che puntano ancora ad `archive.ubuntu.com` con il nome in codice `plucky`. Il sesto elenca `i386`. Il settimo restituisce errori 404. L'ottavo dice *No new release found*. Se invece l'ottavo comando propone la 25.10, allora il blocco è altrove e la diagnosi va rifatta sui messaggi reali.

Da notare, sulla forma dei file di configurazione: dalla 24.04 Ubuntu usa il formato deb822, quindi le sorgenti stanno in `/etc/apt/sources.list.d/ubuntu.sources` e non più nel vecchio `/etc/apt/sources.list`. Su una 25.04 il file da guardare è quello, e un appunto che citasse solo il vecchio percorso sarebbe fuorviante.

## Le due strade

Chiarita la causa, restano due modi di uscirne, e non sono equivalenti.

### Strada A: aggiornamento in posto, in due salti

Si ripuntano le sorgenti su `old-releases.ubuntu.com`, si porta la 25.04 a un aggiornamento completo, si imposta `Prompt=normal`, si esegue il salto alla 25.10, e da lì si ripete l'operazione per arrivare alla 26.04 LTS. Ad ogni tappa i repository di terze parti vanno disattivati prima e riallineati dopo, e l'architettura `i386` va tenuta d'occhio.

Il vantaggio è che il sistema installato, con i suoi programmi e le sue configurazioni, sopravvive. Gli svantaggi sono tre e vanno pesati insieme: due aggiornamenti di rilascio consecutivi attraverso versioni fuori supporto sono la configurazione più fragile in cui si possa fare questa operazione, l'archivio storico non riceve più correzioni quindi si transita per pacchetti già superati, e il risultato finale è un sistema che porta la sedimentazione di tre rilasci più le manipolazioni fatte a mano su Wine.

### Strada B: installazione pulita di Ubuntu Studio 26.04 LTS, conservando la home

Si scarica l'immagine di Ubuntu Studio 26.04 LTS, si riformatta la sola partizione root e si rimonta `/home` esistente senza formattarla. Il partizionamento adottato all'installazione originaria, con `/home` su una partizione separata, è esattamente ciò che rende questa operazione a basso rischio: i dati personali, i progetti e i prefix di Wine restano dove sono.

È la strada che questa documentazione raccomanda, per quattro motivi.

Il primo è che è più corta e più prevedibile: una installazione contro due aggiornamenti di rilascio in cascata su archivi storici.

Il secondo è che coincide con l'obiettivo dichiarato di avere un setup pulito. Ricostruire l'ambiente Wine da zero, seguendo la procedura documentata nelle pagine di questo blocco, elimina proprio la sedimentazione che ha generato i guasti registrati nella pagina di troubleshooting, dove il prefix `~/.wine` risultava corrotto o incompatibile.

Il terzo è che la licenza di Akabak non è a rischio. È legata all'identificativo hardware della macchina, non al prefix e non all'installazione del sistema operativo, come spiegato nella pagina sulla differenza fra Wine e una macchina virtuale: reinstallare il sistema, ricreare i prefix e reinserire lo stesso release code funziona. L'unico caso che richiederebbe di ricontattare l'autore è un cambio significativo di hardware.

Il quarto è che porta su una base con supporto fino ad aprile 2031, e da una LTS gli aggiornamenti futuri procedono da LTS a LTS con `Prompt=lts`, che è la configurazione corretta su una base stabile e la stessa che oggi contribuisce al blocco solo perché la macchina non è su una LTS.

L'unica precauzione seria prima di procedere è il backup, perché una installazione che sbaglia la selezione della partizione cancella `/home`. La pagina va letta insieme al manifest di trasferimento del progetto, che è il momento in cui i materiali pesanti vengono spostati e quindi anche l'occasione naturale per verificare che esista una copia fuori dalla macchina.

## Cosa resta da verificare

Tre punti sono dichiaratamente non verificati e non vanno trattati come fatti finché la macchina non è raggiungibile.

Lo stato reale della macchina, cioè la versione effettivamente installata, il contenuto delle sorgenti apt e il messaggio esatto restituito dallo strumento di aggiornamento. Il documento sorgente dichiara la 25.04 ma è stato scritto nel 2025.

La disponibilità di Ubuntu Studio 26.04 LTS come immagine scaricabile e la sua data effettiva di rilascio. La cadenza dei rilasci di Ubuntu è regolare e le derivate ufficiali seguono la stessa numerazione, ma la conferma va presa dal sito del progetto, non dedotta dal calendario.

Lo stato del kernel a bassa latenza sulla 26.04 e l'eventuale cambiamento nel modo in cui la distribuzione lo fornisce. Su questo la documentazione ufficiale di Ubuntu Studio è la sola fonte da usare, perché è il tipo di dettaglio che cambia fra un rilascio e l'altro.
