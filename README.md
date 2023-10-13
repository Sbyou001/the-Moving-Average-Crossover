# the-Moving-Average-Crossover
Trading Robot throught the MetraTrader5 using the Moving Average Crossover
# Import relevant packages

import MetaTrader5 as mt5  
import pandas as pd  
from datetime import datetime
import time
# Function to send a market order
# Sends a market order to buy or sell a specified symbol with a given volume and order type
# The order is executed at the current market price
# Returns the result of the order execution

def market_order(symbol, volume, order_type, **kwargs):
    tick = mt5.symbol_info_tick(symbol)

    order_dict = {'buy': 0, 'sell': 1}
    price_dict = {'buy': tick.ask, 'sell': tick.bid}

    request = {
        "action": mt5.TRADE_ACTION_DEAL,
        "symbol": symbol,
        "volume": volume,
        "type": order_dict[order_type],
        "price": price_dict[order_type],
        "deviation": DEVIATION,
        "magic": 100,
        "comment": "python market order",
        "type_time": mt5.ORDER_TIME_GTC,
        "type_filling": mt5.ORDER_FILLING_IOC,
    }

    order_result = mt5.order_send(request)
    print(order_result)

    return order_result
# Function to close an open position
# Closes the position associated with the given ticket number
# Searches for the position in the list of open positions retrieved from MetaTrader 5
# Inverts the order type (buy/sell) to close the position
# Sends a close order request with the necessary parameters to close the position
# Returns the result of the close order execution or a message if the ticket does not exist

def close_order(ticket):
    positions = mt5.positions_get()

    for pos in positions:
        tick = mt5.symbol_info_tick(pos.symbol)
        type_dict = {0: 1, 1: 0}  # 0 represents buy, 1 represents sell - inverting order_type to close the position
        price_dict = {0: tick.ask, 1: tick.bid}

        if pos.ticket == ticket:
            request = {
                "action": mt5.TRADE_ACTION_DEAL,
                "position": pos.ticket,
                "symbol": pos.symbol,
                "volume": pos.volume,
                "type": type_dict[pos.type],
                "price": price_dict[pos.type],
                "deviation": DEVIATION,
                "magic": 100,
                "comment": "python close order",
                "type_time": mt5.ORDER_TIME_GTC,
                "type_filling": mt5.ORDER_FILLING_IOC,
            }

            order_result = mt5.order_send(request)
            print(order_result)

            return order_result

    return 'Ticket does not exist'
# Function to get the exposure of a symbol
# Retrieves the open positions associated with the specified symbol from MetaTrader 5
# Calculates the total exposure by summing the volume of all open positions
# Returns the total exposure of the symbol

def get_exposure(symbol):
    positions = mt5.positions_get(symbol=symbol)
    if positions:
        pos_df = pd.DataFrame(positions, columns=positions[0]._asdict().keys())
        exposure = pos_df['volume'].sum()

        return exposure
# Function to generate a trading signal based on Simple Moving Average (SMA)
# Retrieves historical price data for a symbol and timeframe from MetaTrader 5
# Calculates the SMA based on the specified period
# Determines the direction of the signal (buy, sell, or flat) based on the relationship between the last closing price and the SMA
# Returns the last closing price, SMA value, and signal direction

def signal(symbol, timeframe, sma_period):
    bars = mt5.copy_rates_from_pos(symbol, timeframe, 1, sma_period)
    bars_df = pd.DataFrame(bars)

    last_close = bars_df.iloc[-1].close
    sma = bars_df.close.mean()

    direction = 'flat'
    if last_close > sma:
        direction = 'buy'
    elif last_close < sma:
        direction = 'sell'

    return last_close, sma, direction
if __name__ == '__main__':
    # Main program execution for a trading strategy

    # strategy parameters
    SYMBOL = "EURUSD"
    VOLUME = 1.0
    TIMEFRAME = mt5.TIMEFRAME_M1
    SMA_PERIOD = 10
    DEVIATION = 20

    mt5.initialize()

    while True:
        
        # calculating account exposure
        exposure = get_exposure(SYMBOL)

        # calculating last candle close and simple moving average and checking for trading signal
        last_close, sma, direction = signal(SYMBOL, TIMEFRAME, SMA_PERIOD)

        # trading logic
        if direction == 'buy':
            # if we have a BUY signal, close all short positions
            for pos in mt5.positions_get():
                if pos.type == 1:  # pos.type == 1 represent a sell order
                    close_order(pos.ticket)

            # if there are no open positions, open a new long position
            if not mt5.positions_total():
                market_order(SYMBOL, VOLUME, direction)
        elif direction == 'sell':
            # if we have a SELL signal, close all short positions
            for pos in mt5.positions_get():
                if pos.type == 0:  # pos.type == 0 represent a buy order
                    close_order(pos.ticket)

            # if there are no open positions, open a new short position
            if not mt5.positions_total():
                market_order(SYMBOL, VOLUME, direction)
             
            # Printing current trading status

        print('time: ', datetime.now())
        print('exposure: ', exposure)
        print('last_close: ', last_close)
        print('sma: ', sma)
        print('signal: ', direction)
        print('-------\n')

        # update every 1 second
        time.sleep(1)
