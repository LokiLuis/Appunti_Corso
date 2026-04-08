# LAB Extra — Performance Analysis con KQL + Incident Simulation & Troubleshooting

## Autore

**Luis Dragos Istrate** — Lab creato come contributo personale al percorso Observability, Modulo 3.

---


# LABORATORIO GUIDATO — Parte A

## Step A1 - Prepara l'ambiente locale

Apri WSL Ubuntu ed esegui:

```bash
mkdir -p ~/course/lab_extra && cd ~/course/lab_extra
script -a cmdlog_lab_extra.txt
mkdir -p docs
```

### Che cosa fanno questi comandi

- `mkdir -p ~/course/lab_extra && cd ~/course/lab_extra` crea la cartella del laboratorio e vi entra
- `script -a cmdlog_lab_extra.txt` registra la sessione del terminale
- `mkdir -p docs` crea la cartella per le evidenze

---

## Step A2 - Genera traffico diversificato

Prima di fare analisi, genera un mix realistico di traffico verso la tua ACI.

Imposta la variabile IP:

```bash
export RG="rg-observability-lab"
export ACI_PUBLIC_IP=$(az container show --resource-group "$RG" --name "obsapp-aci" --query ipAddress.ip --output tsv)
echo "$ACI_PUBLIC_IP"
```

Poi genera traffico misto:

```bash
for i in {1..5}; do curl -s "http://$ACI_PUBLIC_IP:8000/health" > /dev/null; done
for i in {1..10}; do curl -s "http://$ACI_PUBLIC_IP:8000/time" > /dev/null; done
for i in {1..7}; do curl -s "http://$ACI_PUBLIC_IP:8000/nope" > /dev/null; done
```

### Perché questo mix

- 5 chiamate a `/health` (successo)
- 10 chiamate a `/time` (successo, endpoint più trafficato)
- 7 chiamate a `/nope` (errore 404)

Questo crea un dataset con distribuzione non uniforme, più realistico di un test con solo `/nope`.

### Evidenza richiesta

Annota che il traffico è stato generato e attendi **3-5 minuti** per l'ingestione.

---

## Step A3 - Apri Log Analytics e verifica i dati

Nel portale Azure, apri il tuo workspace **law-observability-francia** e vai su **Logs**.

Lancia questa query di verifica:

```kusto
ContainerInstanceLog_CL
| extend parsed = parse_json(Message)
| extend path = tostring(parsed.path), status = toint(parsed.status), duration = toreal(parsed.duration_ms)
| where isnotempty(path)
| take 10
```

### Che cosa fa questa query

- legge i log
- estrae `path`, `status` e `duration_ms` dal JSON
- filtra solo le righe che hanno un path (esclude log di avvio del container)
- mostra 10 righe

### Che cosa devi osservare

Verifica che nelle colonne `path`, `status` e `duration` compaiano valori reali.

### Evidenza richiesta

Copia o fotografa il risultato.

---

## Step A4 - Calcola la latenza media per endpoint

Esegui:

```kusto
ContainerInstanceLog_CL
| extend parsed = parse_json(Message)
| extend path = tostring(parsed.path), duration = toreal(parsed.duration_ms)
| where isnotempty(path)
| summarize avg_latency_ms = avg(duration) by path
| sort by avg_latency_ms desc
```

### Che cosa misura

Ti mostra quale endpoint è mediamente più lento.

### Perché è utile

Se un endpoint ha latenza media molto più alta degli altri, potrebbe indicare un problema specifico su quella funzione dell'app.

### Evidenza richiesta

Copia il risultato nel file evidenze.

---

## Step A5 - Trova i top endpoint per volume di traffico

Esegui:

```kusto
ContainerInstanceLog_CL
| extend path = tostring(parse_json(Message).path)
| where isnotempty(path)
| summarize total_calls = count() by path
| sort by total_calls desc
```

### Che cosa misura

