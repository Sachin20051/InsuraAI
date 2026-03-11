# InsuraAI — India Life Insurance Claims Analytics

A full-stack web application that lets users type natural language questions about India's life insurance claims data and instantly receive interactive charts powered by Google Gemini AI.

---

## Overview

```
User types: "Show top 10 insurers by claims paid amount"
  → Gemini generates a SQL query
  → SQLite executes it
  → Chart type is auto-selected
  → An interactive Recharts dashboard is rendered
```

### Tech Stack

| Layer    | Technology                             |
| -------- | -------------------------------------- |
| Frontend | React 18 + Vite, Recharts, TailwindCSS |
| Backend  | Python FastAPI                         |
| AI       | Google Gemini API (gemini-2.0-flash)   |
| Database | SQLite (in-memory, loaded from CSV)    |

---

## Project Structure

```
ai-dashboard/
├── backend/
│   ├── main.py              # FastAPI app & API endpoints
│   ├── gemini_service.py    # Gemini API integration
│   ├── database.py          # SQLite setup & query execution
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── PromptInput.jsx
│   │   │   ├── ChartRenderer.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   ├── FilterBar.jsx
│   │   │   ├── MetricCard.jsx
│   │   │   └── Sidebar.jsx
│   │   ├── services/
│   │   │   └── api.js
│   │   ├── pages/
│   │   │   └── Home.jsx
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── postcss.config.js
├── data/
│   └── insurance_claims.csv
└── README.md
```

---

## Prerequisites

- **Python 3.10+**
- **Node.js 18+** (with npm)
- **Google Gemini API key** — get one at https://aistudio.google.com/apikey

---

## Environment Variables

Create a `.env` file (or export in your shell) before starting the backend:

```
GEMINI_API_KEY=your_google_gemini_api_key_here
```

---

## Setup & Run

### 1. Backend

```bash
cd ai-dashboard/backend

# Create a virtual environment (recommended)
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # macOS / Linux

# Install dependencies
pip install -r requirements.txt

# Set your API key
set GEMINI_API_KEY=your_key  # Windows
# export GEMINI_API_KEY=your_key  # macOS / Linux

# Start the server
uvicorn main:app --reload
```

The API will be available at **http://localhost:8000**  
Docs at **http://localhost:8000/docs**

### 2. Frontend

```bash
cd ai-dashboard/frontend

# Install dependencies
npm install

# Start the dev server
npm run dev
```

The app will open at **http://localhost:5173**

---

## API Reference

### `POST /query`

**Request body:**

```json
{
  "prompt": "Show top 10 insurers by claims paid amount"
}
```

**Response:**

```json
{
  "chart_type": "bar",
  "x_axis": "life_insurer",
  "y_axis": "total_claims_paid",
  "data": [
    { "life_insurer": "LIC", "total_claims_paid": 145230.5 },
    { "life_insurer": "HDFC", "total_claims_paid": 21050.3 }
  ],
  "sql": "SELECT life_insurer, SUM(claims_paid_amt) AS total_claims_paid FROM insurance_claims GROUP BY life_insurer ORDER BY total_claims_paid DESC LIMIT 10"
}
```

### `GET /summary`

Returns KPI metrics: total claims paid, pending amount, claims settled count, average settlement ratio, number of insurers, and years covered.

### `GET /filters`

Returns distinct filter values: `insurers`, `years`, `categories`.

### `GET /health`

Returns `{ "status": "ok" }`

---

## Example Prompts

| Prompt | Expected Chart |
| --- | --- |
| Show claims paid amount by insurer | Bar |
| Show claim settlement ratio trend by year | Line |
| Compare repudiated vs paid claims by insurer | Bar |
| Show top 10 insurers by claims paid amount | Bar |
| Show pending claims trend over years | Line |
| Distribution of total claims by insurer | Pie |

---

## Dataset

The dataset (`data/insurance_claims.csv`) contains 149 rows of India life insurance individual death claims data (2017-18 to 2021-22) from 45 insurers with these key columns:

| Column | Type | Description |
| --- | --- | --- |
| life_insurer | TEXT | Insurer short name (LIC, HDFC, ICICI Prudential, …) |
| year | TEXT | Financial year (2017-18 to 2021-22) |
| claims_paid_no | REAL | Number of claims settled |
| claims_paid_amt | REAL | Claims paid amount (₹ crores) |
| claims_repudiated_no | REAL | Number of claims repudiated |
| claims_pending_end_no | REAL | Claims pending at year end |
| claims_paid_ratio_no | REAL | Settlement ratio by count (0–1) |
| claims_pending_ratio_no | REAL | Pending ratio by count (0–1) |
| category | TEXT | Claim category (Individual Death Claims) |

Additional columns include `claims_intimated_*`, `claims_rejected_*`, `claims_unclaimed_*`, `total_claims_*`, and amount-based ratio columns.

---

## Chart Selection Logic

| Data Pattern | Chart Type |
| --- | --- |
| Trends over years | Line |
| Grouped by insurer | Bar |
| Claim ratios comparison | Bar |
| Distribution / proportion | Pie |

---

## License

MIT
