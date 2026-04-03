## Obiettivi
### Verificare che si siano compresi i **concetti fondamentali** di SRE e Observability già affrontati nel corso.
### Acronimi fondamentali che è necessario conoscere

#### Elenco minimo obbligatorio
- **SRE** = Site Reliability Engineering
- **SLI** = Service Level Indicator
- **SLO** = Service Level Objective
- **SLA** = Service Level Agreement
- **MTTD** = Mean Time To Detect
- **MTTR** = Mean Time To Restore / Recover

#### Altri acronimi utili già da conoscere
- **KPI** = Key Performance Indicator
- **CPU** = Central Processing Unit
- **RAM** = Random Access Memory
- **HTTP** = HyperText Transfer Protocol
- **TCP/IP** = Transmission Control Protocol / Internet Protocol

# Elenco Domande

## Domanda 1
**Che cosa significa SRE?** 

SRE significa **Site Reliability Engineering**.  
È una disciplina che applica principi di ingegneria del software alla gestione operativa dei servizi IT per renderli affidabili, misurabili e sostenibili.

## Domanda 2
**Qual è lo scopo principale dell’SRE?**

Garantire la **reliability** dei servizi, cioè mantenerli affidabili e funzionanti nel tempo, bilanciando stabilità, qualità operativa e velocità di cambiamento.

## Domanda 3
**Che cosa si intende per reliability?**

La reliability è la capacità di un sistema di funzionare correttamente nel tempo, in modo prevedibile e con un livello di qualità coerente con quanto atteso.

## Domanda 4
**Che differenza c’è tra amministrazione di sistema tradizionale e approccio SRE?**

L’amministrazione tradizionale tende a essere più manuale e reattiva.  
L’approccio SRE punta invece a:
- misurare
- automatizzare
- ridurre il lavoro ripetitivo
- migliorare l’affidabilità con un approccio ingegneristico

## Domanda 5
**Che cos’è la Observability?**

La Observability è la capacità di comprendere lo stato interno di un sistema a partire dai segnali che il sistema produce all’esterno, come log, metriche e trace.

## Domanda 6
**Che differenza c’è tra monitoring e observability?**

Il monitoring controlla condizioni e problemi già noti, per esempio CPU alta o servizio non disponibile.  
La observability aiuta invece a capire anche problemi non previsti, esplorando i dati per formulare ipotesi e individuare cause.

## Domanda 7
**Quali sono i tre pilastri classici della observability?**

I tre pilastri classici sono:
- log
- metriche
- trace

## Domanda 8
**Che cosa sono i log?**

I log sono registrazioni di eventi avvenuti nel sistema o nell’applicazione.  
Servono a capire cosa è successo, quando è successo e in quale contesto.

## Domanda 9
**Che cosa sono le metriche?**

Le metriche sono misure numeriche raccolte nel tempo, per esempio:
- numero di richieste
- tempo di risposta
- utilizzo CPU
- memoria
- tasso di errore

Servono a osservare andamento, trend e anomalie.

## Domanda 10
**Che cosa sono le trace?**

Le trace rappresentano il percorso di una richiesta attraverso uno o più componenti di un sistema distribuito.  
Servono a capire dove passa una richiesta e dove si verificano ritardi o errori.

## Domanda 11
**Che cos’è la telemetria?**

La telemetria è l’insieme dei dati raccolti da sistemi e applicazioni per osservarne stato e comportamento.  
Comprende log, metriche, trace e altri segnali utili.

## Domanda 12
**Perché log, metriche e trace sono complementari?**

Perché rispondono a domande diverse:
- le metriche mostrano che c’è un problema o una variazione
- i log aiutano a capire che cosa è accaduto
- le trace mostrano dove il problema si manifesta nel flusso della richiesta

## Domanda 13
**Che cos’è un incidente in ambito SRE?**

Un incidente è un evento che degrada o interrompe il servizio rispetto al livello atteso.

## Domanda 14
**Che cos’è un alert?**

Un alert è una segnalazione automatica che avvisa quando una metrica o una condizione indica un possibile problema.

## Domanda 15
**Quando un alert è utile?**

Un alert è utile quando è:
- chiaro
- azionabile
- collegato a un problema reale
- rilevante per il servizio

## Domanda 16
**Che cos’è l’alert fatigue?**

È la situazione in cui arrivano troppi alert inutili o ripetitivi e il team finisce per ignorarli o considerarli meno importanti.

## Domanda 17
**Che cos’è un dashboard?**

Un dashboard è una vista sintetica di metriche e indicatori utile per monitorare lo stato di un sistema e individuare rapidamente problemi o trend.

## Domanda 18
**Che cos’è un KPI e in cosa differisce da una metrica tecnica?**

Un KPI è un indicatore chiave di prestazione orientato agli obiettivi del servizio o del business.  
Una metrica tecnica misura un aspetto del sistema.  
Una metrica può diventare KPI se è davvero significativa rispetto agli obiettivi

## Domanda 19
**Che cosa significa SLA?**

SLA significa **Service Level Agreement**.  
È un accordo formale sul livello di servizio garantito.

## Domanda 20
**Che cosa significa SLO?**

SLO significa **Service Level Objective**.  
È l’obiettivo misurabile che il team si pone per mantenere un certo livello di servizio.

## Domanda 21
**Che cosa significa SLI?**

SLI significa **Service Level Indicator**.  
È la misura concreta usata per valutare un aspetto del servizio, per esempio disponibilità, latenza o tasso di errore.

## Domanda 22
**Qual è la differenza tra SLA, SLO e SLI?**