Mostra quante richieste ha ricevuto ogni endpoint.

### Perché è utile

Ti dice dove si concentra il traffico. Se il 50% delle richieste va su `/time`, sai che `/time` è l'endpoint critico da monitorare con più attenzione.

### Evidenza richiesta

Copia il risultato nel file evidenze.

---

## Step A6 - Calcola la distribuzione degli status code

Esegui:

```kusto
ContainerInstanceLog_CL
| extend status = toint(parse_json(Message).status)
| where status > 0
| summarize count_per_status = count() by status
| sort by status asc
```

### Che cosa misura

Mostra quante richieste hanno avuto ciascun status code (200, 404, 500, ecc.).

### Perché è utile

Ti dà una vista immediata della "salute" del servizio. Se vedi molti 404 o 500, sai che c'è un problema.

### Collegamento con LAB12

Nel LAB12 hai calcolato l'error_rate come percentuale. Qui vedi i numeri assoluti per status, che è un'informazione complementare.

### Evidenza richiesta

Copia il risultato nel file evidenze.

---

## Step A7 - Analizza il trend della latenza nel tempo

Esegui:

```kusto
ContainerInstanceLog_CL
| extend duration = toreal(parse_json(Message).duration_ms)
| where duration > 0
| summarize avg_latency = avg(duration) by bin(TimeGenerated, 5m)
| sort by TimeGenerated asc
| render timechart
```

### Che cosa misura

Raggruppa la latenza media in finestre di 5 minuti e la mostra come grafico temporale.

### Perché è utile

Ti permette di rispondere alla domanda: "la latenza sta peggiorando nel tempo?"

### Nota sul render

L'operatore `render timechart` suggerisce al portale Azure di mostrare il risultato come grafico a linea. Se il portale non lo supporta in modo automatico, puoi cliccare sul tab **Chart** nei risultati.

### Evidenza richiesta

Fotografa il grafico oppure copia i dati nel file evidenze.

---

## Step A8 - Calcola il percentile 95 della latenza

Esegui:

```kusto
ContainerInstanceLog_CL
| extend duration = toreal(parse_json(Message).duration_ms)
| where duration > 0
| summarize p95 = percentile(duration, 95), p50 = percentile(duration, 50), avg_lat = avg(duration)
```

### Che cosa misura

Calcola tre indicatori sulla stessa colonna:

- **p50** (mediana): il 50% delle richieste è più veloce di questo valore
- **p95**: il 95% delle richieste è più veloce di questo valore
- **avg_lat**: media aritmetica

### Perché è utile

Il confronto tra media e percentile ti dice quanto la distribuzione è "distorta". Se p95 è molto più alto della media, significa che c'è una coda di richieste lente che la media nasconde.

### Evidenza richiesta

Copia il risultato nel file evidenze e scrivi un breve commento sulla differenza tra i tre valori.

---

## Step A9 - Calcola il percentile 95 per endpoint

Esegui:

```kusto
ContainerInstanceLog_CL
| extend parsed = parse_json(Message)
| extend path = tostring(parsed.path), duration = toreal(parsed.duration_ms)
| where isnotempty(path) and duration > 0
| summarize p95 = percentile(duration, 95), avg_lat = avg(duration), calls = count() by path
| sort by p95 desc
```

### Che cosa misura

Combina volume, latenza media e percentile 95 per ogni endpoint.

### Perché è utile

È la query più completa di questa sezione. Ti dà un quadro operativo per endpoint: quante chiamate, quanto è veloce in media, e quanto è lento nel worst case realistico.

### Evidenza richiesta

Copia il risultato nel file evidenze.

---


# LABORATORIO GUIDATO — Parte B

## Step B1 - Modifica `app.py` per aggiungere l'endpoint `/slow`

Apri il file `app.py` del tuo progetto (nella cartella `src/` del LAB09 o equivalente).

Aggiungi questo import all'inizio del file, sotto gli import esistenti:

