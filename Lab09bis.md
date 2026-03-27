# LAB09bis - Monitoraggio dal portale Azure partendo dall'app.py già distribuita

## Obiettivo del laboratorio

In questo laboratorio imparerai a monitorare dal **portale Azure** e dalla **CLI locale** le risorse che hai già creato nel LAB09, collegando in modo esplicito tre elementi che spesso vengono confusi:

- **codice dell'applicazione** (`app.py`)
- **risorsa cloud che esegue l'app** (Azure Container Instance)
- **strumenti di osservazione** disponibili nel portale Azure

Questo laboratorio non serve a creare nuove risorse da zero.

Serve invece a farti capire come si passa da una semplice applicazione Python containerizzata a un servizio osservabile, e come si leggono i segnali disponibili quando l'app gira davvero in Azure.

---

## Collocazione nel percorso

- nel **LAB09** hai già costruito l'immagine, pubblicato il container in ACR e fatto il deploy in ACI
- nel **LAB09bis** impari a osservare in modo strutturato ciò che hai già distribuito
- nel **LAB10** consoliderai concetti Azure SQL
- nel **LAB11** inizierai ad andare oltre, usando strumenti di osservabilità più evoluti

In altre parole:

- **LAB09** = pubblicazione ed esecuzione dell'app
- **LAB09bis** = osservazione e monitoraggio guidato di ciò che hai pubblicato

---

## Durata indicativa

**2 ore e 30 minuti - 3 ore**

---

## Modalità operativa

Per questo laboratorio userai due ambienti distinti, con ruoli chiari.

### WSL Ubuntu locale

Userai WSL Ubuntu per:

- leggere il file `app.py`
- consultare il file `azure_env.md`
- eseguire `curl`
- usare Azure CLI per recuperare log e stato
- preparare le evidenze
- registrare la sessione terminale

### Portale Azure

Userai il portale per:

- osservare il Resource Group
- aprire ACR
- aprire ACI
- consultare Overview, Activity Log, Monitoring, Logs e altre sezioni disponibili

### Regola pratica

In questo laboratorio **non devi creare nuove immagini** e **non devi rifare il deploy** se hai già completato correttamente il LAB09.

Qui il focus è:

1. capire cosa fa `app.py`
2. verificare che l'app sia realmente raggiungibile in Azure
3. collegare il comportamento dell'app ai segnali osservabili nel portale e nella CLI

---

## Risorse che devi già avere disponibili

Prima di iniziare, verifica di avere già queste risorse o questi file.

### Risorse Azure

- Resource Group del modulo, ad esempio `rg-observability-lab`
- Azure Container Registry (ACR)
- Azure Container Instance (ACI) con IP pubblico

### File locali del repository

- file `src/app.py`
- file `docs/azure_env.md`
- file `docs/evidence_lab09.md` del laboratorio precedente, se già compilato

Se non hai ancora queste risorse o questi file, fermati e completa prima il LAB09.

---

# PARTE 1 - Concetti fondamentali

# 1. Perché in questo laboratorio parti da `app.py`

Molti partecipanti, quando aprono il portale Azure, guardano la risorsa ma non ricordano più che cosa dovrebbe fare davvero l'applicazione.

Questo è un errore importante.

Non puoi monitorare bene un servizio se non sai:

- quali endpoint espone
- quali risposte dovrebbe dare
- quali log dovrebbe produrre
- quali segnali osservabili sono attesi

Per questo motivo in questo laboratorio riparti da `app.py`.

Il codice applicativo è il punto da cui derivano:

- gli endpoint HTTP che testerai
- gli header che cercherai
- i campi che vorrai ritrovare nei log

In forma semplice:

```text
app.py
  ↓
endpoint /health, /time, /nope
  ↓
risposte HTTP
  ↓
request_id / status / duration_ms
  ↓
log applicativi
  ↓
osservazione nel portale e via CLI
```

---

# 2. Che cosa significa monitorare una risorsa Azure in modo serio

