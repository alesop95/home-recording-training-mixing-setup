# Catena di ingresso: valutazione dei candidati

> Ricerca documentale del 2026-09-30, condotta per la voce PA-012 del progetto `diy-2way-monitors-home` e per la PA-001 di questo progetto. Nessun dispositivo è stato provato: ogni affermazione dichiara la propria fonte e se quella fonte è stata letta o soltanto elencata da una ricerca, e i punti non confermati sono scritti come tali. La scelta resta dell'utente; questa pagina la prepara e non la prende.

## I requisiti, com'erano dichiarati al momento della ricerca

La macchina è Ubuntu Studio 26.04 LTS con kernel 7.0, PipeWire e Ardour. La catena deve servire due usi: registrare e mixare dal vivo dentro Ardour una batteria elettronica, una chitarra elettrica, un basso e una voce; e misurare diffusori con un microfono XLR calibrato. I vincoli fissati dall'utente sono cinque. Il dispositivo è conforme alla classe audio USB e il kernel lo riconosce senza driver proprietari. Gli ingressi simultanei sono otto, scelti con margine su un minimo reale di cinque. C'è almeno un ingresso strumento ad alta impedenza, intorno a un megaohm, per chitarra e basso passivi, e meglio due. C'è almeno un ingresso microfonico con alimentazione phantom a 48 V, con dati di conversione dichiarati dal produttore. Non c'è un tetto di spesa, quindi i candidati si presentano per fascia.

## Quanti ingressi servono davvero

Senza la batteria su USB gli ingressi sono cinque: due per la batteria in stereo, uno ad alta impedenza per la chitarra, uno per il basso e uno per la voce. Il microfono di misura non si usa insieme alla voce e può riusarne il canale; la misura con riferimento di anello, cioè il loopback di REW, chiede però un secondo ingresso di linea libero. Se il modulo della batteria manda al mixer le proprie uscite dirette analogiche, la sola batteria può salire fino a otto ingressi.

Un modulo con audio USB multitraccia non è un ingresso gratuito, e la ragione va detta perché cambia il conto. Diventa un secondo dispositivo audio con un proprio clock, non agganciato a quello dell'interfaccia. PipeWire può unire due schede USB solo ricampionando in modo adattivo una delle due, e il backend ALSA di Ardour gestisce un secondo dispositivo nello stesso modo. È un'inferenza tecnica, non verificata con una misura né su una fonte di questa ricerca. Ne segue che la via più robusta per registrare dal vivo resta portare le uscite dirette del modulo negli ingressi di linea dell'interfaccia, che lavorano su un unico clock, ed è un argomento concreto per restare sugli otto ingressi.

## Topologia A: interfaccia USB multicanale, con il mix dentro Ardour

| Candidato | Prezzo, Thomann Italia al 2026-09-30 | Mic / linea / Hi-Z | Phantom | Frequenza | Conversione dichiarata | Classe su Linux | Fonte della classe | In commercio nel 2026 |
|---|---|---|---|---|---|---|---|---|
| Behringer UMC1820 | 163 EUR, consegna in 6-8 settimane | 8 combo mic/linea; numero di Hi-Z non confermato: Thomann dice 8, una recensione riporta Hi-Z sulle prese TRS a circa 750 kΩ | 48 V | 24 bit / 96 kHz | nessun dato sulla pagina del produttore letta | dichiarata solo da rivenditori; uso su Linux senza driver riportato da una recensione | pagina Behringer letta, non nomina la classe; recensione letta; rivenditori solo elencati | sì, con attesa |
| Focusrite Scarlett 18i20 4th Gen | 635 EUR, disponibile | 8 mic/linea, 2 Hi-Z da 1 MΩ | sì, indipendente per canale | fino a 192 kHz | EIN -127 dBu A, gamma dinamica mic 116 dB A, uscite 122 dB A | supporto esplicito nel kernel con il driver FCP, ID 1235:821d in `mixer_quirks.c`, dal kernel 6.14, più `fcp-support` e firmware in spazio utente; mixer interno governabile da `alsa-scarlett-gui` | sorgente del kernel letta; `INSTALL.md` di alsa-scarlett-gui letto; pagina di supporto Focusrite sulla classe solo elencata, risposta 403 | sì |
| PreSonus Studio 1824c | 524,99 USD sul sito del produttore, esaurita; pagina Thomann 404 | 8 mic, di cui 2 mic/strumento; impedenza strumento non dichiarata | 48 V | fino a 192 kHz | EIN -128 dBu A, gamma dinamica mic 110 dB A | il produttore non dichiara la classe nella pagina letta; il kernel ha un driver dedicato, `mixer_s1810c.c`, ID 194f:010d | sorgente del kernel letta; patch del 2026 per la 1824 letta; pagina PreSonus letta | a rischio: esaurita dal produttore e tolta da Thomann |
| MOTU UltraLite-mk5 | 739 EUR, disponibile | 2 mic/linea/Hi-Z, 6 linea, 8 analogici | 48 V | 44,1-192 kHz, dato di Thomann | non letti | MOTU dichiara la conformità alla classe per iPad e iOS; Linux non è nominato | pagina MOTU letta | sì |
| RME Fireface UCX II | 1.244 EUR, disponibile; 1.699 USD su rme-usa | 2 mic, 2 strumento/linea da 1 MΩ, 4 linea, 8 analogici | 48 V per canale, dal manuale | fino a 192 kHz con il driver RME, fino a 96 kHz / 24 bit in modalità conforme alla classe | AD 115 dB A, EIN -128 dBu A | il manuale RME dichiara la modalità conforme alla classe "natively supported by operating systems like Windows, Mac OS X and Linux", attivabile dal pannello frontale | manuale RME letto, capitoli 31-36; pagine RME e rme-usa lette | sì |

