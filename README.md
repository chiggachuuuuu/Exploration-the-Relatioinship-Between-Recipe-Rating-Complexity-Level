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

<iframe src="assets/uni/fig_initial.html" width="800" height="600" frameborder="0" ></iframe>

<iframe src="assets/uni/fig_restricted.html" width="800" height="600" frameborder="0" ></iframe>

#### Distribution of Number of Steps (n_steps)

We will create a histogram to show the distribution of the number of steps for recipes.

<iframe src="assets/uni/fig_n_steps.html" width="800" height="600" frameborder="0" ></iframe>

#### Distribution of Number of Ingredients (n_ingredients)

We will create a histogram to show the distribution of the number of ingredients for recipes.

<iframe src="assets/uni/fig_n_ingredients.html" width="800" height="600" frameborder="0" ></iframe>

### Bivariate Analysis

Below, we have plot a bar plot relates to the distribution of ratings while conditional on time_group.

Overall, the distribution indicates a strong preference for recipes with high ratings across all cooking times, with minor variations. Shorter and moderate cooking time recipes slightly edge out in receiving 4-star ratings, while shorter cooking times have a negligible increase in lower ratings. This data suggests that the length of cooking time has a minimal impact on the overall rating distribution, with users generally favoring well-rated recipes regardless of how long they take to prepare.

<iframe src="assets/bi/fig_rate_on_time_group.html" width="800" height="600" frameborder="0" ></iframe>

Below, we also plot a scatter plot for the rating, conditional on complexity index. 

The plot suggests that there is no strong correlation between the complexity index and the rating. Recipes with both high and low complexity indices can receive high ratings, indicating that factors other than complexity may play a significant role in determining the recipe's rating.

<iframe src="assets/bi/fig_rate_on_com.html" width="800" height="600" frameborder="0" ></iframe>

### Interesting Aggregates

To undertand more about the relationship between rating and the cooking time. We create a scatter plot between the average rating over time. As a result, here does not exist a clear pattern between the two variable. However, one thing we noticed is that the average rating tends to various more as the cooking time increases. In other words, the average rating is more dispersive when the cooking time increases. Additonally, there exist more average rating 5 when the cooking time increases.

<iframe src="assets/IA/avg_rating_over_time.html" width="800" height="600" frameborder="0" ></iframe>

Another interesting pattern we find out is that, average calories over time also shows the same pattern as the average rating. The longer the cooking time is, the more dispersive the average calories is. It is still questionable whether there exist a relationship between average calories and average rating, or it is due to some confounding variables. 

<iframe src="assets/IA/avg_cal_over_time.html" width="800" height="600" frameborder="0" ></iframe>

<iframe src="assets/IA/avg_cal_over_time_ols.html" width="800" height="600" frameborder="0" ></iframe>



After undertanding the relationship between time, calories, and average ratings. We decide to compare the relationship between number of steps, minutes, and ratings. The purpose of this graph is to analyze and visualize the relationship between the number of steps and the preparation time of recipes with their average ratings. This can help in identifying patterns and trends regarding how the complexity and time investment in a recipe influence user satisfaction and ratings.

Overall, the graph suggests that recipes with fewer steps and shorter preparation times tend to receive higher ratings, while more complex recipes with longer preparation times show more variability in ratings. This conclusion also aligns with the previous several graphs we have visualized. 

|time_intervals|(0, 20]|(20, 40]|(40, 60]|(60, 80]| ... |(140, 160]|(160, 180]|(180, 200]|(200, 220]|
|       n_steps|       |        |        |        |     |          |          |          |          |
| :----------- | :---- | :----- | :----- | :----- | :-- | :------- | :------- | :------- | :------- |
|1             | 4.74  |  4.46  |  4.72  |  4.61  | ... |   NaN    |   3.00   |   4.11   |   4.89   |
|2             | 4.75  |  4.68  |  4.56  |  4.70  | ... |   4.97   |   5.00   |   4.66   |   4.25   |
|3             | 4.73  |  4.68  |  4.65  |  4.75  | ... |   4.14   |   4.86   |   4.62   |   5.00   |
|     ...      |  ...  |  ...   |  ...   |  ...   | ... |   ...    |   ...    |   ...    |   ...    |
|86            | 5.00  |  NaN   |  NaN   |  NaN   | ... |   NaN    |   NaN    |   NaN    |   NaN    |
|87            |  NaN  |  NaN   |  NaN   |  NaN   | ... |   NaN    |   NaN    |   5.00   |   NaN    |
|88            |  NaN  |  NaN   |  NaN   |  NaN   | ... |   NaN    |   3.00   |   NaN    |   NaN    |




