# Coffee Lovers guide to America - comparing five major US cities

----------------------------------------------------------------------------------------------------------------------
 Felix Reznitskiy
 
 December 18, 2020
 
----------------------------------------------------------------------------------------------------------------------

## Introduction

![image](./coffee-caffeinated-history.jpg)

Coffee first became popular in the U.S. after the Boston Tea Party, when the switch was seen as “patriotic,” [according to PBS](http://www.pbs.org/food/the-history-kitchen/history-coffee/). And since Starbucks debuted in 1971, the drink is now accessible almost anywhere you go. A recent survey by the National Coffee Association found that [62 percent](https://www.ncausa.org/Newsroom/NCA-releases-Atlas-of-American-Coffee) of Americans drink coffee every day, with the average coffee drinker consuming 3 cups daily.
What gave way to java culture? Science, for one, has convinced us that caffeine possesses multiple health benefits besides mental stimulation. At the right dosages, caffeine may contribute to [longevity](https://time.com/5326420/coffee-longevity-study/). Perhaps just as important, though, is coffee’s social purpose. Today, coffee stations are a staple of the workplace, and tens of thousands of shops serve as meeting places for friends, dates and coworkers – though in 2020 many have had to provide take-out service only due to the COVID-19 pandemic.

## Business Problem

Our customer wants to open a coffee shop in one of the major US cities. In order for the new business to be successful, he needs to find the best location for the new place. Therefore, we are requested to find the city and the neighborhood with the highest density of coffee shops.
To determine the best city for the new business, we will find a major city with the highest density of coffee shops out of five major US cities. Next, we will compare the neighborhoods to determine the one with the highest density.

## Data Description

We will fetch data about coffee shops in following 5 largest US cities:
 -	New York City, NY (Population: 8,622,357)
 -	Los Angeles, CA (Population: 4,085,014)
 -	Chicago, IL (Population: 2,670,406)
 -	Houston, TX (Population: 2,378,146)
 -	Phoenix, AZ (Population: 1,743,469)

Using geopy we will find the coordinates for each city center and then using Foursquare API we will collect the coffee shops data. After the data collection we will visualize each city data on a separate Folium map. Then we will measure the density, and we will merge the results into a single table which will be sorted to find the winning city. City with the highest density (lowest mean distance) will be considered the best.



### 1. Geocoders

We require geographical location data for each of the five cities. City center information will be used as a starting point for the FourSquare API (we will run search query around particular geographical location). We will use geopy.geocoders to obtain the city center coordinates for each of the five cities:
- city
- latitude
- longitude

### 2. Foursquare API

We need to make sure we are fetching only coffee shops during the Foursquare API search.
We will run Foursquare API once, and we will fetch one coffee-shop from one city in order to extract the category Id of "Coffee Shop". This Id will be used to limit the search and fetch only one venue category.
- category name
- category Id

After the city center information and category Id are fetched, we will run the FourSquare API search query and pull the list of coffee shops for each city:
- venue name
- venue category
- latitude
- longitude

We will create a Folium map for each city and visualize all the data to make a preliminary analysis of coffee shops density.

Next, we will use this data for measuring the density of coffee shops in selected cities. We will measure density as a mean distance from venues to the city center coordinates. City with the lowest mean distance will be considered as the best.

We will also measure the density as a mean distance from venues to the mean coordinates of all the coffee shops in the city. Then we will create a dataframe with the following columns:
- City
- Average_Proximity_To_The_City_Center
- Average_Distance_To_Mean_Coordinates
- Coffee_Shops_Per_City

As I mentioned above, the city with the lowest mean distance will be considered as the best.

After the best city is found, we will use K'Means clustering to find the neighborhoods with the highest density of coffee shops. We are going to utilize the pandas dataframes and Folium maps to cluster the venues and present the findings on the map.


```python
import numpy as np # library for working with arrays, vectors etc.
import pandas as pd # library for data analsysis
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)
import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe
import folium # library for generating the maps
#import json # library to handle JSON files
from pandas import json_normalize
import math
from scipy.spatial.distance import cdist # for calculating the distance between two points

#!conda install -c conda-forge geopy --yes # uncomment this line if you haven't completed the Foursquare API lab
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values

print('Libraries imported.')
```

    Libraries imported.
    


```python
cityList = ['New York, NY', 'Los Angeles, CA', 'Chicago, IL', 'Houston, TX', 'Phoenix, AZ']

cityCoordinates = {}

for city in cityList:
    address = city # 'New York City, NY'
    geolocator = Nominatim(user_agent="my_coffee_explorer")
    location = geolocator.geocode(address)
    cityCoordinates[city] = [location.latitude, location.longitude]
    print('The geograpical coordinate of {} are {}, {}.'.format(city, cityCoordinates[city][0], cityCoordinates[city][1]))
```

    The geograpical coordinate of New York, NY are 40.7127281, -74.0060152.
    The geograpical coordinate of Los Angeles, CA are 34.0536909, -118.242766.
    The geograpical coordinate of Chicago, IL are 41.8755616, -87.6244212.
    The geograpical coordinate of Houston, TX are 29.7589382, -95.3676974.
    The geograpical coordinate of Phoenix, AZ are 33.4484367, -112.0741417.
    


```python
search_query = 'Coffee'
#search_query = 'Coffee Shop'
radius = 500
#print(search_query + ' .... OK!')
CLIENT_ID = 'your Foursquare ID' # your Foursquare ID
CLIENT_SECRET = 'your Foursquare Secret' # your Foursquare Secret
ACCESS_TOKEN = 'your FourSquare Access Token' # your FourSquare Access Token
VERSION = '20180605'
LIMIT = 1 # we will use this single query result to fetch the category Id
#print('Your credentails:')
#print('CLIENT_ID: ' + CLIENT_ID)
#print('CLIENT_SECRET:' + CLIENT_SECRET)
```

First, we need to figure out the coffee shops category Id in order to proceed with fetching the coffee shops data using Foursquare API


```python
neighborhood_latitude = cityCoordinates['New York, NY'][0]
neighborhood_longitude = cityCoordinates['New York, NY'][1]

url = 'https://api.foursquare.com/v2/venues/search?client_id={}&client_secret={}&ll={},{}&oauth_token={}&v={}&query={}&radius={}&limit={}'.format(CLIENT_ID, CLIENT_SECRET, neighborhood_latitude, neighborhood_longitude,ACCESS_TOKEN, VERSION, search_query, radius, LIMIT)

# checking the URL
#print(url)

#fetching one Coffee Shop in order to get the category Id
queryResult = requests.get(url).json()

# fetching category name and id
#print(queryResult['response']['venues'][0]['categories'][0]['name'] + ", " + queryResult['response']['venues'][0]['categories'][0]['id']) #'4bf58dd8d48988d1e0931735'
print(queryResult['response']['venues'][0]) #'4bf58dd8d48988d1e0931735'
```

    {'id': '49c79540f964a520af571fe3', 'name': 'Blue Spoon Coffee Co.', 'location': {'address': '76 Chambers St', 'crossStreet': 'at Broadway', 'lat': 40.714427584609766, 'lng': -74.00685853301651, 'labeledLatLngs': [{'label': 'display', 'lat': 40.714427584609766, 'lng': -74.00685853301651}], 'distance': 202, 'postalCode': '10007', 'cc': 'US', 'city': 'New York', 'state': 'NY', 'country': 'United States', 'formattedAddress': ['76 Chambers St (at Broadway)', 'New York, NY 10007']}, 'categories': [{'id': '4bf58dd8d48988d1e0931735', 'name': 'Coffee Shop', 'pluralName': 'Coffee Shops', 'shortName': 'Coffee Shop', 'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/food/coffeeshop_', 'suffix': '.png'}, 'primary': True}], 'referralId': 'v-1608571888', 'hasPerk': False}
    

Now we can proceed with pulling the data


```python
LIMIT = 100
results = {}
for city in cityList:
    url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&near={}&limit={}&categoryId={}'.format(
        CLIENT_ID, CLIENT_SECRET, VERSION, city, LIMIT,
        "4bf58dd8d48988d1e0931735") # Category from the previous step
    results[city] = requests.get(url).json()
```


```python
#from pandas import json_normalize
df_venues={}
for city in cityList:
    venues = json_normalize(results[city]['response']['groups'][0]['items'])
    df_venues[city] = venues[['venue.name', 'venue.location.address', 'venue.location.lat', 'venue.location.lng']]
    df_venues[city].columns = ['name', 'address', 'lat', 'lng']
```
