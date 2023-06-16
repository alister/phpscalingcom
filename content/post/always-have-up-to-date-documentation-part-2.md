---
title: 'Always have up to date documentation, part #2'
date: Mon, 24 Mar 2008 15:38:28 +0000
draft: false
tags: ['php', 'tools', 'zend framework']
series: php-documentation
aliases: ['/2008/03/24/always-have-up-to-date-documentation-part-2/']
---

see my [previous post](/post/always-have-up-to-date-documentation-part-1/) on the topic, #1.

My last post ended up more as a how-to than what-to. This time, I'll say why you should have local copies of the documentation for most of the tools you use. I'll also tell you the sort of things I always have handy as well.

Getting a local copy of php.net - and getting installed as an apache vhost and updated (probably weekly) is some effort, but well worth it. I've said it before, but PHP.net is the **best** language reference site that I've seen. It's kept up to date (sometimes ahead of the code releases in fact) and while the notes that are added to it can sometimes confuse, as much as help, when they do help, they will really make the difference.

I don't tend to buy many PHP books, because what can they do besides re-iterate what is is already there?

The most important thing to bear in mind though is not to just have the documentation there to read - you have to know what is available. Projects like PHP, the Zend Framework and PHPUnit have a lot of parts - and knowing that they have things - even if you don't know how they work right now, can save you days or weeks of effort.

It's for that reason that you need to at least scan over all the the docs you have - and indeed for all the libraries and tools that you use. Even I don't read everything and expect to remember it all - but I remember enough to recognise that a paticular tool might have something to help - maybe PHP has something to search the values in an array (http://php.net/array-search), or can use Oracle, or Ldap, or Memcached, or that Zend Framework can let you easily loop over maildirs (or an mbox) to get each mail from within it. If you don't read the manual - at least skimming over it, you would never know that functionality exists, and you inevitably end up reimplementing other people's already debugged code. That's a waste of your time.

So, take an hour now, and assemble a directory to put these docs into, and read through them - not everything, but at least look at headers of every section, just to get an idea of what is available and maybe go back and read up some more on things that may be useful to you. If something isn't so interesting to you now, do bear in mind, your next project, or job, might change that.

Above all, keep learning. Never stop.

### A few of the library docs I have handy - and how

#### PEAR library

I generally just download the latest many-HTML-page tar.bz2 file from http://pear.php.net/manual/ and drop it into my docs directory.

{{% notice warning "Now obsolete" %}}
The PEAR library has been completely replaced with Composer packages.
{{% /notice %}}


#### Zend Framework

For ZF, there's two sets of documentation - the main docs, and the generated API. Also, since they are released as versions, with the code, rather than continually upgraded. Downloading and unpacking both styles of documentation is easy, and then I'll just add a quick index.html file in the base to quickly bounce into either the manual, or API.

{{% notice warning "Now obsolete" %}}
There are newer versions of the Zend Framework (now called Laminas), but Symfony or Laravel are the usual choices for new development.
{{% /notice %}}

#### PHPunit or SimpleTest

Simpletest is easy, copy the documentation from the source tar-ball. While the team that develops it is getting quickly obsoleted by [PHP Unit](http://phpunit.de), they are still ahead on mocking and web-form testing. There are also some ideas somewhat buried in the online docs for upcoming features.

For updating the phpunit docs, I've taken to checking out a [copy of the docs](http://www.phpunit.de/browser/phpunit_pocket_guide/branches/3.3/en) and running the [build script](http://www.phpunit.de/browser/phpunit_pocket_guide/build) to get the HTML, and then like ZF, having a redirect [in this case, a `header('Location: ...')`] to bounce the browser into the right place to start reading. An Apache alias or just an easy link from the top level into the generated docs are other trivial ways to do that.

#### Subversion - the book

http://svnbook.red-bean.com/ and of course, they have the book sources in [subversion](http://svnbook.red-bean.com/trac/browser/trunk/src/en/book) Probably as easy to just download the multi-file tarball occasionally.

Although TortoiseSvn, and other tools like it make the every day check-ins and updates easy, I know I'll still hit the manual for branching for a while yet.

{{% notice warning "Now obsolete" %}}
Git is almost the default choice for code versioning now.
{{% /notice %}}


#### Your own code

You thought you were going to get away without a reminder to write your own documentation? Wrong!

*   [Php Documentor](http://pear.php.net/package/PhpDocumentor)
*   [phpXref](http://phpxref.sourceforge.net/) - A bit of an old-timer, it's actually written in perl, but it will link parts of your code around a large number of HTML pages - useful for code exploration.

The Pear [PHP\_DocBlockGenerator](http://pear.php.net/package/PHP_DocBlockGenerator) is also useful to generate the place-holders in a newly written file that might be a little light on function docblocks, though it does tend to go a little overboard with variables. If there are docblocks - even empty ones, it won't change them, but it will add function parameter names for you to go back and explain what they are, and why they are there.

One little optimisation I've done with a project I'm working on is that since the site is some 45MB - most of that pure PHP (library) code, I don't bother running the document generation unless a PHP file has changed.

{{< gist alister 1386020 >}}

I hope those help, but remember - you have to know where to look for help - so read!
