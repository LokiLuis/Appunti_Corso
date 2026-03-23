# LABA04 - Azure Foundations: macchina virtuale Linux, rete di base, accesso SSH e gestione del ciclo di vita

## Obiettivo del laboratorio

In questo laboratorio imparerai a creare e usare una **macchina virtuale Linux** su Azure.

Prima di arrivare a container, app deployment, observability e servizi gestiti, è importante capire bene che cosa sia una VM nel cloud e quali elementi servano per farla funzionare.

In questo laboratorio vedrai:

- che cos’è una macchina virtuale
- che differenza c’è tra VM, computer fisico e container
- quali componenti Azure vengono coinvolti quando crei una VM
- come creare una VM Linux dal portale
- come collegarti in SSH
- come eseguire i primi comandi Linux sulla VM
- come arrestare correttamente una VM
- come comprenderne il ciclo di vita

Al termine del laboratorio sarai in grado di:

- spiegare che cos’è una VM in Azure
- riconoscere i componenti principali associati a una VM
- creare una VM Linux di base
- collegarti alla VM tramite SSH
- eseguire semplici verifiche sul sistema
- arrestare, avviare e osservare lo stato della VM
- capire i costi e il cleanup di base

---

## Scenario

Nel cloud non esistono solo servizi completamente gestiti come storage account o app service.

Molto spesso si usano ancora macchine virtuali, cioè server creati in modo software dentro l’infrastruttura del provider cloud.

Una VM Azure può essere usata per:

- ospitare applicazioni
- eseguire servizi di backend
- fare test
- simulare ambienti reali
- installare strumenti Linux
- creare un primo laboratorio infrastrutturale

In questo laboratorio creerai una piccola VM Linux e la userai come primo server nel cloud.

---

## Prerequisiti

Per svolgere il laboratorio occorrono:

- account Azure funzionante
- accesso al portale Azure
- accesso a Cloud Shell
- completamento dei laboratori LABA01, LABA02 e LABA03
- un Resource Group disponibile
- connessione Internet stabile
- un client SSH, oppure Cloud Shell dal browser

---

## Durata indicativa

**2 ore circa**

---

## Competenze in uscita

Al termine del laboratorio sarai in grado di:

- spiegare il ruolo di una VM in Azure
- leggere le informazioni principali di una macchina virtuale
- capire il significato di immagine, dimensione, disco, rete e IP
- collegarti a una VM Linux tramite SSH
- eseguire verifiche di sistema di base
- gestire il ciclo di vita elementare della VM

---

# PARTE 1 - Concetti fondamentali

# 1. Che cos’è una macchina virtuale

Una **macchina virtuale** è un computer simulato via software.

Dal punto di vista dell’utente, una VM si comporta come un normale computer o server:

- ha un sistema operativo
- ha CPU virtuali
- ha memoria RAM
- ha spazio disco
- ha interfacce di rete
- può eseguire programmi e servizi

La differenza è che non si tratta di una macchina fisica dedicata sulla tua scrivania.  
La VM gira sull’infrastruttura del provider cloud, in questo caso Azure.

---

# 2. VM, computer fisico e container: differenza essenziale

Questa distinzione è molto importante.

## 2.1 Computer fisico

Un computer fisico è una macchina reale:

- con hardware proprio
- con disco fisico
- con RAM fisica
- con una scheda di rete reale

È il livello materiale.

---

## 2.2 Macchina virtuale

Una macchina virtuale usa risorse virtualizzate.

Sembra un computer completo, ma in realtà sta usando risorse astratte da un’infrastruttura sottostante.

Per te, però, si comporta come un server vero e proprio:

- puoi accedervi
- puoi installare software
- puoi configurarla
- puoi riavviarla
- puoi spegnerla

---

## 2.3 Container

Un container non è una macchina completa.

In modo semplice:

- una VM contiene un intero sistema operativo
- un container contiene soprattutto un’applicazione e il necessario per eseguirla

Il container è più leggero, più rapido da distribuire e più focalizzato sull’applicazione.

La VM invece è più simile a un server tradizionale.

---

# 3. Perché imparare le VM prima dei container

Prima di usare container e servizi gestiti, è utile capire come funziona un server cloud tradizionale.

Questo ti aiuta a comprendere:

