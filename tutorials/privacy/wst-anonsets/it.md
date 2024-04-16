---
name: Whirlpool Stats Tools - Anonsets
description: Comprendere il concetto di anonset e come calcolarlo con WST
---
![cover](assets/cover.jpeg)

*"Interrompi il collegamento lasciato dalle tue monete"*

In questo tutorial, studieremo il concetto di anonset, indicatori che ci permettono di stimare la qualità di un processo di coinjoin su Whirlpool. Copriremo il metodo di calcolo e interpretazione di questi indicatori. Dopo la parte teorica, passeremo alla pratica imparando a calcolare gli anonset di una transazione specifica utilizzando lo strumento Python *Whirlpool Stats Tools* (WST).

## Cos'è un coinjoin su Bitcoin?
**Il coinjoin è una tecnica che interrompe la tracciabilità dei bitcoin sulla blockchain**. Si basa su una transazione collaborativa con una struttura specifica dello stesso nome: la transazione coinjoin.

Le transazioni coinjoin migliorano la privacy degli utenti Bitcoin complicando l'analisi della catena per gli osservatori esterni. La loro struttura consente di unire più monete da diversi utenti in una singola transazione, oscurando così le tracce e rendendo difficile determinare i collegamenti tra gli indirizzi di input e output.

Il principio del coinjoin si basa su un approccio collaborativo: diversi utenti che desiderano mescolare i loro bitcoin depositano importi identici come input della stessa transazione. Questi importi vengono poi ridistribuiti in output di valore equivalente. Al termine della transazione, diventa impossibile associare un output specifico a un dato utente. Non esiste un collegamento diretto tra gli input e gli output, rompendo così l'associazione tra gli utenti e i loro UTXO, così come la storia di ogni moneta.
![coinjoin](assets/1.webp)