Monitorare una risorsa Azure non significa limitarsi a vedere che la risorsa esiste nel portale.

Significa saper distinguere almeno tre livelli diversi.

## 2.1 Livello 1 - Stato della risorsa

Qui guardi se la risorsa:

- esiste
- è in esecuzione
- è raggiungibile
- ha un IP pubblico
- è configurata come ti aspetti

Esempi:

- ACI è `Running`
- ACR esiste
- il Resource Group contiene le risorse previste

## 2.2 Livello 2 - Eventi amministrativi della piattaforma

Qui guardi che cosa è successo dal punto di vista Azure.

Esempi:

- una risorsa è stata creata
- una risorsa è stata modificata
- una risorsa è stata eliminata
- un deploy è stato eseguito

Queste informazioni si leggono nell'**Activity Log**.

## 2.3 Livello 3 - Comportamento dell'applicazione

Qui guardi cosa fa davvero il servizio che gira nella risorsa.

Esempi:

- `/health` restituisce `200`
- `/time` restituisce una risposta valida
- `/nope` restituisce un errore controllato
- i log mostrano `request_id`, `status`, `duration_ms`

Questo è il livello più vicino all'observability applicativa.

---

# 3. Differenza tra portale Azure e log applicativi

Questa distinzione deve esserti molto chiara.

## Portale Azure

Ti aiuta a osservare:

- stato risorsa
- configurazione
- eventi amministrativi
- alcune metriche
- diagnostica disponibile

## Log applicativi

Ti aiutano a osservare:

- richieste ricevute
- status code
- durata delle richieste
- correlation id o request id
- comportamento reale del servizio

### Regola mentale

```text
Portale Azure = osservazione della piattaforma e del runtime della risorsa
Log applicativi = osservazione del comportamento del servizio
```

---

# 4. Perché questo laboratorio è importante nel percorso Observability

Se non impari a collegare il codice applicativo ai segnali che vedi in Azure, nei laboratori successivi rischi di fare confusione su tutto.

Per esempio:

- apri Overview ma non sai se l'app sta davvero rispondendo
- apri Activity Log ma pensi che mostri i dettagli delle richieste HTTP
- guardi un endpoint che fallisce e non sai se il problema è del codice o della risorsa
- leggi un log ma non capisci a quale endpoint o richiesta si riferisca

Questo laboratorio serve proprio a costruire il ponte tra:

- codice
- deploy
- osservazione

---

# 5. I segnali osservabili che cercherai

In questo laboratorio cercherai segnali osservabili molto concreti.

## 5.1 Endpoint HTTP

Cercherai:

- `/health`
- `/time`
- `/nope`

## 5.2 Codici di stato

Controllerai almeno:

- `200`
- `404`

## 5.3 Header di correlazione

Cercherai l'header:

- `X-Request-Id`

## 5.4 Campi nei log

Cercherai campi come:

- `request_id`
- `status`
- `duration_ms`

Questi segnali non sono decorazione. Sono la base minima per iniziare un ragionamento serio di observability.

---

# PARTE 2 - Attività pratiche guidate

## Step 1 - Prepara la cartella del laboratorio e avvia il log dei comandi

Apri WSL Ubuntu ed esegui:

```bash
mkdir -p ~/corso_obs/NOME-REPOSITORY/labs/lab09bis
cd ~/corso_obs/NOME-REPOSITORY/labs/lab09bis
script -a cmdlog_lab09bis.txt
mkdir -p docs
```

### Che cosa fanno questi comandi

#### `mkdir -p ~/corso_obs/NOME-REPOSITORY/labs/lab09bis`

Crea la cartella del laboratorio.

#### `cd ~/corso_obs/NOME-REPOSITORY/labs/lab09bis`

Ti sposta dentro la cartella del laboratorio.

#### `script -a cmdlog_lab09bis.txt`

Registra tutto ciò che fai nel terminale, così potrai conservare traccia dei comandi eseguiti.

#### `mkdir -p docs`

