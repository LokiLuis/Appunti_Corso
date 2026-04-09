# LAB14 - Revisione architettura Modulo 2

## Obiettivo del laboratorio

Questo è il laboratorio conclusivo del Modulo 2.

Qui non devi introdurre nuovi strumenti, ma dimostrare di aver capito e saper collegare tutto il percorso svolto:

- deploy del container osservabile
- raccolta e centralizzazione dei log
- persistenza dati in SQL
- query KQL
- alerting
- workbook / dashboard

La traccia del Modulo 2 definisce infatti il LAB14 come laboratorio finale con quattro obiettivi espliciti:

- rappresentare l’architettura completa del Modulo 2
- dimostrare un flusso **end-to-end**: request → log → query → alert/dashboard
- fare analisi scenari (**failure modes**)
- eseguire il **cleanup completo e verificato** 

---

## Durata stimata

**4 ore** 

---

## Cost Control - Gestione dei costi

Il file è molto chiaro: questo è il laboratorio in cui si **chiude** il Modulo 2 e si **distrugge** l’ambiente. A fine LAB14 la pulizia è **obbligatoria**. 

### Pulizia obbligatoria a fine laboratorio

Al termine del LAB14 devi eliminare tutto con:

```bash
az group delete --name rg-observability-lab --yes --no-wait
````

e poi verificare dal portale che:

* nessuna ACI sia più attiva
* nessun ACR sia più attivo
* nessun SQL sia più attivo
* nessun Log Analytics Workspace sia più attivo
* nessuna Alert Rule sia più attiva
* nessun Workbook resti operativo, perché il RG verrà eliminato

---

# PARTE 1 - Concetti fondamentali, teoria e chiusura del Modulo 2

# 1. Perché esiste un laboratorio finale di revisione

Nei laboratori precedenti hai lavorato su singoli pezzi del sistema:

* LAB08: onboarding Azure e osservabilità della piattaforma
* LAB09: ACR + ACI
* LAB10: Azure SQL e SLI via SQL
* LAB11: Log Analytics e KQL
* LAB12: Alerting
* LAB13: Workbooks

Il rischio, a questo punto, è sapere fare ogni cosa separatamente ma non saper più rispondere alla domanda più importante:

> come funziona davvero, da capo a fondo, l’architettura che ho costruito?

Il LAB14 serve proprio a consolidare la visione complessiva.

---

# 2. Che cosa significa verifica end-to-end

Una verifica **end-to-end** significa dimostrare che il flusso completo funziona, non solo una parte isolata.

Nel contesto del Modulo 2 questo flusso è:

```text
request HTTP
  ↓
ACI / container applicativo
  ↓
log applicativi
  ↓
Log Analytics
  ↓
query KQL
  ↓
dati persistiti / SQL
  ↓
alert o dashboard
```

Il file lo descrive in forma sintetica come:

```text
request → log → query → alert/dashboard
```

---

# 3. Perché fare un diagramma architetturale

La traccia richiede di creare un diagramma, anche semplice, e salvarlo come:

```text
docs/architecture_mod2.png
```

oppure `.jpg`

Il diagramma deve includere:

* ACR
* ACI (`obsapp`)
* SQL (`obsdb`)
* Log Analytics
* Alert Rule + Action Group
* Workbook

Questo è importante perché un tecnico non deve solo saper creare risorse. Deve anche saperle **rappresentare**, cioè spiegare a colpo d’occhio che cosa esiste e come le componenti si collegano.

---

# 4. Dove si vede “la verità” del sistema

La traccia del LAB14 chiede esplicitamente che tu sappia spiegare dove si vede la “verità” per:

* stato runtime
* log centralizzati
* dati persistenti

## 4.1 Stato runtime

La “verità” dello stato runtime si osserva soprattutto sulla **ACI**:

* stato Running / stopping / failed
* IP pubblico
* configurazione runtime

## 4.2 Log centralizzati

La “verità” dei log centralizzati si osserva in:

* **Log Analytics Workspace**
* query KQL su `ContainerInstanceLog_CL`

## 4.3 Dati persistenti

La “verità” dei dati persistiti si osserva in:

* **Azure SQL**
* database `obsdb`
* tabella `requests_log`

Capire questa distinzione è una competenza professionale molto importante.

---

# 5. Perché fare analisi scenari

Il file richiede di analizzare quattro scenari nel report, rispondendo sempre a:

* **Sintomo**
* **Impatto**
* **Diagnosi (dove guardi?)**
* **Mitigazione**

Gli scenari richiesti sono:

A) SQL down ma container up
B) Log Analytics non riceve log
C) Alert troppo rumoroso (false positive)
D) Costi crescono inaspettatamente

Questa parte è molto utile perché sposta il focus da “so usare lo strumento” a “so ragionare da tecnico su un problema”.

---

# 6. Perché il cleanup finale è parte del laboratorio

Molti partecipanti tendono a vivere il cleanup come una noia finale.
È sbagliato.

Nel cloud, la pulizia dell’ambiente è parte della competenza tecnica, perché significa:

* evitare costi residui
* evitare risorse orfane
* chiudere correttamente un ciclo di laboratorio o progetto
* dimostrare responsabilità operativa

Nel LAB14 il cleanup non è accessorio: è **obbligatorio**.

---

# 7. Architettura mentale del LAB14

```text
ACR
  ↓
