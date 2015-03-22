---
layout: post
title: Setting up apache with PAM on Ubuntu
tags: [Ubuntu]
description: 
---

Logging in on a site using apache can be done in a lot of different ways e.g. .htpasswd or mysql. All of them have one thing in common: you get yet another password. What I wanted to accomplish is to be able to login using my normal Ubuntu username/password. This makes it a lot easier to change my password once in a while.

The first step is to install the required applications:

{% highlight bash %}
sudo apt-get install libapache2-mod-authz-unixgroup pwauth  
{% endhighlight %}

Enable the apache module:

{% highlight bash %}
sudo a2enmod authnz_external  
{% endhighlight %}

Edit the appropriate apache site in /etc/apache2/sites-available, make sure the site is only available over SSL otherwise you password will travel over the Internet unencrypted!

{% highlight bash %}
    AddExternalAuth pwauth /usr/sbin/pwauth  
    SetExternalAuthMethod pwauth pipe  
      
    <location /sickbeard/>  
        order deny,allow  
        deny from all  
        allow from all  
      
        ProxyPass http://localhost:8081/sickbeard/  
        ProxyPassReverse http://localhost:8081/sickbeard/  
      
        AuthType Basic  
        AuthName "Boaz' Sick Beard"  
        AuthBasicProvider external  
        AuthExternal pwauth  
        Require valid-user  
    </location>  
{% endhighlight %}

Finally restart apache and you're ready to go!

{% highlight bash %}
sudo /etc/init.d/apache2 restart  
{% endhighlight %}
