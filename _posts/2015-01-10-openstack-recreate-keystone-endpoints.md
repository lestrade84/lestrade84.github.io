---
layout: post
title: How to re-create keystone endpoints
modified: 2015-03-10
author: carlos_spitzer
tags: [openstack, keystone, endpoints, endpoint]
comments: true
image:
  feature: openstack-banner.jpg
---

Delete current endpoints (please obtain the IDs with ``keystone endpoint-list`` and put keystone-endpoint id the last one):

{% highlight bash %}
$ keystone endpoint-delete 1b71ebb5fbf44e38b57cb4e83af70051 
$ keystone endpoint-delete 1bd6683a83dc4a248dba13067b70b22c 
$ keystone endpoint-delete 23c8fdeba551495aab2a96ec60eaf404 
$ keystone endpoint-delete 4f520356d2624fa69e16b7e7dcfeaf25 
$ keystone endpoint-delete 9c7b91a3c55c4ad4a58ee42e8074ad9c 
$ keystone endpoint-delete a0957658c89e486ea03a770924dac856 
$ keystone endpoint-delete ce37d3481fc3471ba8631103a8eeeb47 
$ keystone endpoint-delete 51a969756bfa4ac388036c255efb685e 
{% endhighlight %}

Define AUTH variables: 

{% highlight bash %}
$ export OS_SERVICE_TOKEN="23ccdfcfd7cbcfebf4f1"
$ export OS_USERNAME=admin  
$ export OS_TENANT_NAME=admin 
$ export OS_PASSWORD=<my_pass>
$ export OS_SERVICE_ENDPOINT=http://192.168.100.104:35357/v2.0/ 
{% endhighlight %}
 
Define VIP variables:

{% highlight bash %}
$ export REGION_NAME="MY_REGION" 
$ export PUBLIC_DOMAIN="mydomain.com" 
$ export OS_CTL_NET="192.168.100" 
$ export VIP_MYSQL="${OS_CTL_NET}.102" 
$ export VIP_KEYSTONE="${OS_CTL_NET}.104" 
$ export VIP_GLANCE="${OS_CTL_NET}.105" 
$ export VIP_CINDER="${OS_CTL_NET}.106" 
$ export VIP_NEUTRON="${OS_CTL_NET}.108" 
$ export VIP_NOVA="${OS_CTL_NET}.109" 
$ export VIP_HEAT="${OS_CTL_NET}.110" 
$ export VIP_CEILOMETER="${OS_CTL_NET}.112" 
{% endhighlight %}
 

And finally, create endpoints:

{% highlight bash %}
$ keystone endpoint-create --service keystone --region $REGION_NAME --publicurl "http://keystone.${PUBLIC_DOMAIN}:5000/v2.0" --adminurl "http://keystone.${PUBLIC_DOMAIN}:35357/v2.0" --internalurl "http://$VIP_KEYSTONE:5000/v2.0" 
$ keystone endpoint-create --service glance --region $REGION_NAME --publicurl "http://glance.${PUBLIC_DOMAIN}:9292" --adminurl "http://glance.${PUBLIC_DOMAIN}:9292" --internalurl "http://$VIP_GLANCE:9292" 
$ keystone endpoint-create --service cinder --region $REGION_NAME --publicurl "http://cinder.${PUBLIC_DOMAIN}:8776/v1/%(tenant_id)s" --adminurl "http://cinder.${PUBLIC_DOMAIN}:8776/v1/%(tenant_id)s" --internalurl "http://$VIP_CINDER:8776/v1/%(tenant_id)s" 
$ keystone endpoint-create --service neutron --region $REGION_NAME --publicurl "http://neutron.${PUBLIC_DOMAIN}:9696" --adminurl "http://neutron.${PUBLIC_DOMAIN}:9696" --internalurl "http://$VIP_NEUTRON:9696" 
$ keystone endpoint-create  --service compute --region $REGION_NAME --publicurl "http://nova.${PUBLIC_DOMAIN}:8774/v2/%(tenant_id)s" --adminurl "http://nova.${PUBLIC_DOMAIN}:8774/v2/%(tenant_id)s" --internalurl "http://$VIP_NOVA:8774/v2/%(tenant_id)s" 
$ keystone endpoint-create --service heat --region $REGION_NAME --publicurl "http://heat.${PUBLIC_DOMAIN}:8004/v1/%(tenant_id)s" --adminurl "http://heat.${PUBLIC_DOMAIN}:8004/v1/%(tenant_id)s" --internalurl "http://$VIP_HEAT:8004/v1/%(tenant_id)s" 
$ keystone endpoint-create --service heat-cfn --region $REGION_NAME --publicurl "http://heat.${PUBLIC_DOMAIN}:8000/v1" --adminurl "http://heat.${PUBLIC_DOMAIN}:8000/v1" --internalurl "http://$VIP_HEAT:8000/v1" 
$ keystone endpoint-create --service ceilometer --region $REGION_NAME --publicurl "http://ceilometer.${PUBLIC_DOMAIN}:8777" --adminurl "http://ceilometer.${PUBLIC_DOMAIN}:8777" --internalurl "http://$VIP_CEILOMETER:8777" 
{% endhighlight %}

