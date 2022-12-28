# Stock-Portfolio-Nasdaq-Baltic

## Summary

This study was focused on modeling intrinsic asset prices and modeling the return on the securities in the Nasdaq OMX Baltic Main List. We began by scraping the relevant tickers from the Nasdaq websitem, which yielded a total of 34 tickers. Next, we obtained the fundamental data for these companies, such as Cash Flow, Net Income, Long Term Investments, Cash, etc. using the yfinance API, upon which 4 tickers with no data were removed and we were left with 30 tickers.

The data obtained was then used in a number of investigations. First, we attempted to create an asset pricing model which could aid in a value investing strategy over a 6 month period by regressing the prices of the stocks on the day when the fundamental company data was published and attempting to use the residuals as margins of safety. Then, we moved onto modeling the returns. We first created a new category to classify the securities that had positive returns (gainers) and negative returns (losers), and then trained the model to predict this category. The trained model returned an accuracy score of 83.3%, however, the results proved to be not statistically significant with R-squared of around 0.14. Once a model was trained, a simulated portfolio was constructed in a long/short equity strategy to show how the model’s predicted stock picks would have performed. The simulation showed that the model was profitable when the actual market was in a downturn, mainly because of the ability to short stocks predicted by the model to have negative returns. Lastly, I simulated a bull market to see how the model would perform in the opposite environment than it was trained on. The results showed that the model was again profitable (although not as much as during the downturn) and had a much lower volatility compared to a traditional Buy-and-hold strategy.


## Web Scraping Nasdaq Baltic

![Example of securities in the Baltic Main List](https://github.com/sausis20/Stock-Portfolio-Nasdaq-Baltic/tree/main/images/tickers.jpg?raw=true)
