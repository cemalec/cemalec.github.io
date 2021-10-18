---
layout: default
title: Market Manipulation
permalink: /manip_1/
---

### What is market manipulation?

This is relatively subjective, but the crux of many types of manipulation is 'are you engaging with the market in good faith?' In other words, are you placing orders onto the books because you would like to buy a stock at the price you stated (or at least hope someone will sell to you at that price), or are you sending false signals to manipulate at what prices people are willing to buy/sell.

|time|side|quantity|price|firm   |
|----|----|--------|-----|-------|
|9:30|B   |100     |45.00|A      |
|9:31|B   |200     |45.01|A      |
|9:31|S   |100     |45.05|B      |
|9:32|B   |50      |45.00|A      |
|9:33|S   |100     |45.04|Manip  |
|9:33|S   |200     |45.04|Manip  |
|9:33|S   |500     |45.03|Manip  |
|9:33|S   |100     |45.05|Manip  |
|9:33|S   |400     |45.03|Manip  |
|9:33|S   |100     |45.03|Manip  |
|9:34|S   |600     |45.02|IllBite|
|9:34|B   |600     |45.02|Manip  |

### What is an order book?

Every time a broker wants to make a trade, they put an an order (I want to buy 200 shares of stock A for $100 each). Until the order is canceled, or a seller is found that is willing to meet the buyer at their price, the order 'rests on the book.' Most heavily traded stocks do not have orders that rest long, unless the number of shares is exceptionally large or the price point is completely unreasonable.

|time|side|quantity|price|
|----|----|--------|-----|
|9:30|B   |100     |45.00|
|9:31|B   |200     |45.01|
|9:31|S   |100     |45.05|
|9:32|B   |50      |45.00|

### What is Reg NMS
Reg NMS is an SEC rule that created the 'National Market System.' Stock exchanges became publicly traded companies and firms routing orders to the markets gained the obligation of 'best execution.' This generally means that if NYSE's order book has a stock selling at 45.05 and NASDAQ's order book has the same stock selling at 45.04, your buy order with Charles Schwab account needs to take the shares at NASDAQ at the better price.

The regulation is intended to keep brokerages from giving customers a bad deal, but it also created a complicated set of rules that can be gamed.

### How can anyone find manipulation?

Short answer: Regulators get to see much more detailed information than everyone else. It can be pretty difficult even with the extra information.



