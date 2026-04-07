
## Step 14 - Crea il file delle evidenze

Crea il file richiesto nel repository:

```text
docs/evidence_lab12.md
```

Il file condiviso specifica che l’evidenza deve contenere almeno:

* nome Action Group
* configurazione soglia
* screenshot alert fired / alert history
* breve spiegazione: perché quella soglia e cosa significa

Puoi usare questa struttura completa:

```md
# LAB12 - Evidence

## 1. Action Group
- Nome: ag-observability
- Tipo di azione:
- Indirizzo email usato:

## 2. Alert Rule
- Scope:
- Workspace: law-observability
- Query KQL:

```kql
ContainerInstanceLog_CL
| extend status = toint(parse_json(LogEntry_s).status)
| summarize error_rate = todouble(countif(status >= 400)) / count()
```kql

* Threshold: error_rate > 0.20
* Evaluation frequency: 5 min
* Window size: 5 min
* Severity:

## 3. Simulazione errori

* Comando usato:

```bash
for i in {1..40}; do
  curl -s "http://$ACI_PUBLIC_IP:8000/nope" > /dev/null
done
```

## 4. Verifica alert

* Fired: SÌ/NO
* Orario osservato:
* Screenshot allegato: SÌ/NO

## 5. Motivazione soglia

* Perché è stata scelta questa soglia:
* Perché un alert troppo rumoroso è un problema:

## 6. Note finali

* Che cosa ho capito su Alert Rule:
* Che cosa ho capito su Action Group:
* Che cosa significa Fired:
```
```

---

## Step 15 - Consegna nel repository Git

Il file prevede questa consegna: 

```bash
git add docs/evidence_lab12.md
git commit -m "[LAB12] Alerting completato"
git push
```

### Che cosa fanno i comandi

* `git add` aggiunge il file delle evidenze
* `git commit` salva la versione locale
* `git push` pubblica sul repository remoto

---

# PARTE 3 - Checkpoint, criteri di completamento e significato professionale

# 1. Checkpoint riassuntivi

## Checkpoint #1

L’**Action Group** `ag-observability` è stato creato.

## Checkpoint #2

La **Alert Rule** è stata creata ed è abilitata.

## Checkpoint #3

Dopo la simulazione errori, l’alert è stato verificato in stato **Fired** oppure nella alert history, se la soglia è stata superata.

---

# 2. Criteri di completamento

Secondo il file del modulo, il LAB12 è completato se risultano soddisfatti questi criteri:

* Action Group creato
* Alert Rule attiva
* Alert Fired verificato
* Evidence completa

---

# 3. Che cosa stai imparando

Questo laboratorio ti insegna almeno cinque cose importanti.

## 3.1 Una query può diventare una regola operativa

Non resta solo uno strumento di analisi manuale.

## 3.2 L’alerting trasforma l’osservabilità in azione

Passi dall’osservazione alla reazione automatizzata.

## 3.3 La soglia conta quanto la query

Una soglia sbagliata produce alert inutili o rumorosi.

## 3.4 Alert Rule e Action Group sono due oggetti distinti

La regola rileva la condizione.
Il gruppo azioni definisce la reazione.

## 3.5 Fired è una prova operativa

Non basta creare la regola. Devi dimostrare che rileva davvero una condizione reale.

---

# 4. Errori comuni da evitare

## Errore 1 - Creare l’alert senza verificare prima i log

Se i log non arrivano, la regola non potrà funzionare bene.

## Errore 2 - Impostare lo scope sbagliato

Il file richiede il workspace `law-observability`. Se sbagli scope, l’alert guarda nel posto sbagliato.

## Errore 3 - Usare una soglia poco testabile

Nel laboratorio la soglia deve poter essere superata con i dati generati.

## Errore 4 - Aspettarsi alert immediati

Il file stesso chiede di attendere 5-10 minuti. Quindi no, non è un interruttore magico.

## Errore 5 - Documentare male la motivazione della soglia

Il laboratorio richiede esplicitamente di spiegare perché quella soglia e cosa significa, evitando alert rumorosi.

---

# 5. Conclusione

Questo laboratorio introduce il **monitoraggio proattivo** nel Modulo 2.

Dopo aver completato il LAB12 devi avere chiaro che:

* una query KQL può diventare una condizione di allerta
* Azure Monitor può valutare in autonomia una regola
* un Action Group definisce la reazione
* lo stato **Fired** dimostra che la condizione è stata davvero rilevata
* alerting e observability non sono separati: l’alerting è uno degli esiti più operativi dell’observability

Questo prepara il **LAB13**, che nel file è definito come laboratorio sui **Workbooks**, con dashboard su total requests, error rate e top endpoint, oltre al confronto tra SQL e Log Analytics.
