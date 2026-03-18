## Perché imparare SRE oggi

I sistemi digitali moderni sono sempre più complessi.

* microservizi
* cloud
* container
* sistemi distribuiti
* milioni di utenti

Costruire software è importante.
Ma **farlo funzionare in modo affidabile** è ancora più difficile.

È qui che entra in gioco lo **Site Reliability Engineer (SRE)**.

---

## Cosa fa davvero un SRE

Un SRE ha una missione molto semplice:

**mantenere i sistemi funzionanti, scalabili e osservabili.**

Questo significa saper:

* capire cosa sta succedendo nei sistemi
* misurare prestazioni e affidabilità
* individuare problemi prima che diventino incidenti
* reagire rapidamente quando qualcosa si rompe
* migliorare continuamente l’infrastruttura

In altre parole:

> un SRE trasforma il caos dei sistemi distribuiti in qualcosa di comprensibile e controllabile.

---

## Perché è una figura sempre più richiesta

Le aziende oggi dipendono completamente dai sistemi digitali.

Quando un servizio si ferma:

* clienti non riescono ad accedere
* pagamenti falliscono
* dati non sono disponibili
* l’azienda perde denaro

Per questo motivo le aziende hanno sempre più bisogno di professionisti che sappiano:

* monitorare sistemi complessi
* analizzare metriche e log
* prevenire incidenti
* progettare sistemi resilienti

Queste sono esattamente le competenze di un **SRE**.

---

## Cosa imparerete in questo corso

In questo percorso non studierete solo teoria.

Imparerete a:

* costruire un sistema osservabile
* raccogliere metriche con Prometheus
* analizzare dati con Grafana
* generare traffico e simulare problemi
* capire come reagire a un incidente

Questo è il tipo di lavoro che fanno gli SRE ogni giorno.

---

## Il vantaggio per chi inizia

Diventare **SRE junior** significa sviluppare competenze molto richieste:

* Linux
* networking
* cloud
* monitoring
* sistemi distribuiti
* automazione

Queste competenze sono utili in moltissimi ruoli IT.

Anche se un giorno non diventerete SRE, capire **come funzionano davvero i sistemi in produzione** vi renderà ingegneri molto più completi.

---

## Il punto chiave

Molte persone imparano a scrivere codice.

Molte meno imparano a **gestire sistemi complessi nel mondo reale**.

Chi sviluppa questa capacità diventa una figura molto preziosa.

Ed è esattamente ciò che fa uno **Site Reliability Engineer**.

---

Piccola osservazione finale, giusto per restare sinceri: i sistemi informatici prima o poi si rompono sempre. La differenza è tra chi entra nel panico… e chi guarda le metriche, capisce cosa sta succedendo e sistema la situazione. Gli SRE appartengono alla seconda categoria. E le aziende ne hanno sempre più bisogno.











# Concetti LAB01



## 1. Il filesystem Linux

Concetti fondamentali:

* struttura **ad albero**
* root `/`
* directory e file

Esempio:

```
/
 ├── home
 │    └── user
 │         └── corso_obs
 ├── var
 ├── etc
```

Idea chiave:

> tutto in Linux è un file o una directory.

---

## 2. Path assoluti e relativi

Path assoluto

```
/home/user/corso_obs
```

Path relativo

```
cd lab01
```

Simboli fondamentali:

```
.   directory corrente
..  directory padre
~   home utente
```

---

## 3. WSL filesystem vs Windows filesystem


Filesystem Linux:

```
/home/user/
```

Filesystem Windows montato:

```
/mnt/c
```

Regola del corso:

```
tutti i progetti stanno in

~/corso_obs
```

---

## 4. Comandi base del filesystem

Comandi da sapeer spiegare:

```
pwd
ls
cd
mkdir
touch
cp
mv
rm
cat
less
```

Concetto importante:

```
mkdir -p
```

crea directory multiple.

---

## 5. Il comando find

Serve per:

* cercare file
* filtrare per nome
* dimensione
* data

Esempi:

```
find . -name "*.txt"
find . -type f
find . -size +1k
find . -mtime -1
```

---

## 6. Workspace vs repository

Nel tuo corso:

repo:

```
labs/lab01
```

workspace locale:

```
logs/work
```

Concetto DevOps fondamentale:

workspace = area di lavoro temporanea
repo = codice versionato

---

# Concetti  LAB02

(File, testo, redirection e pipe)

Qui cambia completamente il livello. Non è più filesystem, è **text processing**, che è una delle armi principali di Linux.

Questo lab è importante perché introduce il concetto di **pipeline UNIX**.

---

# 1. Filosofia Unix

 **regola fondamentale**:

> In Unix ogni programma fa **una cosa sola e la fa bene**.

Esempio:

* `grep` filtra
* `sort` ordina
* `uniq` deduplica
* `wc` conta

Poi li combini.

---

# 2. Standard Input / Output

Concetto cruciale.

Ogni comando ha:

| stream | numero |
| ------ | ------ |
| stdin  | 0      |
| stdout | 1      |
| stderr | 2      |

Diagramma:

```
input → comando → output
```

---

# 3. Redirection

Spiegare bene:

### overwrite

```
>
```

esempio

```
ls > file.txt
```

sovrascrive.

---

### append

```
>>
```

```
echo "ciao" >> file.txt
```

aggiunge.

---

### input

```
<
```

esempio

```
wc -l < file.txt
```

---

# 4. Pipe

Il concetto più importante del lab.

```
|
```

Collega output di un comando all’input del successivo.

Esempio:

```
cat log.txt | grep ERROR
```

pipeline più lunga:

```
cat log.txt | grep ERROR | sort | uniq
```



> output di un comando diventa input del successivo.

---

# 5. Comandi di analisi testo

 (i principali)

## grep

filtra testo

```
grep ERROR log.txt
```

regex esempio:

```
grep -E '5[0-9]{2}'
```

---

## wc

conta:

```
wc -l file
```

linee

```
wc -w
```

parole

---

## sort

ordina

```
sort file
```

numerico

```
sort -n
```

reverse

```
sort -nr
```

---

## uniq

rimuove duplicati.

Ma attenzione:

> funziona solo su input ordinato.

Esempio:

```
sort file | uniq
```

---

## tee

molto importante.

Serve per:

* salvare output
* continuare pipeline

```
comando | tee file.txt
```

---

# 6. Perché analizzare log

Collegato al corso Observability.

Un log tipico:

```
timestamp method endpoint status
```

Esempio:

```
2026-02-09T14:01:02Z GET / 200
```

Con pipeline puoi fare:

* error rate
* endpoint più usati
* errori server

Questo è **monitoring manuale da terminale**.

---

# Concetti preliminari LAB03

(Permessi, utenti, processi)

Qui entri nella parte **SRE vera**.

---

# 1. Permessi Unix

Ogni file ha:

```
r  read
w  write
x  execute
```

Per tre categorie:

```
user
group
others
```

Esempio:

```
-rwxr-xr-x
```

significa:

```
user  rwx
group r-x
other r-x
```

---

# 2. Permessi ottali

Sapere:

```
r = 4
w = 2
x = 1
```

Esempi:

```
600
755
644
```

tabella utile:

| permesso | significato  |
| -------- | ------------ |
| 755      | eseguibile   |
| 644      | file normale |
| 600      | privato      |

---

# 3. Ownership

Ogni file ha:

```
owner
group
```

comandi:

```
whoami
id
groups
```

