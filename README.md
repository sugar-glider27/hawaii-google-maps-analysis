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
| `resp` | Business response to the review, including timestamp and response text |
| `gmap_id` | ID of the business |


## Data Cleaning and Exploratory Data Analysis
### Cleaning steps:
- Drop duplicated rows
- Get timestamps from time

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

## Problem Identification

## Baseline Model

## Final Model

## Fairness Analysis

## Project limitations
We will assume that the current state of locations is correct in the dataset.

