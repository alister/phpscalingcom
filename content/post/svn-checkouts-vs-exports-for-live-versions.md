---
title: 'svn checkouts vs exports for live versions'
date: Sun, 09 Mar 2008 19:45:30 +0000
draft: false
tags: ['best-practice', 'linux', 'subversion', 'tools']
---

I've read [http://www.svn-checkout.co.uk/2008/01/19/how-to-release-new-versions-of-websites/](http://www.svn-checkout.co.uk/2008/01/19/how-to-release-new-versions-of-websites/ "How to release new versions of websites") via [http://www.lornajane.net/posts/2008/SVN-Deployment-and-a-New-Site](http://www.lornajane.net/posts/2008/SVN-Deployment-and-a-New-Site) and while I consider revision control an essential tool (a few years ago, my job was the only one in the previous five years where I didn't have to install my own RCS), I somewhat disagree on the idea they suggest.

That first link, 'how to release new versions of websites', suggests checking out a version of the site as a working copy, (it's certainly something I've done before now), but then it goes on to use the 'svn switch' capability to move between versions (of course, they are also doing in in TortoiseSvn, and there for the live web-server is likely to be running Windows, no way I'd run a server on Windows - not even for testing). There is however some trouble with revision switching - especially on a busy site that has to keep running even while the new version is being put into place. While SVN does atomic commits - the new code goes into the repository all at once, or not, it's harder to do the all at once part on a non-transactional file-system - such as a webserver. The bigger problem is that rolling back will also take time - and in an emergency, the time taken to do something is crucial.

Here's what I do - Whenever I want a new version to go live, which might be from every couple of days to as often as a couple of times per day, I'll update run the script below with the specific revision number to export (and a date/time, but that's just for easy reference). When it runs, it also symlinks the given version as 'dev' - which is part of the path to the site 'dev.example.com', wham, instant new version, and I can trivially delete the symlink and on the same command line symlink the older version (with 'rm dev && ln -s 1234.20080102 dev'). After a quick wander around the newly checked out version, maybe run a set of unit tests, just for security, it's just as easy to put a similar 'live' symlink into place with a new link.

If there is ever a problem, rolling back to a previous live version is just as easy. Other configuration changes are made within the code on the apache servername, or the machine's own hostname (more useful for CLI scripts, generally run from cron).

There are some downsides - with a complete checkout of a 40-some megabyte website (it's mostly the Zend Framework and other libraries from which I use a number of files, though rarely all), it takes a little while (not too long, it's a gigabit link between the repository and the main webserver) and there are also some potential caching issues (Etags are usually based on the file inodes), but as we plan to move to a multi-machine cluster - and the images aren't being served from Apache, but a dedicated image webserver, that's not a significant issue - and even on Apache we don't have Etags enabled (Yslow from Yahoo also suggests that).

\[gist id=1385857\]

Update:  Since I first posted this, I've started using Capistrano. Look out for new posts on how to best use that.
---
### Comments:
#### 
[Deployment with Capistrano &#8211; the Gotchas | PHP Scaling](http://www.phpscaling.com/2011/11/24/deployment-with-capistrano-the-gotchas/ "") - <time datetime="2011-11-24 12:17:01">Nov 4, 2011</time>

\[...\] and entirely workable system – I described something just like this in a previous post: “SVN checkouts vs exports for live versions”. That was written and used before I was deploying to multiple machines however – and had to \[...\]
<hr />
