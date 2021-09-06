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
      <td>Duck Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Duck Feather</td>
      <td>Animal Products</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Egg</td>
      <td>Animal Products</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Brown Egg</td>
      <td>Animal Products</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Goat Milk</td>
      <td>Animal Products</td>
      <td>225g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Golden Egg</td>
      <td>Animal Products</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Large Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Large Brown Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Large Goat Milk</td>
      <td>Animal Products</td>
      <td>345g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Large Milk</td>
      <td>Animal Products</td>
      <td>190g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Milk</td>
      <td>Animal Products</td>
      <td>125g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Ostrich Egg</td>
      <td>Animal Products</td>
      <td>600g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rabbit's Foot</td>
      <td>Animal Products</td>
      <td>565g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Void Egg</td>
      <td>Animal Products</td>
      <td>65g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Wool</td>
      <td>Animal Products</td>
      <td>340g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Aged Roe (Albacore)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Anchovy)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Blobfish)</td>
      <td>Artisan Goods</td>
      <td>560g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Aged Roe (Blue Discus)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Bream)</td>
      <td>Artisan Goods</td>
      <td>104g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Bullhead)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Carp)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Catfish)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Chub)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Cockle)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Crab)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Crayfish)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Dorado)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Eel)</td>
      <td>Artisan Goods</td>
      <td>144g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Flounder)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Ghostfish)</td>
      <td>Artisan Goods</td>
      <td>104g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Halibut)</td>
      <td>Artisan Goods</td>
      <td>140g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Herring)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Ice Pip)</td>
      <td>Artisan Goods</td>
      <td>560g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Aged Roe (Largemouth Bass)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Lava Eel)</td>
      <td>Artisan Goods</td>
      <td>760g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Aged Roe (Lingcod)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Lionfish)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Lobster)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Midnight Carp)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Mussel)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Octopus)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Oyster)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Perch)</td>
      <td>Artisan Goods</td>
      <td>114g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Periwinkle)</td>
      <td>Artisan Goods</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Aged Roe (Pike)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Pufferfish)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Rainbow Trout)</td>
      <td>Artisan Goods</td>
      <td>124g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Red Mullet)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Red Snapper)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Salmon)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Sandfish)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Sardine)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Scorpion Carp)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Sea Cucumber)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Sea Urchin)</td>
      <td>Artisan Goods</td>
      <td>220g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Shad)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Shrimp)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Slimejack)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Smallmouth Bass)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Snail)</td>
      <td>Artisan Goods</td>
      <td>124g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Spook Fish)</td>
      <td>Artisan Goods</td>
      <td>280g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Stingray)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Stonefish)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Aged Roe (Sturgeon)</td>
      <td>Artisan Goods</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Aged Roe (Sunfish)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Super Cucumber)</td>
      <td>Artisan Goods</td>
      <td>310g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Aged Roe (Tiger Trout)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Tilapia)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Tuna)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Void Salmon)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aged Roe (Walleye)</td>
      <td>Artisan Goods</td>
      <td>164g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aged Roe (Woodskip)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Beer</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Caviar</td>
      <td>Artisan Goods</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Cheese</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Cloth</td>
      <td>Artisan Goods</td>
      <td>470g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Duck Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>375g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Goat Cheese</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Green Tea</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Honey (Blue Jazz)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Honey (Fairy Rose)</td>
      <td>Artisan Goods</td>
      <td>680g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Honey (Poppy)</td>
      <td>Artisan Goods</td>
      <td>380g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Honey (Summer Spangle)</td>
      <td>Artisan Goods</td>
      <td>280g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Honey (Sunflower)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Honey (Tulip)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
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
    <tr>
      <td>Jelly (Grape)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>130g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jelly (Mango)</td>
      <td>Artisan Goods</td>
      <td>310g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Jelly (Melon)</td>
      <td>Artisan Goods</td>
      <td>550g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Jelly (Orange)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Peach)</td>
      <td>Artisan Goods</td>
      <td>330g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Jelly (Pineapple)</td>
      <td>Artisan Goods</td>
      <td>650g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Jelly (Pomegranate)</td>
      <td>Artisan Goods</td>
      <td>330g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Jelly (Rhubarb)</td>
      <td>Artisan Goods</td>
      <td>490g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Jelly (Salmonberry)</td>
      <td>Artisan Goods</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Jelly (Spice Berry)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Starfruit)</td>
      <td>Artisan Goods</td>
      <td>1,550g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Jelly (Strawberry)</td>
      <td>Artisan Goods</td>
      <td>290g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jelly (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jelly (Wild Plum)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Juice (Amaranth)</td>
      <td>Artisan Goods</td>
      <td>337g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Juice (Artichoke)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Juice (Beet)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Juice (Bok Choy)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Juice (Cauliflower)</td>
      <td>Artisan Goods</td>
      <td>393g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Juice (Corn)</td>
      <td>Artisan Goods</td>
      <td>112g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Juice (Eggplant)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Juice (Fiddlehead Fern)</td>
      <td>Artisan Goods</td>
      <td>202g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Juice (Garlic)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Juice (Green Bean)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Juice (Hops)</td>
      <td>Artisan Goods</td>
      <td>56g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Juice (Kale)</td>
      <td>Artisan Goods</td>
      <td>247g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Juice (Parsnip)</td>
      <td>Artisan Goods</td>
      <td>78g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Juice (Potato)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Juice (Pumpkin)</td>
      <td>Artisan Goods</td>
      <td>720g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Juice (Radish)</td>
      <td>Artisan Goods</td>
      <td>202g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Juice (Red Cabbage)</td>
      <td>Artisan Goods</td>
      <td>585g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Juice (Taro Root)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Juice (Tomato)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Juice (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Juice (Unmilled Rice)</td>
      <td>Artisan Goods</td>
      <td>67g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Juice (Wheat)</td>
      <td>Artisan Goods</td>
      <td>56g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Juice (Yam)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>190g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pale Ale</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pickles (Amaranth)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pickles (Artichoke)</td>
      <td>Artisan Goods</td>
      <td>370g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pickles (Beet)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pickles (Bok Choy)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pickles (Cauliflower)</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pickles (Corn)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Eggplant)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Fiddlehead Fern)</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pickles (Garlic)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Ginger)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Green Bean)</td>
      <td>Artisan Goods</td>
      <td>130g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Hops)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Kale)</td>
      <td>Artisan Goods</td>
      <td>270g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pickles (Parsnip)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Potato)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pickles (Pumpkin)</td>
      <td>Artisan Goods</td>
      <td>690g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pickles (Radish)</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pickles (Red Cabbage)</td>
      <td>Artisan Goods</td>
      <td>570g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pickles (Taro Root)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pickles (Tea Leaves)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Tomato)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Unmilled Rice)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Wheat)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pickles (Yam)</td>
      <td>Artisan Goods</td>
      <td>370g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Truffle Oil</td>
      <td>Artisan Goods</td>
      <td>1,065g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Void Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>275g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Ancient Fruit)</td>
      <td>Artisan Goods</td>
      <td>1,650g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Apple)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Apricot)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Wine (Banana)</td>
      <td>Artisan Goods</td>
      <td>450g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Blackberry)</td>
      <td>Artisan Goods</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Wine (Blueberry)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Wine (Cactus Fruit)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Cherry)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Coconut)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Cranberries)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Crystal Fruit)</td>
      <td>Artisan Goods</td>
      <td>450g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Grape)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Wine (Mango)</td>
      <td>Artisan Goods</td>
      <td>390g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Melon)</td>
      <td>Artisan Goods</td>
      <td>750g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Orange)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Peach)</td>
      <td>Artisan Goods</td>
      <td>420g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Pineapple)</td>
      <td>Artisan Goods</td>
      <td>900g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Pomegranate)</td>
      <td>Artisan Goods</td>
      <td>420g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Rhubarb)</td>
      <td>Artisan Goods</td>
      <td>660g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Salmonberry)</td>
      <td>Artisan Goods</td>
      <td>15g</td>
      <td>Base</td>
      <td>1</td>
    </tr>
    <tr>
      <td>Wine (Spice Berry)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Starfruit)</td>
      <td>Artisan Goods</td>
      <td>2,250g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Strawberry)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Wild Plum)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Algae Soup</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Artichoke Dip</td>
      <td>Cooking</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Autumn's Bounty</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Baked Fish</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Banana Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Bean Hotpot</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Blackberry Cobbler</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Blueberry Tart</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Bread</td>
      <td>Cooking</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Bruschetta</td>
      <td>Cooking</td>
      <td>210g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Carp Surprise</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Cheese Cauliflower</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Chocolate Cake</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Chowder</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Coleslaw</td>
      <td>Cooking</td>
      <td>345g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Complete Breakfast</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Cookie</td>
      <td>Cooking</td>
      <td>140g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Crab Cakes</td>
      <td>Cooking</td>
      <td>275g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Cranberry Candy</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Cranberry Sauce</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Crispy Bass</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Dish O' The Sea</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Eggplant Parmesan</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Escargot</td>
      <td>Cooking</td>
      <td>125g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Farmer's Lunch</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Fiddlehead Risotto</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Fish Stew</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Fish Taco</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Fried Calamari</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Fried Eel</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Fried Egg</td>
      <td>Cooking</td>
      <td>35g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Fried Mushroom</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Fruit Salad</td>
      <td>Cooking</td>
      <td>450g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Ginger Ale</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Glazed Yams</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Hashbrowns</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Ice Cream</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Life Elixir</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Lobster Bisque</td>
      <td>Cooking</td>
      <td>205g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Lucky Lunch</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Maki Roll</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Mango Sticky Rice</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Maple Bar</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Miner's Treat</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Oil of Garlic</td>
      <td>Cooking</td>
      <td>1,000g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Omelet</td>
      <td>Cooking</td>
      <td>125g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pale Broth</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pancakes</td>
      <td>Cooking</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Parsnip Soup</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pepper Poppers</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pink Cake</td>
      <td>Cooking</td>
      <td>480g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pizza</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Plum Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Poi</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Poppyseed Muffin</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pumpkin Pie</td>
      <td>Cooking</td>
      <td>385g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pumpkin Soup</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Radish Salad</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Red Plate</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rhubarb Pie</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rice Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Roasted Hazelnuts</td>
      <td>Cooking</td>
      <td>270g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Roots Platter</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Salad</td>
      <td>Cooking</td>
      <td>110g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Salmon Dinner</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Sashimi</td>
      <td>Cooking</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Seafoam Pudding</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Shrimp Cocktail</td>
      <td>Cooking</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Spaghetti</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Spicy Eel</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Squid Ink Ravioli</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Stir Fry</td>
      <td>Cooking</td>
      <td>335g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Strange Bun</td>
      <td>Cooking</td>
      <td>225g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Stuffing</td>
      <td>Cooking</td>
      <td>165g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Super Meal</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Survival Burger</td>
      <td>Cooking</td>
      <td>180g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Tom Kha Soup</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Tortilla</td>
      <td>Cooking</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Triple Shot Espresso</td>
      <td>Cooking</td>
      <td>450g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Tropical Curry</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Trout Soup</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Vegetable Medley</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Albacore</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Anchovy</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Angler</td>
      <td>Fish</td>
      <td>900g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blobfish</td>
      <td>Fish</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blue Discus</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Bream</td>
      <td>Fish</td>
      <td>45g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Bullhead</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Carp</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Catfish</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Chub</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Cockle</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Crab</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Crayfish</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Crimsonfish</td>
      <td>Fish</td>
      <td>1,500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Dorado</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Eel</td>
      <td>Fish</td>
      <td>85g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Flounder</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Ghostfish</td>
      <td>Fish</td>
      <td>45g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Glacierfish</td>
      <td>Fish</td>
      <td>1,000g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Halibut</td>
      <td>Fish</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Herring</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Ice Pip</td>
      <td>Fish</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Largemouth Bass</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Lava Eel</td>
      <td>Fish</td>
      <td>700g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Legend</td>
      <td>Fish</td>
      <td>5,000g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Lingcod</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Lionfish</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Lobster</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Midnight Carp</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Midnight Squid</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Mussel</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Mutant Carp</td>
      <td>Fish</td>
      <td>1,000g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Octopus</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Oyster</td>
      <td>Fish</td>
      <td>40g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Perch</td>
      <td>Fish</td>
      <td>55g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Periwinkle</td>
      <td>Fish</td>
      <td>20g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Pike</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pufferfish</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Rainbow Trout</td>
      <td>Fish</td>
      <td>65g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Red Mullet</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Red Snapper</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Salmon</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Sandfish</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Sardine</td>
      <td>Fish</td>
      <td>40g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Scorpion Carp</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Sea Cucumber</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Shad</td>
      <td>Fish</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Shrimp</td>
      <td>Fish</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Slimejack</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Smallmouth Bass</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Snail</td>
      <td>Fish</td>
      <td>65g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Spook Fish</td>
      <td>Fish</td>
      <td>220g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Squid</td>
      <td>Fish</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Stingray</td>
      <td>Fish</td>
      <td>180g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Stonefish</td>
      <td>Fish</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Sturgeon</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Sunfish</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Super Cucumber</td>
      <td>Fish</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Tiger Trout</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Tilapia</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Tuna</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Void Salmon</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Walleye</td>
      <td>Fish</td>
      <td>105g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Woodskip</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Blue Jazz</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Cave Carrot</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>25g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Chanterelle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Common Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>40g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Crocus</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Daffodil</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>30g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Dandelion</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>40g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Fairy Rose</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>290g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Ginger</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Hazelnut</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Holly</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Leek</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Magma Cap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>400g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Maple Syrup</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Morel</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Oak Resin</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pine Tar</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Poppy</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>140g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Purple Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Red Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Sap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>2g</td>
      <td>Base</td>
      <td>1</td>
    </tr>
    <tr>
      <td>Snow Yam</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Spring Onion</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>8g</td>
      <td>Base</td>
      <td>1</td>
    </tr>
    <tr>
      <td>Summer Spangle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Sunflower</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Sweet Pea</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>55g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Tulip</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>30g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Wild Horseradish</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Winter Root</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>70g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Ancient Fruit</td>
      <td>Fruits</td>
      <td>550g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Apple</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Apricot</td>
      <td>Fruits</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Banana</td>
      <td>Fruits</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Blackberry</td>
      <td>Fruits</td>
      <td>20g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Blueberry</td>
      <td>Fruits</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Cactus Fruit</td>
      <td>Fruits</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Cherry</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Coconut</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Cranberries</td>
      <td>Fruits</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Crystal Fruit</td>
      <td>Fruits</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Grape</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Hot Pepper</td>
      <td>Fruits</td>
      <td>40g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Mango</td>
      <td>Fruits</td>
      <td>130g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Melon</td>
      <td>Fruits</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Orange</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Peach</td>
      <td>Fruits</td>
      <td>140g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Pineapple</td>
      <td>Fruits</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pomegranate</td>
      <td>Fruits</td>
      <td>140g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Rhubarb</td>
      <td>Fruits</td>
      <td>220g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Salmonberry</td>
      <td>Fruits</td>
      <td>5g</td>
      <td>Base</td>
      <td>1</td>
    </tr>
    <tr>
      <td>Spice Berry</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Starfruit</td>
      <td>Fruits</td>
      <td>750g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Strawberry</td>
      <td>Fruits</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Wild Plum</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Aerinite</td>
      <td>Minerals</td>
      <td>125g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Alamite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Amethyst</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Aquamarine</td>
      <td>Minerals</td>
      <td>180g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Baryte</td>
      <td>Minerals</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Basalt</td>
      <td>Minerals</td>
      <td>175g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Bixite</td>
      <td>Minerals</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Calcite</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Celestine</td>
      <td>Minerals</td>
      <td>125g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Diamond</td>
      <td>Minerals</td>
      <td>750g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Dolomite</td>
      <td>Minerals</td>
      <td>300g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Earth Crystal</td>
      <td>Minerals</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Emerald</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Esperite</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Fairy Stone</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Fire Opal</td>
      <td>Minerals</td>
      <td>350g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Fire Quartz</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Fluorapatite</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Frozen Tear</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Geminite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Ghost Crystal</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Granite</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Helvite</td>
      <td>Minerals</td>
      <td>450g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Hematite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jade</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Jagoite</td>
      <td>Minerals</td>
      <td>115g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jamborite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Jasper</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Kyanite</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Lemon Stone</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Limestone</td>
      <td>Minerals</td>
      <td>15g</td>
      <td>Base</td>
      <td>1</td>
    </tr>
    <tr>
      <td>Lunarite</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Malachite</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Marble</td>
      <td>Minerals</td>
      <td>110g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Mudstone</td>
      <td>Minerals</td>
      <td>25g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Nekoite</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Neptunite</td>
      <td>Minerals</td>
      <td>400g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Obsidian</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Ocean Stone</td>
      <td>Minerals</td>
      <td>220g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Opal</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Orpiment</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Petrified Slime</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Prismatic Shard</td>
      <td>Minerals</td>
      <td>2,000g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pyrite</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Quartz</td>
      <td>Minerals</td>
      <td>25g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Ruby</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Sandstone</td>
      <td>Minerals</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Slate</td>
      <td>Minerals</td>
      <td>85g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Soapstone</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Star Shards</td>
      <td>Minerals</td>
      <td>500g</td>
      <td>Base</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Thunder Egg</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Tigerseye</td>
      <td>Minerals</td>
      <td>275g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Topaz</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Amaranth</td>
      <td>Vegetables</td>
      <td>150g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Artichoke</td>
      <td>Vegetables</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Beet</td>
      <td>Vegetables</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Bok Choy</td>
      <td>Vegetables</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Cauliflower</td>
      <td>Vegetables</td>
      <td>175g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Corn</td>
      <td>Vegetables</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Eggplant</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Fiddlehead Fern</td>
      <td>Vegetables</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Garlic</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Green Bean</td>
      <td>Vegetables</td>
      <td>40g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Hops</td>
      <td>Vegetables</td>
      <td>25g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Kale</td>
      <td>Vegetables</td>
      <td>110g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Parsnip</td>
      <td>Vegetables</td>
      <td>35g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Potato</td>
      <td>Vegetables</td>
      <td>80g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Pumpkin</td>
      <td>Vegetables</td>
      <td>320g</td>
      <td>Base</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Radish</td>
      <td>Vegetables</td>
      <td>90g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Red Cabbage</td>
      <td>Vegetables</td>
      <td>260g</td>
      <td>Base</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Taro Root</td>
      <td>Vegetables</td>
      <td>100g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Tea Leaves</td>
      <td>Vegetables</td>
      <td>50g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Tomato</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Unmilled Rice</td>
      <td>Vegetables</td>
      <td>30g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Wheat</td>
      <td>Vegetables</td>
      <td>25g</td>
      <td>Base</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Yam</td>
      <td>Vegetables</td>
      <td>160g</td>
      <td>Base</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Duck Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Duck Feather</td>
      <td>Animal Products</td>
      <td>250g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Egg</td>
      <td>Animal Products</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Brown Egg</td>
      <td>Animal Products</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Goat Milk</td>
      <td>Animal Products</td>
      <td>225g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Golden Egg</td>
      <td>Animal Products</td>
      <td>500g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Large Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Large Brown Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Large Goat Milk</td>
      <td>Animal Products</td>
      <td>345g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Large Milk</td>
      <td>Animal Products</td>
      <td>190g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Milk</td>
      <td>Animal Products</td>
      <td>125g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Ostrich Egg</td>
      <td>Animal Products</td>
      <td>600g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rabbit's Foot</td>
      <td>Animal Products</td>
      <td>565g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Void Egg</td>
      <td>Animal Products</td>
      <td>65g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Wool</td>
      <td>Animal Products</td>
      <td>340g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Aged Roe (Albacore)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Anchovy)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Blobfish)</td>
      <td>Artisan Goods</td>
      <td>560g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Blue Discus)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Bream)</td>
      <td>Artisan Goods</td>
      <td>104g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Bullhead)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Carp)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Catfish)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Chub)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Cockle)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Crab)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Crayfish)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Dorado)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Eel)</td>
      <td>Artisan Goods</td>
      <td>144g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Flounder)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Ghostfish)</td>
      <td>Artisan Goods</td>
      <td>104g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Halibut)</td>
      <td>Artisan Goods</td>
      <td>140g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Herring)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Ice Pip)</td>
      <td>Artisan Goods</td>
      <td>560g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Largemouth Bass)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lava Eel)</td>
      <td>Artisan Goods</td>
      <td>760g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lingcod)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lionfish)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lobster)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Midnight Carp)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Mussel)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Octopus)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Oyster)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Perch)</td>
      <td>Artisan Goods</td>
      <td>114g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Periwinkle)</td>
      <td>Artisan Goods</td>
      <td>80g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Pike)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Pufferfish)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Rainbow Trout)</td>
      <td>Artisan Goods</td>
      <td>124g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Red Mullet)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Red Snapper)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Salmon)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sandfish)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sardine)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Scorpion Carp)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sea Cucumber)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sea Urchin)</td>
      <td>Artisan Goods</td>
      <td>220g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Shad)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Shrimp)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Slimejack)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Smallmouth Bass)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Snail)</td>
      <td>Artisan Goods</td>
      <td>124g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Spook Fish)</td>
      <td>Artisan Goods</td>
      <td>280g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Stingray)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Stonefish)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sturgeon)</td>
      <td>Artisan Goods</td>
      <td>500g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sunfish)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Super Cucumber)</td>
      <td>Artisan Goods</td>
      <td>310g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tiger Trout)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tilapia)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tuna)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Void Salmon)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Walleye)</td>
      <td>Artisan Goods</td>
      <td>164g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Woodskip)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Beer</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Caviar</td>
      <td>Artisan Goods</td>
      <td>500g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cheese</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Cloth</td>
      <td>Artisan Goods</td>
      <td>470g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Duck Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>375g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Goat Cheese</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Green Tea</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Blue Jazz)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Fairy Rose)</td>
      <td>Artisan Goods</td>
      <td>680g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Poppy)</td>
      <td>Artisan Goods</td>
      <td>380g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Summer Spangle)</td>
      <td>Artisan Goods</td>
      <td>280g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Sunflower)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Tulip)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Wild)*</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Ancient Fruit)</td>
      <td>Artisan Goods</td>
      <td>1,150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Apple)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Apricot)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Banana)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Blackberry)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Blueberry)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cactus Fruit)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cherry)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Coconut)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cranberries)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Crystal Fruit)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Grape)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>130g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Mango)</td>
      <td>Artisan Goods</td>
      <td>310g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Melon)</td>
      <td>Artisan Goods</td>
      <td>550g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Orange)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Peach)</td>
      <td>Artisan Goods</td>
      <td>330g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Pineapple)</td>
      <td>Artisan Goods</td>
      <td>650g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Pomegranate)</td>
      <td>Artisan Goods</td>
      <td>330g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Rhubarb)</td>
      <td>Artisan Goods</td>
      <td>490g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Salmonberry)</td>
      <td>Artisan Goods</td>
      <td>60g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Spice Berry)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Starfruit)</td>
      <td>Artisan Goods</td>
      <td>1,550g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Strawberry)</td>
      <td>Artisan Goods</td>
      <td>290g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Wild Plum)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Amaranth)</td>
      <td>Artisan Goods</td>
      <td>337g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Artichoke)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Beet)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Bok Choy)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Cauliflower)</td>
      <td>Artisan Goods</td>
      <td>393g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Corn)</td>
      <td>Artisan Goods</td>
      <td>112g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Eggplant)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Fiddlehead Fern)</td>
      <td>Artisan Goods</td>
      <td>202g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Garlic)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Green Bean)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Hops)</td>
      <td>Artisan Goods</td>
      <td>56g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Kale)</td>
      <td>Artisan Goods</td>
      <td>247g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Parsnip)</td>
      <td>Artisan Goods</td>
      <td>78g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Potato)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Pumpkin)</td>
      <td>Artisan Goods</td>
      <td>720g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Radish)</td>
      <td>Artisan Goods</td>
      <td>202g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Red Cabbage)</td>
      <td>Artisan Goods</td>
      <td>585g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Taro Root)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Tomato)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Unmilled Rice)</td>
      <td>Artisan Goods</td>
      <td>67g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Wheat)</td>
      <td>Artisan Goods</td>
      <td>56g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Yam)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>190g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pale Ale</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pickles (Amaranth)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Artichoke)</td>
      <td>Artisan Goods</td>
      <td>370g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Beet)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Bok Choy)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Cauliflower)</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Corn)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Eggplant)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Fiddlehead Fern)</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Garlic)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Ginger)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Green Bean)</td>
      <td>Artisan Goods</td>
      <td>130g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Hops)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Kale)</td>
      <td>Artisan Goods</td>
      <td>270g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Parsnip)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Potato)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Pumpkin)</td>
      <td>Artisan Goods</td>
      <td>690g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Radish)</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Red Cabbage)</td>
      <td>Artisan Goods</td>
      <td>570g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Taro Root)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Tea Leaves)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Tomato)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Unmilled Rice)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Wheat)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Yam)</td>
      <td>Artisan Goods</td>
      <td>370g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Truffle Oil</td>
      <td>Artisan Goods</td>
      <td>1,065g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Void Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>275g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Wine (Ancient Fruit)</td>
      <td>Artisan Goods</td>
      <td>1,650g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Apple)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Apricot)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Banana)</td>
      <td>Artisan Goods</td>
      <td>450g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Blackberry)</td>
      <td>Artisan Goods</td>
      <td>60g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Wine (Blueberry)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Cactus Fruit)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Cherry)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Coconut)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Cranberries)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Crystal Fruit)</td>
      <td>Artisan Goods</td>
      <td>450g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Grape)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Mango)</td>
      <td>Artisan Goods</td>
      <td>390g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Melon)</td>
      <td>Artisan Goods</td>
      <td>750g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Orange)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Peach)</td>
      <td>Artisan Goods</td>
      <td>420g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Pineapple)</td>
      <td>Artisan Goods</td>
      <td>900g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Pomegranate)</td>
      <td>Artisan Goods</td>
      <td>420g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Rhubarb)</td>
      <td>Artisan Goods</td>
      <td>660g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Salmonberry)</td>
      <td>Artisan Goods</td>
      <td>15g</td>
      <td>Silver</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Wine (Spice Berry)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Starfruit)</td>
      <td>Artisan Goods</td>
      <td>2,250g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Strawberry)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Wild Plum)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Algae Soup</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Artichoke Dip</td>
      <td>Cooking</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Autumn's Bounty</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Baked Fish</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Banana Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bean Hotpot</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Blackberry Cobbler</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Blueberry Tart</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bread</td>
      <td>Cooking</td>
      <td>60g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bruschetta</td>
      <td>Cooking</td>
      <td>210g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Carp Surprise</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cheese Cauliflower</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Chocolate Cake</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Chowder</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Coleslaw</td>
      <td>Cooking</td>
      <td>345g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Complete Breakfast</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cookie</td>
      <td>Cooking</td>
      <td>140g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crab Cakes</td>
      <td>Cooking</td>
      <td>275g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cranberry Candy</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cranberry Sauce</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crispy Bass</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Dish O' The Sea</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Eggplant Parmesan</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Escargot</td>
      <td>Cooking</td>
      <td>125g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Farmer's Lunch</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fiddlehead Risotto</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fish Stew</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fish Taco</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fried Calamari</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fried Eel</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fried Egg</td>
      <td>Cooking</td>
      <td>35g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fried Mushroom</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fruit Salad</td>
      <td>Cooking</td>
      <td>450g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ginger Ale</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Glazed Yams</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Hashbrowns</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ice Cream</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Life Elixir</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lobster Bisque</td>
      <td>Cooking</td>
      <td>205g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lucky Lunch</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Maki Roll</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Mango Sticky Rice</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Maple Bar</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Miner's Treat</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Oil of Garlic</td>
      <td>Cooking</td>
      <td>1,000g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Omelet</td>
      <td>Cooking</td>
      <td>125g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pale Broth</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pancakes</td>
      <td>Cooking</td>
      <td>80g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Parsnip Soup</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pepper Poppers</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pink Cake</td>
      <td>Cooking</td>
      <td>480g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pizza</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Plum Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Poi</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Poppyseed Muffin</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pumpkin Pie</td>
      <td>Cooking</td>
      <td>385g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pumpkin Soup</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Radish Salad</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Red Plate</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Rhubarb Pie</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Rice Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Roasted Hazelnuts</td>
      <td>Cooking</td>
      <td>270g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Roots Platter</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Salad</td>
      <td>Cooking</td>
      <td>110g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Salmon Dinner</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Sashimi</td>
      <td>Cooking</td>
      <td>75g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Seafoam Pudding</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Shrimp Cocktail</td>
      <td>Cooking</td>
      <td>160g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Spaghetti</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Spicy Eel</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Squid Ink Ravioli</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Stir Fry</td>
      <td>Cooking</td>
      <td>335g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Strange Bun</td>
      <td>Cooking</td>
      <td>225g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Stuffing</td>
      <td>Cooking</td>
      <td>165g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Super Meal</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Survival Burger</td>
      <td>Cooking</td>
      <td>180g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tom Kha Soup</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tortilla</td>
      <td>Cooking</td>
      <td>50g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Triple Shot Espresso</td>
      <td>Cooking</td>
      <td>450g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tropical Curry</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Trout Soup</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Vegetable Medley</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Albacore</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Anchovy</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Angler</td>
      <td>Fish</td>
      <td>900g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blobfish</td>
      <td>Fish</td>
      <td>500g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blue Discus</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Bream</td>
      <td>Fish</td>
      <td>45g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Bullhead</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Carp</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Catfish</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Chub</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Cockle</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Crab</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crayfish</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crimsonfish</td>
      <td>Fish</td>
      <td>1,500g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Dorado</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Eel</td>
      <td>Fish</td>
      <td>85g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Flounder</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Ghostfish</td>
      <td>Fish</td>
      <td>45g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Glacierfish</td>
      <td>Fish</td>
      <td>1,000g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Halibut</td>
      <td>Fish</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Herring</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Ice Pip</td>
      <td>Fish</td>
      <td>500g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Largemouth Bass</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Lava Eel</td>
      <td>Fish</td>
      <td>700g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Legend</td>
      <td>Fish</td>
      <td>5,000g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Lingcod</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Lionfish</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Lobster</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Midnight Carp</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Midnight Squid</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Mussel</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Mutant Carp</td>
      <td>Fish</td>
      <td>1,000g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Octopus</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Oyster</td>
      <td>Fish</td>
      <td>40g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Perch</td>
      <td>Fish</td>
      <td>55g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Periwinkle</td>
      <td>Fish</td>
      <td>20g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pike</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pufferfish</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Rainbow Trout</td>
      <td>Fish</td>
      <td>65g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Red Mullet</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Red Snapper</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Salmon</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Sandfish</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Sardine</td>
      <td>Fish</td>
      <td>40g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Scorpion Carp</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Sea Cucumber</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Shad</td>
      <td>Fish</td>
      <td>60g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Shrimp</td>
      <td>Fish</td>
      <td>60g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Slimejack</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Smallmouth Bass</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Snail</td>
      <td>Fish</td>
      <td>65g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Spook Fish</td>
      <td>Fish</td>
      <td>220g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Squid</td>
      <td>Fish</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Stingray</td>
      <td>Fish</td>
      <td>180g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Stonefish</td>
      <td>Fish</td>
      <td>300g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Sturgeon</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Sunfish</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Super Cucumber</td>
      <td>Fish</td>
      <td>250g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Tiger Trout</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Tilapia</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Tuna</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Void Salmon</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Walleye</td>
      <td>Fish</td>
      <td>105g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Woodskip</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Blue Jazz</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Cave Carrot</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>25g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Chanterelle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>160g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Common Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>40g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Crocus</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>60g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Daffodil</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>30g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Dandelion</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>40g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Fairy Rose</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>290g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Ginger</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>60g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Hazelnut</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>90g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Holly</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Leek</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Magma Cap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>400g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Maple Syrup</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Morel</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Oak Resin</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pine Tar</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Poppy</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>140g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Purple Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>250g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Red Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Sap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>2g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Snow Yam</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Spring Onion</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>8g</td>
      <td>Silver</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Summer Spangle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>90g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Sunflower</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Sweet Pea</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>55g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Tulip</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>30g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Wild Horseradish</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Winter Root</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>70g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Ancient Fruit</td>
      <td>Fruits</td>
      <td>550g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Apple</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Apricot</td>
      <td>Fruits</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Banana</td>
      <td>Fruits</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Blackberry</td>
      <td>Fruits</td>
      <td>20g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Blueberry</td>
      <td>Fruits</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Cactus Fruit</td>
      <td>Fruits</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Cherry</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Coconut</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Cranberries</td>
      <td>Fruits</td>
      <td>75g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Crystal Fruit</td>
      <td>Fruits</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Grape</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Hot Pepper</td>
      <td>Fruits</td>
      <td>40g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Mango</td>
      <td>Fruits</td>
      <td>130g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Melon</td>
      <td>Fruits</td>
      <td>250g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Orange</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Peach</td>
      <td>Fruits</td>
      <td>140g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pineapple</td>
      <td>Fruits</td>
      <td>300g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pomegranate</td>
      <td>Fruits</td>
      <td>140g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Rhubarb</td>
      <td>Fruits</td>
      <td>220g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Salmonberry</td>
      <td>Fruits</td>
      <td>5g</td>
      <td>Silver</td>
      <td>2</td>
    </tr>
    <tr>
      <td>Spice Berry</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Starfruit</td>
      <td>Fruits</td>
      <td>750g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Strawberry</td>
      <td>Fruits</td>
      <td>120g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wild Plum</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Aerinite</td>
      <td>Minerals</td>
      <td>125g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Alamite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Amethyst</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aquamarine</td>
      <td>Minerals</td>
      <td>180g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Baryte</td>
      <td>Minerals</td>
      <td>50g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Basalt</td>
      <td>Minerals</td>
      <td>175g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bixite</td>
      <td>Minerals</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Calcite</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Celestine</td>
      <td>Minerals</td>
      <td>125g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Diamond</td>
      <td>Minerals</td>
      <td>750g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Dolomite</td>
      <td>Minerals</td>
      <td>300g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Earth Crystal</td>
      <td>Minerals</td>
      <td>50g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Emerald</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Esperite</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fairy Stone</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fire Opal</td>
      <td>Minerals</td>
      <td>350g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fire Quartz</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fluorapatite</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Frozen Tear</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Geminite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ghost Crystal</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Granite</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Helvite</td>
      <td>Minerals</td>
      <td>450g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Hematite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jade</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jagoite</td>
      <td>Minerals</td>
      <td>115g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jamborite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jasper</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Kyanite</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lemon Stone</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Limestone</td>
      <td>Minerals</td>
      <td>15g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lunarite</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Malachite</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Marble</td>
      <td>Minerals</td>
      <td>110g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Mudstone</td>
      <td>Minerals</td>
      <td>25g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Nekoite</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Neptunite</td>
      <td>Minerals</td>
      <td>400g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Obsidian</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ocean Stone</td>
      <td>Minerals</td>
      <td>220g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Opal</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Orpiment</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Petrified Slime</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Prismatic Shard</td>
      <td>Minerals</td>
      <td>2,000g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pyrite</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Quartz</td>
      <td>Minerals</td>
      <td>25g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ruby</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Sandstone</td>
      <td>Minerals</td>
      <td>60g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Slate</td>
      <td>Minerals</td>
      <td>85g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Soapstone</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Star Shards</td>
      <td>Minerals</td>
      <td>500g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Thunder Egg</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tigerseye</td>
      <td>Minerals</td>
      <td>275g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Topaz</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Amaranth</td>
      <td>Vegetables</td>
      <td>150g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Artichoke</td>
      <td>Vegetables</td>
      <td>160g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Beet</td>
      <td>Vegetables</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Bok Choy</td>
      <td>Vegetables</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Cauliflower</td>
      <td>Vegetables</td>
      <td>175g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Corn</td>
      <td>Vegetables</td>
      <td>50g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Eggplant</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Fiddlehead Fern</td>
      <td>Vegetables</td>
      <td>90g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Garlic</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Green Bean</td>
      <td>Vegetables</td>
      <td>40g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Hops</td>
      <td>Vegetables</td>
      <td>25g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Kale</td>
      <td>Vegetables</td>
      <td>110g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Parsnip</td>
      <td>Vegetables</td>
      <td>35g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Potato</td>
      <td>Vegetables</td>
      <td>80g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Pumpkin</td>
      <td>Vegetables</td>
      <td>320g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Radish</td>
      <td>Vegetables</td>
      <td>90g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Red Cabbage</td>
      <td>Vegetables</td>
      <td>260g</td>
      <td>Silver</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Taro Root</td>
      <td>Vegetables</td>
      <td>100g</td>
      <td>Silver</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Tea Leaves</td>
      <td>Vegetables</td>
      <td>50g</td>
      <td>Silver</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tomato</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Unmilled Rice</td>
      <td>Vegetables</td>
      <td>30g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Wheat</td>
      <td>Vegetables</td>
      <td>25g</td>
      <td>Silver</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Yam</td>
      <td>Vegetables</td>
      <td>160g</td>
      <td>Silver</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Duck Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Duck Feather</td>
      <td>Animal Products</td>
      <td>250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Egg</td>
      <td>Animal Products</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Brown Egg</td>
      <td>Animal Products</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Goat Milk</td>
      <td>Animal Products</td>
      <td>225g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Golden Egg</td>
      <td>Animal Products</td>
      <td>500g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Large Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Large Brown Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Large Goat Milk</td>
      <td>Animal Products</td>
      <td>345g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Large Milk</td>
      <td>Animal Products</td>
      <td>190g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Milk</td>
      <td>Animal Products</td>
      <td>125g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Ostrich Egg</td>
      <td>Animal Products</td>
      <td>600g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rabbit's Foot</td>
      <td>Animal Products</td>
      <td>565g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Void Egg</td>
      <td>Animal Products</td>
      <td>65g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wool</td>
      <td>Animal Products</td>
      <td>340g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Aged Roe (Albacore)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Anchovy)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Blobfish)</td>
      <td>Artisan Goods</td>
      <td>560g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Blue Discus)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Bream)</td>
      <td>Artisan Goods</td>
      <td>104g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Bullhead)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Carp)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Catfish)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Chub)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Cockle)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Crab)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Crayfish)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Dorado)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Eel)</td>
      <td>Artisan Goods</td>
      <td>144g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Flounder)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Ghostfish)</td>
      <td>Artisan Goods</td>
      <td>104g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Halibut)</td>
      <td>Artisan Goods</td>
      <td>140g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Herring)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Ice Pip)</td>
      <td>Artisan Goods</td>
      <td>560g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Largemouth Bass)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lava Eel)</td>
      <td>Artisan Goods</td>
      <td>760g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lingcod)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lionfish)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lobster)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Midnight Carp)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Mussel)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Octopus)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Oyster)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Perch)</td>
      <td>Artisan Goods</td>
      <td>114g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Periwinkle)</td>
      <td>Artisan Goods</td>
      <td>80g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Pike)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Pufferfish)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Rainbow Trout)</td>
      <td>Artisan Goods</td>
      <td>124g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Red Mullet)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Red Snapper)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Salmon)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sandfish)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sardine)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Scorpion Carp)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sea Cucumber)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sea Urchin)</td>
      <td>Artisan Goods</td>
      <td>220g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Shad)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Shrimp)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Slimejack)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Smallmouth Bass)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Snail)</td>
      <td>Artisan Goods</td>
      <td>124g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Spook Fish)</td>
      <td>Artisan Goods</td>
      <td>280g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Stingray)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Stonefish)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sturgeon)</td>
      <td>Artisan Goods</td>
      <td>500g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sunfish)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Super Cucumber)</td>
      <td>Artisan Goods</td>
      <td>310g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tiger Trout)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tilapia)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tuna)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Void Salmon)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Walleye)</td>
      <td>Artisan Goods</td>
      <td>164g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Woodskip)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Beer</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Caviar</td>
      <td>Artisan Goods</td>
      <td>500g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cheese</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Cloth</td>
      <td>Artisan Goods</td>
      <td>470g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Duck Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>375g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Goat Cheese</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Green Tea</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Blue Jazz)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Fairy Rose)</td>
      <td>Artisan Goods</td>
      <td>680g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Poppy)</td>
      <td>Artisan Goods</td>
      <td>380g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Summer Spangle)</td>
      <td>Artisan Goods</td>
      <td>280g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Sunflower)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Tulip)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Wild)*</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Ancient Fruit)</td>
      <td>Artisan Goods</td>
      <td>1,150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Apple)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Apricot)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Banana)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Blackberry)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Blueberry)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cactus Fruit)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cherry)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Coconut)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cranberries)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Crystal Fruit)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Grape)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>130g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Mango)</td>
      <td>Artisan Goods</td>
      <td>310g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Melon)</td>
      <td>Artisan Goods</td>
      <td>550g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Orange)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Peach)</td>
      <td>Artisan Goods</td>
      <td>330g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Pineapple)</td>
      <td>Artisan Goods</td>
      <td>650g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Pomegranate)</td>
      <td>Artisan Goods</td>
      <td>330g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Rhubarb)</td>
      <td>Artisan Goods</td>
      <td>490g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Salmonberry)</td>
      <td>Artisan Goods</td>
      <td>60g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Spice Berry)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Starfruit)</td>
      <td>Artisan Goods</td>
      <td>1,550g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Strawberry)</td>
      <td>Artisan Goods</td>
      <td>290g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Wild Plum)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Amaranth)</td>
      <td>Artisan Goods</td>
      <td>337g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Artichoke)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Beet)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Bok Choy)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Cauliflower)</td>
      <td>Artisan Goods</td>
      <td>393g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Corn)</td>
      <td>Artisan Goods</td>
      <td>112g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Eggplant)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Fiddlehead Fern)</td>
      <td>Artisan Goods</td>
      <td>202g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Garlic)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Green Bean)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Hops)</td>
      <td>Artisan Goods</td>
      <td>56g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Kale)</td>
      <td>Artisan Goods</td>
      <td>247g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Parsnip)</td>
      <td>Artisan Goods</td>
      <td>78g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Potato)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Pumpkin)</td>
      <td>Artisan Goods</td>
      <td>720g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Radish)</td>
      <td>Artisan Goods</td>
      <td>202g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Red Cabbage)</td>
      <td>Artisan Goods</td>
      <td>585g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Taro Root)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Tomato)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Unmilled Rice)</td>
      <td>Artisan Goods</td>
      <td>67g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Wheat)</td>
      <td>Artisan Goods</td>
      <td>56g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Yam)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>190g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pale Ale</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pickles (Amaranth)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Artichoke)</td>
      <td>Artisan Goods</td>
      <td>370g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Beet)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Bok Choy)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Cauliflower)</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Corn)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Eggplant)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Fiddlehead Fern)</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Garlic)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Ginger)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Green Bean)</td>
      <td>Artisan Goods</td>
      <td>130g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Hops)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Kale)</td>
      <td>Artisan Goods</td>
      <td>270g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Parsnip)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Potato)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Pumpkin)</td>
      <td>Artisan Goods</td>
      <td>690g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Radish)</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Red Cabbage)</td>
      <td>Artisan Goods</td>
      <td>570g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Taro Root)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Tea Leaves)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Tomato)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Unmilled Rice)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Wheat)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Yam)</td>
      <td>Artisan Goods</td>
      <td>370g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Truffle Oil</td>
      <td>Artisan Goods</td>
      <td>1,065g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Void Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>275g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Wine (Ancient Fruit)</td>
      <td>Artisan Goods</td>
      <td>1,650g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Apple)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Apricot)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Banana)</td>
      <td>Artisan Goods</td>
      <td>450g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Blackberry)</td>
      <td>Artisan Goods</td>
      <td>60g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Blueberry)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Cactus Fruit)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Cherry)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Coconut)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Cranberries)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Crystal Fruit)</td>
      <td>Artisan Goods</td>
      <td>450g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Grape)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wine (Mango)</td>
      <td>Artisan Goods</td>
      <td>390g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Melon)</td>
      <td>Artisan Goods</td>
      <td>750g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Orange)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Peach)</td>
      <td>Artisan Goods</td>
      <td>420g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Pineapple)</td>
      <td>Artisan Goods</td>
      <td>900g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Pomegranate)</td>
      <td>Artisan Goods</td>
      <td>420g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Rhubarb)</td>
      <td>Artisan Goods</td>
      <td>660g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Salmonberry)</td>
      <td>Artisan Goods</td>
      <td>15g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wine (Spice Berry)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Starfruit)</td>
      <td>Artisan Goods</td>
      <td>2,250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Strawberry)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Wild Plum)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Algae Soup</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Artichoke Dip</td>
      <td>Cooking</td>
      <td>210g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Autumn's Bounty</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Baked Fish</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Banana Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Bean Hotpot</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Blackberry Cobbler</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blueberry Tart</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Bread</td>
      <td>Cooking</td>
      <td>60g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Bruschetta</td>
      <td>Cooking</td>
      <td>210g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Carp Surprise</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Cheese Cauliflower</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Chocolate Cake</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Chowder</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Coleslaw</td>
      <td>Cooking</td>
      <td>345g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Complete Breakfast</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Cookie</td>
      <td>Cooking</td>
      <td>140g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Crab Cakes</td>
      <td>Cooking</td>
      <td>275g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Cranberry Candy</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Cranberry Sauce</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Crispy Bass</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Dish O' The Sea</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Eggplant Parmesan</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Escargot</td>
      <td>Cooking</td>
      <td>125g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Farmer's Lunch</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Fiddlehead Risotto</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Fish Stew</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Fish Taco</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Fried Calamari</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Fried Eel</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Fried Egg</td>
      <td>Cooking</td>
      <td>35g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Fried Mushroom</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Fruit Salad</td>
      <td>Cooking</td>
      <td>450g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Ginger Ale</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Glazed Yams</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Hashbrowns</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Ice Cream</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Life Elixir</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Lobster Bisque</td>
      <td>Cooking</td>
      <td>205g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Lucky Lunch</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Maki Roll</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Mango Sticky Rice</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Maple Bar</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Miner's Treat</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Oil of Garlic</td>
      <td>Cooking</td>
      <td>1,000g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Omelet</td>
      <td>Cooking</td>
      <td>125g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pale Broth</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pancakes</td>
      <td>Cooking</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Parsnip Soup</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pepper Poppers</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pink Cake</td>
      <td>Cooking</td>
      <td>480g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pizza</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Plum Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Poi</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Poppyseed Muffin</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pumpkin Pie</td>
      <td>Cooking</td>
      <td>385g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pumpkin Soup</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Radish Salad</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Red Plate</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rhubarb Pie</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rice Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Roasted Hazelnuts</td>
      <td>Cooking</td>
      <td>270g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Roots Platter</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Salad</td>
      <td>Cooking</td>
      <td>110g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Salmon Dinner</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Sashimi</td>
      <td>Cooking</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Seafoam Pudding</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Shrimp Cocktail</td>
      <td>Cooking</td>
      <td>160g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Spaghetti</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Spicy Eel</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Squid Ink Ravioli</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Stir Fry</td>
      <td>Cooking</td>
      <td>335g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Strange Bun</td>
      <td>Cooking</td>
      <td>225g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Stuffing</td>
      <td>Cooking</td>
      <td>165g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Super Meal</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Survival Burger</td>
      <td>Cooking</td>
      <td>180g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Tom Kha Soup</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Tortilla</td>
      <td>Cooking</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Triple Shot Espresso</td>
      <td>Cooking</td>
      <td>450g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Tropical Curry</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Trout Soup</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Vegetable Medley</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Albacore</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Anchovy</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Angler</td>
      <td>Fish</td>
      <td>900g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blobfish</td>
      <td>Fish</td>
      <td>500g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blue Discus</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Bream</td>
      <td>Fish</td>
      <td>45g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Bullhead</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Carp</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Catfish</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Chub</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Cockle</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Crab</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crayfish</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crimsonfish</td>
      <td>Fish</td>
      <td>1,500g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Dorado</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Eel</td>
      <td>Fish</td>
      <td>85g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Flounder</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Ghostfish</td>
      <td>Fish</td>
      <td>45g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Glacierfish</td>
      <td>Fish</td>
      <td>1,000g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Halibut</td>
      <td>Fish</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Herring</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Ice Pip</td>
      <td>Fish</td>
      <td>500g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Largemouth Bass</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Lava Eel</td>
      <td>Fish</td>
      <td>700g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Legend</td>
      <td>Fish</td>
      <td>5,000g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Lingcod</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Lionfish</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Lobster</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Midnight Carp</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Midnight Squid</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Mussel</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Mutant Carp</td>
      <td>Fish</td>
      <td>1,000g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Octopus</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Oyster</td>
      <td>Fish</td>
      <td>40g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Perch</td>
      <td>Fish</td>
      <td>55g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Periwinkle</td>
      <td>Fish</td>
      <td>20g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pike</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pufferfish</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rainbow Trout</td>
      <td>Fish</td>
      <td>65g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Red Mullet</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Red Snapper</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Salmon</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Sandfish</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Sardine</td>
      <td>Fish</td>
      <td>40g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Scorpion Carp</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Sea Cucumber</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Shad</td>
      <td>Fish</td>
      <td>60g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Shrimp</td>
      <td>Fish</td>
      <td>60g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Slimejack</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Smallmouth Bass</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Snail</td>
      <td>Fish</td>
      <td>65g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Spook Fish</td>
      <td>Fish</td>
      <td>220g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Squid</td>
      <td>Fish</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Stingray</td>
      <td>Fish</td>
      <td>180g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Stonefish</td>
      <td>Fish</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Sturgeon</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Sunfish</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Super Cucumber</td>
      <td>Fish</td>
      <td>250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Tiger Trout</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Tilapia</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Tuna</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Void Salmon</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Walleye</td>
      <td>Fish</td>
      <td>105g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Woodskip</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Blue Jazz</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Cave Carrot</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>25g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Chanterelle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>160g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Common Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>40g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Crocus</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>60g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Daffodil</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>30g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Dandelion</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>40g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Fairy Rose</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>290g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Ginger</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>60g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Hazelnut</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>90g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Holly</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Leek</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Magma Cap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>400g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Maple Syrup</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Morel</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Oak Resin</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pine Tar</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Poppy</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>140g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Purple Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Red Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Sap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>2g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Snow Yam</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Spring Onion</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>8g</td>
      <td>Gold</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Summer Spangle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>90g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Sunflower</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Sweet Pea</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>55g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Tulip</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>30g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wild Horseradish</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Winter Root</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>70g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Ancient Fruit</td>
      <td>Fruits</td>
      <td>550g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Apple</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Apricot</td>
      <td>Fruits</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Banana</td>
      <td>Fruits</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blackberry</td>
      <td>Fruits</td>
      <td>20g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Blueberry</td>
      <td>Fruits</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Cactus Fruit</td>
      <td>Fruits</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Cherry</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Coconut</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Cranberries</td>
      <td>Fruits</td>
      <td>75g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Crystal Fruit</td>
      <td>Fruits</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Grape</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Hot Pepper</td>
      <td>Fruits</td>
      <td>40g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Mango</td>
      <td>Fruits</td>
      <td>130g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Melon</td>
      <td>Fruits</td>
      <td>250g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Orange</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Peach</td>
      <td>Fruits</td>
      <td>140g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pineapple</td>
      <td>Fruits</td>
      <td>300g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Pomegranate</td>
      <td>Fruits</td>
      <td>140g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Rhubarb</td>
      <td>Fruits</td>
      <td>220g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Salmonberry</td>
      <td>Fruits</td>
      <td>5g</td>
      <td>Gold</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Spice Berry</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Starfruit</td>
      <td>Fruits</td>
      <td>750g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Strawberry</td>
      <td>Fruits</td>
      <td>120g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Wild Plum</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Aerinite</td>
      <td>Minerals</td>
      <td>125g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Alamite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Amethyst</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aquamarine</td>
      <td>Minerals</td>
      <td>180g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Baryte</td>
      <td>Minerals</td>
      <td>50g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Basalt</td>
      <td>Minerals</td>
      <td>175g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bixite</td>
      <td>Minerals</td>
      <td>300g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Calcite</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Celestine</td>
      <td>Minerals</td>
      <td>125g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Diamond</td>
      <td>Minerals</td>
      <td>750g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Dolomite</td>
      <td>Minerals</td>
      <td>300g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Earth Crystal</td>
      <td>Minerals</td>
      <td>50g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Emerald</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Esperite</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fairy Stone</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fire Opal</td>
      <td>Minerals</td>
      <td>350g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fire Quartz</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fluorapatite</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Frozen Tear</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Geminite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ghost Crystal</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Granite</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Helvite</td>
      <td>Minerals</td>
      <td>450g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Hematite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jade</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jagoite</td>
      <td>Minerals</td>
      <td>115g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jamborite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jasper</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Kyanite</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lemon Stone</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Limestone</td>
      <td>Minerals</td>
      <td>15g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lunarite</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Malachite</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Marble</td>
      <td>Minerals</td>
      <td>110g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Mudstone</td>
      <td>Minerals</td>
      <td>25g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Nekoite</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Neptunite</td>
      <td>Minerals</td>
      <td>400g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Obsidian</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ocean Stone</td>
      <td>Minerals</td>
      <td>220g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Opal</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Orpiment</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Petrified Slime</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Prismatic Shard</td>
      <td>Minerals</td>
      <td>2,000g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pyrite</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Quartz</td>
      <td>Minerals</td>
      <td>25g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ruby</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Sandstone</td>
      <td>Minerals</td>
      <td>60g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Slate</td>
      <td>Minerals</td>
      <td>85g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Soapstone</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Star Shards</td>
      <td>Minerals</td>
      <td>500g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Thunder Egg</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tigerseye</td>
      <td>Minerals</td>
      <td>275g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Topaz</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Amaranth</td>
      <td>Vegetables</td>
      <td>150g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Artichoke</td>
      <td>Vegetables</td>
      <td>160g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Beet</td>
      <td>Vegetables</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Bok Choy</td>
      <td>Vegetables</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Cauliflower</td>
      <td>Vegetables</td>
      <td>175g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Corn</td>
      <td>Vegetables</td>
      <td>50g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Eggplant</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Fiddlehead Fern</td>
      <td>Vegetables</td>
      <td>90g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Garlic</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Green Bean</td>
      <td>Vegetables</td>
      <td>40g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Hops</td>
      <td>Vegetables</td>
      <td>25g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Kale</td>
      <td>Vegetables</td>
      <td>110g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Parsnip</td>
      <td>Vegetables</td>
      <td>35g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Potato</td>
      <td>Vegetables</td>
      <td>80g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Pumpkin</td>
      <td>Vegetables</td>
      <td>320g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Radish</td>
      <td>Vegetables</td>
      <td>90g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Red Cabbage</td>
      <td>Vegetables</td>
      <td>260g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Taro Root</td>
      <td>Vegetables</td>
      <td>100g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Tea Leaves</td>
      <td>Vegetables</td>
      <td>50g</td>
      <td>Gold</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tomato</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Gold</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Unmilled Rice</td>
      <td>Vegetables</td>
      <td>30g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Wheat</td>
      <td>Vegetables</td>
      <td>25g</td>
      <td>Gold</td>
      <td>4</td>
    </tr>
    <tr>
      <td>Yam</td>
      <td>Vegetables</td>
      <td>160g</td>
      <td>Gold</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Duck Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Duck Feather</td>
      <td>Animal Products</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Egg</td>
      <td>Animal Products</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Brown Egg</td>
      <td>Animal Products</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Goat Milk</td>
      <td>Animal Products</td>
      <td>225g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Golden Egg</td>
      <td>Animal Products</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Large Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Large Brown Egg</td>
      <td>Animal Products</td>
      <td>95g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Large Goat Milk</td>
      <td>Animal Products</td>
      <td>345g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Large Milk</td>
      <td>Animal Products</td>
      <td>190g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Milk</td>
      <td>Animal Products</td>
      <td>125g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Ostrich Egg</td>
      <td>Animal Products</td>
      <td>600g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Rabbit's Foot</td>
      <td>Animal Products</td>
      <td>565g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Void Egg</td>
      <td>Animal Products</td>
      <td>65g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Wool</td>
      <td>Animal Products</td>
      <td>340g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Aged Roe (Albacore)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Anchovy)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Blobfish)</td>
      <td>Artisan Goods</td>
      <td>560g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Blue Discus)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Bream)</td>
      <td>Artisan Goods</td>
      <td>104g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Bullhead)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Carp)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Catfish)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Chub)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Cockle)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Crab)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Crayfish)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Dorado)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Eel)</td>
      <td>Artisan Goods</td>
      <td>144g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Flounder)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Ghostfish)</td>
      <td>Artisan Goods</td>
      <td>104g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Halibut)</td>
      <td>Artisan Goods</td>
      <td>140g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Herring)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Ice Pip)</td>
      <td>Artisan Goods</td>
      <td>560g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Largemouth Bass)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lava Eel)</td>
      <td>Artisan Goods</td>
      <td>760g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lingcod)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lionfish)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Lobster)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Midnight Carp)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Mussel)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Octopus)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Oyster)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Perch)</td>
      <td>Artisan Goods</td>
      <td>114g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Periwinkle)</td>
      <td>Artisan Goods</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Pike)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Pufferfish)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Rainbow Trout)</td>
      <td>Artisan Goods</td>
      <td>124g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Red Mullet)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Red Snapper)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Salmon)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sandfish)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sardine)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Scorpion Carp)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sea Cucumber)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sea Urchin)</td>
      <td>Artisan Goods</td>
      <td>220g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Shad)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Shrimp)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Slimejack)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Smallmouth Bass)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Snail)</td>
      <td>Artisan Goods</td>
      <td>124g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Spook Fish)</td>
      <td>Artisan Goods</td>
      <td>280g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Stingray)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Stonefish)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sturgeon)</td>
      <td>Artisan Goods</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Sunfish)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Super Cucumber)</td>
      <td>Artisan Goods</td>
      <td>310g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tiger Trout)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tilapia)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Tuna)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Void Salmon)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Walleye)</td>
      <td>Artisan Goods</td>
      <td>164g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aged Roe (Woodskip)</td>
      <td>Artisan Goods</td>
      <td>134g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Beer</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Caviar</td>
      <td>Artisan Goods</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cheese</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Cloth</td>
      <td>Artisan Goods</td>
      <td>470g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Duck Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>375g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Goat Cheese</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Green Tea</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Blue Jazz)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Fairy Rose)</td>
      <td>Artisan Goods</td>
      <td>680g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Poppy)</td>
      <td>Artisan Goods</td>
      <td>380g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Summer Spangle)</td>
      <td>Artisan Goods</td>
      <td>280g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Sunflower)</td>
      <td>Artisan Goods</td>
      <td>260g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Tulip)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Honey (Wild)*</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Ancient Fruit)</td>
      <td>Artisan Goods</td>
      <td>1,150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Apple)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Apricot)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Banana)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Blackberry)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Blueberry)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cactus Fruit)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cherry)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Coconut)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Cranberries)</td>
      <td>Artisan Goods</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Crystal Fruit)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Grape)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>130g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Mango)</td>
      <td>Artisan Goods</td>
      <td>310g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Melon)</td>
      <td>Artisan Goods</td>
      <td>550g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Orange)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Peach)</td>
      <td>Artisan Goods</td>
      <td>330g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Pineapple)</td>
      <td>Artisan Goods</td>
      <td>650g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Pomegranate)</td>
      <td>Artisan Goods</td>
      <td>330g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Rhubarb)</td>
      <td>Artisan Goods</td>
      <td>490g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Salmonberry)</td>
      <td>Artisan Goods</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Spice Berry)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Starfruit)</td>
      <td>Artisan Goods</td>
      <td>1,550g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Strawberry)</td>
      <td>Artisan Goods</td>
      <td>290g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jelly (Wild Plum)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Amaranth)</td>
      <td>Artisan Goods</td>
      <td>337g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Artichoke)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Beet)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Bok Choy)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Cauliflower)</td>
      <td>Artisan Goods</td>
      <td>393g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Corn)</td>
      <td>Artisan Goods</td>
      <td>112g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Eggplant)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Fiddlehead Fern)</td>
      <td>Artisan Goods</td>
      <td>202g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Garlic)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Green Bean)</td>
      <td>Artisan Goods</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Hops)</td>
      <td>Artisan Goods</td>
      <td>56g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Kale)</td>
      <td>Artisan Goods</td>
      <td>247g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Parsnip)</td>
      <td>Artisan Goods</td>
      <td>78g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Potato)</td>
      <td>Artisan Goods</td>
      <td>180g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Pumpkin)</td>
      <td>Artisan Goods</td>
      <td>720g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Radish)</td>
      <td>Artisan Goods</td>
      <td>202g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Red Cabbage)</td>
      <td>Artisan Goods</td>
      <td>585g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Taro Root)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Tomato)</td>
      <td>Artisan Goods</td>
      <td>135g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Unmilled Rice)</td>
      <td>Artisan Goods</td>
      <td>67g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Wheat)</td>
      <td>Artisan Goods</td>
      <td>56g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Juice (Yam)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>190g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Pale Ale</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Pickles (Amaranth)</td>
      <td>Artisan Goods</td>
      <td>350g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Artichoke)</td>
      <td>Artisan Goods</td>
      <td>370g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Beet)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Bok Choy)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Cauliflower)</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Corn)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Eggplant)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Fiddlehead Fern)</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Garlic)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Ginger)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Green Bean)</td>
      <td>Artisan Goods</td>
      <td>130g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Hops)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Kale)</td>
      <td>Artisan Goods</td>
      <td>270g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Parsnip)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Potato)</td>
      <td>Artisan Goods</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Pumpkin)</td>
      <td>Artisan Goods</td>
      <td>690g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Radish)</td>
      <td>Artisan Goods</td>
      <td>230g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Red Cabbage)</td>
      <td>Artisan Goods</td>
      <td>570g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Taro Root)</td>
      <td>Artisan Goods</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Tea Leaves)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Tomato)</td>
      <td>Artisan Goods</td>
      <td>170g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Unmilled Rice)</td>
      <td>Artisan Goods</td>
      <td>110g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Wheat)</td>
      <td>Artisan Goods</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pickles (Yam)</td>
      <td>Artisan Goods</td>
      <td>370g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Truffle Oil</td>
      <td>Artisan Goods</td>
      <td>1,065g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Void Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>275g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Wine (Ancient Fruit)</td>
      <td>Artisan Goods</td>
      <td>1,650g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Apple)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Apricot)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Banana)</td>
      <td>Artisan Goods</td>
      <td>450g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Blackberry)</td>
      <td>Artisan Goods</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Wine (Blueberry)</td>
      <td>Artisan Goods</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Cactus Fruit)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Cherry)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Coconut)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Cranberries)</td>
      <td>Artisan Goods</td>
      <td>225g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Crystal Fruit)</td>
      <td>Artisan Goods</td>
      <td>450g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Grape)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Mango)</td>
      <td>Artisan Goods</td>
      <td>390g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Melon)</td>
      <td>Artisan Goods</td>
      <td>750g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Orange)</td>
      <td>Artisan Goods</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Peach)</td>
      <td>Artisan Goods</td>
      <td>420g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Pineapple)</td>
      <td>Artisan Goods</td>
      <td>900g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Pomegranate)</td>
      <td>Artisan Goods</td>
      <td>420g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Rhubarb)</td>
      <td>Artisan Goods</td>
      <td>660g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Salmonberry)</td>
      <td>Artisan Goods</td>
      <td>15g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wine (Spice Berry)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Starfruit)</td>
      <td>Artisan Goods</td>
      <td>2,250g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Strawberry)</td>
      <td>Artisan Goods</td>
      <td>360g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Traveling Cart)</td>
      <td>Artisan Goods</td>
      <td>400g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wine (Wild Plum)</td>
      <td>Artisan Goods</td>
      <td>240g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Algae Soup</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Artichoke Dip</td>
      <td>Cooking</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Autumn's Bounty</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Baked Fish</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Banana Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bean Hotpot</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Blackberry Cobbler</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Blueberry Tart</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bread</td>
      <td>Cooking</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bruschetta</td>
      <td>Cooking</td>
      <td>210g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Carp Surprise</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cheese Cauliflower</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Chocolate Cake</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Chowder</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Coleslaw</td>
      <td>Cooking</td>
      <td>345g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Complete Breakfast</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cookie</td>
      <td>Cooking</td>
      <td>140g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crab Cakes</td>
      <td>Cooking</td>
      <td>275g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cranberry Candy</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Cranberry Sauce</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crispy Bass</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Dish O' The Sea</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Eggplant Parmesan</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Escargot</td>
      <td>Cooking</td>
      <td>125g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Farmer's Lunch</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fiddlehead Risotto</td>
      <td>Cooking</td>
      <td>350g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fish Stew</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fish Taco</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fried Calamari</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fried Eel</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fried Egg</td>
      <td>Cooking</td>
      <td>35g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fried Mushroom</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fruit Salad</td>
      <td>Cooking</td>
      <td>450g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ginger Ale</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Glazed Yams</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Hashbrowns</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ice Cream</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Life Elixir</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lobster Bisque</td>
      <td>Cooking</td>
      <td>205g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lucky Lunch</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Maki Roll</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Mango Sticky Rice</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Maple Bar</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Miner's Treat</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Oil of Garlic</td>
      <td>Cooking</td>
      <td>1,000g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Omelet</td>
      <td>Cooking</td>
      <td>125g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pale Broth</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pancakes</td>
      <td>Cooking</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Parsnip Soup</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pepper Poppers</td>
      <td>Cooking</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pink Cake</td>
      <td>Cooking</td>
      <td>480g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pizza</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Plum Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Poi</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Poppyseed Muffin</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pumpkin Pie</td>
      <td>Cooking</td>
      <td>385g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pumpkin Soup</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Radish Salad</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Red Plate</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Rhubarb Pie</td>
      <td>Cooking</td>
      <td>400g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Rice Pudding</td>
      <td>Cooking</td>
      <td>260g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Roasted Hazelnuts</td>
      <td>Cooking</td>
      <td>270g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Roots Platter</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Salad</td>
      <td>Cooking</td>
      <td>110g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Salmon Dinner</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Sashimi</td>
      <td>Cooking</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Seafoam Pudding</td>
      <td>Cooking</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Shrimp Cocktail</td>
      <td>Cooking</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Spaghetti</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Spicy Eel</td>
      <td>Cooking</td>
      <td>175g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Squid Ink Ravioli</td>
      <td>Cooking</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Stir Fry</td>
      <td>Cooking</td>
      <td>335g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Strange Bun</td>
      <td>Cooking</td>
      <td>225g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Stuffing</td>
      <td>Cooking</td>
      <td>165g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Super Meal</td>
      <td>Cooking</td>
      <td>220g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Survival Burger</td>
      <td>Cooking</td>
      <td>180g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tom Kha Soup</td>
      <td>Cooking</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tortilla</td>
      <td>Cooking</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Triple Shot Espresso</td>
      <td>Cooking</td>
      <td>450g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tropical Curry</td>
      <td>Cooking</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Trout Soup</td>
      <td>Cooking</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Vegetable Medley</td>
      <td>Cooking</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Albacore</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Anchovy</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Angler</td>
      <td>Fish</td>
      <td>900g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Blobfish</td>
      <td>Fish</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Blue Discus</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Bream</td>
      <td>Fish</td>
      <td>45g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Bullhead</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Carp</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Catfish</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Chub</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Cockle</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Crab</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crayfish</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Crimsonfish</td>
      <td>Fish</td>
      <td>1,500g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Dorado</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Eel</td>
      <td>Fish</td>
      <td>85g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Flounder</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Ghostfish</td>
      <td>Fish</td>
      <td>45g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Glacierfish</td>
      <td>Fish</td>
      <td>1,000g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Halibut</td>
      <td>Fish</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Herring</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Ice Pip</td>
      <td>Fish</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Largemouth Bass</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Lava Eel</td>
      <td>Fish</td>
      <td>700g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Legend</td>
      <td>Fish</td>
      <td>5,000g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Lingcod</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Lionfish</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Lobster</td>
      <td>Fish</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Midnight Carp</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Midnight Squid</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Mussel</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Mutant Carp</td>
      <td>Fish</td>
      <td>1,000g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Octopus</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Oyster</td>
      <td>Fish</td>
      <td>40g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Perch</td>
      <td>Fish</td>
      <td>55g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Periwinkle</td>
      <td>Fish</td>
      <td>20g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pike</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Pufferfish</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Rainbow Trout</td>
      <td>Fish</td>
      <td>65g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Red Mullet</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Red Snapper</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Salmon</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Sandfish</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Sardine</td>
      <td>Fish</td>
      <td>40g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Scorpion Carp</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Sea Cucumber</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Shad</td>
      <td>Fish</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Shrimp</td>
      <td>Fish</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Slimejack</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Smallmouth Bass</td>
      <td>Fish</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Snail</td>
      <td>Fish</td>
      <td>65g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Spook Fish</td>
      <td>Fish</td>
      <td>220g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Squid</td>
      <td>Fish</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Stingray</td>
      <td>Fish</td>
      <td>180g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Stonefish</td>
      <td>Fish</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Sturgeon</td>
      <td>Fish</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Sunfish</td>
      <td>Fish</td>
      <td>30g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Super Cucumber</td>
      <td>Fish</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Tiger Trout</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Tilapia</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Tuna</td>
      <td>Fish</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Void Salmon</td>
      <td>Fish</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Walleye</td>
      <td>Fish</td>
      <td>105g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Woodskip</td>
      <td>Fish</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Blue Jazz</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Cave Carrot</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>25g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Chanterelle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Common Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>40g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Crocus</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Daffodil</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>30g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Dandelion</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>40g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Fairy Rose</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>290g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Ginger</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Hazelnut</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Holly</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Leek</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Magma Cap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>400g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Maple Syrup</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Morel</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Oak Resin</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pine Tar</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Poppy</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>140g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Purple Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Red Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Sap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>2g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Snow Yam</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Spring Onion</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>8g</td>
      <td>Iridium</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Summer Spangle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Sunflower</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Sweet Pea</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>55g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Tulip</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>30g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wild Horseradish</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Winter Root</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>70g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Ancient Fruit</td>
      <td>Fruits</td>
      <td>550g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Apple</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Apricot</td>
      <td>Fruits</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Banana</td>
      <td>Fruits</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Blackberry</td>
      <td>Fruits</td>
      <td>20g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Blueberry</td>
      <td>Fruits</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Cactus Fruit</td>
      <td>Fruits</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Cherry</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Coconut</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Cranberries</td>
      <td>Fruits</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Crystal Fruit</td>
      <td>Fruits</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Grape</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Hot Pepper</td>
      <td>Fruits</td>
      <td>40g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Mango</td>
      <td>Fruits</td>
      <td>130g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Melon</td>
      <td>Fruits</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Orange</td>
      <td>Fruits</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Peach</td>
      <td>Fruits</td>
      <td>140g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Pineapple</td>
      <td>Fruits</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Pomegranate</td>
      <td>Fruits</td>
      <td>140g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Rhubarb</td>
      <td>Fruits</td>
      <td>220g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Salmonberry</td>
      <td>Fruits</td>
      <td>5g</td>
      <td>Iridium</td>
      <td>5</td>
    </tr>
    <tr>
      <td>Spice Berry</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Starfruit</td>
      <td>Fruits</td>
      <td>750g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Strawberry</td>
      <td>Fruits</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Wild Plum</td>
      <td>Fruits</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Aerinite</td>
      <td>Minerals</td>
      <td>125g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Alamite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Amethyst</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Aquamarine</td>
      <td>Minerals</td>
      <td>180g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Baryte</td>
      <td>Minerals</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Basalt</td>
      <td>Minerals</td>
      <td>175g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Bixite</td>
      <td>Minerals</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Calcite</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Celestine</td>
      <td>Minerals</td>
      <td>125g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Diamond</td>
      <td>Minerals</td>
      <td>750g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Dolomite</td>
      <td>Minerals</td>
      <td>300g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Earth Crystal</td>
      <td>Minerals</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Emerald</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Esperite</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fairy Stone</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fire Opal</td>
      <td>Minerals</td>
      <td>350g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fire Quartz</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Fluorapatite</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Frozen Tear</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Geminite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ghost Crystal</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Granite</td>
      <td>Minerals</td>
      <td>75g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Helvite</td>
      <td>Minerals</td>
      <td>450g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Hematite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jade</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jagoite</td>
      <td>Minerals</td>
      <td>115g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jamborite</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Jasper</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Kyanite</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lemon Stone</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Limestone</td>
      <td>Minerals</td>
      <td>15g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Lunarite</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Malachite</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Marble</td>
      <td>Minerals</td>
      <td>110g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Mudstone</td>
      <td>Minerals</td>
      <td>25g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Nekoite</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Neptunite</td>
      <td>Minerals</td>
      <td>400g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Obsidian</td>
      <td>Minerals</td>
      <td>200g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ocean Stone</td>
      <td>Minerals</td>
      <td>220g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Opal</td>
      <td>Minerals</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Orpiment</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Petrified Slime</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Prismatic Shard</td>
      <td>Minerals</td>
      <td>2,000g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Pyrite</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Quartz</td>
      <td>Minerals</td>
      <td>25g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Ruby</td>
      <td>Minerals</td>
      <td>250g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Sandstone</td>
      <td>Minerals</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Slate</td>
      <td>Minerals</td>
      <td>85g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Soapstone</td>
      <td>Minerals</td>
      <td>120g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Star Shards</td>
      <td>Minerals</td>
      <td>500g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Thunder Egg</td>
      <td>Minerals</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tigerseye</td>
      <td>Minerals</td>
      <td>275g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Topaz</td>
      <td>Minerals</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Amaranth</td>
      <td>Vegetables</td>
      <td>150g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Artichoke</td>
      <td>Vegetables</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Beet</td>
      <td>Vegetables</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Bok Choy</td>
      <td>Vegetables</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Cauliflower</td>
      <td>Vegetables</td>
      <td>175g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Corn</td>
      <td>Vegetables</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Eggplant</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Fiddlehead Fern</td>
      <td>Vegetables</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Garlic</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Green Bean</td>
      <td>Vegetables</td>
      <td>40g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Hops</td>
      <td>Vegetables</td>
      <td>25g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Kale</td>
      <td>Vegetables</td>
      <td>110g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Parsnip</td>
      <td>Vegetables</td>
      <td>35g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Potato</td>
      <td>Vegetables</td>
      <td>80g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Pumpkin</td>
      <td>Vegetables</td>
      <td>320g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Radish</td>
      <td>Vegetables</td>
      <td>90g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Red Cabbage</td>
      <td>Vegetables</td>
      <td>260g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Taro Root</td>
      <td>Vegetables</td>
      <td>100g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Tea Leaves</td>
      <td>Vegetables</td>
      <td>50g</td>
      <td>Iridium</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Tomato</td>
      <td>Vegetables</td>
      <td>60g</td>
      <td>Iridium</td>
      <td>7</td>
    </tr>
    <tr>
      <td>Unmilled Rice</td>
      <td>Vegetables</td>
      <td>30g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Wheat</td>
      <td>Vegetables</td>
      <td>25g</td>
      <td>Iridium</td>
      <td>6</td>
    </tr>
    <tr>
      <td>Yam</td>
      <td>Vegetables</td>
      <td>160g</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
  </tbody>
