# FRED-final-project
# Predicting U.S. Inflation Expectations with Financial Market Indicators
## 1. Introduction
This project studies daily macro-financial data from FRED to predict U.S. inflation expectations, measured by the 10-year breakeven inflation rate (T10YIE). This variable reflects the market’s expected average inflation rate over the next ten years, so it provides a useful forward-looking measure of long-term inflation expectations.

We chose this topic because inflation expectations are closely connected to real economic conditions and financial markets. Recent international news made this question especially relevant. Escalating tensions in the Middle East and concerns over disruptions around the Strait of Hormuz have pushed crude oil prices higher, with Reuters reporting Brent crude above $107 per barrel and WTI above $101 amid renewed supply concerns. This also connects to everyday experiences, for example, near the end of the semester,  we found the fuel-related charges of flight tickets became more noticeable. 

Therefore, we wanted to examine whether market indicators, including oil prices, gas prices, Treasury yields, real yields, exchange rates, and volatility measures, can help predict future movements in inflation expectations.

We approached this question in three ways. First, we used regression models to predict the exact 5-business-day future change in the 10-year breakeven inflation rate. Second, we used classification models to predict whether the 10-year breakeven inflation rate would increase over the next five business days. Third, we used time-series forecasting models to predict the future level of T10YIE itself.

Overall, the results show that short-term changes in inflation expectations are difficult to predict. In the regression task, the best KNN regressor achieved an RMSE of about 0.0849, only slightly better than the baseline model’s RMSE of about 0.0850. The classification models performed somewhat better: logistic regression improved accuracy from the baseline of 51.3% to about 54.2%, and lowering the classification threshold to 40% increased recall to about 71.4%, making the model more useful for detecting possible inflation-risk periods.

The strongest results came from the time-series forecasting task. AutoREG with 60 lags performed best, with a MAPE of about 0.128, clearly outperforming the Naive Last model’s MAPE of about 0.283. This suggests that the level of T10YIE is persistent and easier to forecast than its short-term changes. 

The main conclusion is that inflation expectation levels contain meaningful time-series structure, but short-term movements are noisy and difficult to predict with daily market indicators.
## 2. Data Description
The data used in this project comes from the Federal Reserve Economic Data database, commonly known as FRED. FRED provides daily macro-financial time series, including inflation expectations, Treasury yields, TIPS real yields, commodity prices, volatility measures, and exchange rates.

The raw dataset covers the period from January 2000 to December 2024. After cleaning and removing early rows with missing values, the final usable dataset covers the period from January 2, 2003, to December 31, 2024. 

The raw dataset initially contained 6,522 business-day observations and 17 variables. After aligning the data to business-day frequency, forward-filling small holiday gaps, and dropping rows that still had missing values, the cleaned dataset contained 5,739 observations and 17 original variables.

The main target variable is t10yie, the 10-year breakeven inflation rate. We selected this variable because it directly measures market expectations of long-term inflation. Unlike realized inflation, which is backward-looking, breakeven inflation is forward-looking and reflects how investors price future inflation risk.

The explanatory variables were selected based on financial and economic reasoning. We included inflation-expectation variables, such as T10YIE, T5YIE, and T5YIFR, to capture both medium-term and long-term market inflation expectations. We also included Treasury yields and TIPS real yields because breakeven inflation is closely connected to the relationship between nominal interest rates and real interest rates. Yield curve variables were added to capture changes in growth expectations, monetary policy expectations, and recession risk. Commodity variables, including WTI oil, Brent oil, and natural gas, were included because energy prices are directly related to transportation costs, production costs, and inflation pressure. We also used the VIX index to measure market uncertainty and risk sentiment, since risk-off periods can affect both growth expectations and inflation expectations. Finally, exchange-rate variables were included because dollar movements can influence import prices, commodity prices, and global inflation pressure. 

After feature engineering, the final supervised-learning dataset contained 5,714 observations and 28 columns, including 26 predictors and 2 target variables. 

