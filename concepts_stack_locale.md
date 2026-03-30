# Lezione – Stack di Observability: Prometheus e Grafana

## Perché stiamo introducendo questi strumenti

Finora avete:

* creato infrastrutture
* distribuito applicazioni
* lavorato con container
* usato Azure

Ma quando qualcosa va male, nasce una domanda:

**come capiamo cosa sta succedendo?**

Non possiamo entrare nei container e leggere codice ogni volta.
Serve un modo standard per:

* misurare
* osservare
* analizzare
* reagire

Qui entrano in gioco **Prometheus e Grafana**.

---

# Il concetto di metriche

Prima di parlare degli strumenti, partiamo dal concetto.

Una **metrica** è un numero nel tempo.

Esempi:

* CPU = 45%
* richieste = 120/sec
* latenza = 250 ms
* errori = 3/minuto

Le metriche servono per:

* vedere andamento nel tempo
* individuare anomalie
* confrontare periodi
* creare alert

Le metriche sono **la base dell’observability**.

---

# Prometheus – cos’è

Prometheus è un sistema che raccoglie metriche.

Non monitora attivamente.
Non installa agent complessi.

Prometheus **legge le metriche esposte dai servizi**.

Questo modello si chiama:

**pull model**

---

# Come funziona Prometheus

Flusso semplificato:

```text id="9m6tsh"
Application → espone /metrics
Prometheus → legge /metrics
Prometheus → salva dati nel tempo
Grafana → legge da Prometheus
```

Prometheus fa 3 cose:

1. legge metriche
2. salva serie temporali
3. permette query

---

# Endpoint /metrics

Ogni applicazione espone:

```
/metrics
```

Esempio reale:

```
http_requests_total 120
http_request_duration_seconds 0.23
cpu_usage 0.56
```

Prometheus legge questi dati ogni pochi secondi.

Questo processo si chiama:

**scraping**

---

# Scraping interval

Prometheus interroga i servizi a intervalli regolari.

Esempio:

```
scrape_interval: 5s
```

Significa:

ogni 5 secondi Prometheus chiede le metriche.

Questo crea una **serie temporale**.

---

# Serie temporali

Prometheus salva dati nel formato:

```
metrica + timestamp + valore
```

Esempio:

```
http_requests_total
12:00 → 100
12:05 → 150
12:10 → 200
```

Da qui si possono calcolare:

* rate
* media
* percentili
* trend

---

# Tipi di metriche Prometheus

Prometheus usa 4 tipi principali.

## Counter

Conta eventi.

Esempio:

* numero richieste
* numero errori

Cresce sempre.

---

## Gauge

Valore che sale e scende.

Esempio:

* memoria
* CPU
* connessioni attive

---

## Histogram

Distribuzione di valori.

Serve per:

* latenza
* tempi risposta

Permette di calcolare:

* P50
* P95
* P99

---

## Summary

Simile a histogram ma meno flessibile.

Si usa meno.

---

# Query Prometheus (PromQL)

Prometheus ha un linguaggio di query.

Esempio base:

```
http_requests_total
```

Mostra il totale richieste.

---

# Rate

Per vedere richieste al secondo:

```
rate(http_requests_total[1m])
```

Questo è fondamentale per capire traffico.

---

# Latenza P95

Una delle metriche più usate in SRE.

