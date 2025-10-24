---
layout: post
title: The Stardew Valley Grange Display - Analysis
permalink: /Stardew_Grange2/
---
## The Stardew Valley Grange Display - Analysis
According to the Stardew Valley wiki, the grange display's score is calculated according to the formula:

14 + (# of items) + 5*(# of categories - max of 6) + (individual item scores)

So if you had 9 items in the display representing 9 categories, and 3 items had a score of 5, 4 items had a score of 7, and 2 had a score of 8 the final score would be:
14 + 9 + 5*6 + (5+5+5+7+7+7+7+8+8) = 112

To get a perfect score, you need 9 items representing at least 6 categories that are all 8 point items:
14 + 9 + 5*6 + (8+8+8+8+8+8+8+8+8) = 125

So we really need to find items that are worth 8 points. Some items are ineligible and get 0 points, they are marked on the table with a dash.
This table shows us the max score available in each of the categories. Notice that not every category has an 8 point item! Since there are still 6 categories available with 8 point items, we just won't use 'cooking' or 'Minerals'.

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


It's also important to note that not every item in the game is actually in the wiki table, for items outside the tables there is a lookup table using sell price and quality to determine points. Here, I'm using the 'tables' object from the [previous post](Stardew_Grange1).
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

## Professions
There are several profession bonuses available in the game, and most grange display calculators take these into account, but I'm narrowing my focus to items that lead to **maximum scores**. So I bet I can toss a bunch of these out, by finding profession bonuses that will never result in an 8 point item. To do this, let's check out the table for calculating points:


### Spring Onion Mastery

Spring Onions have a sell price of 8g, iridium quality doubles this to be 16g, and with spring onion mastery will be 16g x 5 = 80g. That's still only 6 points, so this won't help us push anything up to 8.

### Bear Knowledge

I love the Bear, but bear knowledge is similarly pumping up pretty low priced items. Salmonberries are even cheaper than spring onions (5g), and Bear Knowledge only gives a x3, so those will never be an 8. Blackberries are 20g, Iridium quality is 40g, and so with bear knowledge, they sell for 120g, which is still only 7 points.

### Blacksmith, Gemologist, and Tapper

These are bonuses on items that can never be iridium quality (bars, minerals, and gems), so these can never be 8 points (see table).

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

In other words, none. Perhaps this is no longer true if I include items outside the wiki tables, but that's for the next post. And if you want to calculate point values for an arbitrary display, the bonuses should obviously be included.
Now I'd like to look at the cheapest, easiest way to get 8 point items in each category. For now, I'll try looking at the items with the lowest sell price in each category.

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

A few of these, though having a low sell price, do not meet my requirement of 'easy'. Hot pepper wine requires a fully upgraded house, and a lot of time (or fairy dust) to get to iridium quality. Snow Yams do not benefit from botanist since you dig them from artifact spots, therefore I *believe* the only way to get an iridium snow yam is to use deluxe fertilizer and winter seeds (and that's two RNG's I don't need). Beets and Taro Root would both require deluxe fertilizer, taro root has the advantage of not requiring watering, though by the time you have deluxe fertizlizer and Ginger island unlocked, you should have iridium sprinklers coming out your ears. Beets grow faster, so I can roll the dice more on less deluxe fertizer in the same amount of time. Since there is no other way to get an iridium vegetable, I'll stick with the beets, the other two will need replacing. Let's look at the fuller list in these two categories.


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

Poppys and Fairy Roses would also require deluxe fertilizer, and I'd like to minimize that, as Qi coins don't grow on trees. It's hardly the scarcest thing in the game though, and I just got one as a drop in the dangerous mines. The mushrooms benefit from Botanist, and Morel's/Chanterelles are available in the secret woods. Magma Cap's can be picked up in the volcano and Purple Mushrooms are available in Professor Snail's mushroom cave.

So that leaves me with a fairly low-stress 125 point grange display:

- 4 Large Milk
- 1 Lionfish
- 1 Coconut
- 1 Cheese
- 1 Morel Mushroom
- 1 Beet

All Iridium quality. 14 pts for being me + 30 pts for 6 categories + 9 pts for 9 items + 9x8 pts for items. In my current playthrough, I have not unlocked the walnut room, so I will be stuck at 124 until I get that iridium beet. The rest of these items I have lying around in chests/fridges in Winter 2. My dream of a perfect Grange should be fulfilled in Year 3. I think someone who really made it a priority could get to this point by Year 2, but Year 1 is probably out of the question. Some people are *very* good at this game though, so I don't want to say impossible.
