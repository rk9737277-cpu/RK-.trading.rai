import streamlit as st
import yfinance as yf
import pandas as pd
import pandas_ta as ta
import feedparser

st.set_page_config(page_title="AI Pro Trading Terminal", layout="wide", page_icon="📈")

st.title("⚡ AI प्रो ट्रेडिंग टर्मिनल (Indian Scalping & Crypto)")
st.caption("रूल्स-बेस्ड मल्टी-टाइमफ्रेम एनालिसिस • 9/15 EMA • RSI मोमेंटम • सेंटीमेंट गाइड")

st.sidebar.header("🎯 मार्केट & एसेट सेलेक्शन")
market_type = st.sidebar.radio("मार्केट चुनें:", ["Indian Index (Scalping)", "Cryptocurrency (Swing/Intraday)"])

if market_type == "Indian Index (Scalping)":
    asset_dict = {
        "NIFTY 50": "^NSEI",
        "BANK NIFTY": "^NSEBANK",
        "FIN NIFTY": "NIFTY_FIN_SERVICE.NS",
        "SENSEX": "^BSESN"
    }
else:
    asset_dict = {
        "Bitcoin (BTC/USDT)": "BTC-USD",
        "Ethereum (ETH/USDT)": "ETH-USD",
        "Solana (SOL/USDT)": "SOL-USD",
        "BNB": "BNB-USD",
        "Ripple (XRP)": "XRP-USD"
    }

selected_asset = st.sidebar.selectbox("इंडेक्स / कॉइन चुनें:", list(asset_dict.keys()))
ticker = asset_dict[selected_asset]

rr_ratio = st.sidebar.selectbox("रिस्क-टू-रिवॉर्ड (RR) रेशियो:", ["1:2 (Recommended)", "1:1.5"])
target_multiplier = 2.0 if "1:2" in rr_ratio else 1.5

st.sidebar.markdown("---")
st.sidebar.subheader("🛡️ रिस्क मैनेजमेंट रूल्स")
st.sidebar.info("""
- प्रति ट्रेड रिस्क: 1-2% कैपिटल
- 2 लगातार स्टॉप-लॉस हिट होने पर स्क्रीन बंद करें!
- न्यूज़ रिलीज के समय कोई नई स्कैल्पिंग एंट्री न लें।
""")

def fetch_indicators(symbol, interval, period):
    try:
        df = yf.download(symbol, period=period, interval=interval, progress=False)
        if df.empty or len(df) < 20:
            return None
        if isinstance(df.columns, pd.MultiIndex):
            df.columns = df.columns.get_level_values(0)
            
        df['EMA_9'] = ta.ema(df['Close'], length=9)
        df['EMA_15'] = ta.ema(df['Close'], length=15)
        df['RSI'] = ta.rsi(df['Close'], length=14)
        df['ATR'] = ta.atr(df['High'], df['Low'], df['Close'], length=14)
        return df
    except Exception:
        return None

def analyze_trend(df):
    if df is None or len(df) < 2:
        return "डाटा अनुपलब्ध", "gray"
    last = df.iloc[-1]
    if last['Close'] > last['EMA_9'] and last['EMA_9'] > last['EMA_15'] and last['RSI'] > 50:
        return "🟢 तेज बुलिश (Bullish)", "green"
    elif last['Close'] < last['EMA_9'] and last['EMA_9'] < last['EMA_15'] and last['RSI'] < 50:
        return "🔴 तेज बेयरिश (Bearish)", "red"
    else:
        return "🟡 साइडवेज़ / न्यूट्रल (Sideways)", "orange"

with st.spinner("लाइव डेटा फेच किया जा रहा है..."):
    if market_type == "Indian Index (Scalping)":
        df_high = fetch_indicators(ticker, interval="15m", period="5d")
        df_mid = fetch_indicators(ticker, interval="5m", period="2d")
        df_low = fetch_indicators(ticker, interval="1m", period="1d")
        t_high_name, t_mid_name, t_low_name = "15 मिनट (Trend)", "5 मिनट (Structure)", "1 मिनट (Entry)"
    else:
        df_high = fetch_indicators(ticker, interval="1d", period="60d")
        df_mid = fetch_indicators(ticker, interval="1h", period="14d")
        df_low = fetch_indicators(ticker, interval="15m", period="5d")
        t_high_name, t_mid_name, t_low_name = "डेली (Macro Trend)", "1 घंटा (Setup)", "15 मिनट (Trigger)"

col1, col2, col3 = st.columns(3)
trend_high, color_high = analyze_trend(df_high)
trend_mid, color_mid = analyze_trend(df_mid)
trend_low, color_low = analyze_trend(df_low)

