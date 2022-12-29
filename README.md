# Stock-Portfolio-Nasdaq-Baltic

## Summary

This study was focused on modeling intrinsic asset prices and modeling the return on the securities in the Nasdaq OMX Baltic Main List. I began by scraping the relevant tickers from the Nasdaq websitem, which yielded a total of 34 tickers. Next, I obtained the fundamental data for these companies, such as Cash Flow, Net Income, Long Term Investments, Cash, etc. using the yfinance API, upon which 4 tickers with no data were removed and we were left with 30 tickers.

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