ACI (obsapp)
  ↓
endpoint /health
  ↓
log container
  ↓
Log Analytics
  ↓
query KQL
  ↓
Alert Rule + Action Group
  ↓
Workbook
```

in parallelo:

```text
richieste applicative
  ↓
requests_log
  ↓
Azure SQL
  ↓
query SQL
```

Il LAB14 ti chiede di dimostrare che questi pezzi non sono solo presenti, ma formano un sistema coerente.

---

# 8. Che cosa imparerai davvero

Alla fine del laboratorio dovrai aver capito che:

* costruire il sistema non basta, bisogna saperlo raccontare
* osservabilità vuol dire anche saper localizzare la fonte giusta per ogni verità tecnica
* il troubleshooting richiede metodo
* la gestione dei costi è parte dell’operatività cloud
* cleanup e responsabilità infrastrutturale fanno parte della competenza finale

---

# PARTE 2 - Laboratorio guidato passo-passo

## Prerequisiti

Secondo la traccia del file, per LAB14 servono:

* **LAB09–LAB13 completati**
* risorse ancora presenti nel Resource Group

---

## Step 1 - Setup locale in WSL Ubuntu

Esegui:

```bash
mkdir -p ~/course/lab14 && cd ~/course/lab14
script -a cmdlog_lab14.txt
mkdir -p docs
```

Questa è la sequenza esatta riportata nel file.

### Che cosa fanno i comandi

#### `mkdir -p ~/course/lab14 && cd ~/course/lab14`

Crea la cartella del laboratorio ed entra nella cartella.

#### `script -a cmdlog_lab14.txt`

Registra la sessione del terminale.

#### `mkdir -p docs`

Crea la cartella per diagramma ed evidenze.

---

## Step 2 - Apri il Resource Group e ricostruisci l’architettura

Nel portale Azure:

* cerca **Resource groups**
* apri `rg-observability-lab`

Osserva l’elenco delle risorse presenti.

### Che cosa devi individuare

Il file richiede che il diagramma includa almeno:

* ACR
* ACI (`obsapp`)
* SQL (`obsdb`)
* Log Analytics
* Alert Rule + Action Group
* Workbook

### Obiettivo didattico

Raccogliere la mappa reale delle risorse prima di disegnare il diagramma.

---

## Step 3 - Crea il diagramma architetturale

Crea un diagramma semplice e salvalo come:

```text
docs/architecture_mod2.png
```

oppure:

```text
docs/architecture_mod2.jpg
```

come richiesto dal file.

### Contenuto minimo del diagramma

Deve includere e collegare logicamente:

* ACR → sorgente immagine
* ACI (`obsapp`) → runtime applicativo
* SQL (`obsdb`) → dati persistenti
* Log Analytics → log centralizzati
* Alert Rule + Action Group → monitoraggio proattivo
* Workbook → vista visuale

### Evidenza richiesta

Salva il file e allegalo al report finale.

---

## Step 4 - Verifica end-to-end: endpoint ACI

Da WSL Ubuntu esegui il comando riportato nel file:

```bash
ACI_PUBLIC_IP=$(az container show --resource-group rg-observability-lab --name obsapp-aci --query ipAddress.ip -o tsv)
curl -i "http://$ACI_PUBLIC_IP:8000/health"
```

### Che cosa fa

* recupera l’IP pubblico della ACI
* invia una richiesta HTTP a `/health`

### Che cosa osservare

Devi verificare che l’endpoint risponda.

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 5 - Verifica end-to-end: log centralizzati in Log Analytics

Nel portale Azure vai in:

* **LAW → Logs**

Esegui la query indicata nel file:

```kql
ContainerInstanceLog_CL
| order by TimeGenerated desc
| limit 10
```

### Che cosa osservare

Verifica che ci siano righe recenti di log centralizzati.

### Che cosa devi capire

Questa è la fonte di verità per i log centralizzati del sistema.

### Evidenza richiesta

Esegui uno screenshot o incolla il risultato nel file delle evidenze.

---

## Step 6 - Verifica end-to-end: dati persistenti in SQL

Nel portale Azure vai in:

* **SQL database `obsdb`**
* **Query editor**

Esegui la query indicata nella traccia:

```sql
SELECT COUNT(*) AS total_requests
FROM requests_log;
```

### Che cosa osservare

Verifica che la tabella `requests_log` contenga dati.

### Che cosa devi capire

Questa è la fonte di verità per i dati persistiti del laboratorio.

### Checkpoint #1

La traccia richiede che tu sappia spiegare dove si vede “la verità” per: stato runtime, log centralizzati e dati persistenti.

### Evidenza richiesta

Esegui uno screenshot o incolla il risultato nel file delle evidenze.

---

## Step 7 - Descrivi il flusso end-to-end

Nel report scrivi una breve descrizione del flusso completo:

* da richiesta HTTP
* a log runtime
* a log centralizzati
* a query
* a alert/dashboard

### Obiettivo

Dimostrare che sai collegare i pezzi in un’unica architettura funzionante.

### Evidenza richiesta

Inserisci una descrizione di 5-10 righe nel file `docs/evidence_lab14.md`.

---

## Step 8 - Analisi scenari: A) SQL down ma container up

Nel report rispondi per lo scenario:

**A) SQL down ma container up**

Usa questa struttura, richiesta dal file:

* **Sintomo**
* **Impatto**
* **Diagnosi (dove guardi?)**
* **Mitigazione**

### Spunto didattico

Esempio di ragionamento:

* la ACI risponde ancora `/health`
* SQL non risponde o non restituisce dati
* i log applicativi possono mostrare errori lato persistenza
* mitigazione: ripristino SQL, verifica networking o credenziali

---

## Step 9 - Analisi scenari: B) Log Analytics non riceve log

Nel report rispondi per lo scenario:

**B) Log Analytics non riceve log**

Struttura:

* **Sintomo**
* **Impatto**
* **Diagnosi (dove guardi?)**
* **Mitigazione**

### Spunto didattico

Esempio:

* l’app può essere viva
* ma le query KQL non mostrano dati recenti
* va verificata configurazione diagnostica / raccolta log / workspace corretto

---

## Step 10 - Analisi scenari: C) Alert troppo rumoroso (false positive)

Nel report rispondi per lo scenario:

**C) Alert troppo rumoroso (false positive)**

Struttura:

* **Sintomo**
* **Impatto**
* **Diagnosi (dove guardi?)**
* **Mitigazione**

### Spunto didattico

Esempio:

* alert scatta troppo spesso
* il team smette di considerarlo serio
* va rivista soglia, finestra temporale o query

---

## Step 11 - Analisi scenari: D) Costi crescono inaspettatamente

Nel report rispondi per lo scenario:

**D) Costi crescono inaspettatamente**

Struttura:

* **Sintomo**
* **Impatto**
* **Diagnosi (dove guardi?)**
* **Mitigazione**

### Spunto didattico

Esempio:

* risorse non pulite
* ingestione log eccessiva
* SKU troppo elevati
* mitigazione: review SKU, cleanup, retention e riduzione risorse attive

---

## Step 12 - Crea il file delle evidenze

Il file richiede che `docs/evidence_lab14.md` contenga almeno:

* diagramma
* descrizione flusso end-to-end
* analisi scenari
* checklist finale competenze

Crea:

```text
docs/evidence_lab14.md
```

Usa questa struttura:

````md
# LAB14 - Evidence

## 1. Diagramma architettura
- File allegato: docs/architecture_mod2.png
- Componenti inclusi:
  - ACR
  - ACI
  - SQL
  - Log Analytics
  - Alert Rule + Action Group
  - Workbook

## 2. Verifica end-to-end

### Endpoint ACI
```bash
ACI_PUBLIC_IP=$(az container show --resource-group rg-observability-lab --name obsapp-aci --query ipAddress.ip -o tsv)
curl -i "http://$ACI_PUBLIC_IP:8000/health"
````