</table>



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



```python
#Find categories with no 8 point entries
grange_df['Points'] = grange_df['Points'].apply(lambda x: 0 if x == '—' else int(x))
max_in_category = grange_df.groupby('category').Points.max()
display(max_in_category)
```


    category
    Animal Products                     8
    Artisan Goods                       8
    Cooking                             6
    Fish                                8
    Foraging, Flowers, and Tree Saps    8
    Fruits                              8
    Minerals                            6
    Vegetables                          8
    Name: Points, dtype: int64


## Professions
There are several profession bonuses available in the game, and most grange display calculators take these into account, but I'm narrowing my focus to items that lead to **maximum scores**. So I bet I can toss a bunch of these out, by finding profession bonuses that will never result in an 8 point item. To do this, let's check out the table for calculating points:


```python
import re
def strip_nonnumeric(string):
    return re.sub("[^0-9]", "", string)

price_df = parse_table(tables[11],get_heading=False)
price_df['Low Price'] = price_df['Sell Price'].apply(strip_nonnumeric)
price_df['High Price'] = price_df['Low Price'].shift(-1)
price_df = price_df.drop('Sell Price',axis = 1)
price_df.columns = ['Base','Silver','Gold','Iridium','Low Price','High Price']
display_table(price_df.loc[:,['Low Price','High Price','Base','Silver','Gold','Iridium']].fillna('Inf'))
```


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Low Price</th>
      <th>High Price</th>
      <th>Base</th>
      <th>Silver</th>
      <th>Gold</th>
      <th>Iridium</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>20</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
      <td>5</td>
    </tr>
    <tr>
      <td>20</td>
      <td>90</td>
      <td>2</td>
      <td>3</td>
      <td>4</td>
      <td>6</td>
    </tr>
    <tr>
      <td>90</td>
      <td>200</td>
      <td>3</td>
      <td>4</td>
      <td>5</td>
      <td>7</td>
    </tr>
    <tr>
      <td>200</td>
      <td>300</td>
      <td>4</td>
      <td>5</td>
      <td>6</td>
      <td>8</td>
    </tr>
    <tr>
      <td>300</td>
      <td>400</td>
      <td>5</td>
      <td>6</td>
      <td>6</td>
      <td>8</td>
    </tr>
    <tr>
      <td>400</td>
      <td>Inf</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>8</td>
    </tr>
  </tbody>