Crea la cartella locale per le evidenze del laboratorio.

---

## Step 2 - Leggi il file `app.py` del LAB09

Ora devi tornare alla cartella del LAB09, dove è già presente il codice dell'applicazione.

Esegui uno di questi comandi, in base a come è organizzato il tuo repository.

### Se ti trovi nella cartella `lab09bis`

```bash
cat ../lab09/src/app.py
```

### Se vuoi consultarlo con numeri di riga

```bash
nl -ba ../lab09/src/app.py | less
```

### Se vuoi cercare solo i punti più importanti

```bash
grep -nE "health|time|nope|request|duration|status|X-Request-Id" ../lab09/src/app.py
```

### Che cosa devi osservare

Nel file `app.py` devi cercare almeno questi aspetti:

- quali endpoint sono gestiti
- dove viene restituito `/health`
- dove viene restituito `/time`
- che cosa accade per un endpoint inesistente come `/nope`
- dove viene generato o gestito un `request_id`
- dove vengono scritti i log
- se nei log compaiono `status` e `duration_ms`

### Perché lo fai

Stai leggendo il comportamento atteso dell'applicazione prima di andare nel portale.

Questo è importante perché poi, quando testerai gli endpoint e leggerai i log, saprai che cosa aspettarti e potrai confrontare teoria e realtà.

### Evidenza richiesta

Nel file delle evidenze annota brevemente:

- endpoint individuati
- campi osservabili individuati nel codice
- eventuale presenza di `X-Request-Id`

---

## Step 3 - Verifica il file ambiente `azure_env.md`

Esegui:

```bash
cat ../lab09/docs/azure_env.md
```

Se il tuo repository usa un percorso diverso ma coerente, apri quello effettivo.

### Che cosa devi controllare

Verifica di avere almeno questi valori:

- `RG`
- `LOCATION`
- `ACR_NAME`
- `ACR_LOGIN_SERVER`
- `ACI_NAME`
- `ACI_PUBLIC_IP`

### Perché lo fai

Per monitorare bene una risorsa devi sapere con precisione come si chiama.

Azure non ti aiuta a ricordare i nomi. Ti aiuta soprattutto a dimenticarli nel momento peggiore.

### Evidenza richiesta

Annota nel file delle evidenze:

- nome del Resource Group
- nome ACR
- nome ACI
- IP pubblico ACI

---

## Step 4 - Esporta le variabili utili nella shell

Per lavorare in modo più comodo, esporta alcune variabili nella sessione WSL.

Sostituisci i valori reali con quelli letti dal tuo `azure_env.md`.

```bash
export RG="rg-observability-lab"
export ACR_NAME="NOME_REALE_ACR"
export ACR_LOGIN_SERVER="NOME_REALE_LOGIN_SERVER"
export ACI_NAME="obsapp-aci"
export ACI_PUBLIC_IP="IP_REALE_ACI"
```

### Verifica

```bash
echo "$RG"
echo "$ACR_NAME"
echo "$ACR_LOGIN_SERVER"
echo "$ACI_NAME"
echo "$ACI_PUBLIC_IP"
```

### Perché lo fai

Queste variabili semplificano tutti i comandi dei passaggi successivi e riducono la probabilità di errori di digitazione.

---

## Step 5 - Apri il portale Azure

Apri il browser e vai su:

```text
https://portal.azure.com
```

Accedi con il tuo account Azure.

### Obiettivo dello step

Prepararti a usare il portale non come una semplice interfaccia grafica, ma come strumento di osservazione delle risorse già create nel LAB09.

---

## Step 6 - Osserva il Resource Group dal portale

Nella barra di ricerca del portale cerca:

```text
Resource groups
```

Apri il tuo Resource Group, ad esempio:

```text
rg-observability-lab
```

### Che cosa osservare nella schermata Overview

Controlla:

- nome
- subscription
- region
- numero di risorse contenute
- eventuali tag
- menu laterale disponibile

### Cosa devi capire

