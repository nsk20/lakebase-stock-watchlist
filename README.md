# Real-Time Stock Watchlist & Data Synchronization Service

A production-grade OLTP data engineering application built with **Flask**, **Databricks Lakebase** (managed PostgreSQL), and the **Massive Financial API**. This application enables users to track live equity prices, manage personalized stock watchlists, and perform batch data ingestion with Change Data Capture (CDC) upserts into an operational relational database.

---

## 🏗️ Project Architecture

```
                       +------------------------+
                       |   Web User Interface   |
                       +-----------+------------+
                                   |
                                   v
                       +------------------------+
                       |    Flask REST API      |
                       |       (app.py)         |
                       +---+----------------+---+
                           |                |
             +-------------+                +-------------+
             |                                            |
             v                                            v
+------------------------+                    +------------------------+
| Massive Financial API  |                    |   Databricks Lakebase  |
| (Live Market Data)     |                    | (Managed PostgreSQL)   |
+------------------------+                    +------------------------+
             ^                                            ^
             |       +------------------------+           |
             +-------| Databricks SDK Secrets |-----------+
                     |  Scope ("massive" &    |
                     |       "database")      |
                     +------------------------+
```

---

## ✨ Features

- **Real-Time Stock Fetching:** Fetch live quotes for specific equity tickers using direct single-call endpoints to minimize API overhead and honor strict rate limits.
- **Change Data Capture & Operational Upserts:** High-throughput batch synchronization into PostgreSQL utilizing native `ON CONFLICT DO UPDATE` (upsert) clauses for state consistency.
- **Identity-Aware Multi-Tenancy:** Personalizes watchlists per user by leveraging Databricks App identity headers (`X-Forwarded-Email`) with seamless fallback to the Databricks SDK user context during local execution.
- **Secure Secret Management:** Integrates with Databricks Secret Scopes (`massive` and `database`), ensuring no plain-text credentials or API tokens exist in source code or configuration files.
- **Modern Responsive Interface:** Clean, dark-mode dashboard with real-time UI status updates and ticker watchlists.

---

## 🛠️ Tech Stack & Prerequisites

### Tech Stack
- **Backend Framework:** Python 3.10+, Flask
- **Database Layer:** Databricks Lakebase (PostgreSQL), `psycopg2`, `SQLAlchemy`
- **Integrations:** Databricks SDK, Massive Financial Market API
- **Deployment:** Databricks Apps (`app.yaml`)

### Prerequisites
- Python `3.10+` installed locally.
- Access to a **Databricks Workspace** with Lakebase enabled.
- A valid **Massive API Key**.
- Databricks CLI configured locally (`databricks configure`).

---

## ⚙️ Environment Variables & Secret Configuration

The application reads database connection strings and API credentials securely from Databricks Secret Scopes or local environment variables.

| Variable Name | Required | Default Value | Description |
| :--- | :---: | :--- | :--- |
| `LAKEBASE_URL` | Local | - | PostgreSQL connection URL (`postgresql://user:pass@host:5432/db`) |
| `LAKEBASE_SECRET_SCOPE` | Prod | `database` | Databricks Secret Scope containing Lakebase connection URL |
| `LAKEBASE_SECRET_KEY` | Prod | `lakebase-url` | Secret key name for the Lakebase URL |
| `MASSIVE_API_KEY` | Dev/Setup | - | API token for Massive Financial API |
| `MASSIVE_API_BASE_URL` | Optional | `https://api.massive.com` | Base URL endpoint for market data |
| `MASSIVE_SECRET_SCOPE` | Prod | `massive` | Databricks Secret Scope for Massive API token |
| `MASSIVE_SECRET_KEY` | Prod | `api-key` | Secret key name for the Massive API token |

---

## 🚀 Local Setup & Installation

### 1. Clone & Set Up Virtual Environment

```bash
# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configure Secrets in Databricks

Run the included secret initialization utility to provision secret scopes in your Databricks workspace:

```bash
python setup_secrets.py
```
*You will be prompted to paste your Massive API Key and Lakebase connection string.*

Alternatively, for local testing without Databricks SDK secrets, create a `.env` file from `.env.example`:

```bash
cp .env.example .env
```

### 3. Launch the Application

```bash
python app.py
```
Navigate to `http://localhost:8000` in your web browser.

---

## 📡 API Reference

### Watchlist Management
- **`GET /watchlist`**: Fetch watchlist items for the authenticated user.
- **`POST /watchlist`**: Add a ticker symbol to the watchlist. Fetches latest quote from Massive API and updates Lakebase.
  ```json
  {
    "symbol": "AAPL"
  }
  ```

### Data Synchronization & Ingestion
- **`GET /records`**: Retrieve paginated ingested records stored in Lakebase.
- **`POST /sync`**: Trigger a batch ingestion pipeline from Massive API into Lakebase.

### Health check
- **`GET /healthz`**: Returns `{"status": "ok"}` for container health monitoring.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for details.
