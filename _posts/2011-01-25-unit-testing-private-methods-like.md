---
layout: post
title: Unit testing private methods like a Software Engineer
date: '2011-01-25T01:02:00.001-06:00'
author: Jonathan Cone
tags: 
modified_time: '2011-01-26T19:48:31.808-06:00'
blogger_id: tag:blogger.com,1999:blog-850212609563989685.post-5134458037586504464
blogger_orig_url: http://www.machineversus.me/2011/01/unit-testing-private-methods-like.html
---
Right now, there are three conventional schools of thought on unit testing private methods:

* Make the private method protected to facilitate unit testing
* Use some reflective utility to make the private method accessible from the test
* Don't unit test private methods since they aren't part of the public API

The first two obviously break encapsulation which leads to tight-coupling, which, as we all know, leads to less maintainable code and probably gang violence. Because of this, I consider these solutions to be a hack or a kludge:

> (n) **kludge**: a badly assembled collection of parts hastily assembled to serve some particular purpose (often used to refer to computing systems or software that has been badly put together)

And since the title of the entry contains the phrase **like a Software Engineer**, we'll distance ourselves from either of these approaches. 

But before we get too far, if you're working with properly defined abstractions the third approach is actually viable.  If you have cohesive classes that follow the <a href="http://en.wikipedia.org/wiki/Single_responsibility_principle">single responsibility principle</a>, your private methods can probably be implemented by public methods on another class.  Read that sentence again if you didn't understand it the first time.  I've heard engineers say things along the lines of "**That's great, but it doesn't really work in my scenario.**"  Generally, this is an indication that the abstraction needs to be reevaluated, and likely refactored into more <a href="http://en.wikipedia.org/wiki/Granularity">granular</a> chunks of complexity.

Now, if you still want to unit test a private method, _there is another method you can use that won't break your black box_.  Let's walk through an example.  Note that this is extremely contrived, but I hope you can see how the principle applies:

**1) Examine your production code.**  In this case we have a `Calculator` class with a private `square` method we want to test which is being called from `getSquare`:

{% highlight java %}
public class Calculator {

  public String getSquare(int x) {
    return "The square of the number is: " + square(x);
  }

  private int square(int x) {
    return x * x;
  }

}
{% endhighlight %}

**2) Create a test class to test the public interface.** Our public interface is the `getSquare` method of the `Calculator` class:

{% highlight java %}
public class CalculatorTest {
  Calculator calc = new Calculator();

  @Test
  public void getSquareOf6() {
    // TODO
  }
}
{% endhighlight %}

**3) Create a method in your test class that has the same implementation contract as the private method you'd like to test.**  This is the interesting part, the code in your test might even be the same as the private method's production code, but for the sake of the example, its not.  We want to emphasize _testing the contract semantic, not how the semantic is fulfilled_:

{% highlight java %}
public class CalculatorTest {
  Calculator calc = new Calculator();

  @Test
  public void getSquareOf6() {
    // TODO
  }

  // Different implementation, same contract semantics
  private int doSquare(int number) {
    int x = 0;
    for(int i = 0; i < number; i++) {
      x += number;     
    }
    return x; 
  }
}
{% endhighlight %}

4) Create a series of tests to test the new method you've created. This will ensure that you've validated the correctness of your implementation.  _These are the unit tests for your private method._  You're going to test your private method by testing a **completely different** method and comparing the results of the well-tested method with the results of your untested private production code method.

{% highlight java %}
public class CalculatorTest {
  Calculator calc = new Calculator();

  @Test
  public void getSquareOf6() {
    // TODO
  }

  private int doSquare(int number) {
    int x = 0;
    for(int i = 0; i < number; i++) {
      x += number;     
    }
    return x; 
  }

  @Test
  public void doSquare2() {
    assertEquals(4, doSquare(2));
  }

  @Test
  public void doSquare4() {
    assertEquals(16, doSquare(4));
  } 
  // More tests
}
{% endhighlight %}

5) At this point, we're pretty confident in the code we wrote within the test class, **but we still have yet to test our private method through the public interface provided by the production class**:

{% highlight java %}
public class CalculatorTest {
  Calculator calc = new Calculator();

  @Test
  public void getSquareOf6() {
	// expected, actual
	assertEquals("The square of the number is: " + doSquare(6), calc.getSquare(6));
  }

  private int doSquare(int number) {
    int x = 0;
    for(int i = 0; i < number; i++) {
      x += number;     
    }
    return x; 
  }

  // Other tests, etc. truncated for clarity
}
{% endhighlight %}

In summary, we've tested the public interface of our production code, but we've also **written tested code** to validate the implementation of the public method _without_ exposing the implementation details.  As I mentioned, generally its preferable to ensure that you're using the proper abstractions. In this contrived case the number squaring behavior might have been abstracted behind a class that implements behaviors dealing with exponents.  Also, our public method was pretty straight forward as well.  Your method might return a more complex object, but this technique will still apply.
