---
layout: post
title: IbPy - Getting a Historical SMA
---

_Note: This blog post builds on topics covered in previous posts: see the [Getting Started with IbPy]({% post_url 2015-8-4-IbPy-Getting-Started %}), [register() vs registerAll()]({% post_url 2015-8-5-IbPy-register-vs-registerall %}), [Anatomy of an IB Message]({% post_url 2015-8-6-IbPy-Anatomy-of-an-IB-Message %}), and [Passing Arguments to a Callback in register()/registerAll()]({% post_url 2015-8-6-IbPy-passing-args-to-callback-in-register-registerall %}) blog posts if you're brand new to IbPy._

When I first started working with the Interactive Brokers API, I assumed a task as common as getting a historical Simple Moving Average would be baked into the API - something like getHistoricalSMA(). I was wrong: getting an SMA is actually quite an involved process. That's why I've created a package set for IbPy users that makes it easy to get a historical SMA. 

First, download the [package set from Github](https://github.com/valiant-falstaff/IbPy-Get-Historical-SMA), then make sure that the three 3rd-party modules this code uses - [dateutil](https://labix.org/python-dateutil), [pytz](https://pypi.python.org/pypi/pytz/), and [tzlocal](https://pypi.python.org/pypi/tzlocal) - are installed. Then open _get\_historical\_sma.py_ in your IDE/editor of choice. The two important lines in this script are:

```python3
my_security = Security(my_ib, symbol='GOOG', secType='STK', exchange='SMART')
```

and

```python3
sma = my_security.get_historical_sma(length=150, barSizeSetting='1 day', ohlc='CLOSE', whatToShow='MIDPOINT', endDateTime='now')
```

The arguments of these two functions are what you'll edit to get your SMA of choice. The first function targets the security you want to get an SMA for, and it can be any ticker symbol, even symbols that aren't technically securities, like indexes such as VIX. The second function is where you'll define the parameters of your historical SMA. The example code retrieves a 150-day SMA for the midpoint (avg of bid/ask) of Google close prices. You can edit these arguments to get a 10-day IBM SMA of Open Bid prices or a 10-hour APPL High Ask or a 97 minute MSFT Last Bid, etc. See IB's [reqHistoricalData() documentation page](https://www.interactivebrokers.com/en/software/api/apiguide/java/reqhistoricaldata.htm) for the acceptable argument inputs. There are also limitations to how far back your SMAs can reach: see [IB's Historical Data Limitations](https://www.interactivebrokers.com/en/software/api/apiguide/tables/historical_data_limitations.htm) page for details. You can also see the docstrings of the function definitions within the code if IB's online documentation is incomplete or otherwise sucks (which it often does). There are several things to know about this code: