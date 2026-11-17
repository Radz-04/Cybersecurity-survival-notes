# Penetration Testing

## Parte 2 — Footprinting e Scanning

**Ricognizione attiva, mappatura della rete e analisi delle scansioni**  
Reti e protocolli, host discovery, port scanning, servizi, sistemi operativi, Nmap Scripting Engine e documentazione tecnica.

**Radzivon Ferrigni Dioguardi**  
Novembre 2026

---

## 1. Scopo e perimetro della ricognizione

Il footprinting consiste nel ricostruire le caratteristiche di un ambiente partendo da evidenze disponibili: indirizzi, host, reti raggiungibili, porte, servizi e loro relazioni. La scansione è una componente della ricognizione attiva, non una scorciatoia che sostituisce una metodologia. La sequenza di lavoro più utile è **perimetro → mappatura → scoperta degli host → porte → servizi → verifica → report**.

Un elenco di indirizzi non è ancora una mappa; una porta aperta non prova che l'applicazione sia vulnerabile; una stringa di versione non dimostra da sola la presenza di una vulnerabilità. Un assessment credibile conserva sia le conclusioni sia le ragioni per cui sono state formulate.

| Elemento | Che cosa va fissato prima della scansione |
|---|---|
| Asset inclusi | IP, reti CIDR, FQDN, applicazioni, interfacce o ambienti autorizzati. |
| Esclusioni | Host sensibili, CDN, tenant di terzi, reti collegate ma fuori perimetro. |
| Attività ammesse | Discovery, scansioni TCP/UDP, fingerprinting, script, richieste applicative. |
| Limiti | Fasce orarie, velocità, numero di tentativi, indisponibilità accettabile. |
| Evidenze | Origine della scansione, data/ora, comando, versione, output, interpretazione. |

Se un nome autorizzato risolve verso l'IP di una CDN, **non** ne consegue il permesso di scansionare l'intero provider. La risoluzione dei nomi può cambiare nel tempo, soprattutto con servizi distribuiti. Analogamente, la presenza di un indirizzo in una vecchia documentazione non lo rende automaticamente ancora assegnato allo stesso soggetto.

### Parametri utilizzati negli esempi

Nei comandi Linux, `$TARGET` indica un host compreso nel perimetro e `$NETWORK` una subnet CIDR autorizzata. Impostarli secondo l'ambiente effettivo prima di utilizzare gli esempi:

```bash
TARGET='INSERISCI_IP_O_HOST_AUTORIZZATO'
NETWORK='INSERISCI_CIDR_AUTORIZZATO'
mkdir -p scans
```

Le righe di comando descrivono la procedura; non sono risultati di scansioni eseguite.

### Ricognizione passiva e ricognizione attiva

Una consultazione di un archivio pubblico è generalmente passiva *rispetto al bersaglio*, anche se comporta traffico verso il sito che ospita l'archivio. Un ICMP Echo Request, una connessione TCP e una richiesta HTTP a un server bersaglio sono invece attività attive. Non è lo strumento a determinare sempre la categoria: sono **le richieste realmente inviate**.

## 2. Network mapping: obiettivi e metodologia

Il network mapping raccoglie e organizza le informazioni tecniche sugli asset individuati nel perimetro di test. Può descrivere le subnet comprese nel perimetro, i gateway, gli host identificati, gli endpoint esposti, i nomi associati e le differenze tra segmenti. Va trattata come una rappresentazione **parziale e dipendente dal punto di osservazione**.

Un dispositivo può rispondere da una LAN e non essere raggiungibile dalla VPN. Un firewall può modificare la visibilità delle porte, un NAT può far apparire diversi servizi dietro un unico indirizzo e un sistema multihomed può avere interfacce su reti distinte. Non si deve dedurre l'intera topologia fisica da un semplice elenco di risposte.

| Informazione | Osservazione utile | Limite |
|---|---|---|
| Indirizzo IP | Host o interfaccia che risponde alle sonde. | NAT, proxy o indirizzi virtuali possono intervenire. |
| Nome host | DNS o nome comunicato da un servizio. | Il nome può essere vecchio o non verificato. |
| MAC address | Indirizzo di livello 2 osservabile sul segmento locale. | Non attraversa normalmente i router. |
| Porta | Endpoint di trasporto e suo stato dal punto di vista dello scanner. | Non identifica da sola l'applicazione. |
| Servizio/versione | Corrispondenza con una risposta applicativa. | Può essere incompleta, mascherata o personalizzata. |
| Sistema operativo | Ipotesi ricavata dallo stack TCP/IP o dai servizi. | Non è sempre un'identificazione univoca. |

L'inventario deve indicare almeno asset, punto di osservazione, tecnica utilizzata, risultati e limiti delle verifiche.

### Topologia logica e traceroute

Il risultato di un port scan non mostra come il traffico raggiunge il bersaglio. **Traceroute** permette di osservare alcuni router intermedi sfruttando il TTL in IPv4 o l'Hop Limit in IPv6: quando il valore scende a zero, un router può rispondere con un messaggio di tempo scaduto. Nmap supporta anche `--traceroute`, utilizzabile insieme a una scansione su un host autorizzato.

```bash
traceroute -n "$TARGET"   # IPv4: nodi intermedi che rispondono
sudo nmap -sS -p 443 --traceroute -n "$TARGET"
```

I percorsi possono essere asimmetrici e alcuni dispositivi non rispondono alle sonde o non espongono un indirizzo direttamente riconducibile alla loro funzione. L'output descrive **il percorso osservato**, non tutte le connessioni fisiche della rete. Anche gli hop intermedi possono appartenere a operatori esterni: valutare il percorso e le regole di ingaggio prima di eseguire sonde che li attraversano.

## 3. Modelli di rete e incapsulamento

Un protocollo definisce regole di comunicazione comprensibili ai dispositivi che lo implementano. Lo stack di rete non è un unico protocollo: combina il livello di collegamento, quello IP, il trasporto e protocolli applicativi. Il modello OSI a sette livelli è utile per ragionare sui problemi; lo stack Internet viene spesso rappresentato in quattro livelli. Questa corrispondenza è un modello di lettura, non una descrizione rigida di ogni implementazione.

| Livelli OSI | Modello Internet | Funzione e esempi |
|---|---|---|
| 7 Applicazione, 6 Presentazione, 5 Sessione | Applicazione | HTTP, DNS, SSH, SMTP; gestione dei dati e delle sessioni applicative. |
| 4 Trasporto | Trasporto | TCP, UDP; comunicazione tra endpoint e porte. QUIC utilizza UDP. |
| 3 Rete | Internet | IPv4, IPv6, ICMP e inoltro dei pacchetti. |
| 2 Collegamento, 1 Fisico | Accesso alla rete | Ethernet, Wi-Fi, frame e trasmissione sul mezzo. |

**Incapsulamento.** Per una richiesta HTTP trasportata su TCP, il dato applicativo si trova in un segmento TCP, inserito in un pacchetto IP, che viaggia nel payload di un frame Ethernet quando il collegamento usa Ethernet. Ogni livello aggiunge campi propri. In ricezione si svolge il processo inverso. Un frame può contenere anche trailer e i protocolli di controllo possono avere strutture differenti: «ogni pacchetto contiene sempre un header e un payload» è soltanto una semplificazione.

```text
Applicazione       richiesta / risposta applicativa
Trasporto          [TCP o UDP | dati applicativi]
Rete               [IPv4/IPv6 | unità di trasporto]
Collegamento       [Ethernet | pacchetto IP | FCS]
Mezzo              segnale elettrico, ottico o radio
```

Una porta TCP che rifiuta la connessione e un errore HTTP 404 sono eventi di livelli differenti: nel primo caso non è stata stabilita la connessione al servizio; nel secondo il server HTTP ha ricevuto una richiesta e ha restituito una risposta applicativa.

## 4. Indirizzamento IP, subnetting e instradamento

Gli indirizzi IP sono identificatori di livello rete assegnati a interfacce; le porte identificano endpoint di trasporto; gli indirizzi MAC sono utilizzati nella comunicazione sul collegamento locale. Per raggiungere un server remoto via Ethernet, il mittente può mantenere l'IP remoto come destinazione del pacchetto, ma inviare il **frame** al MAC del gateway. Il router ricrea il frame sul collegamento successivo.

