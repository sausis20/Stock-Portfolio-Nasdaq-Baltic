# Stock-Portfolio-Nasdaq-Baltic

## Summary

This study was focused on modeling intrinsic asset prices and modeling the return on the securities in the Nasdaq OMX Baltic Main List. I began by scraping the relevant tickers from the Nasdaq websitem, which yielded a total of 34 tickers. Next, I obtained the fundamental data for these companies, such as Cash Flow, Net Income, Long Term Investments, Cash, etc. using the yfinance API, upon which 4 tickers with no data were removed and we were left with 30 tickers.

The data obtained was then used in a number of investigations. First, I attempted to create an asset pricing model which could aid in a value investing strategy over a 6 month period by regressing the prices of the stocks on the day when the fundamental company data was published and attempting to use the residuals as margins of safety. Then, I moved onto modeling the returns. I first created a new category to classify the securities that had positive returns (gainers) and negative returns (losers), and then trained the model to predict this category. The trained model returned an accuracy score of 83.3%, however, the results proved to be not statistically significant with R-squared of around 0.14. Once a model was trained, a simulated portfolio was constructed in a long/short equity strategy to show how the model’s predicted stock picks would have performed. The simulation showed that the model was profitable when the actual market was in a downturn, mainly because of the ability to short stocks predicted by the model to have negative returns. Lastly, I simulated a bull market to see how the model would perform in the opposite environment than it was trained on. The results showed that the model was again profitable (although not as much as during the downturn) and had a much lower volatility compared to a traditional Buy-and-hold strategy.


## Web Scraping Nasdaq OMX Baltic

[Link to the scrape file](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/ticker_scrape.ipynb)

Although there are many companies listed in Nasdaq OMX Baltic, I scraped only the securities from the Baltic Main List, as these securities have the highest Market Cap and liquidity. Here's an example of how the data looked after the scrape:
![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/tickers.png)

## Fundamental Data pull using Yahoo Finance API

[Link to the fundamental data pull](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/fundamental_data_pull.ipynb)

Next, I pulled all fundamental companies data - Company Info, Balance Sheet, Profit & Loss, Cash Flow, Dvidends, Major and Institutional Holders, as well as Sustainability and Recommendations - but companies in the Baltic market did not have any data in these categories. 

![](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/blob/main/images/pl.jpg)

## Modeling the returns
