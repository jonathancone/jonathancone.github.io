---
layout: post
title: BlackBerry Development Gotchas
date: '2009-11-16T21:05:00.001-06:00'
author: Jonathan Cone
tags:
- BlackBerry Development
modified_time: '2010-09-15T13:52:37.309-05:00'
blogger_id: tag:blogger.com,1999:blog-850212609563989685.post-365545301206636311
blogger_orig_url: http://www.machineversus.me/2009/11/blackberry-development-gotchas_16.html
---

If you're developing for the Blackberry, you've likely tried the <a href="http://na.blackberry.com/eng/developers/javaappdev/javaeclipseplug.jsp">BlackBerry JDE Plugin for Eclipse</a>.  However, I've noticed a couple of things that you might want to know before you get going:

* In the 4.5 component pack, calls to getName() from the static .class instance (i.e. Object.class.getName()) can cause weird preverification errors that don't manifest themselves outside the IDE.

* If you have a project that uses the maven layout, you'll want to be aware that the BlackBerry tools are going to rewrite your sourcecode to the ${basedir}/src directory for preverification.  This means that you'll see multiple copies of the same Java file in a single project.

* Its basically impossible to work with a 3rd party JAR file inside the IDE.  If you can, get the source for the JAR and import it into your project to be built alongside your application code.

* Preprocessor support exists, but its kind of weird.  If you install the 4.7 component pack, you'll be able to build using the preprocessor and the 4.7 APIs on a 4.5 device with no errors in the 'Problems' window (assuming your preprocessor defines actually exclude 4.7 code).  However, you will still see red errors in the code editor window for APIs that don't exist in 4.5 (after you switch back to the 4.5 component pack)

* Since JUnit uses reflection, having your unit tests and your BlackBerry code in the same project is basically impossible.  Its easiest to create a separate project for this.

* The JDE spits out all the build time artifacts to the top-level project directory.  This can be annoying as you'll find yourself continually adding things to .cvsignore.
