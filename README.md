import yfinance as yf
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

plt.style.use('dark_background')

df=yf.download('GGAL.BA',auto_adjust=True)
# df=df.iloc[-252:]
def mancu_bands2_orig(df):
      """
      Calcula las bandas de mancuso y un oscilador segun el video de francisco: https://www.youtube.com/watch?v=I1jyY2eilzo&ab_channel=franciscoopciones-dolarMEP (6:20 hs)
      df es un dataframe con la estructura de yfinance.
      devuelve el mismo dataframe con las columnas:
      BI_10: Banda inferior de 10
      BS_10: Banda superior de 10
      BI_20: Banda inferior de 20
      BS_20: Banda superior de 20
      BMI: Banda media inferior
      BMS: Banda media superior

      osciladores: valor normalizado entre 0 y 1, min-max del valor de cierre entre las bandas (0: precio de cierre sobre banda inferior, 1: precio de cierre sobre banda superior)
      """
      df['BI_10']=df['Low'].rolling(10).mean()-1.5*(df['High']-df['Low']).rolling(3).mean()
      df['BS_10']=df['High'].rolling(10).mean()+1.5*(df['High']-df['Low']).rolling(3).mean()      
      df['BI_20']=df['Low'].rolling(20).mean()-2*(df['High']-df['Low']).rolling(3).mean()
      df['BS_20']=df['High'].rolling(20).mean()+2*(df['High']-df['Low']).rolling(3).mean()
      df['BMI_10']=df['BI_10']+(df['BS_10']-df['BI_10'])/2-(df['High']-df['Low']).rolling(3).mean()/2
      df['BMS_10']=df['BI_10']+(df['BS_10']-df['BI_10'])/2+(df['High']-df['Low']).rolling(3).mean()/2

      df['oscilador_10']=(df['Close']-df['BI_10'])/(df['BS_10']-df['BI_10'])
      df['oscilador_20']=(df['Close']-df['BI_20'])/(df['BS_20']-df['BI_20'])
      return df

df=mancu_bands2_orig(df)

df=df.iloc[-252:]

#figura
fig = plt.figure()
fig.set_size_inches((20, 16))
fig.suptitle('Bandas de mancuso nuevas para GGAL',fontsize=24)
ax_candle = fig.add_axes((0, 0.44, 1, 0.52))
ax_oscilador = fig.add_axes((0, 0.22, 1, 0.2), sharex=ax_candle)

# Plot candlestick chart
df['Close'].plot(ax=ax_candle)
df['BI_20'].plot(ax=ax_candle,color='r')
df['BS_20'].plot(ax=ax_candle,color='r')
df['BMI_10'].plot(ax=ax_candle,color='y')
df['BI_10'].plot(ax=ax_candle,color='b')
df['BS_10'].plot(ax=ax_candle,color='b')
df['BMS_10'].plot(ax=ax_candle,color='y')

# Plot osciladores
df['oscilador_10'].plot(ax=ax_oscilador, color='m')
df['oscilador_20'].plot(ax=ax_oscilador, color='y')
ax_oscilador.set_ylabel('Osciladores')
ax_oscilador.axhline(1,color='r',linestyle='--')
ax_oscilador.axhline(0,color='r',linestyle='--')
ax_oscilador.axhline(0.5,color='w',linestyle='--')

ax_candle.set_ylabel("precio")

ax_candle.legend()
ax_oscilador.legend()
ax_candle.get_xaxis().set_visible(False)
plt.show()