Il Resource Group è il contenitore logico dell'intero scenario che hai costruito con il LAB09.

Non esegue l'applicazione, ma ti permette di vedere insieme le risorse coinvolte.

### Evidenza richiesta

Esegui uno screenshot della schermata **Overview** del Resource Group.

---

## Step 7 - Esplora l'elenco delle risorse contenute nel Resource Group

Nel Resource Group apri la sezione:

```text
Resources
```

### Che cosa osservare

Verifica la presenza di risorse come:

- ACR
- ACI
- eventuale IP pubblico
- eventuali altre risorse del modulo

### Cosa devi capire

Questa vista ti permette di rispondere a una domanda molto semplice ma fondamentale:

> quali oggetti cloud esistono davvero in questo scenario?

### Evidenza richiesta

Esegui uno screenshot dell'elenco delle risorse.

---

## Step 8 - Apri Activity Log del Resource Group

Nel menu del Resource Group cerca:

```text
Activity log
```

Aprilo.

### Che cosa osservare

Individua eventi come:

- creazione del Resource Group
- creazione ACR
- creazione ACI
- eventuali aggiornamenti
- eventuali modifiche di configurazione

### Campi da annotare

Per almeno due eventi annota:

- timestamp
- operation name
- status
- risorsa coinvolta

### Cosa devi capire

L'Activity Log racconta la storia amministrativa della piattaforma Azure.

Non ti mostra il dettaglio delle richieste HTTP fatte alla tua app.

### Evidenza richiesta

Esegui uno screenshot dell'Activity Log filtrato sul Resource Group.

---

## Step 9 - Apri Azure Container Registry e osserva la Overview

Dal Resource Group clicca sul tuo **Azure Container Registry**.

### Che cosa osservare nella Overview

Controlla:

- nome ACR
- Resource Group
- region
- SKU
- login server
- menu laterale disponibile

### Cosa devi capire

ACR non esegue l'applicazione.

Conserva l'immagine container che hai già pubblicato nel LAB09.

### Evidenza richiesta

Esegui uno screenshot della **Overview** di ACR.

---

## Step 10 - Verifica repository e tag nel portale ACR

Dentro ACR apri la sezione:

```text
Repositories
```

### Che cosa osservare

Verifica che sia presente il repository:

```text
obsapp
```

Aprilo e controlla la presenza del tag:

```text
v2
```

### Cosa devi capire

Questa vista conferma che l'immagine costruita a partire dalle risorse del LAB09 è realmente disponibile nel registry Azure.

### Evidenza richiesta

Esegui uno screenshot con repository e tag visibili.

---

## Step 11 - Apri la risorsa ACI e osserva la Overview

Dal Resource Group apri la tua **Azure Container Instance**.

### Che cosa osservare

Controlla almeno:

- nome
- stato
- Resource Group
- IP pubblico
- porta esposta
- immagine usata
- eventuale restart policy, se visibile

### Cosa devi capire

Questa è la vista principale del runtime della tua applicazione dal punto di vista Azure.

Qui stai osservando la **risorsa cloud che ospita l'app**, non ancora il comportamento interno del codice.

### Evidenza richiesta

Esegui uno screenshot della **Overview** di ACI.

---

## Step 12 - Verifica da CLI lo stato runtime della ACI

Torna in WSL ed esegui:

```bash
az container show --resource-group "$RG" --name "$ACI_NAME" --query instanceView.state -o tsv
```

### Che cosa osservare

L'output dovrebbe essere simile a:

```text
Running
```

### Cosa devi capire

Qui stai controllando il livello **stato della risorsa**.

Questo non basta ancora per dire che l'applicazione funziona correttamente, ma è il primo controllo da fare.

### Evidenza richiesta

Copia l'output nel file delle evidenze.

---

## Step 13 - Richiama l'app dal browser

Ora usa l'IP pubblico che hai già ottenuto nel LAB09 e apri nel browser questi indirizzi:

```text
http://<ACI_PUBLIC_IP>:8000/health
```

