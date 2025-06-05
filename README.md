# Investigating Protein Content in Vegetarian Recipes and Non-Vegetarian Recipes

Author: Abby McLeod

## Overview

This data science project, conducted at UC San Diego, explores the relationship between protein content and different tags that a recipe is given. Specifically, there is an emphasis on recipes tagged with "vegan" or "vegetarian" vs those lacking those tags. 

## Introduction

Food is essential to our everyday lives, and while it brings joy to some, the act of deciding what to eat or cook is a decision that puts a damper on others' days. There are endless recipes on the internet to parse through, and many factors to consider when deciding what to cook. Many poeple prioritize an active lifestyle, and something that goes hand-in-hand with that is nutrition. There are many fad diets and clickbait statistics on the internet about the protein content of food, so I want to investigate exactly **what features of recipes have the highest correlation with a higher protein content?** To carry out the investigation, I am analyzing two datasets obtained from [food.com](https://www.food.com) which include recipes and ratings dating back to 2008. Originally, the datasets were used for the recommender system research paper, [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) by Majumder er al.

The dataset titled `recipe` contatins 83782 unique recipes, each as their own row, with the following columns:

- **name**: Recipe name
- **id**: Recipe ID
- **minutes**: Minutes to prepare recipe
- **contributor_id**: User ID who submitted this recipe
- **submitted**: Date recipe was submitted
- **tags**: Food.com tags for recipe
- **nutrition**: Nutrition information in the form `[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]`; PDV stands for “percentage of daily value”
- **n_steps**: Number of steps in recipe
- **steps**: Text for recipe steps, in order
- **description**: User-provided description
- **ingredients**: Text for recipe ingredients
- **n_ingredients**: Number of ingredients in recipe

The other dataset titled `interactions` contains 731927 reviews on a specific recipe, each corresponding to their own row, with the following columns:

- **user_id**: User ID
- **recipe_id**: Recipe ID
- **date**: Date of interaction
- **rating**: Rating given
- **review**: Review text

**Using this data, I am discovering what tags tend to correlate with protien (PDV) the most.** In order to do this, the percent daily value (PDV) of protein was extracted from the `nutrition` column and turned into its own column. This new column, called `protein`, represents the proportion of total daily protein intake that a serving of the corresponding recipe includes. 

In my analysis, the `protein` and `tags` columns will be the most important. The `tags` column includes a list of different labels attached to the corresponding recipe that makes it easier for users to filter through the recipes and quickly understand what it is offering. For example, I will focus a large amount of analysis on recipes marked as 'vegetarian' or 'vegan' as these are diets that exclude animal products- a primary source of protein for many people.

By exploring this topic, I will uncover the types of recipes that tend to have more protein content and what features are typically correlated with this. Additioanlly, this information can be used by the public to refine their eating habits to more closely suit their lifestyle goals, while deubunking dietary misinformation online.

## Data Cleaning and Exploratory Data Analysis

Before starting any analysis, I checked the data type of all the columns to understand what steps I needed to take to make the analysis process smoother and more efficient. The types were as follows:

- **name**: object
- **id**: int64
- **minutes**: int64
- **contributor_id**: int64
- **submitted**: object
- **tags**: object
- **nutrition**: object
- **n_steps**: int64
- **steps**: object
- **description**: object
- **ingredients**: object
- **n_ingredients**: int64
- **user_id**: float64
- **recipe_id**: float64
- **date**: object
- **rating**: float64
- **review**: object

To begin, I sought to streamline my analysis by cleaning the datasets as follows:

1. Left merging the `recipes` and `interactions` datasets together

    - This step matched the unique values in `recipe_id` to correctly match reviews with their intended recipes.

1. Replacing values of '0' in the column `ratings` with `np.nan`

    - This step is essential because ratings typically fall on a scale of 1 to 5, not including 0. Any values of '0' indicate a missing value rather than a true rating of 0, and would therefore introduce bias into `ratings`.

1. Adding a column called `avg_rating` containing the average rating per recipe.

    - Averaged all the ratings for each recipe provided by users  to gain a more comprensive metric on how the rceipe was rated as a whole.

1. Converting values in `nutrition` to be a list, rather than a string.

    - This allows for easy extraction of the individual values later on during analysis.

1. Adding a `protein` column containing the percent daily value of protein in the recipe.

    - This was done by applying a lambda function to the lists in `nutrition`, extracting protein (PDV) as a float from index = 4.

    - This step allows for the analysis of trends relating to recipe protein content.

1. Creating `tags_str` containing all the values in the original tags column as one string with each tag seperated by a space. 

    - Allows for text based vectorization techniques and simplifies preprocessing for later token analysis.

The final data frame had the following data types:

- **name**: object 
- **id**: int64 
- **minutes**: int64
- **contributor_id**: int64
- **submitted**: object 
- **tags**: object 
- **nutrition**: object 
- **n_steps**: int64
- **steps**: object 
- **description**: object 
- **ingredients**: object 
- **n_ingredients**: int64
- **user_id**: float64
- **recipe_id**: float64
- **date**: object
- **rating**: float64 
- **review**: object 
- **avg_rating**: float64
- **protein**: float64 
- **tags_str**: object 


## Assessment of Missingness

The columns `'date'`, `'rating'`, and `'review'` have a significant amount of missing values in the dataset, so I will assess the missingness of these variables. 

### NMAR Analysis

I beleive it is likely that the missingness of `'review'` is **NMAR** because people usually only write reviews if they have strong feelings about a recipe. Writing a review takes considerably more time and energy than just submitting a rating, so people tend to only write reviews when they have something to say. While it could be argued that this is MAR, with the missinginess correlating with extreme scores in `'rating'`, I do not believe this is the case because people sometimes include tips or details about the recipe in their reviews, not just their opinions. 