```python
import random
```

Poi aggiungi questo endpoint, sotto gli endpoint già presenti:

```python
@app.route("/slow")
def slow():
    delay = random.uniform(0.01, 3.0)
    time.sleep(delay)
    return jsonify({"status": "ok", "simulated_delay_ms": int(delay * 1000)})
```

### Che cosa fa questo endpoint

- genera un ritardo casuale tra 10 ms e 3000 ms
- aspetta quel tempo prima di rispondere
- restituisce una risposta con il ritardo simulato

### Perché è utile

Simula un endpoint con problemi di latenza intermittenti, che è uno dei pattern più comuni in produzione.

### Evidenza richiesta

Copia il codice aggiunto nel file evidenze.

---

## Step B2 - Ricostruisci l'immagine Docker

Dalla cartella dove si trova il `Dockerfile`, esegui:

```bash
docker build -t obsapp:v5 .
docker images | grep obsapp
```

### Che cosa fa

Costruisce una nuova versione dell'immagine con l'endpoint `/slow` incluso.

### Checkpoint

Devi vedere `obsapp:v5` nella lista delle immagini locali.

---

## Step B3 - Esegui il push della nuova immagine in ACR

```bash
export ACR_LOGIN_SERVER=$(az acr show --name "$ACR_NAME" --query loginServer -o tsv)
az acr login --name "$ACR_NAME"

docker tag obsapp:v5 "$ACR_LOGIN_SERVER/obsapp:v5"
docker push "$ACR_LOGIN_SERVER/obsapp:v5"
```

### Nota importante

Se la variabile `$ACR_NAME` è vuota, recuperala dal tuo file `azure_env.md` o con:

```bash
az acr list --resource-group "$RG" --query "[0].name" -o tsv
```

### Evidenza richiesta

Annota che il push è avvenuto con successo.

---

## Step B4 - Ridistribuisci il container su ACI

Recupera le credenziali necessarie e ricrea il container con la nuova immagine:

```bash
ACR_USERNAME=$(az acr credential show --name "$ACR_NAME" --query username -o tsv)
ACR_PASSWORD=$(az acr credential show --name "$ACR_NAME" --query "passwords[0].value" -o tsv)

WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group "$RG" \
  --workspace-name "law-observability-francia" \
  --query customerId -o tsv)

WORKSPACE_KEY=$(az monitor log-analytics workspace get-shared-keys \
  --resource-group "$RG" \
  --workspace-name "law-observability-francia" \
  --query primarySharedKey -o tsv)

az container create \
  --resource-group "$RG" \
  --name "obsapp-aci" \
  --image "$ACR_LOGIN_SERVER/obsapp:v5" \
  --ip-address public \
  --ports 8000 \
  --os-type Linux \
  --cpu 1 \
  --memory 1.5 \
  --registry-login-server "$ACR_LOGIN_SERVER" \
  --registry-username "$ACR_USERNAME" \
  --registry-password "$ACR_PASSWORD" \
  --log-analytics-workspace "$WORKSPACE_ID" \
  --log-analytics-workspace-key "$WORKSPACE_KEY"
```

### Perché ricrei il container

Perché devi aggiornare l'immagine da `v4` a `v5` con il nuovo endpoint `/slow`.

### Nota importante

L'IP pubblico potrebbe cambiare. Recuperalo dopo il deploy:

```bash
export ACI_PUBLIC_IP=$(az container show --resource-group "$RG" --name "obsapp-aci" --query ipAddress.ip --output tsv)
echo "$ACI_PUBLIC_IP"
```

### Evidenza richiesta

Annota il nuovo IP pubblico.

---

## Step B5 - Verifica che il nuovo endpoint funzioni

```bash
curl -i "http://$ACI_PUBLIC_IP:8000/health"
curl -i "http://$ACI_PUBLIC_IP:8000/slow"
curl -i "http://$ACI_PUBLIC_IP:8000/slow"
```

