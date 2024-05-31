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

**Data Included: The recipts related data from 2008 to 2018.**


