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

The first five rows of `'merged'` is displayed below.

| id | minutes | contributor_id | submitted | tags | nutrition | n_steps | ingredients | n_ingredients | user_id | rating | avg_rating | protein | tags_str |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 333281 | 40 | 985201 | 2008-10-27 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings'] | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0] | 10 | ['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour'] | 9 | 386585.0 | 4.0 | 4.0 | 3.0 | 60-minutes-or-less time-to-make course main-ingredient preparation for-large-groups desserts lunch snacks cookies-and-brownies chocolate bar-cookies brownies number-of-servings |
| 453467 | 45 | 1848091 | 2011-04-11 | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings'] | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0] | 12 | ['white sugar', 'brown sugar', 'salt', 'margarine', 'eggs', 'vanilla', 'water', 'all-purpose flour', 'whole wheat flour', 'baking soda', 'chocolate chips'] | 11 | 424680.0 | 5.0 | 5.0 | 13.0 | 60-minutes-or-less time-to-make cuisine preparation north-american for-large-groups canadian british-columbian number-of-servings |
| 306168 | 40 | 50969 | 2008-05-30 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli'] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions'] | 9 | 29782.0 | 5.0 | 5.0 | 22.0 | 60-minutes-or-less time-to-make course main-ingredient preparation side-dishes vegetables easy beginner-cook broccoli |
| 306168 | 40 | 50969 | 2008-05-30 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli'] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions'] | 9 | 1196280.0 | 5.0 | 5.0 | 22.0 | 60-minutes-or-less time-to-make course main-ingredient preparation side-dishes vegetables easy beginner-cook broccoli |
| 306168 | 40 | 50969 | 2008-05-30 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli'] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions'] | 9 | 768828.0 | 5.0 | 5.0 | 22.0 | 60-minutes-or-less time-to-make course main-ingredient preparation side-dishes vegetables easy beginner-cook broccoli |

The resulting DataFrame contains multiple rows for one recipe if the recipe had multiple reviews mapped to it. As I will not be using the data in the reviews column during my analysis, I decided to create a DataFrame called `'grouped'` where I grouped `'merged'` by `'id'` so that each recipe is represented in just one row. Additionally, I dropped the columns that were not consistent across all instances of that recipe as they were not pertinant to my analysis. These columns included `'rating'`, `'user_id'`,`'submitted'`, and `'contributor_id'`.This step reduces bias as recipes with many reviews will not be weigthed higher than those with fewer reviews in the analysis. Going forward, I will mainly be using the resulting DataFrame. The first five rows of `'grouped'` are shown below.