I rischi su Linux, candidato per candidato. La UMC1820 è la più semplice perché non ha un mixer interno da programmare, ma il produttore non dichiara la classe nella pagina letta, si ferma a 96 kHz, e il numero di ingressi ad alta impedenza va verificato sul manuale: le due fonti si contraddicono, e circa 750 kΩ stanno sotto il megaohm richiesto. La 18i20 ha il supporto Linux migliore della fascia, con instradamento, mixer e phantom governabili da Linux, e chiede due componenti in spazio utente, cioè il server FCP e il firmware, che la distribuzione deve avere o che vanno installati; va anche disattivata la modalità di archiviazione di massa, che l'aggiornamento del firmware spegne da sé secondo la documentazione letta. Il kernel 7.0 soddisfa il vincolo del 6.14. La 1824c passa lo streaming al driver generico e il controllo a un driver dedicato nato da ingegneria inversa e non dal produttore; una segnalazione alla lista linux-sound, elencata e non letta perché il server rispondeva 502, riferisce che con il firmware 3.11 non esporrebbe più l'interfaccia del mixer; ed è esaurita. La UltraLite-mk5 ha il mixer interno CueMix 5 solo per macOS, Windows e iOS, quindi mixer e instradamento non si configurano da Linux salvo strumenti di terze parti non verificati: rischio alto. La UCX II, in modalità conforme alla classe, non offre impostazioni hardware né TotalMix dal computer, ma il pannello frontale dà accesso a guadagni, instradamento, monitor, equalizzazione e dinamica, e l'unità conserva fino a sei configurazioni preparate con TotalMix FX su Windows o macOS; i limiti sono i 96 kHz e il fatto che equalizzazione e dinamica di ingresso stanno sempre nel percorso di registrazione, quindi per una misura vanno lasciate neutre. Nel kernel attuale non c'è un driver di mixer dedicato alla UCX II, e un driver in spazio utente di terze parti per la modalità non conforme esiste ed è sperimentale.

Per fascia. Sotto i 200 EUR la UMC1820, a patto di verificare prima gli ingressi ad alta impedenza, rinunciando alla dichiarazione del produttore e ai 192 kHz. Fra 600 e 750 EUR la 18i20 di quarta generazione, preferita alla UltraLite-mk5 perché il suo mixer si governa da Linux con codice nel kernel, e perché ha otto ingressi microfonici contro due. Oltre i 1.200 EUR la UCX II: salendo si guadagnano il supporto Linux dichiarato dal produttore nel manuale, tutte le funzioni sul pannello senza software, la configurazione salvata nell'unità, dati di conversione documentati e la stabilità nota dei driver RME; si perde il margine, cioè due soli ingressi microfonici e otto analogici esatti, estendibili via ADAT. Il limite dei 96 kHz non pesa sulla misura acustica.