In IPv4 su Ethernet, ARP risolve l'IP del prossimo salto nel MAC usato sul segmento locale. In IPv6 viene impiegato Neighbor Discovery, che utilizza ICMPv6. Una scansione ARP normalmente non può scoprire il MAC di server posti oltre un router.

```bash
ip -br address       # Indirizzi e interfacce locali, Linux
ip route             # Tabella di routing IPv4
ip -6 route          # Tabella di routing IPv6
ip neigh show        # Cache dei vicini IPv4/IPv6
ip route get 192.0.2.10  # Scelta di percorso, non prova di connettività
```

Il prefisso `/24` indica 24 bit di rete su IPv4 e lascia 8 bit per gli indirizzi nel blocco. La rete privata `192.168.56.0/24` contiene 256 valori; in una tipica subnet broadcast, `.0` è l'indirizzo di rete e `.255` è il broadcast. La regola «indirizzi utilizzabili = 2^n - 2» non si applica indiscriminatamente a tutti i prefissi: le reti `/31` punto-punto hanno regole specifiche.

Gli intervalli privati IPv4 sono `10.0.0.0/8`, `172.16.0.0/12` e `192.168.0.0/16`. Non confonderli con gli intervalli destinati soltanto agli esempi documentali.

## 5. IPv4 e IPv6: struttura dei pacchetti

IPv4 rappresenta gli indirizzi in 32 bit; IPv6 in 128 bit. Non sono intercambiabili: differiscono nelle intestazioni, nella scoperta dei vicini e nella gestione della frammentazione. La struttura del pacchetto è importante perché un firewall o uno strumento di cattura può distinguere traffico usando indirizzi, protocollo, TTL/Hop Limit, frammenti e altri campi.

### Campi principali dell'header IPv4

| Campo | Dimensione | Perché è utile |
|---|---:|---|
| Version | 4 bit | Identifica IPv4. |
| IHL | 4 bit | Indica la lunghezza dell'intestazione in parole da 32 bit. |
| DSCP / ECN | 8 bit | Segnalazione di trattamento del traffico e congestione. |
| Total Length | 16 bit | Lunghezza complessiva del pacchetto IPv4, intestazione inclusa. |
| Identification | 16 bit | Associazione dei frammenti ai fini del riassemblaggio. |
| Flags / Fragment Offset | 3 / 13 bit | Controllo e posizione dei frammenti. |
| TTL | 8 bit | Si riduce attraversando i router e limita la durata del pacchetto. |
| Protocol | 8 bit | Identifica il contenuto, per esempio ICMP=1, TCP=6, UDP=17. |
| Header Checksum | 16 bit | Controlla l'intestazione IPv4, non l'intero contenuto. |
| Source / Destination | 32 bit ciascuno | IP sorgente e destinazione. |
| Options | Variabile | Campi opzionali, non sempre presenti. |

Un header IPv4 senza opzioni è normalmente lungo **20 byte**. Il TTL osservato non coincide con il numero di hop attraversati: senza conoscere il valore iniziale non si può ricostruire esattamente il percorso, né usarlo come prova definitiva del sistema operativo. Un router IPv4 può frammentare un pacchetto se le condizioni lo richiedono e il flag DF lo consente; un router IPv6 **non** frammenta i pacchetti in transito.

La versione IPv6 utilizza il campo **Hop Limit** e il campo **Next Header**; eventuali informazioni aggiuntive possono stare in extension headers. In un confronto di catture bisogna riconoscere il protocollo effettivo, non trasporre meccanicamente la struttura IPv4 su IPv6.

## 6. ICMP e diagnostica di rete

ICMP accompagna il funzionamento di IP: fornisce messaggi di controllo, segnalazioni di errore e diagnostica. `ping` utilizza tipicamente ICMP Echo Request/Echo Reply su IPv4. I messaggi ICMP sono individuati da **tipo e codice**, non da porte TCP o UDP. In ICMPv4, secondo la specifica ICMP (RFC 792), Echo Request è tipo 8, codice 0; Echo Reply è tipo 0, codice 0.

| Risposta | Interpretazione prudente |
|---|---|
| Echo Reply | Un dispositivo ha risposto alla sonda ICMP. |
| Destination Unreachable | Un host o router segnala un problema di consegna, secondo tipo/codice. |
| Time Exceeded | Il pacchetto è scaduto per TTL o altra condizione prevista dal protocollo. |
| Nessuna risposta | La sonda può essere filtrata, persa, limitata o il target può non essere attivo. |

```bash
ping -c 3 "$TARGET"      # Linux: invia tre Echo Request
traceroute "$TARGET"    # Se disponibile; il percorso può risultare incompleto
```

Traceroute invia sonde con TTL/Hop Limit progressivo per osservare risposte da dispositivi lungo il percorso. I router possono limitare o ignorare le risposte, e i percorsi possono essere asimmetrici. `* * *` non dimostra che in quel punto esista un guasto o un firewall specifico.

## 7. TCP: connessioni, porte e flag

TCP offre un flusso di byte ordinato con meccanismi di acknowledgment, ritrasmissione, controllo del flusso e congestione. L'apertura convenzionale di una connessione utilizza il **three-way handshake** descritto dallo standard TCP.

```text
Client                              Server
  | ------ SYN, seq=x -------------> |
  | <----- SYN+ACK, seq=y ---------- |
  | ------ ACK, ack=y+1 -----------> |
  |             ESTABLISHED          |
```

Il client invia SYN, il server risponde con SYN+ACK e il client conferma con ACK. Questo è il meccanismo a cui si riferiscono le scansioni SYN e Connect. La presenza di SYN+ACK è un indizio che il sistema ha accettato il tentativo di apertura; l'identificazione di ciò che ascolta su quella porta richiede ulteriori verifiche.

| Flag TCP | Significato essenziale |
|---|---|
| SYN | Sincronizzazione per l'avvio della connessione. |
| ACK | Conferma di sequenze ricevute. |
| RST | Reset: rifiuto o interruzione di una connessione. |
| FIN | Chiusura ordinata di una direzione del flusso. |
| PSH | Richiesta di rendere disponibili prontamente i dati all'applicazione. |
| URG | Segnalazione dell'uso del campo urgent pointer. |

Le porte sono numeri da **0 a 65535**, sia per TCP sia per UDP, ma l'uso di un servizio è convenzionale e non garantito dalla porta. La IANA distingue System Ports (0-1023), User Ports (1024-49151) e Dynamic/Private Ports (49152-65535). Un server HTTP può ascoltare su 80, 8080 o qualsiasi altra porta configurata: la risposta applicativa va identificata, non indovinata.

## 8. UDP: caratteristiche e limiti delle sonde

UDP scambia datagrammi senza handshake di trasporto e non fornisce, da solo, garanzia di consegna, ordinamento o ritrasmissione. Il software applicativo può aggiungere meccanismi propri. DNS, NTP, DHCP e varie applicazioni multimediali utilizzano UDP; QUIC opera sopra UDP. Dire che UDP sia «sempre più veloce» o che una porta silenziosa sia certamente aperta è scorretto.

| Comportamento osservato da una sonda UDP | Valutazione |
|---|---|
| Risposta applicativa UDP valida | Forte evidenza che il servizio sulla porta sia raggiungibile. |
| ICMPv4 Destination Unreachable, port unreachable | Tipicamente indicazione di porta `closed`. |
| Alcuni altri errori ICMP | Possibile `filtered`, a seconda del tipo/codice. |
| Nessuna risposta dopo i tentativi | Possibile `open\|filtered`: l'incertezza non è risolta. |

Il servizio può ignorare datagrammi non validi e i messaggi ICMP possono essere soggetti a rate limiting. La classificazione dipende da risposte e tecnica: Nmap invia, per alcune porte UDP, payload specifici del servizio per aumentare la possibilità di riconoscimento. `-sV` può aggiungere sonde applicative sui risultati pertinenti.

## 9. Origine della scansione e visibilità di rete

Prima di scansionare una rete è utile sapere da **quale interfaccia, namespace e segmento** partirà il traffico. In una macchina virtuale NAT, la visibilità differisce da una VM collegata a una rete host-only; un container può essere su una rete bridge isolata; una VPN può introdurre route o policy aggiuntive.

