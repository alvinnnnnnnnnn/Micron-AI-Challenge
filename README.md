# Micron AI Challenge

The Micron AI Challenge centered on predicting wafer-level output measurements in semiconductor manufacturing. The provided dataset included sensor readings, tool lifetime information, chip location data indexed by (_x, y_) coordinates, and final chip measurement values.

The objective was to develop a model capable of accurately forecasting point-level measurement outcomes, while accounting for tool degradation, sensor signals, and spatial variability across the wafer. The model was also required to generalize across different tools and production conditions. Performance was evaluated on a held-out test set. Additional details are available in the attached documentation.

## Data Overview

Exploratory data analysis revealed the following:

1. Only one unique Process ID is present.
2. Each Tool ID is associated with multiple unique Run IDs.
3. Each Run ID corresponds to a single Tool ID.
4. Each Run ID contains multiple Step ID and Sensor Name pairs.

## Data Processing

Sensor readings were aggregated for each `(Run ID, Sensor Name, Step ID)` key by computing the average value. This aggregation produced a single representative sensor measurement for each key. The Sensor Name and Step ID columns were then combined to form a new column, denoted as `test`, with values in the format `(Sensor Name;Step ID)`.

The dataset was subsequently pivoted so that each unique Run ID corresponded to a single row, with column names representing the unique values in the `test` column. The data under these columns consisted of the aggregated sensor values described above.

This process was applied to both the Incoming Run and Run datasets. The resulting datasets were then merged with the Metrology data to form the final dataset used for modeling.

## Data Splitting

To ensure the model's ability to generalize across various tools, the data was split by Tool ID into training and validation sets. Of the 19 unique Tool IDs, 80% were randomly allocated to the training set, with the remaining 20% assigned to the validation set.

## Model Training

Three different machine learning models were evaluated:

1. Linear Regression
2. Extra Trees Regression
3. Multi-Layer Perceptron

All models were trained using the complete set of features. Model performance was assessed using root mean squared error (RMSE) and the coefficient of determination R². The table below summarizes the results:

| Model                   | RMSE   | R²        |
|:-----------------------:|:------:|:---------:|
| Linear Regression       | 0.2982 | -1.9152   |
| Extra Trees Regression  | 0.0449 | 0.9339    |
| Multi-Layer Perceptron  | 4.6550 | -709.2075 |

The Extra Trees Regression model demonstrated significantly superior performance compared to the other models. An optimal RMSE score is as close to 0 as possible, while an optimal \( R^2 \) score is as close to 1 as possible.

To avoid overfitting, we narrowed down the
features for training and tuned the hyperparameters.

For feature selection, we plotted the feature importance table using the Extra Trees Regressor model and selected the top-k features, which were then sent for model training, and the results were evaluated. Since the X, Y and Consumable Life features are crucial, they will be included as features for training. An example of the feature importance is shown below.

![feature importance table](https://github.com/alvinnnnnnnnnn/Micron-AI-Challenge/blob/main/plots/feature_importance_extra_trees-20.png)

We tested on 4 different k-values, and their respective results are shown below.
| k-value | RMSE   | R²     | Training Time |
|:-------:|:------:|:------:|:-------------:|
| 20      | 0.0529 | 0.8953 | 1m 9s         |
| 30      | 0.0539 | 0.9036 | 1m 39s        |
| 45      | 0.0489 | 0.9217 | 2m 45s        |
| 70      | 0.0416 | 0.9396 | 3m 13s        |
| all     | 0.0449 | 0.9339 | 34m 27s       |

To balance between computational resources required to train the model and model performance, we chose to use 45 features. 

## Model Optimisation 
We utilised Optuna Tuning to obtain the set of hyperparameters in order to obtain the best performance for our model. We managed to improve our model’s performance in both evaluation metrics.
| Model    | RMSE   | R²     |
|:--------:|:------:|:------:|
| Tuned    | 0.0477 | 0.9255 |
| Un-tuned | 0.0489 | 0.9217 |

Plotting an error distribution graph for both the untuned and tuned models, we can see that a higher frequency of data has its prediction error very close to 0 for the tuned model as compared to the untuned one. This indicates that the tuned model has a larger proportion of its predictions having very small errors, highlighting the improved performance of the tuned model over the untuned version.

![Error Distribution Chart Tuned](https://github.com/alvinnnnnnnnnn/Micron-AI-Challenge/blob/main/plots/error_distribution_Extra%20Trees%20Regressor%20Tuned-46.png)

_Error Distribution Graph for Tuned Model_

![Error Distribution Chart Un-Tuned](https://github.com/alvinnnnnnnnnn/Micron-AI-Challenge/blob/main/plots/error_distribution_Extra%20Trees%20Regressor-46.png)

_Error Distribution Graph for Un-tuned Model_
