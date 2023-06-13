---
title: 'Doing the work elsewhere - Sidebar - running the worker'
date: Tue, 23 Jun 2009 21:19:33 +0000
draft: false
tags: ['beanstalkd', 'php', 'queues', 'scaling', 'workers']
---

I'm taking a slight diversion now, to show you how the main worker processor runs. There are two parts to it - the actual worker, written in PHP, and the script that keeps running it.

For testing with return from the worker, we'll just return a random number. In order to avoid returning a normally used exit value, I've picked a few numbers for our controls, up around the 100 range. By default a 'die()' or 'exit' will return a '0', so we can't use that to act on - though we will use it as a fall-back as a generic error. Ideally, we won't get one, instead we want the code in all the workers to just run as planned, and then have the worker execute a planned restart - and we will just immediately restart. We may also choose to have the worker process specifically stop - and so we'll have an exit code for that. If there are any codes we don't understand, we'll slow the system down with a 'sleep()' to avoid running away with the process.

\[gist id=1386210\]

The actual script that is run from the command line is a pretty simple BASH script - all it's got to do is to loop, until it gets a particular set of exit values back.

\[gist id=1386212\] So, if it's an exit value we know, we either 1/ pause, then restart 2/ immediately restart 3/ exit the loop. If its any other value, we pause, and restart.

The bash command 'exec $0 $@' will re-run the current script ($0) with the original arguments ($@) - but with the 'exec', replaces the current process with a specified command. Normally, when the shell encounters a command, it forks off a child process to actually execute the command. Using the exec builtin, the shell does not fork, and the command exec'ed replaces the shell.

Save both the PHP and bash script, and then you can start the script with 'sh runBeanstalkd-worker.sh', run it a few times to see a lot of (deliberate) errors that cause the bash script to pause before restart, immediately restart and finally exit.

With this bash script in place, we can now run the script as many times as we need - and it will keep running, until we specifically tell it to exit. As usefully, we can exit the php worker, and have it execute a planned restart - which will clear any overheads that the script may have picked up with memory or resource allocation.

Next time, we'll put some simple tasks into the queue.
---
### Comments:
#### 
[cole]( "cabennett85@gmail.com") - <time datetime="2012-07-24 23:19:16">Jul 2, 2012</time>

for the shell script i found i need to put an sh in the exec for it to rerun the script: exec sh $0 $@; is this something missing in the above code, or a config issue on my end that will break some key functionality here? sorry, bit of a n00b shell scripter
<hr />
#### 
[alister](http://abulman.co.uk/ "abulman@gmail.com") - <time datetime="2012-07-24 23:46:43">Jul 2, 2012</time>

Hi! Questions are good, I'm happy to help out. Putting a '`sh`' in there is equally valid, but I think that's just the way you are calling it that you need it. I'm going to guess that the current directory, where the script you are running is placed, is also on the executable path (run `set | grep -a ^PATH` and look for a lone `.`, possibly at the start or end of the PATH). That's actually not recommended to do - see the below URLs, and particularly the notes about security. When I setup to run the workers, I'll either use the 'sh' on the initial call and a full path to the command, or `./command-name`. Either way, with a path to the code (and not just a bare script name), it knows how to be able to call itself, without the extra 'sh'. Since it's already running under a shell script, there's no need to add another - and besides, without it, it's a little easier to fin under its own name in the process list :-) The difference is quite easy to see when you run this script: `#!/bin/sh echo "i'm here" echo "$0 $@" sleep 2 exec $0 $@` First, we'll run it with ./test.sh (or from another directory, with a full path - eg `do/stuff/scripts/test.sh`): `# ./test.sh param1 p2 p3 i'm here ./test.sh param1 p2 p3 i'm here ./test.sh param1 p2 p3 i'm here (.... etc)` Now, running it with 'sh': `% sh test.sh param1 p2 p3 i'm here test.sh param1 p2 p3 test.sh: 7: exec: test.sh: not found` The $0 in the script needs to be able to re-run the same command, but it doesn't get the 'sh' call. It needs to be able to run what it does get. Give this a try as well: `watch pstree -Apu` You can also add a process-ID at the end there (those are the numbers in brackets) to narrow down the part of the tree to look at. Alister Here's a couple of peoples thoughts as to why it's not a great idea to have the current directory in the PATH. \* http://technonstop.com/dot-slash-meaning-linux \* http://www.linuxforums.org/forum/miscellaneous/27942-linux-doesnt-automatically-add-current-directory-path.html#post141980
<hr />
#### 
[Maxime]( "max44410@gmail.com") - <time datetime="2012-07-27 00:30:27">Jul 5, 2012</time>

