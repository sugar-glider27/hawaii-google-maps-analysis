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


### NMAR Analysis

We believe that `price` may be Not Missing At Random(NMAR) because high-end or more expensive businesses may deliberately choose not to display their price tier on Google Maps because they do not want to scare off potential customers. A business's pricing may also span across multiple tiers making it difficult for the business to be categorized under one price tier. Additionally, lower, budget-friendly businesses may be more likely to display their price tier because the lower prices will make a business much more attractive towards people constrained to a lower budget. So, if based on this logic, we would expect `price` to be missing more often for businesses that are more expensive.

In order to attempt to explain the missingness in 'price', we would like to obtain data about a businesses true pricing. This could be done by looking at menu prices or the typical price of goods a business sells. This would allow us to determine which tier a business should be in and if more expensive businesses tend to have `price` missing.

### Missingness Dependency

We performed two permutation tests to assess whether the missingness of the `state` column 
depends on other variables in the dataset. 

**Test 1: Does missingness of `state` depend on `avg_rating`?**

- **Test statistic:** absolute difference in mean `avg_rating` between businesses where `state` is 
missing and businesses where it is not
- We are testing for the presence of difference and not the direction of difference
- **Observed statistic:** 0.0192
- **P-value:** 0.027
- **Significance level:** 0.01

At a significance level of 0.01, we fail to reject the null hypothesis. The missingness of 
`state` does not appear to depend on `avg_rating`. Although the p-value of 0.027 would be 
considered significant at the 0.05 level, it does not meet our stricter threshold of 0.01. 
We do not have statistically significant evidence to conclude that businesses with missing 
`state` have different average ratings than those with a recorded state.

**Test 2: Does missingness of `state` depend on `category`?**

- **Test statistic:** TVD
- We have multiple categories
- **Observed TVD:** 0.5127
- **P-value:** < 0.0001
- - **Significance level:** 0.01

At a significance level of 0.01, we reject the null hypothesis. The missingness of `state` 
is strongly dependent on `category`. The large TVD of 0.5127 indicates that the category 
distribution with 'missing' State is very different from those with a recorded 'state'. 
This test suggests that certain types of businesses are more likely to have their state 
recorded than others.

<iframe
  src="missingness-state-avg-rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

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

The baseline model uses two features:

| Feature | Type |
|----------|----------|
| `avg_rating` | Quantitative |
| `num_of_reviews` | Quantitative |

with a Decision Tree classifier.

This model contains:

* 2 quantitative features
* 0 ordinal features
* 0 nominal features

Because both features are numerical, no categorical encoding was required. Missing values were handled using median imputation before training the model.

### Performance

The baseline model achieved the following performance:

| Model | Train Accuracy | Test Accuracy | Train F1 | Test F1 |
|---------|---------:|---------:|---------:|---------:|
| Baseline | 0.486 | 0.484 | 0.159 | 0.153 |

We do not consider this baseline model to be a strong prediction model. In fact, both training and test accuracies are below 50% and the F1 score is very low, indicating that the model does not correctly identify closed businesses.
This is not surprising because the model relies only on two features: business’s average rating and number of reviews, which do not capture factors such as recent customer activity, changes in review patterns, or characteristics of the business itself. The baseline model serves primarily as a reference point against which more complex models can be compared.

## Final Model

Our final model is a **RandomForestClassifier** (Binary Classifier) trained to predict whether a business is permanently closed (`is_closed`).

The final model uses:

* Business characteristics
* Category information
* Review activity features
* Review trend features

Compared to the baseline model, we added several features that capture business characteristics and customer engagement patterns over time. These features were chosen because business closures are often associated with declining customer activity rather than simply low ratings.

### Original Columns Used

| Feature | Source |
|----------|----------|
| `avg_rating` | Meta dataset |
| `num_of_reviews` | Meta dataset |
| `category` | Meta dataset |

#### Original Columns Reasons for Inclusion

| Feature | Reason for Inclusion |
|----------|-------------|
| `category` | Different types of businesses may have different closure rates. For example, restaurants may face different challenges than retail stores or service businesses. |

### Engineered Features

| Feature | Description |
|----------|-------------|
| `reviews_last_6_months` | Number of reviews received in the last six months |
| `days_since_review` | Number of days since the most recent review |
| `rating_change` | Difference between the first and most recent review rating |
| `mean_review_rating` | Mean review rating across all reviews for the business |
| `has_recent_review` | Indicator for whether the business received at least one review in the last six months |
| `has_price` | Indicator for whether price information is available |


#### Engineered Features Reasons for Inclusion

