---
layout: post
title: Delete cinder orphan volumes
modified: 2015-03-12
author: carlos_spitzer
tags: [openstack, cinder, volumes, mariadb, orphan, error]
comments: true
image:
  feature: openstack-banner.jpg
---

Sometimes you may find that there are orphan cinder volumes.  
What is an "orphan" volume? It is a volume that is not effectively attached to an instance but the database doesn't reflect the current status of the volume.  

<br>

First, we must clean and update database entries:

{% highlight bash %}
MariaDB [cinder]> select attach_status,status,id,instance_uuid from volumes where status != 'deleted'; 

+---------------+--------+--------------------------------------+--------------------------------------+ 
| attach_status | status | id                                   | instance_uuid                        | 
+---------------+--------+--------------------------------------+--------------------------------------+ 
| attached      | in-use | e35e2564-bb76-4f6c-8f6d-2e9565a1f374 | 3bfded9c-ce1d-4f11-9429-e70c17501112 | 
+---------------+--------+--------------------------------------+--------------------------------------+ 
{% endhighlight %}

If the number of affected volumes is high, one good advise is to use the following query:

{% highlight bash %}
MariaDB [cinder]> select attach_status,status,id,instance_uuid from volumes where status = 'available' and attach_status = 'attached'; 

+---------------+-----------+--------------------------------------+--------------------------------------+ 
| attach_status | status    | id                                   | instance_uuid                        | 
+---------------+-----------+--------------------------------------+--------------------------------------+ 
| attached      | available | f82de4f3-dc90-4572-83cc-bfb4ff3192e9 | 26f623bc-11f1-48b2-8de8-25d6860655b8 | 
+---------------+-----------+--------------------------------------+--------------------------------------+ 
{% endhighlight %}

Then, fix the volume status:

{% highlight bash %}
MariaDB [cinder]> update volumes set attach_status='detached',status='available',instance_uuid=NULL where id='e35e2564-bb76-4f6c-8f6d-2e9565a1f374'; 
{% endhighlight %}

And check if the status have been set in the right way:

{% highlight bash %}
MariaDB [cinder]> select attach_status,status,id,instance_uuid from volumes where status != 'deleted'; 

+---------------+-----------+--------------------------------------+---------------+ 

| attach_status | status    | id                                   | instance_uuid | 

+---------------+-----------+--------------------------------------+---------------+ 
| detached      | available | e35e2564-bb76-4f6c-8f6d-2e9565a1f374 | NULL          | 
+---------------+-----------+--------------------------------------+---------------+
{% endhighlight %}

If so, you can go out mariaDB interpreter and delete the volume from OpenStack Dashboard (cleanest way).

