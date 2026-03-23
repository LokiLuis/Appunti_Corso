# LABA05 - Azure Foundations: primo deployment applicativo con Azure App Service

## Obiettivo del laboratorio

In questo laboratorio imparerai a pubblicare una **semplice applicazione web** su Azure usando **Azure App Service**.

Dopo aver visto identità, Resource Group, storage e macchina virtuale, è il momento di capire un concetto fondamentale del cloud moderno: non sempre per pubblicare un’applicazione serve creare e gestire una VM.

Azure mette a disposizione servizi gestiti che permettono di distribuire applicazioni in modo molto più semplice.

In questo laboratorio vedrai:

- che cos’è Azure App Service
- che differenza c’è tra VM e servizio gestito per il deployment
- che cos’è un App Service Plan
- come creare una Web App
- come distribuire una pagina web semplice
- come verificare il funzionamento dell’app pubblicata
- come osservare i log e il ciclo di vita base della risorsa

Al termine del laboratorio sarai in grado di:

- spiegare che cos’è Azure App Service
- distinguere VM e piattaforma gestita per applicazioni
- creare una Web App
- distribuire una semplice pagina HTML
- testare l’endpoint pubblico dell’app
- osservare le principali proprietà della risorsa distribuita

---

## Scenario

Non tutte le applicazioni devono essere installate su una macchina virtuale amministrata manualmente.

Molte applicazioni web possono essere distribuite su una piattaforma che si occupa già di:

- sistema operativo di base
- hosting del runtime
- endpoint pubblico
- integrazione con il portale
- gestione semplificata del deployment

Questo modello è molto comune nel cloud e rappresenta una logica diversa rispetto alla VM.

In questo laboratorio pubblicherai una piccola applicazione web statica su Azure App Service, così da comprendere il concetto di **application deployment gestito**.

---

## Prerequisiti

Per svolgere il laboratorio occorrono:

- account Azure funzionante
- accesso al portale Azure
- accesso a Cloud Shell
- completamento dei laboratori LABA01, LABA02, LABA03 e LABA04
- un Resource Group disponibile
- connessione Internet stabile

---

## Durata indicativa

**2 ore circa**

---

## Competenze in uscita

Al termine del laboratorio sarai in grado di:

- spiegare il ruolo di Azure App Service
- capire che cos’è un App Service Plan
- creare una Web App nel portale Azure
- caricare una pagina HTML semplice
- testare il sito pubblicato via browser e via CLI
- osservare configurazione e log di base dell’app

---

# PARTE 1 - Concetti fondamentali

# 1. Che cos’è un deployment applicativo

Fare un **deployment applicativo** significa pubblicare un’applicazione in un ambiente dove possa essere eseguita e raggiunta dagli utenti.

In modo semplice, quando sviluppi un’applicazione web hai bisogno di:

- un ambiente in cui eseguirla
- un indirizzo pubblico o endpoint
- configurazioni di base
- un meccanismo per pubblicare i file o il codice

Nel mondo tradizionale questo spesso significava:

- preparare un server
- installare il sistema operativo
- installare il web server
- configurare porte e sicurezza
- copiare manualmente i file dell’app

Nel cloud esistono anche approcci più semplici.

---

# 2. Perché non sempre serve una VM

Nel laboratorio precedente hai creato una macchina virtuale Linux.

Quella soluzione è utile, ma comporta che tu debba gestire direttamente molti aspetti:

- sistema operativo
- accessi
- aggiornamenti
- configurazione del server
- servizi di runtime
- manutenzione generale

Per una semplice applicazione web, tutto questo può essere eccessivo.

Azure App Service nasce proprio per evitare che ogni applicazione richieda la gestione completa di una VM.

---

# 3. Che cos’è Azure App Service

**Azure App Service** è un servizio PaaS, cioè **Platform as a Service**.

Questo significa che Azure ti fornisce una piattaforma pronta per eseguire applicazioni, senza che tu debba amministrare direttamente l’intera macchina.

In pratica, con App Service:

- non gestisci una VM come server completo
- non devi configurare manualmente ogni dettaglio del sistema operativo
- puoi concentrarti di più sull’applicazione
- Azure gestisce gran parte della piattaforma sottostante

App Service è molto usato per pubblicare:

- siti web
- applicazioni web
- API
- backend applicativi

---

# 4. VM vs App Service

Questa differenza deve essere molto chiara.

## 4.1 VM

Con una VM:

- gestisci un server completo
- scegli sistema operativo e accesso amministrativo
- puoi installare quello che vuoi
- hai più controllo
- hai anche più responsabilità operative

## 4.2 App Service

Con App Service:

- distribuisci un’app su una piattaforma gestita
- Azure gestisce gran parte dell’ambiente
- lavori più vicino all’applicazione e meno all’infrastruttura
- il deployment è spesso più rapido e semplice

In forma molto semplificata:

- **VM** = amministri un server
- **App Service** = pubblichi un’app su una piattaforma pronta

---

# 5. Che cos’è un App Service Plan

Prima di creare una Web App, è importante capire un oggetto chiamato **App Service Plan**.

L’App Service Plan definisce il contesto computazionale in cui girerà l’applicazione.

In pratica stabilisce elementi come:

- area geografica
- capacità computazionale
- caratteristiche del piano
- livello di servizio

La relazione semplificata è questa:

```text
Resource Group
 └── App Service Plan
      └── Web App
````

Quindi:

* l’App Service Plan è il “contenitore di esecuzione”
* la Web App è l’applicazione concreta pubblicata su quel piano

---

# 6. Che cos’è una Web App

La **Web App** è la risorsa che rappresenta il sito o l’applicazione web pubblicata.

Ha:

* un nome
* un URL pubblico
* configurazioni proprie
* impostazioni applicative
* strumenti di monitoraggio e log

Per esempio, se la tua Web App si chiama:

```text
app-laba05-mario
```

Azure le assegnerà un URL simile a:

```text
https://app-laba05-mario.azurewebsites.net
```

Questo URL è l’endpoint pubblico del tuo sito.

---

# 7. Perché iniziare con una pagina HTML semplice

Per un laboratorio introduttivo è utile partire con qualcosa di minimo e chiaro.

Se usassi subito un’applicazione complessa, dovresti affrontare troppi problemi insieme:

* runtime
* dipendenze
* build
* configurazioni
* debug applicativo

Con una semplice pagina HTML puoi concentrarti sul concetto più importante:

**come si pubblica un’applicazione su una piattaforma Azure gestita**

---

# 8. Differenza tra contenuto statico e applicazione dinamica

In questo laboratorio userai una pagina HTML statica.

Questo significa che il contenuto mostrato è un file già pronto, non generato dinamicamente da logica server-side.

È utile come primo passo perché:

* è semplice da creare
* è facile da verificare
* permette di concentrarsi sul deployment
* riduce il numero di variabili tecniche

In laboratori successivi potresti pubblicare applicazioni più complesse.

---

# 9. Cosa farai nella parte pratica

Nella parte pratica eseguirai queste attività:

1. verificherai la subscription attiva
2. controllerai il Resource Group disponibile
3. creerai un App Service Plan
4. creerai una Web App
5. preparerai una semplice pagina HTML
6. la pubblicherai sull’App Service
7. testerai l’URL pubblico
8. verificherai il deployment con browser e CLI
9. osserverai le proprietà e i log base della Web App
10. comprenderai la differenza operativa tra questo modello e una VM

---

# PARTE 2 - Attività pratiche guidate

## Step 1 - Verifica la subscription attiva

Apri Cloud Shell in modalità Bash ed esegui:

```bash
az account show --output table
```

### Che cosa fa questo comando

* `az` richiama Azure CLI
* `account` indica che lavori sul contesto account
* `show` mostra il contesto attivo
* `--output table` rende il risultato più leggibile

### Che cosa devi osservare

Controlla:

* nome subscription
* stato
* tenant ID
* utente
* subscription predefinita

### Perché lo fai

Prima di creare nuove risorse devi verificare il contesto corretto.

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 2 - Verifica il Resource Group disponibile

Esegui:

```bash
az group list --output table
```

### Che cosa fa questo comando

* `group` indica che stai lavorando sui Resource Group
* `list` elenca quelli esistenti
* `--output table` mostra i risultati in tabella

### Che cosa devi osservare

Individua il Resource Group che userai per il laboratorio, ad esempio:

```text
rg-laba02-mario
```

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 3 - Apri il servizio App Services nel portale

Nel portale Azure cerca:

```text
App Services
```

Apri il servizio.

### Che cosa devi osservare

Guarda la schermata iniziale e individua:

* eventuale elenco di app già presenti
* pulsante di creazione
* eventuali colonne come nome, resource group, location, status

### Obiettivo dello step

Prendere familiarità con la sezione del portale dedicata alle applicazioni pubblicate su Azure.

### Evidenza richiesta

Esegui uno screenshot della pagina App Services.

---

## Step 4 - Avvia la creazione della Web App

Dalla pagina **App Services**, clicca su **Create** e poi su **Web App**.

Compila i campi principali in modo semplice e controllato.

### Impostazioni suggerite

#### Basics

* **Subscription**: seleziona la subscription del laboratorio
* **Resource Group**: seleziona il tuo Resource Group
* **Name**: scegli un nome univoco, ad esempio:

```text
app-laba05-<tuonome>-01
```

Esempio:

```text
app-laba05-mario-01
```

* **Publish**: Code
* **Runtime stack**: scegli uno stack semplice se richiesto dal portale, ad esempio `Node`, `.NET`, oppure uno stack disponibile. Per questo laboratorio non userai ancora codice applicativo complesso.
* **Operating System**: Linux
* **Region**: scegli una regione disponibile, ad esempio `West Europe`

#### App Service Plan

Se richiesto:

* crea un nuovo piano, ad esempio:

```text
plan-laba05-<tuonome>
```

Esempio:

```text
plan-laba05-mario
```

* scegli un piano di test o base compatibile con il laboratorio e con i limiti disponibili

### Che cosa stai facendo

Stai creando:

* una piattaforma di esecuzione, cioè l’App Service Plan
* una Web App con endpoint pubblico

### Attenzione

Il nome della Web App deve essere univoco, perché farà parte dell’URL pubblico.

Se il nome è già usato, aggiungi iniziali o numeri.

### Evidenza richiesta

Esegui uno screenshot della schermata di riepilogo prima della creazione oppure della Web App appena creata.

---

## Step 5 - Attendi il completamento del provisioning

Dopo aver confermato la creazione, attendi che Azure completi il provisioning.

### Cosa significa provisioning

Azure sta creando e configurando:

* la Web App
* il piano di esecuzione
* l’endpoint pubblico
* le impostazioni iniziali del servizio

### Evidenza richiesta

Annota nel file evidenze che il provisioning è terminato con successo.

---

## Step 6 - Apri la Web App e osserva le informazioni principali

Apri la Web App appena creata.

### Che cosa devi osservare

Individua almeno:

* nome dell’app
* Resource Group
* URL pubblico
* stato
* sistema operativo
* App Service Plan associato
* regione

### Cosa devi capire

La Web App è la risorsa applicativa, mentre il piano sottostante è il contesto di esecuzione.

### Evidenza richiesta

Esegui uno screenshot della pagina principale della Web App.

---

## Step 7 - Testa l’URL pubblico iniziale

Nella schermata della Web App individua il campo **Default domain** oppure l’URL pubblico.

Aprilo nel browser.

L’URL sarà simile a questo:

```text
https://app-laba05-mario-01.azurewebsites.net
```

### Che cosa devi osservare

La pagina iniziale potrebbe mostrare:

* una pagina di default
* una pagina generica di avvio
* una pagina vuota o standard del servizio

### Perché lo fai

Devi verificare che l’applicazione sia effettivamente raggiungibile tramite Internet.

### Evidenza richiesta

Esegui uno screenshot della pagina visualizzata nel browser.

---

## Step 8 - Prepara una semplice pagina HTML in Cloud Shell

In Cloud Shell crea una cartella di lavoro:

```bash
mkdir -p laba05site
cd laba05site
```

### Spiegazione dei comandi

* `mkdir -p laba05site` crea una directory chiamata `laba05site`
* `-p` evita errori se la cartella esiste già
* `cd laba05site` entra nella cartella

### Evidenza richiesta

Annota i comandi eseguiti nel file delle evidenze.

---

## Step 9 - Crea il file index.html

Esegui:

```bash
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <title>LABA05 - Prima Web App su Azure</title>
</head>
<body>
  <h1>LABA05 - Prima Web App su Azure</h1>
  <p>Questa pagina è stata pubblicata su Azure App Service.</p>
  <p>Partecipante: INSERIRE_NOME</p>
  <p>Obiettivo: comprendere il deployment applicativo su piattaforma gestita.</p>