esempio output:

```
uid=1000(user) gid=1000(user)
```

---

# 4. Directory vs file permissions

Errore comune.

Per directory:

| permesso | significato      |
| -------- | ---------------- |
| r        | vedere contenuto |
| x        | entrare          |
| w        | modificare       |

Se manca `x`:

```
Permission denied
```

anche se il file è leggibile.

---

# 5. Processi Linux

Concetto:

un programma in esecuzione = processo.

Ogni processo ha:

```
PID
```

comandi:

```
ps
top
pgrep
```

---

# 6. Segnali

Quando gestisci processi.

Due principali:

SIGTERM

```
kill -15 PID
```

terminazione gentile.

SIGKILL

```
kill -9 PID
```

terminazione forzata.

---

# 7. Porte e servizi

Nel lab usi:

```
python3 -m http.server 8080
```

concetto:

una applicazione **ascolta su una porta**.

verifica:

```
ss -ltnp
```

Se una porta è già occupata:

```
address already in use
```

---

# 8. Bash scripting base

Prima dello script devi conoscere:

shebang

```
#!/bin/bash
```

parametri

```
$1
```

exit code

```
exit 0
exit 1
exit 2
```

controlli:

```
if
```

regex test:

```
[[ ]]
```

---

---



# il comando awk

cosa fa? 
awk '{print $1}' raw_access.log | sort | uniq -c | sort -nr

Il comando che hai indicato serve per estrarre gli indirizzi IP (o il primo campo) da un file di log e visualizzarli in ordine di frequenza, dal più comune al meno comune.

Ecco cosa fa ogni singolo pezzo della pipeline:

awk '{print $1}' raw_access.log: Legge il file raw_access.log ed estrae solo la prima colonna di ogni riga. In un file di log standard (come quelli di Apache o Nginx), questa colonna contiene solitamente l'indirizzo IP del visitatore.

| sort: Ordina alfabeticamente (o numericamente) gli indirizzi estratti. Questo passaggio è necessario perché il comando successivo (uniq) funziona correttamente solo se le righe identiche sono adiacenti.

| uniq -c: Rimuove i duplicati e conta le occorrenze. L'opzione -c aggiunge un numero davanti a ogni riga che indica quante volte quell'IP è apparso nel file.

| sort -nr: Ordina nuovamente il risultato finale in base al conteggio numerico (-n) e in ordine inverso/decrescente (-r). 

Risultato finale
Otterrai una lista dove gli IP che hanno effettuato più richieste appaiono per primi, accompagnati dal numero totale di accessi. È uno strumento molto utile per: 
Identificare i visitatori più attivi sul tuo sito.
Individuare potenziali attacchi (es. tentativi di brute force o scraping) da un singolo indirizzo IP. 



# Avviare un Web server "al volo"
python -u -m http.server 8000 
python -u -m http.server 8000  &

# Stati di un processo Linux
in quali stati può trovarsi un processo linux e cosa significano?

In Linux, ogni processo attraversa diverse fasi durante il suo ciclo di vita. Questi stati indicano cosa sta facendo il processo e come sta interagendo con la CPU e le risorse di sistema. 

I principali stati in cui può trovarsi un processo sono i seguenti:

Running o Runnable (R): Il processo è attualmente in esecuzione sulla CPU oppure è pronto per essere eseguito e sta aspettando il suo turno nello scheduler.

Interruptible Sleep (S): Il processo è "addormentato" in attesa di un evento esterno, come un input dall'utente o la disponibilità di una risorsa di rete. Può essere risvegliato da un segnale.

Uninterruptible Sleep (D): Simile allo stato precedente, ma il processo è in attesa di un'operazione di I/O (solitamente sul disco) che non può essere interrotta. In questo stato, il processo non risponde nemmeno ai segnali di terminazione (come kill -9) finché l'operazione non si conclude.

Stopped (T): Il processo è stato sospeso, solitamente da un utente o da un debugger (ad esempio premendo Ctrl+Z nel terminale). Rimane fermo finché non riceve un segnale per riprendere l'esecuzione.

Zombie (Z): Il processo ha terminato la sua esecuzione ma non è ancora stato rimosso completamente dalla tabella dei processi. Esiste ancora solo perché il processo padre deve ancora leggere il suo codice di uscita (operazione di "reaping").

Dead (X): È uno stato transitorio molto raro da vedere, in cui il processo viene definitivamente eliminato dal sistema. 

-------------------------------------------------------------------------------------------------------------------------------


13-03-26 
# Introduzione alle reti e al protocollo TCP/IP

Le applicazioni che analizzerete nel corso (API, server web, Prometheus, ecc.) funzionano tutte grazie alla **rete TCP/IP**.

Schema generale:

```
Client (browser)
      │
      │ HTTP request
      ▼
Router / Gateway
      │
      │ Internet
      ▼
Server Web
      │
      │ Application
      ▼
Database / servizi
```

Ogni comunicazione avviene tramite:

* indirizzi IP
* porte
* protocolli (TCP/UDP)
* applicazioni (HTTP, DNS ecc.)

---

# 1. Rete locale (LAN) e rete geografica (WAN)

## LAN – Local Area Network

È la rete interna di:

* casa
* ufficio
* laboratorio

Esempio tipico:

```
                Router
            192.168.1.1
                │
 ┌──────────────┼──────────────┐
 │              │              │
PC1          PC2           Server
192.168.1.10 192.168.1.11 192.168.1.20
```

Caratteristiche:

* indirizzi **privati**
* velocità elevata
* gestione locale

---

## WAN – Wide Area Network

Collega reti locali tra loro.

Esempio:

```
LAN azienda A ───── Internet ───── LAN azienda B
```

Internet è la WAN più grande.

---

# 2. Indirizzi IP

Un **indirizzo IP** identifica un dispositivo sulla rete.

IPv4 esempio:

```
192.168.1.25
```

È composto da **4 ottetti** (8 bit ciascuno).

```
192.168.1.25
```

Range di ogni numero:

```
0 – 255
```

---

## IP privati

Usati nelle reti locali.

| Range                         | Uso             |
| ----------------------------- | --------------- |
| 10.0.0.0 – 10.255.255.255     | reti aziendali  |
| 172.16.0.0 – 172.31.255.255   | reti private    |
| 192.168.0.0 – 192.168.255.255 | reti domestiche |

---

# 3. Subnet

Una subnet divide una rete in porzioni.

Esempio:

```
IP: 192.168.1.25
Subnet mask: 255.255.255.0
```

Significa:

```
rete: 192.168.1.0
host: 1 – 254
```

Schema:

```
192.168.1.0/24

rete       host
192.168.1.1
192.168.1.2
...
192.168.1.254
```

---

## CIDR notation

Forma compatta:

```
192.168.1.0/24
```

Significa:

```
24 bit rete
8 bit host
```

---

# 4. Gateway

Il **gateway** è il dispositivo che permette alla rete locale di uscire verso altre reti.

Di solito è il router.

Esempio:

```
PC
IP 192.168.1.10
Gateway 192.168.1.1
```

Schema:

```
PC → Router → Internet
```

Se il gateway è sbagliato:

* Internet non funziona
* LAN funziona

---

# 5. DHCP

Il **Dynamic Host Configuration Protocol** assegna automaticamente:

* IP
* subnet mask
* gateway
* DNS

Schema:

```
Client → DHCP DISCOVER
Server → DHCP OFFER
Client → DHCP REQUEST
Server → DHCP ACK
```

