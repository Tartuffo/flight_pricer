# Flight Price Tracker

This directory contains scripts for tracking flight prices using the Amadeus API, with automatic data collection and analysis tools.

## Setup

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Create your secrets files:
   - `secrets-test.yaml` for test environment
   - `secrets-prod.yaml` for production environment
   
   Format:
   ```yaml
   api_key: YOUR_API_KEY
   api_secret: YOUR_API_SECRET
   ```

3. Configure your flight routes in `config.yaml`

## Configuration

The `config.yaml` file has been restructured to include global settings and multiple itineraries:

```yaml
metadata:
  max_duration_hours: 15    # Global default max flight duration

itineraries:
  - from: JFK
    to: SFO
    target_date: 2025-12-20
    currency: USD
  - from: JFK
    to: LAX
    target_date: 2025-12-21
    currency: USD
    max_duration_hours: 8   # Override global setting for this route
```

### Configuration Options

- **Global Settings (metadata)**:
  - `max_duration_hours`: Default maximum flight duration filter (client-side filtering)

- **Per-Route Settings (itineraries)**:
  - `from`: Origin airport code (IATA)
  - `to`: Destination airport code (IATA)
  - `target_date`: Flight date (YYYY-MM-DD)
  - `currency`: Price currency (USD, EUR, etc.)
  - `max_duration_hours`: (optional) Override global duration limit for this route

## Scripts

### track_flights

The main flight tracking script that queries the Amadeus API and saves results.

**Usage:**
```bash
./track_flights <environment> <output_directory>
```

**Parameters:**
- `environment`: `test` or `prod`
- `output_directory`: Base directory for output (will be suffixed with `-<environment>`)

**Examples:**
```bash
# Run with test environment, save to /tmp/flights-test/
./track_flights test /tmp/flights

# Run with prod environment, save to /var/data/flights-prod/
./track_flights prod /var/data/flights
```

**Output:**
- Creates directory `/path/to/output-<environment>/`
- Saves JSON data as `YYYY-MM-DD.json`
- Shows flight data in console table format
- Applies duration filtering based on configuration

### dump_flights

Utility script to read and display saved flight data in table format.

**Usage:**
```bash
./dump_flights <path> [options]
```

**Parameters:**
- `path`: JSON file or directory containing JSON files

**Options:**
- `--sort-by <field>`: Sort by `date`, `price`, `duration`, or `departure` (default: date)
- `--route <route>`: Filter by route (e.g., `JFK-SFO`)

**Examples:**
```bash
# Display all flights from directory
./dump_flights /tmp/flights-test

# Display specific JSON file
./dump_flights /tmp/flights-test/2025-08-29.json

# Sort by price (cheapest first)
./dump_flights /tmp/flights-test --sort-by price

# Filter specific route
./dump_flights /tmp/flights-test --route JFK-SFO

# Combine filtering and sorting
./dump_flights /tmp/flights-test --route JFK-SFO --sort-by price
```

**Output Format:**
```
Run Date    Route    Airline  Flight          Departure  Arrival  Next Day  Duration  Stops  Price       Fare Code
2025-08-29  JFK-SFO  HA       HA4928          20:29      23:57    No        6h 28m    0      188.33 USD  LH2OASBN
2025-08-29  JFK-SFO  AS       AS17 + AS1082   18:30      09:11    Yes       17h 41m   1      246.63 USD  QH2OAVBN
```

## Features

- **Duration Filtering**: Client-side filtering of flights by maximum duration
- **Environment Support**: Separate test/prod configurations
- **JSON Output**: Structured data storage for analysis
- **Multiple Routes**: Support for tracking multiple flight routes
- **Next-Day Detection**: Automatic detection of overnight flights
- **Connection Handling**: Shows complete trips with layovers as single entries
- **Historical Analysis**: Tools to review and compare historical flight data

## Automation

### Cron Setup

To run daily at 9 AM:
```bash
0 9 * * * cd /path/to/flights && ./track_flights prod /var/data/flights
```

To run multiple times per day:
```bash
# Every 6 hours
0 */6 * * * cd /path/to/flights && ./track_flights prod /var/data/flights

# Twice daily (9 AM and 6 PM)
0 9,18 * * * cd /path/to/flights && ./track_flights prod /var/data/flights
```

## Data Analysis

Use `dump_flights` to analyze trends:

```bash
# Find cheapest flights across all dates
./dump_flights /var/data/flights-prod --sort-by price | head -10

# Compare prices for specific route over time
./dump_flights /var/data/flights-prod --route JFK-SFO --sort-by date

# Find shortest flight times
./dump_flights /var/data/flights-prod --sort-by duration | head -5
```