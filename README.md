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

3. Configure your flight routes:
   - `config-test.yaml` for test environment routes
   - `config-prod.yaml` for production environment routes
   - Or use `--config` option to specify a custom config file

## Configuration

The configuration system supports environment-specific files with global settings and multiple itineraries:

```yaml
metadata:
  max_duration_hours: 15    # Global default max flight duration

itineraries:
  - from: JFK
    to: SFO
    departure_dates: 2025-12-18,2025-12-20  # Multiple specific dates
    currency: USD
  - from: SFO
    to: JFK
    departure_dates: 2026-01-01-3           # Date range (Jan 1-3)
    currency: USD
    max_duration_hours: 8   # Override global setting for this route
```

### Environment-Specific Configuration

By default, the system uses:
- `config-test.yaml` for test environment
- `config-prod.yaml` for production environment

You can override this with the `--config` option to use any config file.

### Configuration Options

- **Global Settings (metadata)**:
  - `max_duration_hours`: Default maximum flight duration filter (client-side filtering)

- **Per-Route Settings (itineraries)**:
  - `from`: Origin airport code (IATA)
  - `to`: Destination airport code (IATA)
  - `departure_dates`: Flight dates - supports multiple formats:
    - **Individual dates**: `2025-12-18,2025-12-20` (comma-separated)
    - **Date ranges**: `2026-01-01-3` (Jan 1, 2, 3, 2026)
    - **Mixed**: `2025-11-28-30,2025-12-15` (Nov 28-30 and Dec 15)
  - `currency`: Price currency (USD, EUR, etc.)
  - `max_duration_hours`: (optional) Override global duration limit for this route

### Date Range Format

The `departure_dates` field supports flexible date specifications:
- **Single dates**: `2025-12-20`
- **Multiple dates**: `2025-12-18,2025-12-20,2025-12-22`
- **Date ranges**: `2025-12-01-05` (expands to Dec 1, 2, 3, 4, 5)
- **Short ranges**: `2026-01-01-3` (expands to Jan 1, 2, 3)
- **Combined**: `2025-12-18-20,2025-12-25` (Dec 18-20 and Dec 25)

## Scripts

### track_flights

The main flight tracking script that queries the Amadeus API and saves results.

**Usage:**
```bash
./track_flights <environment> <output_directory> [--config CONFIG_FILE]
```

**Parameters:**
- `environment`: `test` or `prod`
- `output_directory`: Base directory for output (will be suffixed with `-<environment>`)
- `--config`: (optional) Config file to use (default: `config-<environment>.yaml`)

**Examples:**
```bash
# Run with test environment, uses config-test.yaml, save to /tmp/flights-test/
./track_flights test /tmp/flights

# Run with prod environment, uses config-prod.yaml, save to /var/data/flights-prod/
./track_flights prod /var/data/flights

# Override config file
./track_flights test /tmp/flights --config custom-config.yaml
./track_flights test /tmp/flights --config config.yaml
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
Run Date     Dep Date   Route    IATA   Flight          Depart   Arrive   Next Day  Duration   Stops  Price        Fare Code
Unknown      2025-12-18 JFK-SFO  UA     UA123           08:00    11:30    No        5h 30m     0      299.99 USD   Y
Unknown      2025-12-20 JFK-SFO  AA     AA456           14:15    17:45    No        5h 30m     0      349.99 USD   Y
Unknown      2026-01-01 SFO-JFK  UA     UA789           09:30    18:00    No        5h 30m     0      399.99 USD   Y
```

**Column Descriptions:**
- **Run Date**: Date when the script was executed
- **Dep Date**: Requested departure date for the flight
- **Route**: Origin-Destination airport codes
- **IATA**: Airline code
- **Flight**: Flight number(s)
- **Depart/Arrive**: Departure and arrival times
- **Next Day**: Whether arrival is next day
- **Duration**: Total flight time
- **Stops**: Number of connections
- **Price**: Ticket price with currency
- **Fare Code**: Fare class code

## Features

- **Duration Filtering**: Client-side filtering of flights by maximum duration
- **Environment Support**: Separate test/prod configurations with automatic config file selection
- **Flexible Date Input**: Support for individual dates, date ranges, and combinations
- **JSON Output**: Structured data storage for analysis with departure date tracking
- **Multiple Routes**: Support for tracking multiple flight routes
- **Next-Day Detection**: Automatic detection of overnight flights
- **Connection Handling**: Shows complete trips with layovers as single entries
- **Historical Analysis**: Enhanced tools to review and compare historical flight data
- **Configuration Override**: Ability to specify custom config files
- **Compact Display**: Optimized table format for better readability

## Automation

### Cron Setup

To run daily at 9 AM (uses config-prod.yaml automatically):
```bash
0 9 * * * cd /path/to/flights && ./track_flights prod /var/data/flights
```

To run multiple times per day:
```bash
# Every 6 hours
0 */6 * * * cd /path/to/flights && ./track_flights prod /var/data/flights

# Twice daily (9 AM and 6 PM)
0 9,18 * * * cd /path/to/flights && ./track_flights prod /var/data/flights

# Use custom config for specific schedules
0 12 * * * cd /path/to/flights && ./track_flights prod /var/data/flights --config weekend-config.yaml
```

## Advanced Date Range Examples

The new date range functionality enables flexible flight tracking scenarios:

```yaml
# Weekend getaway - check Fri-Sun
departure_dates: 2025-12-05-07

# Holiday travel - multiple specific dates
departure_dates: 2025-12-23,2025-12-24,2026-01-02,2026-01-03

# Business trip - weekdays only
departure_dates: 2025-12-01,2025-12-02,2025-12-03,2025-12-04,2025-12-05

# Extended vacation - full week range
departure_dates: 2026-01-15-21

# Mixed ranges and individual dates
departure_dates: 2025-11-28-30,2025-12-15,2025-12-20-22
```

Each date in the range will be searched separately, giving you comprehensive coverage for flexible travel planning.

## Data Analysis

Use `dump_flights` to analyze trends:

```bash
# Find cheapest flights across all dates
./dump_flights /var/data/flights-prod --sort-by price | head -10

# Compare prices for specific route over time
./dump_flights /var/data/flights-prod --route JFK-SFO --sort-by date

# Find shortest flight times
./dump_flights /var/data/flights-prod --sort-by duration | head -5

# Analyze flights by specific departure date
./dump_flights /var/data/flights-prod --sort-by date | grep "2025-12-18"
```