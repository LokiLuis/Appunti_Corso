# LAB05 – Mini-servizio HTTP locale: endpoint + log strutturati + request-id (monorepo)

**Obiettivo:** costruire un micro-servizio HTTP locale (Python) con logging strutturato e request-id, testarlo, indurre guasti tipici e produrre evidenze ripetibili.

---

## 1. Obiettivi del laboratorio
Al termine del LAB05 dovrai essere in grado di:
- Capire differenza tra processo, servizio ed endpoint HTTP
- Avviare un servizio su una porta e verificarne stato (`ss`, `curl`)
- Gestire errori comuni: porta occupata, endpoint 404, input non valido (400)
- Generare log strutturati (JSON) con `request_id`
- Correlare una richiesta a una riga di log tramite `request_id`
- Automatizzare test con uno script `requests.sh`

---

## 2. Dove lavorare (nuova alberatura)
**Terminale:** Ubuntu (WSL)

Posizionati nella cartella del LAB05:

```bash
cd ~/corso_obs/NOME-REPOSITORY/labs/lab05
pwd
ls -la
```

Struttura:
- codice in `src/`
- evidenze in `docs/`
- log runtime in `logs/` (ignorati da git)

---

## 3. Prerequisite Check
```bash
cd ~/corso_obs/NOME-REPOSITORY/labs/lab05
python3 --version
which python3 curl ss
```

Se manca qualcosa:

```bash
sudo apt update
sudo apt install -y python3 curl iproute2
```

---

## 4. Regola operativa
- Il servizio si avvia **dalla root del lab** (`labs/lab05`).
- I log finiscono in `logs/app.log` (non versionato).
- Nel report incollerai estratti di log e request-id (non committare il log).

---

## 5. Creazione app (codice) in src/app.py
Creare `src/app.py` (sovrascrivere se già presente):

```python
import json
import os
import time
import uuid
from datetime import datetime
from http.server import BaseHTTPRequestHandler, HTTPServer

PORT = int(os.environ.get("PORT", "9000"))
LOG_PATH = os.environ.get("LOG_PATH", "logs/app.log")

def log_line(level, request_id, method, path, status, duration_ms, extra=None):
    record = {
        "ts": datetime.utcnow().isoformat() + "Z",
        "level": level,
        "request_id": request_id,
        "method": method,
        "path": path,
        "status": status,
        "duration_ms": duration_ms,
    }
    if extra:
        record.update(extra)

    os.makedirs(os.path.dirname(LOG_PATH), exist_ok=True)
    with open(LOG_PATH, "a", encoding="utf-8") as f:
        f.write(json.dumps(record) + "\n")

class Handler(BaseHTTPRequestHandler):
    def _send_json(self, status, obj, request_id, start_ts, level="INFO", extra=None):
        payload = json.dumps(obj).encode("utf-8")
        self.send_response(status)
        self.send_header("Content-Type", "application/json")
        self.send_header("X-Request-Id", request_id)
        self.send_header("Content-Length", str(len(payload)))
        self.end_headers()
        self.wfile.write(payload)

        duration_ms = int((time.time() - start_ts) * 1000)
        log_line(level, request_id, self.command, self.path, status, duration_ms, extra=extra)

    def do_GET(self):
        start_ts = time.time()
        request_id = str(uuid.uuid4())

        if self.path == "/health":
            self._send_json(200, {"status": "ok"}, request_id, start_ts)
            return

        if self.path == "/time":
            self._send_json(200, {"utc": datetime.utcnow().isoformat() + "Z"}, request_id, start_ts)
            return

        self._send_json(404, {"error": "not found", "path": self.path}, request_id, start_ts)

    def do_POST(self):
        start_ts = time.time()
        request_id = str(uuid.uuid4())

        if self.path != "/echo":
            self._send_json(404, {"error": "not found", "path": self.path}, request_id, start_ts)
            return

        try:
            length = int(self.headers.get("Content-Length", "0"))
            raw = self.rfile.read(length).decode("utf-8")
            data = json.loads(raw) if raw else None
        except Exception:
            self._send_json(400, {"error": "bad json"}, request_id, start_ts, level="WARN", extra={"error": "bad_json"})
            return

        self._send_json(200, {"echo": data}, request_id, start_ts)

def main():
    server = HTTPServer(("0.0.0.0", PORT), Handler)
    print(f"Serving on http://localhost:{PORT}")
    server.serve_forever()

if __name__ == "__main__":
    main()
```

Checkpoint: `src/app.py` esiste e contiene endpoint `/health`, `/time`, `/echo`.

---

## 6. Avvio servizio e verifica porta (2 terminali)

### Terminale 1 (lascia il servizio in esecuzione)
```bash
cd ~/corso_obs/NOME-REPOSITORY/labs/lab05
mkdir -p logs
PORT=9000 LOG_PATH="logs/app.log" python3 src/app.py
```

### Terminale 2 (diagnostica e test)
```bash
cd ~/corso_obs/NOME-REPOSITORY/labs/lab05
ss -ltnp | grep 9000
```

