---
layout: post
title: Sagepay Updated SSL causing issues with PHP
date: '2021-03-25 12:50:00'
---

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Hi,<br>Our development teams have updated our security certificates in our Live environment. <br>You can access our site (<a href="https://t.co/5lQwoXMCGP">https://t.co/5lQwoXMCGP</a>) in a browser and pull the latest root certificate down for this.<br><br>Apologies for any disruption this may have caused.</p>&mdash; Opayo Support (@OpayoSupport) <a href="https://twitter.com/OpayoSupport/status/1374747630555828225?ref_src=twsrc%5Etfw">March 24, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

We had a Craft/Craft Commerce store that started throwing a warning yesterday:

```
ERROR! [CURL] 60: SSL CERTIFICATE PROBLEM: SELF SIGNED CERTIFICATE IN CERTIFICATE CHAIN [URL] HTTPS://LIVE.SAGEPAY.COM/GATEWAY/SERVICE/VSPSERVER-REGISTER.VSP
```

It turned out after much panicing that Sagepay had updated their SSL, and our implementation of Sagepay (using Omnipay/Guzzle) had a cached root certificate file which was failing validation.

We simply searched for the `cacert.pem` files (ours were located `vendor/guzzle/guzzle/src/Guzzle/Http/Resources/cacert.pem`) and removed them. When browsing again, this re-cached the root certificates which resolved.