- il concetto di sistema operativo remoto
- l’accesso via rete
- la differenza tra IP privato e IP pubblico
- il ruolo del firewall
- il concetto di porta di rete aperta
- la gestione del ciclo di vita di una macchina

Quando poi passerai ai container, capirai meglio che cosa viene astratto e semplificato.

---

# 4. Componenti principali di una VM Azure

Quando crei una VM su Azure, non stai creando un solo oggetto “magico”.  
Dietro le quinte entrano in gioco più componenti.

A livello introduttivo devi conoscere almeno questi:

- **VM**: la macchina virtuale vera e propria
- **Image**: il sistema operativo di partenza
- **Size**: quantità di CPU e RAM
- **Disk**: disco del sistema operativo
- **Virtual network**: rete virtuale
- **Subnet**: sottorete interna
- **Network interface**: interfaccia di rete
- **Public IP**: indirizzo raggiungibile dall’esterno
- **NSG**: regole di sicurezza di rete

Non entrerai ancora in profondità su tutti questi elementi, ma è importante sapere che esistono.

---

# 5. Che cos’è un’immagine

L’**image** è il modello iniziale da cui viene creata la VM.

Per esempio puoi scegliere:

- Ubuntu Server
- Debian
- Red Hat
- Windows Server

L’immagine definisce il sistema operativo di base installato sulla macchina.

In questo laboratorio userai una VM Linux, ad esempio Ubuntu.

---

# 6. Che cos’è la size della VM

La **size** indica le caratteristiche computazionali della macchina virtuale.

In pratica definisce quante risorse avrà la VM, per esempio:

- numero di CPU virtuali
- quantità di memoria RAM

Per un laboratorio introduttivo userai una size piccola, sufficiente per fare test.

---

# 7. Disco OS e dati

Una VM ha almeno un disco di sistema operativo.

Su quel disco sono presenti:

- il sistema operativo
- i file di configurazione
- i pacchetti installati
- i file utente locali

In scenari più avanzati possono esistere anche dischi aggiuntivi dati.  
In questo laboratorio userai solo il disco essenziale.

---

# 8. Rete: IP privato e IP pubblico

Quando una VM viene creata, in genere ha almeno un indirizzo IP privato.

Questo serve per comunicare dentro la rete Azure.

Se vuoi raggiungerla da Internet o dal tuo browser/terminale esterno, spesso serve anche un **IP pubblico**.

In forma semplice:

- **IP privato** = usato dentro la rete interna
- **IP pubblico** = usato per raggiungere la macchina dall’esterno

---

# 9. SSH: che cos’è

**SSH** significa Secure Shell.

È un protocollo che permette di collegarsi in modo sicuro a un sistema remoto da riga di comando.

Con SSH puoi:

- aprire una sessione sulla VM
- eseguire comandi
- leggere file
- installare software
- amministrare il server

Per una VM Linux, SSH è il metodo standard di accesso amministrativo.

---

# 10. Security di base: porta 22

Per usare SSH, la VM deve consentire traffico sulla **porta 22**.

Questa apertura viene controllata da regole di sicurezza di rete.

Nel laboratorio userai un’impostazione minima e guidata, ma devi capire che:

- non tutto il traffico è aperto di default
- le porte devono essere autorizzate
- aprire una porta significa consentire una forma di accesso dall’esterno

---

# 11. Stato e ciclo di vita della VM

Una VM non è solo “accesa o spenta” in senso generico.

In Azure devi distinguere almeno questi concetti:

- **Running** = la VM è attiva
- **Stopped** = il sistema operativo è stato arrestato
- **Stopped (deallocated)** = la VM è stata fermata e le risorse di calcolo sono state rilasciate

Questa differenza è importante perché nel cloud il costo dipende spesso dal fatto che le risorse computazionali siano ancora allocate oppure no.

Per un beginner basta ricordare una regola pratica:

**Se non ti serve, arrestala correttamente dal portale Azure in modo che risulti deallocated.**

---

# 12. Cosa farai nella parte pratica

Nella parte pratica eseguirai queste attività:

1. verificherai la subscription attiva
2. verificherai il Resource Group
3. creerai una VM Linux dal portale
4. osserverai le risorse create insieme alla VM
5. individuerai l’IP pubblico
6. ti collegherai via SSH
7. eseguirai alcuni comandi Linux semplici
8. controllerai lo stato della VM da Azure CLI
9. arresterai e riavvierai la VM
10. osserverai il ciclo di vita e le evidenze amministrative

---

# PARTE 2 - Attività pratiche guidate

## Step 1 - Verifica la subscription attiva

Apri Cloud Shell in modalità Bash ed esegui:

```bash
az account show --output table
````

### Che cosa fa questo comando

* `az` richiama Azure CLI
* `account` indica che stai lavorando sulle informazioni dell’account
* `show` mostra il contesto attivo
* `--output table` visualizza il risultato in formato tabellare

### Che cosa devi osservare

Controlla:

* nome della subscription
* stato
* tenant ID
* utente
* subscription di default

### Perché lo fai

Prima di creare una VM devi essere sicuro di lavorare nella subscription corretta.

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
* `list` elenca i gruppi esistenti
* `--output table` rende il risultato leggibile

### Che cosa devi osservare

Individua il Resource Group che userai per la VM, ad esempio:

```text
rg-laba02-mario
```

oppure il nome assegnato nel laboratorio precedente.

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 3 - Apri il servizio Virtual machines nel portale

Nel portale Azure cerca:

```text
Virtual machines
```

Apri il servizio.

### Che cosa devi osservare

Guarda la schermata iniziale e individua:

* eventuale elenco di VM esistenti
* pulsante di creazione
* colonne come nome, resource group, location, status

### Obiettivo dello step

Prendere familiarità con la sezione del portale dedicata alle macchine virtuali.

### Evidenza richiesta

Esegui uno screenshot della pagina Virtual machines.

---

## Step 4 - Avvia la creazione della VM

Dalla pagina **Virtual machines**, clicca su **Create** e poi su **Azure virtual machine**.

Compila i campi principali in modo semplice e controllato.

### Impostazioni suggerite

#### Project details

* **Subscription**: seleziona la subscription del laboratorio
* **Resource group**: seleziona il tuo Resource Group

#### Instance details

* **Virtual machine name**: scegli un nome semplice, ad esempio:

```text
vm-laba04-<tuonome>
```

Esempio:

```text
vm-laba04-mario
```

* **Region**: scegli una regione disponibile, ad esempio `West Europe`
* **Image**: seleziona una distribuzione Linux, ad esempio `Ubuntu Server`
* **Size**: scegli una size piccola disponibile nel tuo ambiente

#### Administrator account

* **Authentication type**: per semplicità puoi usare **Password** se il laboratorio lo consente, oppure **SSH public key** se già disponibile
* **Username**: scegli un nome utente, ad esempio:

```text
azureuser
```

* **Password**: imposta una password conforme alle regole richieste dal portale, se usi autenticazione a password

#### Inbound port rules

* abilita **SSH (22)**

### Che cosa stai facendo

Stai definendo:

* il nome della VM
* il sistema operativo
* le risorse computazionali
* l’utente amministrativo
* l’accesso remoto di base

### Evidenza richiesta

Esegui uno screenshot della schermata di riepilogo prima della creazione oppure della VM appena creata.

---

## Step 5 - Attendi il completamento della creazione

Una volta confermata la creazione, attendi che Azure completi il provisioning.

### Cosa significa provisioning

Il provisioning è il processo automatico con cui Azure crea e collega gli elementi necessari alla VM.

### Che cosa viene creato dietro le quinte

In molti casi Azure crea o collega automaticamente anche:

* scheda di rete
* rete virtuale
* subnet
* IP pubblico
* disco OS
* regole di sicurezza necessarie

### Evidenza richiesta

Annota nel file evidenze che il provisioning è terminato con successo.

---

## Step 6 - Apri la VM e osserva i dettagli principali

Apri la VM appena creata.

### Che cosa devi osservare nella schermata principale

Individua almeno queste informazioni:

* nome della VM
* stato
* Resource Group
* region
* sistema operativo
* dimensione
* IP pubblico
* IP privato, se visibile

### Cosa devi capire

La VM non è solo “un server”.
È una risorsa Azure con proprietà precise e collegamenti a componenti di rete e storage.

### Evidenza richiesta

Esegui uno screenshot della pagina principale della VM.

---

## Step 7 - Osserva le risorse correlate create automaticamente

Dal Resource Group, apri l’elenco delle risorse contenute.

### Che cosa devi osservare

Controlla se oltre alla VM compaiono anche risorse come:

* public IP
* network interface
* virtual network
* network security group
* disk

### Cosa devi capire

Quando crei una VM, spesso Azure crea anche altre risorse di supporto.

### Evidenza richiesta

Esegui uno screenshot dell’elenco risorse nel Resource Group.

---

## Step 8 - Recupera l’IP pubblico della VM

Nella schermata principale della VM individua il valore di **Public IP address**.

Annotalo nel file delle evidenze.

Esempio:

```text
20.x.x.x
```

### Perché è importante

Questo indirizzo ti serve per raggiungere la VM dall’esterno tramite SSH.

---

## Step 9 - Collegati in SSH dalla Cloud Shell

In Cloud Shell esegui un comando di questo tipo, sostituendo username e IP con i tuoi valori reali:

```bash
ssh azureuser@<IP_PUBBLICO>
```

Esempio:

```bash
ssh azureuser@20.120.10.25
```

### Che cosa fa questo comando

* `ssh` avvia una connessione Secure Shell
* `azureuser` è l’utente remoto
* `@` separa utente e host
* `<IP_PUBBLICO>` è l’indirizzo della VM

### Prima connessione

Alla prima connessione potresti vedere un messaggio che chiede di confermare l’identità dell’host.

Se sei sicuro che l’IP è corretto, digita:

```text
yes
```

Se stai usando autenticazione con password, inserisci poi la password della VM.

### Evidenza richiesta

Copia nel file delle evidenze il comando usato e annota se la connessione è riuscita.

---

## Step 10 - Verifica dove ti trovi nella VM

Una volta entrato nella VM, esegui:

```bash
whoami
```

### Che cosa fa questo comando

Mostra l’utente con cui sei autenticato.

### Che cosa devi osservare

Dovresti vedere il nome utente creato per la VM, ad esempio:

```text
azureuser
```

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 11 - Verifica il nome host della VM

Esegui:

```bash
hostname
```

### Che cosa fa questo comando

Mostra il nome host della macchina.

### Perché è utile

Conferma che stai lavorando dentro una macchina Linux reale nel cloud.

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 12 - Verifica la distribuzione Linux installata

Esegui:

```bash
cat /etc/os-release
```

### Che cosa fa questo comando

* `cat` visualizza il contenuto di un file di testo
* `/etc/os-release` contiene informazioni sulla distribuzione Linux installata

### Che cosa devi osservare

Cerca campi come:

* `NAME`
* `VERSION`
* `PRETTY_NAME`

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 13 - Verifica data e ora della VM

Esegui:

```bash
date
```

### Che cosa fa questo comando

Mostra data e ora del sistema.

### Perché è utile

Ti aiuta a capire che la VM è attiva e sta eseguendo un sistema operativo reale.

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 14 - Verifica indirizzi di rete dalla VM

Esegui:

```bash
ip a
```

### Che cosa fa questo comando

* `ip` è uno strumento di rete Linux
* `a` significa `address`
* mostra le interfacce di rete e gli indirizzi associati

### Che cosa devi osservare

Prova a individuare:

* interfaccia di rete principale
* indirizzo IP privato interno

### Cosa devi capire

Dentro la VM vedrai normalmente l’IP privato, non l’IP pubblico come interfaccia locale.

### Evidenza richiesta

Copia almeno la parte iniziale dell’output nel file delle evidenze.

---

## Step 15 - Verifica lo spazio disco

Esegui:

```bash
df -h
```

### Che cosa fa questo comando

* `df` mostra l’utilizzo dei filesystem
* `-h` visualizza le dimensioni in formato leggibile

### Che cosa devi osservare

Controlla:

* filesystem principale
* spazio totale
* spazio usato
* spazio disponibile

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 16 - Crea un file di test nella VM

Esegui:

```bash
echo "File creato nella VM del laboratorio LABA04" > file_laba04.txt
```

### Spiegazione del comando

* `echo` stampa il testo
* `>` reindirizza l’output in un file
* `file_laba04.txt` è il file creato nella directory corrente

Poi verifica il contenuto con:

```bash
cat file_laba04.txt
```

### Che cosa stai facendo

Stai creando un file direttamente nella VM per verificare che il sistema sia operativo.

### Evidenza richiesta

Copia il comando e l’output di `cat` nel file delle evidenze.

---

## Step 17 - Esci dalla VM

Per uscire dalla sessione SSH esegui:

```bash
exit
```

### Che cosa fa questo comando

Chiude la sessione remota e ti riporta alla Cloud Shell.

### Evidenza richiesta

Annota nel file evidenze che la sessione SSH è stata chiusa correttamente.

---

## Step 18 - Verifica lo stato della VM da Azure CLI

Tornato in Cloud Shell, esegui il comando seguente sostituendo il nome corretto della VM e del Resource Group:

```bash
az vm show --resource-group <NOME_RESOURCE_GROUP> --name <NOME_VM> --show-details --output table
```

Esempio:

```bash
az vm show --resource-group rg-laba02-mario --name vm-laba04-mario --show-details --output table
```

### Che cosa fa questo comando

* `vm` indica che lavori sulle macchine virtuali
* `show` chiede i dettagli della VM
* `--resource-group` specifica dove si trova la VM
* `--name` specifica quale VM vuoi interrogare
* `--show-details` aggiunge dettagli operativi
* `--output table` mostra i dati in tabella

### Che cosa devi osservare

Controlla elementi come:

* nome
* power state
* public IP
* location

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 19 - Arresta la VM dal portale

Vai nel portale Azure, apri la VM e clicca su **Stop**.

Conferma l’operazione se richiesto.

### Che cosa stai facendo

Stai arrestando la VM in modo gestito da Azure.

### Che cosa devi osservare

Attendi che lo stato cambi.

Idealmente devi vedere uno stato che indichi che la VM non è più in esecuzione.

### Evidenza richiesta

Esegui uno screenshot della VM dopo l’arresto oppure annota lo stato mostrato.

---

## Step 20 - Verifica di nuovo lo stato da CLI

Esegui di nuovo:

```bash
az vm show --resource-group <NOME_RESOURCE_GROUP> --name <NOME_VM> --show-details --output table
```

### Che cosa devi osservare

Controlla se il power state è cambiato rispetto a prima.

### Cosa devi capire

Il ciclo di vita della VM può essere osservato sia dal portale sia da CLI.

### Evidenza richiesta

Copia l’output nel file delle evidenze.

---

## Step 21 - Riavvia o riaccendi la VM dal portale

Dal portale clicca su **Start** per riaccendere la VM.

Attendi che lo stato torni operativo.

### Evidenza richiesta

Annota nel file evidenze che la VM è stata riavviata correttamente.

---

## Step 22 - Verifica l’Activity Log

Nel portale Azure cerca:

```text
Activity Log
```

Apri il servizio.

### Che cosa osservare

Prova a individuare eventi legati a:

* creazione della VM
* arresto della VM
* riavvio o start della VM

### Cosa devi capire

L’Activity Log registra le operazioni amministrative eseguite sulle risorse Azure.

### Evidenza richiesta

Esegui uno screenshot dell’Activity Log con almeno un evento collegato alla VM.

---

## Step 23 - Riepiloga mentalmente la struttura

A questo punto devi saper leggere una struttura come questa:

```text
Subscription
 └── Resource Group
      └── Virtual Machine
      ├── Network Interface
      ├── Public IP
      ├── Disk
      └── Virtual Network / Subnet
