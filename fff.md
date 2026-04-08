

Usa questa struttura:

````md
# LAB13 - Evidence

## 1. Workbook
- Nome workbook: wb-observability-dashboard
- Workspace usato: law-observability

## 2. Sezione A - Total Requests
- Query usata:
```kql
ContainerInstanceLog_CL
| summarize total=count() by bin(TimeGenerated, 5m)
| render timechart
````

* Screenshot allegato: SÌ/NO
* Osservazioni:

## 3. Sezione B - Error Rate

* Query usata:

```kql
ContainerInstanceLog_CL
| extend status = toint(parse_json(LogEntry_s).status)
| summarize total=count(), errors=countif(status >= 400) by bin(TimeGenerated, 5m)
| extend error_rate = todouble(errors)/total
| render timechart
```

* Screenshot allegato: SÌ/NO
* Osservazioni:

## 4. Sezione C - Top Endpoint

* Query usata:

```kql
ContainerInstanceLog_CL
| extend path = tostring(parse_json(LogEntry_s).path)
| summarize hits=count() by path
| order by hits desc
```

* Screenshot allegato: SÌ/NO
* Osservazioni:

## 5. Confronto SQL

### Query SQL

```sql
SELECT TOP 5 path, COUNT(*) AS hits
FROM requests_log
GROUP BY path
ORDER BY hits DESC;
```

### Output

[incollare output o descrivere risultato]

## 6. Confronto ragionato

Perché i numeri potrebbero differire tra SQL e Log Analytics?

[risposta in 3-6 righe]

## 7. Note finali

* Che cosa ho capito sui Workbooks:
* Che cosa ho capito sul confronto tra fonti dati:
* Quale visualizzazione considero più utile e perché:

````


