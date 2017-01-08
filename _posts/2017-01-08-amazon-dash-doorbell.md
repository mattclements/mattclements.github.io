---
layout: post
title: Amazon Dash Doorbell
date: '2017-01-08 09:01:11'
---

So, my weekend hack project came to me in the form of a tweet. Thanks to [Christian Hambly](http://freelancegeek.co.uk) for the idea:

<blockquote class="twitter-tweet" data-cards="hidden" data-lang="en-gb"><p lang="en" dir="ltr">My weekend project set, silent doorbell! : <a href="https://t.co/rbvz34P4As">https://t.co/rbvz34P4As</a><br><br>Cc <a href="https://twitter.com/mattclementsuk">@mattclementsuk</a> <a href="https://twitter.com/bseymour">@bseymour</a></p>&mdash; Christian Hambly (@ChristianHambly) <a href="https://twitter.com/ChristianHambly/status/817626090919301120">7 January 2017</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Myself and my eldest set about installing Node, developing the required code [mainly based upon the original article](https://github.com/initialstate/silent-doorbell/wiki), and working with [Pushover](https://pushover.net) which I have already used previously for notifications, the final code looks as follows:

{% highlight js %}
function send_pushover_notification() {

  var push = require( 'pushover-notifications' );

  var p = new push( {
      user: "xxxx",
      token: "xxxx"
  });

  var msg = {
      message: "Somebody is at the door!",
      title: "Doorbell",
      sound: 'pushover',
      priority: 1
  };

  p.send( msg, function( err, result ) {
      if ( err ) {
          return false;
      }

      if(result.status == 1) {
        return true;
      }
  });

  return false;
}

var dash_button = require('node-dash-button'),
    dash = dash_button('xx:xx:xx:xx:xx:xx'),
    exec = require('child_process').exec;

dash.on('detected', function() {
    console.log('Button pushed!');

    send_pushover_notification();
});


console.log('Ready');
{% endhighlight %}

We have a server fitted in the loft which I installed Node onto, then [installed PM2](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-14-04#install-pm2) in order to run the application as a service.

The eldest installed [Pushover](https://pushover.net) on his iPod Touch which means he knows exactly when the doorbell rings!

The final result:

![Doorbell](/assets/img/2017/01/doorbell.jpg)