## Topologia B: mixer analogico che somma verso due canali USB

| Candidato | Prezzo | Mic / linea / Hi-Z | Phantom | USB | Dati | Classe su Linux | Fonte | In commercio nel 2026 |
|---|---|---|---|---|---|---|---|---|
| Yamaha MG10XU | 266 EUR, disponibile | 4 mic/linea, 10 linea; nessun Hi-Z nella scheda tecnica, impedenza mic 3 kΩ, linea 10 kΩ | 48 V | 2 in / 2 out, 24 bit, fino a 192 kHz | EIN -128 dBu | "USB Audio Class 2.0 compliant" nella scheda tecnica Yamaha | scheda tecnica letta da una copia ospitata da terzi, creata nel gennaio 2022; pagine Yamaha solo elencate, risposta 403 | sì, secondo Thomann |
| Mackie ProFX10v3, e v3+ | 229 EUR la v3, 349 EUR la v3+, dalla stessa pagina Thomann | 4 mic, Hi-Z sui canali 1-2 con impedenza non dichiarata | 48 V | 2 in / 4 out, 192 kHz; la modalità Interface della v3+ registra soltanto i canali 1-2 | non letti | solo Thomann dichiara la conformità alla classe | pagina Thomann letta; pagina Mackie letta, senza dichiarazione; manuale oltre i 10 MB, non letto | sì; la v3+ è in arretrato presso Mackie |

Qui la conformità alla classe pesa poco, perché un flusso stereo è il caso più semplice del driver generico. Il limite è di impianto: si perdono le tracce separate e il mix va deciso in ripresa. Per la misura, il riferimento di anello chiede di mandare il microfono tutto a sinistra e un ritorno di linea tutto a destra, che si può fare ed è fragile. La MG10XU non ha ingressi ad alta impedenza, quindi chitarra e basso passivi chiedono due DI. Conviene sotto i 300 EUR e soltanto se le tracce separate non servono; la MG10XU ha l'unica dichiarazione UAC2 letta su fonte primaria in questa topologia, la Mackie ha l'Hi-Z e nessuna dichiarazione del produttore.

## Topologia C: mixer digitale o ibrido che è anche interfaccia multicanale

| Candidato | Prezzo | Mic / Hi-Z | Phantom | USB | Frequenza | Dati | Classe su Linux | Fonte | Note Linux |
|---|---|---|---|---|---|---|---|---|---|
| Behringer X Air XR18 | 385 EUR, disponibile | 16 mic Midas, Hi-Z sui canali 1-2 da 1 MΩ | 48 V per ingresso | 18x18 | solo 44,1 / 48 kHz | A/D 114 dB, EIN -128 dBu A | la guida rapida Behringer elenca Linux fra i sistemi supportati per l'audio USB, con il driver ASIO solo per Windows; le parole "class compliant" non compaiono | guida rapida letta da una copia ospitata da terzi; pagina Behringer letta, con X AIR EDIT per PC, Mac e Linux | nessun fader fisico: si governa da app o da X AIR EDIT, che ha un eseguibile nativo per Linux; è l'unico mixer della rosa configurabile per intero da Linux |
| TASCAM Model 12 | 599 EUR, consegna in 2-3 settimane | 8 mic XLR, 8 Hi-Z | 48 V | 12 in, cioè 10 canali più il mix stereo, / 10 out | solo 44,1 / 48 kHz | non letti | "USB Audio Class 2.0" secondo TASCAM; Linux non è fra i sistemi elencati | pagina TASCAM letta; due forum letti | fader analogici fisici; il pannello Model Mixer Settings esiste solo per Windows e macOS; su Linux Mint 22.3 funziona come interfaccia in Ardour, e come controller DAW vanno soltanto i tasti di trasporto, dal forum di Ardour; un caso di crepitii e cadute del 2023 risolto con un cavo da USB-A a USB-C su porta USB 3, dal forum di Mabox |
| Allen & Heath CQ-18T | circa 999 EUR, dato di un comparatore solo elencato | 16 mic, non verificato; Hi-Z non trovato nella scheda | 48 V | USB-B, 24 canali a 48 kHz o 16 a 96 kHz | 48 / 96 kHz | non estratti con certezza | la scheda A&H dichiara "Core Audio compliant, ASIO/WDM for Windows"; Linux non è nominato | scheda A&H letta; pagina prodotto solo elencata, risposta 403 | schermo tattile e comandi fisici, niente fader motorizzati; un forum A&H del 2024-2025 lo riporta funzionante su Linux Mint 21 con Ardour; il software MixPad gira solo sotto Wine 9 |

