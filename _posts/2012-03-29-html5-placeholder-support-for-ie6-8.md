---
layout: post
status: publish
published: true
title: HTML5 Placeholder Support for IE6-8
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 233
wordpress_url: http://www.mattclements.co.uk/?p=233
date: 2012-03-29 13:59:59.000000000 +01:00
---
It winds me up when creating awesome sites that use the Placeholder HTML attribute, as then I have to use a Label to show the name of the field instead to make this accessible for IE6-8.

As an alternative to dropping support for IE6-8 (which I just can't do!) I created the following, which uses jQuery to script, and Moderniser to feature detect the support for the placeholder attribute:

<script src="https://gist.github.com/2224869.js"></script>