Se la porta 9000 è occupata, usa una porta alternativa (es. 9100) nel Terminale 1:

```bash
PORT=9100 LOG_PATH="logs/app.log" python3 src/app.py
```

---

## 7. Test base con curl
Sostituire PORT se usi una porta diversa.

```bash
curl -i http://localhost:9000/health
curl -i http://localhost:9000/time
curl -i -X POST http://localhost:9000/echo -H 'Content-Type: application/json' -d '{"msg":"ciao"}'
tail -n 5 logs/app.log
```

Checkpoint: ogni risposta include `X-Request-Id` e nel log compare una riga JSON per richiesta.

---

## 8. Correlazione request-id → log (2 esempi)
Copia un request-id dagli header e cercalo nel log:

```bash
RID="INCOLLA-QUI"
grep "$RID" logs/app.log
```

Ripetere con un secondo request-id.

---

## 9. Scenario pack (3 guasti) – obbligatorio

### Scenario A – Porta occupata
Simula un servizio sulla stessa porta (in un terminale diverso):

```bash
python3 -m http.server 9000
```

Identifica il listener:

```bash
ss -ltnp | grep 9000
```

Libera la porta (PID mirato):

```bash
kill <PID>
```

Riavvia `src/app.py` e verifica `/health`.

Nel report: sintomo → causa → test → fix → evidenza.

### Scenario B – 404 endpoint inesistente
```bash
curl -i http://localhost:9000/nope
```

Copia `X-Request-Id` e cercalo nel log con `grep`.

### Scenario C – 400 input non valido (bad json)
```bash
curl -i -X POST http://localhost:9000/echo -H 'Content-Type: application/json' -d '{"msg":'
```

Copia `X-Request-Id` e cercalo nel log. Verifica:
- status 400
- log con `level` = WARN e `error` = bad_json

---

## 10. Script di test src/requests.sh (automatizza 5 richieste)
Creare `src/requests.sh`:

```bash
cat > src/requests.sh << 'EOF'
#!/bin/bash
set -e

PORT="${PORT:-9000}"
BASE="http://localhost:${PORT}"
OUT="docs/responses_lab05.txt"

mkdir -p docs
: > "$OUT"

echo "== health" | tee -a "$OUT"
curl -is "$BASE/health" | tee -a "$OUT"

echo "== time" | tee -a "$OUT"
curl -is "$BASE/time" | tee -a "$OUT"

echo "== echo ok" | tee -a "$OUT"
curl -is -X POST "$BASE/echo" -H 'Content-Type: application/json' -d '{"msg":"ciao"}' | tee -a "$OUT"

echo "== 404" | tee -a "$OUT"
curl -is "$BASE/nope" | tee -a "$OUT"

echo "== bad json" | tee -a "$OUT"
curl -is -X POST "$BASE/echo" -H 'Content-Type: application/json' -d '{"msg":' | tee -a "$OUT"

echo "DONE. Saved to $OUT"
EOF

chmod 755 src/requests.sh
```

Eseguire:

```bash
PORT=9000 src/requests.sh
tail -n 30 docs/responses_lab05.txt
```

Checkpoint: `docs/responses_lab05.txt` contiene 5 risposte e in ciascuna compare `X-Request-Id`.

---

## 11. Evidenza nel repository
Creare:

```
docs/evidence_lab05.md
```

Struttura obbligatoria:

```markdown
# LAB05 – Evidence (Mini-servizio HTTP)

## Setup e avvio servizio
(porta, PID, ss)

## Test base (health, time, echo)
(output curl + request-id)

## Correlazione request-id → log (2 esempi)
(request-id + riga JSON dal log)

## Scenario A: porta occupata
(sintomo/causa/test/fix/evidenza)

## Scenario B: 404 endpoint non esistente
(request-id + log)

## Scenario C: 400 input non valido (echo)
(request-id + log)

## Script requests.sh
(conferma file docs/responses_lab05.txt)

## Note finali
(cosa ho imparato / cosa resta confuso)
```

Nel report incolla:
- almeno 3 righe JSON da `logs/app.log`
- almeno 2 correlazioni complete request-id → riga log

---

## 12. Consegna
Assicurati che il servizio sia fermato (CTRL+C nel Terminale 1).

Poi:

```bash
cd ~/corso_obs/NOME-REPOSITORY/labs/lab05
git status
git add src/app.py src/requests.sh docs/responses_lab05.txt docs/evidence_lab05.md
git commit -m "[LAB05] Mini-servizio e logging completato"
git push
```

---

## 13. Criteri di completamento
Il LAB05 è completato quando:
- `/health` e `/time` rispondono 200 con JSON
- `POST /echo` risponde 200 e ritorna il payload
- Ogni richiesta produce log JSON con request_id
- Sono completati i 3 scenari guasto con evidenze
- `docs/evidence_lab05.md` + `docs/responses_lab05.txt` sono presenti su GitHub
- Il commit è visibile