Esempio configurazione ricevuta:

```
IP       192.168.1.15
Gateway  192.168.1.1
DNS      8.8.8.8
```

---

# 6. DNS

Il **Domain Name System** traduce nomi in indirizzi IP.

Esempio:

```
google.com → 142.250.180.14
```

Processo:

```
Browser
   │
   ▼
DNS query
   │
   ▼
DNS server
   │
   ▼
IP address
```

---

## Esempio da terminale

```bash
nslookup google.com
```

oppure

```bash
dig google.com
```

Output:

```
google.com.  300  IN  A  142.250.180.14
```

---

# 7. VLAN

Una VLAN separa logicamente una rete fisica.

Schema:

```
Switch
│
├── VLAN10 → ufficio
├── VLAN20 → amministrazione
└── VLAN30 → server
```

Benefici:

* isolamento
* sicurezza
* gestione traffico

---

# 8. VPN

Una VPN crea un **tunnel cifrato** tra reti.

Schema:

```
Laptop
   │
   │ VPN tunnel
   ▼
Gateway aziendale
   │
   ▼
Rete aziendale
```

Utilizzi:

* accesso remoto
* collegare sedi
* sicurezza

Tecnologie comuni:

* OpenVPN
* WireGuard
* IPSec

---

# 9. TCP e UDP

I protocolli principali di trasporto.

---

## TCP

Transmission Control Protocol.

Caratteristiche:

* affidabile
* orientato alla connessione
* ritrasmissione pacchetti

Schema handshake:

```
Client → SYN
Server → SYN ACK
Client → ACK
```

Connessione stabilita.

---

## UDP

User Datagram Protocol.

Caratteristiche:

* veloce
* senza connessione
* nessuna garanzia consegna

Usato per:

* DNS
* streaming
* VoIP
* gaming

---

# 10. Porte

Una porta identifica un servizio su un host.

Schema:

```
IP + PORTA = servizio
```

Esempio:

```
192.168.1.20:80
```

---

## Porte comuni

| Porta | Servizio |
| ----- | -------- |
| 22    | SSH      |
| 53    | DNS      |
| 80    | HTTP     |
| 443   | HTTPS    |
| 3306  | MySQL    |

---

## Verifica porte su Linux

```bash
ss -ltnp
```

Output esempio:

```
LISTEN 0 128 0.0.0.0:8080
```

Significa:

```
porta 8080 aperta
```

---

# 11. Sessioni di rete

Una sessione identifica una connessione tra due host.

Definita da:

```
IP sorgente
porta sorgente
IP destinazione
porta destinazione
protocollo
```

Schema:

```
Client
192.168.1.10:51500
        │
        │ TCP
        ▼
Server
10.1.1.20:443
```

---

# 12. HTTP

HTTP è il protocollo applicativo usato dal web.

Funziona sopra TCP.

Schema:

```
Browser
   │
   │ HTTP request
   ▼
Web server
   │
   │ HTTP response
   ▼
Browser
```

---

## Esempio richiesta HTTP

```
GET /api/users HTTP/1.1
Host: example.com
```

---

## Risposta

```
HTTP/1.1 200 OK
Content-Type: application/json
```

---

# 13. HTTP methods

| Metodo | Funzione     |
| ------ | ------------ |
| GET    | leggere dati |
| POST   | creare       |
| PUT    | aggiornare   |
| DELETE | eliminare    |

---

# 14. Status code HTTP

| Codice | Significato         |
| ------ | ------------------- |
| 200    | OK                  |
| 301    | redirect            |
| 404    | not found           |
| 500    | server error        |
| 503    | service unavailable |

---

# 15. Collegamento con i laboratori

Nel laboratorio analizzerete log HTTP.

Esempio riga log:

```
192.168.1.10 - - [11/Mar/2026:10:15:23 +0100] "GET /api/users HTTP/1.1" 200 1543
```

Informazioni presenti:

| campo      | significato         |
| ---------- | ------------------- |
| IP         | client              |
| timestamp  | richiesta           |
| GET        | metodo HTTP         |
| /api/users | endpoint            |
| 200        | status              |
| 1543       | dimensione risposta |

---

# 16. Collegamento con Observability

Questi dati servono per:

* monitorare traffico
* individuare errori
* analizzare performance

Esempi:

contare errori server

```bash
grep " 500 " access.log
```

endpoint più usati

```bash
awk '{print $7}' access.log | sort | uniq -c | sort -nr
```

---

------------------------------------------------------
Concetti / Ripetizione per LAB04
------------------------------------------------------


## Rete, servizio HTTP locale e primi concetti di observability


# 1. Obiettivo della lezione

Prima di affrontare i laboratori 04, 05 e 06, bisogna capire tre idee fondamentali:

1. **come un host comunica in rete**
2. **come funziona un servizio HTTP**
3. **come si passa dai log alle metriche operative**

Questi tre lab non sono separati.
In realtà formano una piccola catena logica:

```text
Rete
→ servizio HTTP
→ log
→ indicatori operativi
```

Questa catena va compresa chiaramente, altrimenti i laboratori sembrano esercizi casuali.

---

# 2. Prima parte: la rete

## 2.1 Che cos’è una rete

Una rete è un insieme di dispositivi collegati tra loro che possono scambiarsi dati.

Esempi di dispositivi:

* PC
* server
* router
* switch
* stampanti
* macchine virtuali
* container

Quando un computer vuole parlare con un altro, deve sapere:

* **chi** è il destinatario
* **come** raggiungerlo
* **quale servizio** vuole usare

Schema semplice:

```text
Client
   │
   │ richiesta di rete
   ▼
Gateway / Router
   │
   ▼
Server
```

---

## 2.2 Che cos’è un indirizzo IP

Un indirizzo IP identifica un host in rete.

Esempio:

```bash
192.168.1.20
```

Questo numero dice: “questa macchina è raggiungibile con questo indirizzo”.

### Importante

L’indirizzo IP identifica **la macchina**, non ancora il servizio.
Sulla stessa macchina possono esistere più servizi:

* SSH
* HTTP
* database
* applicazioni custom

Per distinguerli servono le **porte**.

---

## 2.3 Rete locale e rete esterna

Se una macchina appartiene a una rete locale, per esempio:

```bash
192.168.1.20/24
```

significa che è dentro una rete privata.

Una rete del genere contiene spesso:

* client
* server locali
* router

Esempio:

```text
PC client         192.168.1.10
Server locale     192.168.1.20
Gateway/router    192.168.1.1
```

Tutte queste macchine stanno nella stessa rete.

---

## 2.4 Che cos’è la subnet

La subnet definisce quale parte dell’indirizzo IP identifica la rete e quale parte identifica l’host.

Esempio:

```bash
192.168.1.20/24
```

`/24` significa, in pratica, che la rete è:

```bash
192.168.1.0/24
```

Quindi macchine come:

```bash
192.168.1.10
192.168.1.20
192.168.1.50
```

sono nella stessa rete.

### Ricordare

La subnet serve a capire se due host possono parlarsi direttamente oppure devono passare da un router.

---

## 2.5 Che cos’è il gateway

Il gateway è la macchina a cui un host invia il traffico destinato a reti esterne.

Di solito è il router.

Esempio:

```bash
IP host:    192.168.1.20
Gateway:    192.168.1.1
```

Se l’host vuole raggiungere Internet o una rete diversa, manda il traffico al gateway.

### Importante

