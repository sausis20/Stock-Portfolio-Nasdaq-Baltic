# Stock-Portfolio-Nasdaq-Baltic

## Summary

This study was focused on modeling intrinsic asset prices and modeling the return on the securities in the Nasdaq OMX Baltic Main List. I began by scraping the relevant tickers from the Nasdaq website, which yielded a total of 34 tickers. Next, I obtained the fundamental data for these companies, such as Cash Flow, Net Income, Long Term Investments, Cash, etc. using the yfinance API, upon which 4 tickers with no data were removed and we were left with 30 tickers.

The data obtained was then used in a number of investigations. First, I attempted to create an asset pricing model which could aid in a value investing strategy over a 6 month period by regressing the prices of the stocks on the day when the fundamental company data was published and attempting to use the residuals as margins of safety. Then, I moved onto modeling the returns. I first created a new category to classify the securities that had positive returns (gainers) and negative returns (losers), and then trained the model to predict this category. The trained model returned an accuracy score of 83.3%, however, the results proved to be not statistically significant with R-squared of around 0.14. Once a model was trained, a simulated portfolio was constructed in a long/short equity strategy to show how the model’s predicted stock picks would have performed. The simulation showed that the model was profitable when the actual market was in a downturn, mainly because of the ability to short stocks predicted by the model to have negative returns. Lastly, I simulated a bull market to see how the model would perform in the opposite environment than it was trained on. The results showed that the model was again profitable (although not as much as during the downturn) and had a much lower volatility compared to a traditional Buy-and-hold strategy.


## Web Scraping Nasdaq OMX Baltic

[Link to the scrape notebook](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/ticker_scrape.ipynb)

Although there are many companies listed in Nasdaq OMX Baltic, I scraped only the securities from the Baltic Main List, as these securities have the highest Market Cap and liquidity. Here's an example of how the data looked after the scrape:
![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/tickers.png)

## Fundamental Data pull using Yahoo Finance API

[Link to the fundamental data pull](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/fundamental_data_pull.ipynb)

Next, I pulled all fundamental companies data - Company Info, Balance Sheet, Profit & Loss, Cash Flow, Dvidends, Major and Institutional Holders, as well as Sustainability and Recommendations - but companies in the Baltic market did not have any data in these categories. 

## Data exploration and pricing data pull

[Link to the data exploration and pricing data pull](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/data_exploration.ipynb)

I first visualized some features to get a better sense of the data I'm working with, for example the Market Capitalization.
![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/marketcap.png)

Also I pulled the closing stock prices from the period when the fundamental data was published (around 2022-05-21) to the time of this study (2022-11-30) - approximately 6 months worth of data. At this point I also converted the prices to log prices (for various reasons discussed in the notebook) and visualized them.

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/logreturns.png)

## Modeling current price

[Link to the notebook](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/modeling_current_price.ipynb)

Here I've modeled the securities intrinsic prices using the fundamental data found in the companies financial statements, and subsequently compared the model's residuals against the securities returns. The idea behind calculating residuals is a value investing principal of 'margin of safety', which is the difference between the current stock price in the market, vs. the price calculated subjectively by a value investor. We would expect that higher residuals (or in other words, how over or under valued a given security is compared to what the model estimates its value to be) would be correlated to the returns of the securities over the six month period since the data were scraped, as overvalued stocks would see their prices move down as the market adjusts toward their true value, and undervalued stocks would move up.

We know that markets are at least partially efficient, meaning that the current prices of assets reliably reflect their value, with some error and noise present. However, it is also reasonable to assume that it takes some time for 'price discovery' to occur, or in other words for sellers and buyers to agree on the new price after the companies fundamental data has been published, meaning there might be an edge for the investor.

First, I removed the pricing data, as leaving pricing related features and then trying to predict price would be detrimental to the model. At this point the dataset had many missing values, so I moved on to imputting it using various regressor and imputation techniques combinations. 

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/missingdata.png)

