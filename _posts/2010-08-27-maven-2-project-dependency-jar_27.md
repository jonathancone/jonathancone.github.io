---
layout: post
title: Maven 2 Project Dependency JAR Corruption
date: '2010-08-27T20:43:00.002-05:00'
author: Jonathan Cone
tags:
- Java
- Jetty
- Maven
modified_time: '2010-09-15T19:20:20.829-05:00'
blogger_id: tag:blogger.com,1999:blog-850212609563989685.post-5029241289953907058
blogger_orig_url: http://www.machineversus.me/2010/08/maven-2-project-dependency-jar_27.html
---

Earlier today, I was experiencing a weird deployment problem using the <a href="http://docs.codehaus.org/display/JETTY/Maven+Jetty+Plugin">Jetty plugin</a> with <a href="http://maven.apache.org/">Maven</a>.  Some of the dependency JARs appeared to be corrupted.  Because of this, my application was throwing the following exceptions.  If you see this issue, try deleting the archives and allow maven to re-download the dependencies.

{% highlight java %}
2010-08-27 19:29:12.240:WARN::Failed to read file: C:\Users\jcone\.m2\repository\antlr\antlr\2.7.6\antlr-2.7.6.jar
java.util.zip.ZipException: error in opening zip file
at java.util.zip.ZipFile.open(Native Method)
...
2010-08-27 19:29:12.245:WARN::Failed to read file: C:\Users\jcone\.m2\repository\dom4j\dom4j\1.6.1\dom4j-1.6.1.jar
java.util.zip.ZipException: error in opening zip file
at java.util.zip.ZipFile.open(Native Method)
...
2010-08-27 19:29:12.248:WARN::Failed to read file: C:\Users\jcone\.m2\repository\javax\transaction\jta\1.1\jta-1.1.jar
java.util.zip.ZipException: error in opening zip file
at java.util.zip.ZipFile.open(Native Method)
...
{% endhighlight %}