Se il gateway è sbagliato:

* la macchina può magari funzionare in LAN
* ma non raggiunge fuori

---

## 2.6 Che cos’è il DNS

Il DNS, Domain Name System, traduce nomi in indirizzi IP.

Esempio:

```text
example.com → 93.184.216.34
```

Noi umani ricordiamo i nomi.
La rete usa IP.
Il DNS fa da traduttore.

Se scriviamo:

```bash
curl https://example.com
```

prima di tutto deve avvenire la risoluzione DNS.

### Errore tipico

Se il DNS non funziona, il problema non è HTTP.
Il client non sa proprio dove andare.

---

## 2.7 Che cos’è una porta

Una porta identifica un servizio su una macchina.

Esempio:

* `192.168.1.20:22` → SSH
* `192.168.1.20:80` → HTTP
* `192.168.1.20:443` → HTTPS
* `192.168.1.20:9000` → servizio del nostro lab

### Formula semplice da dare in aula

```text
IP = macchina
Porta = servizio su quella macchina
```

Questa distinzione è fondamentale.

---

## 2.8 Che cos’è HTTP

HTTP è un protocollo applicativo usato per la comunicazione web.

Quando un client invia una richiesta HTTP, manda:

* un metodo
* un percorso
* eventuali header
* a volte un body

Esempio:

```text
GET /health HTTP/1.1
Host: localhost:9000
```

Il server risponde con:

* status code
* header
* body

Esempio:

```text
HTTP/1.1 200 OK
Content-Type: application/json
```

---

# 3. Diagnostica di rete

## 3.1 Perché serve distinguere i problemi

Quando “non funziona”, il problema può trovarsi in livelli diversi.

Bisogna imparare a distinguerli.

### Tipologie principali

1. problema DNS
2. problema di connettività IP
3. problema di porta / servizio
4. problema applicativo HTTP

---

## 3.2 Problema DNS

### Sintomo

Il nome non si risolve.

Esempio:

```bash
curl https://hostinesistente.local
```

Errore tipico:

```text
Could not resolve host
```

### Significato

Il client non riesce a tradurre il nome in un indirizzo IP.

---

## 3.3 Problema di connettività IP

### Sintomo

Il nome magari si risolve, ma il traffico non arriva.

Esempio:

```bash
ping -c 2 1.1.1.1
```

Se non risponde, c’è un problema di raggiungibilità o filtraggio.

### Attenzione

`ping` non prova HTTP.
Prova la raggiungibilità IP tramite ICMP.

---

## 3.4 Problema di porta

### Sintomo

L’host esiste, ma il servizio su quella porta non risponde.

Esempio:

```bash
nc -vz example.com 81
```

Se la porta è chiusa o non c’è listener, fallisce la connessione.

### Messaggio da fissare

Macchina raggiungibile non significa servizio attivo.

---

## 3.5 Problema applicativo HTTP

### Sintomo

La rete funziona, la porta è aperta, ma il server risponde con errore.

Esempio:

```bash
curl -I http://localhost:8081/non-esiste
```

Risposta:

```text
HTTP/1.0 404 File not found
```

### Interpretazione

Rete ok.
Porta ok.
Servizio ok.
Errore logico applicativo.

---

## 3.6 Metodo di troubleshooting

Questo è un concetto fondamentale di questa lezione.

Schema:

```text
Sintomo
→ ipotesi
→ test
→ conclusione
→ fix
```

### Esempio

**Sintomo**: `curl https://example.com` non funziona
**Ipotesi**: DNS rotto
**Test**:

```bash
dig +short example.com
```

**Conclusione**: nessuna risoluzione
**Fix**: usare DNS funzionante o bypass temporaneo

Questa forma deve diventare una disciplina mentale.

---

# 4. Comandi base del LAB04

## 4.1 `ip a`

### Definizione

Mostra interfacce di rete e indirizzi IP.

### Comando

```bash
ip a
```

### Cosa osservare

* nome interfaccia
* stato UP o DOWN
* indirizzo IP
* subnet

### Esempio semplificato

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.20/24
```

### Significato

L’interfaccia `eth0` è attiva e ha IP `192.168.1.20`.

---

## 4.2 `ip r`

### Definizione

Mostra la tabella di routing.

### Comando

```bash
ip r
```

### Esempio

```text
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.20
```

### Spiegazione

* `default via 192.168.1.1` indica il gateway
* la rete locale `192.168.1.0/24` è raggiungibile tramite `eth0`

---

## 4.3 `ping`

### Definizione

Verifica la raggiungibilità IP tramite ICMP.

### Comando

```bash
ping -c 2 1.1.1.1
```

### Opzione

* `-c 2` invia 2 pacchetti e si ferma

### Cosa dimostra

Se funziona, l’host è raggiungibile a livello IP.

---

## 4.4 `dig`

### Definizione

Interroga il DNS.

### Esempio

```bash
dig +short google.com
```

### Opzione

* `+short` rende l’output minimale

### Cosa dimostra

Se produce un IP, il DNS ha risolto il nome.

---

## 4.5 `curl -I`

### Definizione

Invia una richiesta HTTP e mostra solo gli header della risposta.

### Esempio

```bash
curl -I https://example.com
```

### Cosa dimostra

* il server risponde
* status code
* header essenziali

---

## 4.6 `curl -v`

### Definizione

Mostra i dettagli della comunicazione HTTP/TLS.

### Esempio

```bash
curl -Iv https://example.com
```

### Opzioni

* `-I` solo header
* `-v` verbose

### Cosa permette di vedere

* DNS
* apertura connessione
* handshake TLS
* richiesta e risposta

---

## 4.7 `ss -ltnp`

### Definizione

Mostra le socket TCP in ascolto e i processi associati.

### Esempio

```bash
ss -ltnp
```

### Opzioni

* `-l` listening
* `-t` TCP
* `-n` numerico
* `-p` processo

### Esempio output

```text
LISTEN 0 5 0.0.0.0:8081 0.0.0.0:* users:(("python3",pid=1234,fd=3))
```

### Significato

C’è un processo python3 che ascolta sulla porta 8081.

---

# 5. Ingresso al LAB05

## 5.1 Dal problema di rete al servizio applicativo

Dopo aver svolto il LAB04 avremo imparato a capire **se**:

* la rete c’è
* il DNS funziona
* una porta è aperta
* HTTP risponde

Nel LAB05 facciamo il passo successivo:
**creiamo noi un piccolo servizio HTTP e impariamo a osservarlo**.

---

## 5.2 Differenza tra file, programma, processo, servizio

### File

È un oggetto sul disco.

Esempio:

```bash
src/app.py
```

### Programma

È codice eseguibile, o interpretabile, che descrive un comportamento.

### Processo

È un programma in esecuzione.

Quando fai:

```bash
python3 src/app.py
```

hai creato un processo.

### Servizio

È un processo che resta attivo e offre una funzione.

Nel nostro caso il servizio:

* ascolta su una porta
* riceve richieste HTTP
* risponde in JSON
* scrive log

---

## 5.3 Che cos’è un servizio HTTP locale

Significa che il servizio gira sulla macchina del partecipante, non su un server remoto.

URL tipico:

```bash
http://localhost:9000
```

### `localhost`

È il nome che indica la macchina locale.
Di solito corrisponde a `127.0.0.1`.

Quindi:

```bash
http://localhost:9000
```

e

```bash
http://127.0.0.1:9000
```

di solito puntano allo stesso host.

---

## 5.4 Che cos’è un endpoint

Un endpoint è un percorso HTTP esposto dal servizio.
Possiamo vederlo come una funzione accessibile via URL.

Nel LAB05 ci sono tre endpoint.

---

### Endpoint `/health`

#### Definizione

È un endpoint molto semplice che serve a verificare se il servizio è vivo.

#### Perché si usa nel mondo reale

* health check
* monitoring
* orchestratori
* load balancer

#### Esempio

```bash
curl -i http://localhost:9000/health
```

#### Risposta tipica

```text
HTTP/1.0 200 OK
Content-Type: application/json
X-Request-Id: ...
```

Body:

```json
{"status":"ok"}
```

#### Cosa significa

Il servizio sta rispondendo correttamente.

---

### Endpoint `/time`

#### Definizione

Restituisce l’orario corrente del server.

#### Perché è utile

Mostra che il server sa generare una risposta dinamica.

#### Esempio

```bash
curl -i http://localhost:9000/time
```

#### Body tipico

```json
{"utc":"2026-03-16T10:15:22.123Z"}
```

#### Cosa dimostra

Il server non risponde solo con contenuti statici, ma esegue logica.

---

### Endpoint `/echo`

#### Definizione

Riceve un dato in input e lo restituisce.

#### Perché si usa come esercizio

È il modo più semplice per mostrare una chiamata POST con JSON.

#### Esempio corretto

```bash
curl -i -X POST http://localhost:9000/echo \
  -H 'Content-Type: application/json' \
  -d '{"msg":"ciao"}'
