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

````

![Filled Data](path/to/filled_data.png)

## 5. Calculating Metrics
### 5.1 Death Rate
Compute death rate and add column:
```python
df_filt['death_rate'] = df_filt['total_deaths'] / df_filt['total_cases'] * 100
````
![Death rate ](image\Death Rate.jpg)

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
# Similar to above but with total_deaths
```

### 6.3 Cumulative Vaccinations Over Time

```
# Use total_vaccinations and forward-fill
```

### 6.4 Vaccinated Population Pie Chart

```
# Pie of people_vaccinated_per_hundred
```

### 6.5 Choropleth Maps

#### Cases Density

```python
# Plotly Express choropleth with iso_code and total_cases
```

#### Vaccination Rates

```python
# Swap to people_vaccinated_per_hundred for choropleth
```

## 7. Usage

* Update paths in the code and image placeholders.
* Run each section sequentially in a Jupyter notebook or script.

##