* Output:
  [incollare output]

### Log Analytics

```kql
ContainerInstanceLog_CL
| order by TimeGenerated desc
| limit 10
```

* Output / screenshot:
  [incollare o descrivere]

### SQL

```sql
SELECT COUNT(*) AS total_requests
FROM requests_log;
```

* Output / screenshot:
  [incollare o descrivere]

## 3. Dove si vede “la verità”

* Stato runtime (ACI):
* Log centralizzati (Log Analytics):
* Dati persistenti (SQL):

## 4. Descrizione flusso end-to-end

[descrizione 5-10 righe]

## 5. Analisi scenari

### A) SQL down ma container up

* Sintomo:
* Impatto:
* Diagnosi:
* Mitigazione:

### B) Log Analytics non riceve log

* Sintomo:
* Impatto:
* Diagnosi:
* Mitigazione:

### C) Alert troppo rumoroso (false positive)

* Sintomo:
* Impatto:
* Diagnosi:
* Mitigazione:

### D) Costi crescono inaspettatamente

* Sintomo:
* Impatto:
* Diagnosi:
* Mitigazione:

## 6. Checklist finale competenze (SÌ/NO)

* so deployare container:
* so fare query SQL:
* so fare query KQL:
* so impostare alert:
* so costruire workbook:
* so contenere costi:
* so fare cleanup completo:

