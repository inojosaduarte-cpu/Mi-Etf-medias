qimport streamlit as st
import yfinance as yf
import pandas_ta as ta
import pandas as pd

st.set_page_config(page_title="ETF Strategy Scanner", layout="wide")

st.title("📊 Escáner de ETFs: Estrategia de Recompra")
st.write("Buscando activos sobre MA200 con RSI entre 30 y 40.")

# Lista de ETFs que quieres monitorear (puedes agregar más)
tickers = ["QQQM", "NVDA", "VOO", "DIA", "XLK", "SMH", "IWM", "TSLA", "AAPL", "MSFT"]

def procesar_datos(ticker):
    df = yf.download(ticker, period="1y", interval="1d", progress=False)
    if df.empty: return None
    
    # Cálculos Técnicos
    df['MA20'] = ta.sma(df['Close'], length=20)
    df['MA50'] = ta.sma(df['Close'], length=50)
    df['MA100'] = ta.sma(df['Close'], length=100)
    df['MA200'] = ta.sma(df['Close'], length=200)
    df['RSI'] = ta.rsi(df['Close'], length=14)
    
    last = df.iloc[-1]
    
    # Lógica de tu estrategia
    cumple_ma200 = last['Close'] > last['MA200']
    cumple_rsi = 30 <= last['RSI'] <= 40
    
    return {
        "Ticker": ticker,
        "Precio": round(float(last['Close']), 2),
        "RSI": round(float(last['RSI']), 2),
        "Sobre MA200": "✅" if cumple_ma200 else "❌",
        "RSI en Rango": "✅" if cumple_rsi else "❌",
        "ESTADO": "🔥 COMPRA" if cumple_ma200 and cumple_rsi else "Esperar"
    }

if st.button('Actualizar Mercado'):
    resultados = []
    for t in tickers:
        res = procesar_datos(t)
        if res: resultados.append(res)
    
    df_final = pd.DataFrame(resultados)
    
    # Resaltar filas de compra
    def highlight_buy(s):
        return ['background-color: #2ecc71; color: white' if s.ESTADO == '🔥 COMPRA' else '' for _ in s]

    st.table(df_final.style.apply(highlight_buy, axis=1))
    
  # Mi-Etf-medias
