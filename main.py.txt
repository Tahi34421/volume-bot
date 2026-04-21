import time
import requests
import os
from binance.client import Client

# Get secrets from Railway
BOT_TOKEN = os.getenv("BOT_TOKEN")
CHAT_ID = os.getenv("CHAT_ID")

client = Client()

def send_telegram(message):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    requests.get(url, params={"chat_id": CHAT_ID, "text": message})

alerted = set()

def check_symbol(symbol):
    global alerted

    klines = client.get_klines(symbol=symbol, interval="1m", limit=5)

    green = 0
    for k in klines[-4:]:
        if float(k[4]) > float(k[1]):  # close > open
            green += 1

    if green == 4:
        if symbol not in alerted:
            send_telegram(f"{symbol} → 4 green volume bars")
            alerted.add(symbol)
    else:
        alerted.discard(symbol)

def run():
    symbols = [s['symbol'] for s in client.get_exchange_info()['symbols'] if s['quoteAsset'] == 'USDT']

    while True:
        for s in symbols:
            try:
                check_symbol(s)
            except:
                pass
        time.sleep(60)

run()