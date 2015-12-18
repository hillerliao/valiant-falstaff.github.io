---
layout: post
title: IbPy - Getting a Historical SMA
description: Oh, that Interactive Brokers had included a getHistoricalSMA() function in their API. No such luck. Fortunately, I've written a Python package that makes it easy to retrieve historical SMAs programmatically from IB, so read this before you tear your hair out trying to create a solution on your own.
sitemap:
  lastmod: 2015-11-02
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
---

_Note: This blog post builds on topics covered in previous posts: see the [Getting Started with IbPy]({% post_url 2015-8-4-IbPy-Getting-Started %}), [register() vs registerAll()]({% post_url 2015-8-5-IbPy-register-vs-registerall %}), [Anatomy of an IB Message]({% post_url 2015-8-6-IbPy-Anatomy-of-an-IB-Message %}), and [Passing Arguments to a Callback in register()/registerAll()]({% post_url 2015-8-6-IbPy-passing-args-to-callback-in-register-registerall %}) blog posts if you're brand new to IbPy._

When I first started working with the Interactive Brokers API, I assumed a task as common as getting a historical Simple Moving Average would be baked into the API - something like getHistoricalSMA(). I was wrong: getting an SMA is actually quite an involved process. That's why I've created a package set for IbPy users that eliminates the pain of getting a historical SMA. 

First, download the [package set from Github](https://github.com/valiant-falstaff/IbPy-Get-Historical-SMA), then make sure that the three 3rd-party modules this code uses - [dateutil](https://labix.org/python-dateutil), [pytz](https://pypi.python.org/pypi/pytz/), and [tzlocal](https://pypi.python.org/pypi/tzlocal) - are installed. Then open _get\_historical\_sma.py_ in your IDE/editor of choice. The two important lines in this script are:

```python3
my_security = Security(my_ib, symbol='GOOG', secType='STK', exchange='SMART')
```

and

```python3
sma = my_security.get_historical_sma(length=150, barSizeSetting='1 day', ohlc='CLOSE', whatToShow='MIDPOINT', endDateTime='now')
```

The arguments of these two functions are what you'll edit to get your SMA of choice. The first function targets the security you want to get an SMA for, and it can be any ticker symbol, even symbols that aren't technically securities, like indexes such as VIX. The second function is where you'll define the parameters of your historical SMA. The example code retrieves a 150-day SMA for the midpoint (avg of bid/ask) of Google close prices. You can edit these arguments to get a 10-day IBM SMA of Open Bid prices or a 10-hour APPL High Ask or a 97 minute MSFT Last Bid or whatever you want. See IB's [reqHistoricalData() documentation page](https://www.interactivebrokers.com/en/software/api/apiguide/java/reqhistoricaldata.htm) for the acceptable argument inputs. There are also limitations to how far back your SMAs can reach: see [IB's Historical Data Limitations](https://www.interactivebrokers.com/en/software/api/apiguide/tables/historical_data_limitations.htm) page for details. You can also see the docstrings of the function definitions within the code if IB's online documentation is incomplete or otherwise sucks (which it often does).

There are several things to know about this code:

<ol>
<li>The SMAs don't include any after-hours values, only trading-hours values (most devs will want this functionality). This means that if you get a 10-hour SMA, for example, the SMA will include however many hours have passed in today's trading day so far and then it will jump to the final hour of the previous trading day and include hours from there. The SMAs never include after-hours information, even if an SMA spans multiple trading days.</li>
<li>You can enter a custom datetime into the <em>endDateTime</em> arg of the <em>get_historical_sma</em> function if instead of historical data that includes info from 1 second ago you want information just from last week, last month, or last year. For instance, if you want a 10-day SMA from the two weeks of June 15-26, 2015, pass in a datetime object set to June 26, 2015 anytime after trading hours.</li>
<li>As you play around with this code you'll invariably raise this error from IB:
<pre><code>error id=1, errorCode=162, errorMsg=Historical Market Data Service error message:invalid step: 1</code></pre>
The reason is because you've violated IB's historical data request limitations noted above. You may think you haven't violated the limitations, but you have: <a href = 'https://www.interactivebrokers.com/en/software/api/apiguide/tables/historical_data_limitations.htm'>IB's Historical Data Limitations</a> page doesn't always correspond with IB's true limitations. For example, with my paper trading account, I can get an SMA with <em>barSizeSetting='1-sec'</em> and <em>length=1998</em> but <em>length=1999</em> throws the error; I can get a 260 '30 mins' SMA but not 261. IB's documentation page doesn't mention anything about these oddly-numbered limits. This is a headache that you'll just have to work around.<br />
Also, Second and Minute SMAs can throw this error if the endDateTime is deep in after-hours (for instance, if endDateTime is set to 'now' and you're running this code after-hours). Let's say you're playing with this code at 11pm and the trading day has been over since 4pm: an SMA of <em>length=1000</em> and <em>barSizeSetting='1 sec'</em> will throw the 'invalid step' error. The solution is to set the <em>endDateTime</em> to today at 4pm, then the code will work.</li>
<li>The only other file you'll need to edit besides <em>get_historical_sma.py</em> is <em>exchange_info.py</em>, which holds important info on the trading exchange you're targeting, such as opening times, closing times and trading holidays. The info in the downloaded code is set for US exchanges so you'll need to edit this file if you're trading on a non-US exchange.</li>
</ol>

That's all there is to it. Leave a comment if you have any questions or if you think you've found a bug that needs fixing (or better yet, upload a patch and merge request on Github).