### Che cosa osservare

- `/health` deve rispondere normalmente
- `/slow` deve rispondere con latenze diverse ogni volta
- nel body di `/slow` dovresti vedere `simulated_delay_ms` con valori diversi

### Evidenza richiesta

Copia l'output delle due chiamate a `/slow` e nota la differenza di latenza.

---

## Step B6 - Genera traffico misto che simula un incidente

```bash
echo "--- Traffico normale ---"
for i in {1..15}; do curl -s "http://$ACI_PUBLIC_IP:8000/health" > /dev/null; done
for i in {1..15}; do curl -s "http://$ACI_PUBLIC_IP:8000/time" > /dev/null; done

echo "--- Traffico problematico ---"
for i in {1..25}; do curl -s "http://$ACI_PUBLIC_IP:8000/slow" > /dev/null; done

echo "--- Errori ---"
for i in {1..10}; do curl -s "http://$ACI_PUBLIC_IP:8000/nope" > /dev/null; done

echo "Traffico generato. Attendi 3-5 minuti per l'ingestione."
```

### Che cosa simula

Un mix realistico:

- traffico normale su `/health` e `/time`
- un endpoint problematico `/slow` con latenze casuali
- alcuni errori su `/nope`

### Evidenza richiesta

Annota che il traffico è stato generato e l'orario approssimativo.

---

## Step B7 - Investiga: qual è l'endpoint problematico?

Attendi 3-5 minuti, poi apri **Logs** nel workspace e lancia:

```kusto
ContainerInstanceLog_CL
| extend parsed = parse_json(Message)
| extend path = tostring(parsed.path), duration = toreal(parsed.duration_ms)
| where isnotempty(path) and duration > 0
| summarize avg_lat = avg(duration), p95 = percentile(duration, 95), calls = count() by path
| sort by p95 desc
```

### Che cosa devi osservare

L'endpoint `/slow` dovrebbe avere:

- latenza media molto più alta degli altri
- p95 ancora più alto
- un gap evidente rispetto a `/health` e `/time`

### Cosa devi capire

Stai facendo esattamente quello che farebbe un SRE o un DevOps engineer: hai un sistema con un problema e stai usando le query per isolarlo.

### Evidenza richiesta

Copia il risultato e scrivi quale endpoint risulta problematico.

---

## Step B8 - Investiga: come si comporta `/slow` nel tempo?

```kusto
ContainerInstanceLog_CL
| extend parsed = parse_json(Message)
| extend path = tostring(parsed.path), duration = toreal(parsed.duration_ms)
| where path == "/slow" and duration > 0
| summarize avg_lat = avg(duration), p95 = percentile(duration, 95) by bin(TimeGenerated, 5m)
| sort by TimeGenerated asc
| render timechart
```

### Che cosa misura

Mostra il trend della latenza di `/slow` raggruppato per finestre di 5 minuti.

### Evidenza richiesta

Fotografa il grafico oppure copia i dati.

---

## Step B9 - Investiga: quanto impatta `/slow` sul servizio complessivo?

```kusto
ContainerInstanceLog_CL
| extend parsed = parse_json(Message)
| extend path = tostring(parsed.path), duration = toreal(parsed.duration_ms)
| where isnotempty(path) and duration > 0
| summarize
    global_avg = avg(duration),
    global_p95 = percentile(duration, 95),
    slow_pct = round(100.0 * countif(path == "/slow") / count(), 1)
```

### Che cosa misura

- latenza media globale del servizio
- p95 globale
- percentuale di traffico che passa da `/slow`

### Perché è utile

Ti aiuta a capire se il problema di `/slow` sta impattando le metriche complessive del servizio.

### Evidenza richiesta

Copia il risultato nel file evidenze.

---

## Step B10 - Crea un alert sulla latenza

Nel portale Azure, crea una nuova **Alert Rule**:

