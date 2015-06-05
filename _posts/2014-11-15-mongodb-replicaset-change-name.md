---
layout: post
title: How to change hostnames in a MongoDB replica set
modified: 2014-11-15
author: carlos_spitzer
tags: [mongodb, replicaset, hostname]
comments: true
image:
  feature: mongodb-banner.jpg
---

For most replica sets, the hostnames in the host field never change. However, if organizational needs change, you might need to migrate some or all host names.
If this is your case, take a look at the following procedure:

From mongoDB replica set primary node (in our case, node 1), execute the following commands:

{% highlight bash %}
$ mongo 
rs.stepDown() 
{% endhighlight %}
 
Then, go to the second node (that maybe has been chosen as the primary node):

{% highlight bash %}
$ mongo 
cfg = rs.conf()
cfg.members[0].host = "<new_node_name>:27017"
rs.reconfig(cfg) 
{% endhighlight %}
 
Before go out mongoDB interpreter, check if everything is OK now:

{% highlight bash %}
rs.status()
{% endhighlight %}


## Deep dive

If you want to go further, I suggest you to check this <a href="http://docs.mongodb.org/manual/tutorial/change-hostnames-in-a-replica-set/" target="_blank">post.</a>
{: .notice}