```python
import pandas as pd
manip_df = pd.read_csv('example_manip.csv',
                       sep=',',
                       header=1)
display(manip_df)
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
      <th>Time</th>
      <th>Firm</th>
      <th>Side</th>
      <th>Price</th>
      <th>Quantity</th>
      <th>Order ID</th>
      <th>Action</th>
      <th>NBO</th>
      <th>NBB</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10/13/2021 9:31:00.0 AM</td>
      <td>Firm S</td>
      <td>S</td>
      <td>50.05</td>
      <td>1000</td>
      <td>1</td>
      <td>Order</td>
      <td>50.05</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10/13/2021 9:31:00.0 AM</td>
      <td>Firm B</td>
      <td>B</td>
      <td>49.95</td>
      <td>1000</td>
      <td>2</td>
      <td>Order</td>
      <td>50.05</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10/13/2021 9:31:00.2 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>200</td>
      <td>3</td>
      <td>Order</td>
      <td>50.04</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10/13/2021 9:31:01.5 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.03</td>
      <td>100</td>
      <td>4</td>
      <td>Order</td>
      <td>50.03</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10/13/2021 9:31:03.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.03</td>
      <td>100</td>
      <td>4</td>
      <td>Cancel</td>
      <td>50.04</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10/13/2021 9:31:03.2 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.01</td>
      <td>150</td>
      <td>5</td>
      <td>Order</td>
      <td>50.01</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10/13/2021 9:31:03.3 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.00</td>
      <td>300</td>
      <td>6</td>
      <td>Order</td>
      <td>50.00</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>7</th>
      <td>10/13/2021 9:31:04.5 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.03</td>
      <td>500</td>
      <td>7</td>
      <td>Order</td>
      <td>50.00</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>8</th>
      <td>10/13/2021 9:31:04.6 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>200</td>
      <td>8</td>
      <td>Order</td>
      <td>50.00</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10/13/2021 9:31:05.7 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.96</td>
      <td>200</td>
      <td>9</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10/13/2021 9:31:05.7 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>100</td>
      <td>8</td>
      <td>Cancel</td>
      <td>49.96</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>11</th>
      <td>10/13/2021 9:31:05.5 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.97</td>
      <td>600</td>
      <td>10</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>12</th>
      <td>10/13/2021 9:31:05.6 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.96</td>
      <td>300</td>
      <td>11</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>13</th>
      <td>10/13/2021 9:31:06.8 AM</td>
      <td>Firm S</td>
      <td>S</td>
      <td>49.96</td>
      <td>1000</td>
      <td>12</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>14</th>
      <td>10/13/2021 9:31:06.9 AM</td>
      <td>Firm M</td>
      <td>B</td>
      <td>49.96</td>
      <td>1000</td>
      <td>13</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.96</td>
    </tr>
    <tr>
      <th>15</th>
      <td>10/13/2021 9:31:06.9 AM</td>
      <td>Firm M</td>
      <td>X</td>
      <td>49.96</td>
      <td>1000</td>
      <td>12_13</td>
      <td>Trade</td>
      <td>49.96</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.96</td>
      <td>300</td>
      <td>11</td>
      <td>Cancel</td>
      <td>49.97</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>17</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.97</td>
      <td>600</td>
      <td>10</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>18</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.96</td>
      <td>200</td>
      <td>9</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>19</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>100</td>
      <td>8</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>20</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.03</td>
      <td>500</td>
      <td>7</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>21</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.00</td>
      <td>300</td>
      <td>6</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>22</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.01</td>
      <td>150</td>
      <td>5</td>
      <td>Cancel</td>
      <td>50.04</td>
      <td>49.95</td>
    </tr>
    <tr>
      <th>23</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>200</td>
      <td>3</td>
      <td>Cancel</td>
      <td>50.05</td>
      <td>49.95</td>
    </tr>
  </tbody>
</table>
</div>



```python
def string_to_timestamp(string):
    timestamp_components = string.split(' ')
    timestamp_string = 'T'.join(timestamp_components[:2])
    return pd.to_datetime(timestamp_string)
