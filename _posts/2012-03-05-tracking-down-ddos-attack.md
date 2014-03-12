---
layout: post
status: publish
published: true
title: Tracking Down DDOS Attack
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 215
wordpress_url: http://www.mattclements.co.uk/?p=215
date: 2012-03-05 16:55:28.000000000 +00:00
---
Over the years I have spent a fair amount of time tracking down scripts that are performing DDOS attacks on other machines. The latest this weekend were a number of Perl scripts that had been uploaded, and were being used to create a Cron Job to run itself every minute to perform DDOS attacks.

I found the scripts using the following scripts, and then removed all cron-jobs, perl scripts, and changed the FTP details to stop this happening in future.

<pre class="language-bash"><code>cd /var/www/vhosts/
find */cgi-bin/* -name '*.cgi'
find */cgi-bin/* -name '*.pl'
for user in $(cut -f1 -d: /etc/passwd); do echo $user; crontab -u $user -l; done</code></pre>
