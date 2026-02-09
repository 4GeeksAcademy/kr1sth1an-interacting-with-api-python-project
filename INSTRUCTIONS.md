# Interacting with the World Bank v2 API

In this project, you will build a complete data workflow: API consumption, DataFrame transformation, visual analysis, and SQL database loading.

## Exercise Context

You will work with the public World Bank v2 API (no authentication required). The goal is to analyze the socioeconomic and environmental evolution of 5 countries chosen by you between 2010 and 2024.

### Dataset Selection

1. Choose 5 countries (ISO3) that interest you.
2. Choose the indicators you want to analyze.
3. Suggested indicators (optional):

- `SP.POP.TOTL`: Total population
- `NY.GDP.PCAP.CD`: GDP per capita (current USD)
- `EN.ATM.CO2E.PC`: CO2 emissions per capita (metric tons)
- `SP.DYN.LE00.IN`: Life expectancy at birth (years)

Base API URL: `https://api.worldbank.org/v2`

## Technical Requirements

You must work in a `.ipynb` file and use:

- `requests`
- `pandas`
- `matplotlib` and/or `seaborn`
- `sqlalchemy`

## Step 1: Prepare the environment

Install dependencies:

```bash
pip install requests pandas matplotlib seaborn sqlalchemy
```

Create a notebook, for example: `src/world_bank_analysis.ipynb`.

## Step 2: Explore the API

Review these reference endpoints:

- Countries: `https://api.worldbank.org/v2/country`
- Indicators: `https://api.worldbank.org/v2/indicator`

Check the response structure. The API paginates results (usually up to `per_page=50`), so you should plan a strategy to loop through pages and store all the information you need.

Example request in Python (template):

```python
import requests

url = "https://api.worldbank.org/v2/country"
params = {
    "format": "json",
    "per_page": 50,
    "page": 1
}

response = requests.get(url, params=params, timeout=30)
response.raise_for_status()
payload = response.json()

# payload[0] -> pagination metadata
# payload[1] -> current page data
print("Metadata:", payload[0])
print("First item:", payload[1][0])
```

*IMPORTANT:* The code above is only a guide. In the link below you can find all the information needed to make API calls:

https://datahelpdesk.worldbank.org/knowledgebase/articles/898581

## Step 3: Download data

Download 2010-2024 time series for the countries and indicators you selected.

Goal:

- Consume the API for multiple countries and indicators
- Handle pagination when needed
- Store responses in a temporary structure (list of dictionaries)

## Step 4: Transform responses into DataFrames

Create one table (DataFrame) per indicator to make country comparisons easier.

Suggested columns for each table:

- `country`
- `year`
- `value`

Minimum cleaning:

- Remove rows with null `value` when needed
- Convert `year` to integer
- Convert `value` to numeric

## Step 5: Analysis and visualizations

Create at least 2 charts and explain findings in Markdown cells.

Examples:

1. Line chart: evolution of one indicator by country (2010-2024)
2. Scatter plot: relationship between two indicators for a recent year

## Step 6: Load results into an SQL database

Use SQLite with SQLAlchemy to persist data:

- Database: `world_bank_analysis.db`
- Teaching recommendation: one table per indicator (example: `indicator_gdp_per_capita`, `indicator_life_expectancy`, etc.)

Recommended flow:

1. Create an engine with SQLAlchemy
2. Save each DataFrame with `to_sql(..., if_exists="replace")`
3. Read a sample with `pd.read_sql()` to validate the load

## Closing

You already have everything you need to start!
Take your time exploring the API documentation and understanding the response structure.
If you have any questions during the process, contact your mentors.