manip_df['timestamp'] = manip_df['Time'].apply(string_to_timestamp)
manip_df
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
      <th>Time</th>
      <th>Firm</th>
      <th>Side</th>
      <th>Price</th>
      <th>Quantity</th>
      <th>Order ID</th>
      <th>Action</th>
      <th>NBO</th>
      <th>NBB</th>
      <th>timestamp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10/13/2021 9:31:00.0 AM</td>
      <td>Firm S</td>
      <td>S</td>
      <td>50.05</td>
      <td>1000</td>
      <td>1</td>
      <td>Order</td>
      <td>50.05</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:00.000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10/13/2021 9:31:00.0 AM</td>
      <td>Firm B</td>
      <td>B</td>
      <td>49.95</td>
      <td>1000</td>
      <td>2</td>
      <td>Order</td>
      <td>50.05</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:00.000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10/13/2021 9:31:00.2 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>200</td>
      <td>3</td>
      <td>Order</td>
      <td>50.04</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:00.200</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10/13/2021 9:31:01.5 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.03</td>
      <td>100</td>
      <td>4</td>
      <td>Order</td>
      <td>50.03</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:01.500</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10/13/2021 9:31:03.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.03</td>
      <td>100</td>
      <td>4</td>
      <td>Cancel</td>
      <td>50.04</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:03.000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10/13/2021 9:31:03.2 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.01</td>
      <td>150</td>
      <td>5</td>
      <td>Order</td>
      <td>50.01</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:03.200</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10/13/2021 9:31:03.3 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.00</td>
      <td>300</td>
      <td>6</td>
      <td>Order</td>
      <td>50.00</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:03.300</td>
    </tr>
    <tr>
      <th>7</th>
      <td>10/13/2021 9:31:04.5 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.03</td>
      <td>500</td>
      <td>7</td>
      <td>Order</td>
      <td>50.00</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:04.500</td>
    </tr>
    <tr>
      <th>8</th>
      <td>10/13/2021 9:31:04.6 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>200</td>
      <td>8</td>
      <td>Order</td>
      <td>50.00</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:04.600</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10/13/2021 9:31:05.7 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.96</td>
      <td>200</td>
      <td>9</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:05.700</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10/13/2021 9:31:05.7 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>100</td>
      <td>8</td>
      <td>Cancel</td>
      <td>49.96</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:05.700</td>
    </tr>
    <tr>
      <th>11</th>
      <td>10/13/2021 9:31:05.5 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.97</td>
      <td>600</td>
      <td>10</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:05.500</td>
    </tr>
    <tr>
      <th>12</th>
      <td>10/13/2021 9:31:05.6 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.96</td>
      <td>300</td>
      <td>11</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:05.600</td>
    </tr>
    <tr>
      <th>13</th>
      <td>10/13/2021 9:31:06.8 AM</td>
      <td>Firm S</td>
      <td>S</td>
      <td>49.96</td>
      <td>1000</td>
      <td>12</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:06.800</td>
    </tr>
    <tr>
      <th>14</th>
      <td>10/13/2021 9:31:06.9 AM</td>
      <td>Firm M</td>
      <td>B</td>
      <td>49.96</td>
      <td>1000</td>
      <td>13</td>
      <td>Order</td>
      <td>49.96</td>
      <td>49.96</td>
      <td>2021-10-13 09:31:06.900</td>
    </tr>
    <tr>
      <th>15</th>
      <td>10/13/2021 9:31:06.9 AM</td>
      <td>Firm M</td>
      <td>X</td>
      <td>49.96</td>
      <td>1000</td>
      <td>12_13</td>
      <td>Trade</td>
      <td>49.96</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:06.900</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.96</td>
      <td>300</td>
      <td>11</td>
      <td>Cancel</td>
      <td>49.97</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:07.000</td>
    </tr>
    <tr>
      <th>17</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.97</td>
      <td>600</td>
      <td>10</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:07.000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>49.96</td>
      <td>200</td>
      <td>9</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:07.000</td>
    </tr>
    <tr>
      <th>19</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>100</td>
      <td>8</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:07.000</td>
    </tr>
    <tr>
      <th>20</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.03</td>
      <td>500</td>
      <td>7</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:07.000</td>
    </tr>
    <tr>
      <th>21</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.00</td>
      <td>300</td>
      <td>6</td>
      <td>Cancel</td>
      <td>50.00</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:07.000</td>
    </tr>
    <tr>
      <th>22</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.01</td>
      <td>150</td>
      <td>5</td>
      <td>Cancel</td>
      <td>50.04</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:07.000</td>
    </tr>
    <tr>
      <th>23</th>
      <td>10/13/2021 9:31:07.0 AM</td>
      <td>Firm M</td>
      <td>S</td>
      <td>50.04</td>
      <td>200</td>
      <td>3</td>
      <td>Cancel</td>
      <td>50.05</td>
      <td>49.95</td>
      <td>2021-10-13 09:31:07.000</td>
    </tr>
  </tbody>
</table>
</div>




