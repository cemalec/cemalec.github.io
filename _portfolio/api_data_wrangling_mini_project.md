
This exercise will require you to pull some data from the Qunadl API. Qaundl is currently the most widely used aggregator of financial market data.

As a first step, you will need to register a free account on the http://www.quandl.com website.

After you register, you will be provided with a unique API key, that you should store:


```python
# Store the API key as a string - according to PEP8, constants are always named in all upper case
API_KEY = 
```

Qaundl has a large number of data sources, but, unfortunately, most of them require a Premium subscription. Still, there are also a good number of free datasets.

For this mini project, we will focus on equities data from the Frankfurt Stock Exhange (FSE), which is available for free. We'll try and analyze the stock prices of a company called Carl Zeiss Meditec, which manufactures tools for eye examinations, as well as medical lasers for laser eye surgery: https://www.zeiss.com/meditec/int/home.html. The company is listed under the stock ticker AFX_X.

You can find the detailed Quandl API instructions here: https://docs.quandl.com/docs/time-series

While there is a dedicated Python package for connecting to the Quandl API, we would prefer that you use the *requests* package, which can be easily downloaded using *pip* or *conda*. You can find the documentation for the package here: http://docs.python-requests.org/en/master/ 

Finally, apart from the *requests* package, you are encouraged to not use any third party Python packages, such as *pandas*, and instead focus on what's available in the Python Standard Library (the *collections* module might come in handy: https://pymotw.com/3/collections/ ).
Also, since you won't have access to DataFrames, you are encouraged to us Python's native data structures - preferably dictionaries, though some questions can also be answered using lists.
You can read more on these data structures here: https://docs.python.org/3/tutorial/datastructures.html

Keep in mind that the JSON responses you will be getting from the API map almost one-to-one to Python's dictionaries. Unfortunately, they can be very nested, so make sure you read up on indexing dictionaries in the documentation provided above.


```python
# First, import the relevant modules
import requests
from collections import defaultdict
```


```python
# Now, call the Quandl API and pull out a small sample of the data (only one day) to get a glimpse
# into the JSON structure that will be returned
url = 'https://www.quandl.com/api/v3/datasets/FSE/AFX_X.json'

payload = {'start_date':'2017-04-25',
              'end_date':'2017-04-25',
              'order':'asc',
              'collapse':'none',
              'transform':'none',
              'api_key':API_KEY}

r = requests.get(url,params = payload)
r.raise_for_status()
print('The frequency of this data is '+r.json()['dataset']['frequency'])
```

    The frequency of this data is daily


We can see that the data is granulated at the level of daily, which is important for future questions.

Collect data from the Franfurt Stock Exchange, for the ticker AFX_X, for the whole year 2017 (keep in mind that the date format is YYYY-MM-DD).

This is accomplished with a simple call to the quandl API for the requested dates. If the request is denied an exception will be raised.


```python
url = 'https://www.quandl.com/api/v3/datasets/FSE/AFX_X.json'

payload = {'start_date':'2017-01-01',
              'end_date':'2017-12-31',
              'order':'asc',
              'collapse':'none',
              'transform':'none',
              'api_key':API_KEY}

r = requests.get(url,params = payload)
r.raise_for_status()
```

After looking at the JSON fields, I used the columns names as keys to create a dictionary. The file returns a list of all fields for each date, but I wanted a dictionary which contained a list of all values for a given key. For example, the key of 'Date' contained a list of all the dates in the JSON file.


```python
#Parse the JSON object and slice out the list of lists containing the data.
data_lists = r.json()['dataset']['data']

#Create a list of keys
keys = ['Date',
              'Open',
              'High',
              'Low',
              'Close',
              'Change',
              'Traded Volume',
              'Turnover',
              'Last Price of the Day',
              'Daily Traded Units',
              'Daily Turnover']

#Initialize a dictionary
data_dict = defaultdict(list)

#Create a list for each key, and extract the i'th element from each list
for i, key in enumerate(keys):
    for j, datum in enumerate(data_lists):
        data_dict[key].append(datum[i])
```

Calculate what the highest and lowest opening prices were for the stock in this period.

There are a number of 'none' entries in the data that throw off computations, I got rid of these by only accepting values that returned a type of 'float'.


```python
clean_open = [x for x in data_dict['Open'] if isinstance(x,float)]
print('The lowest opening price is '+str(min(clean_open))
      +', and the highest opening price is '+str(max(clean_open)))
```

    The lowest opening price is 34.0, and the highest opening price is 53.11


What was the largest change between any two days (based on Closing Price)?

I thought it was easier to make another API call to get the data transformed using the 'diff' operator. This calculates the difference between sequential entries. 

I will need the dictionary I made earlier later, so I gave this one a different name. If I was going to do this operation even once more, it would probably be time to write a function JSON2dict.


```python
#A new request asks for just the 'close' column and differentiates the data.
payload = {'start_date':'2017-01-01',
              'end_date':'2017-12-31',
              'column_index':'4',
              'order':'asc',
              'collapse':'none',
              'transform':'diff',
              'api_key':API_KEY}

r2 = requests.get(url,params = payload)
r2.raise_for_status()

data_lists2 = r2.json()['dataset']['data']
keys2 = ['Date','Close']

data_dict2 = defaultdict(list)
for i, key in enumerate(keys2):
    for j, datum in enumerate(data_lists2):
        data_dict2[key].append(datum[i])

#clean the data of non-float values
clean_change = [x for x in data_dict2['Close'] if isinstance(x,float)]

#Find the max element and its index without numpy
(biggest_change,biggest_change_i) = max((v,i) for i,v in enumerate(clean_change))

#Find the dates that result in this change
date1 = data_dict2['Date'][biggest_change_i-1]
date2 = data_dict2['Date'][biggest_change_i]
print('The biggest change is '+str(biggest_change)+' between '+str(date1)
      +' and '+str(date2))
```

    The biggest change is 1.72 between 2017-05-10 and 2017-05-11


What was the average daily trading volume during this year?

The data is cleaned in a similar way and the mean of the output is calculated.


```python
clean_volume = [x for x in data_dict['Traded Volume'] if isinstance(x,float)]
print('The mean traded volume is '+str(round(sum(clean_volume)/len(clean_volume),2)))
```

    The mean traded volume is 89124.34


What was the median trading volume during this year. (Note: you may need to implement your own function for calculating the median.)

I defined a median function, it works.


```python
#median function
def median(numlist):
    numlist.sort()
    N = len(numlist)
    if N%2 == 0:
        hN = int(N/2)
        return (numlist[hN-1]+numlist[hN])/2
    else:
        hN = int((N-1)/2)
        return numlist[hN]

#The median of the list of data is calculated
print('The median traded volume is '+str(round(median(clean_volume),2)))
```

    The median traded volume is 76286.0



```python

```