| Feature | Reason for Inclusion |
|----------|-------------|
| `reviews_last_6_months` | Businesses that are actively operating are more likely to receive recent reviews, while inactive or struggling businesses may receive fewer recent reviews |
| `days_since_review` | A long period without reviews may indicate that a business is no longer attracting customers or may have already closed |
| `rating_change` | Changes in customer satisfaction over time may signal improvements or declines in business performance |
| `mean_review_rating` | Provides a business-level measure of customer satisfaction based on individual reviews |
| `has_recent_review` | Indicates whether a business has received any recent customer activity, which may be associated with continued operation |
| `has_price` | The completeness of a business listing may reflect how actively the business maintains its online presence. |

### Preprocessing:
Before training, missing values in numerical features were imputed using the median value of the corresponding feature. Missing categorical values were imputed using the most frequent category. The categorical feature `category` was then transformed using one-hot encoding so that it could be used by the Decision Tree model.

We performed a `GridSearchCV` with 5-fold cross-validation hyperparameter search over multiple combinations of:
- `max_depth`
- `min_samples_split`
- `min_samples_leaf`

The model achieving the highest test-set F1 score was selected as the final model.

Best hyperparameters:

 | Hyperparameter | Value |
|----------|----------|
| `max_depth` | `None` |
| `min_samples_split` | `10` |
| `min_samples_leaf` | `1` |

### Model Performance

| Model | Train Accuracy | Test Accuracy | Train F1 | Test F1 |
|---------|---------:|---------:|---------:|---------:|
| Baseline | 0.486 | 0.484 | 0.159 | 0.153 |
| Final | 0.973 | 0.922 | 0.828 | 0.420 |

The final model outperformed the baseline model. Test accuracy increased from 48.4% to 92.2%, while the test F1 score improved from 0.153 to 0.420. This suggests that the engineered features related to review activity, recency, and business characteristics provide valuable information for predicting whether a business is permanently closed.

This model still dtruggles to correcly identify closed bussiness. One of the reason is the lack of important information which could help with predictions such as revenue, foot traffic, and business age. However, the performance is increased compared to the baseline model.

#### Confusion Matrix

| Predicted Not Closed | Predicted Closed |
|----------|----------:|----------:|
| **Actual Not Closed** | 4785 | 221 |
| **Actual Closed** | 198 | 152 |

The model correctly identified 4,785 businesses that were not closed and 152 businesses that were permanently closed. It incorrectly classified 221 active businesses as closed (false positives) and failed to identify 198 closed businesses (false negatives).


### Why We Expected This Model to Perform Better

The baseline model relied only on average rating and review count. However, business closures are often correlated with changes in customer engagement rather than simply low ratings. By incorporating additional feautures, the final model may indentify businesses that are becoming inactive or losing customers.

## Fairness Analysis

**Group X**: High-review businesses — businesses at or above the median number of reviews

**Group Y**: Low-review businesses — businesses below the median number of reviews

**Evaluation Metric:** F1-score, chosen because our dataset is class-imbalanced (far more open businesses than closed ones), making F1 a more meaningful measure of model performance than accuracy.

**Null Hypothesis:** Our model is fair. Its F1-score for high-review businesses and low-review businesses are roughly the same, and any differences are due to random chance.

**Alternative Hypothesis:** Our model is unfair. Its F1-score for high-review businesses is different from its F1-score for low-review businesses.

**Test Statistic:** Difference in F1-score between high-review and low-review businesses.

**Significance Level:** 0.01

**Observed F1 Difference:** 0.2326

**P-value:** < 0.0001

**Conclusion:** At a significance level of 0.05, we reject the null hypothesis. The model performs significantly better on high-review businesses than low-review businesses. Our model relies heavily on review-based features like `days_since_review`, `reviews_last_6_months`, and `mean_review_rating`, which are more informative for businesses with many reviews. For businesses with few reviews, these features are less reliable, leading to weaker predictions. This suggests our model may be less fair toward newer or less popular businesses that have not yet accumulated many reviews.

## Project limitations
- We will assume that the current state of locations is correct in the dataset.
- Google Maps information may be outdated or incomplete.
- Many potentially important predictors, such as revenue, foot traffic, and business age, are not available.
- The dataset only contains businesses in Hawaii, so results may not generalize to other regions.
- For the modeling purposes, we defined a cleaning function for the column `category` to retrieve only the first value of the list containing all categories of the location. While this loses some of the original information, this method allows us to keep samples independent of each other and still keep the main information since, one might assume, catehories of a single location should be related.

