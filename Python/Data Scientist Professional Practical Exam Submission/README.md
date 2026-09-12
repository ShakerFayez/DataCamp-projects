# Recipe Site Traffic Prediction

Predicting which recipes will drive high traffic to the Tasty Bytes recipe website, so the team can decide which recipe to feature on the homepage each day.

## Project Overview

Tasty Bytes currently picks a recipe to feature on its homepage largely by intuition. Their own internal model claims **80% accuracy at identifying recipes that lead to high traffic**, but that figure is reported for only one class and can't be fully trusted on its own. This project builds and evaluates classification models that predict whether a recipe will generate high site traffic, using historical recipe data (nutrition facts, category, and serving size).

**Problem type:** Binary classification (`high_traffic`: High vs. Low)

## Dataset

`recipe_site_traffic_2212.csv` — 947 recipes, 8 columns:

| Column | Description |
|---|---|
| `recipe` | Unique recipe ID |
| `calories`, `carbohydrate`, `sugar`, `protein` | Nutritional content per serving |
| `category` | Recipe category (e.g., Chicken, Dessert, Breakfast) |
| `servings` | Number of servings |
| `high_traffic` | Target label — "High" if the recipe drove high traffic, otherwise missing/Low |

## Data Validation & Cleaning

- **`recipe`**: Verified as a clean, unique identifier (no duplicates; sequential range).
- **`calories`, `carbohydrate`, `sugar`, `protein`**: 895 of 947 rows populated (52 missing values each, same rows) — left as-is for now and addressed later via transformation.
- **`category`**: Had an inconsistent duplicate label (`Chicken Breast`), consolidated into `Chicken`, leaving 10 valid categories.
- **`servings`**: Stored as text with inconsistent values (e.g., `"4 as a snack"`); stripped the qualifier and cast to integer.
- **`high_traffic`**: Only positive ("High") values were recorded; missing values were treated as "Low" traffic and filled accordingly.
- **Outliers**: Removed using an IQR-based filter (threshold = 2×IQR) across all four nutritional columns, dropping the dataset from 895 to 715 usable rows.

## Exploratory Analysis

- **Distributions**: Histograms of `calories`, `carbohydrate`, `sugar`, and `protein` showed strong right-skew, with a long tail of high-value outliers in each — motivating the outlier removal step above.
- **Outlier check**: Boxplots confirmed the same nutritional columns carried significant outliers before cleaning.
- **Category mix**: A count plot of `category` showed traffic-driving recipes are not evenly distributed across categories, with some (e.g., Breakfast, Chicken) far more frequent than others (e.g., Beverages).
- **Traffic balance**: The target class is moderately imbalanced, with more "High" traffic recipes than "Low" in the cleaned set.
- **Feature relationships**: A pairplot across the four nutritional variables did not reveal strong linear correlation between them, suggesting each contributes distinct signal to the model.

## Model Development

- **Preprocessing**: Applied a square-root transform to the nutritional columns to reduce skew (in place of standard scaling), and label-encoded the `category` and `high_traffic` columns.
- **Split**: 80/20 train-test split (572 train / 143 test rows), stratified by feature matrix shape.
- **Baseline model**: Logistic Regression — chosen for its interpretability and resistance to overfitting on a modest, moderately-imbalanced dataset.
- **Comparison models**: Decision Tree, Random Forest, and Gradient Boosting classifiers — chosen to test whether more flexible, non-linear models could better capture patterns in the nutritional/category features.

## Model Evaluation

| Model | Accuracy (Test) | Precision (Test) | Recall (Test) | F1 (Test) | Accuracy (Train) |
|---|---|---|---|---|---|
| **Logistic Regression** | **0.762** | 0.662 | **0.804** | **0.726** | 0.747 |
| Decision Tree | 0.650 | 0.545 | 0.643 | 0.590 | 1.000 |
| Random Forest | 0.755 | **0.691** | 0.679 | 0.685 | 1.000 |
| Gradient Boosting | 0.720 | 0.648 | 0.625 | 0.636 | 0.941 |

**Key finding:** The Decision Tree, Random Forest, and Gradient Boosting models all show a large gap between train and test performance (the tree-based models hit 94–100% accuracy on training data), indicating overfitting. Logistic Regression has near-matching train/test scores (0.747 vs. 0.762), showing it generalizes best despite having the simplest structure — and it also achieves the highest recall and F1-score on the test set.

## Business Metric

Recall on the "High traffic" class is the most business-relevant metric: missing a recipe that *would* have driven high traffic (a false negative) is more costly than occasionally featuring one that underperforms. On this basis, Logistic Regression is the strongest candidate — its 80.4% test recall means it correctly identifies the large majority of genuinely high-traffic recipes, and it does so without the overfitting seen in the other models, making it more reliable to deploy in production.

## Final Summary & Recommendations

This project validated and cleaned the recipe traffic dataset, explored the distributions and relationships among its features, and built four candidate classification models to predict recipe popularity. Logistic Regression emerged as the recommended baseline model: it best balances test accuracy, recall, and generalization, and satisfies the business's ~80% positive-class performance expectation without overfitting.

**Recommendations for the business:**
1. **Clarify the current model's performance.** Tasty Bytes has only shared performance figures for low-traffic recipes; a full picture (across both classes) is needed to properly benchmark any new model against it.
2. **Enrich the dataset.** Nutritional and category data alone yield moderate predictive power (~76% accuracy). Additional signals — such as recipe name/description text, images, or user engagement history — would likely improve predictions more than further tuning on the current features.
3. **Explore text-based features in a follow-up phase.** Recipe names may carry useful signal (e.g., appealing or seasonal language), but this requires more computational resources than were available during this exercise and is recommended as a next step.

## Tech Stack

- Python, pandas, NumPy
- matplotlib, seaborn (visualization)
- scikit-learn (`LogisticRegression`, `DecisionTreeClassifier`, `RandomForestClassifier`, `GradientBoostingClassifier`, `LabelEncoder`, `train_test_split`)
