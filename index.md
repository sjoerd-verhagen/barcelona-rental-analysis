<img src="https://github.com/sjoerd-verhagen/barcelona-rental-analysis/blob/main/headerbarca.png" width="1000">

**Author:** Sjoerd Verhagen  
**Tools used:** **SQL** (joins, aggregation, filtering, calculated fields), **Python** (data cleaning, regex, web scraping), **Tableau** (trend analysis, data visualisation), **Google Sheets** (data QA)  
**Live Tableau dashboard:** [View Here](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)


---

## 📌 TL;DR 

I scraped 2,218 rental listings from Idealista and used Python, SQL, and Tableau to analyse where to find the best-value rentals in Barcelona.

**Goal:**
- Find districts with the best balance of rent, size, and availability

**What I did:**
- Excluded _Nou Barris_ and _Sant Andreu_ due to too few listings for reliable analysis
- Compared average rent per m² and price variation by district
- Looked at average apartment sizes alongside rents
- Compared supply and pricing differences between one- and two-bedroom flats

**Key takeaway:**
- _Horta Guinardó_ offers the best value, with the lowest average rent and the second-largest apartments, but has limited listings
- _Ciutat Vella_ and _Eixample_ have the largest rental supply, but also the highest prices per m²
- _Sarrià-Sant Gervasi_ and _Sants-Montjuïc_ show the most stable prices among districts with high supply

**_Sants-Montjuïc_** combines a low rent premium for two bedrooms with decent apartment sizes and good availability, while **_Sarrià-Sant Gervasi_** offers slightly larger flats for only a small extra cost. Based on my goal of finding a spacious, affordable flat with a second bedroom for guests, these are the two districts I’d prioritise.


<img src="https://github.com/sjoerd-verhagen/barcelona-rental-analysis/blob/main/herograph4.png" width="1000">

---

## 🎯 Background & Project

My girlfriend and I are moving to **Barcelona** in **September 2025**, and I decided to turn our house hunt into a **data project**. Using **Python**, **SQL** and **Tableau**, I analysed rental listings from Idealista, Barcelona’s most popular housing platform, to see which neighbourhoods might suit us before we even start viewing flats in person.

I scraped **2,218 listings**, focusing on **furnished flats under €2,000 a month**. This fits our starter budget and makes sense since we are not bringing furniture from the **UK** or the **Netherlands**. The dataset included **price**, **size**, **location** and **number of bedrooms**.

The aim was simple: find **neighbourhoods that fit our needs**, get an early feel for the city, and see **where the best options are** for our move. And because we want space for friends to visit, I also looked at **how much extra a two bedroom flat costs compared to a one bedroom**.

---

## 🔍 Key Questions

With the data in hand, I set out to answer some practical questions — the kind that actually matter when you are moving somewhere new:
- Which districts offer the best value for money (€/m²)?
- Which districts have the largest rental supply?
- What are the average sizes and rents by district?
- How much more does a two bedroom flat cost than a one bedroom?
---

## 🛠️ Tools Used

1. Open Web Scraper for collecting rental data
2. Python for cleaning and preparing the data
3. Google Sheets – quick quality checks on the scraped data
4. Open Geo Code for mapping coordinates (python supported)
5. SQL for queries and data analysis
6. Tableau for creating clear visual insights and dashboard

---

## Part 1 – Cleaning the Raw Data

<details>
  <summary>🏠 Step 1.1 – Combining the raw files</summary

**Step overview**

For the first step, I used Python to combine all individual CSV files into one dataset. I loaded each file from the project folder, appended them to a list of DataFrames, and then concatenated everything into a single DataFrame. I saved this as a new CSV file and quickly explored its structure to check that everything had loaded correctly.

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

</details> <details> <summary>🏠 Step 1.2 – Cleaning up key columns</summary>

**Step overview**

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
  <summary>🏠 Step 1.3 – Further cleaning</summary

**Step overview**

For step 1.3, I extracted and cleaned the number of bedrooms from the listing text, creating a numeric column for analysis.


```python
import pandas as pd

# Load the dataset
file_path = "/Users/sjoerdv/Documents/PERSOONLIJK/Portfolio/Data 27 jul/all_rent_data_cleaned.csv"
df = pd.read_csv(file_path)

# Extract number of bedrooms (e.g. "2 bed." → 2.0)
df['bedrooms_clean'] = df['bedrooms'].str.extract(r'(\d+)').astype(float)

# Preview cleaned results
print("Bedrooms cleaned preview:")
print(df[['bedrooms', 'bedrooms_clean']].head(10))

# Show descriptive statistics
print("Descriptive statistics for bedrooms:")
print(df['bedrooms_clean'].describe())

# Save the updated file
output_path = "/Users/sjoerdv/Documents/PERSOONLIJK/Portfolio/Data 27 jul/all_rent_data_cleaned_v2.csv"
df.to_csv(output_path, index=False)

print(f"File saved with cleaned bedrooms column: {output_path}")
```

</details> <details> <summary>🏠 Step 1.4 – GeoData</summary>

**Step overview**

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


## Part 2 – Data Analysis 

<details>
  <summary>🏠 Step 2.1 - Descriptive Statistics </summary

**Step overview**

In this step, I calculated key rental price statistics per district including the number of listings, average, minimum, maximum rents, and price variation. Subsequently, I ranked the districts by average rent to get a broad overview of the market.

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

Let us take a look at the table:

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

**What I learned**

_Nou Barris_ and _Sant Andreu_ had very few listings compared to other districts. Since exploratory analysis usually needs a sample size of at least 30 for meaningful results, I excluded them from later steps to keep the insights reliable.


</details> <details>
  <summary>🏠 Step 2.2 – Which districts give the best value for money (€/m²)? </summary

**Step overview**

In this step, I examined how rental prices per square metre vary across districts (excluding _Sant Andreu_ and _Nou Barris_). I counted listings and calculated average, minimum, maximum, and standard deviation of price per m², then ranked districts from cheapest to most expensive.

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


| district            | number_of_listings | avg_price_per_m2 | min_price_per_m2 | max_price_per_m2 | stddev_price_per_m2 |
|---------------------|--------------------|------------------|------------------|------------------|---------------------|
| Horta Guinardó      |                 88 |            20.19 |             12.5 |            35.83 |                5.68 |
| Les Corts           |                 66 |            22.92 |            13.33 |               50 |                6.06 |
| Gràcia              |                199 |            25.04 |            13.33 |               40 |                5.28 |
| Sants-Montjuïc      |                200 |            25.55 |            14.09 |               54 |                6.07 |
| Sant Martí          |                134 |            25.84 |             7.14 |            58.33 |                6.56 |
| Sarrià-Sant Gervasi |                200 |            26.06 |             11.5 |            56.67 |                7.05 |
| Eixample            |                434 |            26.15 |            10.83 |            61.67 |                6.84 |
| Ciutat Vella        |                846 |            27.09 |            11.24 |               95 |                8.64 |

<img src="https://github.com/sjoerd-verhagen/barcelona-rental-analysis/blob/main/Value-for-money-v15aug.png" alt="Which districts give the best value for money" width="800">

**What I learned:**

The overall average price per m² is **€24,86**. Only two districts fall below this: _Horta Guinardó_ and _Les Corts_, which have the lowest average prices per square metre and similar price variation to other districts. Their lower averages without greater volatility suggest they offer the best value for money, although they also have the fewest listings with 88 and 66 respectively.

</details>
<details>
  <summary>🏠 Step 2.3 – Where is the largest rental supply? </summary

**Step overview**

In this step, I looked at how rental prices vary across districts, excluding _Sant Andreu_ and _Nou Barris_. I counted listings, measured price variance and standard deviation, and then ranked the districts by how volatile their rental prices are.


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

**What I learned**

The largest rental supply is in _Ciutat Vella_, followed by _Eixample_, which are also the two districts with the highest average price per m² (see step 2.2). Among districts with plenty of listings, _Sarrià-Sant Gervasi_ and _Sants-Montjuïc_ have the lowest price variability, shown by their relatively small standard deviations, despite higher average prices than _Les Corts_ and _Horta Guinardó_ (as seen in section 2.2). _Sant Martí_ shows the greatest price variance, even though it has the fifth lowest average price per m², suggesting more fluctuation in rental costs there. 