```

#### Risposta tipica

```json
{"echo":{"msg":"ciao"}}
```

#### Cosa dimostra

* lettura del body
* parsing JSON
* risposta strutturata
* differenza tra GET e POST

---

## 5.5 GET e POST

### GET

Serve di solito per leggere dati.

Esempio:

```bash
curl http://localhost:9000/health
```

### POST

Serve di solito per inviare dati al server.

Esempio:

```bash
curl -X POST http://localhost:9000/echo -d '{"msg":"ciao"}'
```

Nel LAB05:

* `/health` usa GET
* `/time` usa GET
* `/echo` usa POST

---

## 5.6 HTTP 200, 404, 400

### 200 OK

La richiesta è andata bene.

### 404 Not Found

Il server esiste, ma quel percorso non esiste.

Esempio:

```bash
curl -i http://localhost:9000/nope
```

Interpretazione: servizio attivo, endpoint inesistente.

### 400 Bad Request

Il server ha ricevuto la richiesta, ma il contenuto è invalido.

Esempio:

```bash
curl -i -X POST http://localhost:9000/echo \
  -H 'Content-Type: application/json' \
  -d '{"msg":'
```

Il JSON è malformato.
Il servizio risponde 400.

---

## 5.7 Che cos’è un log

Un log è una registrazione di eventi prodotti da un sistema.

Nel LAB05 ogni richiesta genera una riga di log.

Perché è importante:

* capire cosa è successo
* fare troubleshooting
* misurare il comportamento del sistema

---

## 5.8 Log strutturato JSON

Nel nostro lab il log è JSON.

Esempio:

```json
{
  "ts": "2026-03-16T10:18:00Z",
  "level": "INFO",
  "request_id": "abc123",
  "method": "GET",
  "path": "/health",
  "status": 200,
  "duration_ms": 3
}
```

### Spiegazione dei campi

* `ts`: timestamp
* `level`: livello log
* `request_id`: identificatore della richiesta
* `method`: metodo HTTP
* `path`: endpoint richiesto
* `status`: codice HTTP
* `duration_ms`: durata in millisecondi

### Perché è meglio di un log “libero”

Perché è parsabile automaticamente.

---

## 5.9 Che cos’è il request_id

Ogni richiesta riceve un identificatore unico.

### Perché serve

Se nel log ci sono molte richieste, puoi usare il request_id per collegare:

* la risposta ricevuta dal client
* la riga precisa di log

### Esempio

```bash
curl -i http://localhost:9000/health
```

Tra gli header trovi:

```text
X-Request-Id: 123e4567...
```

Poi fai:

```bash
grep '123e4567' logs/app.log
```

e trovi la riga corrispondente.

Questa si chiama **correlazione**.

---

## 5.10 Porta occupata

### Sintomo

Quando avvii il servizio, compare un errore tipo:

```text
Address already in use
```

### Significato

Qualcun altro sta già usando quella porta.

### Diagnosi

```bash
ss -ltnp | grep 9000
```

### Soluzioni

* fermare il processo che la usa
* cambiare porta

---

## 5.11 Perché serve uno script di test

Nel lab c’è `requests.sh`.

### Definizione

È uno script shell che invia automaticamente una sequenza di richieste.

### Perché è utile

* ripetibilità
* meno errori manuali
* evidenze più uniformi

### Cosa contiene

Una piccola sequenza di test:

* health
* time
* echo valido
* 404
* JSON malformato

## Chiarimenti Lab05

In un'**API REST**, un endpoint è l'indirizzo specifico (URL) attraverso il quale un sistema può accedere a una determinata risorsa o funzionalità di un server. 
Immaginiamolo come l'indirizzo di casa di un dato specifico: se vuoi recuperare le informazioni di un utente o pubblicare un nuovo post, devi sapere "dove" bussare nel sistema. 

**Caratteristiche principali**

* Struttura URL: È un percorso unico, ad esempio https://api.esempio.com.
* Interazione tramite Metodi HTTP: Ogni endpoint reagisce in modo diverso a seconda del "verbo" usato nella richiesta:
* GET: Per leggere i dati (es. visualizzare una lista).
   * POST: Per creare nuovi dati (es. registrare un nuovo utente).
   * PUT/PATCH: Per aggiornare dati esistenti.
   * DELETE: Per eliminare una risorsa.
* Punto di Contatto: È il confine dove il software client (come un'app o un browser) comunica con il software server. 

Esempio pratico
Se stai usando l'API di un servizio meteo, potresti avere questi endpoint:

   1. GET /meteo/milano → Ti restituisce le previsioni per Milano.
   2. GET /meteo/roma → Ti restituisce le previsioni per Roma.

In sintesi, mentre l'API è l'intero sistema di regole e interfacce, gli endpoint sono i singoli punti di accesso che compongono quel sistema. 

**Gli endpoint di un servizio Rest hanno a che fare con il routing in programmazione?**


Il concetto di endpoint e quello di routing sono due facce della stessa medaglia: ha a che fare con il routing in programmazione?

* Il Routing è il meccanismo interno (il "vigile urbano"): è il codice sul server che analizza l'URL in arrivo e decide quale funzione deve gestire quella richiesta.
* L'Endpoint è la destinazione finale (l'indirizzo sulla mappa): è il percorso pubblico che esponi all'esterno.

Come lavorano insieme
Quando scrivi il codice di una API (ad esempio in Node.js, Python, Java, .Net o PHP), definisci delle "rotte" (routes).
Esempio concettuale:

   1. L'utente chiama l'endpoint: GET /prodotti/123.
   2. Il sistema di routing del server vede la richiesta e dice: "Ok, il percorso inizia con /prodotti, quindi devo attivare la funzione mostraDettagliProdotto() e passarle l'ID 123".

In sintesi:
Il routing è il processo di instradamento (la logica), mentre l'endpoint è il punto di arrivo di quel processo (l'URL specifico). Senza routing, il server non saprebbe quale endpoint corrisponde a quale pezzo di codice.


# VM, Container, Docker, immagini, comandi fondamentali, Docker Compose, prima app containerizzata

---

# 1. Obiettivo della lezione

Essere in grado di capire:

* che cos’è una macchina virtuale
* che cos’è un container
* quali differenze ci sono tra i due
* quali vantaggi e svantaggi esistono
* che cos’è Docker
* che cos’è un’immagine Docker
* che cos’è un container Docker
* come si costruisce un’immagine
* come si esegue un container
* come si leggono i log
* come si usano i volumi
* come si usano le variabili d’ambiente
* che cos’è Docker Compose
* come si containerizza una piccola app locale

Questa lezione è la base culturale necessaria prima del LAB07 

---

# 2. Prima idea chiave: applicazione, ambiente, packaging

Prima ancora di parlare di VM e container, devi chiarire un problema reale.

Quando sviluppiamo un’applicazione, non basta avere il codice.
Serve anche l’ambiente corretto:

* sistema operativo
* runtime
* librerie
* dipendenze
* configurazione
* porte
* variabili d’ambiente

Il problema classico è questo:

```text
Sul mio PC funziona
Sul tuo no
Sul server no
In produzione rompe tutto
```

VM e container nascono anche per risolvere questo problema.

---

# 3. Che cos’è una VM

## Definizione

Una **VM, Virtual Machine**, è un computer virtuale completo che gira su un computer fisico.

Ha:

* CPU virtuale
* RAM virtuale
* disco virtuale
* scheda di rete virtuale
* sistema operativo completo

Schema:

```text
Hardware fisico
   ↓
