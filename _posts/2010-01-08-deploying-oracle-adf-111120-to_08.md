---
layout: post
title: Deploying Oracle ADF 11.1.1.2.0 to a standalone WebLogic 10.3.2 instance
date: '2010-01-08T11:32:00.002-06:00'
author: Jonathan Cone
tags:
- ADF
- Commons Logging
- Java EE
- Oracle
- WebLogic
modified_time: '2010-09-15T19:24:29.337-05:00'
blogger_id: tag:blogger.com,1999:blog-850212609563989685.post-5228318149130704682
blogger_orig_url: http://www.machineversus.me/2010/01/deploying-oracle-adf-111120-to_08.html
---
<h3>Introduction</h3>
Oracle provides documentation for performing this deployment but it assumes you're deploying the application to an EAR file constructed using Oracle JDeveloper.  Since that never happens in the real world (because we all use things like build management systems which swap property files in and out and automatically setup datasources), here are some things that might be good to know, especially if you're doing this as part of an upgrade from a previous version (like 11.1.1.1.0):

<h3>Prerequisites</h3>
You'll need to install the ADF runtime to the WLS 10.3.2 instance you're deploying to.  There are literally dozens of documents and blogs describing those so I won't go into detail here.

<h3>EAR Structure</h3>
Generally, when we talk about ADF we're discussing a web application module and maybe a couple of EJBs.  For a web application, you'll want to make sure the following lines are contained in your __weblogic.xml__ file:

{% highlight xml %}
<library-ref>
	<library-name>adf.oracle.domain.webapp</library-name>
</library-ref>
<library-ref>
	<library-name>jstl</library-name>
	<specification-version>1.2</specification-version>
</library-ref>
<library-ref>
	<library-name>jsf</library-name>
	<specification-version>1.2</specification-version>
</library-ref>
{% endhighlight %}

Also, be sure to include the following in your __weblogic-application.xml__ file:

{% highlight xml %}
<library-ref>
	<library-name>adf.oracle.domain</library-name>
</library-ref>
{% endhighlight %}

These two stanzas will allow your application access to the shared libraries you installed as part of the prerequisite step.  In doing so, you shouldn't need to package any of the ADF JAR files directly in your EAR.  If you do so, WebLogic will complain and your log file will start to look very busy.

<h3>Startup Classpath</h3>
You'll also need to make sure your managed server's startup classpath looks something like this:
<pre>
$WLS/jdk160_14_R27.6.5-32/lib/tools.jar
:$WLS/utils/config/10.3/config-launch.jar
:$WLS/wlserver_10.3/server/lib/weblogic_sp.jar
:$WLS/wlserver_10.3/server/lib/weblogic.jar
:$WLS/modules/features/weblogic.server.modules_10.3.2.0.jar
:$WLS/wlserver_10.3/server/lib/webservices.jar
:$WLS/wlserver_10.3/server/lib/wlcommons-logging.jar
:$WLS/modules/org.apache.ant_1.7.0/lib/ant-all.jar
:$WLS/modules/net.sf.antcontrib_1.0.0.0_1-0b2/lib/ant-contrib.jar
:$WLS/oracle_common/modules/oracle.jrf_11.1.1/jrf.jar
:$WLS/wlserver_10.3/server/lib/xqrl.jar
</pre>

Where `$WLS` is your WebLogic installation.  The big one here is `jrf.jar`.  This is a JAR file with zero classes in it.  Basically it just contains a JAR manifest (`MANIFEST.MF`) that has `Class-Path` entries for all the ADF dependencies.

<h3>Apache Commons Logging</h3>
Unfortunately, `jrf.jar` introduces a dependency on commons logging 1.0.4 which contains the memory leak problem and lacks the diagnostic tools contained in JCL 1.1.1.  The end result is that this will cause problems if you try to package commons logging with your enterprise application.

<h3>References</h3>

<a href="http://download.oracle.com/docs/cd/E12840_01/wls/docs103/logging/config_logs.html#wp1014610">WebLogic 10.3.2 Documentation</a>

<a href="http://commons.apache.org/logging/commons-logging-1.1/troubleshooting.html">Commons Logging Documentation</a>

<a href="http://www.oracle.com/technology/documentation/jdev.html">Oracle ADF 11.1.1.2.0 Documentation</a>

As an aside, I've found the best thing to do is open Oracle JDeveloper 11.1.1.2.0 and use the help with the IDE to link out to other documentation.  Often times it can be impossible to Google for Oracle  software documentation because any results don't contain enough information about which version they refer to.  Using the embedded help to click out seems to be the best way to get accurate information.