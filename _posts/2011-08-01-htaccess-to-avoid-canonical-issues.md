---
layout: post
status: publish
published: true
title: ".htaccess to avoid Canonical Issues"
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 113
wordpress_url: http://www.mattclements.co.uk/?p=113
date: 2011-08-01 21:21:25.000000000 +01:00
---
I have had canonical problems previously where Google detects a site as multiple sites, and therefore you don't get the full impact of the site (as Google see's this as duplicate content on 2 sites). I have seen some pretty shoddy versions before, but this is by far the best

<pre class="language-bash"><code>
Options +FollowSymLinks
RewriteEngine On
RewriteBase /
RewriteCond %{HTTP_HOST} !^www\.siteurl\.co\.uk$
RewriteRule ^(.*)$ http://www.siteurl.co.uk/$1 [R=301,L]
</code></pre>
