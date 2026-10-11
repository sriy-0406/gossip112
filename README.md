# 🥦 Gossip Web

> **A Hyperlocal, Crowd-Sourced Price & Demand Sharing Network for Street Vendors & Small Farmers in India**

Gossip Web transforms how informal vegetable and fruit vendors (*sabziwalas*) in Indian cities stay informed. Instead of relying on guesswork or exploitative middlemen, vendors broadcast quick price and demand updates using **natural speech or text** in their native languages (Bengali, Hindi, Hinglish, English). 

The platform extracts structured market signals via LLMs, scores report trustworthiness against local peer consensus, spreads relevant gossip to nearby vendors via an animated geospatial routing engine, and generates hyper-actionable commercial advice.

---

## 🏛️ System Architecture

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                CLIENT BROWSER DASHBOARD                                │
│                                                                                        │
│  [ Web Speech API ]       [ Leaflet + OSM Map ]       [ Gossip Spread Simulator ]      │
│  Mic / Multilingual Voice  Demand-coded Pins (Kolkata) Hop Propagation Wave & Radar     │
│           │                          ▲                           ▲                     │
│           ▼                          │                           │                     │
│  [ Left Submission Panel ] [ Right Feed & Advice ]     [ Bottom Trust Leaderboard ]    │
│  Vendor Picker & Presets   Nearest-first Produce Feed   Rankings: Pillar -> Flagged    │
└──────────────────────────────────────┬─────────────────────────────────────────────────┘
                                       │ HTTP / JSON
                                       ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              FASTAPI BACKEND (Python)                                  │
│                                                                                        │
│  POST /report         GET /feed/{id}        GET /advice/{id}      POST /feedback       │
│         │                    │                     │                     │             │
│         ▼                    ▼                     ▼                     ▼             │
│  ┌──────────────┐    ┌──────────────┐      ┌──────────────┐      ┌──────────────┐      │
│  │ Extraction   │    │ Gossip Feed  │      │ AI Advice    │      │ Peer Vote    │      │
│  │ Gemini 2.5 / │    │ Filtering &  │      │ Generator    │      │ Trust Score  │      │
│  │ Regex Backup │    │ Distance Sort│      │ (bn, hi, en) │      │ Adjustment   │      │
│  └──────┬───────┘    └──────┬───────┘      └──────┬───────┘      └──────┬───────┘      │
│         │                   │                     │                     │              │
│         ▼                   │                     │                     │              │
│  ┌──────────────┐           │                     │                     │              │
│  │ Geocoder     │           │                     │                     │              │
│  │ 15 Kolkata   │           │                     │                     │              │
│  │ Markets Dict │           │                     │                     │              │
│  └──────┬───────┘           │                     │                     │              │
│         │                   │                     │                     │              │
│         ▼                   │                     │                     │              │
│  ┌──────────────┐           │                     │                     │              │
│  │ Trust Engine │           │                     │                     │              │
│  │ Weighted Med.│           │                     │                     │              │
│  │ Outlier Flag │           │                     │                     │              │
│  └──────┬───────┘           │                     │                     │              │
└─────────┼───────────────────┼─────────────────────┼─────────────────────┼──────────────┘
          │                   │                     │                     │
          ▼                   ▼                     ▼                     ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                               SQLITE DATABASE (SQLAlchemy)                             │
│                                                                                        │
│     [ Vendors Table ]             [ Reports Table ]            [ Feedback Table ]      │
│  15 Seeded Kolkata Vendors    ~60 Seeded Active Reports     Peer Upvotes & Downvotes   │
│  Trust: 0.00 to 1.00          Item, Price, Demand, Flags    Audit Log                  │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## ⚡ Tech Stack

- **Backend**: Python 3.13 + **FastAPI** + **Uvicorn**
- **Database**: **SQLite** via **SQLAlchemy 2.0** ORM
- **Intelligence & LLM**:
  - Primary: **Google Gemini API** (`gemini-2.5-flash` via `google-genai` SDK)
  - Zero-Break Fallback: Deterministic regex and multilingual token parser for Hindi, Bengali, Hinglish, and English
- **Voice Input**: Browser native **Web Speech API** (`webkitSpeechRecognition`) with zero paid speech APIs
- **Geospatial & Maps**: **Leaflet.js** + **OpenStreetMap** (free, no API key required)
- **Frontend Dashboard**: Responsive single-page architecture built with **Tailwind CSS**, Lucide/FontAwesome iconography, and dynamic JavaScript

---

## 🚀 Quickstart & Run Steps

The entire application runs locally with **one simple command**. No Docker or cloud subscriptions needed.

