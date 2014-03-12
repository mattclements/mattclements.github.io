---
layout: post
status: publish
published: true
title: Subversion Merge
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 167
wordpress_url: http://www.mattclements.co.uk/?p=167
date: 2012-01-01 16:14:03.000000000 +00:00
---
I always forget how to do this and am forever having to search my history for various versions of this. On a number of projects I now use <a href="http://www.beanstalkapp.com/" target="_blank">Beanstalk</a> as a Subversion Repository, and work on a branch when developing new features. As with all projects I make changes on the trunk, which are put live and I want to test these changes with the branch I am working on. I therefore need to merge my Subversion trunk with the branch.

I navigate to my branch's working directory:
<pre class="language-bash"><code>cd "/Users/matt/Documents/Sites/Project A"</code></pre>
I then test the merge, then run for real if I dont spot any issues:
<pre class="language-bash"><code>svn merge -r 83:103 https://matt.svn.beanstalkapp.com/my-project/trunk/ . --dry-run
svn merge -r 83:103 https://matt.svn.beanstalkapp.com/my-project/trunk/ .</code></pre>
<p>Note that 83 first commit of the new branch (or if you run this more than once for a branch, the last commit that you merged on), and 103 is the latest commit.</p>