<iframe src="assets/IA/fig_heatmap.html" width="800" height="600" frameborder="0" ></iframe>

## Assessment of Missingness

Three columns, '`date`', '`rating`', and '`review`', in the merged dataset have a significant amount of missing values, therefore, we need to assess the missingness on the dataframe.

**Notes**: Due to extreme values, the graph will have an extreme large x-axis and really bad data visualization if we don't rescale the x-axis. Therefore, for the graph belows, we restricted the x-axis maximum so that we can see a pattern of the graph. 

### NMAR Analysis

We believe that the missingness of the `review` column is NMAR. In the previous plots, we have seen that the reviews generally has a `rating` of 4 or 5 while the `rating` of 3 or below is very little. Therefore, it might be the case where if they like a recipe more, then they are more willing to leave a review. Otherwise, they might not be willing to leave a rating ore a review. Thus, it is hight likely that the missingness on the `'review'` column is NMAR, not affected by other columns but itself.


### Missingness Dependency

After determine the missingness of the `review` column. We now need to examine the missingness of the column `rating`. Since our research question is the complexity level for the recipe. We will check the distribution of missingness of `complexity_index`. Therefore, we will check whether the missiness in the `rating` column is depends on the `complexity_index` column and the `calories (#)` column to see if there's any relationship. 


#### Proportion of Complexity Index and Rating

**Null Hypothesis**: The missingness of ratings does not depend on the complexity index in the recipe.

**Alternate Hypothesis**: The missingness of ratings does depend on the complexity index in the recipe.

**Test Statistic**: The absolute difference of mean in the complexity index of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

<iframe src="assets/miss/come_on_rate.html" width="800" height="600" frameborder="0" ></iframe>
**Observed Statistic:** 2.0996452610910588

**P-value:** 0.122
<iframe src="assets/miss/kdeplot.html" width="800" height="600" frameborder="0" ></iframe>

#### Conclusion

Based on the results of the hypothesis test:

- **Observed Statistic**: The absolute difference in the mean complexity index between recipes with missing ratings and those with non-missing ratings is approximately 2.
- **P-value**: The p-value is approximately 0.12.

Given that our significance level is 0.05, we observe that the p-value (0.12) is much greater than 0.05. This means that we do not have enough evidence to reject the null hypothesis.

#### Interpretation

Our null hypothesis stated that the missingness of ratings does not depend on the complexity index of the recipe. Since the p-value is very high (0.12), we fail to reject the null hypothesis. This suggests that there is no significant evidence to support the claim that the missingness of ratings is dependent on the complexity index of the recipes. 

In simpler terms, the complexity index  does not appear to influence whether or not a recipe has a missing rating. The observed statistic's value being relatively low and the high p-value indicate that any difference in the complexity index between recipes with missing and non-missing ratings is likely due to random chance rather than a systematic difference.



#### Proportion of Calories and Rating

**Null Hypothesis**: The missingness of ratings does not depend on the calories in the recipe.

**Alternate Hypothesis**: The missingness of ratings does depend on the calories in the recipe.

**Test Statistic**: The absolute difference of mean in the calories of the distribution of the group without missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

<iframe src="assets/miss/cal_and_rate.html" width="800" height="600" frameborder="0" ></iframe>
**Observed Statistic:** 69.00722806375853

**P-value:** 0.0
<iframe src="assets/miss/hh.html" width="800" height="600" frameborder="0" ></iframe>

#### Conclusion

Based on the results of the hypothesis test:

- **Observed Statistic**: The absolute difference in the mean calories between recipes with missing ratings and those with non-missing ratings is approximately 69.
- **P-value**: The p-value is approximately 0.00.

Given that our significance level is 0.05, we observe that the p-value (0.00) is much lower than 0.05. This means that wehave enough evidence to reject the null hypothesis.

#### Interpretation

Our null hypothesis stated that the missingness of ratings does not depend on the complexity index of the recipe. Since the p-value is very low (0.00), we have enough evidence  to reject the null hypothesis. This suggests that there is significant evidence to support the claim that the missingness of ratings is dependent on the calories of the recipes. 

In simpler terms, the calories (calculated as the sum of preparation time and the number of steps) does not appear to influence whether or not a recipe has a missing rating. The observed statistic's value being relatively high and the low p-value indicate that any difference in the calories between recipes with missing and non-missing ratings is ot likely due to a systematic difference.


## Step 4: Hypothesis Testing

As stated in the beginning, We are interested in cooking and we often find recipes that are less redundant are easier to follow. Hence, we wonder how does the complexity level of recipes related to the user experience/ratings of recipes. 

