---
layout: post
status: publish
published: true
title: Sage ACT! - Last Sync Dates
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 198
wordpress_url: http://www.mattclements.co.uk/?p=198
date: 2012-02-20 11:38:41.000000000 +00:00
---
On Friday I was tasked with putting together a script to email the last dates that our team had synced with Sage ACT! - Admittidly I had put this off until first thing today, mainly due to the issues with PHP not being setup to connect to a MSSQL Database (now resolved, and will become part of another post soon!)

Anyway, the query is as follows:
<pre class="language-sql"><code>SELECT	DBS.DATABASE_NAME as database_name,
   convert(varchar,DBMI.[SYNCDBMAPINFO LAST_COMPLETEDATE],120) as last_sync
FROM	dbo.FNB_SYNCDBMAP() DBM
LEFT OUTER JOIN dbo.FNB_SYNCDBMAPINFO() DBMI
   ON DBM.[SYNCDBMAP SYNCDBMAPID] = DBMI.[SYNCDBMAPINFO SYNCDBMAPID]
LEFT OUTER JOIN dbo.SYNC_DATABASE DBS
   ON DBM.[SYNCDBMAP SUBDBID] = DBS.SYNCDBID
ORDER BY DBMI.[SYNCDBMAPINFO LAST_COMPLETEDATE] desc</code></pre>
This provides a list of the Database Name, and the last date of sync. Using PHP I then concatenate these into a HTML table, which then gets sent via email.
