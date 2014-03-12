---
layout: post
status: publish
published: true
title: Plesk - Retrieve FTP & Mail Details
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 196
wordpress_url: http://www.mattclements.co.uk/?p=196
date: 2012-02-16 06:36:33.000000000 +00:00
---
I am often tasked with checking something out over FTP, or retrieving a customers mail details. The shared hosting servers I use all use Plesk, which makes it fairly easy!

<strong>Change to the Plesk Database</strong>
<pre class="language-sql"><code>USE psa;</code></pre>
&nbsp;

<strong>Check FTP Details</strong>
<pre class="language-sql"><code>SELECT REPLACE(sys_users.home,'/var/www/vhosts/','') AS domain,
sys_users.login,accounts.password, sys_users.home AS home_directory
FROM sys_users
LEFT JOIN accounts on sys_users.account_id=accounts.id
ORDER BY sys_users.home ASC;</code></pre>

<strong>Check Mail Details</strong>
<pre class="language-sql"><code>SELECT mail.mail_name, accounts.password, domains.name
FROM domains
LEFT JOIN mail ON domains.id = mail.dom_id
LEFT JOIN accounts ON mail.account_id = accounts.id;</code></pre>
The mail details query will even show NULL for the mail_name &amp; password if they don't have any email accounts on their domain. Useful for checking when you are told that a mailbox exists that turns out to be on a different server!!!
