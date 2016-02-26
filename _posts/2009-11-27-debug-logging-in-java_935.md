---
layout: post
title: Debug logging in Java
date: '2009-11-27T15:10:00.002-06:00'
author: Jonathan Cone
tags:
- Java
- Logging
modified_time: '2010-09-15T19:33:01.388-05:00'
blogger_id: tag:blogger.com,1999:blog-850212609563989685.post-7704880851829909597
blogger_orig_url: http://www.machineversus.me/2009/11/debug-logging-in-java_935.html
---

Some people really dislike debug log messages in production code.  Though they can be extremely useful, I think a lot of developers are turned off to the fact that when logging is done poorly, it really makes the code look like a mess.  For example, we've all seen code like this:

{% highlight java %}
public class PressureValveController {
  public void closeValve() {
    logger.debug("my out temp" + temperature + " press: " + pressure + " and flowRate " + flowRate);
  }
}
{% endhighlight %}

The developer basically just dumped the stack to debug a specific problem they were having.  This is one of a combination of factors that make code painful to read when logging is involved:

* poor/no grammar
* multiple log statements per method 
* excessive string concatenation

Sometimes, I like to create a helper method for logging in Java to make things more readable:

{% highlight java %}
private void debug(String message, Object... args) {
  if(logger.isDebugEnabled()) {
    logger.debug(String.format(message, args));
  }
}
{% endhighlight %}

The next thing to do is to follow certain guidelines with respect to uniformity and sentence structure.  For example:

{% highlight java %}
public class PressureValveController {
  public void closeValve() {
    debug("Closing valve with output temperature: %d, pressure: %f PSI, and a flow of %f CFM.", temperature, pressure, flowRate);
  }
}
{% endhighlight %}

I think this ends up being a lot more useful in terms of code clarity and actual output.  Loggers like slf4j and log4j2 can already do this out of the box and even provide support for deferred lambda expressions:

{% highlight java %}
logger.trace("Some long-running operation returned {}", () -> expensiveOperation());
{% endhighlight %}