</table>


### Spring Onion Mastery

Spring Onions have a sell price of 8g, iridium quality will then be 16g, and with spring onion mastery will be 16g x 5 = 80g. That's still only 6 points, so this won't help us push anything up to 8.

### Bear Knowledge

I love the Bear, but bear knowledge is similarly pumping up pretty low priced items. Salmonberries are even cheaper than spring onions (5g), and Bear Knowledge only gives a x3, so those will never be an 8. Blackberries are 20g, Iridium quality is 40g, and so with bear knowledge, they sell for 120g, which is still only 7 points.

### Blacksmith, Gemologist, and Tapper

This is a bonus on items that can never be iridium quality (bars, minerals, and gems), so these can never be 8 points (see table).

### Tiller, Artisan, Rancher, Fisher, and Angler

These all require some actual calculation. We're looking for 8 point items, and so we can narrow our focus to iridium quality. 

- **Tiller** will apply to vegetables, non-foraged fruits, and flowers.
- **Artisan** will apply to artisan goods
- **Rancher** will apply to animal products
- **Fisher and Angler** will apply to Fish

Though some items within a category will be unaffected by these bonuses (e.g. flowers in the Foraging Flowers and Tree Saps category is the only one affected by tiller) there is no cross-category bonuses, and only one category with two (Fish).

