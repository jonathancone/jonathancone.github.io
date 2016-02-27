---
layout: post
title: Eclipse Juno, Maven, M2E and EGit Compatibility Problem and Solution
date: '2012-08-18T14:10:00.000-05:00'
author: Jonathan Cone
tags:
- m2eclipse
- Git
- Maven
- Eclipse
modified_time: '2012-08-18T18:41:13.688-05:00'
thumbnail: http://4.bp.blogspot.com/-JI57M1TZ8Ec/UC_p5Ag6I2I/AAAAAAAAAII/Rcm870bXTmc/s72-c/Screenshot+from+2012-08-19+13:39:12.png
blogger_id: tag:blogger.com,1999:blog-850212609563989685.post-6417952837343338340
blogger_orig_url: http://www.machineversus.me/2012/08/if-youve-upgraded-to-eclipse-juno.html
---

If you've upgraded to <b>Eclipse Juno</b> recently, you're likely having trouble using the following plugins in harmony with each other:

* EGit
* m2e
* m2eclipse-egit connector 

If you try to install the <b>m2eclipse-egit</b> connector, you'll get an error message similar to the following:

> Operation details
>
> Cannot complete the install because of a conflicting dependency.
>
> Software being installed: Maven SCM Handler for EGit 0.14.0.201110251725

The problem is that the m2e connector available on the update site through the marketplace is not compatible with the latest version of EGit that ships with Eclipse Juno.

To get around this problem, you'll need to install the most up to date m2e-egit connector version manually.

To do this, perform the following steps:

* Click Help
* <b>Install New Software</b>
* Uncheck the box labeled <b>Group items by category</b> (this step is important or you won't see the connector in the table)
* Paste in this URL: 

> http://repository.tesla.io:8081/nexus/content/sites/m2e.extras/m2eclipse-egit/0.14.0/N/0.14.0.20120701011/

* Finish the plugin install wizard and restart the workspace

Its worth noting that this problem exists for the other connectors as well, i.e.the subversion connector. If you're still having trouble, the following thread may help:

<a href="http://dev.eclipse.org/mhonarc/lists/m2e-users/msg02748.html">http://dev.eclipse.org/mhonarc/lists/m2e-users/msg02748.html</a>