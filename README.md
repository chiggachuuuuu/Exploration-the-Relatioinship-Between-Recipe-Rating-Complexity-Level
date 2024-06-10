# Exploration of the Relationship between Recipe Rating and Complexity Level

Authors: Lin Tian & Jefferson Chen

## Overview

This is a data science project for DSC80 at UCSD focusing on exploring the relationship between recipe rating and the complexity level of given recipes.

## Introduction

Food is an indispensable portion of our daily life. On an average day in 2014, Americans aged 18 and over spent 37 minutes in food preparation and cleanup. According to the U.S. Bureau of Labor Statistics, people in the United States aged 15 and older spend an average of 53 minutes preparing food and drinks per day. Usually, people believe that a more exquisite and delicious diet would require more time to cook. However, it is questionable whether this is true. For home cooks, understanding whether quicker recipes are rated higher can help them choose dishes that are both time-efficient and likely to be enjoyed by their family and friends. For recipe developers and food bloggers, this analysis can guide the creation of content that aligns with user preferences, potentially increasing engagement and satisfaction.

Therefore, we decided to research **What is the relationship between the complexity level of the recipes and the recipes' ratings? (For the measurement of complexity level we specifically use the preparation time and the number of steps)**. If there exist such correlations to some degree, then this exploration can contribute to broader culinary research by identifying if there is a sweet spot for preparation time and the number of steps that optimizes both efficiency and user satisfaction.

The dataset we are working with includes recipes and their corresponding ratings. The data is split into two main CSV files: one containing recipe details (RAW_recipes.csv) and the other containing user interactions, specifically ratings and reviews (RAW_interactions.csv). Our subset focuses on entries from 2008 to 2018.

### 1st Dataset "recipes"

The first dataset, `recipes`

**Shape: 83782 rows, 12 columns**

**Data Included: The recipe-related data from 2008 to 2018.**

| Column             | Description                                                    |
| :----------------- | :--------------------------------------------------------------|
| `'name'`           | Recipe name                                                    |
| `'id'`             | Recipe ID                                                      |
| `'minutes'`        | Minutes to prepare recipe                                      |
| `'contributor_id'` | User ID who submitted this recipe                              |
| `'submitted'`      | Date recipe was submitted                                      |
| `'tags'`           | Food.com tags for recipe                                       |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                      |
| `'steps'`          | Text for recipe steps, in order                                |
| `'description'`    | User-provided description                                      |
| `'ingredients'`    | Text for recipe ingredients                                    |
| `'n_ingredients'`  | Number of ingredients in recipe                                |


