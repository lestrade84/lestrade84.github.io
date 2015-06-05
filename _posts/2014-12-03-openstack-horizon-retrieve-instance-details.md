---
layout: post
title: Fix ``Unable to retrieve details for instance XXX`` Horizon error message
modified: 2015-03-11
author: carlos_spitzer
tags: [openstack, horizon, mariadb, nova, cinder, error]
comments: true
image:
  feature: openstack-banner.jpg
---

Sometimes you may find the following error message when you try to access to an OpenStack instance:

<figure><img align="center" src="{{ site.url }}/images/horizon-retrieve-instance-error.jpg"></figure>

This is a generic error that sometimes is related with the metadata of the affected instance.  
A good practice is to check what is the current status of the instance:

{% highlight bash %}
$ nova show 99fd0970-d348-495b-a9ed-39555e839c9e
{% endhighlight %}

If you find that there is a volume attached more than once:

{% highlight bash %}
...
os-extended-volumes:volumes_attached    | [{"id": "d83a67dd-df9e-4a6e-bbfd-8dbb08483c44"}, {"id": "3abba94a-cb3f-4b25-82e0-1197f9f0c365"}, {"id": "3abba94a-cb3f-4b25-82e0-1197f9f0c365"}, {"id": "3abba94a-cb3f-4b25-82e0-1197f9f0c365"}, {"id": "2594fa29-a144-4c4d-b2ae-b9eed6d6ec7c"}] 
...
{% endhighlight %}

You have found the problem. This is a very odd situation, but if you had messaging problems, maybe you can reach this situation.  
In fact, you can reach another "ugly" situation if you realize that the volume doesn't exist yet:

{% highlight bash %}
$ cinder list --all-tenants | grep 3abba94a-cb3f-4b25-82e0-1197f9f0c365
$
{% endhighlight %}

In order to fix this problem, you have two alternatives:
* The easy way.
* The ninja way.

The affected table is "block_device_mapping" belonging to Nova database.

### The easy way

Check how many entries already have this status:

{% highlight mysql %}
MariaDB [nova]> select id,instance_uuid,volume_id,deleted from nova.block_device_mapping where volume_id in (select id from cinder.volumes where status='deleted') and deleted=0;
{% endhighlight %}

And fix it:

{% highlight mysql %}
MariaDB [nova]> delete from nova.block_device_mapping where volume_id in (select id from cinder.volumes where status='deleted') and deleted=0;
{% endhighlight %}

### The ninja way

With this method, you must update some Nova database entries too.

```
MariaDB [nova]> describe block_device_mapping; 

+-----------------------+--------------+------+-----+---------+----------------+ 
| Field                 | Type         | Null | Key | Default | Extra          | 
+-----------------------+--------------+------+-----+---------+----------------+ 
| created_at            | datetime     | YES  |     | NULL    |                | 
| updated_at            | datetime     | YES  |     | NULL    |                | 
| deleted_at            | datetime     | YES  |     | NULL    |                | 
| id                    | int(11)      | NO   | PRI | NULL    | auto_increment | 
| device_name           | varchar(255) | YES  |     | NULL    |                | 
| delete_on_termination | tinyint(1)   | YES  |     | NULL    |                | 
| snapshot_id           | varchar(36)  | YES  | MUL | NULL    |                | 
| volume_id             | varchar(36)  | YES  | MUL | NULL    |                | 
| volume_size           | int(11)      | YES  |     | NULL    |                | 
| no_device             | tinyint(1)   | YES  |     | NULL    |                | 
| connection_info       | mediumtext   | YES  |     | NULL    |                | 
| instance_uuid         | varchar(36)  | YES  | MUL | NULL    |                | 
| deleted               | int(11)      | YES  |     | NULL    |                | 
| source_type           | varchar(255) | YES  |     | NULL    |                | 
| destination_type      | varchar(255) | YES  |     | NULL    |                | 
| guest_format          | varchar(255) | YES  |     | NULL    |                | 
| device_type           | varchar(255) | YES  |     | NULL    |                | 
| disk_bus              | varchar(255) | YES  |     | NULL    |                | 
| boot_index            | int(11)      | YES  |     | NULL    |                | 
| image_id              | varchar(36)  | YES  |     | NULL    |                | 
+-----------------------+--------------+------+-----+---------+----------------+
```

If we execute the following query, we may find the issue:

```
MariaDB [nova]> select id,volume_id,deleted_at from block_device_mapping where instance_uuid='99fd0970-d348-495b-a9ed-39555e839c9e'; 

+------+--------------------------------------+------------+ 
| id   | volume_id                            | deleted_at | 
+------+--------------------------------------+------------+ 
| 3885 | d83a67dd-df9e-4a6e-bbfd-8dbb08483c44 | NULL       | 
| 3888 | 3abba94a-cb3f-4b25-82e0-1197f9f0c365 | NULL       | 
| 3891 | NULL                                 | NULL       | 
| 3894 | NULL                                 | NULL       | 
| 3897 | NULL                                 | NULL       | 
| 3900 | NULL                                 | NULL       | 
| 3903 | NULL                                 | NULL       | 
| 3906 | NULL                                 | NULL       | 
| 3909 | NULL                                 | NULL       | 
| 3912 | NULL                                 | NULL       | 
| 3933 | 3abba94a-cb3f-4b25-82e0-1197f9f0c365 | NULL       | 
| 3936 | 3abba94a-cb3f-4b25-82e0-1197f9f0c365 | NULL       | 
| 3939 | 2594fa29-a144-4c4d-b2ae-b9eed6d6ec7c | NULL       | 
+------+--------------------------------------+------------+
```

We must change these entries to NULL:

```
MariaDB [nova]> delete from block_device_mapping where id='3888'; 
Query OK, 1 row affected (0.15 sec) 

MariaDB [nova]> delete from block_device_mapping where id='3933'; 
Query OK, 1 row affected (0.00 sec) 

MariaDB [nova]> delete from block_device_mapping where id='3936'; 
Query OK, 1 row affected (0.00 sec) 

MariaDB [nova]> select id,volume_id,deleted_at from block_device_mapping where instance_uuid='99fd0970-d348-495b-a9ed-39555e839c9e'; 

+------+--------------------------------------+------------+ 
| id   | volume_id                            | deleted_at | 
+------+--------------------------------------+------------+ 
| 3885 | d83a67dd-df9e-4a6e-bbfd-8dbb08483c44 | NULL       | 
| 3891 | NULL                                 | NULL       | 
| 3894 | NULL                                 | NULL       | 
| 3897 | NULL                                 | NULL       | 
| 3900 | NULL                                 | NULL       | 
| 3903 | NULL                                 | NULL       | 
| 3906 | NULL                                 | NULL       | 
| 3909 | NULL                                 | NULL       | 
| 3912 | NULL                                 | NULL       | 
| 3939 | 2594fa29-a144-4c4d-b2ae-b9eed6d6ec7c | NULL       | 
+------+--------------------------------------+------------+ 
10 rows in set (0.00 sec)
```

After these changes, we go to OpenStack dashboard again and click on the affected instance. We should access to it now.