- **Scope**: workspace `law-observability-francia`
- **Condition**: Custom log search
- **Query**:

```kusto
ContainerInstanceLog_CL
| extend duration = toreal(parse_json(Message).duration_ms)
| where duration > 0
| summarize avg_latency = avg(duration)
```

- **Threshold**: `avg_latency > 500`
- **Evaluation frequency**: 5 min
- **Window size**: 5 min
- **Severity**: 2
- **Action Group**: `ag-observability` (quello già creato nel LAB12)

### Perché questa soglia

500 ms è una soglia ragionevole per un servizio web semplice. Con `/slow` che genera latenze fino a 3000 ms, la media dovrebbe facilmente superare 500 ms.

### Che cosa devi capire

Ora hai **due alert** attivi sul tuo workspace:

- uno sull'**error_rate** (LAB12)
- uno sulla **latenza** (questo step)

Questo è un setup di monitoraggio molto più completo.

### Evidenza richiesta

Esegui uno screenshot della nuova Alert Rule creata.

---

## Step B11 - Scrivi il mini incident report

Crea il file:

```text
docs/incident_report.md
```

Usa questa struttura:

```markdown
# Incident Report — Endpoint /slow

## 1. Descrizione del problema
- Cosa è successo:
- Quando è stato rilevato:
- Quale endpoint è coinvolto:

## 2. Rilevamento
- Come è stato scoperto il problema (query KQL, alert, osservazione manuale):
- Query usata per identificare l'endpoint problematico:

## 3. Dati di conferma
- Latenza media di /slow:
- P95 di /slow:
- Latenza media degli altri endpoint per confronto:
- Percentuale del traffico su /slow:

## 4. Impatto
- L'endpoint problematico ha impattato le metriche globali del servizio? SÌ/NO
- L'alert sull'error_rate (LAB12) è scattato? SÌ/NO
- L'alert sulla latenza è scattato? SÌ/NO

## 5. Causa
- Causa del problema (in questo caso: ritardo simulato con random.uniform):

## 6. Lezione appresa
- Che cosa ho capito sulla differenza tra errori e latenza:
- Perché serve monitorare entrambi:
- Quale query è stata più utile per l'investigazione:
```

---

## Step B12 - Crea il file delle evidenze completo

Crea:

```text
docs/evidence_lab_extra.md
```

Usa questa struttura:

```markdown
# LAB Extra - Evidence

## Parte A — Performance Analysis

### Query: Latenza media per endpoint
[incollare query e risultato]

### Query: Top endpoint per volume
[incollare query e risultato]

### Query: Distribuzione status code
[incollare query e risultato]

### Query: Trend latenza nel tempo
[incollare query e risultato o screenshot grafico]

### Query: Percentile 95 globale
[incollare query e risultato]

### Query: Percentile 95 per endpoint
[incollare query e risultato]

## Parte B — Incident Simulation

### Modifica app.py
- Endpoint aggiunto: /slow
- Codice aggiunto: [incollare snippet]

### Deploy
- Immagine: obsapp:v5
- IP pubblico:
- Container running: SÌ/NO

### Investigazione
- Endpoint problematico identificato:
- Query più utile:
- Risultato chiave:

### Alert sulla latenza
- Creato: SÌ/NO
- Soglia:
- Fired: SÌ/NO

## Sintesi finale
- Che cosa ho imparato sulla differenza tra error_rate e latenza:
- Perché il percentile è più utile della media:
- Che cosa ho capito sul troubleshooting con KQL:
- Che cosa ho capito sulla simulazione di incidenti:
```

---

## Step B13 - Verifica la struttura locale

```bash
ls -R
```

Devi vedere:

```text
docs/
cmdlog_lab_extra.txt
```

e dentro `docs`:

```text
evidence_lab_extra.md
incident_report.md
```

---

## Step B14 - Consegna nel repository Git