- SLI = indicatore misurato  
- SLO = obiettivo su quell’indicatore  
- SLA = accordo formale verso cliente o organizzazione

## Domanda 23
**Che cos’è un error budget?**

L’error budget è il margine di errore tollerabile rispetto allo SLO.  
Serve a bilanciare affidabilità e velocità di cambiamento.

## Domanda 24
**Perché l’error budget è importante?**

Perché permette di decidere quanto rischio operativo è accettabile, evitando sia cambiamenti troppo rischiosi sia eccessiva rigidità.

## Domanda 25
**Che cosa significa availability?**

La availability è la disponibilità del servizio, cioè la percentuale di tempo in cui il servizio è accessibile e funzionante.

## Domanda 26
**Che cosa significa latency?**

La latency è il tempo necessario perché una richiesta riceva risposta.

## Domanda 27
**Che cosa significa throughput?**

Il throughput è la quantità di lavoro che un sistema riesce a gestire in un dato intervallo di tempo, per esempio richieste al secondo.

## Domanda 28
**Che cosa significa error rate?**

L’error rate è la percentuale di richieste che terminano con errore rispetto al totale delle richieste.

## Domanda 29
**Che cosa significa MTTR?**

MTTR significa **Mean Time To Restore** oppure **Mean Time To Recover**.  
È il tempo medio necessario per riportare il servizio in condizioni normali dopo un incidente.

## Domanda 30
**Che cosa significa MTTD?**

MTTD significa **Mean Time To Detect**.  
È il tempo medio necessario per rilevare un problema.

## Domanda 31
**Perché ridurre MTTD e MTTR è importante?**

Perché significa:
- rilevare prima i problemi
- intervenire più rapidamente
- ridurre l’impatto sugli utenti e sul servizio

## Domanda 32
**Che cos’è il toil?**

Il toil è lavoro operativo manuale, ripetitivo, poco creativo, automatizzabile e che non produce un miglioramento strutturale del sistema.

## Domanda 33
**Perché l’SRE vuole ridurre il toil?**

Perché il toil:
- consuma tempo
- aumenta il rischio di errore umano
- sottrae energie a miglioramenti più utili come automazione e affidabilità

## Domanda 34
**Che cos’è un runbook?**

Un runbook è una procedura documentata che spiega come eseguire un’operazione o come gestire un problema in modo standardizzato.

## Domanda 35
**Che cos’è un postmortem?**

Il postmortem è l’analisi fatta dopo un incidente per capire:
- cosa è successo
- perché è successo
- quale impatto ha avuto
- come evitare che si ripeta

## Domanda 36
**Che cosa significa blameless postmortem?**

Significa analizzare l’incidente senza cercare colpevoli personali, con l’obiettivo di capire il sistema e migliorarlo.

## Domanda 37
**Che cos’è un health check?**

Un health check è un controllo automatico che verifica se un servizio è vivo, raggiungibile o pronto a rispondere correttamente.

## Domanda 38
**Che differenza c’è tra un sistema up e un sistema healthy?**

Un sistema può essere up, cioè acceso o in esecuzione, ma non essere healthy, cioè non essere realmente in grado di servire correttamente le richieste.

## Domanda 39
**Che cos’è un endpoint?**

Un endpoint è un punto di accesso a un servizio o a una risorsa applicativa, spesso identificato da URL oppure da host, porta e path.

## Domanda 40
**Che cos’è un request ID o correlation ID?**

È un identificatore univoco associato a una richiesta, usato per correlare log, eventi e altri segnali relativi alla stessa operazione.

## Domanda 41
**Perché la correlazione è importante in observability?**

Perché permette di collegare dati provenienti da componenti diversi e ricostruire il comportamento di una singola richiesta o di un incidente.

## Domanda 42
**Che cosa significa root cause?**

La root cause è la causa originaria del problema, non soltanto il sintomo visibile.

## Domanda 43
**Che cosa si intende per troubleshooting?**

Il troubleshooting è il processo di analisi e diagnosi di un problema basato su osservazione, raccolta dati, ipotesi, verifica e conclusione.

## Domanda 44
**Qual è una buona sequenza di troubleshooting?**

Una sequenza corretta è:
1. osservare il sintomo
2. raccogliere dati
3. formulare ipotesi
4. verificare le ipotesi
5. individuare la causa
6. applicare la correzione
7. verificare il risultato

## Domanda 45
**Perché in observability servono dati strutturati?**

Perché i dati strutturati sono più facili da filtrare, aggregare, correlare e analizzare automaticamente.

## Domanda 46
**Che cos’è la latenza P95?**

La latenza P95 indica il valore sotto il quale cade il 95% delle richieste.  
Serve a descrivere meglio l’esperienza reale rispetto alla sola media.

## Domanda 47
**Perché la media da sola non basta per valutare la latenza?**

Perché la media può nascondere code lente e casi peggiori.  
I percentili come P95 o P99 mostrano meglio il comportamento percepito dagli utenti.

## Domanda 48
**Che cosa sono i Golden Signals?**

I Golden Signals sono quattro categorie di segnali molto usate per osservare un servizio:
- latency
- traffic
- errors
- saturation

## Domanda 49
**Che cosa significa saturation?**

La saturation è il livello di saturazione delle risorse, cioè quanto il sistema si sta avvicinando ai suoi limiti.

## Domanda 50
**Perché observability e SRE sono collegati?**

Perché l’SRE ha bisogno di dati affidabili per misurare il servizio, rilevare problemi, diagnosticare incidenti e prendere decisioni.  
La observability fornisce i segnali, l’SRE li usa per migliorare affidabilità e operatività.