Hypervisor
   ↓
VM 1 → Linux
VM 2 → Windows
VM 3 → Ubuntu
```

---

## 3.1 Che cos’è l’hypervisor

L’hypervisor è il software che permette di creare e gestire VM.

Esempi:

* VirtualBox
* VMware
* Hyper-V
* KVM

L’hypervisor prende le risorse fisiche e le suddivide in macchine virtuali.

---

## 3.2 Esempio concreto di VM

Immagina un PC Windows su cui installi VirtualBox e crei una VM Ubuntu.

Dentro la VM hai:

* boot Linux
* kernel Linux
* filesystem Linux
* utente Linux
* processi Linux
* applicazioni Linux

Quella VM si comporta come un vero computer autonomo.

---

## 3.3 Caratteristiche delle VM

Una VM ha:

* un sistema operativo completo
* isolamento forte
* maggiore consumo di risorse
* tempi di avvio più lunghi

Esempio tipico:

```text
Windows host
 └── VM Ubuntu
      └── Python + Flask
```

---

# 4. Che cos’è un container

## Definizione

Un **container** è un’unità di esecuzione isolata che contiene un’applicazione e le sue dipendenze, ma **non contiene un intero sistema operativo completo**.

Schema:

```text
Hardware fisico
   ↓
OS host
   ↓
Docker Engine
   ↓
Container 1 → app A
Container 2 → app B
Container 3 → app C
```

---

## 4.1 Differenza chiave rispetto alla VM

Un container **non virtualizza il computer intero**.
Isola soprattutto:

* processo
* filesystem
* rete
* configurazione

ma condivide il kernel dell’host.

Questa è la differenza più importante da far capire.

---

## 4.2 Esempio concreto

Se esegui:

```bash
docker run nginx
```

non stai avviando un nuovo sistema operativo completo.
Stai avviando un processo nginx in un ambiente isolato.

---

## 4.3 Cosa c’è dentro un container

Dentro un container tipicamente trovi:

* applicazione
* librerie richieste
* filesystem minimale
* eventuale shell
* file di configurazione

Non c’è, in genere:

* boot completo
* init system tradizionale
* desktop
* OS completo come in una VM

---

# 5. Schema visuale VM vs Container

## VM

```text
Hardware
   ↓
Hypervisor
   ├── VM 1
   │    ├── Guest OS
   │    └── App
   ├── VM 2
   │    ├── Guest OS
   │    └── App
   └── VM 3
        ├── Guest OS
        └── App
```

## Container

```text
Hardware
   ↓
OS Host
   ↓
Docker Engine
   ├── Container 1 → App
   ├── Container 2 → App
   └── Container 3 → App
```

---

# 6. Differenza tra VM e container

## Definizione semplice da dire in aula

* **VM** = virtualizza una macchina completa
* **Container** = isola un processo e il suo ambiente

---

## Tabella di confronto

| Aspetto             | VM                | Container          |
| ------------------- | ----------------- | ------------------ |
| Unità virtualizzata | macchina          | processo/app       |
| Sistema operativo   | completo          | condiviso con host |
| Peso                | alto              | basso              |
| Avvio               | lento             | rapido             |
| Isolamento          | forte             | più leggero        |
| Portabilità         | buona             | molto alta         |
| Uso tipico          | ambienti completi | microservizi/app   |

---

# 7. Pro e contro delle VM

## Vantaggi

### Isolamento forte

Ogni VM ha il suo sistema operativo.
Questo riduce l’impatto reciproco tra ambienti.

### Flessibilità

Puoi avere sistemi diversi sullo stesso host:

* una VM Windows
* una VM Linux
* una VM BSD

### Sicurezza

L’isolamento è generalmente più forte rispetto ai container.

---

## Svantaggi

### Più pesanti

Ogni VM richiede:

* disco
* RAM
* CPU
* OS completo

### Avvio più lento

Una VM deve fare boot del sistema operativo.

### Gestione più costosa

Patch, aggiornamenti, spazio, backup, snapshot, tutto è più pesante.

---

# 8. Pro e contro dei container

## Vantaggi

### Leggeri

Un container contiene solo ciò che serve all’applicazione.

### Veloci

Partono molto rapidamente.

### Portabili

Se l’immagine è fatta bene, gira allo stesso modo su più ambienti.

### Ideali per microservizi

Ogni servizio può vivere nel suo container.

---

## Svantaggi

### Isolamento meno forte delle VM

Non hai un OS completo separato.

### Debug meno intuitivo per principianti

Bisogna capire immagini, tag, volumi, mapping porte, rete container.

### Persistenza non automatica

Se non usi volumi, i dati scritti nel container possono perdersi.

---

# 9. Quando usare VM e quando container

## VM

Meglio quando ti serve:

* un sistema completo
* forte separazione
* ambienti diversi a livello OS

## Container

Meglio quando ti serve:

* distribuire un’app
* fare deploy rapidi
* avere ambienti replicabili
* orchestrare servizi

---

# 10. Che cos’è Docker

## Definizione

Docker è una piattaforma che permette di:

* costruire immagini
* eseguire container
* gestire rete, volumi, log e configurazione

Schema:

```text
Docker CLI
   ↓
Docker Engine
   ↓
Images
   ↓