```bash
git add docs/evidence_lab_extra.md docs/incident_report.md
git commit -m "[LAB Extra] Performance Analysis + Incident Simulation completato"
git push
```

---

# ═══════════════════════════════════════════════════════════════
# CHECKPOINT E CRITERI DI COMPLETAMENTO
# ═══════════════════════════════════════════════════════════════

# Checkpoint riassuntivi

## Parte A

### Checkpoint A1
Sono state eseguite query KQL per latenza media, top endpoint e distribuzione status code.

### Checkpoint A2
È stato calcolato il percentile 95 della latenza, globale e per endpoint.

### Checkpoint A3
È stato generato un grafico temporale della latenza con `render timechart`.

## Parte B

### Checkpoint B1
L'endpoint `/slow` è stato aggiunto all'app e il container è stato ridistribuito con `obsapp:v5`.

### Checkpoint B2
L'endpoint problematico è stato identificato tramite query KQL.

### Checkpoint B3
È stato creato un alert sulla latenza media.

### Checkpoint B4
L'incident report è stato compilato.

---

# Criteri di completamento

Il LAB Extra è da considerarsi completato se:

- sono state eseguite le query KQL della Parte A
- l'applicazione è stata modificata e ridistribuita
- il traffico misto è stato generato
- l'investigazione KQL ha identificato `/slow` come endpoint problematico
- è stato creato un alert sulla latenza
- i file `evidence_lab_extra.md` e `incident_report.md` sono completi

---

# Che cosa stai imparando davvero

Questo laboratorio ti insegna almeno sei cose importanti.

## 1. L'error_rate non basta

Un servizio può avere zero errori ed essere comunque inutilizzabile per colpa della latenza.

## 2. Il percentile è più onesto della media

La media nasconde le code. Il p95 ti dice la verità sull'esperienza del 5% peggiore degli utenti.

## 3. La distribuzione del traffico è un'informazione operativa

Sapere dove va il traffico ti aiuta a sapere dove guardare quando c'è un problema.

## 4. Modificare l'app e osservare l'impatto è il cuore dell'Observability

Non basta leggere dati. Bisogna saper creare scenari, osservare e ragionare.

## 5. Il troubleshooting è una competenza, non un talento

Si impara seguendo un metodo: generare traffico, interrogare i dati, isolare il problema, documentare.

## 6. L'incident report è una pratica professionale

Scrivere cosa è successo, come lo hai trovato e cosa hai imparato è fondamentale nel mondo DevOps e SRE.

---

# Errori comuni da evitare

## Errore 1 - Fare le query senza aver generato traffico sufficiente

Se hai pochi dati, le query non saranno significative.

## Errore 2 - Dimenticare di aspettare l'ingestione

I log non appaiono istantaneamente nel workspace. Attendi 3-5 minuti dopo aver generato traffico.

## Errore 3 - Guardare solo la media senza calcolare il percentile

La media ti può ingannare. Calcola sempre almeno il p95.

## Errore 4 - Creare l'alert senza aver verificato prima la query

Lancia sempre la query manualmente nel workspace prima di usarla in una Alert Rule.

## Errore 5 - Non compilare l'incident report

L'incident report non è burocrazia. È la prova che sai collegare dati, analisi e decisioni.

---

# Conclusione

Questo laboratorio aggiunge due competenze fondamentali al percorso Observability:

- **analisi di performance con KQL** (Parte A), con query su latenza, percentili, trend temporali e distribuzione traffico
- **incident simulation e troubleshooting** (Parte B), con modifica dell'app, investigazione guidata e documentazione

Dopo aver completato questo LAB Extra devi avere chiaro che:

- l'observability non è solo error_rate
- la latenza è un indicatore critico quanto gli errori
- KQL è uno strumento potente per investigare problemi reali
- il troubleshooting segue un metodo, non l'intuizione
- documentare gli incidenti è una pratica professionale essenziale