</details>

<details>
  <summary>🏠 Step 2.4 – What are the average sizes and rents by district? </summary

**Step overview**

To get a clearer view of what each district offers, I calculated the average apartment size and rental price, as well as the minimum, maximum and standard deviation of rents. This shows not only the typical flat you might find, but also the range and variability of rental options across Barcelona’s districts.

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

<img src="https://github.com/sjoerd-verhagen/barcelona-rental-analysis/blob/main/rent-vs-size" alt="Average Rent vs Apartment Size by District" width="800">

**What I learned**

_Horta Guinardó_ offers the best deal, combining the lowest average rent with the second-largest apartment sizes, although it has the second fewest listings at 88. _Ciutat Vella_ also has low rents but much smaller apartments, making _Horta Guinardó_ the better value for both space and price. _Sants-Montjuïc_ ranks third for lowest average rent yet has the second smallest apartments. The graph shows a general pattern where average apartment size increases with average rent, though _Les Corts_ stands out by having the largest average size despite higher rents.


</details> <details>
  <summary>🏠 Step 2.5 – Comparing 1- and 2-bedroom apartments </summary

**Step overview**

To better understand what each district offers, I compared one and two bedroom apartments by looking at their average size, rent, and the variation in rents. This not only shows the typical flats available and the range of rental options, but also answers a key question for our move: how much extra we would need to pay for a second bedroom so friends and family can stay over.

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

<img src="https://github.com/sjoerd-verhagen/barcelona-rental-analysis/blob/main/bubble chart.png" alt="Which districts give the best value for money" width="600">

**What I learned:**

This bubble chart compares the average rent for one and two bedroom flats in each district. Bubble size shows the number of listings, and darker colour indicates a bigger difference in price between the two.

It shows that _Horta Guinardó_ offers the best value, with the lowest average rent and the second-largest apartments. In contrast, _Ciutat Vella_ has similarly low rents but much smaller flats, making it less appealing if space is important. _Les Corts_ and _Sants-Montjuïc_ have the smallest price differences between one and two bedroom flats (4.11% and 7.63% respectively). _Sants-Montjuïc_ stands out here, combining a small price gap with reasonable apartment sizes.

</details> <details>
  <summary>🏠 Step 2.6 – Conclusion </summary

Two districts, **_Nou Barris_** and **_Sant Andreu_**, had too few listings to analyse reliably, so I excluded them from the main results. Across the remaining areas, the average rent is **€24.86** per m². Only _**Horta Guinardó**_ and _**Les Corts**_ fall below this, offering the lowest prices per m² without higher volatility, although both have relatively few listings.

_**Ciutat Vella**_ and _**Eixample**_ have the largest rental supply, but they also have the highest prices per m². Among districts with plenty of listings,_ **Sarrià-Sant Gervasi**_ and _**Sants-Montjuïc**_ show the most stable prices. _**Sant Martí**_ has the fifth lowest price per m² but the highest price variation, making its costs less predictable.

_**Horta Guinardó**_ offers the best overall value, with the lowest average rent and the second-largest apartments, though supply is limited. _**Sants-Montjuïc**_ combines a low rent premium for two bedrooms with decent apartment sizes and good availability, while _**Sarrià-Sant Gervasi**_ offers slightly larger flats for only a small extra cost. Based on my goal of finding a spacious, affordable flat with a second bedroom for guests, _**Sants-Montjuïc**_ and _**Sarrià-Sant Gervasi**_ are the most promising districts to focus on.

</details>

## What I learned (and Challenges I faced)

- Scraping data from Idealista proved challenging due to the site’s protection measures and inconsistencies in listing formats. I addressed this by breaking the scraping process into smaller batches and cleaning address data post-scrape. One notable issue was floor numbers occasionally being misinterpreted as bedroom counts, leading to anomalies such as listings showing eight bedrooms. Regular checks of descriptive statistics helped identify and correct these errors, reinforcing the importance of continuous data validation throughout a project.
- In hindsight, I also collected more variables than were necessary for my analysis. This highlighted the value of clearly defining research questions and prioritising relevant variables at the outset. In future projects, I would refine the data collection scope earlier to focus on what is essential for answering the core questions.

