# FBI Wanted Search — v4

Un'applicazione web full-stack per cercare e confrontare persone nel database ufficiale degli **FBI Most Wanted**, con sistema di autenticazione utenti e storico delle segnalazioni.

---

## Descrizione

FBI Wanted Search consente di:

- **Cercare** soggetti ricercati dall'FBI tramite filtri (nome, razza, sesso, capelli, occhi, ufficio distrettuale, fascia d'età).
- **Inviare una segnalazione** descrivendo le caratteristiche fisiche di un soggetto sospetto: l'API interroga il database FBI e restituisce i profili più compatibili con un punteggio di corrispondenza.
- **Registrarsi e accedere** con un account personale per salvare e consultare lo storico delle proprie segnalazioni.

---

## Struttura del progetto

```
.
├── main.py            # Backend FastAPI (API REST + logica di matching)
├── index.html         # Frontend single-page (servito direttamente da FastAPI)
├── fbi_search.db      # Database SQLite (creato automaticamente all'avvio)
├── requirements.txt   # Dipendenze Python
└── README.md
```

---

## Requisiti

- Python **3.10+**
- Connessione internet (le ricerche vengono effettuate in tempo reale sull'API pubblica `api.fbi.gov`)

---

## Installazione e avvio

```bash
# 1. Clona il repository
git clone <url-del-repo>
cd fbi-wanted-search

# 2. (Opzionale) Crea un ambiente virtuale
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\activate           # Windows

# 3. Installa le dipendenze
pip install -r requirements.txt

# 4. Avvia il server
python main.py
```

Il server si avvierà su **http://127.0.0.1:8000**.

| URL | Descrizione |
|-----|-------------|
| `http://127.0.0.1:8000` | Interfaccia web |
| `http://127.0.0.1:8000/docs` | Documentazione interattiva (Swagger UI) |

---

## Endpoint API

### Autenticazione

| Metodo | Endpoint | Descrizione |
|--------|----------|-------------|
| `POST` | `/auth/register` | Registra un nuovo utente |
| `POST` | `/auth/login` | Login e ottenimento del token Bearer |
| `POST` | `/auth/logout` | Logout (invalida il token) |
| `GET`  | `/auth/me` | Dati dell'utente corrente |

### Ricerca

| Metodo | Endpoint | Descrizione |
|--------|----------|-------------|
| `GET`  | `/search` | Ricerca soggetti FBI per filtri |
| `POST` | `/sighting` | Segnalazione con matching fisico automatico |
| `GET`  | `/reports/my` | Storico segnalazioni dell'utente autenticato |

### Parametri di `/search`

| Parametro | Tipo | Descrizione |
|-----------|------|-------------|
| `title` | string | Nome o alias del soggetto |
| `race` | string | Razza (es. `White`, `Black`, `Hispanic`) |
| `sex` | string | Sesso (`Male` / `Female`) |
| `hair` | string | Colore capelli |
| `eyes` | string | Colore occhi |
| `field_offices` | string | Ufficio FBI distrettuale |
| `age_min` / `age_max` | int | Intervallo di età |
| `page` | int | Pagina dei risultati (default: 1) |

### Body di `/sighting`

```json
{
  "reporter_name": "Mario Rossi",
  "suspect_name": "John Doe",
  "race": "White",
  "sex": "Male",
  "hair": "Brown",
  "eyes": "Blue",
  "age_approx": 35,
  "age_tolerance": 8,
  "height_ft": 5,
  "height_in": 11,
  "weight_lbs": 180,
  "weight_tolerance": 20,
  "location": "New York, NY",
  "field_office": "new_york",
  "date_seen": "2024-01-15",
  "notes": "Note aggiuntive"
}
```

---

## Algoritmo di matching

Il sistema assegna un **punteggio percentuale** a ogni soggetto FBI in base alle caratteristiche fisiche fornite:

| Caratteristica | Punteggio massimo |
|----------------|:-----------------:|
| Razza | 25 pt |
| Sesso | 20 pt |
| Capelli | 15 pt |
| Occhi | 15 pt |
| Età (con tolleranza) | 15 pt |
| Altezza (±3 in) | 5 pt |
| Peso (con tolleranza) | 5 pt |

Vengono restituiti al massimo i **10 migliori risultati**; i profili con punteggio inferiore al 30% vengono esclusi se si hanno già almeno 3 risultati.

---

##  Database

Il file `fbi_search.db` (SQLite) viene creato automaticamente al primo avvio. Contiene due tabelle:

- **`users`** — credenziali e dati degli utenti registrati
- **`reports`** — storico delle segnalazioni con i relativi risultati di matching

---

## Autenticazione

L'autenticazione è basata su **token Bearer** (UUID v4) conservati in memoria sul server. Le sessioni non sono persistenti tra riavvii del server.

Includi il token in ogni richiesta protetta:

```
Authorization: Bearer <token>
```

---

## Dipendenze

| Pacchetto | Utilizzo |
|-----------|----------|
| `fastapi` | Framework web API |
| `uvicorn` | Server ASGI |
| `httpx` | Client HTTP async per le chiamate a `api.fbi.gov` |
| `pydantic` | Validazione dei dati |

---

## Note

- I dati provengono dall'**API pubblica ufficiale dell'FBI** (`api.fbi.gov`) e sono aggiornati in tempo reale.
- Le sessioni utente sono **in-memory**: un riavvio del server invalida tutti i token attivi.
- Per ambienti di produzione si raccomanda di sostituire l'hashing SHA-256 delle password con **bcrypt** o **argon2** e di persistere le sessioni.

---

## Licenza

Questo progetto è distribuito a scopo educativo e dimostrativo. I dati FBI sono proprietà del Federal Bureau of Investigation.
