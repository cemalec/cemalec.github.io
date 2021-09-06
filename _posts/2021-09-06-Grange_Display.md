---
layout: default
title: The Stardew Valley Grange Display - Basic Web Scraping
permalink: /Stardew_Grange1/
---
## The Stardew Valley Grange Display - Basic Web Scraping
Lately, I've been pretty obsessed with Stardew Valley. It's a lovely little farming game, with a lot more to do besides grow crops. For the purpose of the next couple posts, I'd like to focus on an event that happens every fall, the Stardew Valley Fair. Specifically, you have the option of creating a Grange Display, and winning gets you 1000 star points. The star points aren't the most useful thing in the world, but like many things in the game, that's hardly the point. I'd just like to figure out how to get a perfect score with the least amount of work.

Step one that I'll describe in this post is some super basic web-scraping of the [stardew valley wiki](https://stardewvalleywiki.com/). My goal is to grab the tables on the page about the fair to find the point values for the different items you can put in the grange display. I'll use the requests package to get the web page and parse the html using Beautiful Soup. My ultimate goal is to create a pandas data frame.

```python
#import libraries
import requests
from bs4 import BeautifulSoup
from IPython.display import display, HTML
import pandas as pd
```


```python
#get html from stardew wiki
r = requests.get('https://stardewvalleywiki.com/Stardew_Valley_Fair')
html_text = r.text
soup = BeautifulSoup(html_text, 'html.parser')
```

Beautiful Soup turns the html page into an object that can be accessed in a much more pythonic way. For example, here I can find all the 'wikitable's on the page. Dealing with the actual text inside each row and column requires a slightly different procedure depending on if the text is a single string, multiple strings, or an image. Cells that have both an image and text are read just as text, since the image is usually a 'gold' or 'star point' image. It would be possible to reproduce these cells, but it would be more work, and doesn't really help me.

```python
#find all the tags that are part of the wikitable css class
tables = soup.find_all(class_="wikitable")
```


```python
#a little helper function for when there are several strings withtin the same tag
def join_strings(tag):
    return ''.join([string for string in tag.stripped_strings])
```


```python
#function to parse the table into a pandas dataframe
#optionally searches for the heading to add a 'category' column to the dataframe
def parse_table(table, get_heading = True):
    if get_heading:
        table_title = [join_strings(table.find_previous(class_='mw-collapsible').th)]
    else:
        table_title = []
    row = table.tr #the first row has several 'th' tags, this is pretty general for tables on this site
    headers = [elem.string.strip() for elem in row.find_all("th")]
    #print(headers)
    records = []
    row = row.next_sibling
    while row:
        if row.string:
            row = row.next_sibling #prevents reading in rows that are just a newline without columns
            continue
        else:
            column = row.td #find first column
        record = []
        while column:
            if column.string:
                cell_text = column.string.strip() #get string within the td tag
            else:
                cell_text =  join_strings(column) #get string when there are multiple strings in a tag''.join([string for string in column.stripped_strings])
            try:
                image = column.img
                #Don't want the star token and gold images
                if (image.attrs['alt'] != 'Token.png')&(image.attrs['alt'] != 'Gold.png'):
                    if image.attrs['src'][:5]!='https':
                        image.attrs['src'] = 'https://stardewvalleywiki.com'+image.attrs['src'] #get full location
                    image = str(image)#Needs to be a string to be resolved in DataFrame
                else:
                    image = None
            except:
                image = None
            
            if cell_text != '':
                record.append(cell_text) #adds column to row record if it isn't empty
            elif image:
                record.append(image) #adds column to row record if it's an image (and just an image)
            column = column.next_sibling #goes to the next column
        records.append(tuple(record+table_title)) #adds row to the list of records
        #print(records)
        row = row.next_sibling #goes to the next row
    if get_heading:
        columns = headers+['category'] #adds category column if we are finding the table heading
    else:
        columns = headers #just uses the column names found in th tags
    table_df = pd.DataFrame.from_records(data=records,columns = columns).dropna() #creates a pandas dataframe from a list of records
    return table_df
```


```python
def display_table(table_df):
    return display(HTML(table_df.to_html(escape=False,index=False))) #helper function to make it display like I want
```


```python
dfs = []
for table in tables[3:11]:
    dfs.append(parse_table(table))#Get all the tables related to the grange display
grange_df = pd.concat(dfs,axis=0,ignore_index=True)#Concatenate the tables into one flat table
grange_df = grange_df.melt(id_vars = ["Item","category","Price"],
                             value_vars = ["Base","Silver","Gold","Iridium"],
                             var_name = "Quality",
                             value_name = "Points")#Melt the table so that the points are all in one column
```


```python
display_table(grange_df)
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Item</th>
      <th>category</th>
      <th>Price</th>
      <th>Quality</th>
      <th>Points</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Honey (Wild)*</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jelly (Ancient Fruit)</td>
      <td>Artisan Goods</td>
      <td>1,150g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Jelly (Apple)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Apricot)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jelly (Banana)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Jelly (Blackberry)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jelly (Blueberry)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jelly (Cactus Fruit)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Cherry)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Coconut)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Cranberries)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Crystal Fruit)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
  </tbody>
</table>

And here's an example of one of the tables that has images in it. For my goal of finding an easy perfect score grange display, I won't really need it, but it's a nice thing to know how to do.

```python
display_table(parse_table(tables[12]))
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Image</th>
      <th>Name</th>
      <th>Description</th>
      <th>Price</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><img alt="Dried Sunflowers.png" decoding="async" height="96" src="https://stardewvalleywiki.com/mediawiki/images/a/af/Dried_Sunflowers.png" width="48"/></td>
      <td>Dried Sunflowers</td>
      <td>Can be placed inside your house.</td>
      <td>100</td>
      <td>Vegetables</td>
    </tr>
    <tr>
      <td><img alt="Fedora.png" decoding="async" height="54" src="https://stardewvalleywiki.com/mediawiki/images/5/5e/Fedora.png" width="54"/></td>
      <td>Fedora</td>
      <td>A city-slicker's standard.</td>
      <td>500</td>
      <td>Vegetables</td>
    </tr>
    <tr>
      <td><img alt="Rarecrow 1.png" decoding="async" height="96" src="https://stardewvalleywiki.com/mediawiki/images/6/62/Rarecrow_1.png" width="48"/></td>
      <td>Rarecrow</td>
      <td>Collect them all! (1 of 8)</td>
      <td>800</td>
      <td>Vegetables</td>
    </tr>
    <tr>
      <td><img alt="Stardrop.png" decoding="async" height="48" src="https://stardewvalleywiki.com/mediawiki/images/a/a5/Stardrop.png" width="48"/></td>
      <td>Stardrop</td>
      <td>A mysterious fruit that empowers those who eat it. The flavor is like a dream... a powerful personal experience, yet difficult to describe to others.</td>
      <td>2,000</td>
      <td>Vegetables</td>
    </tr>
    <tr>
      <td><img alt="Light Green Rug.png" decoding="async" height="64" src="https://stardewvalleywiki.com/mediawiki/images/e/ec/Light_Green_Rug.png" width="94"/></td>
      <td>Light Green Rug</td>
      <td>Can be placed inside your house.</td>
      <td>500</td>
      <td>Vegetables</td>
    </tr>
  </tbody>
</table>

And that's it! In the next post, I'll analyze my pandas dataframes.
