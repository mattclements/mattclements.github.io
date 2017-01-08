---
layout: post
title: Changing PHP Version on Plesk
date: '2014-10-27 12:21:56'
---

Recently, I had to setup a server which uses Plesk, and PHP 5.4 to also run PHP 5.3 side-by-side for a certain domain. Ramblings of my efforts are below.

Server currently runs on 5.4.33, however we have configured an additional version of PHP 5.3 for a site which requires a PHP 5.3 Depreciated Function. This is installed in `/usr/local/php53-cgi/`. We can switch Domains over to this version manually if requires as follows. The guide below uses example.com as the domain.

First we create a PHP wrapper.
{% highlight bash %}
cd /var/www/vhosts/example.com/cgi-bin
mkdir .cgi_wrapper
cd .cgi_wrapper
touch .phpwrapper
{% endhighlight %}

Edit the `.phpwrapper` file just created, and populate it with the following:
{% highlight bash %}
#!/bin/sh
export PHPRC=/var/www/vhosts/example.com/etc/
export PHP_FCGI_CHILDREN=4
export PHP_FCGI_MAX_REQUESTS=1000
exec /usr/local/php53-cgi/bin/php-cgi
{% endhighlight %}

Set the necessary permissions and privileges for the created files and directories (Run `ls -l /var/www/vhosts/example.com` to find the domains username - listed in the below example as example):
{% highlight bash %}
chgrp psaserv /var/www/vhosts/example.com/cgi-bin
cd /var/www/vhosts/example.com/cgi-bin
chmod 101 .cgi_wrapper
chmod 500 .cgi_wrapper/.phpwrapper
chown example:psacln .cgi_wrapper -R
chattr -R +i .cgi_wrapper
{% endhighlight %}

Make Apache aware of our new PHP wrapper. Plesk offers an option to change the httpd setup per vhost (it could be either domain or subdomain). We will use this option to tell Apache that, as per our example, example.com needs to use our new PHP wrapper instead of the one provided by Plesk:
{% highlight bash %}
cd /var/www/vhosts/example.com/conf
{% endhighlight %}

Create a vhost.conf file with the following content:

{% highlight apache %}
<Directory /var/www/vhosts/example.com/httpdocs>
RemoveHandler fcgid-script
<IfModule mod_fcgid.c>
    AddHandler fcgid-script .php
    <Files ~ (\.php)>
        SetHandler fcgid-script
        FCGIWrapper /var/www/vhosts/example.com/cgi-bin/.cgi_wrapper/.phpwrapper .php
        Options +ExecCGI
        allow from all
    </Files>
</IfModule>
</Directory>
{% endhighlight %}

So far, we have told Apache not to use the default Plesk PHP wrapper (`RemoveHandler fcgid-script`) and instead, we created a new handler for PHP files. When executing PHP files on the "example.com" domain, Apache will call the new PHP wrapper and use the version that was installed.

Reconfigure the "example.com" domain:
{% highlight bash %}
/usr/local/psa/admin/sbin/httpdmng --reconfigure-domain example.com
{% endhighlight %}

Test that the Apache Configuration is ok, then Restart Apache:
{% highlight bash %}
service httpd configtest
service httpd restart
{% endhighlight %}