```bash
ip -br address
ip route
ip neigh show
nmap --version
```

Una tabella di routing dice quale percorso il kernel sceglie, non se il server remoto risponderà. Una cache ARP/ND contiene vicini già osservati o risolti: non è un inventario esaustivo. Anche l'interfaccia di cattura conta: ascoltare sulla scheda sbagliata può far sembrare assente il traffico effettivamente inviato.

Un inventario minimo annota: dispositivo scanner, sistema operativo, interfaccia, indirizzo sorgente, eventuale VPN o NAT, subnet consentite e ora dei test. Questo contesto permette di spiegare differenze tra risultati eseguiti in momenti o punti diversi.

## 10. Tecniche di host discovery

L'host discovery cerca risposte a sonde progettate per distinguere indirizzi che mostrano attività da quelli che non rispondono. «Host down» nell'output è una **conclusione operativa della discovery effettuata**, non una prova fisica che il dispositivo sia spento. La scelta delle sonde dipende dal tipo di collegamento, dai privilegi, dal routing e dalle regole di filtraggio.

| Tecnica | Meccanismo | Considerazione |
|---|---|---|
| ARP IPv4 | Richiesta MAC sul link locale. | Spesso efficace in LAN; non attraversa router. |
| ND IPv6 | Neighbor Solicitation/Advertisement. | Scoperta locale IPv6, non ARP. |
| ICMP Echo | Richiesta/risposta echo. | Può essere bloccata da policy. |
| TCP SYN ping | Sonda SYN a porte indicate. | Può rivelare attività anche se ICMP è bloccato. |
| TCP ACK ping | Sonda ACK e osservazione delle risposte. | Dipende dai firewall e dallo stack. |
| UDP ping | Datagramma UDP a porta selezionata. | Spesso si basa sulla risposta applicativa o ICMP. |
| IP protocol ping | Sonda di un protocollo IP scelto. | L'esito dipende dal protocollo e dalla risposta. |

Nmap può combinare più tecniche: non è corretto considerare l'host discovery sempre sinonimo di ping ICMP. Su una rete Ethernet locale, Nmap può preferire ARP/ND anche quando sono state indicate opzioni ICMP o `-Pn`, salvo controlli appropriati.

## 11. Nmap: host discovery e ping sweep

Un **ping sweep** applica sonde di discovery a un insieme di indirizzi. La terminologia storica suggerisce ICMP, ma nel linguaggio operativo si possono usare TCP o ARP. La modalità di Nmap destinata alla sola scoperta degli host è `-sn`: non effettua il port scan ordinario, ma può comunque inviare traffico di discovery e risolvere nomi.

```bash
# Solo target autorizzati; non avvia il port scan delle porte.
sudo nmap -sn -n "$NETWORK"

# Scoperta mediante ICMP Echo su target non locale.
sudo nmap -sn -PE -n "$TARGET"

# Discovery TCP a due porte selezionate.
sudo nmap -sn -PS22,443 -n "$TARGET"
sudo nmap -sn -PA80,443 -n "$TARGET"

# Sonde UDP a porte specifiche.
sudo nmap -sn -PU53,161 -n "$TARGET"

# Nella LAN IPv4, discovery ARP esplicita.
sudo nmap -sn -PR -n "$NETWORK"
```

**`-PE` non forza automaticamente l'uso esclusivo di ICMP in una LAN**: il comportamento locale ARP è una scelta separata. `-n` evita la risoluzione DNS dei nomi, riducendo rumore e dipendenze esterne. Le sonde UDP e TCP possono produrre una risposta da un servizio o da un filtro, e tali risultati vanno interpretati.

### Altre sonde di discovery

Oltre a ICMP Echo (`-PE`), Nmap supporta ICMP Timestamp (`-PP`) e Address Mask (`-PM`); queste richieste non sono necessariamente supportate dai sistemi moderni. Il protocol ping (`-PO`) prova numeri di protocollo IP anziché porte TCP o UDP. Le opzioni dipendono da famiglia IP, privilegi e infrastruttura: non vanno sommate indiscriminatamente.

```bash
sudo nmap -sn -n -PP "$TARGET"          # ICMP Timestamp, IPv4
sudo nmap -sn -n -PO1,6,17 "$TARGET"    # IP protocol ping
```

Se si vuole distinguere la risposta ICMP da quella ARP sulla rete locale, è possibile utilizzare `--disable-arp-ping` **solo quando appropriato al collegamento e al metodo scelto**. Nelle reti IPv6, la discovery locale impiega Neighbor Discovery anziché ARP.

### Discovery ARP con altri strumenti

Nelle LAN IPv4, **arp-scan** e **netdiscover** possono integrare Nmap nella ricerca degli indirizzi che rispondono a richieste ARP. A differenza delle sonde IP instradate, ARP opera sul segmento di collegamento locale e non consente di ottenere una vista completa delle reti remote.

```bash
# Esempi per una rete di laboratorio collegata a eth0 e autorizzata.
sudo arp-scan --interface=eth0 192.168.56.0/24
sudo netdiscover -i eth0 -r 192.168.56.0/24
```

La sintassi va verificata con l'help delle versioni installate. `192.168.56.0/24` è un esempio di rete privata, non una rete automaticamente autorizzata; `eth0` va sostituita con l'interfaccia effettiva. Un MAC individuato può appartenere a una macchina virtuale, a un proxy ARP o a un apparato che risponde per altri indirizzi. Le risposte vanno correlate con routing e configurazione locale.

### Opzione `-Pn`: scansione senza discovery preliminare

```bash
nmap -Pn -sT -p 22,80,443 -n "$TARGET"
```

`-Pn` **salta la fase di discovery** e tratta i target come candidati attivi per le fasi richieste. Non dimostra che siano online. Applicarlo a tutta una grande subnet comporta tentativi sulle porte di tutti gli indirizzi, con possibili tempi e volumi molto maggiori. È utile quando le sonde preliminari sono filtrate, ma va limitato ai sistemi effettivamente compresi nello scope. Anche con `-Pn`, su una LAN possono essere usate richieste ARP/ND necessarie al collegamento.

### Nmap su Windows e Npcap

Nmap è disponibile anche su Windows. Per inviare e acquisire pacchetti raw si appoggia normalmente a **Npcap**; le possibilità effettive dipendono dal driver, dall'interfaccia di rete e dai privilegi. In un terminale PowerShell, dopo avere verificato la subnet autorizzata, i comandi di discovery hanno la stessa sintassi Nmap:

```powershell
$Network = 'INSERISCI_CIDR_AUTORIZZATO'
nmap --version
nmap -sn -n $Network
```

Su alcuni collegamenti VPN e interfacce non supportate per il traffico raw, i risultati possono differire da quelli su Ethernet. Senza Npcap, per un **singolo host** autorizzato il metodo TCP Connect con `-sT -Pn` può funzionare tramite l'API socket del sistema, ma non equivale a una discovery degli host. Non usare `-Pn` indiscriminatamente su una rete intera.

## 12. Nmap: target, porte ed esclusioni

Un CIDR, un file di target e una lista di porte non sono equivalenti: descrivono dimensioni diverse dello scan. L'opzione `-sL` consente di **elencare i target** prima di inviare sonde di scanning; senza `-n`, possono comunque verificarsi interrogazioni DNS per risolvere nomi. `-iL` legge target da file; `--exclude` e `--excludefile` permettono di rimuovere indirizzi in modo esplicito.

```bash
nmap -sL -n "$NETWORK"
nmap -sL -n -iL targets.txt --excludefile esclusi.txt

# Solo porte elencate.
nmap -sT -p 22,80,443 "$TARGET"

# Le 20 porte più diffuse del database di Nmap.
nmap -sT --top-ports 20 "$TARGET"

# Tutte le porte TCP numerate 1-65535.
nmap -sT -p- "$TARGET"
```

Con `-p-` Nmap scandisce le porte **1-65535** (non tutte le possibili combinazioni di trasporto, a meno di attivare anche i relativi metodi). Senza opzioni di porta, Nmap controlla per impostazione tipica le **1.000 porte TCP più comuni**, sulla base del proprio database `nmap-services`: non le prime 1.000 porte numeriche. `-F` usa una selezione ridotta. L'ordine di scansione può essere modificato con `-r`.

