# LAB03 - Dal monolite didattico all’app distribuita FE/BE
## Monorepo GitHub, due immagini Docker e pipeline Azure DevOps multi-image

---

# 1. Obiettivo del laboratorio

In questo laboratorio farai il passaggio corretto dalla app standalone usata nei primi due laboratori al progetto reale che userai nei laboratori successivi.

Non userai più una sola applicazione containerizzata, ma una applicazione composta da:

- **frontend**
- **backend**

Alla fine del laboratorio dovrai essere in grado di:

- spiegare perché frontend e backend vanno trattati come servizi distinti
- organizzare un **monorepo** GitHub FE/BE
- containerizzare i due servizi separatamente
- costruire localmente due immagini Docker
- pubblicare entrambe le immagini su **Azure Container Registry**
- usare una pipeline Azure DevOps che costruisce e pubblica **due immagini**

---

# 2. Scopo di questo documento

Nei laboratori precedenti hai lavorato con una sola applicazione, perché serviva prima capire bene:

- repository
- pipeline
- build
- registry
- deploy
- smoke test

Ora però bisogna passare alla struttura reale del progetto.

Questo laboratorio serve quindi a fare bene la transizione da:

- un contenitore solo
- una pipeline semplice
- una immagine sola

a:

- due servizi distinti
- due Dockerfile
- due immagini
- una pipeline che sa gestire un progetto multi-componente

In questo laboratorio **non facciamo ancora il deploy finale**.  
La scelta è voluta: prima prepariamo bene la struttura del progetto, poi nei laboratori successivi faremo il deploy su una piattaforma più adatta al progetto FE/BE.

---

# Parte 1 - Spiegazione dei termini e degli strumenti

# 3. Architettura logica del laboratorio

L’architettura applicativa del progetto è questa:

