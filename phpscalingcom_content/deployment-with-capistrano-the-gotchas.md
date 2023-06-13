---
title: 'Deployment with Capistrano - the Gotchas'
date: Thu, 24 Nov 2011 12:16:59 +0000
draft: false
tags: ['best-practice', 'deployment', 'tools']
---

Capistrano, makes deployment of code easy. If you need to do a number of additional steps as well, then the fact that they can be scripted and run automatically is a huge win.

If you've only got a single machine (or maybe two), then you could certainly write your own quite simple, and entirely workable system - I described something just like this in a previous post: ["SVN checkouts vs exports for live versions"](http://www.phpscaling.com/2008/03/09/svn-checkouts-vs-exports-for-live-versions/). That was written and used before I was deploying to multiple machines however - and had to be run from the command line of the machine itself. It was OK even when I had a couple of machines to deploy - I just opened an SSH to both, and ran the command on them both at the same time. When I attended the [London Devops roundtable on Deployment](https://devblog.timgroup.com/index.php/2010/12/01/devops-comes-to-youdevise) I even advocated for that as a valid deployment mechanism. But, at the same time, as I was saying that (and it's in the video), I was also writing [Chef](http://www.opscode.com/chef/) cookbooks and a Capistrano script to be able to build, and then deploy code to at least four different machines at once.

A number of people have already written about how to setup Capistrano to deploy PHP scripts. I'll not repeat their work, instead I'll just tell you some of the problems you might come across afterwards.

`cap shell` is a wonderful thing, until it bites you
----------------------------------------------------

The Capistrano shell will let you run a simple command or an internal task on one, or as many machines as you want. This can be useful when you are trying things out - and if you are in anyway unsure where a command can be run - you can practice it, just do:

> `cap> with web uptime cap> on host.example.com uptime`

Those two commands just show how long a machine has been up, and the current load average. Easy, and safe, but as they run, they show the list of machines they succeed on.

There are some other useful commands you can try:

> `## show the currently live REVISION file on each machine cap> cat /mnt/html/deployed/current/REVISION ## This file is created as each new/updated checkout is done. ## change your path to the ./current/ path as appropriate`

Since you should be deploying the same codebase to all your live machines at a time (or staging, or qa/test), the versions (or git sha1's) should be the same as well.

Finally, in the 'useful' list is `cap deploy:cleanup` - this will remove old deployments. Keeping a few around are useful, but they can take up a lot of space. As `cap --explain deploy:cleanup` says:

> Clean up old releases. By default, the last 5 releases are kept on each server (though you can change this with the keep\_releases variable). All other deployed revisions are removed from the servers. By default, this will use sudo to clean up the old releases, but if sudo is not available for your environment, set the :use\_sudo variable to false instead.

If you want to change the default to something other than 5, that can be set with the line "`set :keep_releases, 10`" in deploy.rb.

A few gotcha's
--------------

### When cap shell checks the source repo version

I've found that the latest version available in the main source code repository is only apparently checked when the Capistrano shell is first run. This can be useful if you want to check out to a limited set of machines, run a test and then check out to all the machines (you end up with the same version checked out in the same-named 'releases/' directory), but if you are sitting on the `cap>` prompt in Capistrano shell and doing multiple `!deploy` commands, you won't get new versions of code that have been committed to the repository. Exit the shell, and re-run to solve this.

### You checked out a new version, but you can't see it

Be wary if you are logged into the machine, and sitting somewhere inside the ./current/ directory. Because of the symlink is being changed underneath you to a new directory that is being pointed to (the newest subdirectory in releases/), if you do not do a `cd .` to refresh your location within the real directory tree, you will still be in an old copy of the code. The 'cd' makes sure you are in the latest place on disk, via the (now changed) symlink.

### Rolling back

Capistrano has the ability to remove the currently live version, and change the 'current' symlink to the previous location. Should the worst happen, and a website deployment fail, this can help, if 'rolling forward', with a fast-fix, check-in and redeploy may not be easily possible.

> `# to roll back to a previous deployment: cap> with !deploy:rollback`

If you have rolled back the webservers (php/app servers) you will have to restart php-fpm (or maybe Apache) on the servers, as they do not necessarily pick up the (old) versions of code that is being run now. The same would also be true if you have set [APC](http://uk.php.net/manual/en/book.apc.php) to cache the byte-code and not look at the time-stamp of files in case they change. I've found that [PHP-FPM](http://uk.php.net/manual/en/install.fpm.php "FastCGI Process Manager (FPM)") also has this issue.