## 3. Models and Methods
This project uses three major modeling approaches: regression, classification, and time-series forecasting. The regression task predicts the exact size of future changes in inflation expectations. The classification task predicts the direction of future changes. The time-series forecasting task predicts the future level of the 10-year breakeven inflation rate itself.

For the supervised learning models, we used a time-based train-test split instead of a random split. Observations before 2020 were used as the training set, while observations from 2020 onward were used as the testing set. This setup better reflects a real forecasting situation because the model is trained on earlier data and evaluated on later data.

### Regression Models
For the regression task, we used today’s financial market indicators to predict the exact 5-business-day-ahead change in T10YIE. We first built a baseline model that predicts the average future change from the training period. This gives us a simple benchmark for comparison.

We then estimated a simple linear regression model using the most direct recent T10YIE movement as the predictor. After that, we estimated a multiple linear regression model using the full set of financial indicators, including inflation-expectation variables, Treasury yields, real yields, commodity variables, exchange-rate variables, and volatility measures.

Because the predictors have different units and scales, we also used a pipeline version of linear regression with standardized predictors. In addition, we compared different feature groups, such as inflation-expectation variables only, rates and real yields, commodities, and risk and foreign-exchange variables. This helped us examine which type of information was most useful for prediction, and check if the full model is overfitting.

Finally, we used KNN regression. KNN predicts future T10YIE changes by looking for historical days with similar market conditions. Since KNN depends on distance, standardized predictors were used. We also used grid search to select the best number of neighbors.

### Classification Models
For the classification task, we predicted whether T10YIE would increase over the next five business days. This changes the problem from predicting the exact size of the movement to predicting its direction.

We began with a baseline classifier that always predicts the most common class in the training data. We then used logistic regression as the main classification model. Since the predictors have different units, we standardized them before fitting the model.

The default logistic regression model uses a 50% probability threshold to classify an observation as an inflation-increase case. Because our project is also interested in inflation-risk monitoring, we manually lowered this threshold to 40% to make the model more sensitive to potential upward movements in T10YIE. We also used a validation set to select the threshold that maximized accuracy.

Finally, we tested KNN classification as a comparison. KNN classification predicts whether T10YIE will increase by comparing current market conditions with similar historical observations. This gives us a non-linear, similarity-based alternative to logistic regression.

### Time-Series Feature Extension
After the basic regression and classification models, we extended the feature set using lagged variables, rolling averages, rolling volatility measures, and momentum variables. The purpose was to test whether recent market trends and short-term historical patterns could improve prediction.

We tested the expanded feature set using Ridge regression and logistic regression. Ridge regression was used because the expanded dataset contains many predictors, and regularization can help reduce overfitting.

We also experimented with tsfresh as an additional feature-engineering extension. Tsfresh automatically extracts statistical features from rolling time windows, allowing us to test whether more complex time-series patterns could provide useful predictive information beyond our manually created features.

### Time-Series Forecasting Models
For the time-series forecasting task, we predicted the future level of T10YIE itself. This task is different from the supervised regression and classification tasks because it relies mainly on the historical behavior of T10YIE rather than predicting its 5-day change.

We compared several forecasting models. The Naive Forecasting model uses the most recent observed value as the forecast. The Naive Drift model extends the historical trend. Exponential Smoothing gives more weight to recent observations. Holt-Winters adds trend and possible weekly business-day seasonality. AutoREG uses past values of T10YIE to predict future values. Finally, ARIMA models were tested because they are commonly used for time-series forecasting and can model persistence and short-term error patterns.

## 4. Results and Interpretation
The modeling results show a clear pattern: predicting the exact short-term movement of inflation expectations is difficult, while predicting the general level of inflation expectations is more successful. This difference is important because it suggests that the predictability of T10YIE depends on the type of target we choose. The level of T10YIE is relatively persistent over time, but its short-term changes are much noisier and more sensitive to new market information.

### Regression Results 

In the regression task, the models performed weakly overall. The baseline regression model, which simply predicts the average future change from the training period, produced an RMSE of approximately 0.0850. The best regression model was the optimized KNN regressor, which produced an RMSE of approximately 0.0849. Although this was technically the lowest RMSE among the regression models, the improvement over the baseline was extremely small. From a practical perspective, this difference is not large enough to suggest strong predictive power.