I evaluated Linear Regression, Gradient Boosting Regressor, XGBRegressor and Random Forest Regressor, which proved to be the best performing regressor when measured using R-sqaured (the reasoning behind choosing R-squared as the scoring metric is explained in the notebook). We can see that in respect to R-squared score, the Iterative Imputation method performed the best compared to other imputation techinques, so I used this method combination (Random Forest Regressor and Iterative Imputation) to input the remaing missing values in the dataset.

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/imputation.png)

For the modeling of the current price, Gradient Boosting Regressor was the best performer with an R2 score of 0.26; I continued by doing grid search (tuning the model to find the best parameters to use with in model), and calculated the residuals (actual current price - price predicted by the model). Resulting residuals appeared to be symmetrically distributed around 0, meaning that there isn't any difference between the actual stock prices in the market and the price predicted by the model. However, there are two instances of positive residuals, which indicate these stocks could be potentially overvalued.

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/residuals.png)

To confirm the results, I run a simple linear regression to see if there is any relationship between the residuals and the returns. Unfortunately, the findings did't show such a relationship. The determination coefficient (R2) was just 0.03, which did not indicate a powerful statistical relationship. At this point it was cocluded that the hypothesis that the residuals of the asset pricing model would be correlated to the returns over the six month period since the time when the fundamental data was published is not supported.

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/returnsresiduals.png)

## Modeling returns

[Link to the notebook](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/modeling_returns.ipynb)

In the second part of the analysis I modeled the returns over the six month period since the companies fundamental data was published. This was done by creating a binary classifier to predict gainers/losers (returns above or less than or equal to zero). The model was then tested on the holdout set to observe the results.

The problem could be described as follows:

null hypothesis - the securities fundamental data has no predictive information about the securities returns
alternative hypothesis - there is predictive information about securities returns in their fundamental data

First, I tackled multicollinearity among the features, removing the ones with the highest values. Although this step was not mandatory, as I was not checking feature importance, multicollinearity may have still impacted the model negatively. 

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/corr.png)

Then, I dealt with the missing values in a similar fashion as before, that is by trying different regressor and imputation techniques and choosing the combination with the best score. This time, the target variable is a category (1 - gainer, 0 - loser), so I evaluating different classification regressors - LogisticRegression, RandomForestClassifier, GradientBoostingClassifier, AdaBoostClassifier and KNeighborsClassifier. This classifier, combined with KNN imputation technique, proved to have the best performance in terms of roc_auc score of around 0.62. The reason behind choosing roc_auc score is detailed in the notebook. 

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/roc_auc.png)

For the modeling of class prediction, RandomForestClassifier provided the best performance. After doing a grid search, that is tuning the model to find the best parameters to use with it, I calculated the probabilities of being assigned to class 1 or 0 on the holdout set and fit a regression line. 

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/classprob.png)

The visualization of the probabilities of securities being assigned to gainers = 1, it could be seen that all of the securities in the test sample had negative returns, and the model correctly assigned most of them to the 0 (losers) category (we can see that from the x axis, which shows the probability of being in the 1 (gainers) category).
The regression line has the slope we expected - probability of being assigned to the gainers category (1) increases as the log returns increase, so this is encouraging. However, the R-squared for this simple linear regression of the log returns using the predicted probabilities is very low at .144. The p-value, however, is only at 0.315, or in other words, there is 31.5% chance that the test results occured under the null hypothesis i.e. by pure chance.

The conclusion of this is that the model is not effective in predicting the securities returns from the fundamental data. Although the regression line seems promising, the determination coefficient (R-squared) and p-test indicate that the model is not statistically significant and we can't reject the null hypothesis.

## Portfolio Construction

Although the model was found to not have statistical significance, I still constructed a portfolio based on its predictions, which could serve as a template if the study is performed at some point in the future with a different dataset.