```text
http://<ACI_PUBLIC_IP>:8000/time
```

```text
http://<ACI_PUBLIC_IP>:8000/nope
```

Sostituisci `<ACI_PUBLIC_IP>` con il tuo IP reale.

### Che cosa osservare

Verifica:

- risposta corretta sugli endpoint validi
- comportamento di errore controllato sull'endpoint inesistente

### Cosa devi capire

Qui stai verificando il comportamento dell'applicazione dal punto di vista di un client esterno.

### Evidenza richiesta

Esegui almeno uno screenshot di una risposta valida e annota il comportamento dell'endpoint `/nope`.

---

## Step 14 - Testa gli stessi endpoint da terminale con `curl`

In WSL esegui:

```bash
curl -i "http://$ACI_PUBLIC_IP:8000/health"
curl -i "http://$ACI_PUBLIC_IP:8000/time"
curl -i "http://$ACI_PUBLIC_IP:8000/nope"
```

### Che cosa osservare

Controlla:

- `HTTP/1.1 200 OK` sugli endpoint validi
- `HTTP/1.1 404` o equivalente per `/nope`
- presenza dell'header `X-Request-Id`, se implementato nell'app

### Perché questo step è importante

Nel browser vedi il comportamento in modo semplice.

Con `curl -i` vedi anche gli header HTTP e puoi osservare dettagli tecnici molto più utili per l'analisi.

### Evidenza richiesta

Copia nel file delle evidenze:

- output completo di `/health`
- output completo di `/time`
- output completo di `/nope`
- presenza o assenza di `X-Request-Id`

---

## Step 15 - Consulta i log del container dal portale Azure

Apri di nuovo la risorsa ACI nel portale e cerca una sezione come:

- **Containers**
- **Logs**
- **Log**
- o una sezione equivalente disponibile nella UI

### Che cosa osservare

Controlla se compaiono righe di log con informazioni come:

- richieste ricevute
- endpoint chiamati
- status
- durata
- eventuale request id

### Cosa devi capire

Qui stai osservando i segnali emessi dall'applicazione mentre gira nel runtime cloud.

### Collegamento con `app.py`

Confronta quello che vedi nei log con quanto avevi osservato prima leggendo `app.py`.

Chiediti:

- i campi attesi compaiono davvero?
- i path chiamati compaiono davvero?
- l'errore di `/nope` è coerente con il codice e con il test HTTP?

### Evidenza richiesta

Esegui uno screenshot dei log visibili dal portale, se disponibili.

---

## Step 16 - Recupera gli stessi log da CLI per confronto

In WSL esegui:

```bash
az container logs --resource-group "$RG" --name "$ACI_NAME"
```

### Che cosa osservare

Nei log cerca campi come:

- `request_id`
- `status`
- `duration_ms`
- endpoint chiamati

### Cosa devi capire

Portale e CLI ti mostrano due modi diversi di leggere lo stesso fenomeno osservabile.

Il portale è più visivo.
La CLI è più tecnica e più semplice da copiare nelle evidenze.

### Evidenza richiesta

Copia un estratto dei log nel file evidenze.

---

## Step 17 - Collega i log al comportamento di `app.py`

Torna al file `app.py` e rileggi i punti in cui:

- vengono gestiti gli endpoint
- viene costruita la risposta HTTP
- vengono generati i log

Puoi usare ancora:

```bash
grep -nE "health|time|request|duration|status|nope" ../lab09/src/app.py
```

### Attività da svolgere

Compila una piccola tabella mentale o scritta come questa:

| Elemento | Dove lo vedo nel codice | Dove lo vedo a runtime |
|---|---|---|
| `/health` | `app.py` | browser / curl / log |
| `/time` | `app.py` | browser / curl / log |
| `/nope` | comportamento non gestito o errore | browser / curl / log |
| `request_id` | `app.py` | header o log |
| `status` | `app.py` o log formatting | log |
| `duration_ms` | `app.py` o logging | log |

### Perché lo fai

