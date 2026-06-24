# StockSense AI — Stock Price Prediction

A full-stack ML web application that predicts stock closing prices for the
**Top 50 S&P 500 companies** using Random Forest and Linear Regression models.

---

## Project Structure

```
stock_predictor/
├── backend/
│   ├── app.py          ← Flask REST API
│   ├── model.py        ← ML training & inference logic
│   └── utils.py        ← Data loading, feature engineering, metrics
├── frontend/
│   ├── index.html      ← Single-page application
│   ├── style.css       ← Dark terminal UI
│   └── script.js       ← Chart.js + API communication
├── data/
│   ├── generate_data.py  ← Synthetic dataset generator
│   └── stocks.csv        ← 63 000-row OHLCV dataset (50 tickers × ~1260 days)
├── model/              ← Trained .joblib model files (auto-created)
├── requirements.txt
└── README.md
```

---

## Features

| Feature | Details |
|---|---|
| **ML Models** | Random Forest + Linear Regression |
| **Metrics** | MAE, RMSE, R² (train & test sets) |
| **Feature Engineering** | MA(5/10/20/50), RSI-14, Bollinger width, lag features, volatility |
| **Charts** | Price history, Predicted vs Actual, Volume trend, Feature importance |
| **API** | Flask REST: `/stocks`, `/train`, `/predict`, `/model/<ticker>` |
| **UI** | Dark financial-terminal design, responsive, auto-fills last OHLCV row |

---

## Quick Start — Local

### 1 — Install dependencies

```bash
cd stock_predictor
pip install -r requirements.txt
```

### 2 — Generate (or replace) the dataset

```bash
python data/generate_data.py
```

> **To use the real Kaggle dataset**: Download
> [ibrahimshahrukh/top-50-companies-dataset](https://www.kaggle.com/datasets/ibrahimshahrukh/top-50-companies-dataset),
> rename the CSV to `stocks.csv`, and place it in `data/`.
> Make sure it has columns: `Ticker, Company, Sector, Date, Open, High, Low, Close, Volume`.

### 3 — Run the Flask server

```bash
cd backend
python app.py
```

Open **http://localhost:5000** in your browser.

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET`  | `/stocks` | List all companies + trained status |
| `GET`  | `/stocks/<TICKER>` | Full OHLCV history for one ticker |
| `POST` | `/train` | Train RF + LR models for a ticker |
| `POST` | `/predict` | Predict closing price |
| `GET`  | `/model/<TICKER>` | Fetch saved model metadata & chart arrays |

### Train example

```json
POST /train
{ "ticker": "AAPL", "test_size": 0.2 }
```

### Predict example

```json
POST /predict
{ "ticker": "AAPL", "model_type": "rf", "Open": 152.0, "High": 156.5, "Low": 151.3, "Volume": 82000000 }
```

---

## Deploying on Render

1. Push the `stock_predictor/` folder to a GitHub repository.
2. Create a new **Web Service** on [render.com](https://render.com).
3. Set:
   - **Build command**: `pip install -r requirements.txt`
   - **Start command**: `gunicorn backend.app:app --chdir /opt/render/project/src`
4. Add environment variable `FLASK_ENV=production`.
5. Deploy. Render will assign a public URL.

## Deploying on Railway

1. Push to GitHub.
2. Create a new project on [railway.app](https://railway.app), link repo.
3. Railway auto-detects Python. Add start command:
   ```
   cd backend && gunicorn app:app -b 0.0.0.0:$PORT
   ```
4. Done — Railway assigns a public URL.

---

## Model Details

### Random Forest
- 150 estimators, max depth 12
- Trained on 80% of chronological data (no shuffle to prevent data leakage)
- Feature importance ranking available in the UI

### Linear Regression
- Standard least-squares with feature scaling (StandardScaler)
- Provided as a baseline comparison

### Features Used (24 total)
| Category | Features |
|---|---|
| Raw inputs | Open, High, Low, Volume |
| Price-derived | Range, PriceChange, PctChange |
| Moving averages | MA5, MA10, MA20, MA50 |
| Volume averages | Vol_MA5, Vol_MA10, Vol_MA20, Vol_MA50 |
| Volatility | Volatility10, Volatility20 |
| Technical | RSI14, BB_Width |
| Lag features | Lag1, Lag2, Lag3, Lag5 |

---

## License
MIT — free for educational and commercial use.
