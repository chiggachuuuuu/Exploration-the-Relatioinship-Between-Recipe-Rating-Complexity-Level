# Exploration of the Relationship between Recipe Rating and Complexity Level

Authors: Lin Tian & Jefferson Chen

## Overview

This is a data science project for DSC80 at UCSD focusing on exploring the relationship between recipe rating and complexity level of given recipes.

## Introduction

Food is an indepensible portion of our daily life. On an average day in 2014, Americans age 18 and over spent 37 minutes in food preparation and cleanup. According to the U.S. Bureau of Labor Statistics, people in the United States aged 15 and older spend an average of 53 minutes preparing food and drinks per day. Usually, people believe that a more exquisite and delicious diet would require more time to cook. However, it is questionable whether this is true. For home cooks, understanding whether quicker recipes are rated higher can help them choose dishes that are both time-efficient and likely to be enjoyed by their family and friends. For recipe developers and food bloggers, this analysis can guide the creation of content that aligns with user preferences, potentially increasing engagement and satisfaction.

Therefore, we decide to research **What is the relationship between the complexity level of the recipes and the recipes' ratings? (For the measurement of complexity level we specifically use the preparation time and the number of steps)**. If there exist such correlations in some degree, then this exploration can contribute to broader culinary research by identifying if there is a sweet spot for preparation time and the number of steps that optimizes both efficiency and user satisfaction.

The dataset we are working with includes recipes and their corresponding ratings. The data is split into two main CSV files: one containing recipe details (RAW_recipes.csv) and the other containing user interactions, specifically ratings and reviews (RAW_interactions.csv). Our subset focuses on entries from 2008 to 2018.

### 1st Dataset "recipes"

The first dataset, `recipe`

**Shape: 83782 rows, 12 columns**

**Data Included: The recipes related data from 2008 to 2018.**

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

### 2nd Dataset "ratings"

**Shape: 731927 rows, 5 columns**

**Data Included: The rating's corresponding recipe related data**

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

As we are investing with the given datasets, we must find a way to calculate the complexity of a recipe. Within the columns, `minutes`, `n_steps`, `n_ingredients` relates to our research question the most. Additionally, the values inside the `nutrition` column, such as `calories (#)` might also be helpful for future calculation, when determining the relationship between the recipe complexity and calories. Therefore, we need to separate all the sub values within the `nutrition` column into new columns. We also need to set the cooking time group for each unique recipe. A recipe that spends less than 30 minutes would be in group "Short", 30-60 minutes would be in group "middle", more than 60 minutes would be in group "Long". We will add this information as the `time_group` column. 

For the simplicity of future calculation, we must set up a definition of how to determine the complexity of a recipe. The easiest way to determine this would be 

**$\text{complexity index} = \frac{\text{minutes}}{\text{number of steps}+ \text{number of ingredients}} $** 

It can help us find the average time it takes for each step and ingredient. If it is larger, meaning the time for each step and ingredient would be larger, and opposite otherwise. 

By calculating the complexity index, we simplify the complexity analysis into a manageable and insightful metric. This not only aids in our investigation of how complexity affects recipe ratings but also provides valuable insights for recipe creators and users alike. The complexity index, therefore, is a crucial tool for understanding and optimizing the relationship between recipe complexity and user satisfaction.
