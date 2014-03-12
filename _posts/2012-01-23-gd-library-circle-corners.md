---
layout: post
status: publish
published: true
title: GD Library Circle Corners
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 176
wordpress_url: http://www.mattclements.co.uk/?p=176
date: 2012-01-23 07:33:22.000000000 +00:00
---
Ok, I have been working on a number of projects over the last 3 months or so which use the GD Library. While I do not confess to being an expert by any means, I'm starting to get into the swing of things!

Last night I was faced with a challenge. Within a table I held the pixel co-ordinates for the X&amp;Y Axis, and the radius of the circle (Green &amp; Red respectively). Now all I needed to do was to find the X&amp;Y co-ordinates for each corner of the circle, so that I can place text correctly

<pre class="language-php"><code>$top_left = array();
$top_left['x'] = $left-(($radius/sin(deg2rad(90)))*sin(deg2rad(45)));
$top_left['y'] = $top-(($radius/sin(deg2rad(90)))*sin(deg2rad(45)));

$top_right = array();
$top_right['x'] = $left+(($radius/sin(deg2rad(90)))*sin(deg2rad(45)));
$top_right['y'] = $top-(($radius/sin(deg2rad(90)))*sin(deg2rad(45)));

$bottom_left = array();
$bottom_left['x'] = $left-(($radius/sin(deg2rad(90)))*sin(deg2rad(45)));
$bottom_left['y'] = $top+(($radius/sin(deg2rad(90)))*sin(deg2rad(45)));

$bottom_right = array();
$bottom_right['x'] = $left+(($radius/sin(deg2rad(90)))*sin(deg2rad(45)));
$bottom_right['y'] = $top+(($radius/sin(deg2rad(90)))*sin(deg2rad(45)));</code></pre>
<strong>Stay tuned for more snippets about the GD Library within PHP!</strong>
