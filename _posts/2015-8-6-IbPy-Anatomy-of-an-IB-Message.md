---
layout: post
title: IbPy - Anatomy of an IB Message
sitemap:
  lastmod: 2015-11-02
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
---

My [Getting Started with IbPy post]({% post_url 2015-8-4-IbPy-Getting-Started %}) shows how to receive and react to messages that IB sends to your application, but it doesn't explain how to extract the valuable information contained in those messages. In this post I explain the structure of an IB message and demonstrate how to extract its info.

### IB Messages Are Objects

The Getting Started post might have left you with the impression that messages sent from IB are strings, but they're not. They're actually objects, and as such, they have attributes, and the attributes are what contain the valuable information. For example,  one of the messages you'll automatically receive from IB as soon as you connect to its servers is the managedAccounts message. When you print that message you'll see:

```
<managedAccounts accountsList=123456789>
```

Printing the message shows all its attributes. The first attribute is typeName, which identifies the type of message, and every IB message has one. the value of the typeName attribute is the first word of a printed IB message (sans the left angle bracket), in this case _managedAccounts_. What follows is one or more attributes in _attribute=value_ pairs, each separated by a comma. This managedAccounts message has only one attribute other than typeName, which is _accountsList_, whose value is 123456789. Let's take a look at an error message, which has 4 attributes:

```
<error id=-1, errorCode=2104, errorMsg=Market data farm connection is OK:usfarm.us>
```

The attributes of this message are:

```
msg.typeName = 'error'
msg.id = -1
msg.errorCode = 2104
msg.errorMsg = "Marked data farm connection is OK: usfarm.us"
```

Building on this information along with the info in the [Getting Started]({% post_url 2015-8-4-IbPy-Getting-Started %}) post, let's code a program that requests the current exchange time from IB's servers and then extracts the current time from the IB message:

```python3
from ib.opt import Connection

def extract_time_from_currentTime_message(msg):
    print(msg)
    current_exchange_time = msg.time
    print("current exchange time: {}".format(current_exchange_time))
    
def main():
    conn = Connection.create(port=7496, clientId=100)
    conn.register(extract_time_from_currentTime_message, 'CurrentTime')
    conn.connect()
    
    conn.reqCurrentTime()
    
    import time
    time.sleep(1) #give IB time to send us messages
    
    conn.disconnect()
    
if __name__ == "__main__": main()
```

As you can see, targeting message attributes uses Python's simple, ubiquitous dot notation: _msg.time_. The output of the _extract\_time\_from\_currentTime\_message_ function is:

```
<currentTime time=1438900794>
current exchange time: 1438900794
```

For completeness' sake, here's how to convert the Unix timestamp that IB sends into a timezone-aware datetime object of your local time. Replace the _extract\_time\_from\_currentTime\_message_ function with this function (note that this requires installing the 3rd-party modules [pytz](https://pypi.python.org/pypi/pytz/#downloads) and [tzlocal](https://pypi.python.org/pypi/tzlocal)):

```python3
def extract_time_from_currentTime_message(msg):
    print(msg)
    
    from datetime import datetime
    import pytz, tzlocal
    current_exchange_time_east_coast = datetime.fromtimestamp(msg.time, pytz.timezone('US/Eastern'))
    current_exchange_time_local = current_exchange_time_east_coast.astimezone(tzlocal.get_localzone())
    
    print("current exchange time local timezone: {}".format(current_exchange_time_local))

```

### IbPy Preconverts Message Attributes to the Appropriate Python Type

IbPy does a good job converting message attributes into appropriate Pyhton types before it sends messages IB messages your callbacks. For instance, the msg.time attribute of a currentTime message is an integer, not a string:

```python3
def extract_time_from_currentTime_message(msg):
    print(type(msg.time))
```

outputs:

```
<class 'int'>
```

This is good because the _datetime.fromtimestamp()_ function requires an integer as its first argument - passing in a string would raise an exception. In the same way, when you request prices from IB, the message's price attributes are floats, etc. I haven't found one scenario where IbPy serves up an inappropriate type.

### Message Attributes Can Be Objects With Their Own Attributes

So far, every message type we've seen has had attributes whose values are Python built-in types: strings, integers, floats, etc. But this isn't always the case: more complex messages contain attributes whose values are custom objects themselves with their own attributes. For example, imagine that your program placed an order to go long on some stocks a few minutes ago and that order has just been executed; IB will send several different messages concerning this order execution, one of which is the execDetails message:

```
<execDetails reqId=-1, contract=<ib.ext.Contract.Contract object at 0x00000000035E2A58>, execution=<ib.ext.Execution.Execution object at 0x00000000035E2780>>
```

As you can see, the msg.contract and msg.execution attributes aren't simple integers or strings, they're custom IbPy objects that have their own attributes. In the same way we print out an IB message to see its attributes, you can also print out one of these 'sub-objects' to see its attributes, e.g. _print(msg.execution)_ outputs this (I've added the line breaks for readability):

```
execution=<ib.ext.Execution.Execution object at 0x00000000035E2780>
'm_acctNumber': '123456789',
'm_avgPrice': 0.37,
'm_clientId': 100,
'm_cumQty': 8,
'm_evMultiplier': 0,
'm_evRule': None,
'm_exchange': 'SMART',
'm_execId': '9999e999.999d999a.01.01',
'm_liquidation': 0,
'm_orderId': 18,
'm_orderRef': None,
'm_permId': 999999999,
'm_price': 0.37,
'm_shares': 8,
'm_side': 'BOT',
'm_time': '20150804  12:08:45'
```

Explaining what these values mean is beyond the scope of this blog post, but the point is that message objects can have attributes which are other objects, and those objects themselves can have attributes which are other objects, in a sort of branched hierarchy structure. Although this structure might seem complex, extracting valuable information from messages is easy once you know where the info within the message is located. For instance, if you need to get the _m\_avgPrice_ attribute from an execDetails message, it's just _value = msg.execution.m\_avgPrice_. Extracting information from IB messages is one of the primary tasks of any IbPy application, so hopefully this blog post has helped you learn exactly how that process works.













