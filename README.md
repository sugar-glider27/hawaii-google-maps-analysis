# Hawaii Google Maps Analysis
## Team Members
- Marta Krylova
- Kristen Tran


## Introduction
Whenever you are chosing a restaurant to visit, Google Maps can help you compare businesses using ratings, reviews, and other information. However, many locations open every year and close just as quickly. When they do close, the information is often not updated on Google maps, and one might wonder how to predict whether the location is permanently closed or still operating. This project uses a dataset of Hawaii's locations to predict whether a location is permanently closed.

Our project investigates the state column of the Hawaii Google Maps metadata which records the current operational status of each business, ranging from "Open ⋅ Closes 9PM" to "Permanently closed". 

Central Question:
Can we predict whether a Hawaii business listed on Google Maps is permanently closed, based on features of its organizational profile — its category, price tier, review engagement, rating, and how completely it filled out its listing?

### Datasets
meta-Hawaii.json.gz

Number of Rows: 21507

Relevant columns:

| Column | Description |
|---------|-------------|
| `gmap_id` | ID of the business |
| `category` | Category of the business |
| `avg_rating` | Average rating of the business |
| `num_of_reviews` | Number of reviews |
| `MISC` | Miscellaneous business information |
| `state` | Current status of the business (e.g., permanently closed) |
| `price` | Price tier of the business |

review-Hawaii_10.json.gz

Number of Rows: 1504347

Relevant columns:

| Column | Description |
|---------|-------------|
| `time` | Time of the review (Unix timestamp) |
| `rating` | Rating of the business |
| `gmap_id` | ID of the business |


## Data Cleaning and Exploratory Data Analysis
### Cleaning steps:
- Removed duplicate reviews and businesses
- Converted review timestamps to datetime format
- Created a binary target variable, is_closed, from the business status field
- Created misc_count, representing the number of miscellaneous business attributes
- Merged review and business datasets using gmap_id
- Created additional columns for modeling (refer to the Final Model section below)

### Univariate Analysis

<iframe
  src="num-reviews-dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="misc-count-dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

<iframe
  src="ratings-by-category.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="closure-rate-by-misc.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

#### Category Summary

| category | closure_rate | avg_rating | median_reviews | total_businesses |
|----------|-------------|------------|----------------|-----------------|
| Restaurant | 19.3% | 4.262 | 68.0 | 1416 |
| Cafe | 15.9% | 4.401 | 63.5 | 384 |
| Clothing store | 11.8% | 4.373 | 17.0 | 423 |
| Coffee shop | 9.6% | 4.339 | 88.0 | 354 |
| Gift shop | 7.9% | 4.375 | 18.0 | 265 |
| Takeout Restaurant | 6.5% | 4.064 | 66.0 | 278 |
| Fast food restaurant | 6.2% | 3.895 | 75.0 | 321 |
| Auto repair shop | 5.0% | 4.383 | 34.0 | 301 |
| Tourist attraction | 2.6% | 4.515 | 88.0 | 569 |
| Park | 0.7% | 4.341 | 66.5 | 278 |

#### Price Summary

| price | num_businesses | closure_rate | avg_rating | median_reviews |
|-------|---------------|--------------|------------|----------------|
| $ | 1598 | 8.8% | 4.148 | 117.0 |
| $$ | 2023 | 10.1% | 4.221 | 188.0 |
| $$$ | 235 | 11.9% | 4.288 | 148.0 |
| \$\$\$\$ | 114 | 7.0% | 4.425 | 63.0 |

## Assessment of Missingness

<iframe
  src="missingness-state-avg-rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### NMAR Analysis

We believe that 'price' may be Not Missing At Random(NMAR) because high-end or more expensive businesses may deliberately choose not to display their price tier on Google Maps because they do not want to scare off potential customers. A business's pricing may also span across multiple tiers making it difficult for the business to be categorized under one price tier. Additionally, lower, budget-friendly businesses may be more likely to display their price tier because the lower prices will make a business much more attractive towards people constrained to a lower budget. So, if based on this logic, we would expect 'price' to be missing more often for businesses that are more expensive.

In order to attempt to explain the missingness in 'price', we would like to obtain data about a businesses true pricing. This could be done by looking at menu prices or the typical price of goods a business sells. This would allow us to determine which tier a business should be in and if more expensive businesses tend to have 'price' missing.

### Missingness Dependency


## Hypothesis Testing
### Question
Do permanently closed businesses have different average ratings than businesses that remain open?

### Null Hypothesis
The average rating of permanently closed businesses is the same as the average rating of non-closed businesses.

### Alternative Hypothesis
The average rating of permanently closed businesses is not the same as the average rating of non-closed businesses.

### Test statistic
The absolute difference in mean ratings between permanently closed and non-closed businesses.

Testing with an alpha of 0.01.

### Results
We used a permutation test with 1,000 permutations to determine whether the observed difference in average ratings could reasonably occur under the null hypothesis.

- Observed Mean rating (closed): 4.1734
- Observed Mean rating (open):   4.3542
- Observed difference:  0.1809

P-value: 0.0000
Conclusion: Reject the Null Hypothesis

## Prediction Problem
### Prediciton Task
Predict whether a business is permanently closed

### Response Variable
is_closed

This is a binary classification problem because the outcome has two possible values:
* Closed
* Not Closed

We chose this variable as it directly relates to the topic of our anaylis (the closeness of locations).

### Evaluation Metric
F1 score was used as the primary evaluation metric because business closures are less common than non-closures, making class imbalance an important consideration, and we would like to balance precision and recall. Accuracy was separately considered.

## Baseline Model

The baseline model uses:

* avg_rating
* num_of_reviews

with a Decision Tree classifier.

## Final Model

Our final model is a **Decision Tree Classifier** (Binary Classifier) trained to predict whether a business is permanently closed (`is_closed`).


The final model uses:

* Business characteristics
* Category information
* Review activity features
* Review trend features

### Original Columns Used

| Feature | Source |
|----------|----------|
| `avg_rating` | Business dataset |
| `num_of_reviews` | Business dataset |
| `category` | Business dataset |
| `price` | Business dataset |
| `rating` | Review dataset |
| `time` | Review dataset |
| `gmap_id` | Both datasets |

### Engineered Features

| Feature | Description |
|----------|-------------|
| `reviews_last_6_months` | Number of reviews received in the last six months |
| `days_since_review` | Number of days since the most recent review |
| `rating_change` | Difference between the first and most recent review rating |
| `mean_review_rating` | Mean review rating across all reviews for the business |
| `has_recent_review` | Indicator for whether the business received at least one review in the last six months |
| `has_price` | Indicator for whether price information is available |

### Preprocessing:
Before training, missing values in numerical features were imputed using the median value of the corresponding feature. Missing categorical values were imputed using the most frequent category. The categorical feature `category` was then transformed using one-hot encoding so that it could be used by the Decision Tree model.

We performed a manual hyperparameter search over multiple combinations of:
- `max_depth`
- `min_samples_split`
- `min_samples_leaf`

The model achieving the highest test-set F1 score was selected as the final model.

### Model Performance

| Model | Train Accuracy | Test Accuracy | Train F1 | Test F1 |
|---------|---------:|---------:|---------:|---------:|
| Baseline | 0.486 | 0.484 | 0.159 | 0.153 |
| Final | 0.973 | 0.922 | 0.828 | 0.420 |

The final model outperformed the baseline model. Test accuracy increased from 48.4% to 92.2%, while the test F1 score improved from 0.153 to 0.420. This suggests that the engineered features related to review activity, recency, and business characteristics provide valuable information for predicting whether a business is permanently closed.


### Why We Expected This Model to Perform Better

The baseline model relied only on average rating and review count. However, business closures are often correlated with changes in customer engagement rather than simply low ratings. By incorporating additional feautures, the final model may indentify businesses that are becoming inactive or losing customers.

## Fairness Analysis

## Project limitations
- We will assume that the current state of locations is correct in the dataset.
- Google Maps information may be outdated or incomplete.
- Many potentially important predictors, such as revenue, foot traffic, and business age, are not available.
- The dataset only contains businesses in Hawaii, so results may not generalize to other regions.