</body>
</html>
EOF
```

### Che cosa fa questo comando

* `cat > index.html` crea un file chiamato `index.html`
* `<<'EOF'` indica che il testo successivo verrà scritto nel file fino alla riga finale `EOF`

### Che cosa devi fare dopo

Apri il file e sostituisci `INSERIRE_NOME` con il tuo nome, se necessario.

Puoi verificare il contenuto con:

```bash
cat index.html
```

### Evidenza richiesta

Copia l’output di `cat index.html` nel file delle evidenze.

---

## Step 10 - Crea un archivio ZIP del sito

Esegui:

```bash
zip site.zip index.html
```

### Che cosa fa questo comando

* `zip` crea un archivio compresso
* `site.zip` è il nome del file ZIP
* `index.html` è il file da inserire nell’archivio

### Perché è utile

Uno dei modi più semplici per distribuire un piccolo sito su App Service è fare il deployment di un file ZIP.

### Evidenza richiesta

Annota il comando eseguito e verifica la presenza del file con:

```bash
ls -l
```

Copia l’output nel file delle evidenze.

---

## Step 11 - Recupera il nome della Web App e del Resource Group

Prima del deployment assicurati di conoscere con precisione:

* nome della Web App
* nome del Resource Group

Esempio:

* Resource Group: `rg-laba02-mario`
* Web App: `app-laba05-mario-01`

Questi valori saranno usati nei comandi successivi.

---

## Step 12 - Esegui il deployment ZIP con Azure CLI

Esegui il comando seguente sostituendo i valori corretti:

```bash
az webapp deploy --resource-group <NOME_RESOURCE_GROUP> --name <NOME_WEBAPP> --src-path site.zip
```

Esempio:

```bash
az webapp deploy --resource-group rg-laba02-mario --name app-laba05-mario-01 --src-path site.zip
```

### Che cosa fa questo comando

* `webapp` indica che stai lavorando con una Web App
* `deploy` avvia un deployment applicativo
* `--resource-group` specifica il gruppo di risorse
* `--name` specifica la Web App
* `--src-path site.zip` indica il file ZIP da pubblicare

### Che cosa devi osservare

Azure CLI mostrerà informazioni sul deployment.

### Evidenza richiesta

Copia l’output del comando nel file delle evidenze.

---

## Step 13 - Attendi il completamento del deployment

Aspetta qualche istante dopo il deployment.

### Perché lo fai

Anche se il comando termina, l’aggiornamento dell’applicazione può richiedere qualche secondo.

### Evidenza richiesta

Annota nel file evidenze che hai atteso il completamento del deployment.

---

## Step 14 - Apri di nuovo l’URL pubblico

Apri di nuovo nel browser l’URL della Web App, ad esempio:

```text
https://app-laba05-mario-01.azurewebsites.net
```

### Che cosa devi osservare

Ora dovresti vedere la tua pagina HTML personalizzata con:

* titolo del laboratorio
* testo descrittivo
* nome del partecipante

### Cosa devi capire

Hai pubblicato con successo un contenuto applicativo su una piattaforma gestita Azure.

### Evidenza richiesta

Esegui uno screenshot della pagina finale pubblicata.

---

## Step 15 - Testa l’endpoint con curl

In Cloud Shell esegui:

```bash
curl -I https://<NOME_WEBAPP>.azurewebsites.net
```

Esempio:

```bash
curl -I https://app-laba05-mario-01.azurewebsites.net
```

### Che cosa fa questo comando

* `curl` effettua una richiesta HTTP
* `-I` chiede solo le intestazioni HTTP della risposta

### Che cosa devi osservare

Cerca nell’output una risposta come:

```text
HTTP/1.1 200 OK
```

### Perché è utile

Ti permette di verificare via terminale che il sito risponda correttamente.

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 16 - Visualizza i dettagli della Web App da Azure CLI

Esegui il comando seguente sostituendo i valori corretti:

```bash
az webapp show --resource-group <NOME_RESOURCE_GROUP> --name <NOME_WEBAPP> --output json
```

Esempio:

```bash
az webapp show --resource-group rg-laba02-mario --name app-laba05-mario-01 --output json
```

### Che cosa fa questo comando

* `show` restituisce il dettaglio della Web App
* `--output json` mostra il risultato in formato strutturato

### Che cosa devi osservare

Individua almeno questi campi, se presenti:

* `name`
* `state`
* `defaultHostName`
* `location`
* `resourceGroup`
* `kind`

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 17 - Visualizza le impostazioni generali del piano

Se vuoi vedere anche il piano associato, puoi eseguire:

```bash
az appservice plan list --output table
```

### Che cosa fa questo comando

Mostra i piani App Service disponibili nella subscription.

### Che cosa devi osservare

Cerca il piano creato per il laboratorio, ad esempio:

```text
plan-laba05-mario
```

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 18 - Esplora i log dal portale

Apri la Web App nel portale e cerca sezioni come:

* Log stream
* Monitoring
* Activity Log
* Deployment Center

### Che cosa devi osservare

Per un laboratorio introduttivo è sufficiente capire che la Web App offre strumenti di osservazione e verifica del comportamento e del deployment.

### Cosa devi capire

Anche se non stai ancora facendo observability avanzata, stai iniziando a vedere dove si controlla il funzionamento di un’app su Azure.

### Evidenza richiesta

Esegui almeno uno screenshot di una sezione di monitoring, log o deployment relativa alla Web App.

---

## Step 19 - Verifica l’Activity Log

Nel portale Azure cerca:

```text
Activity Log
```

Apri il servizio.

### Che cosa osservare

Prova a individuare eventi legati a:

* creazione della Web App
* creazione del piano App Service
* eventuali aggiornamenti o operazioni amministrative

### Cosa devi capire

L’Activity Log registra le attività amministrative sulla risorsa, non il contenuto applicativo HTML in sé.

### Evidenza richiesta

Esegui uno screenshot dell’Activity Log con almeno un evento relativo alla Web App.

---

## Step 20 - Riepiloga mentalmente la struttura

A questo punto devi saper leggere una struttura come questa:

```text
Subscription
 └── Resource Group
      ├── App Service Plan
      └── Web App
           └── Endpoint pubblico
