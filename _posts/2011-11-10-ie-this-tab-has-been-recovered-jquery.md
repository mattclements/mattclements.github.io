---
layout: post
status: publish
published: true
title: 'IE: This tab has been recovered - jQuery'
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 143
wordpress_url: http://www.mattclements.co.uk/?p=143
date: 2011-11-10 11:30:32.000000000 +00:00
---
I have recently put a new dynamic edit screen live at work for our clients; and have been having a lot of phone calls with people reporting "issues" and "being kicked out"; this baffled me - and I put it down to IE being very silly; or a session being timed out and the client being requested to log-in again.

However this morning I got a phone call, and email from our call center with some more detailed information, I attempted the same edit and received the following:
<blockquote><em>This tab has been recovered</em></blockquote>
&nbsp;

The page had completed the POST from the form, and the edit had been made, but on reloading the confirmation the tab crashed and the page was recovered, loosing the POST variables, and so showing an error the user.

I had a quick search and found the usual:
<blockquote>Delete browsing history, cookies etc etc within IE</blockquote>
Then I found the following link: <a title="http://robspangler.com/blog/random-ie8-crashes-this-tab-has-been-recovered/" href="http://robspangler.com/blog/random-ie8-crashes-this-tab-has-been-recovered/">http://robspangler.com/blog/random-ie8-crashes-this-tab-has-been-recovered/</a> on a blog from Rob Spangler

Low and behold, I was using jQuery 1.6.2 which has a (pretty major) bug within it, causing the tab to crash with background properties

(Bug Report: <a title="http://bugs.jquery.com/ticket/9823" href="http://bugs.jquery.com/ticket/9823" target="_blank">http://bugs.jquery.com/ticket/9823</a>)

Solution: Update to jQuery latest (<a href="http://code.jquery.com/jquery-1.7.min.js" target="_blank">1.7</a> at the time of publishing
