# How to Predict Recipe Ratings Based on Recipe Complexity
**Authors:** Sophia Papadopoulos and Leo Udell <br>
**Emails:** spapadop@umich.edu & leoudell@umich.edu

## Introduction 
Picture this. You come home from a long day at work at the world's best data science company. All you want is a quick bite, but what are the odds this meal will be 5 stars? What is the relation between complexity and star ratings? This analysis explores a dataset of over 730,000 reviews on food.com to determine the relationship between highly rated recipes and recipe complexity. We have defined how complex a recipe can be based on how long it takes to prep and cook, the number of different ingredients, and the number of steps. By analyzing this complexity, we can help chefs optimize the complexity of their recipes for better ratings and different audiences.

**Total Rows in Dataset:** 

| Column Recipe in Dataset   | Description |
|-------------|-------|
| 'name'   | Recipe name|
| 'minutes' | Minutes to prepare recipe |
| 'n_steps' | Number of steps in recipe    |

| Column Rating in Dataset    | Description |
|-------------|-------|
| 'user_id'   | User ID |
| 'rating' | Star rating given by user |

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
1. The Food.com data came in two separate CSV files, one for the recipes and one for the ratings. We performed an inner join of the recipes and ratings CSVs on the recipe_id field to ensure every rating in our merged set had a corresponding recipe record.
2. We removed columns that were not needed for modeling, like rating text and written steps. After column drops, we checked for NaNs across all remaining fields and found none. However, some ratings had 0 as a rating value; we dropped those rows before training.
3. Our dataframe started around 731,000 rows with 17 columns, and we ended with 220,000 rows with 5 columns after cleaning.

### Univariate Analysis
This plot shows the distribution of ratings between recipes. As shown by the plot, most of the recipes have a 5-star rating. There are also a few recipes with a 0 rating, indicating that the user did not leave a rating for their posted comment. This does not mean that the user rated the recipe poorly, but that they chose not to rate the recipe. Given our goal, keeping these 0-star ratings would give inaccurate final results, so we chose to drop these recipes from our dataset. 
 <iframe
 src="assests/uniAnalysis.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
 
### Bivariate Analysis
This plot measures the average rating, number of steps, and number of ingredients in each recipe. Typically, as the number of steps increases, so does the preparation time for the recipe. Likewise, when the number of steps increases, so does the number of ingredients. 
  <iframe
 src="assests/bivAnalysisPT2.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
This plot measures the average rating, number of steps, and minutes to prepare each recipe in the dataset. Both plots are heavily skewed to the left, with most recipes not taking very long; however, there are a few recipes that take over 200,000 minutes. 
  <iframe
 src="assests/NEWBI2.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
 
### Interesting Aggregates
When constructing the bivariate analysis of the dataset, we were curious about the recipes with the longest preparation time. We came across one recipe with a strange title. Taking over 1 million minutes, "How to Preserve a Husband" has 2 ingredients (cream and peaches) and 2 five-star reviews. 
**Top 5 Longest Recipes:** 

| name | minutes | n_steps | n_ingredients | rating|
|---|-------|-------|-------|-------|                                                         
| how to preserve a husband | 1051200 | 9 | 2 | 5 |
| homemade fruit liquers | 288000 | 12 | 3  | 4 |
| homemade vanilla | 259205  |  9 | 2 | 5 |
| peach cordial | 86415 | 7 | 6 | 5 |
| chocolate chunk vanilla cake | 72000 | 10 | 9 | 4 |