Inspecting the items that are in the table, none of the profession bonuses actually boost an item enough to make an item an 8 that was not already an 8.


```python
import re
def strip_nonnumeric(string):
    return re.sub("[^0-9]", "", string)

def assign_points(price):
    if price >= 200:
        return 8
    elif price >= 90:
        return 7
    elif price >= 20:
        return 6
    elif price >= 0:
        return 5
    else:
        return '—'
    
bonus_dict = {'Animal Products':1.2,
              'Artisan Goods':1.4,
              'Cooking':1,
              'Fish (Fishing)':1.25,
              'Fish (Angler)':1.5,
              'Foraging, Flowers, and Tree Saps':1.1,
              'Fruits':1.1,
              'Minerals':1,
              'Vegetables':1.1}
    
df = grange_df.loc[(grange_df['Quality']=='Iridium')&(grange_df['Points']!='—'),:]
fishing_df = df.loc[df['category']=='Fish',:].copy()
fishing_df['category'] = 'Fish (Fishing)'
angler_df = df.loc[df['category']=='Fish',:].copy()
angler_df['category'] = 'Fish (Angler)'
df = pd.concat([df.loc[df['category']!='Fish',:].copy(),fishing_df,angler_df], axis = 0)
#df['Points'] = df['Points'].apply(lambda x: 0 if x == '—' else int(x))
df['Price'] = df['Price'].apply(strip_nonnumeric).astype(int)
df['Price w/ Bonus'] = (df['Price']*df['category'].apply(lambda x: bonus_dict[x])).astype(int)
df['Points w/ Bonus'] = df['Price w/ Bonus'].apply(assign_points)

boost_filter = (df['Points'].astype(int) < 8)&(df['Points w/ Bonus'] == 8)
print("Items that were less than 8 points and were boosted to 8 by profession bonuses:")
display_table(df.loc[boost_filter,:])
```

    Items that were less than 8 points and were boosted to 8 by profession bonuses:



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Item</th>
      <th>category</th>
      <th>Price</th>
      <th>Quality</th>
      <th>Points</th>
      <th>Price w/ Bonus</th>
      <th>Points w/ Bonus</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>



