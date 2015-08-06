---
layout: post
title: Getting Started with IbPy
---

Interactive Brokers, the popular online brokerage firm, has an API that lets you code automated trading applications, but the API doesn't support Python. Fortunately, several talented programmers have written IbPy, an implementation of the API for Python. Having spent several months working on a recent IbPy project, I can confirm that IbPy works just as well as the 'official' Java/C++/C# APIs. In this post, I'll show you how to get started using IbPy, and in subsequent posts I'll demonstrate specific tasks such as placing orders, getting option chains, getting real-time prices, SMAs, EMAs, and more. 

### Setting Up
#### 1. Install IbPy
IbPy requires Python 2.5 or newer, and thanks to the work by [Ben Alex](https://github.com/benalexau), IbPy also supports Python 3, which is the syntax I'll be using in this blog (onward and upward). Download the [IbPy module](https://github.com/blampe/IbPy) at GitHub and install using the included setup.py file (see [this page](https://code.google.com/p/ibpy/wiki/GettingStarted) for platform-specific setup instructions).

#### 2. Install Trader Workstation
Trader Workstation (TWS) is Interactive Brokers' nonprogrammer-friendly standalone GUI that allows anyone with an IB account to trade directly from their computer. TWS is also the means by which a coded application connects programatically to IB's servers. Download the [TWS installer](https://www.interactivebrokers.com/en/index.php?f=552&ns=T) and install on your local machine. Note that as of this writing (August 2015) there are two download choices, TWS and TWS Latest, and only TWS Latest supports API connections, so make sure to choose TWS Latest.  

The TWS installer is straightforward except for this dialog which asks whether you want to install IB Information System:

![TWS Installer IBIS]({{ site.baseurl }}/images/ibpy/getting_started/tws-installer-ibis-choice.png)

IB Information System is irrelevant to the API, so no need to install it. But you'll also notice that the installer requires you to install two programs rather than just one: Trader Workstation _and_ IB Gateway. These two programs can't be installed separately, they are installed as a set. IB Gateway is made specifically for API users: you can use it as an alternative to TWS to connect to IB's servers. The advantage of using IB Gateway is that it doesn't require as much processing power and memory as TWS. The disadvantage of IB Gateway is that the visual feedback it displays on what your coded application is doing is far less organized and readable than in TWS. For instance, using TWS, if our coded application places an order for 100 Google stocks, that order will show up immediately in the _orders_ table of TWS exactly as if we had placed the order manually:

![TWS Google Order Screenshot]({{ site.baseurl }}/images/ibpy/getting_started/tws-screenshot-google-order.png)

IB Gateway, rather, displays a cryptic log message after the order is placed:

![IB Gateway Google Order Screenshot]({{ site.baseurl }}/images/ibpy/getting_started/ib-gateway-screenshot-google-order.png)

It's completely up to you whether to use TWS or IB Gateway to connect to IB's servers: both do the same crucial job of relaying information back and forth between your program and IB's servers, and they do it in the same exact way (your Python application won't receive different messages from IB's servers if you use IB Gateway instead of TWS, for instance). The only difference is that TWS provides easier-to-read feedback at a small CPU and memory cost. I use TWS. _(Side note: IB has a web-based alternative to TWS called WebTrader, but WebTrader doesn't support API connections - TWS and IB Gateway are your only two API connection options.)_

#### 3. Configure TWS / IB Gateway

Once you have TWS / IB Gateway installed, you'll need to change some configuration settings in order for your software to successfully connect to IB's servers. (The following instructions use TWS as the example, but the IB Gateway instructions are virtually identical.) Navigate to the API Settings within Global Configuration in TWS. The exact way to open API settings changes with TWS updates: sometimes the click path is Edit->Global Configuration->API->Settings and then a week later the edit button will be gone and the click path will be File->Global Configuration->API->Settings. Poke around to find it. Once in API Settings, make these changes:

1. Check the box "Enable ActiveX and Socket Clients."
2. Set the "Socket port" to an unused port; most online tutorials recommend 7496, which is unassigned by default. It's what I use and I've never had a problem.
3. Set the "Master API client ID" in the settings to 100
4. Create a Trusted IP Address set to 127.0.0.1

![TWS API Settings]({{ site.baseurl }}/images/ibpy/getting_started/tws-api-global-configuration-settings.png)

See [IB's API Connections page](https://www.interactivebrokers.com/php/whiteLabel/globalConfig/configureApi.htm) for additional information on these settings.

### Your first IbPy program
Now that everything is set up, it's time to open your IDE of choice and run your first IbPy script:

```python3
from ib.opt import Connection

def print_message_from_ib(msg):
    print(msg)
    
def main():
    conn = Connection.create(port=7496, clientId=100)
    conn.registerAll(print_message_from_ib)
    conn.connect()
    
    #In future blog posts, this is where we'll write code that actually does
    #something useful, like place orders, get real-time prices, etc.
    
    import time
    time.sleep(1) #Simply to give the program time to print messages sent from IB
    conn.disconnect()
    
if __name__ == "__main__": main()
```

When you run this code, the output will be something like this:  

    Server Version: 76  
    TWS Time at connection:20150804 10:16:44 MST  
    <managedAccounts accountsList=123456789>  
    <nextValidId orderId=1>  
    <error id=-1, errorCode=2104, errorMsg=Market data farm connection is OK:usfarm.us>  
    <error id=-1, errorCode=2104, errorMsg=Market data farm connection is OK:usfarm>  
    <error id=-1, errorCode=2106, errorMsg=HMDS data farm connection is OK:ushmds>  

If you get this output:  

    <error id=-1, errorCode=502, errorMsg=Couldn't connect to TWS.  Confirm that "Enable ActiveX and Socket Clients" is enabled on the TWS "Configure->API" menu.>  

it means that you forgot to start TWS, and herein lies an important point: TWS needs to be running in order for your software to connect to IB's servers; it's not enough simply to have TWS installed on your computer.

So you've run this program, but you're not sure what it means (why does the output contain errors?) and it doesn't appear to have done anything useful. The truth is, this program doesn't do anything terribly useful: it doesn't place any orders or get any real-time prices or anything like that. All it does is connect to IB's servers, print out any messages that IB automatically sends to us after we're connected, and then disconnect from IB's servers. Pretty boring, I know, but before we dive into the fun stuff, it's crucial that you understand how to connect to IB and how to react to information that IB sends to your program. So let's analyze the the main() function line-by-line:

```python3
conn = Connection.create(port=7496, clientId=100)
```
You might think that this line connects to IB's servers, but it doesn't; all it does is create a Connection object in your code. If this were the only line in main(), you wouldn't receive any messages from IB because you haven't actually connected to IB's servers. But this Connection object is what we'll use to connect to IB later on in the program - it's the crucial first step in an IbPy application. The value of the port argument needs to match the "Socket port" in the TWS API settings. The value of the clientId arg doesn't actually need to match the "Master API client ID" in the TWS API settings - you could pass in 132 or 47 or 1 as the argument and the program would work fine - but unless you're running multiple different scripts to connect to the same IB account, there's no need to pass in a different value, so keep things simple by passing in the same value as "Master API client ID".

```python3
conn.registerAll(print_message_from_ib)
```
This line is what causes the program to react to the messages that IB sends to it. If this line didn't exist, our program wouldn't do anything when IB sends us a message; the message would just arrive at our computer and then disappear forever. When we call registerAll(), the program starts a separate thread in the background whose job is to do nothing but wait for messages that IB sends to the program, and once it receives a message from IB, react to the message. How does it react? By calling the function we pass in as an argument to registerAll() (passing a function as an argument to another function is known as the [callback mechanism](http://stackoverflow.com/questions/1319074/parallel-python-what-is-a-callback)). In this case we passed in the _print\_message\_from\_ib_ function, which doesn't do anything interesting, it simply prints out the message for demonstration purposes. In a real-world case this function would save historical stock prices to a list or save the brokerage fees of a financial order or set a flag confirming that a stock has dipped below a certain price, etc.

<div class="newbie-notes">
<h4>Newbie Notes</h4>
<ol>
<li><a href="http://inventwithpython.com/blog/2013/04/22/multithreaded-python-tutorial-with-threadworms/">What's a thread?</a></li>
<li>When you pass in the reaction function as the argument to registerAll() (this reaction function is known as the 'callback'), don't call it, just pass it in the function itself. In other words, <em>conn.registerAll(print_message_from_ib())</em> is wrong.
If you're new to programming, you might be confused as to why you would ever write out a function name without calling it, e.g. why ever write <em>print_message_from_ib</em> instead of <em>print_message_from_ib()</em>? And you're even more confused about how a function can be passed in as an argument of a separate function call. As the teacher, I now have a choice: I could explain to you that functions are objects just like strings and numbers are, and they can be arguments to function calls just like strings and numbers can. But the problem with this approach is that a discussion of objects can also fly over the newbie's head. The antidote is <a href="https://www.jeffknupp.com/blog/2013/02/14/drastically-improve-your-python-understanding-pythons-execution-model/">this excellent tutorial</a> by Jeff Knupp, but if for now you're just interested in getting going as quickly as possible, then let me explain in plain english that when you pass in <em>print_message_from_ib</em> as an argument, you're essentially telling the separate thread that reacts to IB messages, "Hey, when we get a message from IB, react by calling the function that I'm sending you. I'm not calling this reaction function right now, I'm just showing you which function <em>you</em> should call when IB sends us a message."</li>
<li>You'll notice that <em>conn.registerAll(print_message_from_ib)</em> doesn't make any mention of which or how many arguments the <em>print_message_from_ib</em> function (AKA the callback) takes. So how is the separate thread supposed to know what arguments to pass into the callback? The answer is that it assumes it should only pass one argument into the callback, and that that argument should be the message received from IB. Therefore when you write the function definition of your callback, it must have only one parameter: the IB message. (If you need to pass more than one argument in to the callback, I show how in this blog post.)
</li>
</ol>
</div>

I should also mention that the Connection object has both a registerAll() and a register() method. For simplicity's sake I'm only using the registerAll() method in this blog post, but I've devoted a separate blog post to explaining the difference between register() and registerAll() and when should you use one over (or in combination with) the other.

```python3
conn.connect()
```
This line actually connects to IB's servers. The most important thing to know here is that you should call this _after_ calling _registerAll()_, not before, because once you connect, IB automatically sends you messages without you requesting them. At the bare minimum, it sends you a message saying that your connection to its servers was successful, but it will also send you order information on any orders that the software has placed in the past 24 hours. And sometimes (depending on when the order was executed and when the last time your software connected to IB's servers was), these order messages cannot be retrieved any other way, so if you don't process these messages immediately after calling _connect()_, you could lose the info in those messages forever. The exact details on handling order messages from IB is a discussion for another blog post, but my whole point is that it's crucially important that your callbacks are registered with the _registerAll()/register()_ methods _before_ you call the _connect()_ method. if you run _register()/registerAll()_ after _connect()_, messages might arrive from IB's servers before the _register()/registerAll()_ methods get the callbacks registered (a classic example of a race condition), in which case those messages disappear. To be clear, the messages don't sit in some list waiting to be handled - if you don't have any callbacks registered, the messages will pass through your program unhandled and you may never be able to re-request them from IB.

```python3
import time
time.sleep(1) #Simply to give the program time to print messages sent from IB
```
As the comment explains, the only reason this small wait exists is for the demonstration purposes of this tutorial: it gives IB's servers some time to send messages to our program and to gives our separate thread that processes IB messages some time to print messages to the output. Without a little wait, you'd _disconnect()_ before IB has a chance to send you any messages. 

```python3
conn.disconnect()
```
The _disconnect()_ method disconnects from IB's servers, but that's all it does: it doesn't destroy your Connection object or change/destroy the callback registry that you created with registerAll(). All that information is retained. If you want to re-connect, simply call conn.connect() again - no need to re-create another Connection object or re-call registerAll().

Finally, let's go over the program's output. You may be wondering why we did nothing in this blog post with the messages that IB sent to us except print them out. How is that helpful to you? After all, when you write your IbPy applications, you'll want to do more with IB messages than just print them: you'll want to extract important information from them and react appropriately. Why doesn't this post show how to do that? Because that topic is worthy of its own blog post. For now, just know that the information you see in the printed output can indeed be extracted by your program and that other posts will show you how to do that when such info is needed to complete tasks.

    Server Version: 76
	TWS Time at connection:20150804 10:16:44 MST
	<managedAccounts accountsList=123456789>
	<nextValidId orderId=1>

This info is automatically sent by IB every time you successfully connect. I won't go over what it all means in this post; for now just be aware that your program should be expecting this info to be sent automatically from IB upon a successful connection.

    <error id=-1, errorCode=2104, errorMsg=Market data farm connection is OK:usfarm.us>
    <error id=-1, errorCode=2104, errorMsg=Market data farm connection is OK:usfarm>
    <error id=-1, errorCode=2106, errorMsg=HMDS data farm connection is OK:ushmds>

These errors aren't errors at all, they actually mean that you've successfully connected to IB's servers and that the connection is working properly. I know, it's stupid, and this is just one of the several instances of code cruft you'll find in the IB API. Fortunately, none of the code cruft I've run across has hindered my ability to actually do things, they're just odd quirks that you can ignore or work around once you know about them. When you get a message with errorCode 2104 or 2106, you can safely ignore it (as I do) or set a flag in your program saying that the connection is working properly. These aren't the only faux error messages: the [IB API Error Codes page](https://www.interactivebrokers.com/en/software/api/apiguide/tables/api_message_codes.htm) will help you determine which error messages are genuine errors and which aren't. If you're still in doubt, [call IB Technical Assistance](https://www.interactivebrokers.com/en/?f=customerService). I've found IB's customer service team to be pretty knowledgeable; after having called them more than 75 times (my mom is so jealous), I give them a ~92% success rate in correctly answering my questions; sometimes an employee will give me incorrect information, but that's to be expected in any medium to large company.

Hopefully this post gives you a leg up in getting started with IbPy. Feel free to leave a comment with any questions, and see my other blog posts to get started coding the real juicy juicy.