time:  2023-06-18 16:34:39.048751
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:40.068026
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:41.076330
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:42.086626
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:43.101911
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:44.114205
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:45.128215
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:46.140999
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:47.154690
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:48.166374
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:49.177628
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:50.189484
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:51.192853
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:52.208232
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:53.218920
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:54.227617
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:55.242644
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:56.247511
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:57.253431
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:58.275364
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:34:59.294990
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:00.315403
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:01.339423
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:02.391296
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:03.392782
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:04.397633
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:05.417187
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:06.426999
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:07.460629
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:08.461655
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:09.482265
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:10.483183
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:11.492361
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:12.502258
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:13.513885
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:14.526278
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:15.545548
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:16.556885
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:17.576132
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:18.697690
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:19.708669
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:20.717375
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:21.720466
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:22.729051
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:23.735049
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:24.738058
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:25.745880
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:26.748648
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:27.764593
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:28.769479
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:29.786412
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:30.800878
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:31.811727
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:32.824770
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:33.837616
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:34.839013
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:35.854953
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:36.859283
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:37.868968
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:38.873395
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:39.895795
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:40.901202
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:41.912097
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:42.918965
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:43.940470
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:44.949251
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:45.957808
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:46.964714
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:47.977735
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:48.995916
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:50.004494
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:51.019243
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:52.023800
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:53.036705
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:54.051631
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:55.071400
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:56.076003
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:57.083677
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:58.099141
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:35:59.104093
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:00.118123
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:01.128643
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:02.131818
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:03.133281
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:04.156812
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:05.165550
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:06.174466
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:07.188817
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:08.200364
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:09.209219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:10.218075
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:11.218266
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:12.221850
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:13.226712
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:14.241356
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:15.259389
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:16.274268
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:17.278523
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:18.291777
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:19.305714
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:20.322845
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:21.340318
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:22.348795
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:23.359563
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:24.365750
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:25.379180
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:26.391842
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:27.412707
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:28.417616
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:29.425695
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:30.434316
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:31.438826
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:32.451812
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:33.467321
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:34.469962
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:35.474228
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:36.490751
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:37.496317
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:38.509331
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:39.523370
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:40.538681
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:41.553298
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:42.560819
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:43.578049
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:44.592843
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:45.611577
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:46.620202
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:47.639124
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:48.653443
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:49.673678
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:50.686052
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:51.796356
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:52.805756
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:53.819886
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:54.824014
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:55.836911
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:56.848047
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:57.865451
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:58.872637
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:36:59.872651
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:00.879492
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:01.891681
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:02.906963
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:03.908115
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:04.918078
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:05.932695
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:06.962884
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:07.970227
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:08.980296
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:09.984129
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:11.002475
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:12.008756
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:13.028917
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:14.045088
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:15.052100
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:16.062520
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:17.068807
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:18.083189
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:19.094359
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:20.102226
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:21.114004
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:22.125640
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:23.148092
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:24.158140
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:25.165337
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:26.179895
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:27.184613
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:28.199583
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:29.209692
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:30.223886
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:31.238301
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:32.238317
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:33.253225
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:34.260035
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:35.268058
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:36.274021
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:37.281346
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:38.281673
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:39.284363
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:40.291742
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:41.302408
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:42.317333
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:43.330540
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:44.333112
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:45.345870
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:46.355522
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:47.363255
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:48.374088
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:49.388404
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:50.389216
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:51.392115
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:52.407676
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:53.419709
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:54.423414
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:55.436686
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:56.449727
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:57.455342
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:58.463408
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:37:59.476354
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:00.487219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:01.491590
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:02.495353
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:03.512589
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:04.525879
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:05.534704
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:06.546730
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:07.557540
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:08.565784
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:09.575471
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:10.590112
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:11.604138
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:12.604878
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:13.608000
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:14.618117
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:15.633352
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:16.636271
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:17.648466
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:18.654973
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:19.663092
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:20.664594
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:21.665718
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:22.675544
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:23.691459
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:24.695459
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:25.709104
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:26.718396
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:27.732869
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:28.751474
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:29.760242
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:30.763803
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:31.776098
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:32.783399
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:33.790526
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:34.803756
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:35.819930
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:36.830424
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:37.837914
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:38.858160
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:39.874418
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:40.894311
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:41.910998
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:42.933155
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:43.946180
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:44.964608
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:45.980024
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:47.000022
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:48.018094
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:49.027589
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:50.036705
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:51.043291
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:52.060265
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:53.083219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:54.097971
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:55.115054
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:56.126345
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:57.146178
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:58.156993
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:38:59.165764
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:00.171356
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:01.186859
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:02.209019
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:03.226639
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:04.245603
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:05.259324
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:06.264538
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:07.278294
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:08.281485
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:09.297106
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:10.319301
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:11.324465
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:12.338573
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:13.343788
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:14.362252
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:15.373448
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:16.391337
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:17.407816
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:18.423046
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:19.434578
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:20.450277
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:21.470912
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:22.487523
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:23.502454
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:24.505999
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:25.512531
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:26.525437
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:27.544532
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:28.556481
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:29.564117
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:30.587689
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:31.601166
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:32.609763
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:33.626176
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:34.648384
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:35.661071
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:36.672774
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:37.688293
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:38.703070
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:39.723964
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:40.739861
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:41.754104
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:42.770377
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:43.780503
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:44.792350
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:45.800347
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:46.802337
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:47.814409
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:48.822681
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:49.843599
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:50.858627
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:51.873150
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:52.887820
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:53.907238
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:54.929106
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:55.942991
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:56.951202
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:57.965771
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:39:58.977941
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:00.000026
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:01.014240
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:02.019322
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:03.037942
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:04.052864
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:05.070207
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:06.087355
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:07.105073
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:08.125534
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:09.134673
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:10.150166
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:11.163885
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:12.179151
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:13.193768
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:14.208568
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:15.224042
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:16.224899
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:17.238855
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:18.252807
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:19.267674
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:20.271475
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:21.271803
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:22.284589
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:23.290826
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:24.305748
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:25.320856
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:26.320979
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:27.336069
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:28.349476
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:29.357134
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:30.369611
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:31.370836
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:32.400933
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:33.416185
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:34.431661
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:35.432334
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:36.446074
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:37.458772
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:38.463760
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:39.477793
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:40.506223
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:41.521315
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:42.535875
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:43.550921
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:44.564323
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:45.578270
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:46.591608
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:47.607202
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:48.621478
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:49.635267
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:50.650427
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:51.660957
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:52.674081
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:53.688334
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:54.703380
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:55.718306
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:56.731156
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:57.746111
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:58.760272
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:40:59.775199
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:00.778623
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:01.784180
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:02.797577
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:03.825422
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:04.840732
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:05.854161
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:06.866494
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:07.881569
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:08.895699
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:09.910779
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:10.925736
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:11.940120
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:12.954408
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:13.968301
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:14.983324
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:15.997583
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:17.012291
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:18.016447
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:19.030028
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:20.043943
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:21.057467
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:22.071718
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:23.084942
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:24.097634
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:25.112589
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:26.127307
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:27.142072
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:28.156997
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:29.163302
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:30.163557
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:31.178675
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:32.193047
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:33.208401
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:34.221219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:35.235808
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:36.252025
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:37.267157
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:38.281925
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:39.293859
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:40.308386
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:41.308670
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:42.312059
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:43.312337
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:44.327346
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:45.342549
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:46.356987
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:47.371538
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:48.387103
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:49.402329
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:50.414915
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:51.430114
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:52.443645
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:53.457782
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:54.471283
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:55.483946
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:56.497014
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:57.510079
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:58.524504
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:41:59.539405
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:00.552134
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:01.566356
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:02.580606
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:03.593915
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:04.606214
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:05.614879
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:06.628744
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:07.643081
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:08.657671
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:09.672900
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:10.687612
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:11.701193
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:12.714284
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:13.729570
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:14.742853
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:15.755430
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:16.770143
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:17.784335
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:18.798954
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:19.813634
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:20.828195
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:21.841042
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:22.853614
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:23.866867
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:24.880015
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:25.893170
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:26.906740
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:27.921457
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:28.936878
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:29.950561
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:30.962183
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:31.977583
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:32.991879
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:34.007229
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:35.022282
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:36.037042
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:37.038572
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:38.054175
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:39.069063
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:40.083897
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:41.098214
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:42.127997
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:43.129710
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:44.143318
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:45.157115
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:46.157607
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:47.172497
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:48.186448
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:49.199540
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:50.214529
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:51.228791
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:52.242481
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:53.256949
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:54.269500
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:55.282014
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:56.297620
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:57.313113
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:58.313478
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:42:59.327759
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:00.334994
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:01.338723
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:02.352823
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:03.367961
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:04.382887
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:05.396867
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:06.411787
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:07.425334
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:08.438978
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:09.452471
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:10.453084
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:11.466725
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:12.480058
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:13.493713
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:14.508335
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:15.522014
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:16.534419
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:17.546849
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:18.560910
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:19.574391
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:20.589299
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:21.604522
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:22.613953
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:23.640205
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:24.655099
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:25.668743
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:26.682202
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:27.691990
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:28.707349
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:29.718779
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:30.733481
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:31.748219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:32.762958
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:33.777916
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:34.792685
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:35.792750
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:36.807216
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:37.820891
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:38.834441
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:39.849638
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:40.864347
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:41.879119
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:42.884981
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:43.899475
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:44.913546
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:45.913714
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:46.927347
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:47.941688
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:48.956522
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:49.971171
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:50.985883
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:51.985984
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:52.986618
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:54.000841
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:55.015314
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:56.030742
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:57.044692
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:58.044763
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:43:59.058835
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:00.072878
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:01.088199
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:02.102932
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:03.118073
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:04.132541
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:05.145172
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:06.145300
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:07.157946
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:08.173346
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:09.191037
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:10.204494
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:11.221609
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:12.238554
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:13.253812
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:14.268982
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:15.282153
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:16.296005
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:17.308164
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:18.321483
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:19.322587
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:20.331871
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:21.345418
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:22.350249
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:23.365613
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:24.381476
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:25.395432
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:26.406456
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:27.429161
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:28.440171
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:29.452297
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:30.475291
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:31.492116
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:32.494416
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:33.503572
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:34.505113
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:35.505140
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:36.510670
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:37.524981
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:38.534942
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:39.548625
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:40.563297
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:41.573688
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:42.579377
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:43.593222
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:44.611320
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:45.625793
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:46.635797
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:47.637227
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:48.643387
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:49.644563
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:50.659600
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:51.665025
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:52.668990
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:53.671522
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:54.680266
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:55.684568
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:56.685273
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:57.693547
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:58.705175
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:44:59.714250
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:00.722264
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:01.729425
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:02.750080
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:03.760534
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:04.763797
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:05.766449
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:06.769441
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:07.775625
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:08.777937
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:09.781925
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:10.785875
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:11.798147
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:12.804283
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:13.814977
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:14.826766
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:15.834851
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:16.847004
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:17.861053
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:18.865779
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:19.869194
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:20.869737
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:21.873928
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:22.875476
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:23.880763
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:24.883090
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:25.901548
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:26.905606
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:27.911677
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:28.925754
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:29.930206
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:30.934301
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:31.939431
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:32.945234
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:33.960837
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:34.967720
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:35.973165
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:36.988846
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:38.004959
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:39.008804
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:40.012461
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:41.018390
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:42.019410
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:43.024285
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:44.028764
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:45.044043
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:46.067409
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:47.076303
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:48.085767
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:49.090830
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:50.092297
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:51.096027
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:52.102175
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:53.103424
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:54.109227
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:55.114272
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:56.130831
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:57.137957
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:58.152528
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:45:59.153949
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:00.158707
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:01.160958
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:02.168534
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:03.184119
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:04.190276
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:05.196469
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:06.210484
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:07.220420
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:08.221212
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:09.224692
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:10.245701
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:11.258178
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:12.260759
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:13.260823
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:14.267668
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:15.279318
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:16.291284
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:17.309067
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:18.323038
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:19.325733
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:20.336068
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:21.343934
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:22.351100
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:23.365859
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:24.370734
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:25.376706
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:26.393412
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:27.399983
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:28.400262
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:29.408005
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:30.422926
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:31.424641
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:32.427990
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:33.434207
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:34.434302
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:35.439207
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:36.460962
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:37.465900
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:38.480144
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:39.484389
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:40.485757
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:41.489182
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:42.494240
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:43.497504
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:44.503344
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:45.503617
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:46.513672
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:47.514386
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:48.522716
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:49.523851
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:50.532847
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:51.545483
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:52.546783
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:53.551079
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:54.555113
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:55.558110
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:56.561859
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:57.571184
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:58.572825
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:46:59.584018
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:00.597353
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:01.598107
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:02.599069
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:03.603733
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:04.616542
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:05.623392
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:06.637111
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:07.648829
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:08.660733
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:09.674178
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:10.683223
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:11.685116
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:12.703728
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:13.716173
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:14.727832
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:15.733623
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:16.736981
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:17.746021
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:18.760017
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:19.761344
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:20.768329
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:21.772052
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:22.776200
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:23.781578
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:24.796897
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:25.803276
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:26.813638
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:27.821560
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:28.827641
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:29.831848
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:30.833889
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:31.836945
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:32.839531
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:33.845847
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:34.848434
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:35.857124
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:36.862287
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:37.870539
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:38.883648
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:39.898935
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:40.911015
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:41.918874
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:42.935472
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:43.956072
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:44.967159
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:45.976145
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:46.997258
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:48.022743
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:49.040324
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:50.055241
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:51.070743
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:52.090346
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:53.097014
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:54.110223
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:55.124450
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:56.142264
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:57.155382
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:58.163722
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:47:59.181952
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:00.192749
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:01.209094
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:02.217787
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:03.231239
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:04.243444
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:05.247784
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:06.262326
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:07.273281
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:08.291712
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:09.306825
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:10.324703
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:11.342082
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:12.351573
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:13.366812
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:14.384035
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:15.399158
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:16.416169
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:17.428229
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:18.448234
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:19.462548
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:20.478759
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:21.497258
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:22.519894
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:23.536057
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:24.556009
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:25.570671
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:26.589319
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:27.601639
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:28.618272
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:29.638299
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:30.645509
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:31.662066
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:32.672383
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:33.691487
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:34.707981
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:35.721621
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:36.736293
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:37.751773
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:38.773630
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:39.788813
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:40.800042
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:41.818195
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:42.830737
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:43.845213
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:44.858158
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:45.869313
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:46.888348
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:47.907017
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:48.918371
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:49.935141
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:50.944046
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:51.959753
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:52.980755
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:53.995861
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:55.017704
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:56.032105
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:57.041395
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:58.055817
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:48:59.065245
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:00.079304
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:01.100677
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:02.115591
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:03.129358
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:04.144848
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:05.154963
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:06.171803
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:07.184687
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:08.200578
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:09.215772
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:10.229848
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:11.242029
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:12.256555
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:13.270539
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:14.283340
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:15.292924
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:16.308021
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:17.324699
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:18.341832
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:19.355056
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:20.368522
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:21.376642
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:22.393888
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:23.407588
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:24.420964
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:25.442661
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:26.458637
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:27.470710
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:28.481607
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:29.504862
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:30.534316
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:31.546523
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:32.558638
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:33.566976
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:34.584755
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:35.598082
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:36.616105
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:37.633262
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:38.644959
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:39.653855
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:40.673685
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:41.684711
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:42.698850
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:43.713642
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:44.729387
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:45.744125
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:46.759471
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:47.770866
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:48.783292
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:49.789842
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:50.812455
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:51.824825
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:52.830681
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:53.843574
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:54.866683
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:55.879803
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:56.893401
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:57.909021
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:58.921829
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:49:59.934973
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:00.946670
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:01.964194
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:02.977220
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:03.989873
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:05.005761
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:06.018302
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:07.027142
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:08.038611
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:09.046487
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:10.063055
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:11.077065
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:12.092216
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:13.108868
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:14.128946
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:15.141079
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:16.152524
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:17.175006
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:18.193967
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:19.205739
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:20.208577
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:21.219903
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:22.241810
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:23.262921
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:24.278481
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:25.300303
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:26.318217
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:27.337336
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:28.355217
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:29.374029
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:30.397541
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:31.417525
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:32.435859
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:33.447684
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:34.461163
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:35.478614
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:36.494653
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:37.503740
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:38.510538
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:39.521595
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:40.534401
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:41.558078
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:42.565869
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:43.587982
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:44.596607
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:45.614073
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:46.629326
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:47.638622
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:48.655997
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:49.676420
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:50.688975
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:51.705639
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:52.719531
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:53.734025
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:54.751728
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:55.769526
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:56.778568
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:57.793952
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:58.816979
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:50:59.827639
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:00.836793
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:01.856173
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:02.868249
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:03.876982
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:04.885152
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:05.901513
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:06.908366
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:07.917822
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:08.928873
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:09.939977
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:10.954439
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:11.969649
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:12.987690
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:13.999064
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:15.016436
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:16.033634
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:17.048719
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:18.065119
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:19.077565
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:20.088473
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:21.104719
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:22.122569
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:23.136127
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:24.149271
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:25.170619
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:26.181465
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:27.196386
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:28.209731
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:29.216369
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:30.229550
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:31.240912
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:32.248954
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:33.263803
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:34.273699
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:35.285395
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:36.294377
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:37.307427
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:38.316432
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:39.340650
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:40.352415
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:41.365820
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:42.377887
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:43.389145
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:44.401895
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:45.414638
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:46.427997
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:47.446297
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:48.456546
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:49.474173
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:50.483685
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:51.494951
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:52.510243
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:53.527332
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:54.537700
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:55.549470
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:56.562924
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:57.575150
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:58.593419
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:51:59.608366
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:00.627029
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:01.632202
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:02.643977
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:03.659094
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:04.661642
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:05.672105
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:06.684471
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:07.699675
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:08.714116
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:09.721913
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:10.737490
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:11.739957
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:12.745022
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:13.757159
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:14.765255
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:15.776498
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:16.780728
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:17.793993
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:18.797861
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:19.806301
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:20.809875
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:21.823519
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:22.830588
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:23.844126
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:24.867645
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:25.875077
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:26.883554
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:27.887770
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:28.899356
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:29.908538
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:30.910189
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:31.929634
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:32.943553
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:33.945745
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:34.955880
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:35.959474
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:36.973472
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:37.979117
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:38.980527
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:39.987237
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:40.999043
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:42.002135
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:43.008101
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:44.011050
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:45.014443
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:46.017108
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:47.021603
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:48.026221
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:49.044401
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:50.046618
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:51.051358
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:52.054126
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:53.055640
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:54.058917
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:55.070258
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:56.074910
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:57.098596
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:58.099136
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:52:59.110394
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:00.111511
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:01.119235
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:02.131486
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:03.139119
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:04.147286
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:05.153940
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:06.168254
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:07.172587
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:08.175072
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:09.186055
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:10.199822
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:11.213074
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:12.233077
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:13.234550
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:14.237977
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:15.240235
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:16.251148
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:17.256575
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:18.264701
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:19.285772
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:20.294855
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:21.297818
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:22.301900
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:23.319460
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:24.319909
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:25.339385
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:26.350873
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:27.365174
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:28.376006
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:29.379043
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:30.383314
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:31.386655
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:32.401199
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:33.416257
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:34.427578
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:35.438537
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:36.442378
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:37.446033
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:38.449926
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:39.452988
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:40.463393
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:41.475368
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:42.478590
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:43.481057
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:44.485199
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:45.488530
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:46.492626
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:47.495328
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:48.499858
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:49.519506
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:50.532570
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:51.539728
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:52.541583
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:53.546826
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:54.551711
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:55.553599
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:56.557999
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:57.561989
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:58.565040
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:53:59.584418
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:00.593709
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:01.605367
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:02.608165
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:03.612167
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:04.616256
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:05.620007
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:06.623251
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:07.627309
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:08.630393
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:09.649771
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:10.657199
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:11.671531
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:12.674830
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:13.677494
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:14.682005
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:15.685738
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:16.689620
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:17.692644
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:18.696885
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:19.715429
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:20.720953
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:21.721479
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:22.724035
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:23.728158
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:24.731397
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:25.735396
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:26.740171
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:27.742057
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:28.746395
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:29.757223
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:30.767932
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:31.771820
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:32.774929
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:33.778784
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:34.781981
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:35.786069
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:36.788787
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:37.793083
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:38.795883
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:39.804071
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:40.817505
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:41.821699
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:42.824804
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:43.826846
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:44.831577
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:45.835513
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:46.838561
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:47.842927
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:48.857369
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:49.869712
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:50.883083
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:51.886979
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:52.890088
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:53.894768
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:54.897839
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:55.901112
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:56.905181
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:57.908162
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:58.919147
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:54:59.933943
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:00.948498
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:01.952185
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:02.956004
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:03.960480
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:04.963635
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:05.966744
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:06.970613
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:07.972381
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:08.984759
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:10.001439
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:11.014451
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:12.026791
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:13.035052
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:14.039191
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:15.057696
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:16.066287
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:17.081459
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:18.084912
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:19.088883
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:20.098412
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:21.109923
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:22.113317
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:23.116320
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:24.120050
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:25.139754
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:26.154891
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:27.162901
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:28.166897
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:29.169768
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:30.173573
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:31.177250
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:32.179289
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:33.182653
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:34.186448
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:35.189590
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:36.193765
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:37.196753
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:38.200445
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:39.204854
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:40.210082
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:41.210459
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:42.227296
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:43.234021
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:44.236122
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:45.249449
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:46.259217
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:47.263419
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:48.266268
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:49.270410
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:50.276926
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:51.290612
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:52.294440
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:53.298443
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:54.301527
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:55.317389
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:56.324669
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:57.328413
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:58.331619
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:55:59.335587
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:00.337166
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:01.340997
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:02.344203
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:03.348409
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:04.352061
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:05.359402
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:06.359849
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:07.362509
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:08.367010
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:09.376206
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:10.387014
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:11.391090
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:12.394860
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:13.398365
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:14.417369
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:15.424492
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:16.425186
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:17.427988
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:18.432055
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:19.437823
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:20.453111
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:21.457517
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:22.460348
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:23.463755
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:24.476471
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:25.493780
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:26.505810
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:27.510158
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:28.513265
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:29.524124
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:30.534198
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:31.538117
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:32.543121
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:33.544665
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:34.563863
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:35.573133
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:36.586836
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:37.591187
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:38.594909
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:39.597839
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:40.599636
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:41.602667
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:42.606682
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:43.610689
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:44.629439
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:45.638484
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:46.653304
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:47.656264
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:48.660128
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:49.663622
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:50.665026
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:51.669049
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:52.671802
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:53.676045
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:54.690260
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:55.705297
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:56.718734
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:57.722598
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:58.725796
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:56:59.732427
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:00.747017
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:01.750140
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:02.753835
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:03.757846
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:04.760928
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:05.771817
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:06.784135
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:07.788039
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:08.790815
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:09.802593
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:10.813090
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:11.816168
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:12.820141
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:13.823201
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:14.842026
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:15.849515
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:16.851168
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:17.853209
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:18.857400
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:19.867936
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:20.878475
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:21.882447
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:22.884475
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:23.888762
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:24.907316
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:25.915753
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:26.915818
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:27.918520
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:28.922585
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:29.929078
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:30.943038
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:31.947633
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:32.950756
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:33.953892
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:34.967584
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:35.984904
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:36.996937
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:38.000787
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:39.003582
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:40.011643
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:41.025026
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:42.027679
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:43.032295
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:44.035799
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:45.040558
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:46.043141
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:47.045977
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:48.050281
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:49.053520
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:50.062652
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:51.064744
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:52.079427
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:53.085652
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:54.086672
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:55.088184
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:56.092806
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:57.097536
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:58.104453
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:57:59.105543
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:00.110681
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:01.113709
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:02.116753
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:03.123799
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:04.138179
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:05.147287
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:06.161939
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:07.163131
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:08.168517
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:09.170448
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:10.174075
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:11.179029
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:12.181624
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:13.186811
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:14.189645
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:15.205747
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:16.211067
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:17.220727
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:18.236451
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:19.250952
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:20.258654
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:21.275125
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:22.288995
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:23.301531
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:24.306907
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:25.322461
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:26.330956
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:27.343905
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:28.354147
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:29.354558
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:30.362532
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:31.386007
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:32.389903
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:33.401145
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:34.413714
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:35.431610
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:36.439879
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:37.445709
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:38.453881
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:39.461847
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:40.470992
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:41.478593
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:42.487822
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:43.491762
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:44.503772
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:45.522442
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:46.526444
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:47.540234
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:48.556429
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:49.571002
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:50.572555
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:51.582112
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:52.593801
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:53.598393
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:54.633667
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:55.648752
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:56.655208
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:57.655514
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:58.691772
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:58:59.709892
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:00.714064
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:01.722332
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:02.723660
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:03.739929
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:04.758082
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:05.767248
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:06.782018
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:07.798727
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:08.809343
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:09.822732
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:10.837503
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:11.841318
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:12.841793
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:13.862934
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:14.894365
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:15.915247
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:16.922753
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:17.939570
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:18.956664
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:19.973325
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:20.984370
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:22.007013
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:23.027027
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:24.041356
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:25.053578
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:26.069171
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:27.086470
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:28.102183
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:29.119334
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:30.142217
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:31.162062
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:32.168164
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:33.181992
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:34.189623
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:35.203647
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:36.213239
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:37.229563
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:38.242218
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:39.249503
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:40.268768
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:41.287622
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:42.298114
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:43.322826
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:44.335089
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:45.352973
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:46.364871
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:47.381165
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:48.403931
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:49.419585
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:50.427414
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:51.448856
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:52.458525
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:53.466904
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:54.482289
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:55.497691
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:56.506498
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:57.518668
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:58.528724
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 16:59:59.539181
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:00.558392
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:01.567016
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:02.577480
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:03.591008
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:04.599411
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:05.616169
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:06.630475
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:07.641133
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:08.661462
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:09.672219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:10.690546
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:11.702628
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:12.715616
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:13.729519
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:14.748138
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:15.768726
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:16.779999
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:17.792549
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:18.811542
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:19.827783
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:20.839322
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:21.858542
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:22.877592
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:23.891449
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:24.911742
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:25.918219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:26.934390
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:27.947444
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:28.957091
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:29.978898
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:30.987219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:32.009416
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:33.027667
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:34.040728
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:35.055672
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:36.068318
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:37.076933
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:38.092005
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:39.097277
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:40.112859
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:41.121257
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:42.134565
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:43.142172
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:44.161220
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:45.172211
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:46.191018
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:47.207027
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:48.228670
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:49.233314
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:50.251510
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:51.270089
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:52.283111
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:53.301434
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:54.306979
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:55.325629
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:56.339269
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:57.358935
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:58.368830
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:00:59.382896
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:00.397769
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:01.403706
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:02.414434
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:03.428986
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:04.449099
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:05.467528
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:06.486017
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:07.497515
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:08.508631
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:09.524100
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:10.529689
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:11.543057
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:12.553658
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:13.566876
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:14.581761
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:15.595168
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:16.611528
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:17.627546
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:18.642571
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:19.647594
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:20.662777
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:21.677649
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:22.685626
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:23.704435
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:24.711287
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:25.720794
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:26.732489
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:27.749139
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:28.762038
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:29.775749
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:30.794986
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:31.813651
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:32.824106
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:33.841921
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:34.856859
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:35.863991
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:36.875034
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:37.895687
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:38.910369
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:39.924352
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:40.939439
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:41.949802
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:42.964497
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:43.976615
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:44.999464
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:46.017645
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:47.021182
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:48.034716
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:49.047496
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:50.064590
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:51.079897
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:52.096542
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:53.134297
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:54.155263
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:55.172948
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:56.189536
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:57.200226
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:58.209520
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:01:59.225141
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:00.231815
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:01.240342
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:02.254190
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:03.261864
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:04.274997
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:05.290314
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:06.310957
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:07.321452
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:08.329618
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:09.343554
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:10.354222
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:11.367884
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:12.381793
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:13.392399
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:14.396005
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:15.411476
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:16.432598
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:17.455258
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:18.470865
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:19.482934
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:20.496390
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:21.508271
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:22.522053
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:23.548114
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:24.555577
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:25.565924
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:26.575553
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:27.594093
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:28.602948
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:29.618060
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:30.636096
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:31.646393
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:32.661642
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:33.669672
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:34.683817
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:35.691241
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:36.706779
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:37.719574
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:38.728750
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:39.736537
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:40.745073
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:41.759307
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:42.773150
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:43.791292
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:44.812064
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:45.829879
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:46.844190
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:47.854615
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:48.865348
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:49.881056
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:50.887518
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:51.900814
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:52.919852
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:53.925820
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:54.944953
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:55.958365
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:56.968067
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:57.984250
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:02:59.007934
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:00.025654
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:01.031564
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:02.040016
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:03.050973
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:04.064989
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:05.075452
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:06.087971
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:07.098090
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:08.115467
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:09.136734
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:10.143547
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:11.158192
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:12.173690
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:13.190478
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:14.208558
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:15.221501
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:16.239435
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:17.254031
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:18.268745
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:19.275018
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:20.282134
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:21.287633
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:22.303920
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:23.311726
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:24.326018
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:25.335492
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:26.349431
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:27.355600
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:28.375127
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:29.396983
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:30.411496
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:31.428966
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:32.442919
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:33.450185
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:34.469931
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:35.483601
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:36.498051
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:37.507306
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:38.527484
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:39.533243
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:40.553061
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:41.574356
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:42.581017
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:43.603238
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:44.621543
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:45.631423
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:46.649319
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:47.662609
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:48.682692
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:49.696012
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:50.715238
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:51.732721
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:52.748021
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:53.754962
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:54.761982
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:55.780081
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:56.801944
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:57.820053
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:58.830524
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:03:59.843397
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:00.849116
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:01.865116
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:02.872407
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:03.884537
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:04.897927
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:05.913843
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:06.936076
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:07.945263
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:08.949869
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:09.961991
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:10.968291
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:11.982670
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:12.990608
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:14.008605
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:15.020888
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:16.027625
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:17.046854
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:18.059319
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:19.069665
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:20.072988
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:21.088417
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:22.094676
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:23.113256
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:24.128683
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:25.140918
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:26.159780
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:27.172020
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:28.181328
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:29.194061
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:30.216785
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:31.235696
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:32.254083
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:33.272710
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:34.287649
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:35.299182
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:36.316806
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:37.327088
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:38.342534
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:39.357516
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:40.369760
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:41.384066
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:42.390255
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:43.398602
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:44.409101
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:45.424396
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:46.437429
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:47.446249
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:48.455123
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:49.471240
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:50.481717
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:51.492461
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:52.502073
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:53.515882
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:54.533461
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:55.547735
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:56.564937
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:57.568643
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:58.579788
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:04:59.585481
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:00.597345
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:01.611805
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:02.627365
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:03.631142
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:04.639906
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:05.646240
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:06.655398
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:07.666996
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:08.683715
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:09.691718
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:10.705922
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:11.720959
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:12.734129
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:13.754167
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:14.762999
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:15.788220
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:16.800283
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:17.818950
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:18.836301
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:19.853262
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:20.862330
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:21.872968
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:22.884412
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:23.900193
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:24.906312
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:25.914604
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:26.925619
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:27.940398
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:28.961177
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:29.984165
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:31.000956
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:32.011325
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:33.023999
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:34.036390
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:35.048431
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:36.066291
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:37.077544
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:38.085979
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:39.094904
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:40.114581
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:41.129608
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:42.149681
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:43.162456
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:44.172737
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:45.186057
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:46.196383
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:47.208354
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:48.211540
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:49.220633
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:50.239049
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:51.247625
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:52.261361
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:53.270288
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:54.286672
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:55.290337
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:56.302707
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:57.311749
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:58.328599
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:05:59.339199
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:00.348718
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:01.354445
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:02.363899
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:03.384706
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:04.397754
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:05.407787
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:06.414332
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:07.419204
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:08.432661
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:09.442962
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:10.455571
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:11.484622
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:12.505807
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:13.517296
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:14.527545
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:15.548363
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:16.566563
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:17.588171
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:18.605750
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:19.620241
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:20.624624
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:21.629069
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:22.639649
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:23.656628
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:24.669463
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:25.685716
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:26.691374
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:27.696635
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:28.706427
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:29.710867
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:30.725293
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:31.730598
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:32.743377
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:33.750246
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:34.761651
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:35.778155
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:36.795584
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:37.812709
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:38.829748
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:39.838842
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:40.851088
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:41.863841
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:42.874730
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:43.882857
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:44.902767
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:45.924718
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:46.940828
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:47.953027
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:48.962382
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:49.974454
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:50.983974
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:51.996988
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:53.010010
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:54.016706
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:55.031354
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:56.049056
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:57.057848
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:58.070062
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:06:59.095170
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:00.126450
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:01.175763
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:02.192590
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:03.209652
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:04.223304
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:05.269713
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:06.292756
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:07.308992
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:08.319146
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:09.327929
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:10.334723
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:11.343318
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:12.355273
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:13.375020
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:14.383737
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:15.395193
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:16.410525
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:17.427452
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:18.433301
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:19.441020
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:20.454939
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:21.468212
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:22.489566
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:23.502383
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:24.518570
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:25.529580
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:26.543542
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:27.558194
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:28.573913
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:29.583021
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:30.595201
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:31.610735
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:32.616208
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:33.626037
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:34.633247
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:35.647776
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:36.659918
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:37.673047
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:38.686863
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:39.701635
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:40.717904
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:41.733642
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:42.745149
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:43.757705
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:44.766550
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:45.777428
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:46.789639
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:47.805379
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:48.820758
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:49.829367
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:50.840221
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:51.855300
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:52.875580
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:53.886837
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:54.901277
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:55.914850
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:56.931293
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:57.942127
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:58.957969
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:07:59.975749
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:00.983350
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:01.992958
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:03.001671
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:04.010435
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:05.014541
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:06.017092
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:07.026254
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:08.041521
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:09.054136
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:10.069640
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:11.085036
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:12.100412
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:13.115681
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:14.131192
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:15.146580
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:16.161834
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:17.177336
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:18.191914
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:19.244642
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:20.262149
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:21.264900
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:22.278374
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:23.279312
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:24.285884
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:25.289012
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:26.308502
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:27.321134
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:28.337123
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:29.346120
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:30.358368
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:31.372962
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:32.384469
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:33.391650
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:34.398489
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:35.413158
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:36.423585
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:37.438270
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:38.445240
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:39.457260
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:40.467969
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:41.494234
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:42.504122
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:43.525664
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:44.539446
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:45.555501
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:46.558425
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:47.576637
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:48.599415
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:49.615366
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:50.625345
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:51.649404
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:52.661280
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:53.670310
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:54.692185
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:55.703812
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:56.721403
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:57.734388
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:58.753025
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:08:59.771226
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:00.780467
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:01.800221
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:02.821084
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:03.840697
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:04.865272
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:05.881932
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:06.891702
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:07.905627
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:08.917411
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:09.936697
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:10.955179
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:11.974844
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:12.986766
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:13.992706
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:15.020758
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:16.037572
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:17.051625
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:18.067048
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:19.082576
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:20.092824
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:21.111211
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:22.128823
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:23.142898
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:24.156158
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:25.172660
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:26.191453
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:27.210125
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:28.218387
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:29.233985
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:30.244326
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:31.264638
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:32.283056
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:33.300047
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:34.317598
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:35.333202
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:36.347550
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:37.356559
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:38.371176
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:39.385256
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:40.399665
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:41.408390
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:42.429246
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:43.439916
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:44.452032
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:45.468056
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:46.488464
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:47.500766
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:48.510359
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:49.517813
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:50.526064
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:51.543348
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:52.558539
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:53.575998
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:54.589161
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:55.601670
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:56.616858
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:57.631905
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:58.645671
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:09:59.661515
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:00.668877
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:01.681023
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:02.696273
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:03.709152
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:04.712165
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:05.728288
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:06.741196
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:07.749765
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:08.755504
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:09.760407
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:10.773170
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:11.782969
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:12.798737
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:13.813740
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:14.825525
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:15.837026
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:16.845742
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:17.861719
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:18.872114
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:19.889931
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:20.898294
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:21.919715
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:22.929957
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:23.945110
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:24.956678
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:25.965834
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:26.979139
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:27.992974
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:29.009963
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:30.025733
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:31.029807
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:32.040050
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:33.047318
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:34.060698
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:35.076119
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:36.087943
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:37.101930
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:38.112751
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:39.122735
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:40.139553
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:41.158234
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:42.172480
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:43.180007
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:44.191235
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:45.214485
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:46.233280
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:47.243975
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:48.255135
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:49.267560
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:50.279011
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:51.295693
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:52.300487
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:53.321403
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:54.336234
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:55.352017
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:56.365633
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:57.380618
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:58.393513
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:10:59.402768
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:00.417882
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:01.430785
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:02.437805
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:03.455402
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:04.474899
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:05.488493
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:06.508348
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:07.516272
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:08.523681
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:09.526068
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:10.528633
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:11.550153
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:12.561392
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:13.577983
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:14.589305
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:15.593880
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:16.596697
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:17.606068
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:18.607731
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:19.618800
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:20.624880
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:21.639409
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:22.643662
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:23.648986
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:24.665157
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:25.670042
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:26.681787
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:27.698542
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:28.707570
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:29.720555
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:30.722259
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:31.724447
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:32.742327
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:33.764733
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:34.774762
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:35.799106
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:36.805674
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:37.817868
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:38.826147
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:39.838148
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:40.848424
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:41.849503
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:42.870208
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:43.886614
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:44.902404
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:45.911289
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:46.926290
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:47.938347
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:48.945333
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:49.963253
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:50.979976
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:51.994403
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:53.012417
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:54.023191
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:55.033043
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:56.054303
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:57.067389
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:58.090766
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:11:59.109295
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:00.119742
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:01.134705
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:02.153543
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:03.157772
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:04.162272
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:05.173507
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:06.191274
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:07.205678
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:08.209447
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:09.219741
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:10.221007
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:11.235510
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:12.244440
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:13.249655
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:14.261903
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:15.278389
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:16.289134
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:17.304561
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:18.314105
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:19.314988
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:20.318951
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:21.328788
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:22.331261
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:23.342373
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:24.363150
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:25.378528
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:26.393654
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:27.409776
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:28.416622
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:29.424651
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:30.429014
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:31.431384
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:32.439013
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:33.449084
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:34.464284
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:35.475801
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:36.479022
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:37.488776
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:38.494114
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:39.512659
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:40.529376
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:41.541593
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:42.560164
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:43.577964
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:44.597036
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:45.598562
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:46.611287
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:47.621135
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:48.629640
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:49.654422
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:50.668274
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:51.690127
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:52.694870
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:53.706906
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:54.717521
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:55.717889
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:56.724812
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:57.741945
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:58.743369
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:12:59.762767
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:00.777582
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:01.792208
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:02.807101
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:03.807307
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:04.808652
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:05.824065
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:06.829196
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:07.831750
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:08.847884
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:09.862750
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:10.888024
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:11.906674
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:12.922491
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:13.931443
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:14.946711
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:15.962614
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:16.987256
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:17.998684
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:19.015194
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:20.031781
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:21.041833
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:22.047658
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:23.061246
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:24.074429
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:25.084961
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:26.092458
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:27.107966
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:28.116855
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:29.125098
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:30.143720
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:31.150348
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:32.163246
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:33.180679
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:34.192759
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:35.213660
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:36.223113
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:37.235157
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:38.252834
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:39.265794
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:40.278661
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:41.283535
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:42.295880
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:43.303963
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:44.319810
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:45.333283
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:46.347860
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:47.357697
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:48.369980
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:49.383202
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:50.394895
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:51.409374
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:52.430480
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:53.442343
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:54.454168
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:55.465279
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:56.482953
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:57.503727
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:58.516273
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:13:59.534398
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:00.551504
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:01.563616
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:02.577422
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:03.592577
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:04.599821
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:05.616947
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:06.625267
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:07.641824
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:08.657481
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:09.678634
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:10.691023
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:11.713208
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:12.729530
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:13.743909
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:14.755333
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:15.768342
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:16.778448
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:17.790585
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:18.803950
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:19.824073
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:20.832278
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:21.844975
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:22.852734
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:23.865903
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:24.872296
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:25.879479
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:26.894433
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:27.905774
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:28.911014
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:29.931745
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:30.936084
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:31.946365
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:32.963393
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:33.981089
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:34.996178
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:36.014652
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:37.032482
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:38.042645
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:39.061016
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:40.066912
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:41.072219
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:42.090609
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:43.112386
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:44.133003
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:45.153986
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:46.175795
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:47.210851
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:48.227147
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:49.240932
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:50.250770
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:51.266931
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:52.286410
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:53.293566
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:54.307959
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:55.322945
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:56.335972
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:57.345341
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:58.364362
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:14:59.379703
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:00.389961
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:01.406155
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:02.419111
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:03.431283
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:04.450401
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:05.470111
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:06.482054
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:07.498998
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:08.514325
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:09.528826
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:10.550497
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:11.560457
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:12.579173
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:13.595198
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:14.608333
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:15.621177
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:16.628647
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:17.648234
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:18.652859
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:19.658494
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:20.663796
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:21.684438
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:22.709193
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:23.742443
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:24.762954
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:25.783580
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:26.809321
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:27.823844
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:28.835652
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:29.852884
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:30.863484
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:31.867572
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:32.880575
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:33.900936
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:34.915532
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:35.935073
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:36.944722
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:37.956719
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:38.958847
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:39.966628
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:40.981650
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:41.998747
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:43.003205
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:44.008008
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:45.014638
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:46.024198
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:47.031793
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:48.055051
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:49.058213
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:50.070940
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:51.072244
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:52.102176
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:53.113965
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:54.125045
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:55.137612
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:56.147154
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:57.155302
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:58.165043
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:15:59.180765
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:00.193558
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:01.204593
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:02.212766
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:03.229481
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:04.242593
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:05.259447
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:06.283433
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:07.297990
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:08.313527
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:09.339199
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:10.349874
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:11.360673
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:12.372094
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:13.390354
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:14.401388
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:15.421525
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:16.440308
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:17.454057
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:18.467101
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:19.481999
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:20.499526
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:21.521472
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:22.528580
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:23.545236
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:24.562948
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:25.576854
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:26.591215
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:27.610016
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:28.628814
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:29.649934
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:30.666852
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:31.670777
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:32.679625
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:33.700975
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:34.722399
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:35.735863
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:36.750913
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:37.781039
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:38.791540
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:39.806980
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:40.813572
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:41.822272
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:42.825594
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:43.830040
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:44.843368
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:45.863140
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:46.870970
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:47.883080
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:48.897322
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:49.908737
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:50.924565
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:51.947538
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:52.962647
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:53.983410
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:54.990587
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:56.002790
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:57.024441
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:58.041490
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:16:59.053530
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:00.059670
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:01.075367
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:02.092430
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:03.101004
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:04.122188
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:05.133289
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:06.150713
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:07.174513
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:08.189539
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:09.205387
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:10.207488
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:11.219870
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:12.237396
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:13.256139
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:14.267223
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:15.286701
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:16.298319
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:17.323295
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:18.343463
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:19.357239
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:20.360605
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:21.372129
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:22.379274
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:23.403468
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:24.419435
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:25.437941
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:26.446056
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:27.454306
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:28.469688
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:29.485079
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:30.496214
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:31.519496
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:32.548414
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:33.553008
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:34.568481
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:35.577453
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:36.592950
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:37.592965
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:38.608339
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:39.609389
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:40.613442
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:41.628600
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:42.643995
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:43.659379
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:44.674799
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:45.690144
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:46.708211
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:47.735558
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:48.753186
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:49.772363
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:50.785693
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:51.787271
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:52.809357
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:53.815877
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:54.825931
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:55.845565
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:56.862323
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:57.878078
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:58.893293
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:17:59.916997
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:18:00.935662
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:18:01.951900
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:18:02.967426
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:18:03.982792
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:18:05.011085
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:18:06.017923
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

time:  2023-06-18 17:18:07.018598
exposure:  1.0
last_close:  1.09403
sma:  1.0939800000000002
signal:  buy
-------

The result gives us the timestamp of the current time and shows the account exposure, which is the current position volume. Then it prints us the last closing price of the symbol and displays the calculated value of the simple moving average SMA. As well as it shows the trading signal generated based on the comparaison between the last closing and SMA
To run the script, we open command prompt and go to the location where our python file is located, then after running it, we are going to see the rate is running and we are going to loop and to ckeck for new signals, As I have noticed, that we open a newposition because of the prices are above the moving average.

 