### 1. Clone & Enter Directory
```bash
cd c:\Users\ACER\Downloads\vibes-folder-1\gossip112
```

### 2. (Optional) Set Gemini API Key
Create or edit `.env`:
```env
GEMINI_API_KEY=your_gemini_api_key_here
```
> *Note: If no API key is provided, the platform automatically engages its intelligent regex & multilingual fallback parser, ensuring tests and live demos run without interruption.*

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Seed Database
Seeds 15 realistic vendors across 15 Kolkata market hubs and ~60 market reports (including 4 deliberate price outliers to prove outlier detection):
```bash
python seed.py
```

### 5. Launch the Application
```bash
python run.py
```
Open your browser at: **`http://127.0.0.1:8000`**

---

## 🎯 Core Features

### 1. Multilingual Report Submission (`POST /report`)
- Accepts conversational natural language voice transcripts or text (e.g. *"Aaj tomato 30 rupaye, bahut bik raha hai, Gariahat"* or *"Sealdah te aloo 24 taka kg, khub demand"*).
- Extracts strict structured JSON: `{item, price, unit, demand, location_name}`.
- Normalizes local names (*tamatar/bilati -> tomato*, *aloo/alu -> potato*, *pyaz/piyaj -> onion*, *patakopi -> cabbage*, *kela/kola -> banana*).
- Geocodes locations against a hardcoded dictionary of 15 Kolkata markets (*Gariahat, Sealdah, Salt Lake, New Market, Esplanade, Behala, Howrah, Park Street, Dumdum, Jadavpur, Lake Market, Maniktala, Baghbazar, Shyambazar, Hatibagan*).

### 2. Consensus Trust Scoring Algorithm
- Every vendor starts with a trust score from `0.0` to `1.0` (default `0.5`).
- When a new report arrives, the engine queries active reports (< 6 hours) for the same produce within a **5 km radius** and **3-hour window**.
- Computes a **trust-weighted median price**.
- **Within 25% of consensus**: Vendor trust increases (`+0.03`), and report is marked `Verified`.
- **Outlier (> 25% discrepancy)**: Vendor trust drops (`-0.08`), and the report is marked `Flagged Outlier`.
- Community validation via `POST /feedback`: Peer vendors can vote `correct` (boosts reporter trust) or `wrong` (penalizes reporter trust).

### 3. Hyperlocal Gossip Routing (`GET /feed/{vendor_id}`)
- Filters reports exclusively to items the active vendor trades in.
- Automatically excludes expired (> 6h) and flagged outlier reports.
- Computes Haversine distance and sorts **nearest first** (e.g., *"0.3 km away - Gariahat"*).

### 4. Animated Gossip Spread Simulator
- Demonstrates real-time information diffusion.
- When a vendor reports a price, the system calculates proximity to all other vendors and simulates hop-by-hop propagation with an animated map radar wave and timeline hops.

### 5. AI Hyperlocal Advice (`GET /advice/{vendor_id}`)
- Generates 2-3 short, jargon-free actionable advice bullets in the vendor's preferred language (**Bengali `bn`**, **Hindi `hi`**, or **English `en`**).
- E.g.: *"Gariahat-এ Tomato-এর চাহিদা খুব বেশি (₹30.0/kg)। স্টক পাঠালে ভালো লাভ পাবেন।"*

---



## 🧪 Testing

The codebase includes an automated test suite with **19 tests** covering trust scoring, Haversine distance, weighted median math, regex fallback parsing, and full API end-to-end flows.

Run tests:
```bash
pytest -v
```

All 19 tests pass:
- `tests/test_extraction.py`: Multilingual keyword extraction, regex parsing, unit normalization, geocoding.
- `tests/test_trust.py`: Weighted median calculation, Haversine distance, corroborated report boosts, outlier penalties, peer feedback.
- `tests/test_api_e2e.py`: End-to-end integration tests for report submission, feed routing, AI advice, and data resets.

---

## 🔮 Future Scope & Non-Goals

- **Real WhatsApp & SMS Gateway**: In production, vendors interact via WhatsApp voice notes through WhatsApp Business API and low-cost IVR phone lines.
- **ONDC Integration**: Direct integration with ONDC (Open Network for Digital Commerce) to connect surplus vendor inventory with nearby bulk buyers.
- **Offline Mesh Gossip**: Bluetooth / Wi-Fi Direct peer-to-peer gossip dissemination for wholesale markets with poor cellular reception.
- **No Payments / Speculation**: Gossip Web focuses strictly on trustworthy information exchange.