Containers
```

---

## 10.1 Componenti principali di Docker

### Docker CLI

È il comando `docker` che usi da terminale.

### Docker Engine

È il motore che crea e gestisce container.

### Docker Image

È il modello, il pacchetto immutabile.

### Docker Container

È l’istanza in esecuzione dell’immagine.

---

# 11. Che cos’è una Docker Image

## Definizione

Una **image** è un pacchetto immutabile che contiene:

* base OS minimale
* runtime
* librerie
* applicazione
* configurazione iniziale

È il blueprint da cui nasce il container.

---

## Esempio concettuale

```text
Image = fotografia del filesystem + istruzioni di avvio
Container = esecuzione concreta di quella fotografia
```

---

## 11.1 Tag delle immagini

Un’immagine si identifica spesso con:

```text
nome:tag
```

Esempi:

```bash
python:3.12-slim
nginx:latest
obsapp:v1
obsapp:v2
```

Il tag serve a distinguere versioni diverse.

---

# 12. Che cos’è un container Docker

## Definizione

Un container è un’istanza avviata da un’immagine.

Esempio:

```bash
docker run nginx
```

significa:

* se serve, usa l’immagine nginx
* crea un container
* esegue il processo definito

---

## Esempio utile da spiegare

Immagina:

```text
Image = stampo
Container = oggetto realizzato usando quello stampo
```

Puoi avere più container dalla stessa immagine.

---

# 13. Che cos’è un Dockerfile

## Definizione

Il `Dockerfile` è il file che descrive come costruire un’immagine.

Contiene istruzioni come:

* immagine di base
* copia file
* installazione dipendenze
* variabili ambiente
* comando di avvio

---

## Esempio commentato

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/app.py .
ENV PORT=8000
EXPOSE 8000
CMD ["python","app.py"]
```

Spiegazione riga per riga:

### `FROM python:3.12-slim`

usa come base un’immagine Python minimale

### `WORKDIR /app`

imposta la directory di lavoro nel container

### `COPY src/requirements.txt .`

copia il file requirements dentro il container

### `RUN pip install --no-cache-dir -r requirements.txt`

installa le dipendenze

### `COPY src/app.py .`

copia il codice dell’app

### `ENV PORT=8000`

imposta una variabile ambiente di default

### `EXPOSE 8000`

documenta che l’app usa la porta 8000

### `CMD ["python","app.py"]`

comando da eseguire all’avvio del container

---

# 14. Ciclo di vita base in Docker

Schema semplice:

```text
Dockerfile
   ↓
docker build
   ↓
Image
   ↓
docker run
   ↓
Container
```

---

# 15. Comandi fondamentali Docker

## 15.1 Verifica installazione

```bash
docker version
docker ps
docker compose version
```

### `docker version`

mostra client e server Docker

### `docker ps`

mostra i container in esecuzione

### `docker compose version`

verifica il supporto Compose

Questi sono anche i comandi di verifica richiesti nel LAB07 

---

## 15.2 Cercare immagini già presenti

```bash
docker images
```

Mostra:

* repository
* tag
* image ID
* data creazione
* dimensione

---

## 15.3 Costruire una immagine

```bash
docker build -t obsapp:v1 .
```

Spiegazione:

* `build` = costruisci l’immagine
* `-t obsapp:v1` = assegna nome e tag
* `.` = usa il contesto corrente

---

## 15.4 Eseguire un container

```bash
docker run obsapp:v1
```

Questo però spesso non basta, perché l’app potrebbe usare una porta.

Più realisticamente:

```bash
docker run -d --name obsapp -p 8000:8000 obsapp:v1
```

Spiegazione:

* `-d` = detached, gira in background
* `--name obsapp` = assegna nome al container
* `-p 8000:8000` = mappa porta host:container

---

## 15.5 Vedere i container in esecuzione

```bash
docker ps
```

Mostra container attivi.

Per vedere anche quelli fermati:

```bash
docker ps -a
```

---

## 15.6 Fermare un container

```bash
docker stop obsapp
```

oppure con ID container.

---

## 15.7 Rimuovere un container

```bash
docker rm obsapp
```

Forzare stop + rimozione:

```bash
docker rm -f obsapp
```

Nel LAB07 viene usato spesso proprio questo pattern 

---

## 15.8 Leggere i log del container

```bash
docker logs obsapp
```

Solo ultime righe:

```bash
docker logs --tail 20 obsapp
```

Seguire in tempo reale:

```bash
docker logs -f obsapp
```

Questo è un concetto centrale nel LAB07, dove viene chiesto esplicitamente di usare `docker logs` 

---

## 15.9 Entrare nel container

Se disponibile una shell:

```bash
docker exec -it obsapp bash
```

oppure:

```bash
docker exec -it obsapp sh
```

Serve per esplorare il filesystem interno del container.

---

## 15.10 Rimuovere immagini

```bash
docker rmi obsapp:v1
```

Attenzione: non puoi rimuovere un’immagine se esistono container che la usano ancora.

---

# 16. Port mapping

## Concetto fondamentale

Se il processo nel container ascolta su porta 8000, non è detto che sia accessibile dall’host.

Serve il mapping.

```bash
docker run -p 8000:8000 obsapp:v1
```

Significa:

* porta 8000 dell’host
* inoltrata alla porta 8000 del container

---

## Esempio di errore utile

```bash
docker run -d --name obsapp -p 8080:8000 obsapp:v1
```

Ora il servizio risponde su:

```bash
http://localhost:8080
```

e non su `http://localhost:8000`

Questo è esattamente il drill sulla porta sbagliata presente nel LAB07 

---

# 17. Volumi

## Perché servono

Un container è, per natura, effimero.
Se scrivi dati solo dentro il suo filesystem interno, potresti perderli quando il container viene rimosso.

Per rendere i dati persistenti usi i volumi o i bind mount.

---

## Esempio

```bash
docker run -v "$(pwd)/logs/data:/data" obsapp:v2
```

Significa:

* directory host `logs/data`
* montata dentro il container come `/data`

Se l’app scrive in `/data/app_events.log`, il file appare anche sull’host.

Questo è uno dei punti centrali del LAB07, dove si richiede persistenza su `logs/data/app_events.log` 

---

## Schema

```text
Host filesystem
  logs/data/app_events.log
        ⇅
Container filesystem
        /data/app_events.log
```

---

# 18. Variabili d’ambiente

## Che cosa sono

Le variabili ambiente permettono di configurare il comportamento dell’app senza modificare il codice.

Esempio:

```bash
docker run -e PORT=9000 obsapp:v2
```

L’app legge `PORT` e si adegua.

Questo compare esplicitamente nel LAB07 

---

## Vantaggi

* configurazione flessibile
* separazione codice/config
* pratiche DevOps corrette

---

# 19. Docker Compose

## Che cos’è

Docker Compose serve a descrivere più configurazioni container in un file YAML, così l’avvio è ripetibile.

---

## Perché usarlo

Con `docker run` la riga di comando può diventare lunga e fragile.

Compose permette di scrivere:

* immagine
* nome container
* porte
* env
* volumi
* restart policy

in modo leggibile.

---

## Esempio dal LAB07

```yaml
services:
  obsapp:
    image: obsapp:v2
    container_name: obsapp
    environment:
      - PORT=8000
    ports:
      - "8000:8000"
    volumes:
      - ./logs/data:/data
    restart: unless-stopped
```

Questo file definisce il servizio `obsapp` in modo ripetibile 

---

## Comandi fondamentali Compose

```bash
docker compose up -d
docker compose ps
docker compose logs --tail 20
docker compose down
```

Spiegazione:

* `up -d` = avvia in background
* `ps` = mostra i servizi attivi
* `logs` = mostra i log
* `down` = ferma e rimuove i container del progetto

---

# 20. Parte pratica guidata

# App semplicissima in VSCode su WSL

Qui ti propongo una mini app molto semplice, compatibile con la logica del LAB07.

---

## 20.1 Creazione struttura