Il controllo di una lista di destinazioni non deve modificare il perimetro: prima di avviare uno scan verificare FQDN, indirizzi risultanti dalla risoluzione, eventuali filtri di esclusione e host di terzi.

## 13. Stati delle porte in Nmap

Il port scanning invia pacchetti a endpoint di trasporto e deduce uno stato osservando risposte, errori e timeout. **Gli stati sono la valutazione di Nmap da una posizione e con una tecnica specifica**, non proprietà eterne della porta. Lo stesso server può apparire `open` dalla rete interna e `filtered` da Internet.

| Stato Nmap | Che cosa afferma e che cosa non afferma |
|---|---|
| `open` | Esiste un endpoint che accetta il tipo di comunicazione esaminato. Non prova assenza di autenticazione. |
| `closed` | La porta è raggiungibile ma non risulta in ascolto un servizio per quel trasporto. |
| `filtered` | Il filtraggio impedisce di determinare con certezza se la porta sia aperta. |
| `unfiltered` | La porta risulta accessibile, ma questa tecnica non ne stabilisce apertura o chiusura. |
| `open\|filtered` | Le risposte disponibili non distinguono un servizio aperto dal filtraggio. |
| `closed\|filtered` | Non è possibile separare le condizioni di chiusura e filtraggio per determinate tecniche. |

Il campo **REASON**, disponibile con `--reason`, è essenziale per capire quale evento abbia provocato la classificazione: `syn-ack`, `reset`, una risposta ICMP o la mancata risposta. La vista di Nmap deve essere letta insieme al metodo di scanning e alle condizioni di rete.

```bash
nmap -sT -p 22,80,443 --reason -n "$TARGET"
```

Uno stesso servizio potrebbe essere disponibile solo in IPv6, solamente dalla VPN, da un determinato indirizzo sorgente o durante una finestra di attività. Per questo risultati ottenuti con modalità o punti di osservazione differenti non andrebbero sommati senza contesto.

## 14. TCP port scanning: SYN, Connect, ACK e altri metodi

### TCP SYN scan (`-sS`)

Nel SYN scan lo scanner costruisce una sonda TCP SYN. Una risposta SYN+ACK indica tipicamente `open`; un RST indica `closed`; assenza di risposte o errori pertinenti può portare a `filtered`. Nella scansione ordinaria Nmap non completa il three-way handshake come farebbe un'applicazione. Su Linux e sistemi Unix sono normalmente necessari privilegi per inviare pacchetti raw.

```bash
sudo nmap -sS -p 22,80,443 --reason -n "$TARGET"
```

### TCP Connect scan (`-sT`)

Il Connect scan si affida alla chiamata di sistema `connect()` per tentare di stabilire una connessione TCP. È disponibile senza privilegi raw e viene spesso usato quando il SYN scan non è praticabile. Può essere registrato più direttamente dai servizi che accettano le connessioni e ha un comportamento diverso dal SYN scan. Non è «una scansione passiva».

```bash
nmap -sT -p 22,80,443 --reason -n "$TARGET"
```

### TCP ACK scan (`-sA`)

L'ACK scan è pensato principalmente per analizzare **filtri e regole di firewall**, non per determinare se l'applicazione sia in ascolto. Una risposta RST può indicare che la sonda è arrivata a un endpoint e produrre `unfiltered`; l'assenza di risposta o specifici errori porta a `filtered`. Non si deve convertire `unfiltered` in `open`.

```bash
sudo nmap -sA -p 22,80,443 --reason -n "$TARGET"
```

### Null, FIN e Xmas

Queste tecniche cambiano i flag TCP inviati: nessun flag (`-sN`), FIN (`-sF`) o combinazione FIN/PSH/URG (`-sX`). Si basano su differenze di risposta previste da determinati comportamenti TCP. Possono restituire `open|filtered`, essere ostacolate dai firewall e comportarsi diversamente con stack non conformi alle assunzioni dello scanner, inclusi numerosi sistemi Windows. **Non sono tecniche universalmente migliori o invisibili**.

```bash
sudo nmap -sN -p 80,443 -n "$TARGET"
sudo nmap -sF -p 80,443 -n "$TARGET"
sudo nmap -sX -p 80,443 -n "$TARGET"
```

Altri metodi, come TCP Window e Maimon, dipendono da comportamenti specifici dei sistemi. L'IP protocol scan `-sO` riguarda **numeri di protocollo IP**, non porte; l'idle scan usa un host intermediario e richiede condizioni particolari. Tali opzioni non sostituiscono SYN, Connect e verifiche applicative nel lavoro ordinario. Il manuale ufficiale documenta condizioni e limiti di ogni metodo.

### Risposte alle sonde TCP

| Sonda / osservazione tipica | Esito prevalente | Limite |
|---|---|---|
| SYN → SYN+ACK | Porta TCP `open` | Identità del servizio da verificare. |
| SYN → RST | Porta TCP `closed` | Un intermediario può alterare le risposte. |
| SYN → nessuna risposta | Possibile `filtered` | Timeout e perdite richiedono cautela. |
| ACK → RST | `unfiltered` nell'ACK scan | Non indica apertura della porta. |

## 15. UDP port scanning

Per UDP, il tentativo di scoprire servizi è spesso più lento e ambiguo rispetto a TCP perché molte applicazioni non rispondono a un datagramma non valido. Con `-sU`, Nmap invia sonde UDP e interpreta risposte applicative o messaggi ICMP. Servizi come DNS o SNMP richiedono di considerare il proprio protocollo, la versione e le condizioni del laboratorio.

```bash
# Solo un insieme limitato di porte UDP autorizzate.
sudo nmap -sU -p 53,123,161 --reason -n "$TARGET"

# Rilevazione del servizio sulle porte UDP scelte.
sudo nmap -sU -sV -p 53,123,161 -n "$TARGET"

# Nello stesso comando, porte diverse per TCP e UDP.
sudo nmap -sS -sU -p T:22,80,443,U:53,123 -n "$TARGET"
```

La sintassi `T:` e `U:` in `-p` separa correttamente i set di porte, ma ha significato soltanto se sono stati selezionati i relativi metodi TCP e UDP. La porta numerica è lo stesso valore, mentre gli endpoint `53/tcp` e `53/udp` sono distinti.

Per un servizio che non risponde, ripetere indiscriminatamente una scansione UDP ad alta velocità non è una garanzia di maggiore accuratezza: eventuali rate limit ICMP e perdita di pacchetti possono complicare ulteriormente l'interpretazione. Conviene circoscrivere le porte, esaminare `--reason` e, quando permesso, richiedere una risposta applicativa riconoscibile.

## 16. Identificazione di servizi e versioni

Un nome nella colonna `SERVICE` può essere dedotto dalla mappatura tradizionale della porta. **Non è necessariamente la prova che quella particolare applicazione stia rispondendo.** Con `-sV`, Nmap tenta una rilevazione applicativa mediante sonde e confronti con il database `nmap-service-probes`. Una stessa porta può ospitare HTTP, SSH o un protocollo proprietario diverso da quello atteso.

```bash
nmap -sT -sV -p 22,80,443,8080 -n "$TARGET"

# Riduce l'intensità delle sonde di versione.
nmap -sT -sV --version-light -p 22,80,443 -n "$TARGET"

# Esamina la motivazione delle classificazioni.
nmap -sT -sV --reason -p 22,80,443 -n "$TARGET"
```

`--version-intensity` accetta livelli **0-9**; un livello maggiore amplia le sonde candidate e può aumentare durata, interazioni e carico. `--version-light` imposta una modalità meno intensa. Una stringa di versione in output può derivare da banner modificati, patch mantenute dal distributore o proxy: non è automaticamente una prova di vulnerabilità.

### Verifica dei protocolli applicativi

Una volta identificato un possibile protocollo, è opportuno verificare le informazioni con una richiesta adatta al servizio, se permessa dalle regole di ingaggio. `curl` è utile per protocolli HTTP e TLS; un collegamento TCP grezzo può essere utile per banner testuali, ma non sostituisce un client che conosce il protocollo.

