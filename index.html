# 🏙️ Barcelona Rental Market Analysis

**Author:** Sjoerd Verhagen  
**Tools used:** SQL • Python • Tableau  
**Live Tableau dashboard:** [View Here](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)

---

## 🎯 Project Goal

Understand the rental market in Barcelona before relocating — specifically:
- Where you get the best €/m²
- Where supply is highest
- What average sizes and rents are in each district

---

## 🔍 Key Questions

- 💸 Which districts offer the best **value for money** (€/m²)?
- 🏘️ Where is **rental supply** most concentrated?
- 📏 What are the **average sizes and rents** by district?
- 🛏️ What’s the **price difference** between one-bedroom and two-bedroom flats?
- 📍 Which non-central districts (like Gràcia, Sants, or Poblenou) offer a good **balance of price and space**?

---

## 🛠️ Tools Used

- 🧹 **Python** – For cleaning and preparing data  
- 📊 **SQL** – For grouping, aggregating and filtering  
- 🌍 **Tableau** – For interactive visualizations  
- 🕸️ **Web Scraping** – Idealista rental data

---

## 📈 Tableau Dashboard

🔗 [View the interactive dashboard](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)

Screenshot preview below:

![Tableau Preview](images/tableau-preview.png) <!-- optional screenshot -->

---

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
