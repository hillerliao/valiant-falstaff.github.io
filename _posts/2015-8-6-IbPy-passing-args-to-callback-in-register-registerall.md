---
layout: post
title: IbPy - Passing Arguments to a Callback in register()/registerAll()
---

In my [blog post]({% post_url 2015-8-4-IbPy-Getting-Started %}) that shows how to get started using IbPy I detail how to register a callback with IbPy's _register()_ and _registerAll()_ methods so that your software can react to messages sent from IB. But the example code in that tutorial is limited in that it only allows the callback to have a single argument passed into it (the message from IB). But what if you want to pass additional arguments in to the callback, how would you do it? The easiest way is with a lambda function. Instead of explaining the basics of the lambda function (which you can find out for yourself online - I recommend [this tutorial](http://www.python-course.eu/lambda.php)), let me instead jump right into an example. Let's say you want to print out every message that IB sends, and at the beginning of the message you also want to print out the name of the person currently running the program. To do this, we'll pass a user's name as an argument to the callback. Let's call our user Rosalind:

```python3
from ib.opt import Connection

def print_message_from_ib(msg, username):
    print("{}: {}".format(username, msg))
    
def main():
    current_user = 'Rosalind'
    
    conn = Connection.create(port=7496, clientId=100)
    conn.registerAll(lambda msg: print_message_from_ib(msg, current_user))
    conn.connect()
    
    #This is where you'll do useful stuff like place orders, get real-time prices,
    #etc. For demonstration purposes, we'll just wait a few seconds to let IB send
    #us some messages that we can print out.
    import time
    time.sleep(2) #give the program time to print messages sent from IB
    
    conn.disconnect()
    
if __name__ == "__main__": main()
```

This outputs:

```
Server Version: 76
TWS Time at connection:20150803 10:55:23 MST
Rosalind: <managedAccounts accountsList=123456789>
Rosalind: <nextValidId orderId=1>
Rosalind: <error id=-1, errorCode=2104, errorMsg=Market data farm connection is OK:usfarm.us>
Rosalind: <error id=-1, errorCode=2104, errorMsg=Market data farm connection is OK:usfarm>
Rosalind: <error id=-1, errorCode=2106, errorMsg=HMDS data farm connection is OK:ushmds>
```

As you can see, instead of calling registerAll() without any additional arguments, as so:

```python3
conn.registerAll(print_message_from_ib)
```

we called it with a lambda function as its argument, which allows us to pass in the current_user variable:

```python3
conn.registerAll(lambda msg: print_message_from_ib(msg, current_user))
```

You can pass in as many arguments as you want using lambda. Let's say your _print\_message\_from\_ib_ function definition has 7 parameters instead of just 2. No problem:

```python3
conn.registerAll(lambda msg: print_message_from_ib(msg, current_user, arg_3, arg_4, arg_5, arg_6, arg_7))
```