| minutes | tags | nutrition | n_steps | ingredients | n_ingredients | avg_rating | protein | tags_str |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 50 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'main-dish', 'eggs-dairy', 'pasta', 'easy', 'cheese', 'dietary', 'high-calcium', 'high-in-something', 'pasta-rice-and-grains', 'elbow-macaroni'] | [386.1, 34.0, 7.0, 24.0, 41.0, 62.0, 8.0] | 11 | ['cheddar cheese', 'macaroni', 'milk', 'eggs', 'bisquick', 'salt', 'red pepper sauce'] | 7 | 3.0 | 41.0 | 60-minutes-or-less time-to-make course main-ingredient preparation main-dish eggs-dairy pasta easy cheese dietary high-calcium high-in-something pasta-rice-and-grains elbow-macaroni |
| 55 | ['60-minutes-or-less', 'time-to-make', 'course', 'preparation', 'healthy', 'pies-and-tarts', 'desserts', 'pies', 'dietary'] | [377.1, 18.0, 208.0, 13.0, 13.0, 30.0, 20.0] | 6 | ['rhubarb', 'eggs', 'bisquick', 'butter', 'salt', 'sugar', 'vanilla', 'milk'] | 8 | 3.0 | 13.0 | 60-minutes-or-less time-to-make course preparation healthy pies-and-tarts desserts pies dietary |
| 45 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'very-low-carbs', 'main-dish', 'eggs-dairy', 'seafood', 'crab', 'cheese', 'dietary', 'low-sodium', 'low-calorie', 'low-carb', 'low-in-something', 'shellfish'] | [326.6, 30.0, 12.0, 27.0, 37.0, 51.0, 5.0] | 7 | ['frozen crabmeat', 'sharp cheddar cheese', 'cream cheese', 'onion', 'milk', 'bisquick', 'eggs', 'salt', 'nutmeg'] | 9 | 3.0 | 37.0 | 60-minutes-or-less time-to-make course main-ingredient preparation very-low-carbs main-dish eggs-dairy seafood crab cheese dietary low-sodium low-calorie low-carb low-in-something shellfish |
| 45 | ['60-minutes-or-less', 'time-to-make', 'course', 'preparation', 'occasion', 'desserts', 'cheesecake', 'gifts', 'taste-mood', 'sweet'] | [577.7, 53.0, 149.0, 19.0, 14.0, 67.0, 21.0] | 11 | ['apple pie filling', 'graham cracker crust', 'cream cheese', 'sugar', 'vanilla', 'eggs', 'caramel topping', 'pecan halves', 'pecans'] | 9 | 5.0 | 14.0 | 60-minutes-or-less time-to-make course preparation occasion desserts cheesecake gifts taste-mood sweet |
| 25 | ['lactose', '30-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'cuisine', 'preparation', 'occasion', 'south-west-pacific', 'desserts', 'fruit', 'australian', 'easy', 'beginner-cook', 'dinner-party', 'summer', 'dietary', 'gluten-free', 'seasonal', 'egg-free', 'free-of-something', 'pears', 'taste-mood', 'sweet'] | [386.9, 0.0, 347.0, 0.0, 1.0, 0.0, 33.0] | 8 | ['midori melon liqueur', 'water', 'caster sugar', 'cinnamon stick', 'vanilla pod', 'lemon rind', 'orange rind', 'pear', 'mint'] | 9 | 5.0 | 1.0 | lactose 30-minutes-or-less time-to-make course main-ingredient cuisine preparation occasion south-west-pacific desserts fruit australian easy beginner-cook dinner-party summer dietary gluten-free seasonal egg-free free-of-something pears taste-mood sweet |

#### Univariate Analysis

I examined the distribution of protein content (PDV) in a recipe. The plot below shows that protein content is skewed to the right, suggesting that most recipes in the dataset have under 50% of daily protein intact in them, with some outliers containing higher amounts of protein. 

<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Bivariate Analysis

To start off, I suspected there must be a positive relationship between the variables `'n_steps'` and `'n_ingredients'`, so I created a scatter plot with a trend line to investigate. The plot below shows that there is a positive relationship between these two variables, but it also looks like there is a decent amount of variance that should be investigated further without drawing any conclusions.   

<iframe
  src="assets/bivariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Interesting Aggregates

To further explore the relationship between number of steps and number of ingredients, I created a pivot table, shown below, that groups recipes into bins based on the number of ingredients they require, and shows the average number of steps, as well as the average rating of each bin. The average steps column reinforces the belief that the number of steps and number of ingredients in a recipe are correlated positively. Until the largest bin, there is a substantial increase in average steps as the number of ingredients increase. There is less of an evident relationship between number of ingredients and rating in this table. However, the last two bins, representing recipes with many steps, do show a significant increase in average rating. This could be due to more complex, advanced recipes involving many steps having higher satisfaction ratings due to their complexity. 

| Ingredient Bin | Average Rating | Average Steps |
| --- | --- | --- |
| 0-5 | 4.648165910532388 | 6.558246258119175 |
| 6-10 | 4.616680318619963 | 9.253105254306513 |
| 11-15 | 4.622962222543363 | 12.377772906619903 |
| 16-20 | 4.634880667436774 | 16.113949355841847 |
| 21-25 | 4.6834991435793265 | 19.464594127806564 |
| 26-30 | 4.776386820934078 | 24.141304347826086 |
| 31+ | 5.0 | 20.666666666666668 |

## Assessment of Missingness

The columns `'date'`, `'rating'`, and `'review'` have a significant amount of missing values in the `merged` dataset, so I will assess the missingness of these variables in `merged`. 

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

## Hypothesis Testing

Through a hypothesis test, I am trying to discover whether there is a relationship between protein content, found in column `'protein'`as percent daily value (PDV), and whether a recipe is vegan or vegetarian, as evidenced by whether they are tagged with 'vegan' or 'vegetarian'. The details of the two sample t test are as follows:

**Null Hypothesis:** There is no differnece in protein content (PDV) in recipes tagged 'vegan' or 'vegetarian' vs those without the tags. 

**Alternative Hypothesis:** Recipes tagged 'vegan' or 'vegetarian' have lower protein (PDV) than those not tagged.

**Test Statistic:** The difference in mean between vegetarian/vegan recipes and normal recipes, divided by the standard error. \( t = \frac{\text{difference in means}}{\text{standard error (SE)}} \)

**Significance Level:** 0.01

The student's t-test is the appropriate test because the predictor (veg vs no veg) is categorical, and we are looking to discover whether the means of these two groups are statistically different, which is what the t-test is designed to do. There is also a decently large sample size, meaning the central limit theorem allows us to assume normality.  

Additionally, when running this test, I used a grouped version of the dataframe `'merged'` that grouped by `'id'` so that each reciped was counted just once in this test. Many recipes had multiple rows due to having multiple reviews correlated with them, and we want to ensure there is no bias due to certain recipes being counted more times during this test. 

It was orignially proposed that vegan and vegetarian recipes have less protein than normal recipes because a primary source of protein in many recipes is animal products, which vegan recipes totally lack, and vegetarian recipes partially lack. The reuslting **p-value** from this test is **0.000**, which is below the significance level, leading us to **reject the null hypothesis** in favor of the alternate. This means that it is unlikely for us to observe the values we did if the null hypothesis is true, suggesting that vegan and vegetarian recipes could correlate with lower protein content. 

The graph below demonstrates just how different the averages between the two groups are with an 18.43% differnece on average.

<iframe
  src="assets/ttest.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Framing a Prediction Problem

I am building a regression model predicting the protein content (PDV) in a recipe based on the tags assigned to it in the `'tags'` column. `'protein'` is the response variable in this model because it represents the continuous variable being estimated. I chose this as the response variable because protein is a nutritional attribute that many people consider when deciding what to eat, and it is essential to living a healthy lifestyle. The set of features comes form the `'tags'` column, which are easily accessible at the time of prediction. They will be processed using a CountVectorizer within a scikit-learn Pipeline in order to convert the text data into numeric features. The pipeline also predicts protein content using a linear regression model. Model performance is evaluated using mean squared error (MSE) as MSE penalizes larger errors more heavily. 

### Baseline Model

The baseline model is a lienar regression model that predicts the percent daily value of protein in a recipe based on the tags assigned to it and minutes. The tags are in the form of one long string with each tag seperated by a space, found in the `'tags_str'` column, and minutes come from `'minutes'`. This string of tags is nominal as it represents categorical labels rather than ordered or continuous values while minutes is a numerical quantitative feature. I encoded the nominal feature using scikit-learn's `'CountVectorizer'` which performs a bag of words transformation and expands the tags into multiple bianry features indicating the presence or absence of each tag. The resulting feature matrix consists entirely of nominal, one-hot encoded features, and no quantitative or ordinal features were used. 

After fitting the model to the training data, I used **mean squared error (MSE)** to evaluate its performance. The MSE of the baseline model was **1732.9666** which indicates that the model's predictions are about 41.68% off on average. There is definitely room for the model's performance to imporve as the error is relatively large. To improve predictive performance, I plan on filtering the features to only include those that have a certain threshold number of recipes that have then to avoid any sparse tags from skewing the model's predictions, and to explore what other variables could have a positive impact on its performance.

### Final Model

The final model contained new features calories per minute, and steps per ingredient, and also only included a refined list of relevant tags. 

#### Tags

Instead of using the entire population of unique tags which was 549, only tags that appeared in over 2000 recipes were used, resulting in a set of **106** tags. This was a necessary step because there were hundreds of tags such as `'irish-st-patricks-day'` that had only one, or close to one use, and therefore wouldn't be of much use in the prediction model.

#### Calories per Minute

This feature was calculated by extracting the calorie count from the `'nutrition'` column and dividing it by the total cooking time in the column `'minutes'` plus one to avoid division by zero. Recipes with higher amounts of protein are likely meat heavy, as illustrated by the hypothesis test from earlier, which can correlate with higher calorie count. Sorter cooking times can also go hand in hand with higher protein due to a variety of short methods of cooking meats such as grilling or searing. By normalizing calorie count by cooking time, recipes that are efficient in delivering calories, and possibly also protein, are captured. 

#### Steps per Ingredient

The steps per ingredient feature was calculated as the number of steps, `'n_steps'`, divided by the number of ingredients, `'n_ingredients'`, plus one. This feature highlights the complexity of a recipe as recipes with more steps might require more advnaced techniques. More advanced techniques could be associated with sophisticated recipes that include multiple protein rich components.

These features were chosen because protein content is not based solely on the textual tags, but also by the complexity of preperation and nutrient profile. Both new features reference the process of making a dish, breaking down all of its steps and components, rather than just the tags that are assigned to the dish after the fact. 

#### Modeling and Hyperparameters

The chosen model was **RandomForestRegressor**. Random forests are esspecially strong for capturing non linear relationships and interactions between features, and they also are good at handling mixed data types. Additionally, random forests are robust to noisy data and less likely to overfit data compared to a single decision tree.

The **hyperparameters** tuned are as follows:
  - `n_estimators`: This is the number of trees in the forest which was tuned to balance model stability and runtime.
  - `max_depth`: This hyperparameter controls tree depth which helped to avoid overfitting.
  - `min_samples_split`: This is the minimum number of samples required to split an internal node, which helps control model complexity.

Out of all the hyperparameters used, the best were `{'regressor__max_depth': 10, 'regressor__n_estimators': 100, 'regressor__min_samples_split': 2}`. This combination does the best job at capturing enough complexity to model the engineered features and target variable while avoiding overfitting. To select the best hyperparameters, I used `GridSearchCV` with 5 fold cross validation on the training data which selected the mdoel with the lowest MSE on validation folds.

#### Performance

The final model had a MSE of **982.58** which is a significant improvement from the baseline model with a MSE of 1737.5018. This value means that, on average, the final model's prediction is off by about 31.35% daily value. This imporvement can be attributed to the model capturing richer interactions with the addition of the engineered numerical features.

### Fairness Analysis

For the assessment of fairness, I chose to look at the number of steps a recipe has, comparing shorter recipes with longer ones.
  - **Group X**: recipes with less than 10 steps
  - **Group Y**: rceipes with 10 or more steps

**Null Hypothesis**: The model is fair. It's RMSE is the sanme for recipes with less than 10 steps and recipes with 10 or more steps. Any observed difference is due to just random chance.

**Alternate Hypothesis**: The model is not fair. The RMSE is higher for recipes with less than 10 steps compared to recipes with 10 or more steps.

**Test Statistic**: root mean squared error (RMSE) of short recipes - RMSE of long recipes
  - Difference in precision = RMSEx - RMSEy

**Significance Level**: 0.01

To start the analysis, I split the recipes into two groups: short recipes and long recipes. Short recipes were designated to be ones where `n_steps` <9 and long recipes were designated to be ones where `n_steps` >= 9. Nine is the median number of steps in the dataset, which is why it was chosen to be the cutoff between high and low, and the median is robust to any outliers and guarantees a whole number result in this case. I evaluated the precision parity of the model because I wanted to test against false positives as to not mislead users with incorrect labels, and therefore discourage users from trying certain rceipes.

To begin, I calculated the RMSE for both group X and group Y to find an observed test statistic of **-2.98**. Then, I shuffled the group labels 1000 times and recalculated the RMSE of each shuffled group to construct a distribution of differences under the null. The resulting p-value was **0.7270**, which is higher than the significance level of 0.01, leading me to **fail to reject the null hypothesis** that the model is fair. According to these reuslts, there is no significant difference between the model's RMSE for recipes of fewer steps and recipes of more steps. 

