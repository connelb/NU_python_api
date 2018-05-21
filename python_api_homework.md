

```python
# import dependencies
from citipy import citipy
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import openweathermapy.core as owm
import json
import requests
from env.config import api_key
import random as random
from pprint import pprint
```


```python
#1. Randomly select at least 500 unique (non-repeat) cities based on latitude and longitude.

coordinates = []

# Create the bounding box
#set latitude values - X values
miny = -45
maxy = 45

#set longitude values - Y values
minx = -180
maxx = 180
```


```python
#2. lookup citipy based on values and get cities
cities = []
cities_found_count = 0

def verify_unique_city(city):
    return city not in cities


def find_city(lng, lat):
    return citipy.nearest_city(lat, lon)


while cities_found_count<500:
    lat, lon =(random.uniform(minx,maxx),random.uniform(miny,maxy))
    city = citipy.nearest_city(lat, lon)
    if verify_unique_city(city)is True:
        cities.append(city)
        cities_found_count = cities_found_count +1


city_names = []   
for city in cities:
    city_names.append(city.city_name)
```


```python
#3.Perform a weather check on each of the cities using a series of successive API calls.

url = "http://api.openweathermap.org/data/2.5/weather?"

city_ids = []
city_lon = []
city_lat = []
city_urls =[]
city_max_temps =[]
city_humidities =[]
city_cloudiness =[]
city_wind_speeds =[]
city_urls =[]

for city_name in city_names:
    # Build query URL
    query_url = url + "appid=" + api_key + "&q=" + city_name
    city_urls.append(query_url)

    # Get weather data
    weather_response = requests.get(query_url)
    weather_json = weather_response.json()
    
#     pprint(weather_json)

    try:
        city_ids.append(weather_json['id'])
    except KeyError:
        city_ids.append("")
        
    try:
        city_lat.append(int(weather_json['coord']['lat']))
    except KeyError:
        city_lat.append(np.nan)
        
    try:
        city_lon.append(int(weather_json['coord']['lon']))
    except KeyError:
        city_lon.append(np.nan)
        
    try:
        city_max_temps.append(weather_json['main']['temp_max'])
    except KeyError:
        city_max_temps.append(np.nan)
        
    try:
        city_humidities.append(weather_json['main']['humidity'])
    except KeyError:
        city_humidities.append(np.nan)
        
    try:
        city_cloudiness.append(weather_json['clouds']['all'])
    except KeyError:
        city_cloudiness.append("")
        
    try:
        city_wind_speeds.append(weather_json['wind']['speed'])
    except KeyError:
        city_wind_speeds.append(np.nan)

```


```python
df = pd.DataFrame({
    'City':city_names,
    'id':city_ids,
    'max_temp':city_max_temps,
    'humidity':city_humidities,
    'cloudiness':city_cloudiness,
    'wind_speed':city_wind_speeds,
    "longitude":city_lon,
    'latitude':city_lat,
    'url':city_urls
})

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
      <th>City</th>
      <th>cloudiness</th>
      <th>humidity</th>
      <th>id</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>max_temp</th>
      <th>url</th>
      <th>wind_speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>qaanaaq</td>
      <td>0</td>
      <td>83.0</td>
      <td>3831208</td>
      <td>77.0</td>
      <td>-69.0</td>
      <td>265.426</td>
      <td>http://api.openweathermap.org/data/2.5/weather...</td>
      <td>2.77</td>
    </tr>
    <tr>
      <th>1</th>
      <td>hermanus</td>
      <td>44</td>
      <td>86.0</td>
      <td>3366880</td>
      <td>-34.0</td>
      <td>19.0</td>
      <td>288.026</td>
      <td>http://api.openweathermap.org/data/2.5/weather...</td>
      <td>5.02</td>
    </tr>
    <tr>
      <th>2</th>
      <td>chuy</td>
      <td>80</td>
      <td>99.0</td>
      <td>3443061</td>
      <td>-33.0</td>
      <td>-53.0</td>
      <td>287.726</td>
      <td>http://api.openweathermap.org/data/2.5/weather...</td>
      <td>4.67</td>
    </tr>
    <tr>
      <th>3</th>
      <td>qaqortoq</td>
      <td>75</td>
      <td>64.0</td>
      <td>3420846</td>
      <td>60.0</td>
      <td>-46.0</td>
      <td>274.150</td>
      <td>http://api.openweathermap.org/data/2.5/weather...</td>
      <td>2.60</td>
    </tr>
    <tr>
      <th>4</th>
      <td>xiropotamos</td>
      <td>75</td>
      <td>88.0</td>
      <td>733818</td>
      <td>41.0</td>
      <td>24.0</td>
      <td>291.150</td>
      <td>http://api.openweathermap.org/data/2.5/weather...</td>
      <td>1.50</td>
    </tr>
  </tbody>
</table>
</div>




```python
#remove NaN values
df.dropna(how='any', inplace=True)
df.head()

#save as csv file
df.to_csv("city_weather.csv",encoding="utf-8", index=False, header=True)
```

## Max Temperature (F) vs. Latitude


```python
# Max Temperature (F) vs. Latitude
plt.scatter(df["latitude"], df["max_temp"], marker="o",alpha=0.7)

# Incorporate the other graph properties
plt.title("City Latitude vs Max Temperature")
plt.ylabel("Max Temperature(F)")
plt.xlabel("Latitude")
plt.grid(True)

# Save the figure
plt.savefig("MaxTemperatureByLatitude.png")

# Show plot
plt.show()
```


![png](output_7_0.png)


## Humidity (%) vs. Latitude


```python
# Humidity(%) vs. Latitude
plt.scatter(df["latitude"], df["humidity"], marker="o",alpha=0.7)

# Incorporate the other graph properties
plt.title("City Latitude vs Humidity(%)")
plt.ylabel("Humidity(%)")
plt.xlabel("Latitude")
plt.grid(True)

# Save the figure
plt.savefig("PercentHumidityByLatitude.png")

# Show plot
plt.show()
```


![png](output_9_0.png)


## Cloudiness (%) vs. Latitude


```python
# Cloudiness(%) vs. Latitude
plt.scatter(df["latitude"], df["cloudiness"], marker="o",alpha=0.7)

# Incorporate the other graph properties
plt.title("City Latitude vs Cloudiness (%)")
plt.ylabel("Cloudiness (%)")
plt.xlabel("Latitude")
plt.grid(True)

# Save the figure
plt.savefig("PercentCloudinessByLatitude.png")

# Show plot
plt.show()
```


![png](output_11_0.png)


## Wind Speed (mph) vs. Latitude


```python
# Cloudiness(%) vs. Latitude
plt.scatter(df["latitude"], df["wind_speed"], marker="o",alpha=0.7)

# Incorporate the other graph properties
plt.title("City Latitude vs Wind Speed(mph)")
plt.ylabel("Wind Speed(mph)")
plt.xlabel("Latitude")
plt.grid(True)

# Save the figure
plt.savefig("WindSpeedByLatitude.png")

# Show plot
plt.show()
```


![png](output_13_0.png)

