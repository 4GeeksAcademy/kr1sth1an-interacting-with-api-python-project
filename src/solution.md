# Solution: Interacting with the World Bank v2 API

This is one valid solution aligned with `INSTRUCTIONS.md`.  
In this example we use:

- Countries: `ARG`, `BRA`, `CHL`, `COL`, `MEX`
- Indicators:
  - `NY.GDP.PCAP.CD` (GDP per capita)
  - `SP.DYN.LE00.IN` (Life expectancy)
  - `EN.ATM.CO2E.PC` (CO2 per capita)

You can run this flow in a notebook such as `src/world_bank_analysis.ipynb`.

## 1. Imports and setup

```python
import requests
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from sqlalchemy import create_engine
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

sns.set_theme(style="whitegrid")
BASE_URL = "https://api.worldbank.org/v2"
TIMEOUT = 60

# Session with retry strategy to reduce transient timeout failures
session = requests.Session()
retry = Retry(
    total=5,
    connect=5,
    read=5,
    backoff_factor=1,
    status_forcelist=[429, 500, 502, 503, 504],
    allowed_methods=["GET"],
)
session.mount("https://", HTTPAdapter(max_retries=retry))
session.mount("http://", HTTPAdapter(max_retries=retry))
```

## 2. Explore API (example request)

```python
url = f"{BASE_URL}/country"
params = {
    "format": "json",
    "per_page": 50,
    "page": 1
}

response = session.get(url, params=params, timeout=TIMEOUT)
response.raise_for_status()
payload = response.json()

print("Pagination metadata:", payload[0])
print("First country record:", payload[1][0])
```

## 3. Download data with pagination

```python
countries = ["ARG", "BRA", "CHL", "COL", "MEX"]
indicators = {
    "NY.GDP.PCAP.CD": "gdp_per_capita",
    "SP.DYN.LE00.IN": "life_expectancy",
    "EN.ATM.CO2E.PC": "co2_per_capita",
}

def fetch_indicator_data(country_codes, indicator_id, start_year=2010, end_year=2024):
    country_path = ";".join(country_codes)
    endpoint = f"{BASE_URL}/country/{country_path}/indicator/{indicator_id}"
    page = 1
    all_rows = []

    while True:
        params = {
            "format": "json",
            "date": f"{start_year}:{end_year}",
            "per_page": 50,
            "page": page
        }
        response = session.get(endpoint, params=params, timeout=TIMEOUT)
        response.raise_for_status()
        payload = response.json()

        if not isinstance(payload, list) or len(payload) == 0:
            raise ValueError(f"Unexpected API response for {indicator_id}: {payload}")

        metadata = payload[0]
        rows = payload[1] if len(payload) > 1 and payload[1] is not None else []
        all_rows.extend(rows)

        total_pages = int(metadata.get("pages", 1))
        if page >= total_pages:
            break
        page += 1

    return all_rows
```

## 4. Build one DataFrame per indicator

```python
tables = {}

for indicator_id, table_name in indicators.items():
    raw_rows = fetch_indicator_data(countries, indicator_id, 2010, 2024)

    records = []
    for row in raw_rows:
        records.append({
            "country": row["country"]["value"],
            "year": row["date"],
            "value": row["value"],
        })

    df = pd.DataFrame(records)
    df["year"] = pd.to_numeric(df["year"], errors="coerce").astype("Int64")
    df["value"] = pd.to_numeric(df["value"], errors="coerce")
    df = df.dropna(subset=["value", "year"]).copy()
    df["year"] = df["year"].astype(int)
    df = df.sort_values(["country", "year"]).reset_index(drop=True)

    tables[table_name] = df

tables["gdp_per_capita"].head()
```

## 5. Visualizations (2 examples)

### Example 1: Line chart (GDP per capita by country)

```python
plt.figure(figsize=(12, 6))
sns.lineplot(data=tables["gdp_per_capita"], x="year", y="value", hue="country", marker="o")
plt.title("GDP per capita (2010-2024)")
plt.xlabel("Year")
plt.ylabel("Current USD")
plt.legend(title="Country", bbox_to_anchor=(1.02, 1), loc="upper left")
plt.tight_layout()
plt.show()
```

### Example 2: Scatter plot (GDP per capita vs CO2 per capita)

```python
latest_year = 2022

gdp_latest = tables["gdp_per_capita"].query("year == @latest_year").rename(columns={"value": "gdp_per_capita"})
co2_latest = tables["co2_per_capita"].query("year == @latest_year").rename(columns={"value": "co2_per_capita"})

merged = pd.merge(
    gdp_latest[["country", "gdp_per_capita"]],
    co2_latest[["country", "co2_per_capita"]],
    on="country",
    how="inner"
)

plt.figure(figsize=(8, 6))
sns.scatterplot(data=merged, x="gdp_per_capita", y="co2_per_capita", hue="country", s=120)
plt.title(f"GDP per capita vs CO2 per capita ({latest_year})")
plt.xlabel("GDP per capita (current USD)")
plt.ylabel("CO2 per capita (metric tons)")
plt.tight_layout()
plt.show()
```

## 6. Save to SQL (one table per indicator)

```python
engine = create_engine("sqlite:///world_bank_analysis.db")

for table_name, df in tables.items():
    sql_table_name = f"indicator_{table_name}"
    df.to_sql(sql_table_name, engine, if_exists="replace", index=False)

# Validation example
pd.read_sql("SELECT * FROM indicator_gdp_per_capita LIMIT 5", engine)
```

## 7. Suggested markdown conclusions (example)

- GDP per capita shows different growth paths across countries, with clear gaps in income levels.
- Countries with higher GDP per capita tend to have higher CO2 emissions per capita in the selected year, although the relationship is not perfectly linear.
