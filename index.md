## CANDLESTICK PATTERN WITH REGRESSION TO THE MEAN STRATEGY


There are almost a million trading startegies out there that intend to outperform the market. Wether it is through Fibonacci numbers, or RSI graphs, things get more and more complicated when someones deepens into these topics. However, when it comes to technical analysis, it has been seen that a simpler strategy normally cuts a better figure.

In this blog, I intend to deliver this message. Sometimes in stock market predicions, one can profit more relying on lesser indicators, and get the most out of those.

### CANDLESTICK PATTERN WITH REGRESSION TO THE MEAN STRATEGY

![Image](https://github.com/Hupperich-Manuel/Candlestick_pattern_RTM_strategy/blob/71c85118ff5dade564d41434e18a2939063aba87/Fotos/1.png)

```python

import numpy as np
import pandas as pd
from pandas_datareader import data as pdr
import yfinance as yf
from yahoofinancials import YahooFinancials
import matplotlib.pyplot as plt
import plotly.graph_objects as go
import datetime
```

Manuel

```python
class organize_stocks:
    
    def __init__(self, data, start, end, ticker):
        self.ticker = ticker
        self.data = pd.read_csv('~/Desktop/trading_strategy/data/'+self.ticker+'.csv', index_col=0)
        self.start = start
        self.end = end
        
        
    def data_org(self):

        data = self.data[self.start:self.end]
        df = data.copy()
        df.drop(['Volume', 'Adj Close'],1, inplace=True)


        '''Conditions Startegy'''
        df['Red'] = np.where(df.Open > df.Close, 1, 0)
        df['Green'] = np.where(df.Close > df.Open, 1, 0)
        df.loc[df.Red==1, 'Condition1'] = df.Open-df.Close
        df.loc[df.Green==1, 'Condition2'] = df.Close-df.Open
        df['Condition3'] = df.Condition2-df.Condition1.shift(1)
        df.loc[df.Condition2 > df.Condition1.shift(1), 'Buy'] = 1.
        df.loc[df.Condition2 <= df.Condition1.shift(1), 'Sell'] = 0.


        '''Returns and Moving Average Strategy'''
        r = np.log(df.Close/df.Close.shift(1))
        window = [(start, start+10) for start in range(r.shape[0]-10+1)]
        #print(window)
        weights = [(r.iloc[win[0]:win[1]]).mean() for win in window]
        #print(weights)
        ret = pd.DataFrame(r.shift(8))
        ret.dropna(inplace=True)
        ret['MovAvg'] = weights
        frame = df.drop(df.index[0:10], axis=0)
        frame['MovAvg'] = ret['MovAvg']
        frame['Rets'] = ret['Close']
        frame.loc[(((1+frame.MovAvg).cumprod())>((1+frame.Rets).cumprod())), 'Buy2'] = 1.
        frame.loc[(((1+frame.MovAvg).cumprod())<((1+frame.Rets).cumprod())), 'Buy2'] = 0.
        frame.loc[(((1+frame.MovAvg).cumprod())<((1+frame.Rets).cumprod())), 'Sell2'] = 1.
        frame.loc[(abs(frame.Condition3)>(2.5*(1+frame.Rets).cumprod().std())), 'Buy3'] = 1.
    
        return frame

    
    
    def rets(self):
        
        frame = self.data_org()
        win = []
        for i in range(0, frame.shape[0]):
            if frame.Buy3.iloc[i] == 1. and frame.Buy.iloc[i] == 1. and frame.Buy2.iloc[i] == 1.:
                new_df = frame.index[frame.Rets == frame.Rets.iloc[i]][0]

                data = frame.loc[new_df:]
                for j in range(0, data.shape[0]):
                    if data.Buy2.iloc[j] == 1.:
                        win.append(data.Rets.iloc[j])
                    else:
                        break
            else:
                continue
                
        res = []
        for i in win:
            if i not in res:
                res.append(i)
                
        index = []
        for i in res:
            ix = frame.index[frame.Rets == i]
            index.append(ix)

        no = []
        for i in range(0, len(index)):
            try: 
                index[i][0] in index
                no.append(index[i])
            except IndexError:
                pass

        new = []
        for i in index:
            n = frame.loc[i].reset_index()
            new.append(n)
        if new != []:    
            trading = pd.DataFrame(np.concatenate(new))
            trading.set_index(0, inplace=True)
            trading.columns = frame.columns
            trading.drop_duplicates(keep='first',inplace=True)
            gain = round(trading.Rets.sum()*100, 3)
        else:
            gain = 'No entry done for '+self.ticker+' during the period '+self.start+' to '+self.end
        
        return gain
    
    def ytd(self):
        
        frame = self.data_org()
        return round((frame['Rets'].sum()*100), 3)
        

    def plotting(ticker):
        
        rec_spans = []

        rec_spans.append([datetime.datetime(2021, 2, 1), datetime.datetime(2021, 2, 2)])
        rec_spans.append([datetime.datetime(2021, 3, 5), datetime.datetime(2021, 3, 5)])
        rec_spans.append([datetime.datetime(2021, 3, 16), datetime.datetime(2021, 5, 4)])
        rec_spans.append([datetime.datetime(2021, 5, 24), datetime.datetime(2021, 6, 25)])
        
        

        plt.figure(figsize=[16,10])
        plt.plot((1+frame[['Rets', 'MovAvg']]).cumprod())
        plt.title('Return for '+ticker+' during the period '+start+ ': {}'.format(round((trading.Rets.sum()*100), 3))+'%'+'  [YTD :{}'.format(round((frame.Rets.sum()*100), 3))+'%]', fontsize=20)
        for i in range(len(rec_spans)):
            plt.axvspan(rec_spans[i][0], rec_spans[i][len(rec_spans[i]) - 1], alpha=0.25, color='lightgreen')```

# My name is Manuel
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

![Image](https://github.com/Hupperich-Manuel/Candlestick_pattern_RTM_strategy/blob/71c85118ff5dade564d41434e18a2939063aba87/Fotos/1.png)

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Hupperich-Manuel/Candlestick_pattern_RTM_strategy/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