```

Esempio concreto:

```text
Subscription: Azure for Students
Resource Group: rg-laba02-mario
App Service Plan: plan-laba05-mario
Web App: app-laba05-mario-01
URL: https://app-laba05-mario-01.azurewebsites.net
```

---

# 3. Mini esercizio finale

Rispondi per iscritto alle seguenti domande.

## Domanda 1

Che differenza c’è tra pubblicare un’app su una VM e pubblicarla su Azure App Service?

## Domanda 2

Che cos’è un App Service Plan?

## Domanda 3

Perché il nome della Web App deve essere univoco?

## Domanda 4

Perché è utile testare l’app sia dal browser sia con `curl`?

## Domanda 5

Qual è il vantaggio principale di una piattaforma gestita come App Service per applicazioni semplici o standard?

---

# 4. Output attesi

Al termine del laboratorio devi aver prodotto le seguenti evidenze:

1. screenshot della pagina **App Services**
2. screenshot della **Web App** creata
3. screenshot della pagina web iniziale
4. screenshot della pagina web finale dopo il deployment
5. screenshot di una sezione di **monitoring, log o deployment**
6. screenshot dell’**Activity Log**
7. output del comando:

```bash
az account show --output table
```

8. output del comando:

```bash
az group list --output table
```

9. comando usato per creare la cartella di lavoro:

```bash
mkdir -p laba05site
cd laba05site
```

10. contenuto del file:

```bash
cat index.html
```

11. output del comando:

```bash
ls -l
```

12. output del comando:

```bash
az webapp deploy --resource-group <NOME_RESOURCE_GROUP> --name <NOME_WEBAPP> --src-path site.zip
```

13. output del comando:

```bash
curl -I https://<NOME_WEBAPP>.azurewebsites.net
```

14. output del comando:

```bash
az webapp show --resource-group <NOME_RESOURCE_GROUP> --name <NOME_WEBAPP> --output json
```

15. output del comando:

```bash
az appservice plan list --output table
```

16. risposte scritte alle 5 domande finali

---

# 5. Modello di file evidenze

Crea un file chiamato:

```text
evidenze_laba05.md
```

e usa una struttura come questa:

```md
# Evidenze LABA05