```bash
# Solo se sulla porta è stato identificato un endpoint HTTP.
curl -I --max-time 5 "http://${TARGET}:8080/"

# Solo se è stato identificato un endpoint HTTPS.
curl -kI --max-time 5 "https://${TARGET}:8443/"
```

Nella seconda riga `-k` disabilita la verifica del certificato: è utile solo per diagnostica **quando necessario**, e non deve essere usato come impostazione di sicurezza abituale. La risposta `Server:` di HTTP può essere rimossa, alterata o prodotta da un reverse proxy.

Attenzione a dispositivi speciali: la documentazione Nmap segnala che alcune stampanti ascoltano su `9100/tcp` e potrebbero stampare letteralmente payload ricevuti. Per questo Nmap evita di default alcune sonde di rilevazione versione su quella porta. Le verifiche invasive non sono adatte a qualunque ambiente.

## 17. Rilevamento del sistema operativo

Con `-O`, Nmap confronta le risposte dello stack TCP/IP con impronte raccolte nel proprio database. Esamina, fra le altre cose, caratteristiche di pacchetti TCP, UDP e ICMP. Non legge magicamente il nome del sistema remoto e non può garantire una corrispondenza esatta. Le condizioni migliorano quando è disponibile almeno **una porta TCP aperta e una chiusa**.

```bash
sudo nmap -O -p 22,80,443,65000 -n "$TARGET"
```

La porta `65000` è qui soltanto una **porta candidata** aggiunta per cercare una risposta da una porta chiusa; non è garantito che lo sia. Se l'ambiente permette solo poche porte, il fingerprinting può essere debole. Un firewall, un bilanciatore o un proxy possono fornire risposte attribuibili a componenti intermedi e non alla macchina che esegue l'applicazione.

È utile distinguere tre livelli di attendibilità:

| Informazione | Affidabilità indicativa |
|---|---|
| Risposta a una sonda e flag osservati | Evidenza diretta di rete, salva la presenza di intermediari. |
| Corrispondenza con fingerprint OS | Inferenza probabilistica. |
| Banner/identificativo dichiarato dal servizio | Informazione dichiarata, verificabile con altre fonti. |

`-A` attiva più funzionalità assieme (**rilevamento OS, service/version detection, script predefiniti e traceroute**), non è un sinonimo innocuo di `-O`. In un test misurato è preferibile attivare ogni funzione consapevolmente.

## 18. Nmap Scripting Engine (NSE)

Il **Nmap Scripting Engine** estende le scansioni usando script Lua che possono raccogliere dati, esaminare servizi o svolgere altre azioni. Le categorie includono, tra le altre, `default`, `discovery`, `safe`, `auth`, `brute`, `vuln`, `intrusive`, `exploit` e `dos`. Una categoria è un'etichetta di organizzazione, **non una garanzia di impatto nullo**. Prima di attivare uno script bisogna leggerne descrizione, prerequisiti e comportamento.

```bash
# Elenca la documentazione dello script disponibile localmente.
nmap --script-help http-title
nmap --script-help http-headers
nmap --script-help ssl-cert

# Esempi mirati a servizi già verificati.
nmap -sT -p 80,8080 --script http-title,http-headers "$TARGET"
nmap -sT -p 443 --script ssl-cert "$TARGET"
nmap -sT -p 22 --script ssh-hostkey "$TARGET"
```

Gli script scelti sono documentati individualmente su **NSEDoc**, con categorie e parametri. Anche uno script destinato all'identificazione può generare più richieste, ottenere certificati o avviare una negoziazione protocollare. Gli script che tentano credenziali, attacchi, denial of service o manipolazioni richiedono autorizzazione specifica e non rientrano in un controllo generico di footprinting.

### Opzioni principali di NSE

| Opzione | Effetto |
|---|---|
| `-sC` | Esegue la selezione di script `default`. |
| `--script http-title` | Seleziona uno script per nome. |
| `--script-help nome` | Mostra l'aiuto di uno script. |
| `--script-args` | Passa argomenti agli script selezionati. |
| `--script-trace` | Mostra interazioni applicative degli script, utile per diagnosi. |

Evita di usare indiscriminatamente `--script all`: include script con finalità e impatti molto differenti. Anche una generica scansione `-sC` dovrebbe essere verificata rispetto alle policy del test.

## 19. Firewall, NAT e IDS/IPS

Un firewall può filtrare traffico per indirizzo sorgente, destinazione, porta, protocollo, stato della connessione o policy applicative. Un IPS può intervenire sul contenuto e sul ritmo delle sonde. Un NAT può modificare l'indirizzamento osservabile, mentre un reverse proxy può rispondere per più backend. Perciò lo scanner vede **l'intero percorso di rete**, non solo il servizio finale.

Un filtro **stateless** valuta i pacchetti in base a regole senza conservare necessariamente lo stato delle connessioni. Un filtro **stateful** può invece tener conto delle sessioni in corso, per esempio distinguendo tentativi SYN iniziali da traffico di una connessione già stabilita. Una scansione ACK può produrre indicazioni sul filtraggio, ma da sola non identifica con certezza tutte le regole o la tecnologia del firewall. Le differenze tra `filtered`, `closed` e `open|filtered` devono essere lette alla luce delle risposte ricevute, non trasformate in ipotesi assolute.

```bash
# Confronta due tecniche senza cambiare l'insieme delle porte.
sudo nmap -sS -p 22,80,443 --reason -n "$TARGET"
sudo nmap -sA -p 22,80,443 --reason -n "$TARGET"
```

Se la prima scansione segnala `filtered` e la seconda segnala `unfiltered`, non è dimostrato un guasto e non si può dedurre il nome del produttore del firewall. Le risposte possono dipendere da filtraggio, temporizzazione, NAT, host firewall o differenti condizioni del percorso.

Per documentare un filtro conviene associare a ogni test: punto di origine, interfaccia, famiglia IP, data/ora, protocollo, porta, sonde effettivamente inviate, risposte e motivazione restituita dallo scanner.

## 20. Tecniche di scansione avanzate e analisi dei filtri

La guida Nmap documenta opzioni che modificano i pacchetti o la loro provenienza: frammentazione (`-f`, `--mtu`), source port (`-g` o `--source-port`), data length, decoy (`-D`), sorgente dichiarata (`-S`) e altre tecniche. Sono funzionalità tecniche, **non un metodo garantito per superare un firewall o restare invisibili a un IDS**. Possono alterare la misurazione e produrre traffico verso indirizzi non previsti.

| Meccanismo | Che cosa cambia | Limite importante |
|---|---|---|
| Frammentazione | Suddivide alcuni pacchetti IP raw. | Apparati e stack possono ricomporre o rifiutare i frammenti. |
| Source port | Imposta la porta sorgente delle sonde supportate. | I filtri stateful e applicativi possono ignorare questo espediente. |
| Decoy | Introduce ulteriori identità sorgente apparenti. | Può coinvolgere terzi e confondere i log; non usarlo senza scope esplicito. |
| Spoofed source | Modifica l'IP sorgente dei pacchetti. | Le risposte non tornano necessariamente allo scanner; possibili controlli anti-spoofing. |
| Payload/MTU modificati | Varia contenuto o segmentazione delle sonde. | Può cambiare il modo in cui target e dispositivi intermedi rispondono. |

`-f` e `--mtu` non sono combinabili fra loro; la frammentazione di Nmap è limitata a determinati metodi raw e non è applicabile al Connect scan. Se si ipotizza una regola di filtraggio, il confronto va progettato con sonde **esplicitamente autorizzate e ripetibili**, preferibilmente in un ambiente controllato. Non è corretto descrivere il filtraggio come «aggirato» soltanto perché una porta cambia stato.

La strategia prioritaria non è eludere i sistemi di protezione: è **capire quali test sono consentiti, quali sonde sono state bloccate e che cosa si può concludere con le evidenze disponibili**.

## 21. Temporizzazione e prestazioni delle scansioni

Il tempo di una scansione dipende dal numero di host, porte, tecniche, RTT, rate limiting, retransmission, privilegi e script abilitati. Nmap impiega logiche adattive: un timeout in un ambiente congestionato non dovrebbe essere letto come prova di una porta filtrata senza controlli. Le opzioni di timing incidono anche sull'impatto del test.