**Null Hypothesis**: People rate all the recipes on the same scale, regardless of the number of steps.
- Null: The mean rating of recipes with <= 10 steps is equal to the mean rating of recipes with > 10 steps.

**Alternative Hypothesis**: People rate recipes with more than 10 steps higher than recipes with 10 or fewer steps.
- Alt: The mean rating of recipes with <= 10 steps is lower than the mean rating of recipes with > 10 steps.

**Test Statistic**: The difference in mean ratings between recipes with <= 10 steps and recipes with > 10 steps.

**Significance Level**: 0.05


### Reason for the Test

We performed this hypothesis test to investigate if the complexity level of recipes, as measured by the number of steps, has an impact on user ratings. Understanding this relationship can provide valuable insights for home cooks, who may prefer less complex recipes if they believe such recipes are rated equally well. It can also guide recipe developers in creating content that aligns with user preferences, potentially increasing user engagement and satisfaction. We proposed the two distributions (with less than 10 steps and more than 10 steps) have the same distribution because from previous plots, we discovered that there's no clear pattern between `ratings` and `n_steps`. 

<iframe src="assets/ht/ht.html" width="800" height="600" frameborder="0" ></iframe>
**Observed Statistic:** 0.0018707051790087803

**P-value:** 0.289


### Conclusion

Based on the hypothesis test conducted, the observed difference in mean ratings between recipes with more than 10 steps and those with 10 or fewer steps is 0.002. The p-value associated with this observed statistic around 0.29.

Given that the p-value (0.29) is significantly greater than the chosen significance level of 0.05, we fail to reject the null hypothesis. This indicates that there is no statistically significant evidence to suggest that the number of steps in a recipe (whether 10 or fewer, or more than 10) affects the mean rating given by users.

### Final Remarks

Our findings suggest that the complexity level, defined by the number of steps in a recipe, does not have a significant effect on user ratings. This implies that users rate recipes consistently regardless of their complexity level. This could encourage recipe developers to focus on various aspects of the recipes without worrying about the number of steps impacting the user ratings.



##  Framing a Prediction Problem

### Problem Identification

**Prediction Problem**: Predicting the calorie content of recipes based on various features such as preparation time, number of steps, ingredients, and other nutritional information.

**Type**: Regression

**Response Variable**: The response variable for this prediction problem is `calories`, which represents the number of calories in a recipe.

**Rationale**: 
- **Practical Relevance**: Predicting the calorie content of a recipe is important for individuals who are conscious of their dietary intake. It helps users make informed decisions about their meals, especially for those following specific calorie-restricted diets.

- **Feature Availability**: All the features used in the model are available at the time of prediction. The preparation time, number of steps, ingredients, and other nutritional information are known before calculating the calories, making them suitable for this prediction task.

### Features Used for Prediction

- **Potential Quantitative Features**:
  - `minutes`: The time required to prepare the recipe.
  - `n_steps`: The number of steps involved in the recipe.
  - `n_ingredients`: The number of ingredients used in the recipe.
  - `total_fat`: Total fat content (percentage of daily value).
  - `sugar`: Sugar content (percentage of daily value).
  - `sodium`: Sodium content (percentage of daily value).
  - `protein`: Protein content (percentage of daily value).
  - `saturated_fat`: Saturated fat content (percentage of daily value).
  - `carbohydrates`: Carbohydrates content (percentage of daily value).

- **Nominal Features**:
  - `tags`: Food.com tags for the recipe, encoded using one-hot encoding.

- **Text Features**:
  - `ingredients`: Text description of the ingredients, processed using hashing vectorization and dimensionality reduction.
  - `steps`: Text description of the steps, processed using hashing vectorization and dimensionality reduction.

### Model Evaluation Metrics

**Metrics**:
- **Root Mean Squared Error (RMSE)**: This metric is chosen because it provides a clear indication of the average error magnitude and penalizes larger errors more significantly than smaller ones. RMSE is widely used for regression problems and is intuitive to interpret as it is in the same units as the response variable (calories).
- **Mean Absolute Error (MAE)**: This metric is chosen to complement RMSE. It provides a straightforward interpretation of the average magnitude of errors without considering their direction. MAE is less sensitive to outliers compared to RMSE.

**Justification**: RMSE is preferred for its sensitivity to large errors, which can be crucial in predicting calorie content accurately. However, MAE is also reported to provide an additional perspective on the model's performance, offering a balance between understanding average error magnitude and sensitivity to outliers.

