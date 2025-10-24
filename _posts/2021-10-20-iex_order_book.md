---
layout: post
title: IEX Order Book - Reading the Data
permalink: /iex_1/
---
# The IEX Order Book - Reading the Data
I recently gave a talk to the [Python Frederick Meetup](https://www.meetup.com/python-frederick/) about Pandas and wanted to show some financial data since that's what I've been working with lately. Specifically, I wanted something that had real time quotes and trades, but at best most sources give you the high, low, and median price aggregated over something like an hour or a day. There are a lot of sources of market data, and they have most definitely put a price on that data, a high price.

[NYSE TAQ](https://www.nyse.com/market-data/historical) is about \\$1000/month for the first 12 months and \$500/month to go farther back in time. [NASDAQ TotalView](https://www.nasdaq.com/solutions/nasdaq-totalview) and similar products have a pretty complex fee structure, so I'm not *entirely* sure how much it costs. You can at least see the real time data on your RobinHood (Gold account) or Charles Schwab account, but historical data's going to cost you.

Which brings us to the [Investor's Exchange](https://iextrading.com/), or IEX.  Featured in the Michael Lewis book [Flash Boys](https://en.wikipedia.org/wiki/Flash_Boys), they do their best to offer access to the market with protections in place that favor long-term over short-term strategies, keeping in mind 'long term' might still mean only holding a stock for a millisecond. They're a pretty small compared to NYSE or NASDAQ, but they allow you to download pretty detailed historical data for free...as a dump of network packets.

### What is an order book?

Why on earth do I want to see the order book in the first place? What is it? Every time a broker wants to make a trade, they put an an order (I want to buy 200 shares of stock A for $100 each). Until the order is canceled, or a seller is found that is willing to meet the buyer at their price, the order 'rests on the book.' Most heavily traded stocks do not have orders that rest long, unless the number of shares is exceptionally large or the price point is completely unreasonable. Here's an extremely simplified view of what an order book might look like.

|time|side|quantity|price|
|----|----|--------|-----|
|9:30|B   |100     |45.00|
|9:31|B   |200     |45.01|
|9:31|S   |100     |45.05|
|9:32|B   |50      |45.00|

### What do you get from the IEX feed?

In the DEEP data product, you get a number of message types, notably, when there is a trade, and when there is a price level update. This isn't quite the order book, in that there could be many orders combined together to create one price level update, but you do see how many shares have a bid (I will buy your stock) or offer (I want to sell you my stock) out at each dollar amount, which is pretty darn good for free.  Important to note again, IEX is only a few percent of the total market volume, so you're not seeing the whole market, but you get a neat view that you would typically have to pay a lot to see. Here, you only pay with parsing pain.


```python
!xxd -l 112 -s 20000 ~/Downloads/data_feeds_20210122_20210122_IEXTP1_DEEP1.0.pcap
```

    00004e20: 7cb9 0500 ed64 2f58 e805 0000 e805 0000  |....d/X........
    00004e30: 0100 5e57 1504 b859 9ff9 2d53 0800 4500  ..^W...Y..-S..E.
    00004e40: 05da e340 4000 4011 9f90 17e2 9b84 e9d7  ...@@.@.........
    00004e50: 1504 288a 288a 05c6 0402 0100 0480 0100  ..(.(...........
    00004e60: 0000 0000 d948 9605 4200 fc42 0000 0000  .....H..B..B....
    00004e70: 0000 1903 0000 0000 0000 f76d ff74 b88d  ...........m.t..
    00004e80: 5c16 1600 4854 c51f ff74 b88d 5c16 4149  \...HT...t..\.AI


And that's what you get. A lot of hex digits and a [specifications document](https://iextrading.com/docs/IEX%20DEEP%20Specification.pdf). Fortunately, there is a super cool person on the internet who made a [parser](https://pypi.org/project/iex-parser/) that can get all this data into a json format based on the specifications document.


```python
#A lovely command line conversion of the binary to a json file
!pip install iex_parser
!iex_to_json -i iex_file.pcap.gz -o iex_file.json.gz -t 'GME' -s
```


```python
import json
with open('iex_deep_quotes_and_trades.json') as file:
    line = file.readline()
    print(line[:500])
```

    [{"type":"trade_report","event":null,"timestamp":"2021-01-22T13:02:31.215300+00:00","status":null,"symbol":"GME","detail":null,"halt_status":null,"reason":null,"flags":96.0,"size":39.0,"price":45.19,"trade_id":2067095.0,"side":null,"security_event":null},{"type":"trade_report","event":null,"timestamp":"2021-01-22T13:08:14.700160+00:00","status":null,"symbol":"GME","detail":null,"halt_status":null,"reason":null,"flags":96.0,"size":50.0,"price":44.87,"trade_id":2639914.0,"side":null,"security_even


It's still hard to understand without reading the documentation and a little knowledge of how order books work, but it's a heck of a lot more readable, and easy to turn into a dataframe.


```python
#Read in json file, many json files that have a schema without 
#a lot of nesting and variation can be read safely as records
from pandas.io.json import read_json

json_df = read_json('iex_deep_quotes_and_trades.json',orient='records')
display(json_df)
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>event</th>
      <th>timestamp</th>
      <th>...</th>
      <th>trade_id</th>
      <th>side</th>
      <th>security_event</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>trade_report</td>
      <td>NaN</td>
      <td>2021-01-22 13:02:31.215300+00:00</td>
      <td>...</td>
      <td>2.067095e+06</td>
      <td>None</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>trade_report</td>
      <td>NaN</td>
      <td>2021-01-22 13:08:14.700160+00:00</td>
      <td>...</td>
      <td>2.639914e+06</td>
      <td>None</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>trade_report</td>
      <td>NaN</td>
      <td>2021-01-22 13:11:52.294756+00:00</td>
      <td>...</td>
      <td>3.063945e+06</td>
      <td>None</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>trade_report</td>
      <td>NaN</td>
      <td>2021-01-22 13:18:22.383301+00:00</td>
      <td>...</td>
      <td>3.669247e+06</td>
      <td>None</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>trade_report</td>
      <td>NaN</td>
      <td>2021-01-22 13:19:09.002873+00:00</td>
      <td>...</td>
      <td>3.739332e+06</td>
      <td>None</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>490763</th>
      <td>price_level_update</td>
      <td>NaN</td>
      <td>2021-01-28 21:59:06.793899+00:00</td>
      <td>...</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>490764</th>
      <td>price_level_update</td>
      <td>NaN</td>
      <td>2021-01-28 21:59:06.797557+00:00</td>
      <td>...</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>490765</th>
      <td>trade_report</td>
      <td>NaN</td>
      <td>2021-01-28 21:59:08.241677+00:00</td>
      <td>...</td>
      <td>3.027362e+09</td>
      <td>None</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>490766</th>
      <td>price_level_update</td>
      <td>NaN</td>
      <td>2021-01-28 21:59:08.241677+00:00</td>
      <td>...</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>490767</th>
      <td>price_level_update</td>
      <td>NaN</td>
      <td>2021-01-28 22:00:00.018339+00:00</td>
      <td>...</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>490768 rows × 14 columns</p>
</div>



```python
#see what the na situation is
json_df.isna().sum()
```




    type                   0
    event             490768
    timestamp              0
    status            490768
    symbol                 0
                       ...  
    size                   0
    price                  0
    trade_id          356056
    side              134712
    security_event    490768
    Length: 14, dtype: int64




```python
#get rid of columns that are entirely null
json_df = json_df.dropna(axis = 1,how='all')
display(json_df)
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>timestamp</th>
      <th>symbol</th>
      <th>...</th>
      <th>price</th>
      <th>trade_id</th>
      <th>side</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>trade_report</td>
      <td>2021-01-22 13:02:31.215300+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>45.19</td>
      <td>2.067095e+06</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>trade_report</td>
      <td>2021-01-22 13:08:14.700160+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>44.87</td>
      <td>2.639914e+06</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>trade_report</td>
      <td>2021-01-22 13:11:52.294756+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>44.58</td>
      <td>3.063945e+06</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>trade_report</td>
      <td>2021-01-22 13:18:22.383301+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>44.04</td>
      <td>3.669247e+06</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>trade_report</td>
      <td>2021-01-22 13:19:09.002873+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>43.78</td>
      <td>3.739332e+06</td>
      <td>None</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>490763</th>
      <td>price_level_update</td>
      <td>2021-01-28 21:59:06.793899+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>262.00</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>490764</th>
      <td>price_level_update</td>
      <td>2021-01-28 21:59:06.797557+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>262.00</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>490765</th>
      <td>trade_report</td>
      <td>2021-01-28 21:59:08.241677+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>262.00</td>
      <td>3.027362e+09</td>
      <td>None</td>
    </tr>
    <tr>
      <th>490766</th>
      <td>price_level_update</td>
      <td>2021-01-28 21:59:08.241677+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>262.00</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>490767</th>
      <td>price_level_update</td>
      <td>2021-01-28 22:00:00.018339+00:00</td>
      <td>GME</td>
      <td>...</td>
      <td>988.00</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
<p>490768 rows × 8 columns</p>
</div>



```python
#What are we left with?
json_df.isna().sum()
```




    type              0
    timestamp         0
    symbol            0
    flags             0
    size              0
    price             0
    trade_id     356056
    side         134712
    dtype: int64




```python
#What data types are we working with?
json_df.dtypes
```




    type                      object
    timestamp    datetime64[ns, UTC]
    symbol                    object
    flags                      int64
    size                       int64
    price                    float64
    trade_id                 float64
    side                      object
    dtype: object




```python
#The objects really should be strings
json_df = json_df.astype({'type':'string','symbol':'string','side':'string'})
json_df.dtypes
```




    type                      string
    timestamp    datetime64[ns, UTC]
    symbol                    string
    flags                      int64
    size                       int64
    price                    float64
    trade_id                 float64
    side                      string
    dtype: object




```python
#Fill in nulls on the side, since that may cause trouble plotting trades
#create a date column for filtering purposes, and change from UTC to EST
from datetime import timezone
json_df = json_df.fillna({'side':'X'})#replace nulls

json_df['date'] = (json_df
                   .apply({'timestamp':lambda x: x.date}))

json_df['timestamp'] = (json_df['timestamp']
                        .apply(lambda x: x.astimezone(tz='EST')
                               .replace(tzinfo=None)))#change to local time

display(json_df)
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>timestamp</th>
      <th>symbol</th>
      <th>...</th>
      <th>trade_id</th>
      <th>side</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>trade_report</td>
      <td>2021-01-22 08:02:31.215300</td>
      <td>GME</td>
      <td>...</td>
      <td>2.067095e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>1</th>
      <td>trade_report</td>
      <td>2021-01-22 08:08:14.700160</td>
      <td>GME</td>
      <td>...</td>
      <td>2.639914e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>2</th>
      <td>trade_report</td>
      <td>2021-01-22 08:11:52.294756</td>
      <td>GME</td>
      <td>...</td>
      <td>3.063945e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>3</th>
      <td>trade_report</td>
      <td>2021-01-22 08:18:22.383301</td>
      <td>GME</td>
      <td>...</td>
      <td>3.669247e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>4</th>
      <td>trade_report</td>
      <td>2021-01-22 08:19:09.002873</td>
      <td>GME</td>
      <td>...</td>
      <td>3.739332e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>490763</th>
      <td>price_level_update</td>
      <td>2021-01-28 16:59:06.793899</td>
      <td>GME</td>
      <td>...</td>
      <td>NaN</td>
      <td>S</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>490764</th>
      <td>price_level_update</td>
      <td>2021-01-28 16:59:06.797557</td>
      <td>GME</td>
      <td>...</td>
      <td>NaN</td>
      <td>S</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>490765</th>
      <td>trade_report</td>
      <td>2021-01-28 16:59:08.241677</td>
      <td>GME</td>
      <td>...</td>
      <td>3.027362e+09</td>
      <td>X</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>490766</th>
      <td>price_level_update</td>
      <td>2021-01-28 16:59:08.241677</td>
      <td>GME</td>
      <td>...</td>
      <td>NaN</td>
      <td>S</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>490767</th>
      <td>price_level_update</td>
      <td>2021-01-28 17:00:00.018339</td>
      <td>GME</td>
      <td>...</td>
      <td>NaN</td>
      <td>S</td>
      <td>2021-01-28</td>
    </tr>
  </tbody>
</table>
<p>490768 rows × 9 columns</p>
</div>



```python
#Create subset that is just the order book updates
mask = json_df['type']=='price_level_update'
select_cols = ['timestamp','size','price','side']
sort_cols = ['timestamp','price']
order_df = json_df.loc[mask,select_cols].sort_values(sort_cols)
```

I'm not sure if this is the most effective way to accomplish the task, but I felt like making a class to do the data manipulation necessary to carry the information forward from one row to the next. The goal is for each timestep to specify the correct number of shares at each price level at each point in time. This basically accumulates all the different price levels forward in time. If there were only a few price levels, breaking them out into their own columns and doing a cumulative sum would be the most effective. However:


```python
print('There are ',len(order_df.price.unique()),' price levels.')
```

    There are  11848  price levels.



```python
class PriceLevels(dict):
    def ignore_item(self,item):
        return self
    
    def add_or_discard(self,size,price,side,quote_side):
        if (size > 0)&(side==quote_side):
            self.update({price:size})
        elif (size == 0)&(side==quote_side):
            self.pop(price)
        else:
            self.ignore_item
        return self
    
    def get_bbo(self,side):
        if (side == 'B')&(len(self)>0):
            return max(self.keys())
        elif (side == 'S')&(len(self)>0):
            return min(self.keys())
        else:
            return None
    
    def get_vwap(self):
        if len(self)==0:
            return None
        volume = self
        return sum([k*v for k,v in self.items()])/sum([v for v in self.values()])
            
    def update_prices(self,size,price,side,quote_side):
        self.add_or_discard(size,price,side,quote_side)
        return PriceLevels(self.copy())
```


```python
#Some examples of how the object works
a = PriceLevels({45.45:100,50:100,55:50})
print('set price levels: ',a)
a.update_prices(100,46.05,'B','B')
print('update price level same side: ',a)
a.update_prices(100,46.10,'S','B')
print('update price level opposite side: ',a)
a.update_prices(0,50,'B','B')
print('remove price same side: ',a)
print('get the best bid: ',a.get_bbo('B'))
print('get vwap: ',a.get_vwap())
```

    set price levels:  {45.45: 100, 50: 100, 55: 50}
    update price level same side:  {45.45: 100, 50: 100, 55: 50, 46.05: 100}
    update price level opposite side:  {45.45: 100, 50: 100, 55: 50, 46.05: 100}
    remove price same side:  {45.45: 100, 55: 50, 46.05: 100}
    get the best bid:  55
    get vwap:  47.6



```python
#For each timestamp, find current sizes of each available price, the best bid and offer,
#as well as the VWAP (value weighted average price) or the buy orders, sell orders, and all orders
bid = PriceLevels()
ofr = PriceLevels()
quotes = dict()
#Use iterrows()
for row in order_df.iterrows():
    timestamp,size,price,side = row[1]
    quotes[timestamp] = {'bid':bid.update_prices(size,price,side,'B'),
                         'ofr':ofr.update_prices(size,price,side,'S'),
                         'best_bid':bid.get_bbo('B'),
                         'bid_vwap':bid.get_vwap(),
                         'best_ofr':ofr.get_bbo('S'),
                         'ofr_vwap':ofr.get_vwap(),
                         'avg_vwap':PriceLevels({**bid,**ofr}).get_vwap(),
                         'date':timestamp.date()}
```


```python
list(quotes.items())[500:504]
```




    [(Timestamp('2021-01-22 09:31:48.205629'),
      {'bid': {24.98: 100},
       'ofr': {43.61: 300},
       'best_bid': 24.98,
       'bid_vwap': 24.98,
       'best_ofr': 43.61,
       'ofr_vwap': 43.61,
       'avg_vwap': 38.9525,
       'date': datetime.date(2021, 1, 22)}),
     (Timestamp('2021-01-22 09:31:48.320642'),
      {'bid': {24.98: 100},
       'ofr': {43.61: 300, 43.59: 300},
       'best_bid': 24.98,
       'bid_vwap': 24.98,
       'best_ofr': 43.59,
       'ofr_vwap': 43.6,
       'avg_vwap': 40.94,
       'date': datetime.date(2021, 1, 22)}),
     (Timestamp('2021-01-22 09:31:48.320661'),
      {'bid': {24.98: 100},
       'ofr': {43.59: 300},
       'best_bid': 24.98,
       'bid_vwap': 24.98,
       'best_ofr': 43.59,
       'ofr_vwap': 43.59,
       'avg_vwap': 38.93750000000001,
       'date': datetime.date(2021, 1, 22)}),
     (Timestamp('2021-01-22 09:31:48.325218'),
      {'bid': {24.98: 100},
       'ofr': {43.6: 300},
       'best_bid': 24.98,
       'bid_vwap': 24.98,
       'best_ofr': 43.6,
       'ofr_vwap': 43.6,
       'avg_vwap': 38.945,
       'date': datetime.date(2021, 1, 22)})]




```python
#Create a dataframe of current in the order book
quote_df = (pd.DataFrame
            .from_dict(quotes,orient='index')
            .reset_index()
            .rename(columns={'index':'timestamp'})
            .dropna(subset=['best_bid','best_ofr'],how='all'))

display(quote_df)
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>timestamp</th>
      <th>bid</th>
      <th>ofr</th>
      <th>...</th>
      <th>ofr_vwap</th>
      <th>avg_vwap</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-01-22 08:25:48.218283</td>
      <td>{44.45: 1000}</td>
      <td>{}</td>
      <td>...</td>
      <td>NaN</td>
      <td>44.45000</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-01-22 08:44:23.578790</td>
      <td>{44.04: 1000}</td>
      <td>{}</td>
      <td>...</td>
      <td>NaN</td>
      <td>44.04000</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-01-22 08:49:02.526710</td>
      <td>{}</td>
      <td>{43.66: 100}</td>
      <td>...</td>
      <td>43.66000</td>
      <td>43.66000</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2021-01-22 08:49:28.189795</td>
      <td>{43.34: 100}</td>
      <td>{}</td>
      <td>...</td>
      <td>NaN</td>
      <td>43.34000</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2021-01-22 08:51:11.786607</td>
      <td>{43.41: 100}</td>
      <td>{}</td>
      <td>...</td>
      <td>NaN</td>
      <td>43.41000</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>342613</th>
      <td>2021-01-28 16:56:26.486624</td>
      <td>{}</td>
      <td>{988.0: 112}</td>
      <td>...</td>
      <td>988.00000</td>
      <td>988.00000</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>342614</th>
      <td>2021-01-28 16:59:06.409716</td>
      <td>{}</td>
      <td>{988.0: 112, 262.0: 100}</td>
      <td>...</td>
      <td>645.54717</td>
      <td>645.54717</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>342615</th>
      <td>2021-01-28 16:59:06.793899</td>
      <td>{}</td>
      <td>{988.0: 112}</td>
      <td>...</td>
      <td>988.00000</td>
      <td>988.00000</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>342616</th>
      <td>2021-01-28 16:59:06.797557</td>
      <td>{}</td>
      <td>{988.0: 112, 262.0: 100}</td>
      <td>...</td>
      <td>645.54717</td>
      <td>645.54717</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>342617</th>
      <td>2021-01-28 16:59:08.241677</td>
      <td>{}</td>
      <td>{988.0: 112}</td>
      <td>...</td>
      <td>988.00000</td>
      <td>988.00000</td>
      <td>2021-01-28</td>
    </tr>
  </tbody>
</table>
<p>342023 rows × 9 columns</p>
</div>



```python
#Get just the trades from the original dataframe
trade_df = json_df.loc[json_df['type']=='trade_report',:]
trade_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>timestamp</th>
      <th>symbol</th>
      <th>...</th>
      <th>trade_id</th>
      <th>side</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>trade_report</td>
      <td>2021-01-22 08:02:31.215300</td>
      <td>GME</td>
      <td>...</td>
      <td>2.067095e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>1</th>
      <td>trade_report</td>
      <td>2021-01-22 08:08:14.700160</td>
      <td>GME</td>
      <td>...</td>
      <td>2.639914e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>2</th>
      <td>trade_report</td>
      <td>2021-01-22 08:11:52.294756</td>
      <td>GME</td>
      <td>...</td>
      <td>3.063945e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>3</th>
      <td>trade_report</td>
      <td>2021-01-22 08:18:22.383301</td>
      <td>GME</td>
      <td>...</td>
      <td>3.669247e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>4</th>
      <td>trade_report</td>
      <td>2021-01-22 08:19:09.002873</td>
      <td>GME</td>
      <td>...</td>
      <td>3.739332e+06</td>
      <td>X</td>
      <td>2021-01-22</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>490754</th>
      <td>trade_report</td>
      <td>2021-01-28 16:54:05.085758</td>
      <td>GME</td>
      <td>...</td>
      <td>3.026635e+09</td>
      <td>X</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>490755</th>
      <td>trade_report</td>
      <td>2021-01-28 16:54:05.592947</td>
      <td>GME</td>
      <td>...</td>
      <td>3.026636e+09</td>
      <td>X</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>490759</th>
      <td>trade_report</td>
      <td>2021-01-28 16:56:26.486624</td>
      <td>GME</td>
      <td>...</td>
      <td>3.026962e+09</td>
      <td>X</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>490762</th>
      <td>trade_report</td>
      <td>2021-01-28 16:59:06.793899</td>
      <td>GME</td>
      <td>...</td>
      <td>3.027353e+09</td>
      <td>X</td>
      <td>2021-01-28</td>
    </tr>
    <tr>
      <th>490765</th>
      <td>trade_report</td>
      <td>2021-01-28 16:59:08.241677</td>
      <td>GME</td>
      <td>...</td>
      <td>3.027362e+09</td>
      <td>X</td>
      <td>2021-01-28</td>
    </tr>
  </tbody>
</table>
<p>134712 rows × 9 columns</p>
</div>




```python
#Combine the two dataframes so that each trade report is associated 
#with a set of 'prevailing quotes.'
plot_df = (pd.concat([quote_df,trade_df],ignore_index=False)
             .fillna({'type':'quote','symbol':'GME'})
             .sort_values(['timestamp','type'])
             .drop('side',axis=1)
             .ffill()
             .dropna(subset=['best_bid','best_ofr'])
             .astype({'date':'string'}))

plot_df = plot_df.loc[plot_df['type']=='trade_report',:]

plot_df['midpt'] = (plot_df['best_bid']+plot_df['best_ofr'])/2

display(plot_df.loc[:,['timestamp','price','best_bid','best_ofr','midpt']])
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>timestamp</th>
      <th>price</th>
      <th>best_bid</th>
      <th>best_ofr</th>
      <th>midpt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>27</th>
      <td>2021-01-22 08:53:12.048708</td>
      <td>43.85</td>
      <td>43.55</td>
      <td>43.66</td>
      <td>43.605</td>
    </tr>
    <tr>
      <th>28</th>
      <td>2021-01-22 08:54:24.385311</td>
      <td>44.09</td>
      <td>43.55</td>
      <td>43.66</td>
      <td>43.605</td>
    </tr>
    <tr>
      <th>31</th>
      <td>2021-01-22 08:57:36.794024</td>
      <td>44.22</td>
      <td>43.55</td>
      <td>44.30</td>
      <td>43.925</td>
    </tr>
    <tr>
      <th>32</th>
      <td>2021-01-22 08:57:39.852032</td>
      <td>44.20</td>
      <td>43.55</td>
      <td>44.30</td>
      <td>43.925</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2021-01-22 08:57:41.975903</td>
      <td>44.28</td>
      <td>43.55</td>
      <td>44.30</td>
      <td>43.925</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>490754</th>
      <td>2021-01-28 16:54:05.085758</td>
      <td>257.58</td>
      <td>257.58</td>
      <td>988.00</td>
      <td>622.790</td>
    </tr>
    <tr>
      <th>490755</th>
      <td>2021-01-28 16:54:05.592947</td>
      <td>257.58</td>
      <td>257.58</td>
      <td>988.00</td>
      <td>622.790</td>
    </tr>
    <tr>
      <th>490759</th>
      <td>2021-01-28 16:56:26.486624</td>
      <td>264.00</td>
      <td>257.58</td>
      <td>988.00</td>
      <td>622.790</td>
    </tr>
    <tr>
      <th>490762</th>
      <td>2021-01-28 16:59:06.793899</td>
      <td>262.00</td>
      <td>257.58</td>
      <td>988.00</td>
      <td>622.790</td>
    </tr>
    <tr>
      <th>490765</th>
      <td>2021-01-28 16:59:08.241677</td>
      <td>262.00</td>
      <td>257.58</td>
      <td>988.00</td>
      <td>622.790</td>
    </tr>
  </tbody>
</table>
<p>134697 rows × 5 columns</p>
</div>


I'd like to make a special note that concatenating two dataframes and then sorting them and using the forward fill option to fill in a prevailing value is a nice trick. The logically obvious way to do this is you only wanted the trades is to do some kind of time range join, which would take waaaaaay longer. Concatenating is fast, sorting is fast, forward filling is fast, and throwing away a ton of rows is fast. Joins are slow.