| Opzione | Utilità | Cautela |
|---|---|---|
| `-T0` … `-T5` | Template di temporizzazione. | `-T4` e `-T5` non sono sempre appropriati per reti fragili. |
| `--max-rate N` | Tetto richiesto alla frequenza delle sonde. | Non equivale a una garanzia di impatto nullo sul servizio. |
| `--scan-delay` | Ritardo tra sonde verso un host. | Aumenta notevolmente la durata. |
| `--max-retries N` | Limita retransmission. | Valori troppo bassi aumentano i falsi negativi. |
| `--host-timeout` | Limita il tempo per un host. | La scansione può terminare incompleta. |
| `--max-rtt-timeout` | Limita il timeout RTT. | Troppo breve per connessioni ad alta latenza. |

```bash
# Esempio con intervallo ristretto, limite prudente e output persistente.
nmap -sT -p 22,80,443 --max-rate 20 --reason \
     -oA scans/tcp-controllato "$TARGET"
```

In generale la prima ottimizzazione è **scansionare il necessario**: selezionare host e porte, evitare script e rilevazioni di versione quando non servono, limitare le ripetizioni. Cercare maggiore velocità con `--min-rate` o `--max-retries 0` può ridurre l'affidabilità o aumentare il carico; non è automaticamente un miglioramento.

## 22. Formati di output e conservazione dei risultati

Nmap distingue output destinato alla lettura umana, output XML strutturato e un formato storico grepable. Conservare l'output è importante perché una sintesi scritta a posteriori può omettere porte, date, hostname, errori o motivazioni della classificazione.

```bash
# Tre file nello stesso comando: .nmap, .xml, .gnmap.
nmap -sT -sV -p 22,80,443 -oA scans/servizi "$TARGET"

# Solo output normale.
nmap -sT -p 22,80 -oN scans/porte.nmap "$TARGET"

# Solo XML per importazioni o parser.
nmap -sT -p 22,80 -oX scans/porte.xml "$TARGET"
```

| Formato | Impiego preferibile |
|---|---|
| `.nmap` (`-oN`) | Lettura manuale, revisione rapida e archiviazione testuale. |
| `.xml` (`-oX`) | Elaborazione strutturata, importazione, confronto automatizzato. |
| `.gnmap` (`-oG`) | Legacy grepable, mantenuto per compatibilità; preferire XML per nuove automazioni. |
| `-oA base` | Produce i tre formati principali in una sola esecuzione. |

### Verbosity, debugging e packet trace

```bash
nmap -sT -p 22,80,443 -v --reason "$TARGET"

# Su un solo host e poche porte: mostra il traffico osservato da Nmap.
sudo nmap -sS -p 80 --packet-trace -n "$TARGET"
```

`-v` aumenta la verbosità; `-d` attiva informazioni di debug; `--reason` rende più esplicita la ragione dello stato, mentre `--packet-trace` mostra pacchetti inviati e ricevuti. Una traccia su un'intera rete può produrre un volume enorme: limitarla a pochi host e porte.

Un file XML contiene attributi che possono includere argomenti, versione scanner, ora, host, stati, servizi e altri risultati. Non va trattato come testo da interpretare con espressioni regolari fragili se è disponibile un parser XML.

### Zenmap e visualizzazione dei risultati

**Zenmap** è l'interfaccia grafica ufficiale di Nmap. Permette di avviare scansioni con profili, leggere l'output del comando e navigare i risultati per host, porte e servizi. Non modifica il significato degli stati e non trasforma una stima OS in una prova certa. Quando si condivide una schermata di Zenmap è opportuno conservare anche la **riga Nmap corrispondente** e l'output originale: una vista grafica isolata può nascondere informazioni importanti, come l'esatto insieme di opzioni e target.

## 23. Analisi dei pacchetti con Wireshark e tcpdump

La cattura dei pacchetti aiuta a distinguere **che cosa è stato inviato** da **che cosa Nmap ha concluso**. È particolarmente utile quando si incontrano differenze tra un SYN scan e un Connect scan, un `open|filtered` UDP o un host che risponde da una rete ma non da un'altra.

```bash
# Prima verificare il nome dell'interfaccia corretta.
ip -br link

# Cattura limitata di TCP e ICMP su un'interfaccia autorizzata.
sudo tcpdump -i eth0 -nn 'tcp or icmp' -w scans/analisi.pcap
```

`eth0` è un segnaposto per il nome dell'interfaccia. La cattura può contenere credenziali, dati applicativi e indirizzi: conservarla soltanto se autorizzato, con accessi e tempi di retention appropriati. Il filtro mostrato è un **capture filter** di tcpdump; i filtri di visualizzazione Wireshark usano una sintassi differente.

| Filtro Wireshark | Cosa mostra |
|---|---|
| `tcp` | Pacchetti decodificati come TCP. |
| `tcp.flags.syn == 1` | Segmenti TCP con SYN impostato, inclusi SYN+ACK. |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Segmenti SYN senza ACK, tipici dell'avvio di connessione. |
| `tcp.flags.reset == 1` | Segmenti con RST. |
| `icmp` | Messaggi ICMPv4. |
| `udp.port == 53` | Traffico UDP con sorgente o destinazione 53. |

Un filtro che mostra solo SYN non documenta necessariamente la risposta: per ricostruire un handshake servono anche SYN+ACK e ACK, con indirizzi e porte corretti. Nel controllo TCP è utile correlare sequenze, flag, timestamp e checksum, ricordando che offloading della scheda di rete può incidere su ciò che si osserva localmente.

## 24. Verifica dei servizi su Linux e Windows

Le scansioni esterne e gli strumenti locali rispondono a domande differenti. Il comando locale può mostrare che un processo ascolta su una porta; non dimostra però che quella porta sia raggiungibile da un'altra rete. Il contrario può verificarsi per NAT, proxy e container: un endpoint appare aperto dall'esterno ma non coincide con un socket sul sistema che si sta osservando.

### Linux

```bash
ip -br address
ip route
ss -lntup       # Socket TCP/UDP in ascolto con processi, se visibili
sudo ss -lntup  # Con privilegi per più dettagli dei processi
```

`ss` interroga lo stato dei socket del kernel. `-l` limita ai socket in ascolto, `-n` evita la risoluzione numerica, `-t` e `-u` indicano TCP e UDP, `-p` aggiunge i processi quando consentito.

### Windows / PowerShell

```powershell
Get-NetIPAddress
Get-NetRoute
Get-NetTCPConnection -State Listen |
    Select-Object LocalAddress,LocalPort,OwningProcess
Get-Process -Id 1234
Test-NetConnection -ComputerName 192.0.2.10 -Port 443
```

`1234` è un PID fittizio da sostituire. Il cmdlet `Test-NetConnection` effettua una verifica di connettività TCP verso la porta indicata e non sostituisce un intero port scan. `Get-NetTCPConnection` mostra endpoint e connessioni TCP conosciuti localmente. Non va usato per attribuire uno stato a porte UDP remote.

Una differenza tra output Nmap esterno e socket locali può derivare da firewall del sistema operativo, binding a `127.0.0.1`, interfacce specifiche, regole di NAT, bilanciamento, proxy o container. Per determinarne la causa servono controlli sul percorso, non una semplice ripetizione dello stesso scan.

## 25. Metodologia operativa di scansione

Questa procedura non descrive un laboratorio effettivamente eseguito: è una **sequenza riutilizzabile** per un ambiente di proprietà o esplicitamente autorizzato. Ogni fase produce una domanda da chiarire e un tipo di evidenza da conservare. La scelta delle opzioni deve rispettare limiti di traffico, finestre orarie e sensibilità degli asset.

### Fase A — Definire perimetro e origine

Registrare sistema scanner, collegamento (LAN, VPN, rete virtuale), interfaccia, indirizzo sorgente, intervalli inclusi ed esclusioni. Non usare un CIDR ricevuto da un vecchio documento senza verificare che sia ancora nello scope. Verificare che le opzioni scelte siano supportate da Nmap e che vi siano privilegi sufficienti per i metodi raw.

```bash
nmap --version
ip -br address
ip route
nmap -sL -n "$NETWORK"
```

L'ultima riga mostra la selezione dei target senza eseguire la normale discovery e senza port scan; `-n` evita lookup DNS. Non prova che gli indirizzi siano attivi. Se un asset risulta escluso, va rimosso **prima** delle scansioni successive.