I've follow the instruction and it's working perfectly when I execute the shell script in the console "./worker.sh". However I want this process to be detached from the console so I've added a "&" at the end of the command "./app/worker.sh &". But it's now working anymore, the detach process is ending automatically: \[user@host\]$ ./workers.sh email & \[2\] 6622 \[user@host\]$ \[2\]+ Stopped ./workers.sh email \[user@host\]$ Any thoughts ?
<hr />
#### 
[alister](http://abulman.co.uk/ "abulman@gmail.com") - <time datetime="2012-07-30 22:34:22">Jul 1, 2012</time>

Arranging for the workers to run is one of the harder parts of a major system. I'll write a post sometime with potential options. In the meantime, here's a few ideas: 1/ If you will be keeping an eye on it, you could just run it within 'screen' or the more recent tmux). Both allow processes to keep running while you disconnect. I've had instances of 'screen' running for literally months. 2/ Upstart, or init.d scripts aren't too hard to write, and considering they are probably already running most of the daemons on your server, they are a great choice that are already available. If you need to run more than one worker on the same machine then you would need multiple copies, so it's not so useful if you are going to run more than a few. 3/ If you will be running many copies (and I've done more than 50 identical workers before on the same machine), then supervisord (http://supervisord.org/) has the max number of workers as a simple number in the configuration, as as many as you need, up to the limit can be started.
<hr />
#### 
[maxime]( "max44410@gmail.com") - <time datetime="2012-07-30 23:22:45">Jul 1, 2012</time>

Actually it was because my PHP script was echoing some data and you can't run the shell script without redirecting the output. \[user@host\]$ ./workers.sh email > /dev/null 2> /dev/null &
<hr />
#### 
[Stanley]( "stanleywym@gmail.com") - <time datetime="2012-08-15 03:44:56">Aug 3, 2012</time>

Thanks for the article series. It is very helpful. I have some questions after reading your articles. How should we decide whether to do a planned pause/restart or planned restart? And what are the criteria used to trigger the planned restart and planned complete exit?
<hr />
#### 
[alister](http://abulman.co.uk/ "abulman@gmail.com") - <time datetime="2012-08-23 13:43:14">Aug 4, 2012</time>

That depends very much on what sort of things your workers are doing. You might decide that if a worker has run, but there's nothing for them to do, you'll let the shell script pause for a while before it restarts. Equally, you might also elect to just have the beanstalk client block, waiting for the next job to become available - in that case, you might not have need of any planned pauses in the shell script at all. The only difference between a planned and unplanned restart is that an unplanned restart was probably a problem, and so waiting a little while before you try again stops things from spiraling out of control by running too quickly.
<hr />
#### 
[How feasible is a daemon written in PHP, using ignore_user abort and set_time_limit(0) - PHP Solutions - Developers Q &amp; A](http://stackoverflow.com/questions/1006891/how-feasible-is-a-daemon-written-in-php-using-ignore-user-abort-and-set-time-li "") - <time datetime="2013-08-12 03:39:56">Aug 1, 2013</time>

\[...\] Addition: I’ve blogged on a Bash/PHP (or other language) pairing so that you can very easily loop in the PHP script, then exit to restart immediately, or pause for a while – Doing the work elsewhere — Sidebar running the worker. \[...\]
<hr />