## 1. Web App
- Nome Web App:
- Resource Group:
- Regione:
- Sistema operativo:
- App Service Plan:
- URL pubblico:

## 2. Output CLI

### az account show --output table
[incollare qui l’output]

### az group list --output table
[incollare qui l’output]

### mkdir -p laba05site
### cd laba05site
[annotare i comandi eseguiti]

### cat index.html
[incollare qui l’output]

### ls -l
[incollare qui l’output]

### az webapp deploy --resource-group <NOME_RESOURCE_GROUP> --name <NOME_WEBAPP> --src-path site.zip
[incollare qui l’output]

### curl -I https://<NOME_WEBAPP>.azurewebsites.net
[incollare qui l’output]

### az webapp show --resource-group <NOME_RESOURCE_GROUP> --name <NOME_WEBAPP> --output json
[incollare qui l’output]

### az appservice plan list --output table
[incollare qui l’output]

## 3. Risposte finali

### Domanda 1
[risposta]

### Domanda 2
[risposta]

### Domanda 3
[risposta]

### Domanda 4
[risposta]

### Domanda 5
[risposta]
```

---

# 6. Errori comuni da evitare

## Errore 1 - Confondere Web App e App Service Plan

Correzione:

* la **Web App** è l’applicazione pubblicata
* l’**App Service Plan** è il contesto di esecuzione che la ospita

---

## Errore 2 - Scegliere un nome Web App non univoco

Correzione:

Il nome entra nell’URL pubblico e deve essere unico. Se il nome è già usato, aggiungi numeri o iniziali.

---

## Errore 3 - Dimenticare il Resource Group corretto nei comandi

Correzione:

Prima del deployment verifica sempre con attenzione nome della Web App e nome del Resource Group.

---

## Errore 4 - Fare il deployment senza controllare il contenuto del file HTML

Correzione:

Verifica sempre il file con:

```bash
cat index.html
```

prima di creare l’archivio ZIP e fare il deployment.

---

## Errore 5 - Pensare che App Service sia una VM tradizionale

Correzione:

App Service è una piattaforma gestita. Tu distribuisci l’applicazione, ma non amministri il server nello stesso modo in cui faresti con una VM.

---

# 7. Conclusione

Questo laboratorio ti ha introdotto al deployment applicativo su **Azure App Service**, uno dei modelli più importanti del cloud moderno.

Dopo aver completato il lab, devi avere chiaro che:

* non tutte le applicazioni richiedono una VM
* Azure App Service permette di pubblicare app in modo più semplice e gestito
* l’App Service Plan definisce il contesto di esecuzione
* la Web App rappresenta l’applicazione concreta pubblicata
* il deployment può essere eseguito in modo rapido anche con strumenti semplici come un archivio ZIP
* il sito pubblicato può essere verificato sia via browser sia via CLI

Con questo laboratorio si completa il blocco introduttivo Azure Foundations necessario per affrontare in modo più consapevole i laboratori successivi del modulo Azure.

```
```