**Conclusion**: Through previous analysis, we find out that the complexity level of a recipe is not correlated with the ratings. However, through the hypothesis test, we find out that `ratings` may be depdend on the `calories`. Therefore, it is important for us to discovery any potential relationship between other elements and calories, therefore understanding better about the missingness about the `ratings`. Additionally, this model can provide valuable insights for users who are monitoring their dietary intake. 


##  Baseline Model

### Description

To develop a baseline model to predict the calorie content of recipes using various features from our dataset. We first decide to choose `minutes` and `total_fat` as our two fundamental features because minutes can represents how complex a recipe is, and in general, more fat will have a higher calories. Moreover, using `tag` as our additional nominal features can help us predict whether there's a specific catories of recipe that contains high calories. 

**Features in the Model**:
1. **Quantitative Features**:
   - `minutes`: Time required to prepare the recipe (Usually food required longer time to cook, such as deep fried food, contains more colories than those required shorter time like salads).
   - `total_fat`: Number of steps involved in the recipe (We can all agree that more fat result in more calories).

2. **Nominal Features**:
   - `tags`: Food.com tags for the recipe. We will use one-hot encoding to handle this categorical feature (Different tags may imply different level of calories of recipes).

### Model and Transformations
- **Numerical Features**: Left as-is, but standardized to ensure all features contribute equally to the model (For LinearRegression it is not needed, but we add such step for potential future improvment).
- **Categorical Features**: Encoded using OneHotEncoder to convert text data into a format suitable for regression.

### Evaluation Metrics
- **Root Mean Squared Error (RMSE)**: To assess the average magnitude of errors.
- **Mean Absolute Error (MAE)**: To provide a straightforward interpretation of average errors.


### Baseline Model Evaluation Summary

#### Model Performance Metrics:
- **Mean Absolute Error (MAE)**: 38.709830978781135
- **Root Mean Squared Error (RMSE)**: 177.9973950204717

#### Interpretation:

The Mean Absolute Error (MAE) of 38.709 indicates that, on average, the predicted calories for the recipes deviate from the actual calories by about 38.709 calories on the calories scale. This metric provides a straightforward interpretation of the average prediction error. 

The Root Mean Squared Error (RMSE) of 177.997 suggests that there are some larger errors present in our model's predictions, as RMSE is more sensitive to larger deviations between predicted and actual calories compared to MAE. 

#### Conclusion:

The baseline model, which uses a simple linear regression with `minutes`, `n_steps`, and `tags` as features, provides a reasonable starting point for predicting recipe calories. While the MAE and RMSE values show that the model has moderate prediction errors, there is significant room for improvement. These baseline performance metrics will be used as a benchmark to compare and evaluate more complex models that we will develop in subsequent steps. The goal will be to reduce both MAE and RMSE, thereby enhancing the model's prediction accuracy and reliability.


##  Final Model

### **Improved Model:**

1. **Features Used:**
   - **Quantitative:** `minutes`, `total_fat`, `protein`, `saturated_fat`, `n_ingredients`
     - `minutes`: Usually food required longer time to cook, such as deep fried food, contains more colories than those required shorter time like salads
     - `total_fat`: We can all agree that more fat result in more calories
     - `protein`: Protein is also a valid source of calories as they offer a lot of energy
     - `saturated_fat`: Saturated fat is also a reliable source of energy so they come with calories
     - `n_ingredients`: Usually the more ingredients the more calories
   - **Categorical:** `tags`, `ingredients`
     - `tags`: Different tags may imply different level of calories of recipes
     - `ingredients`: Different ingredients carry different levels of calories
   
2. **Preprocessing:**
   - **Numerical Features:** Used PolynomialFeatures and StandardScale to standarilized numerical features and find the best polynomial degree.
   - **Categorical Features:** Applied OneHotEncoder to handle multi-label categorical data.

3. **New Feature Engineering:**
   - **Complexity Index:** We added new features `n_steps` and `n_ingredients` as they can also represent the complexity of a recipe. Adding them can help reduce the error if their exist a relationship between complexity and recipe calories. 
   - **Nutrition Groups:** Adding more features related to nutritions like `protein` and `saturated_fat` because in our perceptives, food with more calories tends to have more protein and saturated fat. Therefore we choose to add them to find a higher correlation. 

4. **Model:** LinearRegression is still our model. And we use a GridSearchCV to search for the best hyperparameter. 

5. **Hyperparameter Tuning:**
   - **'model__fit_intercept':** True, False,

   - **'model__normalize':** True, False
   
   - **'preprocessor__num__poly__degree':** [1, 2, 3] 
   

### **Improved Model Results:**