### Fase B — Scoprire gli host

```bash
sudo nmap -sn -n -oA scans/discovery "$NETWORK"
```

La tecnica effettiva dipende da collegamento, privilegi e opzioni. Nella LAN può intervenire ARP; da una rete instradata possono essere usate sonde ICMP e TCP. Annotare la motivazione della risposta e l'eventuale necessità di una seconda forma di discovery su **singoli host autorizzati** che sembrano non rispondere.

### Fase C — Selezionare e scansionare porte

Non considerare automaticamente come «vivi» soltanto gli indirizzi presenti nell'output: la discovery può mancare host protetti da filtri. La selezione dei target successivi deriva da scope e osservazioni. Per uno specifico host è possibile iniziare con un insieme ristretto di porte:

```bash
nmap -sT -p 22,80,443 --reason -n \
     -oA scans/porte-iniziali "$TARGET"
```

Se autorizzata, una scansione più ampia può esaminare le restanti porte TCP. Per UDP è preferibile una selezione ragionata e separata delle porte, perché tempi e risultati hanno caratteristiche differenti.

```bash
nmap -sT -p- --reason -n -oA scans/porte-tcp "$TARGET"
sudo nmap -sU -p 53,123,161 --reason -n \
     -oA scans/porte-udp "$TARGET"
```

### Fase D — Identificare i servizi effettivi

Eseguire la rilevazione della versione solo dove serve, in base a porte, comportamento e autorizzazioni. Su endpoint HTTP/HTTPS verificati può seguire una richiesta applicativa limitata; su altri protocolli servono client appropriati. Non associare una vulnerabilità al solo banner.

```bash
nmap -sT -sV -p 22,80,443 --reason -n \
     -oA scans/servizi "$TARGET"
```

Gli script NSE vanno scelti per nome e dopo aver consultato NSEDoc. Anche una semplice scansione di un endpoint HTTPS può esporre certificati, nomi e dettagli del servizio: tali dati vanno trattati come materiale di assessment.

### Fase E — Correlare e produrre un report

Per ogni servizio riportare **indirizzo, trasporto, porta, stato, tecnica, indizio applicativo, fonte e livello di confidenza**. Allegare la data e l'origine delle prove. Separare osservazioni dai punti ancora da verificare. Un report non deve contenere falsi positivi presentati come certezze né conclusioni legate a una versione che non è stata confermata.

| Campo del report | Esempio di cosa registrare, senza inventare un valore |
|---|---|
| Asset | Host/IP effettivamente autorizzato. |
| Provenienza | Sistema e interfaccia usati per la scansione. |
| Finestra | Timestamp locale con fuso orario. |
| Comando e versione | Riga di comando completa, `nmap --version`. |
| Esito grezzo | File `.nmap`, `.xml` ed eventuale cattura. |
| Interpretazione | Perché una porta è ritenuta aperta, filtrata o ambigua. |
| Correlazione | Conferme con strumenti applicativi o controllo locale. |
| Limiti | Timeout, filtraggio, differenze tra punti di osservazione. |

## 26. Analisi e interpretazione dei risultati

L'output di Nmap comprende righe di intestazione con dati di esecuzione, informazioni sullo stato dell'host, tabella delle porte e, se richiesto, risultati della rilevazione del servizio, script o fingerprint OS. Ogni blocco risponde a una domanda differente.

La tabella `PORT STATE SERVICE VERSION` va letta separando quattro informazioni:

| Campo | Domanda corretta |
|---|---|
| `PORT` | Quale numero di porta e quale trasporto sono stati esaminati? |
| `STATE` | Qual è la conclusione di Nmap con questa tecnica? |
| `SERVICE` | Nome convenzionale o identificazione del protocollo? |
| `VERSION` | Quale informazione di versione è stata ricavata e con quali limiti? |

### Esempio tratto dalla documentazione ufficiale

Il tutorial **A Quick Port Scanning Tutorial**, pubblicato sul sito ufficiale Nmap, contiene output reali forniti dagli autori della documentazione e spiega le diverse scelte di scanning. È una fonte migliore di un finto terminale con indirizzi e servizi costruiti per sembrare autentici. La fonte pubblica è.

La fonte documenta **due scansioni storiche** su `scanme.nmap.org`. Nella prima, condotta sulle 1.000 porte TCP predefinite, Nmap riportò **994 porte filtrate**, tre aperte (`22/tcp`, `53/tcp`, `80/tcp`) e tre chiuse (`25/tcp`, `70/tcp`, `113/tcp`), con un tempo di circa **5 secondi**. Nel secondo esempio, usando `-p0- -v -A -T4`, furono esaminate 65.536 porte numerate **da 0 a 65535** e aggiunti rilevamento versione/OS, NSE e traceroute; la durata pubblicata fu di circa **477 secondi**.

**Sono risultati storici del tutorial, non una scansione effettuata per questo documento e non lo stato attuale di quel server.** In particolare il comando storico `-p0-` include la porta zero, diversamente da `-p-` impiegato negli esempi operativi precedenti. Il confronto dimostra che uno scan più esteso può produrre maggiori dettagli e richiedere molto più tempo, senza necessariamente scoprire ulteriori porte aperte. La verifica attuale richiede osservazioni nuove e autorizzate.

### Confronto tra scansioni

La divergenza può essere significativa senza indicare un errore del tool. Confrontare:

- indirizzo e risoluzione DNS al momento dei test;
- origine della scansione, VPN, routing e famiglia IPv4/IPv6;
- tecnica (`-sS`, `-sT`, `-sA`, `-sU`), privilegi e porte interrogate;
- timeout, retry, rate limit ed eventuali errori di esecuzione;
- presenza di load balancer, proxy, regole dinamiche o servizi avviati dopo la prima prova.

Se una porta è `open` in una prova e `closed` in un'altra, prima di formulare conclusioni occorre verificare che le prove abbiano interrogato **lo stesso endpoint nelle stesse condizioni**.

## 27. Risoluzione dei problemi nelle scansioni

| Sintomo | Ipotesi possibili | Primo controllo utile |
|---|---|---|
| `Host seems down` | Discovery filtrata, route assente, target spento, problemi di DNS. | IP e routing, sonde consentite, `-Pn` su un singolo target quando giustificato. |
| Tutte le porte `filtered` | ACL, firewall host/rete, timeout, target non raggiungibile. | `--reason`, punto di origine, cattura piccola, verifica del percorso. |
| UDP `open\|filtered` | Protocollo silenzioso o filtro; risposta non ricevuta. | `-sV` limitato, protocollo corretto, eventuali ICMP e rate limit. |
| Nessuna corrispondenza OS | Poche porte utili, filtri, sistema non noto al database. | Individuare se vi sono una porta aperta e una chiusa, senza forzare una certezza. |
| Versione anomala | Banner personalizzato, reverse proxy, porta non standard. | Richiesta applicativa appropriata e correlazione. |
| Scansione molto lenta | Retry, perdita, rate limit, troppe porte o host, script pesanti. | Ridurre perimetro e intensità, confrontare condizioni della rete. |
| Privilegi insufficienti | Metodo raw non disponibile o driver necessario. | Privilegi/ambiente; usare `-sT` quando adeguato. |
| Differenze tra Linux/Windows | Stack, firewall, interfaccia, Npcap, routing o metodo scan. | Verificare tecnica effettiva e punto di osservazione. |
| Nessun pacchetto in Wireshark | Interfaccia sbagliata, capture filter, namespace o offloading. | Route, interfaccia, filtro e posizione di cattura. |

### Interpretazioni errate ricorrenti

**«Il ping non risponde, quindi il PC è spento».** Falso: il filtro può bloccare soltanto ICMP. **«La porta 80 è aperta, quindi gira Apache».** Falso: la porta non identifica il prodotto. **«`-sA` mi ha trovato una porta aperta».** Falso: l'ACK scan cerca informazioni sul filtraggio. **«UDP non risponde, quindi la porta è chiusa».** Falso: il silenzio può produrre `open|filtered`. **«`-A` è la modalità più completa e quindi sempre migliore».** Falso: aggiunge rilevazioni e script che vanno scelti in base al test.

