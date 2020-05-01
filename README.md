
# Scraping
* getting html 
* parsing html


```python
# Import our libraries
import json
import requests

import pandas as pd

from bs4 import BeautifulSoup
from pymongo import MongoClient
```


```python
# Scrape an item

# step 1 get a url and it's params


url = "https://www.ebay.com/sch/i.html"
params = {"_nkw": "noise cancelling headphones bluetooth wireless"}
```

# Step 1 - Get HTML


```python
# make a request and start scraping
r = requests.get(url=url, params=params)
```


```python
r.status_code
```




    200




```python
# r.content is just the html file of the webpage
r.content[:3000]
```




    b'<!DOCTYPE html><!--[if IE 9]><html class="ie9 srp-ds6" lang="en"><![endif]--><!--[if gt IE 9]><!--><html class="srp-ds6" lang="en"><!--<![endif]--><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><script>"use strict";if(window.PerformanceObserver&&performance&&performance.mark&&performance.getEntriesByName){window.SRP=window.SRP||{};var paintObserver=new window.PerformanceObserver(function(e){var r=e.getEntries();r.sort(function(e,r){return e.startTime-r.startTime});var n=r[1].startTime;window.SRP.TTI_TIMER={lastInteractiveWindow:n};var t=new window.PerformanceObserver(function(e){for(var r=e.getEntries(),i=0,a=r.length;i<a;i++)r[i].startTime-n>=5e3&&(window.SRP.TTI_TIMER.timeToInteract=n,t.disconnect()),n=r[i].startTime+r[i].duration,window.SRP.TTI_TIMER.lastInteractiveWindow=n});t.observe({entryTypes:["longtask"]}),paintObserver.disconnect()});paintObserver.observe({entryTypes:["paint"]})};</script><link rel="preconnect" href="https://ir.ebaystatic.com" /><meta name="robots" content="noindex" /><title>noise cancelling headphones bluetooth wireless | eBay</title><meta name="description" content="Find great deals on eBay for noise cancelling headphones bluetooth wireless. Shop with confidence." /><meta Property="og:site_name" Content="eBay" /><meta name="referrer" content="unsafe-url" /><meta name="msvalidate.01" content="34E98E6F27109BE1A9DCF19658EEEE33" /><meta name="yandex-verification" content="6e11485a66d91eff" /><link href="https://i.ebayimg.com" rel="preconnect" /><meta name="y_key" content="acf32e2a69cbc2b0" /><meta name="google-site-verification" content="8kHr3jd3Z43q1ovwo0KVgo_NZKIEMjthBxti8m8fYTg" /><meta Property="og:image" Content="https://ir.ebaystatic.com/cr/v/c1/ebay-logo-1-1200x630-margin.png" /><meta property="fb:app_id" content="102628213125203" /><meta Property="og:type" Content="website" /><!-- NGMARS SIGNATURE --><link rel="dns-prefetch" href="//i.ebayimg.com"><link rel="dns-prefetch" href="//ir.ebaystatic.com"><link rel="dns-prefetch" href="//rover.ebay.com"><link rel="dns-prefetch" href="//svcs.ebay.com"><link rel="dns-prefetch" href="//pulsar.ebay.com"><link rel="dns-prefetch" href="//www.google.com/"><link rel="dns-prefetch" href="//tpc.googlesyndication.com"><link rel="dns-prefetch" href="//securepubads.g.doubleclick.net"><meta name="viewport" content="width=device-width, initial-scale=1, minimum-scale=1"><link rel="dns-prefetch" href="//ir.ebaystatic.com"><link rel="dns-prefetch" href="//secureir.ebaystatic.com"><link rel="dns-prefetch" href="//i.ebayimg.com"><link rel="dns-prefetch" href="//rover.ebay.com"><script>$ssgST=new Date().getTime();</script><link rel="stylesheet" href="https://ir.ebaystatic.com/rs/c/inception-rDMaYDUm.css">\n<link rel="stylesheet" href="https://ir.ebaystatic.com/rs/c/search-page-large-V45kSwDO.css"><link rel="stylesheet" type="text/css" href="https://ir.ebaystatic.com/rs/v/zqrwizx0ja4g3c55zecfmzzuqit.css?proc=DU:N"><!--M#s0-3-8--><noscript data-marko-key="@_wbind s0-3-8" id="s0-3-8"'



# Step 2 - Parse HTML


```python
webpage_soup = BeautifulSoup(r.content, 'html.parser')
```

### .find vs .find_all
* .find -> finds the first instance -> return a soup object
* .find_all -> finds all instances -> returns a list of soup objects

### what is a soup object?
* like a cursor that lets you query into a string of html


```python
results_list = webpage_soup.find_all("div", attrs={"class":"srp-river-results"})
results = results_list[0]
```


```python
result_boxes = results.find_all("div", attrs={"class": "s-item__wrapper"})
len(result_boxes)
```




    63




```python
first_box = result_boxes[0]
name = first_box.find("h3", attrs={"class":"s-item__title"}).text
price = first_box.find("span", attrs={"class":"s-item__price"}).text
```




    ('Waterproof Bluetooth 5.0 Earbuds Headphones Wireless Headset Noise Cancelling',
     '$13.99 to $35.99')



# let's do this for all the boxes now! 


```python
def get_list_of_data(result_boxes):
    list_of_data = []
    for box in result_boxes:
        data = {}

        data['name'] = box.find("h3", attrs={"class":"s-item__title"}).text
        data['price'] = box.find("span", attrs={"class":"s-item__price"}).text
        list_of_data.append(data)

    return list_of_data
```


```python
pd.DataFrame(get_list_of_data(result_boxes))
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
      <th>name</th>
      <th>price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Waterproof Bluetooth 5.0 Earbuds Headphones Wi...</td>
      <td>$13.99 to $35.99</td>
    </tr>
    <tr>
      <td>1</td>
      <td>A6S Wireless Earbuds Bluetooth 5.0 Headphones ...</td>
      <td>$20.99</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Bluetooth Wireless Headphones Bluedio T5S Nois...</td>
      <td>$26.00</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Waterproof Bluetooth 5.0 Earbuds Headphones Wi...</td>
      <td>$23.39</td>
    </tr>
    <tr>
      <td>4</td>
      <td>2020 Wireless Earbuds Bluetooth 5.0 Headphones...</td>
      <td>$18.60</td>
    </tr>
    <tr>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <td>58</td>
      <td>AUSDOM ANC8 Active Noise Cancelling Bluetooth ...</td>
      <td>$43.00</td>
    </tr>
    <tr>
      <td>59</td>
      <td>Cowin E7 [2018 Upgraded] Noise Cancelling Head...</td>
      <td>$47.99</td>
    </tr>
    <tr>
      <td>60</td>
      <td>Bluetooth Noise Cancelling Headphones Over Ear...</td>
      <td>$18.99</td>
    </tr>
    <tr>
      <td>61</td>
      <td>Bluedio T5S Bluetooth V4.2 Headphones Wireless...</td>
      <td>$26.00</td>
    </tr>
    <tr>
      <td>62</td>
      <td>Bluedio TN Bluetooth Noise Cancelling Earphone...</td>
      <td>$14.99</td>
    </tr>
  </tbody>
</table>
<p>63 rows × 2 columns</p>
</div>




```python
def get_result_boxes(webpage_soup):
    results_list = webpage_soup.find_all("div", attrs={"class":"srp-river-results"})[0]
    result_boxes = results_list.find_all("div", attrs={"class": "s-item__wrapper"})
    return result_boxes
```


```python
# let's do this for multiple pages
# Scrape an item

# step 1 get a url and it's params

all_data = []
url = "https://www.ebay.com/sch/i.html"
for page_number in range(1, 6):
    params = {"_nkw": "noise cancelling headphones bluetooth wireless", "&_pgn": page_number}
    r = requests.get(url=url, params=params)
    webpage_soup = BeautifulSoup(r.content, 'html.parser')
    result_boxes = get_result_boxes(webpage_soup)
    page_list_of_data = get_list_of_data(result_boxes)
    all_data.extend(page_list_of_data)
```


```python
len(all_data)
```




    315




```python
df = pd.DataFrame(all_data)
df.head()
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
      <th>name</th>
      <th>price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>Waterproof Bluetooth 5.0 Earbuds Headphones Wi...</td>
      <td>$13.99 to $35.99</td>
    </tr>
    <tr>
      <td>1</td>
      <td>A6S Wireless Earbuds Bluetooth 5.0 Headphones ...</td>
      <td>$20.99</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Waterproof Bluetooth 5.0 Earbuds Headphones Wi...</td>
      <td>$23.39</td>
    </tr>
    <tr>
      <td>3</td>
      <td>2020 Wireless Earbuds Bluetooth 5.0 Headphones...</td>
      <td>$18.60</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Bluetooth Wireless Headphones Bluedio T5S Nois...</td>
      <td>$26.00</td>
    </tr>
  </tbody>
</table>
</div>



# We want to store our data
Options
* MongoDB
* SQL
* xlsx
* csv
* tsv
* json

## Let's store in various ways


```python
# csv
df.to_csv("ebay_earbuds.csv", index=False)
```


```python
# tsv
df.to_csv("ebay_earbuds.tsv", index=False, sep='\t')
```


```python
# text
df.to_csv("ebay_earbuds.txt", index=False)
```


```python
# json
df.to_json("ebay_earbuds.json", lines=True, orient='records')
```


```python
import sqlite3
```


```python
conn = sqlite3.connect('ebay_earbuds.db')
```


```python
df.to_sql("earbuds", conn, schema=None, if_exists='fail', 
          index=True, index_label=None, chunksize=None, dtype=None)

```


```python
conn.execute("select name from sqlite_master where type='table';").fetchall()
```




    [('earbuds',)]



# Storing in a MongoDB


```python
client = MongoClient(host='localhost', port=27017)
```


```python
client.list_database_names()
```




    ['AprFT',
     'admin',
     'config',
     'ebay',
     'headstorm_client_db',
     'local',
     'music_tweets',
     'new_db',
     'tweets']




```python
ebay = client['ebay']
```


```python
ebay.list_collection_names()
```




    ['collection_one', 'tech_bags']




```python
earbuds = ebay['earbuds']
```


```python
earbuds.insert_many(all_data)
```




    <pymongo.results.InsertManyResult at 0x1242c3848>




```python
ebay.list_collection_names()
```




    ['earbuds', 'collection_one', 'tech_bags']




```python

```
