# New York Taxi Analysis

The data used in this project can be found on [NYC TLC Trip Record Data](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page). For this project, we use the data on January 2019 for Yellow Taxi. 

## Problem definition 

Predict the average money spent on taxi rides for each region of New York per given day and hour. 

This problem is a supervised regression problem. Supervised because we have the actual value of the value we're trying to predict and regression because we're trying to predict a continuous variable (as opposed to categorical). 

## Data Problems I Fixed

**Too-high values for total_amount**: Some values for total_amount were too high, going as high as 600,000. As these are unlikely values for a taxi fare, I decided to come up with an upper limit. The average taxi_fare was ~$16 and there were only 1,166 data points higher than $200. Compared to the 7,667,792 data points, this is not a great loss of information. Thus, I decided to remove data points with a total_amount value higher than 200.

![Too High Values](/images/too_high_values.png)

**Negative total_amount values**: Removed them since they are likely faulty data points and there weren't many of them. 

**Zero values for total_amount**: Same with negative values, zero values are removed. 

![Negative and Zero Values (1)](/images/negative_and_zero_1.png)
![Negative and Zero Values (2)](/images/negative_and_zero_2.png)

## Original features of the model

Here is the list of features that can be used for model development that came with the original data: `['PULocationID', 'transaction_date', 'transaction_month', 'transaction_day', 'transaction_hour', 'trip_distance', 'total_amount', 'count_of_transactions']`. 

You can refer to [this document](https://www1.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf) for the meaning of each feature. 

## Feature Engineering

I've added 3 sets of new features to the model. First set is time-based feature. These include, weekend and holiday boolean.

Second set is location-based information. We have Location IDs per region but there is a higher level abstraction for regions called Boroughs. This information came from the source of the main data.

The last set is weather-related data. I've downloaded this data from this website. It's free to use for development.

Here is a list of all features used in the final model: 
- Categorical features: `['PULocationID', 'transaction_month', 'transaction_day', 'transaction_hour', transaction_week_day', 'weekend', 'is_holiday', 'Borough']`
- Numerical features: `['temperature', 'humidity', 'wind speed', 'cloud cover', 'amount of precipitation']`
- Target feature: `'total_amount'` 

## The algorithms I tried and the results

I tried Decision Trees, Random Forest and Gradient Boosting. The benchmark model is a Decision Tree. In the benchmark model, I only included the original features of the models as stated above. On the normal models, I used all original features plus the newly created ones. 

Here are the performance results before tuning: 

| Algorithm         | MAE   | RMSE  | R2 Score |
|-------------------|-------|-------|----------|
| Benchmark Model   | 9.758 | 14.70 | 0.2158   |
| Decision Tree     | 8.451 | 13.73 | 0.3154   |
| Random Forest     | 7.624 | 13.43 | 0.3453   |
| Gradient Boosting | 8.314 | 13.13 | 0.3743   |

The Random Forest model is selected to be tuned. Here are the best parameters values: 
n_estimators=200
min_samples_split=10
 min_samples_leaf=1
max_features= 'sqrt'
max_depth=200,
bootstrap=False

This values of n_estimator gives a reasonably fast performance (~100 seconds on my laptop), so I decided to stick with it. 

The performance compares to the previous models is: 
| Algorithm         | MAE   | RMSE  | R2 Score |
|-------------------|-------|-------|----------|
| Benchmark Model   | 9.758 | 14.70 | 0.2158   |
| Decision Tree     | 8.451 | 13.73 | 0.3154   |
| Random Forest     | 7.624 | 13.43 | 0.3453   |
| Gradient Boosting | 8.314 | 13.13 | 0.3743   |
| Tuned Random Forest | 7.294 | 12.62 | 0.4220 |


Below is the True vs Predicted value plot for the **benchmark model** (Decision Tree). X-axis is the true values and y-axis the predicted values. 

![Benchmark Model](/images/benchmark_model.png)

Below is the True vs Predicted value plot for the **Decision Tree Model**. X-axis is the true values and y-axis the predicted values. 

![Decision Tree Model](/images/decision_tree.png)

Below is the True vs Predicted value plot for the **Random Forest Model** (before tuning). X-axis is the true values and y-axis the predicted values. 

![Random Forest Model](/images/random_forest.png)

Below is the True vs Predicted value plot for the **Gradient Boosting Model**. X-axis is the true values and y-axis the predicted values. 

![Gradient Boosting Model](/images/gradient_boosting.png)

Below is the True vs Predicted value plot for the **tuned Random Forest Model**. X-axis is the true values and y-axis the predicted values. 

![Tuned Random Forest Model](/images/tuned_random_forest.png)

## Next Steps

As you can see from the plot above, the performance can be improved. We can do this by: 
- limiting the regions included in this analysis. Removing regions that do not normally get a lot of taxi traffic can be omitted. This might be a good action to take depending on the problem at hand (If the goal is to service all of NYC no matter what, we should keep those data points). 
- hand selecting borough that should be included in the problem. Again, this should be decided based on the problem at hand and how this model is going to be used. But if the goal is solely to increase model performance, only including boroughs with the most transactions can increase the performance since likely most mistakes come from boroughs with fewer data points. Though this assumption should be checked before taking action. 
- transforming the problem into a classification problem. If all we care about is to predict whether the earning at a location in an hour is considered "high" or "low", we can turn this into a classification problem which could be predicted using a Random Forest Classifier. 
- finding the best hyperparameters using grid search.
