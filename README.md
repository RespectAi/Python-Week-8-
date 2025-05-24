# Python-Week-8
# COVID-19 Data Analysis

## Overview

This project analyzes the Our World in Data (OWID) COVID-19 dataset to explore key metrics such as cases, deaths, vaccinations, and rates across selected countries: **Nigeria, Kenya, United States, India, South Africa, China**.

## Setup

### Requirements

* Python 3.7+
* pandas
* matplotlib
* plotly (optional for interactive maps)
* geopandas (optional for Matplotlib-based choropleth)

Install dependencies via:

```bash
pip install pandas matplotlib plotly geopandas
```

## Data

Download the dataset from Our World in Data:

```
owid-covid-data.csv
```

## 1. Data Loading

Load the CSV into a pandas DataFrame:

```python
import pandas as pd

df = pd.read_csv('owid-covid-data.csv')
df['date'] = pd.to_datetime(df['date'])
```

## 2. Data Cleaning & Inspection

* Check columns and preview:

  ```python
  print(df.columns)
  print(df.head())
  print(df.isnull().sum())
  ```
* Visualize missing values:

## 3. Filtering Countries of Interest

Filter for specific countries:

```python
countries = ['Nigeria','Kenya','United States','India','South Africa','China']
df_filt = df[df['location'].isin(countries)].copy()
```

## 4. Handling Missing Values

* Drop critical missing rows or fill/interpolate:

  ```python
  df_filt = df_filt.dropna(subset=['date','new_cases','total_cases'])
  df_filt[['new_cases','total_cases','new_deaths']] = df_filt[['new_cases','total_cases','new_deaths']].fillna(0)
  ```

![Filled Data](image\Death Rate.jpg)

## 5. Calculating Metrics
- Death Rate
- Total cases
- Total deaths over time 
- Daliy new cases
- Vaccination over time 
- Pulation Vaccinated 

Compute death rate and add column:

```python
df_filt['death_rate'] = df_filt['total_deaths'] / df_filt['total_cases'] * 100
````

## 6. Visualizations

### 6.1 Total Cases Over Time

```python
import matplotlib.pyplot as plt

for c in countries:
    data = df_filt[df_filt['location']==c]
    plt.plot(data['date'], data['total_cases'], label=c)
plt.legend()
plt.show()
```

### 6.2 Total Deaths Over Time

```
# Define countries and custom colors
countries_of_interest = ['Nigeria', 'Kenya', 'United States', 'India', 'South Africa', 'China']

# Filter data
df_filtered = df[df['location'].isin(countries_of_interest)].copy()
df_filtered = df_filtered.sort_values(by=['location', 'date'])

# Plot total deaths over time
plt.figure(figsize=(12, 6))

for country in countries_of_interest:
    country_data = df_filtered[df_filtered['location'] == country]
    plt.plot(country_data['date'], country_data['total_deaths'], label=country, color=colors[country])

plt.title('Total COVID-19 Deaths Over Time')
plt.xlabel('Date')
plt.ylabel('Total Deaths')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()


```

### 6.3 Cumulative Vaccinations Over Time

```
import pandas as pd
import matplotlib.pyplot as plt

# 1) Load and prepare data
df = pd.read_csv('owid-covid-data.csv')
df['date'] = pd.to_datetime(df['date'])

# Define your countries and colors
countries = ['Nigeria', 'Kenya', 'United States', 'India', 'South Africa', 'China']
colors = {
    'Nigeria': '#1f77b4',
    'Kenya': '#ff7f0e',
    'United States': '#2ca02c',
    'India': '#d62728',
    'South Africa': '#9467bd',
    'China': '#8c564b'
}

# Filter
df_filtered = df[df['location'].isin(countries)].copy()
df_filtered = df_filtered.sort_values(['location','date'])

# 2) Plot cumulative vaccinations over time
plt.figure(figsize=(12,6))
for country in countries:
    data = df_filtered[df_filtered['location']==country]
    # fillna so plot is continuous
    plt.plot(data['date'], data['total_vaccinations'].fillna(method='ffill'), 
             label=country, color=colors[country])
plt.title('Cumulative COVID-19 Vaccinations Over Time')
plt.xlabel('Date')
plt.ylabel('Total Vaccinations')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

# 3) Prepare latest % vaccinated data
# Use people_vaccinated_per_hundred (people with ≥1 dose per 100)
latest = (
    df_filtered
    .dropna(subset=['people_vaccinated_per_hundred'])
    .sort_values('date')
    .groupby('location')
    .tail(1)
)
```

### 6.4 Vaccinated Population Pie Chart

```
values = latest['people_vaccinated_per_hundred']
labels = latest['location']
pie_colors = [colors[c] for c in labels]

plt.figure(figsize=(8,8))
plt.pie(values, labels=labels, autopct='%1.1f%%', colors=pie_colors, startangle=140)
plt.title('Share of Population Vaccinated (≥1 Dose)')
plt.axis('equal')  # keep pie circular
plt.show()
```

### 6.5 Choropleth Maps

#### Cases Density

```python
import numpy as np
import pandas as pd
import plotly.express as px

# Load & prepare data
df = pd.read_csv('owid-covid-data.csv')
df['date'] = pd.to_datetime(df['date'])

# Filter out non-country entities
df = df[~df['iso_code'].str.startswith('OWID_')]

# Get latest values
latest = (
    df.dropna(subset=['iso_code', 'total_cases'])
    .sort_values('date')
    .groupby('iso_code', as_index=False)
    .last()[['iso_code', 'location', 'total_cases']]
)

# Clean and transform data
latest['total_cases'] = pd.to_numeric(latest['total_cases'], errors='coerce')
latest = latest.dropna(subset=['total_cases'])
latest['log_cases'] = np.log10(latest['total_cases'])

# Custom color scale configuration
min_value = 4  # 10^4 = 10,000 (10K)
max_value = 8   # 10^8 = 100,000,000 (100M)

color_scale = [
    [0.0, 'blue'],   # 10K
    [0.3, 'cyan'],   # 100K
    [0.6, 'yellow'], # 1M
    [1.0, 'red']     # 100M
]

fig = px.choropleth(
    latest,
    locations='iso_code',
    color='log_cases',
    hover_name='location',
    hover_data={'total_cases': ':,', 'log_cases': False},
    color_continuous_scale=color_scale,
    range_color=(min_value, max_value),
    title='COVID-19 Total Cases (10K = Blue → 100M = Red)'
)

fig.update_layout(
    geo=dict(showframe=False, showcoastlines=False),
    coloraxis_colorbar=dict(
        title='Total Cases',
        ticks="outside",
        tickvals=[4, 5, 6, 7, 8],
        ticktext=['10K', '100K', '1M', '10M', '100M']
    )
)
fig.show()
```

## 7. Usage

* Update paths in the code and image placeholders.
* Run each section sequentially in a Jupyter notebook or script.

##