```python
import plotly.express as px
import plotly.graph_objects as go
fig = px.scatter(manip_df,
                 x='timestamp',
                 y='Price',
                 symbol='Action',
                 color='Firm',
                 size='Quantity',
                 color_discrete_sequence=['goldenrod','aqua','lightgreen'],
                 opacity=.5)

fig.add_trace(go.Scatter(x=manip_df['timestamp'],
                         y=manip_df['NBO'],
                         mode='lines',
                         line_color='magenta',
                         line_shape='hv',
                         name='Best Bid'))

fig.add_trace(go.Scatter(x=manip_df['timestamp'],
                         y=manip_df['NBB'],
                         mode='lines',
                         line_color='cyan',
                         line_shape='hv',
                         name='Best Offer'))
fig.update_layout(template='plotly_dark');
```


```python
fig.write_html("manip_example.html")
fig.show()
```

<iframe scrolling='no' seamless='seamless' style='border:none' src='../images/manip_example.html' width='100%' height='400px'></iframe>

In this example a Firm S is selling the stock and a Firm B is buying. The 'spread' or difference between best bid and best ask is 10 cents. The midpoint price, or what is often referred to as the 'price' if buyers and sellers met in the middle is \\$50.

A Firm M would like to buy this stock, but they would like to buy closer to the bid than the ask, or even closer to the bid than the midpoint. What Firm M does is enter a bunch of sell orders to make it look like a lot of people suddenly want to sell this stock. Keep in mind that Firm S and Firm B cannot see who is putting in the orders, just that there are suddenly new orders at several different price points. 

The orders are placed so that the best offer becomes progressively lower. Firm S, noting the sell pressure, panics that they may get stuck with an even lower price if they wait around for their original order of \\$50.05, and so they put an offer of \$49.96, or the current best offer. Now the spread is just a penny, but Firm M won't go any lower, because their sell order would trade with Firm B, and remember they don't actually want to sell the stock.

When Firm M sees a sell order at a favorable price appear, they immediately put in a buy order and trade with Firm S, and then cancel all their other orders. They may even place some cancels beforehand to make the whole thing look more believable. Ultimately, Firm M will pull out quickly since they do not actually want to trade with anyone buying stock at their artificially low prices. The more believable they make their 'non-bona fide' quotes, the more real risk they take on. At some point, if their quotes could not be distinguished from normal activity, they would probably be trading against buyers looking for good prices and end up acting like a real trader in spite of themselves.

Though in this example Firm M got a 9 cent improvement on their buy order of 1000 shares, they netted \\$90, which doesn't sound like much, especially considering they put nearly \\$50,000 on the line to do it. However, for one thing, they can turn around and reverse the whole scheme, netting \\$90 the other way and keep going back and forth. For another thing, Firm M accomplished this in 7s. Though this is fabricated data, 7s is actually a pretty long time in the stock market, so it is a realistic time scale for a manipulation like this. The wage for this scheme is then \\$46,285.71 per hour.

The less realistic aspect of the data is the large spread of 10 cents, though it's not unreasonable for a lightly traded stock. A lightly traded stock would also have a smaller number of firms trading, allowing Firm M to dominate the market more easily. Firm M would probably have to jump around different symbols since traders generally notice when their fills aren't going so hot and may stop treating Firm M's orders as believable, even though they wouldn't be able to prove that all the orders were from the same party. 

In short, if Firm M really was trying to repeat this back and forth constantly all day, someone would notice, and may report them to FinRA. However, you can see that someone can make very good money by cheating. I hear traders who don't cheat also make very good money, so I'd reccommend the honest strategy.

As a final plug for honesty, just imagine if absolutely everyone was trading purely in a 'zero-sum' game mentality where the point is to take as much money for yourself as possible from everyone else's pile. If there is 'a point' to the capital markets it is that companies that have a higher chance of succeeding, or that would benefit the most from a capital infusion, get money to grow their business. If no one is thinking about the value of *owning* a stock, collecting dividends, voting with shareholders, collecting premiums from those who would like to borrow the stock in short and options trades; then any rationality that may exist in the market is gone. Maybe I'm holding on to too much belief in markets as price discovery engines, but without fair markets where traders can basically trust the intentions of the other participants, our 401k's are just funding a money game that produces little to no value.