```python
high_score_df = grange_df.loc[(grange_df['Quality']=='Iridium')&(grange_df['Points']=='8'),:]
high_score_df['Price'] = high_score_df.Price.apply(strip_nonnumeric).astype(int)
summary_df = high_score_df.groupby('category').agg({'Item':'count','Price':min}).rename(columns={'Item':'Total Items','Price':'Minimum Sell Price'}).reset_index()
min_priced_items_df = (high_score_df
                       .merge(summary_df.loc[:,['category','Minimum Sell Price']], on = ['category']))
display_table(min_priced_items_df.loc[min_priced_items_df['Price'] == min_priced_items_df['Minimum Sell Price'],:])
```

    <ipython-input-89-f0dd172bd0dc>:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
      high_score_df['Price'] = high_score_df.Price.apply(strip_nonnumeric).astype(int)



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Item</th>
      <th>category</th>
      <th>Price</th>
      <th>Quality</th>
      <th>Points</th>
      <th>Minimum Sell Price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Milk</td>
      <td>Animal Products</td>
      <td>125</td>
      <td>Iridium</td>
      <td>8</td>
      <td>125</td>
    </tr>
    <tr>
      <td>Wine (Hot Pepper)</td>
      <td>Artisan Goods</td>
      <td>120</td>
      <td>Iridium</td>
      <td>8</td>
      <td>120</td>
    </tr>
    <tr>
      <td>Dorado</td>
      <td>Fish</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Flounder</td>
      <td>Fish</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Largemouth Bass</td>
      <td>Fish</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Lionfish</td>
      <td>Fish</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Midnight Squid</td>
      <td>Fish</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Pike</td>
      <td>Fish</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Slimejack</td>
      <td>Fish</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Tuna</td>
      <td>Fish</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Snow Yam</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Apple</td>
      <td>Fruits</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Coconut</td>
      <td>Fruits</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Orange</td>
      <td>Fruits</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Beet</td>
      <td>Vegetables</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
    <tr>
      <td>Taro Root</td>
      <td>Vegetables</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
      <td>100</td>
    </tr>
  </tbody>
