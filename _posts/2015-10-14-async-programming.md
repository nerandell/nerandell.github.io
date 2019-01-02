--- 
layout: post 
title: Using asyncio.gather in python
date: 2015-10-14 12:32:18
summary: Using asyncio.gather in python to write efficient code
comments: true 
categories: programming
thumbnail: programming
tags:
 - python
 - asyncio
 - efficiency
 - microservices
--- 

We are currently in process of rewriting our entire Java based monolithic app into microservice based architecture using [Vyked][1], our in-house open source python based microservice framework based on `asyncio` which is a part of Python standard library since 3.4. Recently, one of my coworkers came to me with what he thought was a bug in Vyked. We currently have timeout for an api call set to 2 minutes which he found not to be enough for one of the apis he had written. The api essentially wrote some data to an excel file and then prepare the browser to download it. I found it suspicious that it would take that long and started looking at his code. After some investigiation, I found that this was the block of code that took time:

{% highlight python %}
def get_delivery_queue_order_details(self, order_ids): 
	results = [] 
	for order_id in order_ids: 
		details = yield from self._a_network_call(order_id) 
		result = self._process_details(details) 
		results.append(result) 
	return results 
{% endhighlight %}

The code basically returns a list of results. However for each item in this list, a network call has to be made in form of a rpc. When this loop runs, because of the `yield from` statement, the loop will block till the network call is successful and cpu will be idle during this duration. The point to note here is that fetching one item for this list is completely independant of fetching other items. So there is no point blocking the loop and waiting for the result and then moving to the next iteration. We made a simple change and the time taken reduced drastically. This was the fix: 

{% highlight python %}
def order_detail(order_id): 
	details = yield from self._a_network_call(order_id) 
	return self._process_details(details) 

def get_delivery_queue_order_details(self, order_ids): 
	return (yield from asyncio.gather(*[self.order_detail(order_id) for order_id in order_ids]) 
{% endhighlight %}

From Python documentation, this is what `asyncio.gather` does: 

``` 
asyncio.gather(*coros_or_futures, loop=None, return_exceptions=False)
Return a future aggregating results from the given coroutine objects or futures.

All futures must share the same event loop. If all the tasks are done successfully, the returned future’s result is the list of results (in the order of the original sequence, not necessarily the order of results arrival). If return_exceptions is True, exceptions in the tasks are treated the same as successful results, and gathered in the result list; otherwise, the first raised exception will be immediately propagated to the returned future.

Cancellation: if the outer Future is cancelled, all children (that have not completed yet) are also cancelled. If any child is cancelled, this is treated as if it raised CancelledError – the outer Future is not cancelled in this case. (This is to prevent the cancellation of one child to cause other children to be cancelled.)
``` 

After this change, instead of waiting for `self._a_network_call` to return, concurrent calls are made and we wait for the list of results. I started looking at other places where developers might have made the same mistake and to my surprise, I found several instances where `asyncio.gather` would have worked much more efficiently.

 [1]: https://github.com/kashifrazzaqui/vyked