### Imputation
While cleaning the data, we found that there are no missing values in the dataset. For some recipes, there are 0 ratings for that recipe, and that would then be converted to a NAN value. Due to the model we created, the recipes with zero reviews are not useful and were removed. 
## Problem Identification
What is the average rating for a recipe based on its "complexity"? We have defined recipe "complexity" as the number of ingredients, the number of minutes it takes to complete, and the number of steps included. In order to do this, we constructed a regression model to predict the recipe's start rating of 0 - 5. 
- **Type of Problem:** Our goal was to predict a numerical value: the average recipe rating based on complexity. Since this rating is a continuous variable, we were dealing with a regression problem rather than a classification problem.
- **Evaluation Metric:** We used Mean Squared Error (MSE) as our primary evaluation metric. MSE is well-suited for regression tasks because it directly measures the squared difference between predicted ratings and actual ratings. MSE penalizes large errors and prioritizes accuracy on recipes that would otherwise be error-prone. This was particularly useful in our case due to the several extremely long recipes.
## Baseline Model
For the baseline model, we chose to use a linear regression model to predict the ratings. We used three features from the recipes data set that were mentioned in our definition of complexity. <br>
*Features*
1. `n_ingredients` (quantitative): The number of ingredients a recipe requires
2. `minutes` (quantitative): The number of minutes it takes to complete a recipe
3. `n_steps` (quantitative): The number of steps it takes to complete a recipe
Since all of the features selected here are quantitative, we did not have to perform any encoding.

**Model Performance**<br>
As stated, we evaluated the performance of our model using Mean Squared Error. The MSE of this model is approximately 0.505. In the context of the rating scale, our model has a prediction error of about `sqrt(0.505) = 0.71`. This means that if a true rating is 3.0, our model will predict that the recipe is anywhere from 2.3 to 3.7 stars. This model is not optimal due to the large variance in possible predictions. Part of this inaccuracy is due in part to the highly skewed training data. With the transformations used in the next steps, we improved the overall model prediction accuracy.

## Final Model
In the baseline model, we used the raw values of `n_ingredients`, `minutes`, and `n_steps` to predict the `rating`. This method was not robust to the large outliers that we found in the dataset. In order to account for this, we added features that standardized the data while still keeping the complexity of a recipe intact. 

**New Features**
1. **QuantitativeTransformer**: We utilized this transformer on all of the columns from the baseline model to account for the extreme outliers. This created a uniform distribution of the data, which was crucial for decreasing MSE. MSE is very susceptible to large outliers, and this transformer improved our results.
2. **PolynomialFeatures**: A recipe’s complexity, as we defined it, doesn’t affect ratings strictly linearly. Very basic recipes might appeal to cooks looking for quick meals, while highly intricate ones cater to enthusiasts willing to invest more time. By adding polynomial terms, our model can flexibly fit these subtler patterns instead of forcing a purely linear relationship.

**Modeling Algorithm & Hyperparameters**:
We decided to stick with the Linear Regression model because our newly added features account for outliers and adjusting the degree of predictions. For the hyperparameters, we tuned the PolynomialFeature degree. We utlized GridSearchCV to try all degrees between 1 and 9 inclusive. With that method, we found that the best degree was a degree 4 polynomial. 

**Performance Enhancements**:
As stated previously, our baseline model achieved an MSE of 0.505. Our new model achieved a modest improvement, reducing the MSE to 0.503. This final model adjusts for outliers, which was the main factor in decreasing the MSE, and produces predictions that are more accurate relative to the true center of the data. Moving forward, this outlier‑robust approach can be extended with additional regularization or alternative loss functions to further enhance model stability and predictive performance.

**Model Performance Plots**:
To model our model's performance, we needed to keep one of the variables constant. Pictured below are 3D plots to help visualize performance when one variable is kept at the constant median of that variable. Some interesting points provided by these plots are that when steps are held constant, the model performs poorly when minutes are low, and there are 10 ingredients. It could be true that recipes advertising short preparation times but demanding many ingredients may frustrate users, or low-minute, 10-ingredient recipes are rare. Another interesting point is that the model performs best when minutes are kept consistent and ingredients and steps are varied. This is most likely due to the large time-based outliers present in the data. Under these conditions, it performs worse when there are few steps and few ingredients, suggesting that users do not look for or make incredibly simple recipes, such as a baked potato or cereal.

   <iframe
 src="assests/finalVis.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

   <iframe
 src="assests/finMConstING.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
 
   <iframe
 src="assests/finMConstMIN.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
