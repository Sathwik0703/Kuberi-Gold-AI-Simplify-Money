# 🪙 Kuberi Gold AI — Simplify Money
### AI Engineer Internship Assignment

A Kuberi-style gold investment AI assistant that replicates the Simplify Money digital gold workflow.

---

## 📌 Project Summary

This project implements two APIs that mirror the **Kuberi AI workflow** on Simplify Money:

| API | Endpoint | Purpose |
|-----|----------|---------|
| **API 1** | `POST /chat` | LLM-powered gold investment assistant — detects intent, provides facts, nudges to buy |
| **API 2** | `POST /buy-gold` | Digital gold purchase — calculates grams, saves transaction to DB |

---

## 🏗️ Architecture

```
User Message
     │
     ▼
┌─────────────────────────────────────┐
│  API 1: POST /chat                  │
│  ─────────────────                  │
│  • Claude LLM evaluates message     │
│  • is_gold_related? → Yes/No        │
│  • nudge_to_buy → always if gold    │
│  • redirect_to_purchase → if intent │
└──────────────┬──────────────────────┘
               │ redirect_to_purchase = true
               ▼
┌─────────────────────────────────────┐
│  API 2: POST /buy-gold              │
│  ─────────────────                  │
│  • Fetches gold price (live/cached) │
│  • Calculates gold grams            │
│  • Saves to SQLite DB               │
│  • Returns transaction confirmation │
└─────────────────────────────────────┘
```

---

## ⚙️ Tech Stack

- **Python 3.10+**
- **FastAPI** — REST API framework
- **Anthropic Claude** (`claude-sonnet-4`) — LLM for gold intent detection
- **SQLAlchemy + SQLite** — Database for purchase records
- **Pydantic** — Request/response validation
- **Uvicorn** — ASGI server

---

## 🚀 Local Setup & Run

### 1. Clone / Download the project
```bash
cd gold-investment-api
```

### 2. Create virtual environment
```bash
python -m venv venv
source venv/bin/activate        # Linux/Mac
venv\Scripts\activate           # Windows
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure environment variables
```bash
cp .env.example .env
# Edit .env and add your ANTHROPIC_API_KEY
```

### 5. Run the server
```bash
python run.py
```

Server starts at: **http://localhost:8000**
Interactive docs: **http://localhost:8000/docs**

---

## 🧪 Test the Full Flow

```bash
# In a second terminal (server must be running):
python tests/test_flow.py
```

---

## 📡 API Reference

### API 1 — Gold Investment Chat

**`POST /chat`**

Detects gold investment intent and responds with relevant information + nudge to buy.

**Request:**
```json
{
  "user_id": "user_001",
  "message": "Is gold a good investment right now?",
  "session_id": "optional_session_id"
}
```

**Response:**
```json
{
  "user_id": "user_001",
  "session_id": "sess_abc123",
  "message": "Is gold a good investment right now?",
  "is_gold_related": true,
  "nudge_to_buy": true,
  "redirect_to_purchase": false,
  "response": "Gold has delivered ~12-15% CAGR over 10 years in India. With Digital Gold on Simplify Money, you can start with just ₹1! Would you like to invest today? 🪙"
}
```

**Key Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `is_gold_related` | bool | True if message is about gold/investment |
| `nudge_to_buy` | bool | True when gold topic detected |
| `redirect_to_purchase` | bool | **True when user wants to BUY** → call API 2 |

---

### API 2 — Buy Digital Gold

**`POST /buy-gold`**

Processes digital gold purchase and saves to database.

**Request:**
```json
{
  "user_id": "user_001",
  "user_name": "Rahul Sharma",
  "amount_inr": 500.0
}
```

**Response:**
```json
{
  "transaction_id": "TXNABC1234567",
  "user_id": "user_001",
  "user_name": "Rahul Sharma",
  "amount_inr": 500.0,
  "gold_grams": 0.068306,
  "gold_price_per_gram": 7320.0,
  "status": "SUCCESS",
  "timestamp": "2025-03-11T10:30:00",
  "message": "✅ Purchase Successful! You bought 0.068306g of digital gold for ₹500.00. Gold price: ₹7320.00/gram. Transaction ID: TXNABC1234567"
}
```

---

### Utility Endpoints

**`GET /gold-price`** — Current gold price per gram in INR

**`GET /transactions/{user_id}`** — All purchases for a user

---

## ☁️ Deployment (Render.com — Free)

### Option A: Render.com (Recommended, free tier)

1. Push code to GitHub
2. Go to [render.com](https://render.com) → New Web Service
3. Connect your GitHub repo
4. Set:
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
5. Add environment variable: `ANTHROPIC_API_KEY = your_key`
6. Deploy → get live URL like `https://gold-ai.onrender.com`

