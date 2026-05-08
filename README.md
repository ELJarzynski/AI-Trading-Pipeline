# 📈 AI Trading Pipeline

Aplikacja do automatycznego tradingu oparta na LLM — analizuje spółki przez API, generuje raporty i wysyła je mailem.

---

## 🏗️ Architektura

```
[Źródła danych API] → [Scraper/Fetcher] → [LLM Analyst] → [Email Dispatcher]
       ↓                      ↓                  ↓                 ↓
  Market Data API        FastAPI + AWS       Claude/GPT         AWS SES
  (Yahoo, Alpha         Lambda + S3        (analiza spółek,    (wysyłka
   Vantage, etc.)       (przechowywanie)    generowanie        raportów)
                                            raportów)
```

---

## 🗺️ Pipeline — etapy

### Etap 1 — Data Ingestion
> Pobieranie danych rynkowych z zewnętrznych API

- Integracja z Market Data API (np. Alpha Vantage, Yahoo Finance, Polygon.io)
- Filtrowanie spółek wg kryteriów (wolumen, kapitalizacja, branża)
- Zapis surowych danych do S3 / bazy (PostgreSQL lub DynamoDB)
- Obsługa limitów API (rate limiting, retry logic)

**Output:** `raw_data/{date}/companies.json`

---

### Etap 2 — Company Scoring
> Rankingowanie i selekcja najlepszych spółek

- Obliczanie wskaźników: P/E, ROE, momentum, RSI, MACD
- Scoring model (wagi konfigurowane przez użytkownika)
- Filtrowanie Top-N spółek do dalszej analizy
- Zapis shortlisty do bazy

**Output:** `scored_data/{date}/top_companies.json`

---

### Etap 3 — LLM Analysis
> Generowanie raportów przez model językowy

- Przygotowanie promptów z danymi spółek
- Wywołanie API (Claude / OpenAI)
- Generowanie strukturyzowanego raportu (analiza, ryzyko, potencjał)
- Walidacja i parsowanie odpowiedzi LLM

**Output:** `reports/{date}/report_{ticker}.md`

---

### Etap 4 — Email Dispatch
> Wysyłka raportów do subskrybentów

- Formatowanie raportu do HTML
- Wysyłka przez AWS SES
- Logowanie statusu wysyłki
- Obsługa błędów i ponowień

**Output:** Email do subskrybentów + log w CloudWatch

---

## 🎯 Milestones

### Milestone 1 — MVP Infrastructure (tydzień 1–2)
- [ ] Setup projektu: FastAPI + struktura katalogów
- [ ] Konfiguracja AWS (S3, Lambda, SES, IAM)
- [ ] Integracja z pierwszym Market Data API
- [ ] Endpoint `POST /run-pipeline` (trigger manualny)
- [ ] Podstawowe logi (CloudWatch)

### Milestone 2 — Data Layer (tydzień 3–4)
- [ ] Moduł pobierania danych: `fetcher.py`
- [ ] Scoring model v1 (podstawowe wskaźniki)
- [ ] Zapis i odczyt z S3 / bazy danych
- [ ] Testy jednostkowe dla data layer
- [ ] Obsługa błędów API i rate limiting

### Milestone 3 — LLM Integration (tydzień 5–6)
- [ ] Moduł `llm_analyst.py` z prompt templates
- [ ] Integracja z Claude API lub OpenAI
- [ ] Generowanie i walidacja raportów
- [ ] Retry logic dla błędów LLM
- [ ] Testy integracyjne LLM → raport

### Milestone 4 — Email & Notifications (tydzień 7)
- [ ] Moduł `email_dispatcher.py`
- [ ] Template HTML dla raportów
- [ ] Konfiguracja AWS SES (domeny, DKIM)
- [ ] Lista subskrybentów (DB lub env config)
- [ ] Testy wysyłki e2e

### Milestone 5 — Automation & Scheduling (tydzień 8)
- [ ] AWS EventBridge (cron trigger, np. codziennie o 7:00)
- [ ] Lambda handler dla pełnego pipeline
- [ ] Monitoring i alerty (CloudWatch Alarms)
- [ ] Dokumentacja API (FastAPI /docs)
- [ ] Deploy na produkcję

---

## 🛠️ Tech Stack

| Warstwa          | Technologia                        |
|------------------|------------------------------------|
| Backend          | Python 3.11+, FastAPI              |
| Cloud            | AWS (Lambda, S3, SES, EventBridge) |
| LLM              | Claude API / OpenAI API            |
| Dane rynkowe     | Alpha Vantage / Polygon.io         |
| Baza danych      | PostgreSQL (RDS) lub DynamoDB      |
| Testy            | pytest, moto (AWS mock)            |
| CI/CD            | GitHub Actions                     |

---

## 📁 Struktura projektu

```
trading-pipeline/
├── app/
│   ├── main.py              # FastAPI entry point
│   ├── fetcher.py           # Data ingestion
│   ├── scorer.py            # Company scoring
│   ├── llm_analyst.py       # LLM report generation
│   ├── email_dispatcher.py  # AWS SES email
│   └── config.py            # Ustawienia (env vars)
├── prompts/
│   └── company_report.txt   # Prompt template
├── tests/
│   ├── test_fetcher.py
│   ├── test_scorer.py
│   └── test_llm.py
├── infra/
│   └── lambda_handler.py    # AWS Lambda entry
├── .env.example
├── requirements.txt
└── README.md
```

---

## ⚙️ Zmienne środowiskowe

```env
# API Keys
MARKET_DATA_API_KEY=your_key
ANTHROPIC_API_KEY=your_key

# AWS
AWS_REGION=eu-central-1
S3_BUCKET_NAME=trading-pipeline-data
SES_SENDER_EMAIL=reports@yourdomain.com

# Config
TOP_COMPANIES_LIMIT=10
SCHEDULE_CRON=0 7 * * *
```

---

## 🚀 Uruchomienie lokalne

```bash
git clone https://github.com/yourname/trading-pipeline
cd trading-pipeline
pip install -r requirements.txt
cp .env.example .env
uvicorn app.main:app --reload
```

---

## 📬 Przykładowy raport (output LLM)

```
Spółka: NVDA (NVIDIA Corporation)
Data: 2025-05-08

📊 Analiza:
Silny wzrost przychodów napędzany popytem na GPU dla AI...

⚠️ Ryzyko:
Koncentracja przychodów w segmencie data center, presja regulacyjna...

🎯 Potencjał:
Dominująca pozycja w rynku akceleratorów AI. Score: 87/100.
```
