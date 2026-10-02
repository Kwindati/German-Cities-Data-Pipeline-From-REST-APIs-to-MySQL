# 🇩🇪 German Cities Data Pipeline: From REST APIs to MySQL

A small end-to-end data engineering project that pulls city and weather data for Germany's five largest cities from public REST APIs, cleans it with **pandas**, and loads it into a relational **MySQL** database with proper primary/foreign keys.

**Cities covered:** Berlin, Hamburg, Frankfurt, Munich, Cologne

---

## What this project does

1. **Fetches city data** (coordinates, country, population) from the [API Ninjas City API](https://api-ninjas.com/api/city).
2. **Cleans and splits** the response into two tables: `city_list` and `fact_list`.
3. **Loads them into MySQL**, letting the database auto-generate `city_id`, then maps those IDs back onto the facts table to create the relationship.
4. **Reads the coordinates back from SQL** and calls the [OpenWeatherMap 5-day / 3-hour forecast API](https://openweathermap.org/forecast5) for every city.
5. **Parses the nested JSON** (temperature, forecast, wind speed, precipitation probability, timestamps) into a flat DataFrame.
6. **Appends the forecast to MySQL** in the `cities_forecast` table, with a `data_retrieved_at` timestamp for each load.

```
API Ninjas ──► pandas ──► MySQL (city_list, fact_list)
                                   │
                          latitude / longitude
                                   ▼
OpenWeatherMap ──► pandas ──► MySQL (cities_forecast)
```

---

## Repository contents

| File | Description |
|---|---|
| `Extracting_the_latitude_and_longitude_information_and_APIs.ipynb` | The Jupyter notebook with the full workflow (API calls → cleaning → SQL load) |
| `county_city_info_api_dump.sql` | MySQL dump of the `county_city_info_api` database (schema and data) |

> The database also contains the `flight_arrivals` and `flight_iata` tables. The code that populates them will be added to another repo separately.

---

## Database schema

Database name: `county_city_info_api`

| Table | Purpose | Key columns |
|---|---|---|
| `city_list` | One row per city | `city_id` (PK, auto-increment), `name` |
| `fact_list` | City facts | `city_id` (FK), `latitude`, `longitude`, `country`, `population` |
| `cities_forecast` | Weather forecast per city, per 3-hour slot | `city_id` (FK), `forecast_time`, `temperature`, `forecast`, `rain_in_last_3h`, `wind_speed`, `data_retrieved_at` |
| `flight_arrivals` | Scheduled flight arrivals per city | `arrival_id` (PK), `city_id` (FK), `arrival_iata`, `departure_airport_iata`, `scheduled_arrival_time`, `flight_number` |
| `flight_iata` | IATA airport reference data | *(populated by the separate flight script)* |

> **Note:** the `rain_in_last_3h` column is filled from the API's `pop` field, which is the **probability of precipitation** (0 to 1), not a rainfall amount.

---

## Getting started

### 1. Prerequisites

- Python 3.9+
- MySQL Server 8.0 (and optionally MySQL Workbench)
- Free API keys from:
  - [API Ninjas](https://api-ninjas.com/)
  - [OpenWeatherMap](https://openweathermap.org/api)

### 2. Install dependencies

```bash
pip install pandas requests sqlalchemy pymysql jupyter
```

### 3. Restore the database

```bash
mysql -u root -p < county_city_info_api_dump.sql
```

Or in MySQL Workbench: **Server → Data Import → Import from Self-Contained File**.

### 4. Add your credentials

For security, no real keys or passwords are included in this repo. Before running the notebook, replace the placeholders with your own values:

| Placeholder | Replace with |
|---|---|
| `YOUR_API_NINJAS_KEY` | Your API Ninjas key |
| `YOUR_API_KEY` | Your OpenWeatherMap key |
| `YOUR_DB_PASSWORD` | Your local MySQL password |

> 💡 **Tip:** a safer approach is to keep secrets in environment variables (e.g. `os.getenv("OPENWEATHER_API_KEY")`) and never commit them.

### 5. Run the notebook

```bash
jupyter notebook
```

Open the `.ipynb` file and run the cells from top to bottom.

---

## Tech stack

- **Python**: pandas, requests, SQLAlchemy, PyMySQL
- **MySQL 8.0**
- **APIs**: API Ninjas (City), OpenWeatherMap (5-day forecast)
- **Jupyter Notebook**

---

## Known notes

- The final "wrap it into a function" section (`create_weather_dataframe`) was left as-is, including a captured `KeyError`. It happens when every API request is skipped (non-200 response, e.g. an invalid key or rate limit), so the resulting DataFrame has no columns. Check your API key and the status codes if you see it.
- Free API tiers are rate-limited, so avoid re-running the loops excessively.

---

## Author

**<Your Name>**
[LinkedIn](https://www.linkedin.com/in/tovhowani-kwinda-dr-rer-nat) · [GitHub](https://github.com/Kwindati)

---

## License

This project is for educational and portfolio purposes. Data belongs to the respective API providers; please follow their terms of use.