</table>


- Animal Products: Milk
- Artisan Goods: Hot Pepper Wine
- Fish: Dorado, Flounder, Largemouth Bass, Lionfish, Midnight Squid, Pike, Slimejack, Tuna
- Foraging, Flowers, and Tree Saps: Snow Yam
- Fruits: Apple, Coconut, Orange
- Vegetable: Beet, Taro Root

A lot of these are actually pretty easy to get. Iridium quality milk you can get by the pallet even in Year 1 with a cow at full hearts. Dorado, Flounder, and Lionfish are pretty easy fish to get a perfect catch, though you need Ginger Island unlocked to get the Lionfish. Iridium Coconuts can just be picked up off the ground in the Calico Desert provided you have the Botanist profession (and you really should). 

A few of these, though having a low sell price, do not meet my requirement of 'easy'. Hot pepper wine requires a fully upgraded house, and a lot of time (or fairy dust) to get to iridium quality. Snow Yams do not benefit from botanist since you dig them from artifact spots, therefore I *believe* the only way to get an iridium snow yam is to use deluxe fertilizer and winter seeds (and that's two RNG's I don't need). Beets and Taro Root would both require deluxe fertilizer, taro root has the advantage of not requiring watering, though by the time you have deluxe fertizlizer and Ginger island unlocked, you should have iridium sprinklers coming out your ears. Beets grow faster, so I can roll the dice more on less deluxe fertizer in the same amount of time. Since there is no other way to get an iridium vegetable, I'll stick with the beets, the other two will need replacing.


```python
category_filter = high_score_df['category'].isin(['Foraging, Flowers, and Tree Saps','Artisan Goods'])
category_filter &= high_score_df['Item'].str[:4] != 'Wine'
display_table(high_score_df.loc[category_filter,:].sort_values(['category','Price']))
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
      <td>Mayonnaise</td>
      <td>Artisan Goods</td>
      <td>190</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Beer</td>
      <td>Artisan Goods</td>
      <td>200</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Cheese</td>
      <td>Artisan Goods</td>
      <td>230</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Pale Ale</td>
      <td>Artisan Goods</td>
      <td>300</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Goat Cheese</td>
      <td>Artisan Goods</td>
      <td>400</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Snow Yam</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>100</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Poppy</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>140</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Morel</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>150</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Chanterelle</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>160</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Purple Mushroom</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>250</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Fairy Rose</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>290</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
    <tr>
      <td>Magma Cap</td>
      <td>Foraging, Flowers, and Tree Saps</td>
      <td>400</td>
      <td>Iridium</td>
      <td>8</td>
    </tr>
  </tbody>
</table>


Beer and Pale Ale have the same issue as wine. You can get gold quality cheese pretty easy and age it to iridium in about a week, so this is probably the way to go. If you have Ginger Island unlocked and really enjoy doing Professor Snail's bidding, iridium quality ostrich eggs are a way to get iridium quality mayo without the fully upgraded house.

Poppys and Fairy Roses would also require deluxe fertilizer, and I'd like to minimize that, as radioactive ore/Qi coins don't grow on trees (suggestions for 1.6). The mushrooms benefit from Botanist, and Morel's/Chanterelles are available in the secret woods. Magma Cap's can be picked up in the volcano and Purple Mushrooms are available in Professor Snail's mushroom cave.

So that leaves me with a fairly low-stress 125 point grange display:

- 4 Large Milk
- 1 Lionfish
- 1 Coconut
- 1 Cheese
- 1 Morel Mushroom
- 1 Beet

All Iridium quality. 14 pts for being me + 30 pts for 6 categories + 9 pts for 9 items + 9x8 pts for items. In my current playthrough, I have not unlocked the walnut room, so I will be stuck at 124 until I get that iridium beet. The rest of these items I have lying around in chests/fridges in Winter 2. My dream of a perfect Grange should be fulfilled in Year 3. I think someone who really made it a priority could get to this point by Year 2, but Year 1 is probably out of the question. Some people are *very* good at this game though, so I don't want to say impossible.


```python

```