#### Model Performance Metrics:
   - **Training MAE:** 25.203625969551442
   - **Testing RMSE:** 125.18996600029705

#### Hyperparameter Tuning Method:
   - **GridSearchCV:**  {'model__fit_intercept': True, 'model__normalize': False, 'preprocessor__num__poly__degree': 1}

#### Interpretation:

The Mean Absolute Error (MAE) of 25.204 indicates that, on average, the predicted calories for the recipes deviate from the actual calories by about 25.204 calories on the calories scale. This metric provides a straightforward interpretation of the average prediction error. 

The Root Mean Squared Error (RMSE) of 125.19 suggests that there are some larger errors present in our model's predictions, as RMSE is more sensitive to larger deviations between predicted and actual calories compared to MAE. 

#### Conclusion:

The model has shown a significant improvement in performance after incorporating additional features and utilizing grid search for hyperparameter tuning. 

Previously, the baseline model achieved a Mean Absolute Error (MAE) of approximately 38.71 and a Root Mean Squared Error (RMSE) of around 178.00 on the test data. However, the improved model has substantially reduced the MAE to about 25.20, indicating a notable enhancement in predictive accuracy. Moreover, the RMSE has been reduced to approximately 125.19, further affirming the model's improved ability to generalize to unseen data.

By including new features such as 'protein', 'saturated_fat', 'ingredients', 'n_steps', and 'n_ingredients', the model gained a richer representation of the data, potentially capturing more nuanced relationships between the predictors and the target variable. Additionally, the thorough exploration of hyperparameters through grid search enabled the model to identify the most suitable configuration, leading to enhanced performance.


## Fairness Analysis

### Hypothesis Test:
- **Groups**:
    - **Group X (Hotter Seasons)**: Rating Interactions occurring during the hotter seasons (April to September).
    - **Group Y (Colder Seasons)**: Rating Interactions occurring during the colder seasons (January, February, March, October, November, December).

- **Evaluation Metric**: Root Mean Squared Error (RMSE)

- **Null Hypothesis**:
    - The model's RMSE for predicting calories is not significantly different between hotter and colder seasons.

- **Alternative Hypothesis**:
    - The model's RMSE for predicting calories is significantly different between hotter and colder seasons.

- **Test Statistic**: Difference in RMSE between Group X and Group Y.

- **Significance Level**: 0.05

This adjusted analysis will determine whether the model's Root Mean Squared Error (RMSE) is significantly different between interactions occurring during hotter and colder seasons based on the provided data and the best model obtained through grid search.

### Reasons to Exlopre:
Exploring the predictive performance of the model across hotter and colder seasons can unveil insightful patterns and behaviors related to dietary habits, cooking preferences, and ingredient availability, among other factors.

1. **Consumer Behavior Analysis**: Understanding how dietary ratings fluctuate across different seasons can help researchers decipher consumer preferences and behavior. Analyzing patterns in rating trends may reveal seasonal variations in food satisfaction, ingredient preferences, or recipe popularity.

2. **Marketing and Product Development**: Food companies and culinary platforms can leverage seasonal insights to tailor marketing strategies and product offerings to meet consumer demands. By identifying seasonal trends in dietary ratings, businesses can develop targeted seasonal promotions, seasonal menu items, or seasonal recipe collections to attract and retain customers.

3. **Health and Nutrition Insights**: Exploring how dietary ratings change across seasons can offer insights into seasonal variations in nutritional choices and dietary habits. Researchers and healthcare professionals can use this information to develop seasonal nutrition guidelines, educate consumers about seasonal eating patterns, and promote healthy seasonal food choices.

<iframe src="assets/ft/ft.html" width="800" height="600" frameborder="0" ></iframe>
**Observed Difference in RMSE:** -18.139633273754427

**p-value:** 0.001

Result:
Reject the null hypothesis. There is evidence to suggest that the model's RMSE is significantly different between hotter and colder seasons.

### Conclusion

Based on the conducted analysis, the observed difference in RMSE between interactions occurring during hotter and colder seasons is -18.14. The p-value obtained from the permutation test is 0.0, which is less than the significance level of 0.05. Therefore, we reject the null hypothesis.

The statistical conclusion, stating that there is evidence to suggest that the model's RMSE is significantly different between hotter and colder seasons, implies that the model's predictive performance varies depending on the season in which the dietary ratings were made. This finding underscores the importance of considering seasonal variations when analyzing dietary data and developing predictive models. By acknowledging these seasonal differences, we can enhance the accuracy and relevance of our analyses and gain deeper insights into the complex interplay between seasonal factors and dietary preferences.