Questo è il cuore del laboratorio.

Non ti basta vedere che il sistema “sembra funzionare”.
Devi capire **come il codice si manifesta nei segnali osservabili**.

---

## Step 18 - Cerca eventuali metriche disponibili sulla ACI

Nel portale, dentro la risorsa ACI, cerca:

```text
Metrics
```

oppure una sezione equivalente sotto **Monitoring**.

### Che cosa osservare

Verifica:

- se esistono metriche disponibili
- quali nomi hanno
- se sono presenti grafici temporali
- se puoi cambiare intervallo temporale o aggregazione

### Cosa devi capire

Non tutte le risorse Azure offrono lo stesso livello di metriche.

Anche vedere una sezione povera o quasi vuota è utile, perché ti insegna che il monitoraggio cambia da risorsa a risorsa.

### Evidenza richiesta

Esegui uno screenshot della sezione metrics, anche se contiene pochi dati.

---

## Step 19 - Esplora “Diagnose and solve problems” se disponibile

Sempre nella ACI, nell'ACR o in un'altra risorsa del modulo, cerca:

```text
Diagnose and solve problems
```

### Che cosa osservare

Verifica se sono presenti:

- categorie di problemi
- controlli automatici
- suggerimenti diagnostici
- analisi guidate

### Cosa devi capire

Questa sezione è utile soprattutto quando non sai ancora dove guardare per un problema.

### Evidenza richiesta

Esegui uno screenshot della sezione, se presente.

---

## Step 20 - Apri Azure Monitor e osserva il monitoraggio globale

Nel portale cerca:

```text
Monitor
```

Apri il servizio.

### Che cosa osservare

Esplora almeno queste sezioni, se disponibili nel tuo ambiente:

- Activity Log
- Alerts
- Metrics
- Workbooks
- eventuali Logs

### Cosa devi capire

Azure Monitor è il punto di raccolta dei meccanismi di monitoraggio, ma in questa fase del percorso non tutto sarà necessariamente popolato allo stesso modo.

### Evidenza richiesta

Esegui uno screenshot della home di Azure Monitor o di una sezione significativa.

---

## Step 21 - Confronta tre livelli di osservazione

A questo punto devi confrontare tre livelli distinti.

### Livello 1 - Stato risorsa

Lo hai visto in:

- Overview del Resource Group
- Overview di ACR
- Overview di ACI
- `az container show`

### Livello 2 - Eventi amministrativi

Lo hai visto in:

- Activity Log

### Livello 3 - Comportamento applicativo

Lo hai visto in:

- `app.py`
- browser
- `curl`
- log ACI

### Attività da svolgere

Scrivi nel file evidenze una breve sintesi spiegando la differenza tra questi tre livelli.

---

## Step 22 - Crea il file delle evidenze

Crea:

```text
docs/evidence_lab09bis.md
```

Usa una struttura come questa:

```md
# LAB09bis - Evidence

## 1. app.py
- Endpoint individuati:
- Campi osservabili individuati:
- Header di correlazione previsto: SÌ/NO
- Note:

## 2. File ambiente
- RG:
- ACR_NAME:
- ACR_LOGIN_SERVER:
- ACI_NAME:
- ACI_PUBLIC_IP:

## 3. Resource Group
- Numero risorse osservate:
- Note:

## 4. Activity Log
### Evento 1
- Timestamp:
- Operation name:
- Status:
- Risorsa coinvolta:

### Evento 2
- Timestamp:
- Operation name:
- Status:
- Risorsa coinvolta:

## 5. ACR
- Repository osservati:
- Tag osservati:
- Note:

## 6. ACI
- Stato:
- IP pubblico:
- Porta:
- Immagine osservata:

## 7. Test applicativi
### /health
[incollare output o descrizione]

### /time
[incollare output o descrizione]

### /nope
[incollare output o descrizione]

## 8. Header HTTP
- X-Request-Id presente: SÌ/NO
- Note:

## 9. Log ACI
[incollare estratto log]

## 10. Metrics / Monitoring
- Sezione metrics disponibile: SÌ/NO
- Note:
- Diagnose and solve problems disponibile: SÌ/NO
- Note:

## 11. Sintesi finale
- Differenza tra stato risorsa e comportamento applicativo:
- Che cosa mi dice Activity Log:
- Che cosa ho ritrovato nei log che avevo già visto o previsto in app.py:
- Che cosa ho capito sul monitoraggio dal portale Azure:
```