One important limitation is that the best KNN model selected the upper limit of neighbors allowed in our grid search. This means the model performed best when it averaged across many historical observations rather than relying on a small group of highly similar market conditions. In other words, the result does not show a strong local pattern in the data. It may also mean that a larger search range for the number of neighbors could be tested in future work.

The weaker performance of the full linear models is also informative. Multiple linear regression and pipeline linear regression produced much higher test errors, with an RMSE of about 0.1188 and a strongly negative test R-squared. This suggests that adding many macro-financial predictors increased noise and overfitting rather than improving prediction. The feature-group comparison supports this interpretation: inflation-expectation variables alone performed better than the full model. This means that, although oil prices, gas prices, Treasury yields, real yields, exchange rates, and volatility are economically related to inflation, their daily relationships with short-term T10YIE changes are not stable enough to support a precise 5-day prediction.

From a real-world perspective, the regression models should not be used as a precise forecasting tool for the exact size of inflation-expectation changes. A potential user, such as a financial analyst, should treat these regression results as evidence that short-term inflation-expectation movements are highly noisy and difficult to forecast using daily market indicators alone.

### Classification Results 
The classification task performed slightly better than the regression task. The baseline classification model achieved an accuracy of about 51.3%, mainly because one class was slightly more common than the other. Logistic regression improved accuracy to about 54.2%, and the accuracy-optimized threshold improved it further to about 54.8%. These results are modest, but they show that predicting the direction of inflation expectations is somewhat more feasible than predicting the exact numerical change.

However, accuracy alone does not fully explain the classification results. The default logistic regression model had relatively low recall, meaning it missed many actual cases where T10YIE increased. When we lowered the classification threshold, recall increased substantially to about 71.4%, and the F1 score also improved. This is one of the most meaningful results in the classification section. It shows that the model can be adjusted depending on the purpose of the analysis. If the goal is to maximize overall accuracy, the higher threshold is better. If the goal is to monitor inflation risk and avoid missing potential increases in inflation expectations, the lower threshold is more useful.

The KNN classifier did not achieve the highest accuracy, but it produced a more balanced result in terms of recall and F1 score. This suggests that KNN was more willing to identify potential inflation-risk periods than the default logistic regression model. However, its overall accuracy remained below that of logistic regression. Therefore, logistic regression is still the stronger main classification model, while KNN provides a useful comparison because it captures the tradeoff between accuracy and risk detection.
### Time-Series Feature and tsfresh Results 

The expanded time-series feature models did not improve the supervised learning results. After adding lagged variables, rolling averages, rolling volatility measures, and momentum variables, the Ridge regression model performed much worse than the simpler regression models. The logistic regression model with expanded time-series features also had weak recall. This suggests that simply adding more historical features does not solve the prediction problem. The additional variables increased model complexity, but they did not create a stable predictive relationship with the future 5-day change in T10YIE.

The tsfresh extension produced mixed results. It did not improve regression performance, which reinforces the conclusion that predicting the exact short-term change in inflation expectations is very difficult. However, tsfresh produced a more useful classification result, especially in recall and F1 score. This suggests that rolling-window statistical patterns may contain some directional information. Still, the improvement was limited, so tsfresh should be interpreted as a creative feature-engineering extension rather than the main source of predictive success.
Time-Series Forecasting Results 

The time-series forecasting section produced the strongest results. Unlike the regression and classification tasks, this part predicted the future level of T10YIE rather than its short-term 5-day change. The best model was AutoREG with 60 lags, with an MAPE of about 0.128, an RMSE of about 0.354, and an MAE of about 0.307. It clearly outperformed the Naive Last model, Exponential Smoothing, Holt-Winters, and ARIMA models. This suggests that T10YIE is highly persistent: recent values of the 10-year breakeven inflation rate are informative about its future level. 
The forecasting graphs also support this result. Most simpler models produced relatively flat or slowly changing predicted curves, which means they mainly assumed that future inflation expectations would stay close to recent values. In contrast, the AutoREG forecast captured a more upward-moving pattern, making it better aligned with the direction of the test-period series. This suggests that using a longer history of past T10YIE values helped the model capture persistence and trend more effectively.

