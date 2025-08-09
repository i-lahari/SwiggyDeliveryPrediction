# Data Cleaning

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

# Data Preparation

## 1. Imports & Global Config
Imports:

pandas for data handling.

train_test_split from sklearn for splitting datasets.

yaml for reading configuration parameters.

logging for progress and error logs.

Path from pathlib for file path management.

Global Variable:

TARGET = "time_taken" → specifies the target column for modeling.

Logger Setup:

Creates a logger named "data_preparation" at INFO level.

Configures a console log handler with timestamps, logger name, log level, and message.

## 2. Function Definitions
load_data(data_path)

Reads a CSV file into a DataFrame.

Logs an error if the file is not found.

split_data(data, test_size, random_state)

Splits the DataFrame into train and test sets based on given test_size and random_state.

Returns (train_data, test_data).

read_params(file_path)

Reads a YAML file and returns its contents as a dictionary.

save_data(data, save_path)

Saves a DataFrame to CSV at the given path (no index).

## 3. Main Script (if __name__ == "__main__")
Set File Paths:

Defines root_path → 3 levels above the script’s location.

data_path → location of the cleaned Swiggy dataset (data/cleaned/swiggy_cleaned.csv).

save_data_dir → folder for saving train/test splits (data/interim), created if it doesn’t exist.

Defines file names and paths for train (train.csv) and test (test.csv) datasets.

Sets params_file_path → path to params.yaml.

Load Data:

Calls load_data() to read cleaned CSV into a DataFrame.

Logs "Data Loaded Successfully".

Read Parameters:

Reads params.yaml and extracts Data_Preparation section.

Retrieves test_size and random_state values.

Logs "parameters read successfully".

Split Data:

Uses split_data() to divide the dataset into train and test subsets.

Logs "Dataset split into train and test data".

Save Train & Test Sets:

Iterates over (train_data, test_data) with their respective file paths.

Calls save_data() to save each subset as CSV in data/interim/.

Logs messages like "train data saved to location" and "test data saved to location".