````

---

## Step 13 - Avvia il cleanup finale

Da WSL Ubuntu o Cloud Shell esegui il comando richiesto dal file: 

```bash
az group delete --name rg-observability-lab --yes --no-wait
````

### Che cosa fa

Avvia l’eliminazione completa del Resource Group e di tutte le risorse contenute.

### Attenzione

Questa volta il cleanup è **obbligatorio** e non va saltato.

---



### Evidenza richiesta

Esegui uno screenshot che mostri il Resource Group assente oppure in deleting.

---

## Step 15 - Consegna finale nel repository Git

La traccia richiede questa consegna:

```bash
git add docs/
git commit -m "[LAB14] Modulo 2 completato"
git push
```

### Che cosa fanno i comandi

* `git add docs/` aggiunge diagramma ed evidenze
* `git commit` salva la chiusura del modulo
* `git push` pubblica tutto nel repository remoto

---

# PARTE 3 - Checkpoint, criteri di completamento e significato professionale

# 1. Checkpoint riassuntivi

## Checkpoint #1

Sai spiegare dove si vede la “verità” per:

* stato runtime (ACI)
* log centralizzati (Log Analytics)
* dati persistenti (SQL)



---

# 2. Criteri di completamento

Secondo il file del Modulo 2, il LAB14 è completato se risultano soddisfatti questi criteri:

* diagramma presente
* flusso end-to-end descritto correttamente
* analisi scenari completata
* Non effettuare cleanup del RG: servirà nel modulo successivo **DevOps**

---

# 3. Che cosa stai imparando davvero

Questo laboratorio ti insegna almeno cinque cose molto importanti.

## 3.1 Saper costruire non basta, bisogna saper spiegare

Il diagramma e il flusso end-to-end servono proprio a questo.

## 3.2 Ogni verità tecnica ha una fonte migliore

Stato runtime, log centralizzati e dati persistiti non si leggono tutti nello stesso posto.

## 3.3 Il troubleshooting richiede metodo

Gli scenari A-D ti obbligano a ragionare per sintomo, impatto, diagnosi e mitigazione.

## 3.4 Il controllo dei costi è una competenza tecnica

Non è burocrazia. È parte della gestione cloud.

---

# 4. Errori comuni da evitare

## Errore 1 - Disegnare un diagramma incompleto

La traccia richiede esplicitamente 6 componenti minime. Se mancano, il diagramma non assolve il suo scopo.

## Errore 2 - Confondere fonte dati e vista

Workbook, SQL e Log Analytics non sono intercambiabili.

## Errore 3 - Fare analisi scenari troppo vaghe

Il file chiede una struttura precisa: sintomo, impatto, diagnosi, mitigazione.



## Errore 4 - Consegnare without verifica

Il checkpoint finale richiede verifica dal portale, non solo esecuzione del comando.

---

# 5. Conclusione

Questo laboratorio chiude il Modulo 2 e consolida tutte le competenze costruite lungo il percorso.

Dopo aver completato il LAB14 devi avere chiaro che:

* l’architettura del Modulo 2 è un sistema coerente, non una somma di esercizi
* sai verificare il flusso end-to-end
* sai ragionare su failure modes realistici
* sai distinguere le fonti di verità osservabili