### Missingness Dependency

The column `'rating'` is one that I beleive could have a variety of reasons justifying its missingness, so I investigated that column next. I first chose to compare assess the missingness of `'rating'` against the column `'avg_rating'` to see if its missingness depended on the overall rating of the recipe. 

> Average Rating and Rating

**Null Hypothesis:** The missingness of the ratings does not depend on the avegrage rating of the recipe.

**Alternate Hypothesis:** The missingness of the ratings does depend on the average rating of the recipe. 

**Test Statistic:** The total variation distance (TVD) of average ratings between the group with missing ratings and without missing ratings.

**Significance Level:** 0.01

**Observed Statistic:** 0.055

**P-Value:** 0.0000

<iframe
  src="assets/tvd.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

As displayed in the above figure, the observed tvd is much higher than any of the values calculated during the permutation test under the null. The p-value of this test is 0.0000, which is less than the chosen significance level of 0.01. So, if the null hypothesis were true, it would be extremely unlikely that there is a statistic as drastic, or more drastic than the observed statistic, leading me to **reject the null hypothesis**, and conclude that the missingness of the ratings is **MAR**.

This makes sense as people could be less inclined to leave a rating if they were left underwhelmed or uninterested by a recipe. Additionally, food.com or recipe curators may only encourage people to leave ratings if they enjoyed their recipe, but not provide the same encouragement if they didn't.

> Minutes and Rating

**Null Hypothesis:** The missingness of the ratings does not depend on the number of minutes it takes to prepare of the recipe.

**Alternate Hypothesis:** The missingness of the ratings does depend on the number of minutes it takes to prepare of the recipe. 

**Test Statistic:** The mean absolute difference (MAD) of average ratings between the group with missing ratings and without missing ratings.

**Significance Level:** 0.01

**Observed Statistic:** 0.000

**P-Value:** 0.130

<iframe
  src="assets/mad.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed statistic for this data is very close to 0 with a p-value of 0.130. This means that at the 0.1 significance level, we **fail to reject the null hypothesis**. As shown by the data, it is not unlikely that we got the observed statistic that we got given the null is true. 

### Hypothesis Testing

Through a hypothesis test, I am trying to discover whether there is a relationship between protein content, found in column `'protein'`as percent daily value (PDV), and whether a recipe is vegan or vegetarian, as evidenced by whether they are tagged with 'vegan' or 'vegetarian'. The details of the two sample t test are as follows:

**Null Hypothesis:** There is no differnece in protein content (PDV) in recipes tagged 'vegan' or 'vegetarian' vs those without the tags. 

**Alternative Hypothesis:** Recipes tagged 'vegan' or 'vegetarian' have lower protein (PDV) than those not tagged.

**Test Statistic:** The difference in mean between vegetarian/vegan recipes and normal recipes, divided by the standard error. \( t = \frac{\text{difference in means}}{\text{standard error (SE)}} \)

**Significance Level:** 0.01

The student's t-test is the appropriate test because the predictor (veg vs no veg) is categorical, and we are looking to discover whether the means of these two groups are statistically different, which is what the t-test is designed to do. There is also a decently large sample size, meaning the central limit theorem allows us to assume normality.  

Additionally, when running this test, I used a grouped version of the dataframe `'merged'` that grouped by `'id'` so that each reciped was counted just once in this test. Many recipes had multiple rows due to having multiple reviews correlated with them, and we want to ensure there is no bias due to certain recipes being counted more times during this test. 

It was orignially proposed that vegan and vegetarian recipes have less protein than normal recipes because a primary source of protein in many recipes is animal products, which vegan recipes totally lack, and vegetarian recipes partially lack. The reuslting **p-value** from this test is **0.000**, which is below the significance level, leading us to **reject the null hypothesis** in favor of the alternate. This means that it is unlikely for us to observe the values we did if the null hypothesis is true, suggesting that vegan and vegetarian recipes could correlate with lower protein content. 

<iframe
  src="assets/ttest.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Framing a Prediction Problem

I am building a regression model predicting the protein content (PDV) in a recipe based on the tags assigned to it in the `'tags'` column. `'protein'` is the response variable in this model because it represents the continuous variable being estimated. I chose this as the response variable because protein is a nutritional attribute that many people consider when deciding what to eat, and it is essential to living a healthy lifestyle. The set of features comes form the `'tags'` column, which are easily accessible at the time of prediction. They will be processed using a CountVectorizer within a scikit-learn Pipeline in order to convert the text data into numeric features. The pipeline also predicts protein content using a linear regression model. Model performance is evaluated using mean squared error (MSE) as MSE penalizes larger errors more heavily. 

### Baseline Model

The baseline model is a lienar regression model that predicts the percent daily value of protein in a recipe based on the tags assigned to it. The tags are in the form of one long string with each tag seperated by a space, found in the `'tags_str'` column. This string of tags is the only feature of the model and it is nominal as it represents categorical labels rather than ordered or continuous values. I encoded this nominal feature using scikit-learn's `'CountVectorizer'` which performs a bag of words transformation and expands the tags into multiple bianry features indicating the presence or absence of each tag. The resulting feature matrix consists entirely of nominal, one-hot encoded features, and no quantitative or ordinal features were used. 

After fitting the model to the training data, I used **mean squared error (MSE)** to evaluate its performance. The MSE of the baseline model was **1737.5018** which indicates that the model's predictions are about 41.68% off on average. There is definitely room for the model's performance to imporve as the error is relatively large. To improve predictive performance, I plan on filtering the features to only include those that have a certain threshold numbe rof recipes that have then to avoid any sparse tags from skewing the model's predictions.

### Final Model