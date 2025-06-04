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

To begin, I sought to streamline my analysis by cleaning the datasets as follows:

1. Left merging the `recipes` and `interactions` datasets together
2. 