|    |                              name|    id|minutes| ... |                                       ingredients|n_ingredients|
| :- | :------------------------------- | :--- | :---- | :-- | :----------------------------------------------- | :---------- |
|0   |1 brownies in the world best ever |333281|   40  | ... |['bittersweet chocolate', 'unsalted butter', '...]|      9      |
|1   |1 in canada chocolate chip cookies|453467|   45  | ... |['white sugar', 'brown sugar', 'salt', 'margar...]|     11      |
|2   |412 broccoli casserole            |306168|   40  | ... |['frozen broccoli cuts', 'cream of chicken sou...]|      9      |
|3   |millionaire pound cake            |286009|  120  | ... |['butter', 'sugar', 'eggs', 'all-purpose flour...]|      7      |
|4   |2000 meatloaf                     |475785|   90  | ... |['meatloaf mixture', 'unsmoked bacon', 'goat c...]|     13      |

### 2nd Dataset "ratings"

The first dataset, `interactions`

**Shape: 731927 rows, 5 columns**

**Data Included: The rating's corresponding recipe-related data**

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |


|    |    user_id| recipe_id|       date| rating|                                           review|
| :- | :-------- | :------- | :-------- | :---- | :---------------------------------------------- |
|0   |    1293707|     40893| 2011-12-21|   5   |So simple, so delicious! Great for chilly fall...|
|1   |     126440|     85009| 2010-02-27|   5   |I made the Mexican topping and took it to bunk...|
|2   |      57222|     85009| 2011-10-01|   5   |Made the cheddar bacon topping, adding a sprin...|
|3   |     124416|    120345| 2011-08-06|   0   | Just an observation, so I will not rate. I fo...|
|4   | 2000192946|    120345| 2015-05-10|   2   | This recipe was OVERLY too sweet. I would sta...|


As we are investing with the given datasets, we must find a way to calculate the complexity of a recipe. Within the columns, `minutes`, `n_steps`, and `n_ingredients` relate to our research question the most. Additionally, the values inside the `nutrition` column, such as `calories (#)` might also be helpful for future calculation, when determining the relationship between the recipe complexity and calories. Therefore, we need to separate all the sub-values within the `nutrition` column into new columns. We also need to set the cooking time group for each unique recipe. A recipe that spends less than 30 minutes would be in group "Short", 30-60 minutes would be in group "Middle", and more than 60 minutes would be in group "Long". We will add this information as the `time_group` column. 

For the simplicity of future calculation, we must set up a definition of how to determine the complexity of a recipe. The easiest way to determine this would be 

  `complexity_index` =  `minutes` / (`n_steps` + `n_ingredients`)
  

It can help us find the average time it takes for each step and ingredient. If it is larger, meaning the time for each step and ingredient would be larger, and the opposite otherwise. 

By calculating the complexity index, we simplify the complexity analysis into a manageable and insightful metric. This not only aids in our investigation of how complexity affects recipe ratings but also provides valuable insights for recipe creators and users alike. The complexity index, therefore, is a crucial tool for understanding and optimizing the relationship between recipe complexity and user satisfaction.

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To process our analysis, we need to merge the `recipes` and `ratings` datasets together. Since not all recipes have datasets, and we do want to maintain all the recipes, we should do a left merge.

|    |                              name|    id|minutes| ... |rating|                                           review|
| :- | :------------------------------- | :--- | :---- | :-- | :--- | :---------------------------------------------- |
|0   |1 brownies in the world best ever |333281|   40  | ... | 4.0  |These were pretty good, but took forever to ba...|
|1   |1 in canada chocolate chip cookies|453467|   45  | ... | 5.0  |Originally I was gonna cut the recipe in half ...|
|2   |412 broccoli casserole            |306168|   40  | ... | 5.0  |This was one of the best broccoli casseroles t...|
|3   |412 broccoli casserole            |306168|   40  | ... | 5.0  |I made this for my son's first birthday party ...|
|4   |412 broccoli casserole            |306168|   40  | ... | 5.0  |Loved this. Be sure to completely thaw the br ...|


Here's a description of what the merged dataframe contains

| Column             | Data Type   | Description                                                    |
| :----------------- | :-----------| :--------------------------------------------------------------|
| `'name'`           | String      | Recipe name                                                    |
| `'id'`             | Integer     | Recipe ID                                                      |
| `'minutes'`        | Integer     | Minutes to prepare recipe                                      |
| `'contributor_id'` | Integer     | User ID who submitted this recipe                              |
| `'submitted'`      | Date        | Date recipe was submitted                                      |
| `'tags'`           | String/List | Food.com tags for recipe                                       |
| `'nutrition'`      | String/List | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”|
| `'n_steps'`        | Integer     | Number of steps in recipe                                      |
| `'steps'`          | String/List | Text for recipe steps, in order                                |
| `'description'`    | String      | User-provided description                                      |
| `'ingredients'`    | String/List | Text for recipe ingredients                                    |
| `'n_ingredients'`  | Integer     | Number of ingredients in recipe                                |
| `'user_id'`        | Integer     | User ID                                                        |
| `'date'`           | Date        | Date of interaction                                            |
| `'rating'`         | Float       | Rating given                                                   |
| `'review'`         | String      | Review text                                                    |

After merging, we need to fill all the rating values of 0 with `np.nan`. It is valid because all rating values should fall in between 1 and 5. A rating value of 0 means that the user did not rate the recipe. Therefore, we can set it to NaN as it should not be counted.

Then, add a new column '`average`' containing the average rating for each unique recipe.

Each unique recipe might have multiple reviews and ratings. Since a valid rating is from 1-5, we can average all ratings to have a more precise rating for each unique recipe. 

|    |                              name|    id|minutes| ... |rating|                                           review|average|
| :- | :------------------------------- | :--- | :---- | :-- | :--- | :---------------------------------------------- | :---- |
|0   |1 brownies in the world best ever |333281|   40  | ... | 4.0  |These were pretty good, but took forever to ba...|  4.0  |
|1   |1 in canada chocolate chip cookies|453467|   45  | ... | 5.0  |Originally I was gonna cut the recipe in half ...|  5.0  |
|2   |412 broccoli casserole            |306168|   40  | ... | 5.0  |This was one of the best broccoli casseroles t...|  5.0  |
|3   |412 broccoli casserole            |306168|   40  | ... | 5.0  |I made this for my son's first birthday party ...|  5.0  |
|4   |412 broccoli casserole            |306168|   40  | ... | 5.0  |Loved this. Be sure to completely thaw the br ...|  5.0  |


Add new column '`complexity_index`' containing the complexity level for each unique recipe. As mentioned previously, the formula for calculating the complexity level would be 

`complexity_index` = `minutes` / `n_steps` + `n_ingredients` 

This formula helps us find the average time it takes for each step and the total number of ingredients. If the complexity index is larger, it means that either the time per step or the number of ingredients (or both) is relatively high, indicating a more complex recipe. Conversely, a smaller complexity index suggests a simpler recipe in terms of preparation time and ingredient count.

|    |                              name|    id| ... |                                           review|average|complexity_index|
| :- | :------------------------------- | :--- | :-- | :---------------------------------------------- | :---- | :------------- |
|0   |1 brownies in the world best ever |333281| ... |These were pretty good, but took forever to ba...|  4.0  |      2.11      |
|1   |1 in canada chocolate chip cookies|453467| ... |Originally I was gonna cut the recipe in half ...|  5.0  |      1.96      |
|2   |412 broccoli casserole            |306168| ... |This was one of the best broccoli casseroles t...|  5.0  |      2.67      |
|3   |412 broccoli casserole            |306168| ... |I made this for my son's first birthday party ...|  5.0  |      2.67      |
|4   |412 broccoli casserole            |306168| ... |Loved this. Be sure to completely thaw the br ...|  5.0  |      2.67      |

Now we will add the '`time_group`' column. The recipe with time <= 30 minutes would belong to the "Short" group, 30 < time < 60 would belong to the "Middle" group, and the recipe that takes >= 60 minutes would belong to the "Long" group

|    |                              name|    id|contributor_id| ... |average|complexity_index|time_group|
| :- | :------------------------------- | :--- | :----------- | :-- | :---- | :------------- | :------- |
|0   |1 brownies in the world best ever |333281|     985201   | ... |  4.0  |      2.11      |  Middle  |
|1   |1 in canada chocolate chip cookies|453467|     1848091  | ... |  5.0  |      1.96      |  Middle  |
|2   |412 broccoli casserole            |306168|     50969    | ... |  5.0  |      2.67      |  Middle  |
|3   |412 broccoli casserole            |306168|     50969    | ... |  5.0  |      2.67      |  Middle  |
|4   |412 broccoli casserole            |306168|     50969    | ... |  5.0  |      2.67      |  Middle  |

Lastly, we will pull out the '`calories (#)`' from the `nutrition` column for the potential future use. 

|    |                              name|    id|contributor_id| ... |average|complexity_index|time_group|calories (#)|
| :- | :------------------------------- | :--- | :----------- | :-- | :---- | :------------- | :------- | :--------- |
|0   |1 brownies in the world best ever |333281|     985201   | ... |  4.0  |      2.11      |  Middle  |    138.4   |
|1   |1 in canada chocolate chip cookies|453467|     1848091  | ... |  5.0  |      1.96      |  Middle  |    595.1   |
|2   |412 broccoli casserole            |306168|     50969    | ... |  5.0  |      2.67      |  Middle  |    194.8   |
|3   |412 broccoli casserole            |306168|     50969    | ... |  5.0  |      2.67      |  Middle  |    194.8   |
|4   |412 broccoli casserole            |306168|     50969    | ... |  5.0  |      2.67      |  Middle  |    194.8   |


### Univariate Analysis

#### Distribution of Cooking Time

To understand the distribution of the cookiing time, we need to first plot the histogram for "`minutes`"

However, after plotting the histogram, we can see that the range of time various significantly from 0 to 1.2 million miniutes. However, most of the datapoints lies from 0 to 5000. In order to have a more general undertanding about the distribution, we decide to regenrate the plot that only display the distribution of time from 0 to 225 miniutes, as 95% of the datapoints fall within 0 to 255 minutes of cooking time. The rest 5% of the datapoints is highly biased with thousands of minitues of cooking time. Therefore, for the rest of the plot that's related to the cooking time, we decide to only plot the cooking time that's less than 225 minutes. 




#### Distribution of Number of Steps (n_steps)
We will create a histogram to show the distribution of the number of steps for recipes.


#### Distribution of Number of Ingredients (n_ingredients)

We will create a histogram to show the distribution of the number of ingredients for recipes.



### Bivariate Analysis

Below, we have plot a bar plot relates to the distribution of ratings while conditional on time_group.

Overall, the distribution indicates a strong preference for recipes with high ratings across all cooking times, with minor variations. Shorter and moderate cooking time recipes slightly edge out in receiving 4-star ratings, while shorter cooking times have a negligible increase in lower ratings. This data suggests that the length of cooking time has a minimal impact on the overall rating distribution, with users generally favoring well-rated recipes regardless of how long they take to prepare.




Below, we also plot a scatter plot for the complexity index, conditional on complexity index. 

The plot suggests that there is no strong correlation between the complexity index and the rating. Recipes with both high and low complexity indices can receive high ratings, indicating that factors other than complexity may play a significant role in determining the recipe's rating.



### Interesting Aggregates

To undertand more about the relationship between rating and the cooking time. We create a scatter plot between the average rating over time. As a result, here does not exist a clear pattern between the two variable. However, one thing we noticed is that the average rating tends to various more as the cooking time increases. In other words, the average rating is more dispersive when the cooking time increases. Additonally, there exist more average rating 5 when the cooking time increases.




Another interesting pattern we find out is that, average calories over time also shows the same pattern as the average rating. The longer the cooking time is, the more dispersive the average calories is. It is still questionable whether there exist a relationship between average calories and average rating, or it is due to some confounding variables. 





After undertanding the relationship between time, calories, and average ratings. We decide to compare the relationship between number of steps, minutes, and ratings. The purpose of this graph is to analyze and visualize the relationship between the number of steps and the preparation time of recipes with their average ratings. This can help in identifying patterns and trends regarding how the complexity and time investment in a recipe influence user satisfaction and ratings.

Overall, the graph suggests that recipes with fewer steps and shorter preparation times tend to receive higher ratings, while more complex recipes with longer preparation times show more variability in ratings. This conclusion also aligns with the previous several graphs we have visualized. 







## Assessment of Missingness

Three columns, '`date`', '`rating`', and '`review`', in the merged dataset have a significant amount of missing values, therefore, we need to assess the missingness on the dataframe.

**Notes**: Due to extreme values, the graph will have an extreme large x-axis and really bad data visualization if we don't rescale the x-axis. Therefore, for the graph belows, we restricted the x-axis maximum so that we can see a pattern of the graph. 

### NMAR Analysis
We believe that the missingness of the `review` column is NMAR. In the previous plots, we have seen that the reviews generally has a rating of 4 or 5 while the ratings of 3 or below is very little. Therefore, it might be the case where if they like a recipe more, then they are more willing to leave a review. Otherwise, they might not be willing to leave a rating ore a review. Thus, it is hight likely that the missingness on the 'review' column is NMAR, not affected by other columns but itself.


### Missingness Dependency

After determine the missingness of the `review` column. We now need to examine the missingness of the column `rating`. Since our research question is the complexity level for the recipe. We will check the distribution of missingness of `complexity_index`. Therefore, we will check whether the missiness in the `rating` column is depends on the `complexity_index` column and the `calories (#)` column to see if there's any relationship. 



#### Proportion of Complexity Index and Rating

**Null Hypothesis**: The missingness of ratings does not depend on the complexity index in the recipe.

**Alternate Hypothesis**: The missingness of ratings does depend on the complexity index in the recipe.

**Test Statistic**: The absolute difference of mean in the complexity index of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level: 0.05**