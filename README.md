

````md
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
```

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

````

---