### Option B: Railway.app (also free)

```bash
# Install Railway CLI
npm install -g @railway/cli
railway login
railway init
railway up
railway variables set ANTHROPIC_API_KEY=your_key
```

### Option C: Run locally with ngrok (instant public URL)

```bash
# Start server
python run.py

# In another terminal
ngrok http 8000
# → Public URL: https://abc123.ngrok.io
```

---

## 💾 Database Schema

Table: `gold_purchases`

| Column | Type | Description |
|--------|------|-------------|
| `id` | INT | Auto-increment primary key |
| `transaction_id` | VARCHAR | Unique transaction ID (e.g. TXN...) |
| `user_id` | VARCHAR | User identifier |
| `user_name` | VARCHAR | User display name |
| `amount_inr` | FLOAT | Amount invested in INR |
| `gold_grams` | FLOAT | Gold purchased in grams |
| `gold_price_per_gram` | FLOAT | Price at time of purchase |
| `status` | VARCHAR | SUCCESS / FAILED |
| `timestamp` | VARCHAR | UTC timestamp of purchase |
| `created_at` | DATETIME | DB insertion time |

---

## 🔄 Sample User Journey

```
1. User: "What are the returns on gold investment?"
   → is_gold_related: true
   → Response: Facts about 12-15% CAGR + nudge to buy

2. User: "How safe is digital gold?"
   → is_gold_related: true
   → Response: 24K purity, insured vaults, MMTC-PAMP + nudge

3. User: "Yes, I want to buy digital gold now!"
   → redirect_to_purchase: TRUE
   → Frontend calls /buy-gold API

4. POST /buy-gold { user_id, amount_inr: 500 }
   → Transaction saved to DB
   → Returns: "✅ Bought 0.068g gold for ₹500 | TXN ID: TXN..."
```

---

## 💡 Approach & Design Decisions

- **Two-API design** mirrors the Kuberi workflow: chat first, purchase second
- **Claude LLM** handles nuanced intent detection (rule-based fallback if no API key)
- **Gold price** fetched live from metals API, with hardcoded fallback (₹7,320/gram)
- **SQLite** used for simplicity; swappable to PostgreSQL via `DATABASE_URL` env var
- **Fallback logic** ensures the app works even without an Anthropic API key
- **CORS enabled** for easy frontend integration

---

## 📁 Project Structure

```
gold-investment-api/
├── app/
│   ├── __init__.py
│   ├── main.py          ← FastAPI app, all endpoints
│   ├── gold_ai.py       ← LLM chat logic (Claude + fallback)
│   ├── gold_prices.py   ← Live/cached gold price fetcher
│   ├── database.py      ← SQLAlchemy setup
│   └── models.py        ← GoldPurchase DB model
├── tests/
│   └── test_flow.py     ← Full end-to-end test
├── .env.example
├── requirements.txt
├── run.py               ← Entry point
└── README.md
```

---

*Assignment submission — Simplify Money | AI Engineer Internship*
*Replicating the Kuberi AI Gold Investment workflow*
