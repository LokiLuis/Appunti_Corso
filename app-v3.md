# app-v3.py

Questa è la versione standalone dell'applicazione usata dal **LAB12 standalone**.

Caratteristiche principali:

- endpoint `GET /`
- endpoint `GET /health`
- endpoint `GET /time`
- endpoint `POST /echo`
- endpoint `GET /error`
- logging applicativo strutturato in **JSON su stdout**
- campo `status` sempre presente nel log applicativo
- `request_id` generato automaticamente o letto da header `X-Request-Id`
- log compatibili con query KQL basate su `parse_json(Message)` nella tabella `ContainerInstanceLog_CL`

---

## Contenuto del file `app.py`

```python
import json
import os
import time
import uuid
from datetime import datetime, timezone

from flask import Flask, g, jsonify, request
from werkzeug.exceptions import HTTPException

app = Flask(__name__)


def utc_now_iso() -> str:
    return datetime.now(timezone.utc).isoformat()


def write_log(status_code: int, message: str = "request_completed") -> None:
    latency_ms = round((time.perf_counter() - g.start_time) * 1000, 2)

    log_record = {
        "timestamp": utc_now_iso(),
        "level": "INFO" if status_code < 400 else "ERROR",
        "message": message,
        "request_id": g.request_id,
        "method": request.method,
        "path": request.path,
        "status": int(status_code),
        "latency_ms": latency_ms,
        "client_ip": request.headers.get("X-Forwarded-For", request.remote_addr),
        "user_agent": request.headers.get("User-Agent"),
    }

    print(json.dumps(log_record), flush=True)


@app.before_request
def before_request() -> None:
    g.start_time = time.perf_counter()
    g.request_id = request.headers.get("X-Request-Id", str(uuid.uuid4()))


@app.after_request
def after_request(response):
    write_log(response.status_code)
    response.headers["X-Request-Id"] = g.request_id
    return response


@app.errorhandler(HTTPException)
def handle_http_exception(exc: HTTPException):
    response = jsonify(
        {
            "error": exc.name.lower().replace(" ", "_"),
            "status": exc.code,
            "path": request.path,
        }
    )
    response.status_code = exc.code
    return response


@app.errorhandler(Exception)
def handle_generic_exception(exc: Exception):
    response = jsonify(
        {
            "error": "internal_server_error",
            "status": 500,
            "path": request.path,
        }
    )
    response.status_code = 500
    return response


@app.get("/")
def home():
    return jsonify(
        {
            "app": "obsapp",
            "version": "v3",
            "status": "running",
            "timestamp": utc_now_iso(),
        }
    ), 200


@app.get("/health")
def health():
    return jsonify({"status": "ok", "timestamp": utc_now_iso()}), 200


@app.get("/time")
def current_time():
    return jsonify({"time": utc_now_iso()}), 200


@app.post("/echo")
def echo():
    payload = request.get_json(silent=True)
    if payload is None:
        return jsonify({"error": "invalid_json", "status": 400}), 400
    return jsonify({"received": payload, "status": 200}), 200


@app.get("/error")
def simulated_error():
    return jsonify({"error": "simulated_error", "status": 500}), 500


if __name__ == "__main__":
    port = int(os.getenv("PORT", "8000"))
    app.run(host="0.0.0.0", port=port)
```

---

## Perché questa versione è adatta al LAB12 standalone

Ogni richiesta genera una riga JSON su stdout con campi come:

- `status`
- `path`
- `method`
- `latency_ms`
- `request_id`

Di conseguenza, nel workspace Log Analytics è possibile usare query del tipo:

```kql
ContainerInstanceLog_CL
| extend payload = parse_json(Message)
| extend status = toint(payload.status)
| where isnotnull(status)
| summarize error_rate = todouble(countif(status >= 400)) / todouble(count())
```

---

## Test locale rapido facoltativo

Dopo aver creato `app.py` e installato le dipendenze:

```bash
python app.py
```

In un secondo terminale:

```bash
curl http://127.0.0.1:8000/
curl http://127.0.0.1:8000/health
curl http://127.0.0.1:8000/time
curl http://127.0.0.1:8000/nope
curl http://127.0.0.1:8000/error
curl -X POST http://127.0.0.1:8000/echo -H 'Content-Type: application/json' -d '{"hello":"world"}'
```

Vedrai sul terminale dell'applicazione una riga JSON per ogni richiesta.