---

## Step 23 - Verifica la struttura locale del laboratorio

Esegui:

```bash
ls -R
```

### Che cosa devi vedere

Almeno:

```text
cmdlog_lab09bis.txt
docs/
```

E dentro `docs`:

```text
evidence_lab09bis.md
```

---

## Step 24 - Consegna nel repository Git

Esegui:

```bash
git add docs/evidence_lab09bis.md
git commit -m "[LAB09bis] Monitoraggio dal portale Azure completato"
git push
```

### Che cosa fanno i comandi

- `git add` prepara il file per il commit
- `git commit` salva una nuova versione
- `git push` invia tutto al repository remoto

---

# PARTE 3 - Checkpoint e criteri di completamento

## Checkpoint #1

Hai letto `app.py` e hai individuato endpoint e segnali osservabili attesi.

## Checkpoint #2

Hai consultato `azure_env.md` e identificato correttamente le risorse del laboratorio.

## Checkpoint #3

Hai osservato Resource Group, ACR e ACI dal portale Azure.

## Checkpoint #4

Hai consultato Activity Log e annotato almeno due eventi significativi.

## Checkpoint #5

Hai testato l'applicazione sia da browser sia da `curl`.

## Checkpoint #6

Hai recuperato e confrontato i log ACI dal portale e dalla CLI.

## Checkpoint #7

Hai collegato i segnali osservati ai comportamenti definiti in `app.py`.

---

# Criteri di completamento

Il laboratorio è da considerarsi completato se:

- hai letto `app.py` in modo consapevole
- hai identificato le risorse Azure corrette del LAB09
- hai distinto chiaramente stato risorsa, eventi amministrativi e comportamento applicativo
- hai verificato il comportamento dell'app via browser e via `curl`
- hai confrontato codice, risposta HTTP e log runtime
- hai compilato correttamente `docs/evidence_lab09bis.md`

---

# Errori comuni da evitare

## Errore 1 - Guardare il portale senza rileggere `app.py`

Se non ricordi cosa dovrebbe fare l'app, il monitoraggio diventa superficiale.

## Errore 2 - Confondere Activity Log con log applicativi

Activity Log racconta gli eventi amministrativi della piattaforma Azure.

I log applicativi raccontano il comportamento del servizio.

## Errore 3 - Pensare che `Running` significhi automaticamente “app perfettamente funzionante”

No.

`Running` significa che la risorsa è in esecuzione. Non garantisce da solo che gli endpoint rispondano correttamente.

## Errore 4 - Guardare solo il browser

Il browser è comodo, ma per un'osservazione tecnica servono anche `curl`, log e stato risorsa.

## Errore 5 - Non collegare i segnali runtime al codice

Se vedi un `404`, un `request_id` o una durata nei log, devi chiederti da dove arrivino nel codice.

---

# Conclusione

Questo laboratorio ti insegna a monitorare una soluzione Azure non come spettatore passivo del portale, ma come persona che sa collegare:

- **codice applicativo**
- **deploy cloud**
- **comportamento HTTP**
- **log runtime**
- **strumenti di osservazione del portale Azure**

Dopo aver completato il LAB09bis, devi avere chiaro che:

- il portale Azure mostra livelli diversi di informazione
- lo stato della risorsa non coincide automaticamente con il comportamento dell'applicazione
- `app.py` spiega che cosa dovrebbe accadere
- browser, `curl` e log mostrano che cosa accade davvero
- osservabilità significa proprio collegare questi livelli in modo consapevole