```text
Client
  |
  v
Frontend
  |
  v
Backend
````

Nel LAB03 non la porterai ancora nel cloud.
La preparerai correttamente per poterla distribuire nei laboratori successivi.

---

# 4. Che cosa significa frontend e backend come servizi separati

In questo corso:

* il **frontend** riceve la richiesta principale del client
* il **frontend** chiama il **backend**
* il **backend** esegue il lavoro di servizio e restituisce la risposta

## 4.1 Perché vanno separati

Frontend e backend vanno separati perché:

* hanno ruoli diversi
* possono evolvere in modo diverso
* possono avere immagini Docker diverse
* nel cloud saranno distribuiti come componenti distinti
* dal punto di vista DevOps è utile poterli buildare, versionare e aggiornare separatamente

Questa separazione è importante sia dal lato architetturale sia dal lato didattico.

---

# 5. Che cos’è un monorepo

Un **monorepo** è un repository unico che contiene più componenti dello stesso progetto.

Nel nostro caso il monorepo conterrà:

* `frontend/`
* `backend/`
* `azure-pipelines.yml`
* documentazione ed evidenze

## 5.1 Perché lo usiamo qui

Per questo modulo il monorepo è una buona scelta perché:

* mantiene frontend e backend nello stesso contesto
* semplifica la gestione del progetto per i partecipanti
* consente una pipeline unica
* riduce la dispersione del materiale

## 5.2 Monorepo non significa monolite

Questo è un punto importante.

Avere un solo repository non significa avere una sola applicazione.

Il repository è uno.
I servizi restano due.

---

# 6. Perché servono due immagini Docker

Nel LAB01 e LAB02 avevi una sola immagine.

Qui invece avrai:

* una immagine per il **frontend**
* una immagine per il **backend**

## 6.1 Perché non usare una sola immagine

Perché questo nasconderebbe la struttura reale del sistema.

Se frontend e backend finiscono nello stesso contenitore, perdi:

* chiarezza architetturale
* separazione dei ruoli
* controllo sul ciclo di vita dei due componenti
* coerenza con i laboratori successivi

Quindi in LAB03 prepariamo il progetto esattamente nel modo giusto per il passo successivo.

---

# 7. Che ruolo ha GitHub in LAB03

GitHub continua a essere il repository del codice.

In questo laboratorio GitHub conterrà un progetto più ricco rispetto ai primi due lab:

* cartella `frontend`
* cartella `backend`
* file pipeline
* documentazione

GitHub quindi resta il punto in cui vive la struttura del progetto.

---

# 8. Che ruolo ha Azure DevOps in LAB03

Azure DevOps continua a essere il motore della CI/CD.

In questo laboratorio la pipeline non farà ancora il deploy finale, ma farà una cosa nuova e importante:

* costruirà **due immagini**
* pubblicherà **due immagini**

Quindi Azure DevOps smette di lavorare su una sola build lineare e inizia a gestire un piccolo progetto distribuito.

---

# 9. Che ruolo ha ACR in LAB03

ACR continua a essere il registry immagini.

La differenza è che adesso non conterrà una sola immagine, ma due repository logici distinti, ad esempio:

* `frontend`
* `backend`

Dentro ciascuno di questi repository troverai tag diversi.

## 9.1 Cosa significa in pratica

Per esempio, dopo la pipeline potresti avere:

```text
nomeregistry.azurecr.io/frontend:35
nomeregistry.azurecr.io/backend:35
```

dove `35` è il `Build.BuildId` della pipeline.

---

# 10. Perché in LAB03 non facciamo ancora il deploy

Questa scelta è intenzionale.

In questo laboratorio vogliamo concentrarci bene su:

* struttura del progetto
* containerizzazione
* naming
* build
* push su registry
* pipeline multi-image

Se aggiungessimo subito anche il deploy, il laboratorio rischierebbe di diventare troppo carico.

Quindi qui prepariamo il progetto reale.
Il deploy arriverà nel laboratorio successivo, dove useremo una piattaforma più adatta a un’applicazione FE/BE.

---

# 11. Concetti chiave da ricordare prima della parte pratica

Prima di passare allo step-by-step, devi avere chiaro questo:

* **GitHub** conserva il monorepo FE/BE
* **Azure DevOps** esegue la pipeline multi-image
* **ACR** conserva le immagini frontend e backend
* in questo laboratorio **non facciamo ancora il deploy**
* l’obiettivo è preparare correttamente il progetto reale

---

# Parte 2 - Step-by-step guidato

# 12. Prerequisiti del laboratorio

Per eseguire LAB03 devono essere già disponibili:

* account GitHub
* repository GitHub oppure possibilità di crearne uno nuovo
* organizzazione Azure DevOps
* progetto Azure DevOps già pronto
* service connection GitHub funzionante
* service connection Azure Resource Manager funzionante
* Azure Container Registry già esistente
* Azure CLI installata e funzionante
* Docker funzionante
* Git funzionante
* VS Code o editor equivalente

È inoltre utile che l’admin account di ACR sia già abilitato, perché useremo ancora una modalità semplice di autenticazione lato laboratorio.

---

# 13. Creazione della cartella di lavoro locale

Apri il terminale WSL:

```bash
mkdir -p ~/corso_obs/lab03-fe-be-devops/frontend
mkdir -p ~/corso_obs/lab03-fe-be-devops/backend
mkdir -p ~/corso_obs/lab03-fe-be-devops/docs
cd ~/corso_obs/lab03-fe-be-devops
```

Verifica:

```bash
pwd
find . -maxdepth 2 -type d | sort
```

---

# 14. Creazione del backend

## 14.1 File `backend/app.py`

Crea il file:

```bash
code backend/app.py
```

Inserisci questo contenuto:

```python
import time
import random
from flask import Flask, jsonify

app = Flask(__name__)

@app.get("/work")
def work():
    delay = random.uniform(0.1, 0.8)
    time.sleep(delay)

    return jsonify(
        {
            "service": "backend",
            "message": "backend completed",
            "processing_time": round(delay, 3),
        }
    )