However, the AutoREG result also has a limitation. The best model used 60 lags, which was the largest lag value tested. This means the optimal lag length may be larger than 60, or somewhere between the tested values. Therefore, while AutoREG is the best forecasting model in our current project, future work should test a wider and more detailed range of lag values before treating 60 lags as the true optimal choice.

From a practical perspective, the time-series forecasting model is the most useful part of the project for building a baseline inflation-expectation dashboard. It may not predict sudden news shocks, but it can provide a reasonable estimate of where long-term inflation expectations are likely to move based on their own historical behavior.

Overall, the results show that exact short-term prediction is unreliable, directional risk monitoring is moderately useful, and level forecasting is the most stable application. This distinction is important because different users may care about different questions: whether inflation expectations will move, whether they will rise, or where the overall level is heading. 
## 5. Limitations and Recommendations
Although this project provides useful evidence on the predictability of U.S. inflation expectations, it also has several limitations. 
The main limitation of the project is that our dataset only includes quantitative market indicators. It does not directly include qualitative news, geopolitical events, Federal Reserve speeches, CPI announcement surprises, or oil supply shock indicators. These omitted factors may be especially important for short-term changes in inflation expectations. 

Another limitation is the test period. Since the test set begins in 2020, it includes a highly unusual macroeconomic environment, including the pandemic shock, inflation surge, supply-chain disruptions, and rapid monetary tightening. This makes the evaluation realistic but also more challenging because the relationship between predictors and inflation expectations may have changed after the pandemic. 

A further limitation is the short prediction horizon. A 5-business-day horizon may be too short for some macro-financial variables to show their effects. Oil prices, interest rates, and exchange rates may influence inflation expectations over longer horizons, such as several weeks or months, rather than within one week. Therefore, future analysis could test 10-day, 20-day, or monthly prediction horizons.

Based on these limitations, we think future analysis could improve the project in several ways:

First, we could test longer prediction horizons, such as 10-day, 20-day, or monthly changes, because macro-financial variables may affect inflation expectations over a longer period than one business week. 

Second, we could add event-based variables, such as CPI release dates, FOMC meeting dates, Federal Reserve speeches, or oil supply shock indicators. 

Third, we could compare pre-2020 and post-2020 periods separately, since inflation expectations may behave differently during stable periods and crisis periods. These extensions would make the analysis more realistic and could help explain when financial indicators are most useful for predicting inflation expectations.

From a practical perspective, this model framework is more useful as an inflation-risk monitoring tool than as a precise short-term forecasting system. For users such as financial analysts or macro research teams, the model can help organize market signals and highlight periods when inflation expectations may deserve closer attention. However, it should be combined with real-time news, central bank communication, oil-market developments, and CPI releases rather than used alone. 
## 6. Conclusions
This project used FRED macro-financial data to examine whether financial market indicators can predict U.S. inflation expectations. We tested the question through regression, classification, and time-series forecasting, and the results show that the answer depends strongly on the prediction target.

The main takeaway is that short-term changes in inflation expectations are difficult to predict, while the level of inflation expectations is more predictable because it is persistent over time.  Regression models did not meaningfully outperform the baseline, suggesting that the exact 5-business-day change in T10YIE is highly noisy. Classification models provided modest directional signals, especially after lowering the threshold to make the model more sensitive to inflation-risk periods. Time-series forecasting produced the strongest results because it used the historical behavior of T10YIE itself.

Overall, the project shows that financial market indicators are more useful for monitoring inflation-risk direction and forecasting the general level of inflation expectations than for predicting exact short-term changes. With additional event-based variables, longer prediction horizons, and separate analysis of stable and crisis periods, this framework could become a more practical decision-support tool for applied macro-financial analysis.

