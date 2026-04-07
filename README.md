
* nome Action Group  
* configurazione soglia
* screenshot alert fired / alert history
* breve spiegazione: perché quella soglia e cosa significa

Puoi usare questa struttura completa:

````md
# LAB12 - Evidence

## 1. Action Group
- Nome: ag-observability
- Tipo di azione: alert
- Indirizzo email usato:  luisistrate56@gmail.com

## 2. Alert Rule
- Scope: ![alt text](image.png)
- Workspace: law-observability
- Query KQL:
```kql
ContainerInstanceLog_CL
| extend status = toint(parse_json(Message).status)
| summarize error_rate = todouble(countif(status >= 400)) / count()
```

* Threshold: error_rate > 0.20
* Evaluation frequency: 5 min
* Window size: 5 min
* Severity:3

## 3. Simulazione errori

* Comando usato:

```bash
for i in {1..40}; do
  curl -s "http://$ACI_PUBLIC_IP:8000/nope" > /dev/null
done
```

## 4. Verifica alert

* Fired: SÌ
![alt text](image-1.png)
* Orario osservato:  3:11
* Screenshot allegato: SÌ

## 5. Motivazione soglia

* Perché è stata scelta questa soglia: Il 20% è abbastanza basso da essere superabile con un test pilotato (40 curl a /nope), ma abbastanza alto da non scattare per errori occasionali in produzione.
* Perché un alert troppo rumoroso è un problema: il team inizia a ignorare le notifiche, rendendo invisibili i problemi reali quando si verificano.

## 6. Note finali


* Che cosa ho capito su Alert Rule: È una query KQL che Azure valuta automaticamente a intervalli regolari, trasformando l'osservabilità passiva in monitoraggio proattivo.
* Che cosa ho capito su Action Group: È l'oggetto separato dalla regola che definisce la reazione (email, SMS, webhook), così la stessa azione può essere riutilizzata da più alert.
* Che cosa significa Fired: Significa che Azure ha valutato la query, il risultato ha superato la soglia configurata, e l'alert è passato da stato inattivo ad attivo, triggerando l'Action Group.

````

---
