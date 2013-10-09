---
published: true
layout: post
preview: false
comments: true
---
<section id="initial-setup">
<p>So, I have always wanted to play with Asterisk (the Open Source VOIP Solution) but never had chance to put any of my learning into practice. That is until today.</p>
<p>A company I often pick up work from are looking at SIP Trunks to provide to their clients. Whilst I can’t claim to have done all the work on this – I tweaked things and thought I would share.</p>
<p>We decided on a Debian OS on a Linode 1536 (Debian as it is fairly light-weight, and Linode as their AWESOME!!!).</p>

<pre class="language-bash"><code>apt-key adv --keyserver pgp.mit.edu --recv-keys 175E41DF
echo "deb http://packages.asterisk.org/deb squeeze main" >> /etc/apt/sources.list
echo "deb-src http://packages.asterisk.org/deb squeeze main" >> /etc/apt/sources.list
apt-get update
apt-get upgrade
apt-get install asterisk-1.8 asterisk-dahdi postfix mime-construct libtiff-tools</code></pre>

<p>This will add the Repositories for Asterisk, update everything and install Asterisk, Postfix (for sending mail) mime-construct to easily construct email, and libtiff-tools which adds a tiff2pdf function (along with others) to turn our horrid Fax’s into PDFs</p>
</section>

<section id="sip-config">
<h2>Setting up sip.conf</h2>
<p>Edit /etc/asterisk/sip.conf</p>
<pre class="language-markup"><code>[general]
context=incoming_call ; Default context for incoming calls
allowoverlap=no ; Disable overlap dialing support. (Default is yes)
udpbindaddr=0.0.0.0 ; IP address to bind UDP listen socket to (0.0.0.0 binds to all)
; Optionally add a port number, 192.168.1.1:5062 (default is port 5060)
tcpenable=no ; Enable server for incoming TCP connections (default is no)
tcpbindaddr=0.0.0.0 ; IP address for TCP server to bind to (0.0.0.0 binds to all interfaces)
; Optionally add a port number, 192.168.1.1:5062 (default is port 5060)
 
srvlookup=yes ; Enable DNS SRV lookups on outbound calls
 
;--------------------------- SIP DEBUGGING ---------------------------------------------------
sipdebug = no ; Turn on SIP debugging by default, from
; the moment the channel loads this configuration
;recordhistory=yes ; Record SIP history by default
; (see sip history / sip no history)
;dumphistory=yes ; Dump SIP history at end of SIP dialogue
; SIP history is output to the DEBUG logging channel
register => 441234567890:accountname@siptrunkservice.com/441234567890
[ext-sip-account]
type=friend
context=incoming_call
username=441234567890
defaultuser=441234567890
secret=password
host=siptrunkservice.com
fromdomain=siptrunkservice.com
qualify=yes
insecure=invite
nat=no
canreinvite=no
dtmfmode=rfc2833
 
[500]
context=extension500
type=friend
nat=yes
username=500
secret=password
host=dynamic
allow=ulaw
qualify=yes
callerid="Matt" &lt;500&gt;
canreinvite=no
mailbox=500@default
 
[441]
context=extension501
type=friend
nat=yes
username=501
secret=password
host=dynamic
allow=ulaw
qualify=yes
callerid="Sarah" &lt;501&gt;
canreinvite=no
mailbox=501@default</code></pre>

<p>This file sets up the SIP account and sets up some basic extensions to be used by our phones</p>
</section>

<section id="extensions-config">
<h2>Setting up extensions.conf</h2>
<p>Edit /etc/asterisk/extensions.conf</p>
<pre class="language-markup"><code>
exten => 801,1,Goto(record,s,1)
 
[record]
exten => s,1,Answer
exten => s,2,Read(RECORD,/var/lib/asterisk/sounds/enter4digits,4,3)
exten => s,3,Playback(/var/lib/asterisk/sounds/record-instructions)
exten => s,4,Record(/tmp/recording-${RECORD}.wav,2,20)
;exten => s,2,Record(/var/lib/asterisk/sounds/1toaccept.wav,2,20)
exten => s,5,Wait(2)
exten => s,6,Playback(/tmp/recording-${RECORD})
;exten => s,7,Timeout(10)
exten => s,7,Background(/var/lib/asterisk/sounds/1toaccept)
exten => 1,1,Hangup
exten => 2,1,Goto(s,3)
exten => 3,1,Goto(s,2)
 