```bash
mkdir -p ~/corso_obs/demo_docker/src
cd ~/corso_obs/demo_docker
```

---

## 20.2 Codice app in `src/app.py`

```python
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route("/health")
def health():
    return jsonify({"status": "ok", "service": "demo-docker"})

if __name__ == "__main__":
    port = int(os.environ.get("PORT", "8000"))
    app.run(host="0.0.0.0", port=port)
```

---

## 20.3 File `src/requirements.txt`

```text
flask==3.0.3
```

---

## 20.4 Test locale senza Docker

```bash
cd ~/corso_obs/demo_docker
python3 -m venv .venv
source .venv/bin/activate
pip install -r src/requirements.txt
python3 src/app.py
```

In un secondo terminale:

```bash
curl -s http://localhost:8000/health
```

Output atteso:

```json
{"service":"demo-docker","status":"ok"}
```

---

# 21. Containerizzazione dell’app

## 21.1 Dockerfile

Nel root del progetto:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY src/app.py .
ENV PORT=8000
EXPOSE 8000
CMD ["python","app.py"]
```

---

## 21.2 Build

```bash
docker build -t demoapp:v1 .
```

---

## 21.3 Run

```bash
docker rm -f demoapp 2>/dev/null || true
docker run -d --name demoapp -p 8000:8000 demoapp:v1
```

---

## 21.4 Test

```bash
docker ps
curl -s http://localhost:8000/health
docker logs --tail 20 demoapp
```

---

# 22. Esempi di troubleshooting didattico

## Caso 1: porta host sbagliata

```bash
docker rm -f demoapp
docker run -d --name demoapp -p 8080:8000 demoapp:v1
curl -s http://localhost:8000/health || echo FAIL
curl -s http://localhost:8080/health
```

Spiegazione:

* l’app nel container ascolta su 8000
* l’host espone 8080
* quindi devi interrogare 8080 sull’host

Questo corrisponde al drill del LAB07 

---

## Caso 2: container fermo

```bash
docker ps -a
docker logs demoapp
```

Interpretazione:

* se non è in `docker ps`, forse è fermo
* i log aiutano a capire il motivo

---

## Caso 3: volume non montato

Se l’app scrive in `/data/file.log` ma non monti un volume:

* il file resta nel container
* se il container viene rimosso, il dato sparisce

---

# 23. Versioning delle immagini

Nel LAB07 si lavora con `obsapp:v1` e `obsapp:v2` 

Questo introduce un concetto DevOps importante: **versionare l’immagine**.

Esempio:

```bash
docker build -t obsapp:v1 .
docker build -t obsapp:v2 .
```

Puoi poi scegliere quale eseguire:

```bash
docker run -d --name obsapp -p 8000:8000 obsapp:v1
```

oppure

```bash
docker run -d --name obsapp -p 8000:8000 obsapp:v2
```

---

## Rollback

Se la v2 ha problemi:

```bash
docker rm -f obsapp
docker run -d --name obsapp -p 8000:8000 obsapp:v1
```

Questa è una forma base di rollback.

---

# 24. Collegamento con il LAB07

# Concetti preliminari e fondamentali per eseguirlo in autonomia

Il LAB07 chiede di containerizzare un servizio, gestire versioni, volumi, log, env, compose e troubleshooting 
Per eseguirlo in autonomia bisogna aver capito questi concetti.

---

## 24.1 Concetto: differenza tra codice sorgente e immagine

Nel LAB07 il codice sta in `src/`, ma il container non legge direttamente il tuo filesystem locale come farebbe Python lanciato a mano.

Bisogna:

1. avere i file sorgente
2. descrivere come impacchettarli nel Dockerfile
3. fare build
4. ottenere immagine
5. eseguire il container da quell’immagine

Se non si capisce questa catena, non capiremo perché modificheremo il codice ma il container continua a comportarsi come prima.

---

## 24.2 Concetto: build

Nel LAB07 trovi:

```bash
docker build -t obsapp:v1 .
```

Questo comando non avvia l’app.
Costruisce l’immagine.

Bisogna inoltre capire bene la differenza tra:

* `docker build` = costruzione
* `docker run` = esecuzione

---

## 24.3 Concetto: tag v1 e v2

Nel LAB07 ci sono `obsapp:v1` e `obsapp:v2` 

Questo serve a far vedere:

* versioni distinte della stessa app
* confronto tra release
* rollback

È fondamentale comprendere che il tag non è una semplice decorazione.
È una vera etichetta di versione dell’immagine.

---

## 24.4 Concetto: health endpoint

Nel LAB07 si chiede che l’app Flask esponga `/health` 

 Bisogna sapere che:

* `/health` è un endpoint di controllo
* serve a verificare se il servizio è vivo
* è usato da monitoring, orchestratori, load balancer

Quindi:

```bash
curl -s http://localhost:8000/health
```

non è una chiamata casuale. È il test più semplice per verificare il servizio.

---

## 24.5 Concetto: port mapping host/container

Nel LAB07 compare spesso `-p 8000:8000` oppure `-p 8080:8000` 

Interiorizzare questa sintassi:

```text
-p porta_host:porta_container
```

Esempio:

```bash
docker run -p 8080:8000 obsapp:v1
```

Significa:

* dall’esterno chiami `localhost:8080`
* dentro il container l’app ascolta su 8000

Se non è ben chiaro, è uno dei punti che crea più confusione.

---

## 24.6 Concetto: logs del container

Nel LAB07 si usa:

```bash
docker logs --tail 20 obsapp
```

Bisogna sapere:

* il container scrive output standard
* Docker intercetta stdout/stderr
* `docker logs` mostra quell’output

Se l’app non risponde, i log sono uno dei primi strumenti diagnostici.

---

## 24.7 Concetto: persistenza e volume

Nel LAB07 la v2 scrive anche su `/data/app_events.log` e poi monta:

```bash
-v "$(pwd)/logs/data:/data"
```

Questo introduce un concetto fondamentale:
**se il dato ti interessa, non lasciarlo solo nel filesystem del container**.

Dobbiamo tener presente che:

* il container può sparire
* i dati persistenti devono stare fuori o su volume dedicato

---

## 24.8 Concetto: ENV

Nel LAB07 si usa:

```bash
-e PORT=9000
```

Quindi bisogna aver chiaro che l’app può leggere il valore della porta da una variabile ambiente.

È una pratica standard per rendere i container configurabili senza modificare il codice.

---

## 24.9 Concetto: Compose come configurazione ripetibile

Nel LAB07 si crea `docker-compose.yml` con servizio, volume, env, porte, restart policy 

Compose è utile perché:

* evita comandi lunghi
* riduce errori
* rende il setup ripetibile
* documenta l’architettura locale

---

## 24.10 Concetto: troubleshooting strutturato

Il LAB07 chiede anche troubleshooting su due scenari 

Qui ci agganciamo ai lab precedenti:

* sintomo
* ipotesi
* test
* causa
* fix
* evidenza

Esempi nel contesto Docker:

### Porta già in uso

Sintomo: il container non parte o l’app non è raggiungibile
Test: `docker ps`, `ss -ltnp`, `docker logs`

### Volume non montato

Sintomo: il file atteso non compare su host
Test: controllare comando `docker run`, path host, path container

### Container crash

Sintomo: non compare in `docker ps`
Test: `docker ps -a`, `docker logs`

### ENV mancante

Sintomo: l’app parte su porta diversa o fallisce
Test: controllare `docker run`, codice app, log

---


