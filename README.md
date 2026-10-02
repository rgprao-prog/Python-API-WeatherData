# UK Weather Data & SQLite Database

A Python project that retrieves historical weather data for a UK city using the **Open-Meteo API**, stores the results in a **SQLite database**, and uses SQL queries to analyse the weather data.

The project was built to practise working with:

* Python
* REST APIs
* JSON data
* API parameters
* SQLite databases
* SQL queries
* Loops and lists
* Basic data analysis

---

## Features

The program allows the user to enter a UK city and then:

1. Converts the city name into latitude and longitude using the Open-Meteo Geocoding API.
2. Uses those coordinates to request historical weather data.
3. Retrieves:

   * Daily maximum temperature
   * Daily minimum temperature
   * Daily precipitation
4. Stores the weather data in a SQLite database.
5. Prevents duplicate city/date records using a `UNIQUE` constraint.
6. Uses SQL queries to find:

   * Highest temperature
   * Lowest temperature
   * Total rainfall
   * Number of rainy days
   * The dates and rainfall amounts for rainy days

---

## How It Works

The project uses **two API requests**.

### 1. Geocoding API

The user enters a city:

```text
enter a UK city: York
```

The program sends the city name to the Open-Meteo Geocoding API:

```python
geo_url = "https://geocoding-api.open-meteo.com/v1/search"

geo_params = {
    "name": city,
    "count": 1
}
```

The API returns information about the location, including its latitude and longitude.

These values are extracted from the JSON response:

```python
longitude = location_data["results"][0]["longitude"]
latitude = location_data["results"][0]["latitude"]
```

The coordinates are then used in the second API request.

---

### 2. Historical Weather API

The latitude and longitude are passed to the Open-Meteo Historical Weather API.

The program requests weather data between:

```text
2026-01-01
```

and

```text
2026-01-15
```

The requested variables are:

```python
"daily": "temperature_2m_max,temperature_2m_min,precipitation_sum"
```

This provides daily:

* Maximum temperature
* Minimum temperature
* Total precipitation

The timezone is set to:

```python
"timezone": "Europe/London"
```

---

## Database

The weather data is stored in a SQLite database called:

```text
weather.db
```

The database contains a table called:

```text
UK_weather
```

### Table structure

| Column            | Type     | Description                      |
| ----------------- | -------- | -------------------------------- |
| `id`              | INTEGER  | Unique ID for each record        |
| `city`            | TEXT     | Name of the city                 |
| `date`            | DATETIME | Date of the weather observation  |
| `max_temperature` | DECIMAL  | Maximum temperature for that day |
| `min_temperature` | DECIMAL  | Minimum temperature for that day |
| `rainfall`        | DECIMAL  | Total precipitation for that day |

The `id` column is an auto-incrementing primary key.

---

## Preventing Duplicate Data

The table uses a composite `UNIQUE` constraint:

```sql
UNIQUE(city, date)
```

This means the same city cannot have two records for the same date.

The data is inserted using:

```sql
INSERT OR IGNORE
```

Therefore, if a city/date combination already exists, SQLite ignores the duplicate instead of inserting another row.

> **Note:** During development, the current code uses `DROP TABLE IF EXISTS`, which deletes the existing table every time the program runs. This is useful while testing, but it means previously stored data will not persist between runs.

---

## SQL Analysis

Once the weather data has been inserted, SQL is used to analyse it.

### Highest Temperature

```sql
SELECT MAX(max_temperature)
FROM UK_weather
```

This finds the highest maximum temperature in the stored dataset.

---

### Lowest Temperature

```sql
SELECT MIN(min_temperature)
FROM UK_weather
```

This finds the lowest minimum temperature.

---

### Total Rainfall

```sql
SELECT SUM(rainfall)
FROM UK_weather
```

This calculates the total precipitation across all stored days.

---

### Finding Rainy Days

The program identifies days where rainfall was greater than zero:

```sql
SELECT date, rainfall
FROM UK_weather
WHERE rainfall > 0
```

The resulting rows are then counted:

```python
rainy_days = cursor.fetchall()
print("total rainy days:", len(rainy_days))
```

The program also prints each rainy date and its rainfall amount.

---

## Example Output

A run of the program might produce output similar to:

```text
enter a UK city: York

Highest temperature: 10.7
Lowest temperature: -1.8
total rainfall: 24.6
total rainy days: 7

here are the rainy days:
('2026-01-02', 2.4)
('2026-01-04', 5.1)
('2026-01-06', 0.8)
...
```

The exact results depend on the city and weather data returned by the API.

---

## Requirements

You need:

* Python 3
* `requests`

Python's `sqlite3` and `json` modules are included in the standard library, so they do not need to be installed separately.

Install `requests` with:

```bash
pip install requests
```

---

## Running the Project

Clone the repository:

```bash
git clone <your-repository-url>
```

Move into the project directory:

```bash
cd <project-directory>
```

Install the dependency:

```bash
pip install requests
```

Run the Python program:

```bash
python weather.py
```

Enter a UK city when prompted:

```text
enter a UK city: Manchester
```

The program will retrieve the weather data, store it in `weather.db`, and run the SQL analysis.

---

## Project Structure

A simple version of the repository could look like:

```text
weather-project/
│
├── weather.py
├── weather.db
├── README.md
└── .gitignore
```

### `.gitignore`

It is recommended to add the SQLite database to `.gitignore` if you don't want the generated database committed to Git:

```gitignore
weather.db
__pycache__/
*.pyc
```

The database is generated automatically when the program runs.

---

## APIs Used

This project uses [Open-Meteo](https://open-meteo.com/) for both geocoding and historical weather data.

### Geocoding API

Used to convert the user's city name into geographical coordinates.

```text
https://geocoding-api.open-meteo.com/v1/search
```

### Historical Weather API

Used to retrieve historical daily weather data.

```text
https://archive-api.open-meteo.com/v1/archive
```

---

## What I Learned

This project helped me practise the complete process of taking data from an external API and putting it into a relational database.

The main workflow is:

```text
User Input
    ↓
Geocoding API
    ↓
Latitude + Longitude
    ↓
Historical Weather API
    ↓
JSON Response
    ↓
Extract Weather Data
    ↓
SQLite Database
    ↓
SQL Queries
    ↓
Weather Analysis
```

It also gave me practical experience with:

* Making HTTP GET requests with `requests`
* Passing query parameters to an API
* Reading JSON responses
* Extracting nested JSON values
* Working with SQLite
* Creating database tables
* Inserting data using parameterised SQL
* Using SQL aggregate functions such as `MAX()`, `MIN()` and `SUM()`
* Filtering data with `WHERE`
* Handling duplicate records with a `UNIQUE` constraint and `INSERT OR IGNORE`

---

## Possible Future Improvements

Some improvements I could make to the project include:

* Allowing the user to choose the start and end dates.
* Allowing multiple cities to be stored without deleting existing data.
* Removing `DROP TABLE IF EXISTS` once testing is complete.
* Checking whether the city exists before accessing `results[0]`.
* Adding better error handling for API/network failures.
* Storing the country and coordinates in the database.
* Adding SQL queries for average temperature.
* Finding the wettest day.
* Finding the hottest and coldest dates, rather than only the temperatures.
* Creating graphs to visualise temperature and rainfall.
* Separating the API, database, and analysis code into separate functions.
* Adding automated tests.
