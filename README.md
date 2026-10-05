import MetaTrader5 as mt5
import pandas as pd
from datetime import datetime
from zoneinfo import ZoneInfo

SYMBOL = "XAUUSD"
TIMEFRAME = mt5.TIMEFRAME_M15

RSI_PERIOD = 14
EMA_FAST = 50
EMA_SLOW = 200


# -----------------------------
# CONNECT TO MT5
# -----------------------------
if not mt5.initialize():
    print("❌ MT5 connection failed")
    print(mt5.last_error())
    quit()

print("✅ Connected to MT5")


# -----------------------------
# GET MARKET DATA
# -----------------------------
rates = mt5.copy_rates_from_pos(
    SYMBOL,
    TIMEFRAME,
    0,
    300
)

if rates is None:
    print("❌ Could not get XAUUSD data")
    mt5.shutdown()
    quit()

df = pd.DataFrame(rates)

df["time"] = pd.to_datetime(df["time"], unit="s")

# -----------------------------
# EMA
# -----------------------------
df["EMA50"] = df["close"].ewm(
    span=EMA_FAST,
    adjust=False
).mean()

df["EMA200"] = df["close"].ewm(
    span=EMA_SLOW,
    adjust=False
).mean()


# -----------------------------
# RSI
# -----------------------------
delta = df["close"].diff()

gain = delta.where(delta > 0, 0)
loss = -delta.where(delta < 0, 0)

avg_gain = gain.rolling(RSI_PERIOD).mean()
avg_loss = loss.rolling(RSI_PERIOD).mean()

rs = avg_gain / avg_loss

df["RSI"] = 100 - (100 / (1 + rs))


# -----------------------------
# CURRENT DATA
# -----------------------------
current = df.iloc[-1]

price = current["close"]
rsi = current["RSI"]
ema50 = current["EMA50"]
ema200 = current["EMA200"]

now = datetime.now(
    ZoneInfo("Africa/Nairobi")
)


# -----------------------------
# TREND
# -----------------------------
if ema50 > ema200:
    trend = "BULLISH"
elif ema50 < ema200:
    trend = "BEARISH"
else:
    trend = "NEUTRAL"


# -----------------------------
# SIGNAL
# -----------------------------
if rsi < 30 and ema50 > ema200:
    signal = "BUY"

elif rsi > 70 and ema50 < ema200:
    signal = "SELL"

else:
    signal = "HOLD"


# -----------------------------
# DISPLAY
# -----------------------------
print()
print("=" * 45)
print("          XAUUSD RSI BOT")
print("=" * 45)

print(f"Time:       {now.strftime('%Y-%m-%d %H:%M:%S')} EAT")
print(f"Symbol:     {SYMBOL}")
print(f"Timeframe:  M15")
print(f"Price:      {price:.2f}")
print(f"RSI (14):   {rsi:.2f}")
print(f"EMA 50:     {ema50:.2f}")
print(f"EMA 200:    {ema200:.2f}")
print(f"Trend:      {trend}")
print(f"Signal:     {signal}")

print("=" * 45)


# -----------------------------
# CLOSE MT5 CONNECTION
# -----------------------------
mt5.shutdown()