In questa topologia il mixer interno è il prodotto stesso, quindi la domanda decisiva è se lo si possa governare da Linux. Solo l'XR18 lo permette per intero con software nativo. Nel Model 12 il mixer è analogico e fisico, quindi il software per Windows e macOS serve soltanto a funzioni accessorie. Il CQ-18T ha schermo e comandi sull'unità, ma il suo software di controllo gira solo sotto Wine. XR18 e Model 12 si fermano a 48 kHz, che per la musica e per la misura acustica basta. Scartati o non verificati: lo Zoom LiveTrak L-12, la cui modalità conforme alla classe Zoom presenta come pensata per iOS, con l'uso su Linux non confermato dalla pagina letta; il PreSonus StudioLive AR12c, esaurito e senza dichiarazione di classe nella pagina letta; il Behringer X32 Producer, per cui X32-Edit su Linux esiste ma la pagina della scheda X-USB letta non dichiara la classe.

Per fascia. Sui 400 EUR l'XR18, con il massimo dei canali e il controllo nativo da Linux, senza fader. Sui 600 EUR il Model 12, con fader veri, otto ingressi ad alta impedenza e UAC2 dichiarata dal produttore: è il candidato migliore se si vogliono insieme fader fisici e tracce separate. Sui 1.000 EUR il CQ-18T, meno motivato su Linux degli altri due.

## Moduli della batteria con audio USB multitraccia

| Modulo | Audio USB multitraccia | Conforme alla classe | Fonte | Esito per Linux |
|---|---|---|---|---|
| Roland V71 | 32 canali in modalità Vendor, a 44,1 kHz nativi e 48 o 96 kHz via convertitore; 2 canali a 44,1 kHz in modalità Generic, cioè conforme alla classe; 8 uscite dirette analogiche | solo in stereo | specifiche Roland lette | multitraccia non conforme; il kernel ha una voce che rileva automaticamente la maggior parte dei dispositivi Roland vendor-specific recenti, in `quirks-table.h` letto, quindi la modalità Vendor potrebbe funzionare: plausibile, non confermato |
| Roland TD-27 | 28 canali in registrazione, 4 in riproduzione; "USB audio requires the vendor driver" | no, per l'audio | specifiche Roland lette; articoli di supporto Roland solo elencati, risposta 403 | come il V71, non confermato |
| EFNOTE PRO, modulo EFD-PRO | 12 canali in uscita a 48 kHz / 24 bit, 2 in ingresso | parola non usata; su macOS "no additional driver is required", su Windows serve ASIO | pagina EFNOTE letta | probabile UAC2 perché macOS non chiede driver, non confermato su Linux |
| EFNOTE 3 / 5 / 7 | "USB Audio (8-channel Output / 2-channel Input)"; senza ASIO Windows vede solo 2 in / 2 out | come sopra | guida di riferimento letta | come sopra |
| Yamaha DTX-PROX | numero di canali USB non dichiarato; 8 uscite individuali analogiche | "USB Class Compliant compatibility" | pagine Yamaha lette; blog di un rivenditore letto | classe dichiarata, ma probabilmente solo stereo: il multitraccia USB non è confermato |
| Alesis Strata Prime | nessuno: la guida v1.0.2 parla solo di MIDI via USB, e la guida al firmware 1.3.3 non aggiunge audio USB | sì, ma per il MIDI | FAQ e guide Alesis lette | non serve a risparmiare ingressi |

Nessun modulo trovato dichiara insieme la conformità alla classe e il multitraccia. EFNOTE è il più vicino, perché su macOS non chiede driver; i Roland hanno un percorso plausibile nel kernel ma sono vendor-specific. Anche con un modulo multitraccia resta il problema del doppio clock detto sopra. L'interfaccia va quindi dimensionata come se il modulo usasse le uscite analogiche, e l'USB del modulo va trattato come un vantaggio da verificare dopo l'acquisto.

## Raccomandazione della ricerca