```
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

Significa:

il 95% delle richieste è più veloce di questo valore.

---

# Perché Prometheus è così usato

Perché:

* semplice
* pull model
* time series DB
* query potente
* alerting integrato
* cloud native

È diventato uno standard.

## Link di consultazione
   https://bitrock.it/it/blog-it/tecnologia/getting-started-with-prometheus.html

## dove vengono salvati i dati in Prometheus?
I dati esposti da un'applicazione vengono salvati localmente da Prometheus in un database a serie temporali (TSDB) ottimizzato per alte prestazioni. [1, 2] 
Dove vengono salvati i dati?
I dati risiedono in una directory specifica definita all'avvio del server (solitamente ./data o /var/lib/prometheus). La struttura della directory è organizzata in blocchi temporali (di default ogni 2 ore) che contengono i campioni persistiti su disco. 
Struttura tipica della cartella dati:

* wal/: Contiene il Write-Ahead Log, ovvero i dati recenti appena ricevuti e non ancora strutturati in blocchi definitivi, usati per il ripristino in caso di crash.
* Cartelle esadecimali (es. 01H.../): Rappresentano i blocchi di dati storici. Ogni cartella contiene:
* chunks/: I dati reali delle metriche, compressi.
   * index: Un indice che mappa i nomi delle metriche e le etichette (label) ai dati nei chunks.
   * meta.json: Metadati sul blocco (intervallo temporale, statistiche). 
In che formato vengono salvati?
Prometheus salva i dati come serie temporali, composte da coppie (timestamp, valore). 

* Timestamp: Precisione in millisecondi.
* Valore: Numero in virgola mobile a 64 bit (float64).
* Compressione: Viene utilizzato un algoritmo di compressione (spesso basato su XOR) per ridurre drasticamente l'occupazione di spazio su disco (ogni campione occupa mediamente solo 1-2 byte). 

Esempio di dato salvato
Immaginiamo che la tua app esponga una metrica per contare le richieste HTTP. Prometheus la salva internamente associandola a etichette univoche: 

| Metrica + Label | Timestamp | Valore (float64) |
|---|---|---|
| http_requests_total{method="GET", endpoint="/api"} | 1711792800000 | 1250.0 |
| http_requests_total{method="GET", endpoint="/api"} | 1711792815000 | 1262.0 |

Ogni variazione di una singola etichetta (es. method="POST") crea una nuova serie temporale distinta nel database. [
Vuoi sapere come configurare il periodo di conservazione (retention) per evitare che il disco si riempia?




---

# Grafana – cos’è

Grafana è lo strumento di visualizzazione.

Prometheus raccoglie dati.
Grafana li rende leggibili.

Grafana permette:

* dashboard
* grafici
* correlazioni
* visualizzazione realtime

---

# Flusso completo

```text id="2x02i0"
Application
   │
   ▼
Prometheus
   │
   ▼
Grafana
   │
   ▼
Utente
```

---

# Dashboard Grafana

Una dashboard è un insieme di pannelli.

Ogni pannello contiene:

* query
* grafico
* aggiornamento realtime

Esempi pannelli:

* richieste/sec
* error rate
* latenza P95
* CPU

---

# Esempio dashboard SRE

Tipicamente:

```
RPS
Error rate
P95 latency
CPU
Memory
```

Guardando questa dashboard si capisce:

* sistema sotto carico
* errori
* rallentamenti

---

# Aggiornamento realtime

Grafana aggiorna ogni:

* 5s
* 10s
* 30s

Questo permette di:

* vedere incidenti live
* osservare effetti deploy
* analizzare carico

---

# Collegamento Prometheus-Grafana

Grafana usa Prometheus come data source.

Configurazione:

* tipo: Prometheus
* URL: [http://prometheus:9090](http://prometheus:9090)

Grafana non salva metriche.
Le legge.

---

# Esempi grafici utili

## Request rate

Traffico applicazione.

```
rate(http_requests_total[1m])
```

---

## Error rate

Qualità del servizio.

```
sum(rate(http_requests_total{status="500"}[1m]))
```

---

## Latenza

Prestazioni.

```
histogram_quantile(0.95, ...)
```

---

# Come si usa in pratica

Workflow reale:

1. applicazione espone metriche
2. Prometheus le raccoglie
3. Grafana visualizza
4. SRE osserva
5. se qualcosa cambia → analisi

---

# Cosa faremo nel laboratorio

Nel laboratorio:

* creeremo un'app Python
* esporremo metriche
* Prometheus le raccoglierà
* Grafana mostrerà grafici
* genereremo traffico
* simuleremo problemi

---

# Cosa osservare

Durante il lab guarderete:

* aumento traffico
* aumento latenza
* errori
* degradazione

Imparerete a collegare:

metrica → comportamento → problema

---

# Collegamento con Azure

Questi concetti sono identici in cloud.

Prometheus → Azure Monitor
Grafana → Azure dashboards
metrics → Azure metrics

Cambiano gli strumenti, non il metodo.

---

# Riassunto

Prometheus raccoglie metriche
Grafana visualizza metriche
Metriche = numeri nel tempo
Histogram → latenza
Rate → traffico
Dashboard → stato sistema

---

Questo è lo stack base di observability.
Da qui si costruisce tutto il resto.

Certamente c'è ancora molto altro da pprendere. Ma compresobene Prometheus e Grafana, abbiamo già fatto un passo avanti importante in ambito SRE. 
