---
layout: post
title: How to change Identity Management (IdM) admin password
modified: 2015-03-10
author: carlos_spitzer
tags: [rhel, ipa, idm, admin, password]
comments: true
image:
  feature: redhat-banner.jpg
---

Sometimes you may need to change the Identity Management (IdM) admin password. Well, the IPA admin password can be updated with the ldappasswd utility.  
Bind with the 'Directory Manager' account in order to perform this task:  

{% highlight bash %}
$ ldappasswd -ZZ -D 'cn=directory manager' -w $(grep internaldb /etc/pki-ca/password.conf | cut -d= -f2) -S uid=admin,cn=users,cn=accounts,dc=idm,dc=lvtc,dc=gsnet,dc=corp -H ldap://localhost
{% endhighlight %}

If you want more info, please check this link: <a href="https://access.redhat.com/solutions/165853" target="_blank">https://access.redhat.com/solutions/165853</a>.