Esempio di una transazione coinjoin:
[323df21f0b0756f98336437aa3d2fb87e02b59f1946b714a7b09df04d429dec2](https://mempool.space/tx/323df21f0b0756f98336437aa3d2fb87e02b59f1946b714a7b09df04d429dec2)

Per effettuare un coinjoin garantendo che ogni utente mantenga il controllo sui propri fondi in ogni momento, il processo inizia con la costruzione della transazione da parte di un coordinatore, che poi la trasmette a ciascun partecipante. Ogni utente firma quindi la transazione dopo aver verificato che sia di suo gradimento. Tutte le firme raccolte vengono infine integrate nella transazione. Se un tentativo di dirottare fondi viene fatto da un utente o dal coordinatore, modificando gli output della transazione coinjoin, le firme risulteranno invalide, portando al rifiuto della transazione da parte dei nodi.

Esistono diverse implementazioni di coinjoin, come Whirlpool, JoinMarket o Wabisabi, ognuna con l'obiettivo di gestire il coordinamento tra i partecipanti e aumentare l'efficienza delle transazioni coinjoin.
In questo tutorial, ci concentreremo sulla mia implementazione preferita: Whirlpool, disponibile su Samourai Wallet e Sparrow Wallet. A mio avviso, è l'implementazione più efficiente per i coinjoin su Bitcoin.
## Qual è l'utilità del coinjoin su Bitcoin?
L'utilità del coinjoin risiede nella sua capacità di produrre una negabilità plausibile, annegando la tua moneta all'interno di un gruppo di monete indistinguibili. L'obiettivo di questa azione è rompere i collegamenti di tracciabilità, sia dal passato al presente che dal presente al passato.

In altre parole, un analista che conosce la tua transazione iniziale all'ingresso dei cicli di coinjoin non dovrebbe essere in grado di identificare con certezza il tuo UTXO all'uscita dei cicli di remix (analisi dall'ingresso del ciclo all'uscita del ciclo).
![coinjoin](assets/2.webp)
Al contrario, un analista che conosce il tuo UTXO all'uscita dei cicli di coinjoin dovrebbe essere incapace di determinare la transazione originale all'ingresso dei cicli (analisi dall'uscita all'ingresso del ciclo). ![coinjoin](assets/3.webp)
Per valutare la difficoltà per un analista di collegare il passato al presente e viceversa, è necessario quantificare la dimensione dei gruppi all'interno dei quali la tua moneta è nascosta. Questa misura ci dice il numero di analisi che hanno una probabilità identica. Quindi, se l'analisi corretta è annegata tra altre 3 analisi di uguale probabilità, il tuo livello di occultamento è molto basso. D'altra parte, se l'analisi corretta è all'interno di un insieme di 20.000 analisi tutte ugualmente probabili, la tua moneta è molto ben nascosta.

E precisamente, la dimensione di questi gruppi rappresenta indicatori che vengono chiamati "anonsets".

## Comprendere gli anonsets
Gli anonsets fungono da indicatori per valutare il grado di privacy di un particolare UTXO. Più specificamente, misurano il numero di UTXO indistinguibili all'interno dell'insieme che include la moneta studiata. Il requisito di un insieme UTXO omogeneo significa che gli anonsets sono solitamente calcolati sui cicli di coinjoin. L'uso di questi indicatori è particolarmente rilevante per i coinjoin di Whirlpool a causa della loro uniformità.

Gli anonsets consentono, se appropriato, di giudicare la qualità dei coinjoin. Una grande dimensione di anonset significa un livello aumentato di anonimato, poiché diventa difficile distinguere un UTXO specifico all'interno dell'insieme.

Ci sono due tipi di anonsets:
- **L'insieme di anonimato prospettico;**
- **L'insieme di anonimato retrospettivo.**
Il primo indicatore mostra la dimensione del gruppo tra cui l'UTXO studiato è nascosto alla fine del ciclo, conoscendo l'UTXO all'ingresso, cioè il numero di monete indistinguibili presenti all'interno di questo gruppo. Questo indicatore permette di misurare la resistenza della confidenzialità della moneta contro un'analisi dal passato al presente (dall'ingresso all'uscita). In inglese, il nome di questo indicatore è "*forward anonset*", o "*forward-looking metrics*". ![coinjoin](assets/4.webp)
Questa metrica stima il grado in cui il tuo UTXO è protetto contro i tentativi di ricostruire la sua storia dal suo punto di ingresso al suo punto di uscita nel processo di coinjoin.

Ad esempio, se la tua transazione ha partecipato al suo primo ciclo di coinjoin e sono stati completati altri due cicli discendenti, l'anonset prospettico della tua moneta sarebbe `13`:
![coinjoin](assets/5.webp)
Il secondo indicatore mostra il numero di possibili fonti per una data moneta, conoscendo l'UTXO alla fine del ciclo. Questo indicatore misura la resistenza della confidenzialità della moneta contro un'analisi dal presente al passato (dall'uscita all'ingresso), cioè quanto è difficile per un analista risalire all'origine della tua moneta, prima dei cicli di coinjoin. In inglese, il nome di questo indicatore è "*backward anonset*", o "*backward-looking metrics*".
![coinjoin](assets/6.webp)
Conoscendo il tuo UTXO all'uscita dei cicli, l'anonset retrospettivo determina il numero di potenziali transazioni Tx0 che avrebbero potuto costituire il tuo ingresso nei cicli di coinjoin. Nel diagramma sottostante, ciò corrisponde alla somma di tutte le bolle arancioni.
![coinjoin](assets/7.webp)

## Calcolare gli anonsets con Whirlpool Stats Tools (WST)
Per calcolare questi indicatori sulle tue monete che hanno attraversato cicli di coinjoin, puoi utilizzare uno strumento appositamente sviluppato da Samourai Wallet: *Whirlpool Stats Tools*.
Se possiedi un RoninDojo, WST è preinstallato sul tuo nodo. Puoi quindi saltare i passaggi di installazione e seguire direttamente quelli per l'uso. Per coloro che non dispongono di un nodo RoninDojo, vediamo come procedere con l'installazione di questo strumento su un computer.
Avrai bisogno di: Tor Browser (o Tor), Python 3.4.4 o superiore, git e pip. Apri un terminale. Per verificare la presenza e la versione di questi software sul tuo sistema, inserisci i seguenti comandi:
```bash
python --version
git --version
pip --version
```

Se necessario, puoi scaricarli dai rispettivi siti web:
- https://www.python.org/downloads/ (pip è incluso direttamente con Python dalla versione 3.4);
- https://www.torproject.org/download/;
- https://git-scm.com/downloads.
Una volta installati tutti questi software, da un terminale, clona il repository WST:
```bash
git clone https://code.samourai.io/whirlpool/whirlpool_stats.git
```
![WST](assets/8.webp)
Naviga nella directory WST:
```bash
cd whirlpool_stats
```

Installa le dipendenze:
```bash
pip3 install -r ./requirements.txt
```
![WST](assets/9.webp)
Puoi anche installarle manualmente (opzionale):
```bash
pip install PySocks
pip install requests[socks]
pip install plotly
pip install datasketch
pip install numpy
pip install python-bitcoinrpc
```

Naviga nella sottocartella `/whirlpool_stats`:
```bash
cd whirlpool_stats
```

Avvia WST:
```bash
python3 wst.py
```
![WST](assets/10.webp)
Avvia Tor o Tor Browser in background.

**-> Per gli utenti RoninDojo, potete riprendere il tutorial direttamente qui.**

Imposta il proxy su Tor (RoninDojo),
```bash
socks5 127.0.0.1:9050
```

o su Tor Browser a seconda di cosa stai utilizzando:
```bash
socks5 127.0.0.1:9150
```

Questa manipolazione ti permetterà di scaricare dati su OXT tramite Tor, per non divulgare informazioni sulle tue transazioni. Se sei un principiante e questo passaggio ti sembra complesso, sappi che si tratta semplicemente di indirizzare il tuo traffico internet attraverso Tor. Il metodo più semplice consiste nel lanciare il Tor Browser in background sul tuo computer, poi eseguire solo il secondo comando per connettersi tramite questo browser (`socks5 127.0.0.1:9150`).
![WST](assets/11.webp)
Successivamente, naviga nella directory di lavoro da cui intendi scaricare i dati WST utilizzando il comando `workdir`. Questa cartella servirà per memorizzare i dati transazionali che recupererai da OXT sotto forma di file `.csv`. Queste informazioni sono essenziali per calcolare gli indicatori che stai cercando di ottenere. Sei libero di scegliere la posizione di questa directory. Potrebbe essere saggio creare una cartella specificamente per i dati WST. Come esempio, optiamo per la cartella dei download. Se stai utilizzando RoninDojo, questo passaggio non è necessario:
```bash
workdir path/to/your/directory
```

Il prompt dei comandi dovrebbe poi cambiare per indicare la tua directory di lavoro.
![WST](assets/12.webp)
Poi scarica i dati dal pool contenente la tua transazione. Ad esempio, se sono nel pool `100,000 sats`, il comando è:
```bash
download 0001
```
![WST](assets/13.webp)
I codici di denominazione su WST sono i seguenti:
- Pool 0,5 bitcoin: `05`
- Pool 0,05 bitcoin: `005`
- Pool 0,01 bitcoin: `001`
- Pool 0,001 bitcoin: `0001`
Una volta scaricati i dati, caricateli. Ad esempio, se mi trovo nel pool di `100.000 sats`, il comando è:
```bash
load 0001
```

Questo passaggio richiede alcuni minuti a seconda del vostro computer. Ora è un buon momento per prepararsi un caffè! :)
![WST](assets/14.webp)
Dopo aver caricato i dati, digitate il comando `score` seguito dal vostro TXID (identificativo della transazione) per ottenere i suoi anonset:
```bash
score TXID
```

