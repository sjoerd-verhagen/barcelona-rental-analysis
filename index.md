# 🏙️ Barcelona Rental Market Analysis

**Author:** Sjoerd Verhagen  
**Tools used:** SQL • Python • Tableau  
**Live Tableau dashboard:** [View Here](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)
**GOAL**

---

## TL;DR 

---

## 🎯 Project Goal

My girlfriend and I will move to Barcelona in September 2025. To use my new skills in python, SQL and Tableau I thought of a project that is something I really want to figure out and also that will display what I learnt in my courses. 

To prepare for our move, I analysed the rental market using data from Idealista, the city’s most popular housing platform. With a student budget, I focused on furnished flats under €2,000 a month to see what is realistic and where to start our search. Through the analysis i hope to narrow down the choices of neighbourhoods, to get a feel for the city and to see whether there are certain districts better suited for our choice.


---

## 🔍 Key Questions

I aimed to answer:
- Which districts give us the best value for money (€/m²)?
- Which districts have the largest rental supply? 
- What are the average sizes and rents by district?

As I want to have a place where friends can stay over:
- How much more do two-bedroom flats cost compared to one-bedroom flats?


---

## 🛠️ Tools Used

1. Open Web Scraper for collecting rental data
2. Python for cleaning and preparing the data
3.  Open Geo Code for mapping coordinates (python supported)
4.  SQL for queries and data analysis
5. Tableau for creating clear visual insights and dashboard

---

## Chapter 1 – Cleaning the Raw Data

<details>
  <summary>Step 1 – Combining the raw files</summary


For step 1, I used Python to combine all individual CSV files into one dataset. I loaded each file from the project folder, appended them to a list of DataFrames, and then concatenated everything into a single DataFrame. I saved this as a new CSV file and quickly explored its structure to check that everything had loaded correctly.

```python
import os
import pandas as pd

folder_path = "/Users/sjoerdv/Documents/PERSOONLIJK/Portfolio/Data 27 jul"
csv_files = [f for f in os.listdir(folder_path) if f.endswith(".csv")]
df_list = []

# Loop through and read each file
for file in csv_files:
    file_path = os.path.join(folder_path, file)
    df = pd.read_csv(file_path)
    df_list.append(df)

# Combine all dataframes
combined_df = pd.concat(df_list, ignore_index=True)

# Save to new CSV
output_file = os.path.join(folder_path, "all_rent_data.csv")
combined_df.to_csv(output_file, index=False)

print(f"Combined {len(csv_files)} files into: {output_file}")

# Load the new combined file
df = pd.read_csv(output_file)

# Show shape
print("\nSHAPE of dataset:", df.shape)

# Show columns
print("\nCOLUMNS:")
print(df.columns.tolist())

# Show index
print("\nINDEX:")
print(df.index)

# Numeric stats
print("\nDESCRIPTIVE STATISTICS (numerical):")
print(df.describe())

# Non-numeric stats
print("\nDESCRIPTIVE STATISTICS (non-numerical):")
print(df.describe(include=['object']))

# Preview first 5 rows
print("\nFIRST 5 ROWS:")
print(df.head())
```

</details> <details> <summary>Step 2 – Cleaning up key columns</summary>


In this step, I cleaned the price, size, and price-per-m² columns. The raw data included symbols like “€” and “m²”. I stripped those out so the values are now usable as proper numbers.
- Converted price to price_clean, containing just the amount as a float.
- Did the same for price_per_m2 and size, which now have clean numerical values in new columns.
- Finally, I checked for missing values in those cleaned columns.



```python
import pandas as pd

# Load your dataset
file_path = "/Users/sjoerdv/Documents/PERSOONLIJK/Portfolio/Data 27 jul/all_rent_data.csv"
df = pd.read_csv(file_path)

# Clean 'price' (e.g. "1,100 €/month" → 1100.0)
df['price_clean'] = df['price'].str.replace(r'[^\d,]', '', regex=True)\
                                .str.replace(',', '')\
                                .astype(float)

# Clean 'price_per_m2' (e.g. "73.33 €/m²" → 73.33)
df['price_per_m2_clean'] = df['price_per_m2'].str.replace(r'[^\d.,]', '', regex=True)\
                                             .str.replace(',', '')\
                                             .astype(float)

# Clean 'size' (e.g. "15 m²" → 15.0)
df['size_clean'] = df['size'].str.replace(r'[^\d.,]', '', regex=True)\
                              .str.replace(',', '')\
                              .astype(float)

# Preview cleaned values
print("\nCleaned values:")
print(df[['price', 'price_clean', 'price_per_m2', 'price_per_m2_clean', 'size', 'size_clean']].head(10))

# Count non-missing values in each cleaned column
print("\nNon-missing values:")
print("price_clean:         ", df['price_clean'].notna().sum())
print("price_per_m2_clean:  ", df['price_per_m2_clean'].notna().sum())
print("size_clean:          ", df['size_clean'].notna().sum())
```

</details> <details>
  <summary>Step 3 – Further cleaning</summary


For step 3, I further cleaned the columns...


```python
import pandas as pd

# Load the dataset
file_path = "/Users/sjoerdv/Documents/PERSOONLIJK/Portfolio/Data 27 jul/all_rent_data_cleaned.csv"
df = pd.read_csv(file_path)

# Extract number of bedrooms (e.g. "2 bed." → 2.0)
df['bedrooms_clean'] = df['bedrooms'].str.extract(r'(\d+)').astype(float)

# Preview cleaned results
print("\n🛏️ Bedrooms cleaned preview:")
print(df[['bedrooms', 'bedrooms_clean']].head(10))

# Show descriptive statistics
print("\n📊 Descriptive statistics for bedrooms:")
print(df['bedrooms_clean'].describe())

# Save the updated file
output_path = "/Users/sjoerdv/Documents/PERSOONLIJK/Portfolio/Data 27 jul/all_rent_data_cleaned_v2.csv"
df.to_csv(output_path, index=False)

print(f"\n✅ File saved with cleaned bedrooms column:\n{output_path}")


——————
Code Block 3


import numpy as np

# Step 1: Extract numeric bedrooms from 'bedrooms' column
df['bedrooms_clean'] = df['bedrooms'].str.extract(r'(\d+)').astype(float)

# Step 2: Fill in 0 where listing-link indicates a Studio
is_studio = df['listing-link'].str.contains(r'\b[Ss]tudio\b', na=False)
df.loc[is_studio, 'bedrooms_clean'] = 0

# Step 3: Preview the result
print("\n🛏️ Cleaned bedrooms (including Studio):")
print(df[['listing-link', 'bedrooms', 'bedrooms_clean']].head(10))

# Step 4: Show stats
print("\n📊 Descriptive statistics for bedrooms_clean:")
print(df['bedrooms_clean'].describe())

# Step 5: Count missing values
print("\n❌ Missing values in bedrooms_clean:", df['bedrooms_clean'].isna().sum())



———————


# Fix remaining missing bedrooms based on keywords like 'Studio', 'Loft', etc.
studio_keywords = r'(Studio|Loft|Mini|Open-plan)'
studio_mask = df['bedrooms_clean'].isna() & df['listing-link'].str.contains(studio_keywords, case=False, na=False)
df.loc[studio_mask, 'bedrooms_clean'] = 0

# As fallback, fill any remaining NaNs with 0
df['bedrooms_clean'] = df['bedrooms_clean'].fillna(0)

# Ensure the column is numeric
df['bedrooms_clean'] = df['bedrooms_clean'].astype(int)

# Check for remaining missing values
missing = df['bedrooms_clean'].isna().sum()
print(f"❌ Missing values in bedrooms_clean: {missing}")

# Show descriptive statistics
print("\n📊 Descriptive statistics for bedrooms_clean:")
print(df['bedrooms_clean'].describe())
```
</details> 

</details> <details> <summary> Step 4 – GeoData</summary>

In this step i added the latitude and longitude for every apartment, through GeoData I can automate this process. By fillin gin the API and adding everything I made into geodata that I could later load into Tableau to make a visual representation of the barcelona map.

```python
import pandas as pd
from opencage.geocoder import OpenCageGeocode
import time
from tqdm import tqdm  # progress bar

# API key
key = '0a2df27082034108b046e61491ef609d'
geocoder = OpenCageGeocode(key)

# Load cleaned CSV
file_path = "/Users/sjoerdv/Documents/PERSOONLIJK/Portfolio/Data 27 jul/all_rent_data_cleaned_v3.csv"
df = pd.read_csv(file_path)

# Create address query (Street + Subdistrict + District + Barcelona, Spain)
def build_query(row):
    parts = [row['street_cleaned'], row['subdistrict'], row['district'], 'Barcelona, Spain']
    return ', '.join([p for p in parts if pd.notna(p)])

df['full_address'] = df.apply(build_query, axis=1)

# Geocode function
def geocode_address(query):
    try:
        results = geocoder.geocode(query)
        if results and len(results):
            lat = results[0]['geometry']['lat']
            lng = results[0]['geometry']['lng']
            country_code = results[0]['components'].get('country_code', '')
            timezone = results[0]['annotations']['timezone']['name']
            return pd.Series([lat, lng, country_code, timezone])
        else:
            return pd.Series([None, None, None, None])
    except Exception as e:
        print(f"Error geocoding {query}: {e}")
        return pd.Series([None, None, None, None])

# Apply geocoding with progress bar (and respect free tier: 1 request/sec)
tqdm.pandas()
df[['latitude', 'longitude', 'country_code', 'timezone']] = df['full_address'].progress_apply(lambda x: geocode_address(x))
    # Pause between requests to respect API rate limit
    # time.sleep(1)  # Uncomment if needed for free tier limit

# Save results
output_path = "/Users/sjoerdv/Documents/PERSOONLIJK/Portfolio/Data 27 jul/all_rent_data_geocoded.csv"
df.to_csv(output_path, index=False)
print(f"\n✅ Geocoded file saved to: {output_path}")
```
</details>


## Chapter 2 – Data Analysis (SQL)

<details>
  <summary>Step 2.1 – bladibla</summary

For step 1,

```sql
SELECT 
  district, 
  COUNT(*) AS number_of_listings,
  ROUND(AVG(price_clean),2) AS avg_price,
  MIN(price_clean) AS min_price,
  MAX(price_clean) AS max_price,
  ROUND(STDDEV_SAMP(price_clean),2) AS stddev_price
FROM table_v4
GROUP BY district
ORDER BY avg_price DESC;
```

Let us take a look at the table

| "district"            | "number_of_listings" | "avg_price" | "min_price" | "max_price" | "stddev_price" |
|-----------------------|----------------------|-------------|-------------|-------------|----------------|
| "Sant Martí"          | 134                  | 1627.01     | 650         | 2000        | 320.09         |
| "Eixample"            | 434                  | 1611.85     | 750         | 2000        | 293.27         |
| "Sant Andreu"         | 34                   | 1596.03     | 990         | 2000        | 357.77         |
| "Les Corts"           | 66                   | 1591.97     | 850         | 2000        | 295.73         |
| "Gràcia"              | 199                  | 1543.67     | 800         | 2000        | 306.06         |
| "Sarrià-Sant Gervasi" | 200                  | 1523.50     | 865         | 2000        | 244.91         |
| "Nou Barris"          | 17                   | 1479.12     | 850         | 2000        | 344.61         |
| "Sants-Montjuïc"      | 200                  | 1441.33     | 945         | 2000        | 244.22         |
| "Ciutat Vella"        | 846                  | 1339.66     | 700         | 2000        | 303.65         |
| "Horta Guinardó"      | 88                   | 1299.02     | 750         | 1995        | 284.45         |

Just to get an overview, I think as Nou Barris and Sant Andreu only have this many listings, I will exclude them as they are a very small sample size. Statistically with explorationary studies you take 30, and I will take that too as a benchmark. Lets focus on the other neighbourhoods with bigger sample sizes.
</details>

<details>
  <summary>Step 2.2 – Which districts give the best value for money (€/m²)? </summary

To answer this question lets take a look at the sqlquery...

```sql
SELECT 
  district,
  COUNT(*) AS number_of_listings,
  ROUND(AVG(price_per_m2_clean),2) AS avg_price_per_m2,
  MIN(price_per_m2_clean) AS min_price_per_m2,
  MAX(price_per_m2_clean) AS max_price_per_m2,
  ROUND(STDDEV_SAMP(price_per_m2_clean),2) AS stddev_price_per_m2
FROM table_v4
WHERE district NOT IN ('Sant Andreu', 'Nou Barris') 
GROUP BY district
ORDER BY avg_price_per_m2 ASC; 
```

In this table:

| district            | number_of_listings | avg_size_m2 | avg_rent | min_rent | max_rent | rent_stddev |
|---------------------|--------------------|-------------|----------|----------|----------|-------------|
| Horta Guinardó      |                 88 |       68.75 |  1299.02 |      750 |     1995 |      284.45 |
| Ciutat Vella        |                846 |       54.15 |  1339.66 |      700 |     2000 |      303.65 |
| Sants-Montjuïc      |                200 |       59.59 |  1441.33 |      945 |     2000 |      244.22 |
| Nou Barris          |                 17 |       80.76 |  1479.12 |      850 |     2000 |      344.61 |
| Sarrià-Sant Gervasi |                200 |       62.61 |   1523.5 |      865 |     2000 |      244.91 |
| Gràcia              |                199 |        63.9 |  1543.67 |      800 |     2000 |      306.06 |
| Les Corts           |                 66 |       73.26 |  1591.97 |      850 |     2000 |      295.73 |
| Sant Andreu         |                 34 |       69.18 |  1596.03 |      990 |     2000 |      357.77 |
| Eixample            |                434 |       65.59 |  1611.85 |      750 |     2000 |      293.27 |
| Sant Martí          |                134 |       66.08 |  1627.01 |      650 |     2000 |      320.09 |

**Horta Guinardó** and **Les Corts** give the best average price per m2, with a similar st dev compared to the other neighbourhoods. The average price is quie a bit lower then the other neighbourhoods, with a similar stdev. 
</details>

<details>
  <summary>Step 2.3 – Where is the largest rental supply? (with what variance?) (€/m²)? </summary

Here introducing the SQL

```sql
SELECT 
  district,
  COUNT(*) AS number_of_listings,
  ROUND(VAR_SAMP(price_clean),2) AS price_variance,
  ROUND(STDDEV_SAMP(price_clean),1) AS price_stddev
FROM table_v4
WHERE district NOT IN ('Sant Andreu', 'Nou Barris')
GROUP BY district
ORDER BY price_stddev DESC;
```

| district            | number_of_listings | price_variance | price_stddev |
|---------------------|--------------------|----------------|--------------|
| Ciutat Vella        |                846 |       92201.29 |        303.6 |
| Eixample            |                434 |        86004.7 |        293.3 |
| Sarrià-Sant Gervasi |                200 |       59978.54 |        244.9 |
| Sants-Montjuïc      |                200 |       59644.13 |        244.2 |
| Gràcia              |                199 |       93673.88 |        306.1 |
| Sant Martí          |                134 |      102454.79 |        320.1 |
| Horta Guinardó      |                 88 |       80911.56 |        284.4 |
| Les Corts           |                 66 |       87456.15 |        295.7 |

The biggest rental supply is in **Ciuttat Vella**, Followed by **Exiample**. We see here that the standard deviation of the price is also the lowest in **Sarrià-Sant Gervasi** and **Sants-Montjuï**, which have the third and forth most listings available. Even thought they had greater average pricesthen **Les Corts** and **Guinardo**. The biggest variance is in **Sant Marti**. 
</details>

<details>
  <summary>Step 2.4 – What are the average sizes and rents by district? </summary

```sql
SELECT 
  district,
  COUNT(*) AS number_of_listings,
  ROUND(AVG(size_clean),2) AS avg_size_m2,
  ROUND(AVG(price_clean),2) AS avg_rent,
  MIN(price_clean) AS min_rent,
  MAX(price_clean) AS max_rent,
  ROUND(STDDEV_SAMP(price_clean),2) AS rent_stddev
FROM table_v4
WHERE district NOT IN ('Sant Andreu', 'Nou Barris') 
GROUP BY district
ORDER BY avg_rent ASC;  
```

| district            | number_of_listings | avg_size_m2 | avg_rent | min_rent | max_rent | rent_stddev |
|---------------------|--------------------|-------------|----------|----------|----------|-------------|
| Horta Guinardó      |                 88 |       68.75 |  1299.02 |      750 |     1995 |      284.45 |
| Ciutat Vella        |                846 |       54.15 |  1339.66 |      700 |     2000 |      303.65 |
| Sants-Montjuïc      |                200 |       59.59 |  1441.33 |      945 |     2000 |      244.22 |
| Sarrià-Sant Gervasi |                200 |       62.61 |   1523.5 |      865 |     2000 |      244.91 |
| Gràcia              |                199 |        63.9 |  1543.67 |      800 |     2000 |      306.06 |
| Les Corts           |                 66 |       73.26 |  1591.97 |      850 |     2000 |      295.73 |
| Eixample            |                434 |       65.59 |  1611.85 |      750 |     2000 |      293.27 |
| Sant Martí          |                134 |       66.08 |  1627.01 |      650 |     2000 |      320.09 |

The average rent in Horta Guinardo and Cuitat Vella is the lowest, followed by Sants-Montjuïc. If you compare that to avg size, you see that Horta Guinardo also is the district with the second highest average size. 
</details>

<details>
  <summary>Step 2.5 – How much more do two-bedroom flats cost compared to one-bedroom flats? (per district) </summary
                                                                                                              
This query compares average rents for 1- and 2-bedroom apartments across Barcelona districts, excluding Sant Andreu and Nou Barris. It uses a CTE and a self-join to calculate both the absolute and percentage rent difference between the two apartment types. The output reveals how much more expensive 2-bedrooms are per district, offering clear insight into local rental pricing dynamics.                                                                                                                    
  ```sql

WITH rent_by_bedrooms AS (
  SELECT 
    district,
    bedrooms_cleaned,
    AVG(price_clean) AS avg_rent
  FROM table_v4
  WHERE bedrooms_cleaned IN (1, 2)
    AND district NOT IN ('Sant Andreu', 'Nou Barris')
  GROUP BY district, bedrooms_cleaned
)
SELECT 
  r1.district,
  r2.avg_rent - r1.avg_rent AS price_diff_2b_vs_1b,
  ROUND(( (r2.avg_rent - r1.avg_rent) / r1.avg_rent ) * 100, 1) AS pct_diff_2b_vs_1b,
  r1.avg_rent AS avg_rent_1b,
  r2.avg_rent AS avg_rent_2b
FROM rent_by_bedrooms r1
JOIN rent_by_bedrooms r2
  ON r1.district = r2.district
WHERE r1.bedrooms_cleaned = 1 
  AND r2.bedrooms_cleaned = 2
ORDER BY price_diff_2b_vs_1b DESC;
```

