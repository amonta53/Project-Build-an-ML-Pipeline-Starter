# NYC Airbnb Price Prediction Pipeline

![MLflow](https://img.shields.io/badge/MLflow-Pipeline-blue)
![Weights & Biases](https://img.shields.io/badge/W&B-ExperimentTracking-yellow)
![Python](https://img.shields.io/badge/Python-3.13-blue)

This project implements an end-to-end machine learning pipeline for predicting Airbnb listing prices in New York City. The focus of the project is not just training a model, but demonstrating a reproducible ML workflow using modern ML engineering tools.

The pipeline covers the full lifecycle:
- data ingestion  
- cleaning and validation  
- dataset splitting  
- model training  
- hyperparameter tuning  
- model evaluation  
- artifact lineage tracking  

The pipeline is orchestrated using MLflow, configured with Hydra, and tracked with Weights & Biases.

# Project Architecture

The pipeline is organized into modular steps that are executed through MLflow Projects.

download_data  
↓  
basic_cleaning  
↓  
data_check  
↓  
train_val_test_split  
↓  
train_random_forest  
↓  
test_regression_model

Each step produces artifacts that are logged to Weights & Biases, allowing full lineage tracking from the original dataset all the way to the final trained model.

# Data Exploration Highlights

Before building the pipeline, I spent some time exploring the dataset to understand how Airbnb pricing behaves.

### Price Distribution

The price distribution is heavily right-skewed, with a relatively small number of listings priced far above the majority of the dataset.

To prevent those extreme outliers from distorting model training, a filtering rule was applied during the cleaning step:
 - min_price = 10  
 - max_price = 350
This keeps the model focused on the realistic price range most listings fall into.

### Location Matters
Neighborhood group turned out to be one of the strongest predictors of price.

Manhattan listings tend to have significantly higher prices, while outer boroughs show lower median values. That makes geographic location a critical categorical feature.

### Room Type Impact
Room type also plays a major role in pricing:
 - Entire home/apartment > Private room > Shared room

These categorical variables are encoded in the preprocessing pipeline before model training.

# Model Training
The baseline model used in this pipeline is a **Random Forest Regressor** built within a scikit-learn pipeline.

Random Forest was chosen because it:
- captures nonlinear relationships well  
- performs strongly on structured tabular data  
- is relatively robust to noisy features  
- requires minimal feature scaling  

The preprocessing and model training steps are combined into a single pipeline to ensure consistent transformations during both training and inference.

# Hyperparameter Tuning
A small hyperparameter sweep was executed using Hydra multirun to evaluate different model configurations.

The parameters explored were:
 - max_depth
 - n_estimators

This generated four candidate models, each tracked independently in Weights & Biases.

The best performing configuration was:
 - max_depth: 50
 - n_estimators: 200  

Validation performance for that configuration:
 - MAE ≈ 34  
 - R² ≈ 0.55

These parameters were then promoted into `config.yaml` as the default model configuration.

# Model Evaluation
After training, the selected model was evaluated on a **held-out test dataset** to estimate real-world performance.

Final test results:
 - Mean Absolute Error (MAE): ~34  
 - R² Score: ~0.55

This means the model is typically able to predict listing prices within roughly **$34 on average**.

For a dataset with significant variability and categorical influences like Airbnb listings, this represents reasonable baseline performance.

# Artifact Lineage
One of the goals of this project was to demonstrate **traceable ML pipelines**.

Weights & Biases was used to track artifact lineage across the entire workflow.

Example lineage flow:
sample.csv  
↓  
clean_sample.csv  
↓  
trainval_data.csv  
↓  
random_forest_export  
↓  
test_model

This provides full transparency into:

- which dataset version trained the model  
- which hyperparameters were used  
- which model produced the final predictions  

If a model needs to be reproduced later, every dependency is traceable.

# Reproducibility
The pipeline was built with reproducibility in mind.

| Tool | Purpose |
|-----|------|
| **MLflow** | Pipeline orchestration |
| **Hydra** | Configuration management |
| **Weights & Biases** | Experiment tracking and artifact lineage |

Because of this setup, the full pipeline can be reproduced with a single command.

Run locally:
 - mlflow run .

Run directly from GitHub using a tagged release:
 - mlflow run [https://github.com/amonta53/Project-Build-an-ML-Pipeline-Starter.git](https://github.com/amonta53/Project-Build-an-ML-Pipeline-Starter.git) -v 1.0.1

This ensures that the exact version of the pipeline used for training can always be recreated.

# Future Improvements
There are several directions this pipeline could be expanded in future versions.

### Additional Models
While Random Forest performed well as a baseline, additional models could be evaluated:
- Gradient Boosting
- XGBoost
- LightGBM

Boosted tree methods often outperform bagging approaches on tabular datasets.

### Feature Engineering
Additional derived features could improve model performance, including:
- geographic clustering of listings
- price per bedroom
- review frequency trends
- neighborhood density indicators

### Hyperparameter Optimization
The current hyperparameter sweep explores a relatively small grid.

Future work could introduce automated optimization approaches such as:
- Optuna
- Bayesian optimization

### CI/CD Integration
In a production environment, this pipeline could be connected to a CI/CD workflow to support:\
- automated retraining
- scheduled model updates
- deployment automation

# Running the Pipeline
Run the pipeline locally:
 - mlflow run .

Run the pipeline from GitHub:
 - mlflow run [https://github.com/amonta53/Project-Build-an-ML-Pipeline-Starter.git](https://github.com/amonta53/Project-Build-an-ML-Pipeline-Starter.git) -v 1.0.1

# Author
Andrew Montalbano  
Student ID: 012821411
D501 - Machine Learning DevOps – WGU