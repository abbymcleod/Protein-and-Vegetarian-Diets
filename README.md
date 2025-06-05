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

**Test Statistic:** The total variation difference (TVD) of average ratings between the group with missing ratings and without missing ratings.

**Significance Level:** 0.01

<iframe
  src="assets/tvd.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>