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

To determine the best city for coffee lovers, we will find a major city with the highest density of coffee shops out of five major US cities.

## Data Description

We will fetch data about coffee shops in following 5 largest US cities: 
 -	New York City, NY (Population: 8,622,357)
 -	Los Angeles, CA (Population: 4,085,014)
 -	Chicago, IL (Population: 2,670,406)
 -	Houston, TX (Population: 2,378,146)
 -	Phoenix, AZ (Population: 1,743,469)

Using geopy we will find the coordinates for each city center and then using Foursquare API we will collect the coffee shops data. After the data collection we will visualize each city data on a separate Folium map. Then we will measure the density using two different methods, and we will merge the results into a single table which will be sorted to find the winning city.
City with the highest density (lowest mean distance) will be considered the best.
