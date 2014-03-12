---
layout: post
status: publish
published: true
title: Don't just skip past indexes!
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 188
wordpress_url: http://www.mattclements.co.uk/?p=188
date: 2012-02-10 11:57:54.000000000 +00:00
---
It's been a little while since my last blog post. With various new projects that I am working on I have been really busy (and looks like i'm set to stay this busy for a long time yet!)

I have been working on a project over the last few weeks, codename: "Secret Squirrel". Which includes a large volume of data being regularly viewed, updated &amp; reports generated based on the users requests. Unlike projects I have worked on before (which have an attitude of: We'll just load that into a table overnight/hourly etc, and select * from reporting_table_a)); this project has focussed on this information all being real-time. Whilst looking at the capacity for MySQL, compared with other DB's; along with the fact that I will be managing this project ongoing, I have decided to stick with MySQL. Switching the DB is a pretty big risk, especially if theres a problem I need to fix - I need to be working with what I understand.

&nbsp;

So the last few days, I hit a problem such as:
<pre class="language-sql"><code>SELECT e.employee_name, e.employee_identifier, d.employee_phonenumber,
      (SELECT COUNT(r.role_id) FROM roles r WHERE r.employee_id = e.employee_id)
FROM employee e
LEFT JOIN employee_details d ON (d.employee_id = e.employee_id)</code></pre>
Yes I know this isn't a good query, but it will explain the issues I found.

I put an index on:

Employee:
<ul>
	<li>employee_name</li>
	<li>employee_identifier</li>
	<li>employee_id</li>
</ul>
Employee Details:
<ul>
	<li>employee_phonenumber</li>
	<li>employee_id</li>
</ul>
And thought this was enough, but this ran awfully when a lot of data is present. I overlooked the need for the following index on roles:
<ul>
	<li>role_id</li>
	<li>employee_id</li>
</ul>
&nbsp;

The tip is, don't just presume the issue is that you have such a huge amount of data, check the indexes first! I wen't through changing character sets, switching from InnoDB to MyISAM and back again, changing the my.cnf file to allocate more resources.
