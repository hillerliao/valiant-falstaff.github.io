---
layout: post
title: IbPy - register() vs registerAll()
description: The register and registerAll methods are the way your application reacts to info sent by IB. Learn how to use these crucial methods and the difference between them.
sitemap:
  lastmod: 2015-11-02
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
---

In my [blog post]({% post_url 2015-8-4-IbPy-Getting-Started %}) that shows how to get started using IbPy I detail how to use the _registerAll()_ method in an IbPy application. But the IB API also offers a _register()_ method. What's the difference, and when should you use one over the other (and can you use both in the same application)?

## Difference
The difference between the two is that _register()_ will target only one specific type of message, while _registerAll()_ targets any and all message types. For instance, this line

```python
conn.register(callback, 'Error')
```

makes it so that the callback is called only if an error message is received, not any other message type. The _registerAll()_ method, on the other hand, reacts to any and all messages no matter the message type. Unlike _registerAll()_, which takes a single argument (the callback), _register()_ takes two arguments, the first being the callback and the second being a string denoting the message type. What are the different types of messages IB might send us? You've already encountered a few types in the Getting Started post: managedAccounts, nextValidId, and error. The full list of message types can be found at [IB's EWrapper Methods](https://www.interactivebrokers.com/en/software/api/apiguide/java/java_ewrapper_methods.htm) documentation page. When you visit this page, you'll see that the list is made up of method names, not names of message types. But the method names are the same as the names of the message types you can pass into the _register()_ method, with one exception: you must capitalize the first letter of the method name. For instance, if you want to do something specific for all error messages, you can check this list and find that it includes 'error()' methods; therefore, you'll pass in the string 'Error' as the second arg of the register() method. If you want your program to do something specific when receiving historical data, check the list to find the historicalData() method and pass in the string 'HistoricalData'. A complete explanation of what all these message types mean is beyond the scope of this blog post, but the point is that once you know that you want your program to react in a special way to a specific message type, you can use this list to determine the exact syntax to pass into the register() method.

## Capitalization Confusion
One confusing thing about the syntax of message types is that IB capitalizes the first letter in some contexts but doesn't in other contexts. For instance, when you pass a message type into the _register()_ method, the first letter of the message type must be capitalized. But when IB sends your program messages, the first letter of the message type is NOT capitalized. Hence, in the [Getting Started]({% post_url 2015-8-4-IbPy-Getting-Started %}) post, the output of printed messages is 'managedAccounts', nextValidId', and 'error' instead of ManagedAccounts', 'NextValidId', and 'Error'. This is code cruft, which you'll run into regularly in the IB API, and you'll just have to remember this quirk and code around it. As a concrete example, let's say that when you register a function to handle error messages, you want that function to make absolute sure that the message it's dealing with is actually an error message:

```python
from ib.opt import Connection

def raise_error(msg):
    assert(msg.typeName == 'error')
    raise Exception("We've received an error message from IB: {}".format(msg))

def main():
    conn = Connection.create(port=7496, clientId=100)
    conn.register(raise_error, 'Error')
    conn.connect()
    
    #Simply for demonstration purposes, to give
    #IB's servers time to send us messages
    import time
    time.sleep(2) 
    
    conn.disconnect()
    
if __name__ == "__main__": main()
```

The line to focus on here is _assert(msg.typeName == 'error')_, which detects the message object's type by getting its 'typeName' attribute, an attribute that all messages sent from IB will have. The typeName attribute is a string and its first letter is always lowercase, no matter the message type. As seen in the code example above, this is in contrast to the string passed in to the _register()_ method, whose first letter should always be capitalized.

## When to use register() vs registerAll()
Use registerAll() when you want to make your program perform a specific reaction to any and all messages, regardless of message type. Use register() when you want your program to perform a specific reaction only to message of some message types but not all. The _register()_ and _registerAll()_ methods aren't mutually exclusive: you can use them both in the same script, and in fact you can use each of them multiple times. Let's say, for instance, you want to print out all messages, save any historical prices into a list, and raise an exception if IB sends any 'error' messages:

```python
conn.registerAll(print_message)
conn.register(save_historical_prices, 'HistoricalData')
conn.register(raise_exception, 'Error')
```

You can use _register()_ and _registerAll()_ as many times as you want. You can also pass in more than one string argument to _register()_ if you want to perform the same action on more than one message type. For instance, let's say you want to print out any time messages and 'next valid order id' messages that IB sends:

```python
conn.register(print_message, 'CurrentTime', 'NextValidId')
```

## Order Matters
Be aware that the order in which you call _register()_ and _registerAll()_ matters if you have multiple callbacks registered for the same message type. When your program receives a message from IB, it will start by calling the callback that was first registered for that message type, then it will call the second callback registered for that message type, then the third, etc. So for instance, if your lines of code are in this order:

```python
conn.register(raise_exception, 'Error')
conn.registerAll(print_message)
```

then an exception will be raised _before_ the error message is printed out, which may not be what you want. Putting _conn.registerAll(print\_message)_ before _conn.register(raise\_exception, 'Error')_ will make it so that the error message is printed out before the exception is raised.

## Only One Message-Processing Thread Ever Exists
You might wonder whether if every time you call _register()_ or _registerAll()_, a new thread is started to watch for a specific message type. This is not the case. A new thread is started only the first time you call _register()_ or _registerAll()_, but not on subsequent calls. Rather, on subsequent calls the existing message-processing thread is notified to also watch out for x message types _in addition_ to the message types it's already watching out for. No matter how many times you call _register()_ and _registerAll()_, only one message-processing thread exists.

## unregister() and unregisterAll()
You can also unregister callbacks with _unregister()_ and _unregisterAll()_. For instance, let's say you want your program to start out by printing any error messages, but then after a while you want it to stop printing such messages:

```python
conn.register(print_message, 'Error')
#here's where your program will do stuff until you want to stop printing error messages
conn.unregister(print_message, 'Error')
```

the _unregister()_ method is also helpful when you want to perform an action in response to every message type except one or two. For instance, let's say you want to print out every single message type except errors. To achieve this, you could either call _register()_ on every single message type except 'Error':

```python
conn.register(print_message, 'CurrentTime', 'ConnectionClosed', 'TickPrice', 'TickSize', 'TickOptionComputation', 'TickGeneric') #I'm not actually gonna write 'em all out - you get the idea
```

Or you could simply do:

```python
conn.registerAll(print_message)
conn.unregister(print_message, 'Error')
```

That's all there is to _register()_ and _registerAll()_. Hope this article helped!