## 1. Imports & Configurations
Imports:

pandas for data manipulation.

logging for logging progress and errors.

Path from pathlib for handling file paths.

ColumnTransformer from sklearn.compose to combine multiple preprocessing steps.

OneHotEncoder, MinMaxScaler, OrdinalEncoder for data transformations.

joblib to save/load Python objects (like the preprocessor).

set_config from scikit-learn to configure output type.

Sklearn Config:

Sets transformer outputs to pandas DataFrame using set_config(transform_output='pandas').

## 2. Column Definitions
num_cols → Numeric columns to scale (age, ratings, pickup_time_minutes, distance).

nominal_cat_cols → Nominal categorical columns to one-hot encode (weather, type_of_order, etc.).

ordinal_cat_cols → Ordinal categorical columns to ordinal encode (traffic, distance_type).

target_col → "time_taken" (the prediction target).

Ordinal Encoding Orders:

traffic_order: ["low","medium","high","jam"].

distance_type_order: ["short","medium","long","very_long"].

## 3. Logger Setup
Creates logger named "data_preprocessing".

Configures INFO level logging with timestamps and detailed messages.

## 4. Helper Functions
load_data(data_path)

Loads CSV into DataFrame.

Logs error if file is missing.

drop_missing_values(data)

Logs dataset shape before and after dropping missing rows.

Drops any rows containing NaN.

Raises error if NaNs remain.

save_transformer(transformer, save_dir, transformer_name)

Saves the fitted transformer using joblib.

train_preprocessor(preprocessor, data)

Fits the preprocessor on provided data.

perform_transformations(preprocessor, data)

Applies transformations and returns transformed DataFrame.

save_data(data, save_path)

Saves DataFrame to CSV without index.

make_X_and_y(data, target_column)

Splits DataFrame into features X and target y.

join_X_and_y(X, y)

Joins transformed features and target back into a single DataFrame.

## 5. Main Execution Flow (if __name__ == "__main__")
Define Paths:

Loads train/test data from data/interim.

Defines save directory for processed data (data/processed) and creates it if needed.

Sets file paths for transformed train/test datasets.

Build Preprocessor:

Numeric columns → MinMaxScaler().

Nominal categorical → OneHotEncoder(drop="first", handle_unknown="ignore", sparse_output=False).

Ordinal categorical → OrdinalEncoder() with predefined order, unknown values set to -1, missing values encoded as -999.

remainder="passthrough" keeps any extra columns unchanged.

Load & Clean Data:

Reads and drops missing values from train and test datasets.

Split Features & Target:

Separates into X_train, y_train and X_test, y_test.

Fit Preprocessor:

Fits on X_train only (to avoid data leakage).

Transform Data:

Transforms X_train and X_test into scaled/encoded versions.

Rejoin Target:

Joins transformed features with target column to create final processed DataFrames.

Save Transformed Data:

Saves processed train/test CSVs into data/processed/.

Save Preprocessor:

Saves the fitted preprocessor as models/preprocessor.joblib for later use in inference/pipelines.

