---
layout: post
title: IbPy - Getting a Historical SMA
---

_Note: This blog post builds on topics covered in previous posts: see the [Getting Started with IbPy]({% post_url 2015-8-4-IbPy-Getting-Started %}), [register() vs registerAll()]({% post_url 2015-8-5-IbPy-register-vs-registerall %}), [Anatomy of an IB Message]({% post_url 2015-8-6-IbPy-Anatomy-of-an-IB-Message %}), and [Passing Arguments to a Callback in register()/registerAll()]({% post_url 2015-8-6-IbPy-passing-args-to-callback-in-register-registerall %}) blog posts if you're brand new to IbPy._

When I first started working with the Interactive Brokers API, I assumed a task as common as getting a historical Simple Moving Average would be baked into the API - something like getHistoricalSMA(). I was wrong: getting an SMA is actually quite an involved process. That's why I've created a package set for IbPy users that makes it easy to get a historical SMA. 

First, download the [package set from Github](https://github.com/valiant-falstaff/IbPy-Get-Historical-SMA), then make sure that the three 3rd-party modules this code uses - [dateutil](https://labix.org/python-dateutil), [pytz](https://pypi.python.org/pypi/pytz/), and [tzlocal](https://pypi.python.org/pypi/tzlocal) - are installed. Then open _get\_historical\_sma.py_ in your IDE/editor of choice. The two important lines in this script are: