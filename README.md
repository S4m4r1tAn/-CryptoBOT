import ccxt
import pandas as pd
import swig #I had to replace talib with swig because I count installed talib from python library

# Initialize the Binance exchange object
exchange = ccxt.binance({
# We will use Brazilian money as currency or BRL
    'rateLimit': 500,
    'enableRateLimit': True,
})

# Define the symbol for the crypto you want to trade (e.g. BNB/BUSD)
symbol = 'BNB/BUSD'

# Get the historical prices
ohlcv = exchange.fetch_ohlcv(symbol)
df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])

# Calculate the RSI
df['rsi'] = swig.RSI(df['close'], timeperiod=14)

# Calculate the MACD
df['macd'], df['macdsignal'], df['macdhist'] = swig.MACD(df['close'], fastperiod=12, slowperiod=26, signalperiod=9)

# Add a moving average column
df['moving_average'] = df['close'].rolling(window=5).mean()

# Create a trading strategy
if df.loc[df.index[-1],'rsi'] > 70 and df.loc[df.index[-1],'macd'] > df.loc[df.index[-1],'macdsignal'] and df.loc[df.index[-1],'close'] > df.loc[df.index[-1],'moving_average']:
    # Place a buy order
    response = exchange.create_order(symbol, 'market', 'buy', 1)
    print(response)
elif df.loc[df.index[-1],'rsi'] < 30 and df.loc[df.index[-1],'macd'] < df.loc[df.index[-1],'macdsignal'] and df.loc[df.index[-1],'close'] < df.loc[df.index[-1],'moving_average']:
    # Place a sell order
    response = exchange.create_order(symbol, 'market', 'sell', 1)
    print(response)
else:
    print('No action taken')
