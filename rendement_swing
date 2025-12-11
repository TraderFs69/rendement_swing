import streamlit as st
import requests
import os
from dotenv import load_dotenv
import pandas as pd

def load_api_key():
    # 1. Try Streamlit Secrets
    if "POLYGON_API_KEY" in st.secrets:
        return st.secrets["POLYGON_API_KEY"]

    # 2. Try .env file
    load_dotenv()
    if os.getenv("POLYGON_API_KEY"):
        return os.getenv("POLYGON_API_KEY")

    # 3. Try system env variable
    if "POLYGON_API_KEY" in os.environ:
        return os.environ["POLYGON_API_KEY"]

    raise ValueError("❌ No Polygon API Key found. Add it to .env or Streamlit Secrets.")

POLYGON_API_KEY = load_api_key()


# ------- API FUNCTIONS -------

def get_open_price(ticker: str, date: str):
    url = f"{BASE_URL}/v1/open-close/{ticker}/{date}?adjusted=true&apiKey={POLYGON_API_KEY}"
    r = requests.get(url)

    if r.status_code != 200:
        raise Exception(f"Polygon error: {r.text}")

    data = r.json()

    if "status" in data and data["status"] == "NOT_FOUND":
        raise Exception(f"No data found for {ticker} on {date}")

    return data["open"]


def get_last_close(ticker: str):
    url = f"{BASE_URL}/v2/aggs/ticker/{ticker}/prev?adjusted=true&apiKey={POLYGON_API_KEY}"
    r = requests.get(url)

    if r.status_code != 200:
        raise Exception(f"Polygon error retrieving last close: {r.text}")

    data = r.json()
    return data["results"][0]["c"]


def calculate_return(open_price: float, close_price: float) -> float:
    return ((close_price - open_price) / open_price) * 100


# ------- STREAMLIT UI -------

st.set_page_config(page_title="Stock Returns Calculator", page_icon="📈", layout="centered")

st.title("📊 Calculateur de Rendement Boursier (Polygon API)")

st.write("Entrez une liste de tickers (séparés par des virgules) et une date pour calculer le rendement entre "
         "**l'ouverture de cette date** et **le dernier closing disponible**.")

tickers_input = st.text_input("Tickers (ex: HPE, SLB, AVGO, MS, PSX)", "")
date_input = st.date_input("Date d'ouverture à analyser")

if st.button("Calculer les rendements"):
    if not tickers_input:
        st.error("Veuillez entrer au moins un ticker.")
    else:
        tickers = [t.strip().upper() for t in tickers_input.split(",")]
        date_str = date_input.strftime("%Y-%m-%d")

        results = []

        for ticker in tickers:
            try:
                open_price = get_open_price(ticker, date_str)
                last_close = get_last_close(ticker)
                pct_return = calculate_return(open_price, last_close)

                results.append({
                    "Ticker": ticker,
                    "Open Price (" + date_str + ")": round(open_price, 4),
                    "Last Close": round(last_close, 4),
                    "Return %": round(pct_return, 2)
                })

            except Exception as e:
                results.append({
                    "Ticker": ticker,
                    "Error": str(e)
                })

        df = pd.DataFrame(results)
        st.subheader("Résultats")
        st.dataframe(df, use_container_width=True)

        # résumé global
        if "Return %" in df.columns:
            avg_return = df["Return %"].mean()
            st.metric("📌 Rendement moyen du portefeuille", f"{round(avg_return, 2)} %")