Per i due usi dichiarati la scelta più motivata è la topologia A con la Scarlett 18i20 di quarta generazione: otto ingressi microfonici, due ad alta impedenza da un megaohm, phantom, dati di conversione dichiarati, e un mixer interno governabile da Linux con codice nel kernel. A spesa libera la RME UCX II porta la dichiarazione Linux più esplicita del produttore e il controllo completo dal pannello, con il vincolo di due soli ingressi microfonici. Se servono fader fisici, il TASCAM Model 12 è la scelta della topologia C. La topologia B conviene soltanto se le tracce separate non servono.

## Limiti di questa ricerca

Nessun dispositivo è stato provato: le affermazioni sul funzionamento su Linux vengono da fonti primarie dove indicato, altrimenti da forum. Il sorgente del kernel letto è il ramo principale su GitHub al 2026-09-30 e non il kernel 7.0 di Ubuntu, quindi la presenza di un identificativo nel 7.0 non è verificata; per il vincolo 6.14 della Focusrite il dubbio non si pone. Molte pagine dei produttori hanno risposto 403, fra cui il supporto Focusrite, Yamaha, Allen & Heath e il supporto Roland, e la Wayback Machine non era raggiungibile dallo strumento. Due documenti primari sono stati letti da copie ospitate da terzi, cioè la scheda Yamaha MG10XU e la guida rapida Behringer XR18. I prezzi vengono da un solo rivenditore in un solo giorno, e per 1824c e CQ-18T manca un prezzo Thomann letto; i prezzi dei moduli della batteria non sono stati raccolti. Restano aperti il numero di ingressi ad alta impedenza della UMC1820, la frequenza della UltraLite-mk5, gli ingressi microfonici e l'alta impedenza del CQ-18T, e l'EIN del Model 12, visto soltanto in un estratto di ricerca e perciò non riportato in tabella.

## Fonti

Per ogni fonte: che cosa vi si è cercato e se è stata letta. "Letta" vale anche per i documenti scaricati ed estratti in testo quando la lettura diretta era rifiutata.