col1.markdown(f"**{t_high_name}**")
col1.markdown(f"<h4 style='color:{color_high};'>{trend_high}</h4>", unsafe_allow_html=True)
col2.markdown(f"**{t_mid_name}**")
col2.markdown(f"<h4 style='color:{color_mid};'>{trend_mid}</h4>", unsafe_allow_html=True)
col3.markdown(f"**{t_low_name}**")
col3.markdown(f"<h4 style='color:{color_low};'>{trend_low}</h4>", unsafe_allow_html=True)

st.markdown("---")

active_df = df_low if df_low is not None else df_mid

if active_df is not None and len(active_df) >= 2:
    curr = active_df.iloc[-1]
    price = float(curr['Close'])
    rsi = float(curr['RSI'])
    ema9 = float(curr['EMA_9'])
    ema15 = float(curr['EMA_15'])
    atr = float(curr['ATR']) if not pd.isna(curr['ATR']) else (price * 0.003)

    st.subheader("🎯 AI ट्रेड सिग्नल & लेवल्स")
    signal_col, data_col = st.columns([2, 1])
    
    with signal_col:
        if curr['EMA_9'] > curr['EMA_15'] and price > ema9 and rsi > 60:
            sl = price - (atr * 1.2)
            risk = price - sl
            tp = price + (risk * target_multiplier)
            st.success("### 🚀 BUY / CALL सिग्नल सक्रिय")
            st.write(f"- **एंट्री प्राइस:** ₹{price:.2f}")
            st.write(f"- **स्टॉप-लॉस (SL):** ₹{sl:.2f} (जोखिम: ₹{risk:.2f})")
            st.write(f"- **टारगेट ({rr_ratio}):** ₹{tp:.2f} (प्रॉफिट: ₹{risk*target_multiplier:.2f})")
        elif curr['EMA_9'] < curr['EMA_15'] and price < ema9 and rsi < 40:
            sl = price + (atr * 1.2)
            risk = sl - price
            tp = price - (risk * target_multiplier)
            st.error("### 🔻 SELL / PUT सिग्नल सक्रिय")
            st.write(f"- **एंट्री प्राइस:** ₹{price:.2f}")
            st.write(f"- **स्टॉप-लॉस (SL):** ₹{sl:.2f} (जोखिम: ₹{risk:.2f})")
            st.write(f"- **टारगेट ({rr_ratio}):** ₹{tp:.2f} (प्रॉफिट: ₹{risk*target_multiplier:.2f})")
        elif 40 <= rsi <= 60:
            st.warning("### ⏸️ नो ट्रेड ज़ोन (साइडवेज़ मार्केट)")
            st.write(f"RSI **{rsi:.1f}** है। मार्केट चॉपी है, ट्रेड से बचें।")
        else:
            st.info("### ⏳ सेटअप की प्रतीक्षा करें")
            st.write(f"वर्तमान मोमेंटम स्पष्ट नहीं है (RSI: {rsi:.1f})।")

    with data_col:
        st.metric("LTP (मौजूदा भाव)", f"{price:.2f}")
        st.metric("RSI (14)", f"{rsi:.1f}")
        st.metric("9 EMA / 15 EMA", f"{ema9:.1f} / {ema15:.1f}")

st.markdown("---")
st.subheader("📰 लाइव मार्केट न्यूज़ & सेंटीमेंट")
tab1, tab2 = st.tabs(["ग्लोबल / डोमेस्टिक न्यूज़", "डेली प्री-मार्केट चेकलिस्ट"])

with tab1:
    news_feed_url = "https://finance.yahoo.com/news/rssindex" if market_type == "Indian Index (Scalping)" else "https://cointelegraph.com/rss"
    try:
        feed = feedparser.parse(news_feed_url)
        for entry in feed.entries[:5]:
            st.markdown(f"**[{entry.title}]({entry.link})**")
            st.caption(f"प्रकाशित: {entry.get('published', 'हाल ही में')}")
    except Exception:
        st.write("न्यूज़ लोड करने में असमर्थ।")

with tab2:
    if market_type == "Indian Index (Scalping)":
        st.markdown("""
        - [ ] **GIFT Nifty:** सुबह 8:30 - 9:00 बजे चेक करें।
        - [ ] **US Futures & Crude Oil:** क्या US मार्केट पॉजिटिव है?
        - [ ] **FII / DII डेटा:** कल बाइंग थी या सेलिंग?
        - [ ] **इवेंट्स:** RBI पॉलिसी या बजट का दिन तो नहीं है?
        """)
    else:
        st.markdown("""
        - [ ] **Bitcoin Dominance:** बीटीसी डोमिनेंस चेक करें।
        - [ ] **US Fed Rates:** US Dollar Index (DXY) का हाल।
        - [ ] **व्हेल अलर्ट्स:** लिक्विडेशन और बड़ा फंड ट्रांसफर चेक करें।
        """)
