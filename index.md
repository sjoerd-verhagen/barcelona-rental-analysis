# 🏙️ Barcelona Rental Market Analysis

**Author:** Sjoerd Verhagen  
**Tools used:** SQL • Python • Tableau  
**Live Tableau dashboard:** [View Here](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)

---

## 🎯 Project Goal

My girlfriend and I will move to Barcelona in September 2025. To use my new skills in python, SQL and Tableau I thought of a project that is something I really want to figure out and also that will display what I learnt in my courses. 

To prepare for our move, I analysed the rental market using data from Idealista, the city’s most popular housing platform. With a student budget, I focused on furnished flats under €2,000 a month to see what is realistic and where to start our search. Through the analysis i hope to narrow down the choices of neighbourhoods, to get a feel for the city and to see whether there are certain districts better suited for our choice.

I aimed to answer:
- Which districts give us the best value for money (€/m²)?
- Which districts have the largest rental supply? 
- What are the average sizes and rents by district?


---

## 🔍 Key Questions

In order to dig a little deeper into the real interesting questions:
As I want to have a place where friends can stay over  
- How much more do two-bedroom flats cost compared to one-bedroom flats?

As I want a place that also has have some quiet next to the busy city life 
- Which districts just outside the centre (like Gràcia, Poble-Sec, Poblenou, Sants, and Sant Antoni) offer good balance between price, space?


---

## 🛠️ Tools Used

- Open Web Scraper for collecting rental data
- Python for cleaning and preparing the data
- Open Geo Code for mapping coordinates
- SQL for queries and aggregations
- Tableau for creating clear visual insights

---




## 📈 Tableau Dashboard

🔗 [View the interactive dashboard](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)

Screenshot preview below:

![Tableau Preview](images/tableau-preview.png) <!-- optional screenshot -->

---

<details>
  <summary>🏠 Barcelona Rental Market Analysis</summary>

  Summary text...

  <details>
    <summary>📈 Tableau Dashboard</summary>

    <iframe src="..." width="100%" height="400"></iframe>

  </details>

## 🐍 Python Code (Data Cleaning)

```python
import pandas as pd

# Load data
df = pd.read_csv("idealista_raw.csv")

# Clean price and size
df["price"] = df["price"].str.replace("€", "").str.replace(",", "").astype(float)
df["size"] = df["size"].str.replace("m²", "").astype(float)

# Calculate €/m²
df["price_per_m2"] = df["price"] / df["size"]

# Filter only furnished listings under €2000
df_filtered = df[(df["furnished"] == True) & (df["price"] < 2000)]

# Save for SQL analysis
df_filtered.to_csv("cleaned_rentals.csv", index=False)