One potential application of this model could be demonstrated by constructing a long/short equity portfolio using the model’s predictions of over/under performers. The benefit of the long/short equity strategy is that the investor is hedged against the market by taking an equal amount of long and short positions. This way, the portfolio is not affected by overall upward or downward movement of the market, and instead is only affected by how well the investor has predicted the relative performance of securities within it. Although the model proved to be not statistically significant - mainly because of the small sample size and relatively short time period studied - the model is still correctly identifying over and under performers with an accuracy of 83,3%. Combining this predictive power with a long/short equity strategy this can lead to a consistently profitable trading strategy by both diversifying the risk of the portfolio by taking many positions and also hedging against the market. If the market moves up, the investor will gain from their long positions while losing on their short positions, and conversely if the market moves down, they will gain on their short positions while losing on their long positions. By having predicted with better than random accuracy which securities were set to over or underperform the market, the gains from the winning side of the portfolio should average out to be bigger than the losses from the losing side, no matter which way the market moves.

We can simulate how such a portfolio would have performed using the securities of the test set by using the class predictions for each security. For securities that the model predicts we should short, I flipped the sign of their respective log returns to indicate that we will profit from the drop in these stocks' prices. Finally, I averaged all of these adjusted returns together. This would be the equivalent of the investor making equal dollar investments into each security in the test set, going short in any stock predicted by the model to underperform, and going long in any stock predicted to overperform.

We can get a better understanding of how the securities in the test compare to the overall market by visualizing their performance.

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/allmarket.png)

We can see that the securities in the test set have noticably underperformed the overal market, however the overall shape of the returns is almost exactly the same. We can also see that during the study period, the overall market was in a correction / bear market. 

To construct the Long/Short portfolio, I first split the securities in the test set into long and short positions. The portfolio goes long in all companies predicted to outperform, and short in the companies that are predicted to underperform. 

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/buyholdvsportfolio.png)

We can see that the Long/Short portfolio significantly over performed the test set portfolio. This can be explained by the fact that because the overal market was declining over the study period, a strategy that is able to short stocks will perform significantly better than simple Buy-and-hold strategy. Next, I looked at the returns of the long side of the portfolio and the short side of the portfolio separately.

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/longshort.png)

Again, we see that the Short side was performing significantly better than the Long side. 

Notice how both lines start at 0, but the red line finishes at around 0.18, while the green line finishes at around -0.06. The fact that the red line makes more profit than the green line loses is exactly the goal of the strategy. By going long on the securities that the model predicted will over perform, and going short on the securities that the model predicted will under perform, the aggregate result of the portfolio would be positive.

At this point a question comes to mind - why not just take the short positions, and skip the long positions? The answer to this question requires knowing the general direction of the market. Since during the study period the market was in a downturn, the short side was performing better. However, if the market were to enter a bull market, we would expect the long side to become more profitable. Let's simulate how the portfolio would have behaved in the opposite (bull) market scenario.

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/bullmarket.png)

After simulating the bull market (I did this by multiplying the actual returns by -2 and adding that to the actual returns), we can clearly see the advantages of Long/Short portfolio strategy - during the favorable market conditions the portfolio is still overperforming the Buy-and-hold portfolio. On top of that, the volatility of the portfolio is much lower, compared to the big swings in the traditional portfolio.

## Final Thoughts

The study has shown some really interesting results and gave me a chance to investigate an interesting dataset, but an additional study on this subject would be appropriate. Here are a few things which could be improved in future work: 
-   The companies in the Nasdaq Baltic Main List has data for just 30 securities, however the total number of features were 105 (if we exclude features related to pricing information) and 120 features (if we include them). Generally, we would want to have more observations than features, so the model could be improved by making it more parsimonious (checking and removing features that add little or no explanatory power to the model).
-   Additionally, it is relatively difficult to get a working model with such a few observations, therefore it could be worthwile to repurpose this study to include more observations, for example by analysing the whole Central-Eastern European stock market or including other markets that have comparable charachteristics as the Baltic stock market.


## References

This work was inspired and heavily relies on work from Nate Cibik's post on medium, which can be found [here: ](https://medium.com/analytics-vidhya/predicting-returns-with-fundamental-data-and-machine-learning-in-python-a0e5757206e8)
