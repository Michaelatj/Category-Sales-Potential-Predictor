# UMKMentor — Product Success Predictor

A machine learning project for helping UMKM sellers estimate whether a product has good sales potential before they decide to stock or promote it.

## Overview

This project builds a tabular classification model that predicts whether a product is likely to sell well (`Laku`) or not (`Tidak Laku`) based on marketplace product attributes such as price, stock, discount, rating, store trust signals, and category-level statistics.

The project is part of **UMKMentor**, an AI for Business Intelligence and Market Insights solution for UMKM sellers.

## Problem Statement

Many UMKM sellers choose products based on trends or assumptions, not data. This increases the risk of:
- choosing products with weak demand,
- setting prices that are not competitive,
- ignoring discount strategy,
- underestimating the role of trust signals such as official store status and ratings.

This model is designed to reduce that risk by giving a simple data-driven product potential prediction.

## Dataset

The project uses the **Tokopedia Product and Review Dataset**.

### Dataset Summary
- **Source:** Tokopedia public product and review data
- **Products:** 5,553
- **Reviews:** 1M+
- **Categories:** 24 product categories
- **Language:** Indonesian

### Main Files
- `tokopedia_products_with_review.csv`
- `tokopedia_products_with_review.json`

### Dataset Content
The dataset contains:
- product information
- pricing information
- sales-related information
- store attributes
- review texts and ratings

### Important Product Columns
- `product_id`
- `category`
- `name`
- `count_sold`
- `discounted_price`
- `preorder`
- `price`
- `stock`
- `gold_merchant`
- `is_official`
- `is_topads`
- `rating_average`
- `shop_location`

### Review Columns
- `review_id`
- `variant_name`
- `message`
- `review_rating`
- `review_time`
- `review_timestamp`
- `review_response`
- `review_like`
- `bad_rating_reason`

## Data Preparation

In the notebook, the raw data was cleaned and reduced to a final modeling table.

### Final Modeling Dataset
- **Initial loaded rows:** 6,353
- **Final cleaned rows:** 3,039
- **Final features:** 22
- **Train set:** 2,431 rows
- **Test set:** 608 rows

### Additional Synthetic Data
To support a category that did not exist in the original dataset, the notebook includes **800 rows of synthetic data for the `pertukangan` category**.

### Label Definition
The target label is created as:

`is_laku = 1` if `count_sold > cat_sold_median`  
`is_laku = 0` otherwise

This means the model predicts whether a product performs above the median sales of its category.

## Feature Engineering

The notebook creates features that capture business context, including:
- log price transformations
- price position relative to category median
- price rank inside category
- stock rank inside category
- stock sufficiency
- discount presence
- discount percentage
- store trust signals
- category one-hot encoding

### Feature Audit
During model refinement, the feature set was adjusted to improve consistency with real inference use:
- `TopAds` was removed from model inputs
- `gold_merchant` was added as an explicit feature
- `is_official` was kept as a direct trust feature
- ambiguous derived features such as `trust_factor` were simplified

### Final Feature Set
Total final features used by the model: **22**

## Models Tested

The notebook compares several models, including:
- Logistic Regression
- Support Vector Machine (SVM)
- Random Forest
- Gradient Boosting

## Best Model

The final selected model is:

**Gradient Boosting tuned + calibrated (sigmoid)**

### Final Performance
- **Test AUC:** 0.745

## Key Insights from the Notebook

The strongest signals in the model include:
- `discount_pct`
- `rating_average`
- `stock_is_enough`
- `cat_pct_official`
- category-level patterns such as `cat_pertukangan`

This suggests that discount strategy, product rating, stock adequacy, and category context are important indicators for sales potential.

## Explainability

The notebook includes:
- global model interpretability using SHAP / feature importance
- business-rule-based recommendation messages in the inference flow

## Supported Categories

The inference function supports these category groups:
- elektronik
- hiburan
- olahraga
- kecantikan
- makanan_minuman
- fashion
- pertukangan

## Inference

The notebook includes a `predict_product()` function that accepts:
- product category
- selling price
- official store status
- Gold Merchant status
- stock
- rating average
- discounted price

It returns:
- predicted label
- probability score
- risk level
- simple business suggestions

Note: the discount input is handled as **final price after discount**, not discount percentage.

## Exported Artifacts

The notebook exports the following files:
- `tokped_classifier.pkl`
- `feature_cols.json`
- `category_levels.json`
- `category_stats.json`
- `category_aliases.json`
- `cat_rating_median.json`
- `cat_discount_rate.json`
- `model_metadata.json`

## Tech Stack

- Python
- Pandas
- NumPy
- Scikit-learn
- Gradient Boosting
- Matplotlib
- Seaborn
- SHAP
- Joblib

## Business Value

This project helps UMKM sellers:
- reduce product selection risk,
- compare product potential more objectively,
- improve pricing decisions,
- understand category behavior,
- make faster, data-driven decisions.

## Limitations

A few limitations noted in the notebook:
- some signals are weak in certain categories,
- TopAds can create misleading correlations in observational data,
- synthetic data is used for the `pertukangan` category,
- the model is still best treated as an MVP, not a production-grade forecasting system.

## How to Use

1. Load the exported model and metadata files.
2. Prepare input data using the same feature logic from the notebook.
3. Call the `predict_product()` logic or the model directly.
4. Use the probability score as a decision-support signal, not as an absolute truth.

## Project Context

This notebook belongs to the **UMKMentor** project for AI-based business intelligence and market insight, focused on helping UMKM sellers make better decisions using product, review, and marketplace data.