```

Esempio concreto:

```text
Subscription: Azure for Students
Resource Group: rg-laba02-mario
VM: vm-laba04-mario
User: azureuser
Public IP: 20.x.x.x
OS: Ubuntu Server
```

---

# 3. Mini esercizio finale

Rispondi per iscritto alle seguenti domande.

## Domanda 1

Che differenza c’è tra una macchina virtuale e un container?

## Domanda 2

Perché per collegarti in SSH hai bisogno dell’IP pubblico e della porta 22 aperta?

## Domanda 3

Qual è la differenza tra IP pubblico e IP privato?

## Domanda 4

Perché è importante arrestare correttamente una VM quando non serve?

## Domanda 5

Quali risorse Azure, oltre alla VM, possono essere create o collegate automaticamente durante il provisioning?

---

# 4. Output attesi

Al termine del laboratorio devi aver prodotto le seguenti evidenze:

1. screenshot della pagina **Virtual machines**
2. screenshot della **VM** creata
3. screenshot dell’elenco risorse nel **Resource Group**
4. screenshot dell’**Activity Log**
5. output del comando:

```bash
az account show --output table
```

6. output del comando:

```bash
az group list --output table
```

7. comando SSH usato per collegarti alla VM:

```bash
ssh azureuser@<IP_PUBBLICO>
```

8. output del comando:

```bash
whoami
```

9. output del comando:

```bash
hostname
```

10. output del comando:

```bash
cat /etc/os-release
```

11. output del comando:

```bash
date
```

12. output del comando:

```bash
ip a
```

13. output del comando:

```bash
df -h
```

14. comando usato per creare il file di test:

```bash
echo "File creato nella VM del laboratorio LABA04" > file_laba04.txt
```

15. output del comando:

```bash
cat file_laba04.txt
```

16. output del comando:

```bash
az vm show --resource-group <NOME_RESOURCE_GROUP> --name <NOME_VM> --show-details --output table
```

17. secondo output del comando precedente dopo l’arresto della VM
18. risposte scritte alle 5 domande finali

---

# 5. Modello di file evidenze

Crea un file chiamato:

```text
evidenze_laba04.md
```

e usa una struttura come questa:

```md
# Evidenze LABA04