@app.get("/health")
def health():
    return jsonify({"status": "ok", "service": "backend"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

## 14.2 File `backend/requirements.txt`

Crea il file:

```bash
code backend/requirements.txt
```

Inserisci:

```txt
flask==3.0.0
```

## 14.3 File `backend/Dockerfile`

Crea il file:

```bash
code backend/Dockerfile
```

Inserisci:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 8000

CMD ["python", "app.py"]
```

## 14.4 File `backend/.dockerignore`

Crea il file:

```bash
code backend/.dockerignore
```

Inserisci:

```txt
__pycache__/
*.pyc
*.pyo
*.pyd
.venv/
venv/
.env
```

---

# 15. Creazione del frontend

## 15.1 File `frontend/app.py`

Crea il file:

```bash
code frontend/app.py
```

Inserisci questo contenuto:

```python
import os
import requests
from flask import Flask, jsonify

BACKEND_URL = os.environ.get("BACKEND_URL", "http://localhost:8001")

app = Flask(__name__)

@app.get("/demo")
def demo():
    response = requests.get(f"{BACKEND_URL}/work", timeout=5)
    backend_data = response.json()

    return jsonify(
        {
            "service": "frontend",
            "message": "frontend completed",
            "backend_response": backend_data,
        }
    )

@app.get("/health")
def health():
    return jsonify({"status": "ok", "service": "frontend"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

## 15.2 File `frontend/requirements.txt`

Crea il file:

```bash
code frontend/requirements.txt
```

Inserisci:

```txt
flask==3.0.0
requests==2.31.0
```

## 15.3 File `frontend/Dockerfile`

Crea il file:

```bash
code frontend/Dockerfile
```

Inserisci:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 8000

CMD ["python", "app.py"]
```

## 15.4 File `frontend/.dockerignore`

Crea il file:

```bash
code frontend/.dockerignore
```

Inserisci:

```txt
__pycache__/
*.pyc
*.pyo
*.pyd
.venv/
venv/
.env
```

---

# 16. Verifica sintattica minima dei file Python

Esegui:

```bash
python3 -m py_compile backend/app.py
python3 -m py_compile frontend/app.py
```

Se non compare output, la sintassi è corretta.

---

# 17. Build locale delle immagini Docker

## 17.1 Build del backend

```bash
cd ~/corso_obs/lab03-fe-be-devops
docker build -t backend:lab03 ./backend
```

## 17.2 Build del frontend

```bash
docker build -t frontend:lab03 ./frontend
```

## 17.3 Verifica immagini locali

```bash
docker images | grep -E 'frontend|backend'
```

Dovresti vedere entrambe le immagini.

---

# 18. Test locale minimo dei due container

## 18.1 Avvio backend

```bash
docker run -d --rm --name be-lab03 -p 8001:8000 backend:lab03
```

## 18.2 Avvio frontend

```bash
docker run -d --rm --name fe-lab03 -p 8000:8000 \
  -e BACKEND_URL=http://host.docker.internal:8001 \
  frontend:lab03
```

## 18.3 Test backend

```bash
curl http://localhost:8001/health
curl http://localhost:8001/work
```

## 18.4 Test frontend

```bash
curl http://localhost:8000/health
curl http://localhost:8000/demo
```

## 18.5 Arresto container

```bash
docker stop fe-lab03
docker stop be-lab03
```

---

# 19. Creazione del repository GitHub del monorepo

Crea su GitHub un repository, ad esempio:

* `fe-be-devops`

Poi inizializza Git locale e collega il repository remoto:

```bash
cd ~/corso_obs/lab03-fe-be-devops
git init
git branch -M main
git add .
git commit -m "LAB03 - Initial FE/BE monorepo structure"
git remote add origin URL-DEL-TUO-REPO-GITHUB
git push -u origin main
```

Verifica che il repository GitHub contenga:

* `frontend/`
* `backend/`
* `docs/`

---

# 20. Verifica delle risorse Azure

## 20.1 Login Azure

```bash
az login
az account show
```

## 20.2 Verifica ACR

```bash
az acr show --name NOME_ACR --resource-group NOME_RG
```

## 20.3 Login ACR

```bash
az acr login --name NOME_ACR
```

---

# 21. Push manuale iniziale delle immagini su ACR

Prima di automatizzare con la pipeline, fai un push manuale una volta.

## 21.1 Tag immagini

```bash
docker tag frontend:lab03 NOME_ACR.azurecr.io/frontend:lab03-manual
docker tag backend:lab03 NOME_ACR.azurecr.io/backend:lab03-manual
```

## 21.2 Push immagini

```bash
docker push NOME_ACR.azurecr.io/frontend:lab03-manual
docker push NOME_ACR.azurecr.io/backend:lab03-manual
```

## 21.3 Verifica tag in ACR

```bash
az acr repository show-tags --name NOME_ACR --repository frontend -o table
az acr repository show-tags --name NOME_ACR --repository backend -o table
```

Questa verifica è importante perché mostra chiaramente che frontend e backend sono già due repository logici distinti nel registry.

---

# 22. Creazione del file `azure-pipelines.yml`

Crea il file nella radice del progetto:

```bash
code azure-pipelines.yml
```

Inserisci questo contenuto, adattando i valori del tuo ambiente:

```yaml
trigger:
  - main

pr: none

pool:
  vmImage: ubuntu-latest

variables:
  azureServiceConnection: 'sc-obs-azure-rg'
  acrName: 'nomeregistry'
  imageTag: '$(Build.BuildId)'

stages:
- stage: ValidateRepository
  displayName: Validate monorepo structure
  jobs:
  - job: ValidateFiles
    displayName: Check required files and Python syntax
    steps:
    - checkout: self

    - bash: |
        set -e

        echo "Verifica struttura repository"
        test -f frontend/app.py
        test -f frontend/requirements.txt
        test -f frontend/Dockerfile
        test -f backend/app.py
        test -f backend/requirements.txt
        test -f backend/Dockerfile

        echo "Verifica sintassi Python"
        python3 -m py_compile frontend/app.py
        python3 -m py_compile backend/app.py

        echo "Verifica completata"
      displayName: Validate files and Python syntax

- stage: PublishImages
  displayName: Build and push FE/BE images
  dependsOn: ValidateRepository
  condition: succeeded()
  jobs:
  - job: BuildBackend
    displayName: Build and push backend image
    steps:
    - checkout: self

    - task: AzureCLI@2
      displayName: Build and push backend image to ACR
      inputs:
        azureSubscription: '$(azureServiceConnection)'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          set -e

          az account show
          az acr login --name $(acrName)

          BACKEND_IMAGE="$(acrName).azurecr.io/backend:$(imageTag)"

          echo "Build backend image: ${BACKEND_IMAGE}"
          docker build -t ${BACKEND_IMAGE} ./backend

          echo "Push backend image: ${BACKEND_IMAGE}"
          docker push ${BACKEND_IMAGE}

          az acr repository show-tags \
            --name $(acrName) \
            --repository backend \
            -o table

  - job: BuildFrontend
    displayName: Build and push frontend image
    steps:
    - checkout: self

    - task: AzureCLI@2
      displayName: Build and push frontend image to ACR
      inputs:
        azureSubscription: '$(azureServiceConnection)'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          set -e

          az account show
          az acr login --name $(acrName)

          FRONTEND_IMAGE="$(acrName).azurecr.io/frontend:$(imageTag)"

          echo "Build frontend image: ${FRONTEND_IMAGE}"
          docker build -t ${FRONTEND_IMAGE} ./frontend

          echo "Push frontend image: ${FRONTEND_IMAGE}"
          docker push ${FRONTEND_IMAGE}

          az acr repository show-tags \
            --name $(acrName) \
            --repository frontend \
            -o table
```

---

# 23. Perché questa pipeline è corretta per LAB03

Questa pipeline fa esattamente ciò che ci serve in questo laboratorio:

* verifica che il monorepo abbia la struttura minima corretta
* verifica la sintassi Python
* costruisce il backend
* costruisce il frontend
* pubblica entrambe le immagini su ACR

In questo laboratorio non c’è ancora il deploy finale, perché il focus è preparare bene il progetto reale.

---

# 24. Salvataggio e push su GitHub

```bash
git add .
git commit -m "LAB03 - Add FE/BE multi-image pipeline"
git push
```

Verifica che il repository GitHub contenga anche il file `azure-pipelines.yml`.

---

# 25. Creazione della pipeline in Azure DevOps

Nel portale Azure DevOps:

1. vai in **Pipelines**
2. scegli **Create Pipeline**
3. seleziona **GitHub**
4. autorizza l’accesso se richiesto
5. seleziona il repository GitHub del monorepo
6. scegli il file `azure-pipelines.yml`
7. salva ed esegui

Dovresti vedere almeno due stage:

* `ValidateRepository`
* `PublishImages`

---

# 26. Verifica dell’esecuzione pipeline

Durante l’esecuzione controlla che compaiano:

## Stage 1

* verifica file
* verifica sintassi Python

## Stage 2

* build backend
* push backend
* build frontend
* push frontend

Controlla nei log la presenza di:

* `az account show`
* `az acr login`
* `docker build`
* `docker push`

---

# 27. Verifica finale in ACR

Da CLI locale esegui:

```bash
az acr repository show-tags --name NOME_ACR --repository frontend -o table
az acr repository show-tags --name NOME_ACR --repository backend -o table
```

Dovresti vedere:

* almeno il tag manuale `lab03-manual`
* almeno un tag numerico della pipeline

---

# 28. Troubleshooting minimo

## 28.1 Errore nella build frontend

Possibili cause:

* `requests` mancante nel file `frontend/requirements.txt`
* `Dockerfile` frontend errato
* file `app.py` frontend mancante

Verifica:

```bash
cat frontend/requirements.txt
cat frontend/Dockerfile
```

## 28.2 Errore nella build backend

Possibili cause:

* `Dockerfile` backend errato
* file `backend/app.py` mancante
* errore di sintassi Python

Verifica:

```bash
python3 -m py_compile backend/app.py
```

## 28.3 Push su ACR fallito

Possibili cause:

* login Azure errato
* nome ACR errato
* permessi insufficienti

Verifica:

```bash
az account show
az acr show --name NOME_ACR --resource-group NOME_RG
az acr login --name NOME_ACR
```

## 28.4 I job non vanno in parallelo

Possibile causa:

* l’organizzazione Azure DevOps ha un solo parallel job disponibile

Questo non è un errore del laboratorio.
È un comportamento normale dell’ambiente.

## 28.5 Il frontend non riesce a chiamare il backend nel test locale

Possibili cause:

* `host.docker.internal` non disponibile nel tuo contesto
* backend non avviato
* porta errata

In questo caso puoi verificare con:

```bash
docker ps
curl http://localhost:8001/health
```

---

# 29. Evidenze richieste

Crea il file:

```bash
code docs/evidence_lab03.md
```

Inserisci questa struttura:

```md
# Evidence LAB03

## 1. Obiettivo compreso
Spiego con parole mie perché questo laboratorio prepara il progetto reale dei lab successivi.

## 2. Struttura monorepo
Descrivo la struttura del repository FE/BE.

## 3. Codice backend
Riporto il ruolo del backend e gli endpoint disponibili.

## 4. Codice frontend
Riporto il ruolo del frontend e come chiama il backend.

## 5. Build locale
Descrivo come ho costruito localmente le due immagini.

## 6. Test locale
Riporto gli output di:
- GET /health backend
- GET /work backend
- GET /health frontend
- GET /demo frontend

## 7. Push manuale iniziale
Descrivo i tag manuali pubblicati in ACR.

## 8. Pipeline Azure DevOps
Descrivo le stage e i job della pipeline multi-image.

## 9. Verifica finale ACR
Incollo l’output di:
- az acr repository show-tags --repository frontend
- az acr repository show-tags --repository backend

## 10. Problemi incontrati
Descrivo eventuali errori e come li ho risolti.
```

---

# 30. Consegna

Al termine del laboratorio devi consegnare:

* repository GitHub FE/BE completo
* pipeline Azure DevOps funzionante
* file `docs/evidence_lab03.md`

Salva il lavoro:

```bash
git add .
git commit -m "LAB03 completato - FE/BE monorepo with multi-image pipeline"
git push
```

---

# 31. Conclusione del laboratorio

In questo laboratorio hai fatto il passaggio corretto dal caso semplice al progetto reale:

* non più una sola app
* non più una sola immagine
* non più una sola build lineare

Hai preparato:

* struttura FE/BE
* immagini separate
* pipeline che sa costruire due componenti

Nel prossimo laboratorio userai questa base per fare il salto naturale: il deploy FE/BE su una piattaforma cloud più adatta al progetto distribuito.

```