[mainmenu]
exten => s,1,Answer
exten => s,2,Background(/var/lib/asterisk/sounds/mainmenu)
exten => 0,1,Dial(SIP/500)
exten => 0,2,Hangup
exten => 1,1,Dial(SIP/501)
exten => 1,2,Hangup
exten => i,1,Goto(s,2)
exten => s,3,Wait(2)
exten => _50X,1,Dial(SIP/${EXTEN},30)
exten => _50X,2,Voicemail(${EXTEN},u)
exten => _50X,103,Voicemail(${EXTEN},b)
exten => _50X,104,Hangup
exten => s,4,Goto(s,2)</code></pre>

<p>The extensions.conf file handles everything from when sip.conf passes over a call, through the switchboard if the main number was called. If a direct line was phoned, it will phone either extension 500, 501, or passes over to the Fax if the call was directed to 01234567892.</p>

<p>The sections are as follows:
<ul>
<li>incoming_call (Passed from sip.conf) when a call is made into the numbers</li>
<li>clements-incoming_call (Passed from incoming_call). This handles the inbound call and where to initially route it. These sections are useful for completely separating 2 companies on 1 Asterisk system</li>
<li>fax (Passed from clements-incoming_call). When the 01234567892 is called, this will call the Macro to receive the fax and store, this then calls a Bash script (below) to send this via email.</li>
<li>macro-faxreceieve (Passed from fax). This section answers the call, and receives the fax to a temporary file.</li>
<li>sip-out – This handles outgoing calls from the system (either by a phone, or an internal process)</li>
<li>extensionXXX – Handles what happens when an extension is called (i.e. Ring, Voicemail, Hang Up) – Other processes can be added here such as calling a Shell Script which looks up the Caller Number in a Database and sends through a notification to the Recipient on their Desktop Screen!</li>
<li>record – Handles recording a new Voicemail</li>
<li>mainmenu – Handles the main menu system to direct the calls</li>
</ul>
</p>
</section>

<section id="fax2email-setup">
<h2>Setting up Fax to Email</h2>
<p>Edit /var/lib/asterisk/scripts/fax</p>
<pre class="language-bash"><code>#!/bin/sh
FAXFILE=$2
RECIPIENT=$1
FROM=$3
tiff2pdf $FAXFILE |
mime-construct --to $RECIPIENT --subject "Fax received" \
--string "Fax has been received from $FROM. The Fax has been attached as a PDF Document.
" \
--header 'From: fax@mattclements.co.uk' --header 'Sender: fax@mattclements.co.uk' \
--attachment Fax.pdf --type application/pdf --file -
rm -f $FAXFILE</code></pre>

<p>This script will use the arguments passed over from the extensions.conf script to detect where the Fax came from (number), the email address that the PDF should be emailed to, and of course our temporary TIFF file.</p>
<ul>
<li>The TIFF file is firstly converted using tiff2pdf and piped over</li>
<li>mime-contruct then constructs an email which it uses the last command to add the attachment</li>
<li>Finally we remove the temporary TIFF file which is no longer required.</li>
</ul>
</section>

<section id="final-steps">
<h2>Final Steps</h2>
<p>Once you have edited these files, run the following commands:</p>
<pre class="language-bash"><code>chown -R asterisk.asterisk /var/lib/asterisk/scripts
chmod +x /var/lib/asterisk/scripts/fax
/etc/init.d/asterisk reload</code></pre>
<p>This will correct the permissions of the script we have just created, and reload asterisk config to test. When testing the best command is:</p>
<pre class="language-bash"><code>tail -f /var/log/asterisk/messages</code></pre>

<p>Thats all for now folks, more adventures coming soon!</p>
</section>