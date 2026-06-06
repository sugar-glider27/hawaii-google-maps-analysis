# Hawaii Google Maps Analysis
## Team Members
- Marta Krylova
- Kristen Tran


## Introduction
Whenever you are chosing to which restaurant to go, you can search for it on Google maps, and find the one you like. However, many locations open every year and close just as quickly. When they do close, the information is often not updated on Google maps, and one might wonder how to predict whether the location is permanently closed or still operating. This project uses a dataset of Hawaii's locations to predict whether a specified location in permanently closed or not.

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


## Assessment of Missingness

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

Testing for the 95% significance level

## Prediction Problem
### Prediciton Task
Predict whether a business is permanently closed

### Response Variable
is_closed

This is a binary classification problem because the outcome has two possible values:
* Closed
* Not Closed

### Evaluation Metric
F1 score was used as the primary evaluation metric because business closures are less common than non-closures, making class imbalance an important consideration, and we would like to balance precision and recall. Accuracy was separately considered.

## Baseline Model

The baseline model uses:

* avg_rating
* num_of_reviews

with a Decision Tree classifier.

## Final Model

The final model uses:

* Business characteristics
* Category information
* Review activity features
* Review trend features

Categorical variables are one-hot encoded and numerical features are median-imputed when necessary.

Hyperparameters were selected using a manual parameter search over multiple Decision Tree configurations.

## Raw Features Used in the Model

| Feature | Source |
|----------|----------|
| `avg_rating` | Business dataset |
| `num_of_reviews` | Business dataset |
| `category` | Business dataset |
| `price` | Business dataset |
| `rating` | Review dataset |
| `time` | Review dataset |
| `gmap_id` | Both datasets |

## Engineered Features

| Feature | Description |
|----------|-------------|
| `reviews_last_6_months` | Number of reviews received in the last six months |
| `days_since_review` | Number of days since the most recent review |
| `rating_change` | Difference between the first and most recent review rating |
| `mean_review_rating` | Mean review rating across all reviews for the business |
| `has_recent_review` | Indicator for whether the business received at least one review in the last six months |
| `has_price` | Indicator for whether price information is available |

## Fairness Analysis

## Project limitations
We will assume that the current state of locations is correct in the dataset.