## 1. VM
- Nome VM:
- Resource Group:
- Regione:
- Sistema operativo:
- Size:
- Public IP:
- Username amministrativo:

## 2. Output CLI / SSH

### az account show --output table
[incollare qui l’output]

### az group list --output table
[incollare qui l’output]

### ssh azureuser@<IP_PUBBLICO>
[annotare il comando usato]

### whoami
[incollare qui l’output]

### hostname
[incollare qui l’output]

### cat /etc/os-release
[incollare qui l’output]

### date
[incollare qui l’output]

### ip a
[incollare qui l’output]

### df -h
[incollare qui l’output]

### echo "File creato nella VM del laboratorio LABA04" > file_laba04.txt
[annotare il comando eseguito]

### cat file_laba04.txt
[incollare qui l’output]

### az vm show --resource-group <NOME_RESOURCE_GROUP> --name <NOME_VM> --show-details --output table
[incollare qui il primo output]

### az vm show --resource-group <NOME_RESOURCE_GROUP> --name <NOME_VM> --show-details --output table
[incollare qui il secondo output dopo lo stop]

## 3. Risorse correlate osservate
- Network Interface:
- Public IP:
- Disk:
- Virtual Network:
- Eventuali altre risorse:

## 4. Risposte finali

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

## Errore 1 - Pensare che la VM sia l’unica risorsa coinvolta

