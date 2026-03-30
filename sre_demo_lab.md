# LAB – Introduzione pratica a SRE con Observability (stack locale)

## Obiettivo del laboratorio
Questo laboratorio dimostrativo introduce i concetti fondamentali di **Site Reliability Engineering (SRE)** attraverso la costruzione e l'analisi di un piccolo sistema osservabile in ambiente locale.

Lo scopo non è soltanto installare strumenti di monitoring, ma comprendere **come ragiona un SRE quando un sistema degrada o fallisce**.

Durante il laboratorio verranno implementati:

- un microservizio Python osservabile
- esportazione di metriche Prometheus
- raccolta metriche tramite Prometheus
- visualizzazione tramite Grafana
- generazione di traffico applicativo
- simulazione di guasti (failure injection)
- analisi di latenza, errori e throughput

---

# Prerequisiti

Ambiente locale richiesto:

- Windows 10 o Windows 11
- WSL2
- Ubuntu su WSL
- Docker Desktop
- Python 3
- VS Code
- Git

Verificare che Docker funzioni correttamente:

```bash
docker --version
docker compose version
```

---

# Architettura del laboratorio

Il laboratorio simula un piccolo sistema distribuito.

```
Load Generator
     |
     v
Python FastAPI Service
     |
     v
Prometheus (metric scraping)
     |
     v
Grafana (visualizzazione)
```

Componenti principali:

| componente | funzione |
|---|---|
| FastAPI service | applicazione monitorata |
| Prometheus client | esportazione metriche |
| Prometheus | raccolta metriche |
| Grafana | dashboard monitoring |
| load generator | traffico simulato |

---

# Struttura del repository

```
sre-demo-lab/

app/
  app.py
  requirements.txt
  Dockerfile

monitoring/
  prometheus.yml

loadtest/
  load.py


docker-compose.yml
README.md
```

---

# Step 1 – Creazione cartelle

Aprire terminale Ubuntu WSL.

```bash
mkdir -p ~/sre-demo-lab/app
mkdir -p ~/sre-demo-lab/monitoring
mkdir -p ~/sre-demo-lab/loadtest

cd ~/sre-demo-lab
```

---

# Step 2 – File requirements Python

Creare il file:

```
app/requirements.txt
```

Contenuto:

```
fastapi
uvicorn
prometheus-client
```

---

# Step 3 – Applicazione Python osservabile

Creare:

```
app/app.py
```

Il servizio espone:

- endpoint normale
- endpoint lento
- endpoint con errore
- endpoint metriche

```python
from fastapi import FastAPI, Response
from prometheus_client import Counter, Histogram, generate_latest, CONTENT_TYPE_LATEST
import time
import random

app = FastAPI()

REQUEST_COUNT = Counter(
    "http_requests_total",
    "Total HTTP requests",
    ["method", "endpoint", "status"]
)

REQUEST_LATENCY = Histogram(
    "http_request_duration_seconds",
    "HTTP request latency",
    ["endpoint"]
)

@app.get("/")
def home():
    start = time.time()

    delay = random.uniform(0.05, 0.20)
    time.sleep(delay)

    elapsed = time.time() - start

    REQUEST_COUNT.labels("GET", "/", "200").inc()
    REQUEST_LATENCY.labels("/").observe(elapsed)

    return {"status": "ok", "delay": delay}


@app.get("/slow")
def slow():

    start = time.time()

    delay = random.uniform(1.0, 2.5)
    time.sleep(delay)

    elapsed = time.time() - start

    REQUEST_COUNT.labels("GET", "/slow", "200").inc()
    REQUEST_LATENCY.labels("/slow").observe(elapsed)

    return {"status": "slow", "delay": delay}


@app.get("/error")
def error():

    REQUEST_COUNT.labels("GET", "/error", "500").inc()

    return {"error": "simulated"}, 500


@app.get("/metrics")
def metrics():
    return Response(generate_latest(), media_type=CONTENT_TYPE_LATEST)
```

---

# Step 4 – Dockerfile applicazione

Creare:

```
app/Dockerfile
```

```
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

# Step 5 – Configurazione Prometheus

Creare:

```
monitoring/prometheus.yml
```

```
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "python-app"
    metrics_path: /metrics
    static_configs:
      - targets: ["app:8000"]
```

---

# Step 6 – Docker Compose

Creare:

```
docker-compose.yml
```

```
services:

  app:
    build: ./app
    container_name: sre_app
    ports:
      - "8000:8000"

  prometheus:
    image: prom/prometheus
    container_name: sre_prometheus
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    container_name: sre_grafana
    ports:
      - "3000:3000"
```

---

# Step 7 – Avvio dello stack

Da terminale:

```bash
docker compose up -d --build
```

Verifica container:

```bash
docker ps
```

---

# Step 8 – Test del servizio

Test endpoint:

```bash
curl http://localhost:8000/

curl http://localhost:8000/slow

curl http://localhost:8000/error

curl http://localhost:8000/metrics
```

---

# Step 9 – Generatore di traffico

Creare:

```
loadtest/load.py
```

```
import requests
import random
import time

BASE = "http://localhost:8000"

ENDPOINTS = ["/", "/", "/", "/slow", "/error"]

while True:

    ep = random.choice(ENDPOINTS)

    try:
        r = requests.get(BASE + ep, timeout=5)
        print(ep, r.status_code)
    except Exception as e:
        print("error", e)

    time.sleep(0.2)
```

Installare requests:

```bash
pip install requests
```

Eseguire:

```bash
python3 load.py
```

---

# Step 10 – Accesso ai tool

Prometheus:

```
http://localhost:9090
```

Grafana:

```
http://localhost:3000
```

Login Grafana:

```
admin
admin
```

---

# Query Prometheus utili

Request rate:

```
rate(http_requests_total[1m])
```

Request rate per endpoint:

```
sum by (endpoint) (rate(http_requests_total[1m]))
```

Error rate:

```
sum(rate(http_requests_total{status="500"}[1m]))
```

P95 latency:

```
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

---

# Failure Injection

Simulare degrado del sistema.

### Scenario 1 – aumento latenza

Modificare nel codice:

```
delay = random.uniform(1.0, 2.0)
```

Ricostruire:

```bash
docker compose up -d --build
```

Effetto osservabile:

- aumento P95
- riduzione throughput

---

### Scenario 2 – aumento errori

Modificare il load generator aumentando `/error`.

Effetto:

- incremento error rate

---

### Scenario 3 – crash servizio

```
docker stop sre_app
```

Osservare l'effetto sulle metriche.

Riavviare:

```
docker start sre_app
```

---

# Concetti SRE osservabili nel laboratorio

Questo laboratorio permette di discutere:

### SLI

Service Level Indicator:

```
successful_requests / total_requests
```

### SLO

Esempio:

```
99.9% availability
```

### Error Budget

Con 99.9% SLO:

```
~43 minuti downtime mese
```

---

# Output attesi dal laboratorio

Gli studenti dovranno produrre:

- screenshot dashboard Grafana
- metriche osservate
- descrizione comportamento sistema
- analisi di un incidente simulato
- proposta di remediation

---

# Conclusione

Questo laboratorio introduce il **ciclo operativo tipico SRE**:

1 servizio

2 telemetria

3 carico

4 degradazione

5 osservazione

6 analisi

7 decisione

Il focus non è lo strumento ma il **metodo di analisi dei sistemi distribuiti**.