| district            | price_diff_2b_vs_1b | pct_diff_2b_vs_1b | avg_rent_1b | avg_rent_2b |
|---------------------|---------------------|-------------------|-------------|-------------|
| Les Corts           |               62.42 |             4.11% |      1519.1 |     1581.52 |
| Sants-Montjuïc      |              104.96 |             7.63% |     1375.14 |     1480.11 |
| Sarrià-Sant Gervasi |              132.93 |             9.19% |     1447.13 |     1580.06 |
| Eixample            |              176.34 |            11.73% |     1502.96 |      1679.3 |
| Gràcia              |              206.82 |            14.68% |     1409.24 |     1616.06 |
| Sant Martí          |              228.57 |            15.35% |        1489 |     1717.57 |
| Ciutat Vella        |              239.47 |             19.4% |     1234.53 |        1474 |
| Horta Guinardó      |              262.49 |            23.93% |     1096.87 |     1359.35 |

You see here that the lowest change is in Les Corts, followed by Sants-Montjuïc. Interestingly, in Sants-Montjuïc has the single lowest change for a second bedroom, and it has the top 3 lowest average rent costs as well. Where for Cuitat Vella en Horta Guinardo, the difference between one and two bedrooms was the biggest, even though they were the ones with the lowest rent. Could it be that it is skewed because there are way more 1 bedroom listings?
</details>

<details>
  <summary>Step 2.6 – Lets dive a bit deeper (per district) </summary

This query compares average rents for 1- and 2-bedroom apartments per district, excluding two districts. It calculates both the absolute and percentage price difference using a self-join on a grouped subquery. The result highlights how much more expensive 2-bedroom apartments are, giving further insight into rental pricing by district.

```sql
WITH rent_counts AS (
    SELECT 
        district,
        bedrooms_cleaned,
        COUNT(*) AS listings_count,
        ROUND(AVG(price_clean), 2) AS avg_rent
    FROM table_v4
    WHERE bedrooms_cleaned IN (1, 2)
      AND district NOT IN ('Sant Andreu', 'Nou Barris')
    GROUP BY district, bedrooms_cleaned
)
SELECT 
    r1.district,
    r1.listings_count AS one_bed_count,
    r2.listings_count AS two_bed_count,
    ROUND(r1.avg_rent, 2) AS avg_rent_1b,
    ROUND(r2.avg_rent, 2) AS avg_rent_2b,
    ROUND(r2.avg_rent - r1.avg_rent, 2) AS price_diff,
    ROUND(((r2.avg_rent - r1.avg_rent) / r1.avg_rent) * 100, 2) AS pct_diff
FROM rent_counts r1
JOIN rent_counts r2
  ON r1.district = r2.district
WHERE r1.bedrooms_cleaned = 1
  AND r2.bedrooms_cleaned = 2
ORDER BY pct_diff ASC;
```

| district            | one_bed_count | two_bed_count | avg_rent_1b | avg_rent_2b | price_diff | pct_diff |
|---------------------|---------------|---------------|-------------|-------------|------------|----------|
| Les Corts           |            10 |            23 |      1519.1 |     1581.52 |      62.42 |     4.11 |
| Sants-Montjuïc      |            69 |            73 |     1375.14 |     1480.11 |     104.97 |     7.63 |
| Sarrià-Sant Gervasi |            78 |            64 |     1447.13 |     1580.06 |     132.93 |     9.19 |
| Eixample            |           159 |           132 |     1502.96 |      1679.3 |     176.34 |    11.73 |
| Gràcia              |            50 |            85 |     1409.24 |     1616.06 |     206.82 |    14.68 |
| Sant Martí          |            30 |            68 |        1489 |     1717.57 |     228.57 |    15.35 |
| Ciutat Vella        |           371 |           286 |     1234.53 |        1474 |     239.47 |     19.4 |
| Horta Guinardó      |            15 |            31 |     1096.87 |     1359.35 |     262.48 |    23.93 |

As we see here is that indeed Horta Guinardo has the lowest number of listings, followed by Les Corts. Interesting, Cuitat Vella had the most listings. Here it seems that for bigger apartments with 2 bedrooms **Sants-Montjuïc** seems to be a good fit, as it has a good number of listings, a low average rent (top3), for a 1 bedroom apartment, and a low average rent for a 2 bedroom apartment (3rd place).
</details>