Correzione:

Una VM usa spesso anche altre risorse collegate, come IP pubblico, scheda di rete, disco e rete virtuale.

---

## Errore 2 - Confondere VM e container

Correzione:

La VM è un sistema completo con sistema operativo.
Il container è più leggero e contiene soprattutto l’applicazione e ciò che serve per eseguirla.

---

## Errore 3 - Non annotare l’IP pubblico corretto

Correzione:

Per l’accesso SSH serve l’IP pubblico giusto della VM.

---

## Errore 4 - Dimenticare credenziali o username

Correzione:

Annota subito username amministrativo e metodo di autenticazione usato.

---

## Errore 5 - Non capire la differenza tra arresto locale e deallocazione cloud

Correzione:

Nel cloud è importante fermare la VM dal portale Azure o con gli strumenti Azure in modo da rilasciare correttamente le risorse di calcolo quando non servono.

---

# 7. Conclusione

Questo laboratorio ti ha introdotto a uno dei concetti più importanti dell’infrastruttura cloud: la **macchina virtuale**.

Dopo aver completato il lab, devi avere chiaro che:

* una VM è un server virtuale completo
* per crearla servono anche componenti di rete, disco e sicurezza
* l’accesso Linux avviene normalmente tramite SSH
* una VM ha un ciclo di vita che va gestito con attenzione
* il portale Azure e Azure CLI permettono di osservare e amministrare la stessa risorsa in modi diversi

Queste basi saranno molto utili nel laboratorio successivo, in cui entrerà in gioco il deployment di un’applicazione su Azure.

```
```
