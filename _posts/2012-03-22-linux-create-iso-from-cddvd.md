---
layout: post
status: publish
published: true
title: Linux - Create ISO from CD/DVD
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 231
wordpress_url: http://www.mattclements.co.uk/?p=231
date: 2012-03-22 16:20:30.000000000 +00:00
---
I needed to setup a VM, and VirtualBox was complaining that it was sticking when running direct from the disk.

I therefore found the following:
<pre class="language-bash"><code>cd /disk1
dd if=/dev/cdrom of=WindowsXP.iso</code></pre>

Where /disk1 is where you want the ISO to be created, /dev/cdrom is your CD Drive, and WindowsXP.iso is the ISO name you require
