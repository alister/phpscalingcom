---
title: 'Always have up to date documentation, part #1'
date: Sun, 16 Mar 2008 23:41:00 +0000
draft: false
tags: ['php', 'tools']
series: php-documentation
---

As I mentioned in my second post, [ZCE prep – and dumb tests](/post/zce-prep-and-dumb-tests-2/) - about open book tests (like Brainbench), having a copy of all the relevant documentation can be incredibly useful, if only from a speed issue. Knowing you can just open a new tab and type a few words to get the information on a function, or concept from the manual takes away so many problems.

I mentioned there that I have a local copy of the main [PHP manual](http://php.net/manual/en/) – and I wanted to tell you how I keep it, and a couple of other manuals up to date, as well as other documentation.

### Mirroring PHP.net

It took a little exploring to first find it, but there is a page on how to get make a mirror of the site - http://www.php.net/mirroring.php Although they aren't looking for more mirrors (unless your country doesn't already have a couple around), there's no reason you can't make a private mirror for your office, or just for yourself, as long as you don't update it too often and so cause excess load on the source website. I have a cron-job to sync my own copy once a week on a Sunday afternoon - when I think it was fairly quiet.

If you have a linux server (or a Mac OSX would be able to do it as well I expect), and Apache already running, then setting up a virtual host and dragging down a local copy isn't hard. You don't need Mysql running, but if you have SQlite3 installed, you will be also able to use the quick-lookup facility - very handy if you can't remember the order of parameters for strpos - http://php.net/strpos).

Here's my script I use - taken almost verbatim from the above mirroring.php page:

{{< gist alister 1385967 >}}

It may look a little more complicated than it it - but in short, it goes to the server at rsync.php.net, and gets the English version of the manual, but not the others, and it also doesn't bother downloading the large source files. In fact, the two largest parts of the site that is mirrored are the manual itself (currently some 44MB) and the manual backend (which includes all the notes, and most usefully, the lookup functions).

The first time you download the site, it's going to take a while - it is, currently, 133MB (though the -z in the rsync command line will compress the bytes on the wire, so there is less to download for the same results). However, most of that does not change week to week, and so the rsync protocol comes into it's own by only downloading parts of files that have changed (where they are sufficiently large to make a difference). Here's an example:

*   sent 71838 bytes received 9768512 bytes 57045.51 bytes/sec
*   total size is 116855428 speedup is 11.88

Here, the total size it would have downloaded was: 116,855,428 byes, 111MB. However, it only downloaded 9,768,512 (9.3MB) - an over eleven-fold improvement for the same results.

Next time - What else you should be reading, keeping around for reference, getting updated automatically, and generating from your own codebases!