**Attenzione**, la scelta del TXID da utilizzare varia a seconda dell'anonset che si desidera calcolare. Per valutare l'anonset prospettico di una moneta, è necessario inserire, tramite il comando `score`, il TXID corrispondente al suo primo coinjoin, che è il mix iniziale effettuato con questo UTXO. D'altra parte, per determinare l'anonset retrospettivo, è necessario inserire il TXID dell'ultimo coinjoin effettuato. Per riassumere, l'anonset prospettico viene calcolato dal TXID del primo mix, mentre l'anonset retrospettivo viene calcolato dal TXID dell'ultimo mix.

WST mostra quindi il punteggio retrospettivo (*Metriche retrospettive*) e il punteggio prospettico (*Metriche prospettiche*). Ad esempio, ho preso il TXID di una moneta casuale su Whirlpool che non mi appartiene.
![WST](assets/15.webp)
La transazione in questione: [7fe6081fa4f4382be629fb2ef59029d058a22b6fd59cb31d1511fe9e0e7f32be](https://mempool.space/tx/7fe6081fa4f4382be629fb2ef59029d058a22b6fd59cb31d1511fe9e0e7f32be)

Se consideriamo questa transazione come il primo coinjoin effettuato per la moneta in questione, allora essa beneficia di un anonset prospettico di `86.871`. Ciò significa che è nascosta tra `86.871` monete indistinguibili. Per un osservatore esterno che conosce questa moneta all'inizio dei cicli di coinjoin e tenta di tracciare il suo output, si troverà di fronte a `86.871` possibili UTXO, ognuno con una probabilità identica di essere la moneta cercata.

Se consideriamo questa transazione come l'ultimo coinjoin della moneta, essa ha quindi un anonset retrospettivo di `42.185`. Ciò significa che ci sono `42.185` fonti potenziali per questo UTXO. Se un osservatore esterno identifica questa moneta alla fine dei cicli e cerca di tracciarne l'origine, si troverà di fronte a `42.185` possibili fonti, tutte con una probabilità uguale di essere l'origine cercata.
Oltre ai punteggi anonset, WST fornisce anche il tasso di diffusione del tuo output all'interno del pool basato sull'anonset. Questo altro indicatore ti permette semplicemente di valutare il potenziale di miglioramento del tuo pezzo. Questo tasso è particolarmente utile per l'anonset prospettico. Infatti, se il tuo pezzo ha un tasso di diffusione del 15%, significa che può essere confuso con il 15% dei pezzi nel pool. Questo è positivo, ma hai ancora un ampio margine di miglioramento continuando a remixare. D'altra parte, se il tuo pezzo ha un tasso di diffusione del 95%, allora ti stai avvicinando ai limiti del pool. Puoi continuare a remixare, ma il tuo anonset non aumenterà di molto.

È importante notare che gli anonset calcolati da WST non sono perfettamente accurati. Dato l'enorme volume di dati da elaborare, WST utilizza l'algoritmo *HyperLogLogPlusPlus* per ridurre significativamente l'onere associato all'elaborazione dei dati locali e alla memoria necessaria. Questo è un algoritmo che permette di stimare il numero di valori distinti in set di dati molto grandi mantenendo un'elevata precisione nel risultato. Pertanto, i punteggi forniti sono sufficientemente buoni per essere utilizzati nelle tue analisi, poiché sono stime molto vicine alla realtà, ma non dovrebbero essere interpretati come valori esatti all'unità.

In conclusione, tieni presente che non è imperativo calcolare sistematicamente gli anonset per ciascuno dei tuoi pezzi nei coinjoin. Il design stesso di Whirlpool offre già delle garanzie. Infatti, l'anonset retrospettivo raramente è una preoccupazione. Dal tuo mix iniziale, ottieni un punteggio retrospettivo particolarmente alto grazie all'eredità dei mix precedenti dal coinjoin Genesis. Per quanto riguarda l'anonset prospettico, è sufficiente mantenere il tuo pezzo nel conto post-mix per un periodo sufficientemente lungo.

Questo è il motivo per cui considero l'uso di Whirlpool particolarmente rilevante in una strategia *Hodl -> Mix -> Spend -> Replace*. A mio avviso, l'approccio più logico è mantenere la maggior parte dei propri risparmi in bitcoin in un portafoglio freddo, mentre si continua a mantenere un certo numero di pezzi nei coinjoin su Samourai per coprire le spese quotidiane. Una volta spesi i bitcoin dai coinjoin, vengono sostituiti con nuovi, al fine di ritornare alla soglia definita di pezzi mixati. Questo metodo permette di liberarsi dalla preoccupazione dei nostri anonset UTXO, rendendo il tempo necessario per l'efficacia dei coinjoin molto meno vincolante.

**Risorse esterne:**

- [Podcast in francese sull'analisi della catena](https://fountain.fm/episode/6nNoQEUHBCQR8hAXAkEx)
- [Articolo di Wikipedia su HyperLogLog](https://en.wikipedia.org/wiki/HyperLogLog)
- [Repository di Samourai per le statistiche di Whirlpool](https://code.samourai.io/whirlpool/whirlpool_stats)
- [Sito web di Whirlpool di Samourai](https://samouraiwallet.com/whirlpool)
- [Articolo su Medium in inglese sulla privacy e Bitcoin di Samourai](https://medium.com/oxt-research/understanding-bitcoin-privacy-with-oxt-part-1-4-8177a40a5923)
- [Articolo su Medium in inglese sul concetto di set di anonimato di Samourai](https://medium.com/samourai-wallet/diving-head-first-into-whirlpool-anonymity-sets-4156a54b0bc7)