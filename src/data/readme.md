## 1. Imports & Logger Setup
Imports: Loads essential Python packages — numpy, pandas, Path from pathlib, and logging.

Logger Configuration:

Creates a logger named "data_cleaning".

Sets log level to INFO.

Adds a console handler with a timestamped, leveled log format for output.

## 2. Static Configuration
columns_to_drop: A predefined list of columns to remove after cleaning (IDs, coordinates, date components, etc.).

## 3. Helper Functions
load_data(data_path)

Reads a CSV from the given path into a Pandas DataFrame.

Logs an error if the file is missing.

change_column_names(data)

Standardizes column names: lowercases all names and renames specific ones to more descriptive, consistent labels.

data_cleaning(data)

Removes riders under 18 and entries with invalid ratings (e.g., "6").

Replaces "NaN " strings with actual np.nan.

Creates/updates multiple columns:

Extracts city_name from rider_id.

Converts age and ratings to numeric.

Makes latitude/longitude absolute values.

Parses order_date into datetime and derives day, month, weekday, weekend flag.

Parses order and pickup times into datetime objects, calculates pickup time in minutes.

Derives order_time_hour and maps to order_time_of_day via time_of_day() helper.

Cleans categorical text columns (weather, traffic, etc.).

Converts multiple_deliveries to float.

Cleans time_taken to numeric minutes.

Drops raw time columns after processing.

clean_lat_long(data, threshold)

Replaces lat/long values below a set threshold with NaN.

extract_datetime_features(ser)

Given a date series, returns day, month, year, weekday, and weekend flag as a new DataFrame.

time_of_day(ser)

Categorizes hours into labels (after_midnight, morning, afternoon, evening, night).

calculate_haversine_distance(df)

Computes the great-circle (Haversine) distance between restaurant and delivery points (in km).

Adds a new distance column.

create_distance_type(data)

Categorizes deliveries into "short", "medium", "long", "very_long" based on distance bins.

drop_columns(data, columns)

Removes specified columns.

## 4. Main Cleaning Pipeline
perform_data_cleaning(data, saved_data_path)

Passes the raw data through a .pipe() chain:

Rename columns.

Apply main cleaning (data_cleaning).

Clean lat/long anomalies.

Compute Haversine distance.

Categorize distance.

Drop unneeded columns.

Saves cleaned DataFrame to CSV.

## 5. Script Entry Point (if __name__ == "__main__")
Defines paths for raw data (data/raw/swiggy.csv) and cleaned data (data/cleaned/swiggy_cleaned.csv).

Ensures cleaned data folder exists.

Loads raw CSV using load_data.

Logs “Data read successfully”.

Runs perform_data_cleaning to clean & save CSV.

Logs “Data cleaned and saved”.

