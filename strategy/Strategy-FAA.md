---
layout: page
title: Flexible Asset Allocation (FAA)
---


To install [Systematic Investor Toolbox (SIT)](https://github.com/systematicinvestor/SIT) please visit [About](/about) page.





The [Generalized Momentum and Flexible Asset Allocation (FAA): An Heuristic Approach by Wouter J. Keller and Hugo S.van Putten (2012)](http://papers.ssrn.com/sol3/papers.cfm?abstract_id=2193735)
backtest and live signal. For more details please see:
 
* [Generalized Momentum and Flexible Asset Allocation (FAA): An Heuristic Approach by Wouter J. Keller and Hugo S.van Putten (2012)](http://papers.ssrn.com/sol3/papers.cfm?abstract_id=2193735)
* [Flexible Asset Allocation](http://turnkeyanalyst.com/2013/01/flexible-asset-allocation/)
* [Asset Allocation Combining Momentum, Volatility, Correlation and Crash Protection](http://www.cxoadvisory.com/subscription-options/?wlfrom=%2F22480%2Fvolatility-effects%2Fasset-allocation-combining-momentum-volatility-correlation-and-crash-protectio%2F)

The [FAA Strategy](http://papers.ssrn.com/sol3/papers.cfm?abstract_id=2193735)
allocates across Price, Volatility, and Correlation Momentum.





Load historical data from Yahoo Finance:



{% highlight r %}
#*****************************************************************
# Load historical data
#*****************************************************************
library(SIT)
load.packages('quantmod')

tickers = '
US.STOCKS = VTI + VTSMX
FOREIGN.STOCKS = VEU + FDIVX
EMERGING>MARKETS=EEM + VEIEX
US.10YR.GOV.BOND = IEF + VFITX
REAL.ESTATE = VNQ + VGSIX
COMMODITIES = DBC + CRB
CASH = BND + VBMFX
'

# load saved Proxies Raw Data, data.proxy.raw
load('data.proxy.raw.Rdata')

data <- new.env()

getSymbols.extra(tickers, src = 'yahoo', from = '1970-01-01', env = data, raw.data = data.proxy.raw, auto.assign = T, set.symbolnames = T, getSymbols.fn = getSymbols.fn, calendar=calendar)
  for(i in data$symbolnames) data[[i]] = adjustOHLC(data[[i]], use.Adjusted=T)
bt.prep(data, align='remove.na', dates='::')

print(last(data$prices))
{% endhighlight %}



|           | US.STOCKS| FOREIGN.STOCKS| EMERGING>MARKETS| US.10YR.GOV.BOND| REAL.ESTATE| COMMODITIES|  CASH|
|:----------|---------:|--------------:|----------------:|----------------:|-----------:|-----------:|-----:|
|2015-03-06 |    107.46|          48.47|            39.26|           105.49|       80.37|       17.57| 82.12|
    




{% highlight r %}
#*****************************************************************
# Setup
#*****************************************************************
data$universe = data$prices > 0
	# do not allocate to CASH
	data$universe$CASH = NA 

prices = data$prices * data$universe
	n = ncol(prices)
{% endhighlight %}





Code Strategy Rules:



{% highlight r %}
#*****************************************************************
# Code Strategy
#******************************************************************
ret = diff(log(prices))

n.top = 3

mom.lookback = 80
vol.lookback = 80
cor.lookback = 80

weight=c(1, 0.5, 0.5)

hist.vol = sqrt(252) * bt.apply.matrix(ret, runSD, n = vol.lookback)

mom = (prices / mlag(prices, mom.lookback) - 1)[period.ends,]

#*****************************************************************
# Compute Average Correlation
#******************************************************************
avg.cor = data$weight * NA
for(i in period.ends[period.ends > cor.lookback]){
  hist = ret[(i - cor.lookback):i,]
    include.index = !is.na(colSums(hist))
  correlation = cor(hist[,include.index], use='complete.obs',method='pearson')
  avg.correlation = rowSums(correlation, na.rm=T)

  avg.cor[i,include.index] = avg.correlation
}


mom.rank = br.rank(mom)
cor.rank = br.rank(-avg.cor[period.ends,])
vol.rank = br.rank(-hist.vol[period.ends,])

avg.rank = weight[1]*mom.rank + weight[2]*vol.rank + weight[3]*cor.rank
meta.rank = br.rank(-avg.rank)

#absolute momentum filter 
weight = (meta.rank <= n.top)/rowSums(meta.rank <= n.top, na.rm=T) * (mom > 0)

# cash logic
weight$CASH = 1 - rowSums(weight,na.rm=T)

obj$weights$strategy = weight
{% endhighlight %}


![plot of chunk plot-6](/public/images/Strategy-FAA/plot-6-1.png) 

#Strategy Performance:
    




|              |strategy          |
|:-------------|:-----------------|
|Period        |Jun1996 - Mar2015 |
|Cagr          |12.2              |
|Sharpe        |1.22              |
|DVR           |1.13              |
|R2            |0.93              |
|Volatility    |9.91              |
|MaxDD         |-16.17            |
|Exposure      |99.51             |
|Win.Percent   |63.28             |
|Avg.Trade     |0.39              |
|Profit.Factor |2.07              |
|Num.Trades    |610               |
    


![plot of chunk plot-6](/public/images/Strategy-FAA/plot-6-2.png) 

#Monthly Results for strategy :
    




|     |Jan   |Feb   |Mar   |Apr   |May   |Jun   |Jul   |Aug   |Sep   |Oct   |Nov   |Dec   |Year  |MaxDD |
|:----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|
|1996 |      |      |      |      |      |      |  0.0 | -0.3 |  1.7 |  2.0 |  4.0 |  4.0 | 11.8 | -1.9 |
|1997 |  0.2 | -0.7 | -2.5 | -1.3 |  5.9 |  1.4 |  2.3 | -7.4 |  5.2 |  0.0 | -1.0 |  1.5 |  2.9 | -7.4 |
|1998 |  0.2 |  2.2 |  3.2 |  1.5 | -0.7 |  0.1 | -0.7 | -3.2 |  2.5 | -0.7 |  0.2 |  3.3 |  8.1 | -5.9 |
|1999 | -1.7 | -2.9 |  2.2 |  2.6 | -3.2 |  4.3 | -1.0 |  1.1 |  2.4 |  0.2 |  4.2 |  9.3 | 18.2 | -6.8 |
|2000 | -3.6 |  4.8 |  0.6 | -2.6 |  3.3 |  2.8 |  1.7 |  2.4 |  1.0 | -1.5 |  3.0 |  0.8 | 13.0 | -5.7 |
|2001 |  0.6 | -0.4 | -1.3 |  0.3 |  1.0 |  2.3 |  0.9 | -0.5 | -0.2 |  0.2 | -1.7 |  0.1 |  1.2 | -4.9 |
|2002 |  1.3 |  1.9 |  2.9 |  0.2 |  0.4 |  2.5 | -0.9 |  2.9 |  2.8 | -0.9 | -0.5 |  1.6 | 15.1 | -4.6 |
|2003 |  2.6 |  0.9 | -2.1 |  0.6 |  6.1 |  1.9 |  0.2 |  4.1 |  0.2 |  3.5 |  2.9 |  5.3 | 29.2 | -4.4 |
|2004 |  2.7 |  2.2 |  2.5 | -6.6 |  1.7 | -1.3 |  2.1 |  1.1 |  0.1 |  2.5 |  3.9 |  3.0 | 14.2 | -7.5 |
|2005 | -3.5 |  4.2 | -4.1 | -2.2 |  0.5 |  2.6 |  1.2 |  0.3 |  1.3 | -4.0 |  2.9 |  4.1 |  2.8 | -8.4 |
|2006 |  7.0 | -2.7 |  3.6 |  2.5 | -6.3 | -0.9 |  2.2 |  0.6 |  1.3 |  3.3 |  3.0 | -1.6 | 11.9 |-16.2 |
|2007 |  3.3 | -3.0 |  0.3 |  2.8 |  1.8 |  1.2 | -0.2 |  0.0 |  7.0 |  7.1 | -1.4 |  1.8 | 22.2 | -9.0 |
|2008 | -0.9 |  6.1 |  0.2 |  1.0 |  1.6 | -0.1 | -3.3 | -1.7 | -0.4 | -2.3 |  5.2 |  5.1 | 10.7 |-11.9 |
|2009 | -2.7 | -0.6 |  1.8 |  6.4 |  5.7 | -2.4 |  6.1 |  0.8 |  4.4 | -2.7 |  5.3 |  0.5 | 24.1 | -9.3 |
|2010 | -6.7 |  4.3 |  5.1 |  3.4 | -3.5 | -2.7 |  3.7 |  2.1 |  3.8 |  3.7 | -1.8 |  4.4 | 16.2 |-10.1 |
|2011 |  1.8 |  4.2 |  0.5 |  4.4 | -1.7 | -2.7 |  2.3 |  0.2 | -6.2 | -0.4 |  0.1 |  2.6 |  4.6 | -9.5 |
|2012 |  4.1 |  1.5 | -0.2 |  1.5 | -2.7 | -0.1 |  1.5 | -0.1 |  0.5 | -1.2 |  1.2 |  1.7 |  7.8 | -4.6 |
|2013 |  1.4 |  0.1 |  2.3 |  3.1 | -2.3 | -1.7 |  2.1 | -2.0 |  0.5 |  2.8 |  0.9 |  0.7 |  7.8 | -8.5 |
|2014 | -2.0 |  3.4 |  0.5 |  1.4 |  0.9 |  1.7 |  0.7 |  2.5 | -3.7 |  2.1 |  1.9 |  0.6 | 10.2 | -5.3 |
|2015 |  2.8 | -0.2 | -2.3 |      |      |      |      |      |      |      |      |      |  0.3 | -3.7 |
|Avg  |  0.4 |  1.3 |  0.7 |  1.1 |  0.5 |  0.5 |  1.1 |  0.2 |  1.3 |  0.7 |  1.7 |  2.6 | 11.6 | -7.3 |
    


![plot of chunk plot-6](/public/images/Strategy-FAA/plot-6-3.png) ![plot of chunk plot-6](/public/images/Strategy-FAA/plot-6-4.png) 

#Trades for strategy :
    




|strategy         |weight |entry.date |exit.date  |nhold |entry.price |exit.price |return |
|:----------------|:------|:----------|:----------|:-----|:-----------|:----------|:------|
|US.STOCKS        | 33.3  |2014-08-29 |2014-09-30 |32    |102.87      |100.71     |-0.70  |
|EMERGING>MARKETS | 33.3  |2014-08-29 |2014-09-30 |32    | 44.42      | 40.97     |-2.59  |
|US.10YR.GOV.BOND | 33.3  |2014-08-29 |2014-09-30 |32    |103.84      |102.75     |-0.35  |
|US.STOCKS        | 50.0  |2014-09-30 |2014-10-31 |31    |100.71      |103.47     | 1.37  |
|US.10YR.GOV.BOND | 50.0  |2014-09-30 |2014-10-31 |31    |102.75      |104.32     | 0.77  |
|US.STOCKS        | 33.3  |2014-10-31 |2014-11-28 |28    |103.47      |106.04     | 0.83  |
|US.10YR.GOV.BOND | 33.3  |2014-10-31 |2014-11-28 |28    |104.32      |105.67     | 0.43  |
|REAL.ESTATE      | 33.3  |2014-10-31 |2014-11-28 |28    | 77.93      | 79.49     | 0.67  |
|US.STOCKS        | 33.3  |2014-11-28 |2014-12-31 |33    |106.04      |106.00     |-0.01  |
|US.10YR.GOV.BOND | 33.3  |2014-11-28 |2014-12-31 |33    |105.67      |105.65     |-0.01  |
|REAL.ESTATE      | 33.3  |2014-11-28 |2014-12-31 |33    | 79.49      | 81.00     | 0.63  |
|US.STOCKS        | 33.3  |2014-12-31 |2015-01-30 |30    |106.00      |103.10     |-0.91  |
|US.10YR.GOV.BOND | 33.3  |2014-12-31 |2015-01-30 |30    |105.65      |110.20     | 1.43  |
|REAL.ESTATE      | 33.3  |2014-12-31 |2015-01-30 |30    | 81.00      | 86.55     | 2.28  |
|US.STOCKS        | 33.3  |2015-01-30 |2015-02-27 |28    |103.10      |109.02     | 1.91  |
|US.10YR.GOV.BOND | 33.3  |2015-01-30 |2015-02-27 |28    |110.20      |107.47     |-0.82  |
|REAL.ESTATE      | 33.3  |2015-01-30 |2015-02-27 |28    | 86.55      | 83.37     |-1.22  |
|US.STOCKS        | 33.3  |2015-02-27 |2015-03-06 | 7    |109.02      |107.46     |-0.48  |
|US.10YR.GOV.BOND | 33.3  |2015-02-27 |2015-03-06 | 7    |107.47      |105.49     |-0.61  |
|REAL.ESTATE      | 33.3  |2015-02-27 |2015-03-06 | 7    | 83.37      | 80.37     |-1.20  |
    




#Signals for strategy :
    




|           | US.STOCKS| FOREIGN.STOCKS| EMERGING>MARKETS| US.10YR.GOV.BOND| REAL.ESTATE| COMMODITIES| CASH|
|:----------|---------:|--------------:|----------------:|----------------:|-----------:|-----------:|----:|
|2013-07-30 |        50|              0|                0|                0|           0|           0|   50|
|2013-08-29 |        33|              0|                0|                0|           0|          33|   33|
|2013-09-27 |        33|             33|                0|                0|           0|           0|   33|
|2013-10-30 |        50|              0|                0|               50|           0|           0|    0|
|2013-11-27 |        33|             33|                0|               33|           0|           0|    0|
|2013-12-30 |        33|             33|                0|               33|           0|           0|    0|
|2014-01-30 |        33|              0|                0|               33|          33|           0|    0|
|2014-02-27 |        50|              0|                0|                0|          50|           0|    0|
|2014-03-28 |        33|              0|                0|                0|          33|          33|    0|
|2014-04-29 |         0|              0|                0|               33|          33|          33|    0|
|2014-05-29 |         0|              0|               50|                0|          50|           0|    0|
|2014-06-27 |         0|              0|               50|                0|          50|           0|    0|
|2014-07-30 |         0|              0|               33|               33|          33|           0|    0|
|2014-08-28 |        33|              0|               33|               33|           0|           0|    0|
|2014-09-29 |        50|              0|                0|               50|           0|           0|    0|
|2014-10-30 |        33|              0|                0|               33|          33|           0|    0|
|2014-11-26 |        33|              0|                0|               33|          33|           0|    0|
|2014-12-30 |        33|              0|                0|               33|          33|           0|    0|
|2015-01-29 |        33|              0|                0|               33|          33|           0|    0|
|2015-02-26 |        33|              0|                0|               33|          33|           0|    0|
    







For your convenience, the 
[Strategy-FAA](/public/images/Strategy-FAA/Strategy-FAA.pdf)
report can also be downloaded and viewed the pdf format.





















*(this report was produced on: 2015-03-08)*
