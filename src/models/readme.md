# Data Training

## 1. Imports & Global Config
Imports:

pandas for loading training data.

yaml for reading parameters from params.yaml.

joblib for saving trained models/transformers.

logging for logging workflow steps.

Model classes:

RandomForestRegressor (sklearn)

LGBMRegressor (LightGBM)

LinearRegression (sklearn)

StackingRegressor (sklearn)

PowerTransformer for target transformation.

TransformedTargetRegressor for wrapping model + target transformer.

Path from pathlib for file path handling.

Global variable:

TARGET = "time_taken" (target column for prediction).

Logger setup:

Logger named "model_training" at INFO level.

Console handler with timestamped log messages.

## 2. Helper Functions
load_data(data_path) → Reads CSV file into a DataFrame. Logs error if file not found.

read_params(file_path) → Reads and parses YAML parameters into a dictionary.

save_model(model, save_dir, model_name) → Saves a model object using joblib.

save_transformer(transformer, save_dir, transformer_name) → Saves a transformer object using joblib.

train_model(model, X_train, y_train) → Fits a model on the given training data.

make_X_and_y(data, target_column) → Splits a DataFrame into features X and target y.

## 3. Main Execution (if __name__ == "__main__")
Paths setup:

Loads processed train dataset from data/processed/train_trans.csv.

Loads parameter file from params.yaml.

Sets model save directory to models/ (creates it if missing).

Load and split data:

Reads processed training data into DataFrame.

Splits into X_train (features) and y_train (target).

Read training parameters:

Extracts training configuration from params.yaml under Train section.

Reads hyperparameters for Random Forest and LightGBM models.

Model initialization:

Builds RandomForestRegressor with given parameters.

Builds LGBMRegressor with given parameters.

Defines LinearRegression as the meta-model for stacking.

Initializes PowerTransformer for transforming the target variable.

Build stacking model:

Combines Random Forest and LightGBM as base models.

Uses Linear Regression as the final estimator.

Wraps everything in a StackingRegressor with 5-fold CV.

Wrap with target transformer:

Creates TransformedTargetRegressor to apply PowerTransformer to y_train during fitting.

Train model:

Fits the wrapped model (model) on training data.

Logs training completion.

Save models and transformer:

Saves the entire wrapped model as model.joblib (includes preprocessing of target + regressor).

Extracts and saves the stacking regressor separately as stacking_regressor.joblib.

Saves the PowerTransformer separately as power_transformer.joblib.

This script essentially trains a stacked ensemble regressor with target transformation, then saves:

The full wrapped model (ready for prediction).

The core stacking regressor (without target transformation).

The transformer for the target variable.

# Evaluation 

## 1. Setup & Config
Imports libs: pandas, joblib, sklearn metrics/CV, MLflow, DagsHub, Path, json, logging.

Initializes DagsHub + MLflow tracking:

dagshub.init(..., mlflow=True)

mlflow.set_tracking_uri(...) to DagsHub’s MLflow endpoint.

Sets experiment name: “DVC Pipeline.”

Creates a logger (model_evaluation) with timestamped console output.

Sets TARGET = "time_taken".

## 2. Utility functions
load_data(path): read CSV → DataFrame (logs error if missing).

make_X_and_y(df, target): split features X and target y.

load_model(path): load a serialized model via joblib.

save_model_info(path, run_id, artifact_path, model_name): write a small JSON with run metadata for later reuse.

## 3. Load artifacts & data
Builds project root_path.

Points to processed train/test CSVs and the trained model (models/model.joblib).

Loads train and test DataFrames; logs success.

Splits each into X_*, y_*.

## 4. Inference & metrics
Loads the trained model (this is your TransformedTargetRegressor wrapping the stacking ensemble + power transform from earlier).

Runs predictions on train and test.

Computes:

MAE on train and test.

R² on train and test.

Runs 5-fold cross-validation on the training set with scoring = neg_mean_absolute_error (i.e., more negative is worse), then takes the mean and flips sign to a positive MAE for readability.

## 5. MLflow logging (to DagsHub)
Within mlflow.start_run():

Sets a tag: model = "Food Delivery Time Regressor".

Logs parameters from model.get_params() (works for sklearn wrappers).

Logs metrics: train/test MAE, train/test R², and mean CV MAE (sign-corrected).

Logs each CV fold as separate metrics (CV 0, CV 1, …).

Creates MLflow Dataset objects from train/test DataFrames and logs them as inputs with contexts "training" and "validation".

Infers a model signature using a 20-row sample of X_train and the model’s predictions (helps downstream validation/serving).

Logs the full model (mlflow.sklearn.log_model(..., "delivery_time_pred_model", signature=...)).

Logs additional artifacts:

models/stacking_regressor.joblib

models/power_transformer.joblib

models/preprocessor.joblib

Captures the run’s artifact URI for later reference.

Ends the run (everything above is now recorded in the DagsHub MLflow UI).

## 6. Persist run metadata locally
Extracts the run_id and sets model_name = "delivery_time_pred_model".

Writes run_information.json at repo root with:

run_id

artifact_path (the MLflow artifact URI)

model_name
…so other scripts (e.g., registration/deployment) can find the right run/artifacts.

# Registering the Model
## 1. Setup & Config
Imports MLflow, DagsHub, JSON, paths, MlflowClient, and logging.

Initializes a logger named register_model with timestamped console output.

Initializes DagsHub (dagshub.init(..., mlflow=True)) and points MLflow tracking URI to your DagsHub project.

## 2. Helper
load_model_information(file_path)

Opens run_information.json and returns a dict with keys like run_id, artifact_path, and model_name (written by your eval script).

## 3. Main flow (if __name__ == "__main__")
Paths

Computes root_path and points to run_information.json at the repo root.

Load run info

Reads run_id and model_name from the JSON.

Build model URI

Creates MLflow runs URI: runs:/{run_id}/{model_name} — this points at the logged model artifact inside that run.

Register the model

Calls mlflow.register_model(model_uri=..., name=model_name) to create a new Model Registry entry/version in DagsHub’s MLflow.

Capture registry metadata

Extracts registered_model_version and registered_model_name from the returned object and logs the latest version.

Transition stage → “Staging”

Uses MlflowClient().transition_model_version_stage(...) to move that version to the Staging stage in the registry.

Logs confirmation that the model is now in Staging.

