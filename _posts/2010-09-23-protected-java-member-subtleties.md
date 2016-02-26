---
layout: post
title: Protected Java Member Subtleties
date: '2010-09-23T22:53:00.000-05:00'
author: Jonathan Cone
tags: 
modified_time: '2010-09-23T22:53:00.973-05:00'
blogger_id: tag:blogger.com,1999:blog-850212609563989685.post-5215143357441409947
blogger_orig_url: http://www.machineversus.me/2010/09/protected-java-member-subtleties.html
---

Every legitimate Java developer knows there's four access levels and three access modifier keywords in Java.  If you were an instructor and had to pick one to give a quiz on, which one would it be? Ironically, the default level is probably the access level you use the least.  But what about protected?  Take this subtlety you might not have thought about into consideration.  Here is our superclass:

{% highlight java %}
package org.iaea;

public class EnergySource {
  protected long efficiency;
}
{% endhighlight %}

Consider the following variation (1) of a subclass:

{% highlight java %}  
package org.iaea;

public class UraniumBasedReactor extends EnergySource {

  public void bringOnline() {
    EnergySource other = new EnergySource();
    this.efficiency = 0;
    other.efficiency = 12; // Works Fine!
  }
}

{% endhighlight %}

and now this one (2):

{% highlight java %}
package org.iaea.uranium;

import org.iaea.EnergySource;

public class UraniumBasedReactor extends EnergySource {
  public void bringOnline() {
    EnergySource other = new EnergySource();
    this.efficiency = 0;
    other.efficiency = 12; // Compile Error!
  }
}
{% endhighlight %}

Notice that the only difference is the subclass package in the second variation leading to a compile error.  The IS-A `EnergySource` relationship still holds for our subclass, but we can't see protected members of other instances any more.  In this case, let's check the Java Language Spec:

<b>6.6.2 Details on protected Access</b>

<i>A protected member or constructor of an object may be accessed from outside the package in which it is declared only by code that is responsible for the implementation of that object.</i>

This seems confusing at first.  Some of the other sections of the spec seem to imply that both of these examples are valid:

<b>6.6.2.1 Access to a protected Member</b>

<i>Let C be the class in which a protected member m is declared. Access is permitted only within the body of a subclass S of C. In addition, if Id denotes an instance field or instance method, then:</i>
<ul>
	<li><i>If the access is by a qualified name Q.Id, where Q is an ExpressionName, then the access is permitted if and only if the type of the expression Q is S or a subclass of S.</i></li>
	<li><i>If the access is by a field access expression E.Id, where E is a Primary expression, or by a method invocation expression E.Id(. . .), where E is a Primary expression, then the access is permitted if and only if the type of E is S or a subclass of S.</i></li>
</ul>

Certainly, one could argue that the second clause of 6.6.2.1 holds true for our case as it only stipulates types, not instances.  However, the key here is figuring out what <i>code that is responsible for the implementation of that object (6.6.2) </i>means.  Well evidently, it means that since `other` is not the same instance as `this` that we don't have access to `other`'s protected members <i>when we're not in the same package</i>.  Other examples in the JLS seem to support this conclusion as well.  However, if you were to ask me to rank the access levels in order of most restrictive to least, I would say:

<ul>
<li>private</li>
<li>default</li>
<li>protected</li>
<li>public</li></ul>

And generally, <a href="http://download.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html">Oracle</a> would agree. But this seems to be a corner case where we see that protected actually carries an interesting visibility restriction scenario that none of the other access levels could create.