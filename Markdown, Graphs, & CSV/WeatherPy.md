

```python
# Dependencies and Setup
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import seaborn as sns
import requests
import time
from pprint import pprint 

# Import API key
from config import api_key

# Incorporated citipy to determine city based on latitude and longitude
from citipy import citipy

# Output File (CSV)
output_data_file = "output_data/cities.csv"

# Range of latitudes and longitudes
lat_range = (-90, 90)
lng_range = (-180, 180)
```

## Generate Cities List


```python
# List for holding lat_lngs and cities
lat_lngs = []
cities = []

# Create a set of random lat and lng combinations
lats = np.random.uniform(low=-90.000, high=90.000, size=1500)
lngs = np.random.uniform(low=-180.000, high=180.000, size=1500)
lat_lngs = zip(lats, lngs)

# Identify nearest city for each lat, lng combination
for lat_lng in lat_lngs:
    city = citipy.nearest_city(lat_lng[0], lat_lng[1]).city_name
    
    # If the city is unique, then add it to a our cities list
    if city not in cities:
        cities.append(city)

# Print the city count to confirm sufficient count
len(cities)
```




    616



## Perform API Calls


```python
# OpenWeatherMap API Key

# Starting URL for Weather Map API Call
def call_city(city, request, api_key):
    try:
        url = "http://api.openweathermap.org/data/2.5/weather?units=Imperial&appid=" + api_key + "&q=" + city
        data = requests.get(url).json()
        latitude = data['coord']['lat']
    
        if (request == 'temperature'):
            answer = data['main']['temp_max']
        elif(request == 'humidity'):
            answer = data['main']['humidity']
        elif request == 'clouds':
            answer = data['clouds']['all']
        elif request == 'wind':
            answer = data['wind']['speed']
        else:
            print('That request is not valid!\nUse "temperature" , "humidity", "clouds", or "wind"')
    
        return latitude, answer

    except:
        return 'NA', "errorhere" + city

```


```python
def create_df(cities, request, api_key):
    df = pd.DataFrame(columns = ['Latitude', request], index = range(len(cities)))
    df['Latitude'] = [call_city(city, request, api_key)[0] for city in cities]
    df[request] = [call_city(city, request, api_key)[1] for city in cities]
    return df


```

## Temperature (F) vs. Latitude


```python
temp_df = create_df(cities,'temperature',api_key)
temp_df.head()
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
      <th>Latitude</th>
      <th>temperature</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-9.8</td>
      <td>80.54</td>
    </tr>
    <tr>
      <th>1</th>
      <td>37.17</td>
      <td>69.83</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-54.81</td>
      <td>32</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-20.63</td>
      <td>65.33</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-51.62</td>
      <td>33.8</td>
    </tr>
  </tbody>
</table>
</div>




```python
temp_df.to_csv('temperatures.csv')

temp_error_indexes = temp_df['Latitude'][temp_df['Latitude'] == 'NA'].index
cleaned_temp = temp_df.drop(temp_df.index[temp_error_indexes])

plt.scatter(cleaned_temp['Latitude'], cleaned_temp['temperature'])
plt.xlabel('Latitude')
plt.ylabel('Temperature (F)')
plt.title('Temperature at Different Latitudes')
sns.set_style("dark")
```


![png](output_8_0.png)


There's clearly a very strong inverse correlation between distance from the equator and temperature. As in, the closer you get to the equator, the hotter it gets. It seems that this trend is a bit stronger in the southern hemisphere.

## Humidity (%) vs. Latitude


```python
hum_df = create_df(cities,'humidity',api_key)
hum_df.head()
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
      <th>Latitude</th>
      <th>humidity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-9.8</td>
      <td>100</td>
    </tr>
    <tr>
      <th>1</th>
      <td>37.17</td>
      <td>84</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-54.81</td>
      <td>100</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-20.63</td>
      <td>56</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-51.62</td>
      <td>86</td>
    </tr>
  </tbody>
</table>
</div>




```python
hum_df.to_csv('humidity.csv')

hum_error_indexes = hum_df['Latitude'][hum_df['Latitude'] == 'NA'].index
cleaned_hum = hum_df.drop(hum_df.index[hum_error_indexes])

plt.scatter(cleaned_hum['Latitude'], cleaned_hum['humidity'])
plt.xlabel('Latitude')
plt.ylabel('Humidity (%)')
plt.title('Humidity at Different Latitudes')
sns.set_style("dark")
```


![png](output_12_0.png)


It appears that there is a slight correlation with higher humidity the closer you get to the equator, albeit far less strong that temperature correlation with latitude. Mostly it appears that this correlation starts around +/- 20 latitude. The closer you get from that point, the more humid it will be. However, further out than that, the humidity appears to have no correlation with latitude.

## Cloudiness (%) vs. Latitude


```python
clouds_df = create_df(cities,'clouds',api_key)
clouds_df.head()
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
      <th>Latitude</th>
      <th>clouds</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-9.8</td>
      <td>92</td>
    </tr>
    <tr>
      <th>1</th>
      <td>37.17</td>
      <td>64</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-54.81</td>
      <td>75</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-20.63</td>
      <td>48</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-51.62</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
clouds_df.to_csv('clouds.csv')

clouds_error_indexes = clouds_df['Latitude'][clouds_df['Latitude'] == 'NA'].index
cleaned_clouds = clouds_df.drop(clouds_df.index[clouds_error_indexes])

plt.scatter(cleaned_clouds['Latitude'], cleaned_clouds['clouds'])
plt.xlabel('Latitude')
plt.ylabel('Clouds (%)')
plt.title('Cloud Pecentage at Different Latitudes')
sns.set_style("dark")
```


![png](output_16_0.png)


There is clearly no correlation between cloud percentage and different latitudes. However, the data looks discrete, as evidenced by the neat rows. The cloud density is probably estimated by percentages of 5 or 2.5.

## Wind Speed (mph) vs. Latitude


```python
wind_df = create_df(cities,'wind',api_key)
wind_df.head()
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
      <th>Latitude</th>
      <th>wind</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-9.8</td>
      <td>19.39</td>
    </tr>
    <tr>
      <th>1</th>
      <td>37.17</td>
      <td>12.35</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-54.81</td>
      <td>4.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-20.63</td>
      <td>5.97</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-51.62</td>
      <td>10.29</td>
    </tr>
  </tbody>
</table>
</div>




```python
wind_df.to_csv('wind.csv')

wind_error_indexes = wind_df['Latitude'][wind_df['Latitude'] == 'NA'].index
cleaned_wind = wind_df.drop(wind_df.index[wind_error_indexes])

plt.scatter(cleaned_wind['Latitude'], cleaned_wind['wind'])
plt.xlabel('Latitude')
plt.ylabel('Wind Speed (mph)')
plt.title('Wind Speed at Different Latitudes')
sns.set_style("dark")
```


![png](output_20_0.png)


Clearly here, there is absolutely no correlation.