Un'analisi ripetibile preserva anche **gli esiti negativi e ambigui**. Eliminare dal report gli scan non conclusivi porta a una rappresentazione ingannevolmente certa dell'ambiente.

## 28. Riferimento dei comandi Nmap

Questa scheda riunisce opzioni trattate nei capitoli precedenti. Non costituisce un elenco di comandi da eseguire automaticamente su qualsiasi rete.

| Scopo | Opzioni essenziali |
|---|---|
| Elencare i target | `-sL -n`, `-iL`, `--exclude`, `--excludefile` |
| Discovery senza port scan ordinario | `-sn` |
| Saltare la fase discovery | `-Pn` |
| Sonda ICMP Echo | `-PE` |
| Sonde di discovery TCP SYN/ACK | `-PS`, `-PA` |
| Sonde di discovery UDP | `-PU` |
| ARP locale IPv4 | `-PR` |
| Scansione TCP SYN | `-sS` |
| Scansione TCP Connect | `-sT` |
| Scansione TCP ACK | `-sA` |
| Scansione UDP | `-sU` |
| Porte specifiche | `-p 22,80,443` |
| Tutte le porte TCP 1-65535 | `-p-` con metodo TCP |
| Porte più diffuse | `--top-ports N` |
| Service/version detection | `-sV`, `--version-light` |
| OS fingerprinting | `-O` |
| Script NSE specifici | `--script nome` |
| Motivo dello stato | `--reason` |
| Limitare la frequenza | `--max-rate N` |
| Verbosità e trace | `-v`, `-d`, `--packet-trace` |
| Salvataggio dei risultati | `-oN`, `-oX`, `-oA` |
| IPv6 | `-6` con target IPv6 adeguato |

### Porte comunemente associate ai servizi

| Endpoint convenzionale | Protocollo o servizio frequentemente associato | Conferma successiva |
|---|---|---|
| 22/tcp | SSH | Banner, negoziazione SSH, fingerprint chiave server. |
| 53/udp e 53/tcp | DNS | Query valida, ricorsione o autorità secondo scope. |
| 80/tcp | HTTP | Risposta HTTP, header e applicazione. |
| 123/udp | NTP | Risposta NTP e indicazioni del servizio. |
| 139/tcp, 445/tcp | NetBIOS/SMB | Negoziazione e policy SMB. |
| 161/udp | SNMP | Risposte solo se permesse e con credenziali autorizzate. |
| 443/tcp | HTTPS o altro servizio TLS | Negoziazione TLS e contenuto applicativo. |
| 3389/tcp | RDP | Negoziazione del servizio, policy di accesso. |
| 8080/tcp | Spesso HTTP alternativo | Verifica del protocollo reale. |

L'elenco non è esaustivo e le associazioni sono configurabili: per i numeri registrati il riferimento è il registro IANA, non la sola presenza in un file di servizi locale.

## 29. Riferimenti tecnici

Le fonti che seguono permettono di verificare definizioni, comportamenti e opzioni. La documentazione Nmap è il riferimento principale per le opzioni; gli standard IETF/RFC chiariscono il funzionamento dei protocolli; i manuali dei sistemi operativi e di Wireshark coprono i controlli locali. Il repository GitHub ufficiale di Nmap è una fonte di implementazione, **non** un motivo per assumere che una qualsiasi versione già installata abbia gli stessi comportamenti. Prima di applicare un comando consultare sempre l'help del programma installato.

**Nmap: manuale e documentazione ufficiale**

- [Nmap — Reference Guide](https://nmap.org/book/man.html). Quadro generale e sintassi delle opzioni.
- [Host Discovery](https://nmap.org/book/man-host-discovery.html). `-sn`, `-Pn`, ARP/ND e sonde di discovery.
- [Host Discovery Techniques](https://nmap.org/book/host-discovery-techniques.html). Meccanismi e risposte delle sonde.
- [Target Specification](https://nmap.org/book/man-target-specification.html). Target, file, esclusioni e list scan.
- [Port Scanning Basics](https://nmap.org/book/man-port-scanning-basics.html). I sei stati Nmap.
- [Port Scanning Techniques](https://nmap.org/book/man-port-scanning-techniques.html). SYN, Connect, ACK e metodi speciali.
- [UDP Scan](https://nmap.org/book/scan-methods-udp-scan.html). Risposte e ambiguità del protocollo UDP.
- [Service and Version Detection](https://nmap.org/book/man-version-detection.html). Sonde applicative e livelli d'intensità.
- [OS Detection](https://nmap.org/book/man-os-detection.html). Fingerprinting dello stack TCP/IP.
- [Nmap Scripting Engine](https://nmap.org/book/man-nse.html). Script, argomenti, selezione e avvertenze.
- [NSEDoc — Reference Portal](https://nmap.org/nsedoc/). Documentazione degli script disponibili.
- [Firewall/IDS Evasion and Spoofing](https://nmap.org/book/man-bypass-firewalls-ids.html). Meccanismi, limiti e comportamenti.
- [Timing and Performance](https://nmap.org/book/man-performance.html). Template, rate, retry e timeout.
- [Output](https://nmap.org/book/man-output.html). Verbosità, motivazioni e formati.
- [XML Output](https://nmap.org/book/output-formats-xml-output.html). Struttura e importazione dei risultati.

**Registri, standard e protocolli**

- [IANA — Service Name and Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml). Nomi dei servizi e port numbers.
- [IANA — IPv4 Special-Purpose Address Registry](https://www.iana.org/assignments/iana-ipv4-special-registry/iana-ipv4-special-registry.xhtml). Prefissi con usi speciali e documentali.
- [RFC 791 — Internet Protocol](https://www.rfc-editor.org/rfc/rfc791). Struttura e semantica di IPv4; leggere anche gli aggiornamenti collegati.
- [RFC 9293 — Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293). Specifica consolidata di TCP.
- [RFC 826 — Address Resolution Protocol](https://www.rfc-editor.org/rfc/rfc826) e [RFC 4861 — Neighbor Discovery for IPv6](https://www.rfc-editor.org/rfc/rfc4861). Risoluzione e discovery sul collegamento.
- [RFC 768 — User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768). Specifica del trasporto UDP.

**Verifiche su host, catture e progetti originali**

- [Wireshark — User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/). Analisi delle catture e filtri di visualizzazione.
- [Linux `ss(8)`](https://man7.org/linux/man-pages/man8/ss.8.html) e [`ip-route(8)`](https://man7.org/linux/man-pages/man8/ip-route.8.html). Socket e instradamento.
- [Microsoft — Get-NetTCPConnection](https://learn.microsoft.com/en-us/powershell/module/nettcpip/get-nettcpconnection), [Get-NetIPAddress](https://learn.microsoft.com/en-us/powershell/module/nettcpip/get-netipaddress), [Test-NetConnection](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection). Comandi PowerShell.
- [Nmap — A Quick Port Scanning Tutorial](https://nmap.org/book/port-scanning-tutorial.html). Esempi pubblicati con output e spiegazioni originali.
- [GitHub — progetto ufficiale `nmap/nmap`](https://github.com/nmap/nmap). Codice e manuale del progetto Nmap; mirror dell'archivio principale.
- [Zenmap — GUI ufficiale](https://nmap.org/book/zenmap.html). Visualizzazione e gestione dei risultati Nmap.
- [Nmap — Host Discovery and Traceroute](https://nmap.org/book/man-host-discovery.html). Tracciamento del percorso, `--traceroute` e limite dell'osservazione.
- [arp-scan — repository originale](https://github.com/royhills/arp-scan). Scanner ARP per host IPv4 sulla rete locale.
- [Netdiscover — repository ufficiale](https://github.com/netdiscover-scanner/netdiscover). Rilevazione di host su collegamenti Ethernet mediante ARP.
- [RFC 792 — Internet Control Message Protocol](https://www.rfc-editor.org/rfc/rfc792). Tipi, codici e finalità di ICMPv4.
- [RFC 8200 — IPv6 Specification](https://www.rfc-editor.org/rfc/rfc8200). Header IPv6, Hop Limit e frammentazione.
- [Nmap — Installazione e utilizzo su Windows](https://nmap.org/book/inst-windows.html). Supporto per Npcap, limiti delle interfacce e scansioni TCP Connect.