| URL | Autorevole per | Stato |
|---|---|---|
| https://www.behringer.com/en/products/0805-AAN | specifiche UMC1820 del produttore | letta |
| https://www.thomann.it/behringer_umc1820.htm | prezzo e disponibilità UMC1820 | letta |
| https://www.cameratim.com/reviews/audio/behringer-u-phoria-umc1820-audio-interface/ | uso su Linux e Hi-Z a circa 750 kΩ, recensione | letta |
| https://manuals.plus/behringer/umc1820-24-bit-18x20-audiophile-manual | manuale UMC1820, copia | solo elencata, 403 |
| https://www.amazon.com/Behringer-UMC1820-Audiophile-Interface-Preamplifiers/dp/B01EXI8Y9S | rivenditore che dichiara la classe | solo elencata |
| https://www.bhphotovideo.com/c/product/1821217-REG/behringer_umc1820_audiophile_18x20_24_bit_96khz_usb.html | rivenditore che dichiara la classe | solo elencata |
| https://focusrite.com/products/scarlett-18i20 | specifiche 18i20 4th Gen | letta |
| https://support.focusrite.com/hc/en-gb/articles/208530735-Is-my-Focusrite-Product-compatible-with-Linux | posizione Focusrite su classe e Linux | solo elencata, 403 |
| https://github.com/geoffreybennett/alsa-scarlett-gui | dispositivi supportati dall'interfaccia di controllo | letta |
| https://github.com/geoffreybennett/alsa-scarlett-gui/blob/master/docs/INSTALL.md | kernel 6.14, fcp-support, firmware | letta |
| https://github.com/geoffreybennett/linux-fcp | driver FCP | solo elencata |
| https://www.thomann.it/focusrite_scarlett_18i20_4th_gen.htm | prezzo 18i20 | letta |
| https://www.presonus.com/products/Studio-1824c/tech-specs | specifiche 1824c, esaurita | letta |
| https://raw.githubusercontent.com/torvalds/linux/master/sound/usb/mixer_s1810c.c | driver del kernel per 1810c, 1824 e 1824c | letta |
| https://ratatoskr.run/linux-sound/2026/03/11364382/t | patch per la Studio 1824 | letta |
| https://lore-kernel.gnuweeb.org/linux-sound/87ecr27trj.wl-tiwai@suse.de/T/ | difetto del firmware 3.11 della 1824c | solo elencata, 502 |
| https://www.sweetwater.com/store/detail/Studio1824C--presonus-studio-1824c-usb-c-audio-interface | disponibilità 1824c | solo elencata |
| https://motu.com/en-us/products/gen5/ultralite-mk5/ | classe per iOS e CueMix solo Mac, Windows e iOS | letta |
| https://www.thomann.it/motu_ultralite_mk5.htm | prezzo e frequenza mk5 | letta |
| https://rme-audio.de/fireface-ucx-ii.html | specifiche UCX II | letta |
| https://www.rme-usa.com/fireface-ucx-ii.html | frase su classe e Linux, prezzo in USD | letta |
| https://rme-audio.de/downloads/fface_ucx2_e.pdf | manuale UCX II: modalità conforme, Linux, 96 kHz, phantom | letta, capitoli 31-36 |
| https://github.com/yonie/rme-ucx2-linux | driver in spazio utente di terze parti | letta |
| https://www.thomann.it/rme_fireface_ucx_ii.htm | prezzo UCX II | letta |
| https://archiv.rme-audio.de/en/support/techinfo/cc_mode.php | modalità conforme della UCX di prima serie | solo elencata |
| https://raw.githubusercontent.com/torvalds/linux/master/sound/usb/mixer_quirks.c | mixer dedicati del kernel | letta |
| https://raw.githubusercontent.com/torvalds/linux/master/sound/usb/quirks-table.h | voci per interi produttori, Roland compresa | letta |
| https://raw.githubusercontent.com/torvalds/linux/master/sound/usb/quirks.c | streaming Roland e Yamaha | letta |
| https://raw.githubusercontent.com/torvalds/linux/master/sound/usb/card.c | gestione UAC1, UAC2 e UAC3 nel driver generico | letta |
| https://raw.githubusercontent.com/torvalds/linux/master/sound/usb/Kconfig | opzione SND_USB_AUDIO | letta |
| https://data.musicdata.cz/Data/Yamaha/DataSheets/MG10XU.pdf | scheda Yamaha MG10XU, UAC2 | letta, copia ospitata da terzi |
| https://usa.yamaha.com/products/proaudio/mixers/mg_series_xu_model/specs.html | fonte Yamaha originale | solo elencata, 403 |
| https://usa.yamaha.com/files/download/other_assets/2/1507032/MG10XU_technical_specifications_En_B0.pdf | fonte Yamaha originale | solo elencata, 403 |
| https://usa.yamaha.com/products/contents/proaudio/musicianspa/products_usb_daw.html | fonte Yamaha originale | solo elencata, 403 |
| https://www.thomann.it/yamaha_mg10_xu.htm | prezzo MG10XU | letta |
| https://mackie.com/en/products/mixers/profxv3-series/ProFX10v3plus.html | specifiche ProFX10v3+ | letta |
| https://mackie.com/img/file_resources/PROFXV3+_MIXER_SERIES_OM.pdf | manuale Mackie | non letta, troppo grande |
| https://www.thomann.it/mackie_profx10v3.htm | prezzo e classe dichiarata dal rivenditore | letta |
| https://tascam.com/us/product/model_12 | UAC2, sistemi supportati, Hi-Z, software | letta |
| https://www.tascam.eu/en/model12 | Model 12, sito europeo | solo elencata, 403 |
| https://www.thomann.it/tascam_model_12.htm | prezzo Model 12 | letta |
| https://discourse.ardour.org/t/tascam-model-12-as-daw-controller/113297 | Model 12 su Linux Mint e Ardour, forum | letta |
| https://forum.maboxlinux.org/t/solved-tascam-model-12-usb-issues/1391 | problema del cavo USB, forum | letta |
| https://www.tascamforums.com/threads/using-a-model-12-with-linux.8831/ | Model 12 su Linux | solo elencata |
| https://zoomcorp.com/en/jp/digital-mixer-multi-track-recorders/digital-mixer-recorders/livetrak-l-12/ | specifiche L-12, classe per iOS | letta |
| https://www.soundonsound.com/reviews/zoom-livetrak-l12 | recensione L-12 | solo elencata |
| https://linuxmusicians.com/viewtopic.php?t=19580 | L-12 su Linux | solo elencata |
| https://discourse.ardour.org/t/zoom-l-20-works-great-in-ardour-on-linux/106081 | L-20 su Linux | solo elencata |
| https://www.behringer.com/en/products/0605-AAD | XR18, X AIR EDIT per Linux | letta |
| http://warehousesound.com/r/behringerX18-XR18manual.pdf | guida rapida Behringer X18/XR18 | letta, copia ospitata da terzi |
| https://behringerwiki.musictribe.com/index.php?title=X-Air_Edit_with_Ubuntu_64_bit | X-Air Edit su Ubuntu | solo elencata |
| https://www.thomann.it/behringer_x_air_xr18.htm | prezzo XR18 | letta |
| https://www.behringer.com/en/products/0606-ABV | scheda X-USB, nessuna dichiarazione di classe | letta |
| https://www.behringer.com/en/products/0603-ADP | X32 Producer | solo elencata |
| https://www.presonus.com/products/StudioLive-AR12c | specifiche AR12c, esaurita | letta |
| https://www.allen-heath.com/content/uploads/2024/06/CQ-18T-Tech-Datasheet_Iss3.pdf | scheda CQ-18T | letta |
| https://www.allen-heath.com/hardware/cq/cq-18t/ | pagina CQ-18T | solo elencata, 403 |
| https://forums.allen-heath.com/forums/topic/cq-18t-and-linux-mint-21-pulseaudio-jack | CQ-18T su Linux, forum | letta |
| https://www.trovaprezzi.it/prezzo_mixer-audio_allen_$26_heath_cq-18t.aspx | prezzo CQ-18T | solo elencata |
| https://www.roland.com/global/products/v71/specifications/ | USB del V71, Vendor e Generic | letta |
| https://www.roland.com/global/products/td-27/specifications/ | USB del TD-27, driver Vendor | letta |
| https://support.roland.com/hc/en-us/articles/360061473351-TD-27-Setup-for-sending-receiving-audio-and-MIDI-via-USB-to-the-computer | supporto Roland | solo elencata, 403 |
| https://support.roland.com/hc/en-us/articles/360042006852-TD-27-Does-USB-audio-support-parallel-output | supporto Roland | solo elencata, 403 |
| https://www.vdrums.com/forum/performance/in-the-studio/1291261-td27-usb-multitrack-recording-with-linux | TD-27 su Linux, forum | solo elencata, 403 |
| https://linuxmusicians.com/viewtopic.php?t=25248 | TD-50 su Ubuntu Studio, forum | non letta, contenuto non caricato |
| https://support.alesis.com/support/solutions/articles/69000852876-alesis-strata-prime-kit-frequently-asked-questions | classe del Prime, MIDI | letta |
| https://cdn.inmusicbrands.com/alesis/products/strata-prime/PRIME%20Drum%20Module%20-%20User%20Guide%20-%20v1.0.2.pdf | guida PRIME, USB solo MIDI | letta |
| https://cdn.inmusicbrands.com/Software/Drum/133/PRIME%20Drum%20Module%20-%20Firmware%20Update%20Guide%20-%20v1.3.3-1.pdf | firmware 1.3.3 | letta |
| https://www.alesisdrums.com/electronic-drum-kits/strata-prime/ | pagina prodotto | non letta, contenuto non leggibile |
| https://ef-note.com/products/drums/EFNOTEPRO/efnotepro.html | USB EFNOTE PRO, 12 canali | letta |
| https://www.ef-note.com/products/drums/common357/EFNOTE_3_5_7_RG_en04.pdf | USB EFNOTE 3/5/7, 8 canali | letta |
| https://uk.yamaha.com/en/musical-instruments/drums/products/electronic-trigger-modules/dtxprox/ | classe e uscite del DTX-PROX | letta |
| https://www.drum-tec.com/blog/news/yamaha-dtx-prox-sound-module-now-available | classe del DTX-PROX, rivenditore | letta |
| https://www.spinics.net/lists/alsa-devel/msg125243.html | patch del 2021 sul feedback implicito Roland | solo elencata |
| https://mailman.alsa-project.org/pipermail/alsa-devel/2021-March/183044.html | stessa patch | solo elencata |
| https://wiki.archlinux.org/title/Professional_audio/Hardware | compatibilità hardware | solo elencata |
