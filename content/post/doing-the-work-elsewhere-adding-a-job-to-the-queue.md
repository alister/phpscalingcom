---
title: 'Doing the work elsewhere - Adding a job to the queue'
date: Tue, 06 Oct 2009 22:30:26 +0000
draft: false
tags: ['beanstalkd', 'php', 'queues', 'scaling', 'tools', 'workers']
series: queues
aliases: ['/2009/10/06/doing-the-work-elsewhere-adding-a-job-to-the-queue/']
---

I've previously shown you why you may want to put some tasks through a queuing system, what sort of jobs you could define, plus how to keep a worker process running for as long as you would like (but still be mindful of problems that happen).

In this post, I'll show you how to put the messages into the queue, and we'll also make a start on reading them back out.

For PHP, there are two BeanstalkD client libraries available.

*   BeanStalk.class.php - [http://sourceforge.net/projects/beanstalk/](http://sourceforge.net/projects/beanstalk/)
*   Pheanstalk - [http://github.com/pda/pheanstalk](http://github.com/pda/pheanstalk)

Although I've previously used the first class in live code, I'm preferring the second, 'Pheanstalk', for this article. It is more regularly worked on, and uses object orientation to the fullest, plus it's got a test suite (based on Simpletest, which is included in the download).

Using it, according to the example is simple:

{{< gist alister 1385813 >}}

The `pheanstalk_init.php` file adds an autoloader, though you may find it advantageous to move the main class file hierarchy from where it had been downloaded into its own directory so that an existing (for example Zend Framework) auto-loader can find it.

As you see above, the Object Orientation lends itself well to (an optional) 'fluid' programming style, where an object is returned and then can be acted on in turn `$pheanstalk->useTube('testtube')->put("job payload goes here\n");`

So, putting simple data into the queue, is, well, simple (as it should be). There are advantages in wrapping this simplicity into our own class though. Some examples

*   We want to put the same job into the queue multiple times - for example, a call to check some data in 1, 10 and 20 seconds time.
*   Adding a new default priority - or with multiple classes, a small range of defaults
*   adding in other (meta) information about the job that is being run, such as when it was queued, and how important it is. Some tasks might be urgent, but not important - ie, if we have the opportunity, run them now - but it doesn't have to be run at all.

Each may be simple enough to create a simple loop, but it might be advantagous to push that down into a class - and especially with the final idea.

How to store the meta-information then? It should be a text-friendly, but concise format, and quick to parse. Here, JSON (or the related Yaml) fits the bill quite nicely.

{{< gist alister 1385819 >}}

Processing it at the other end, after it has been fetched by the worker is a simple matter of running 'json\_decode()' and extracting the `['task']` from the results before running it.

---

### Comments:

[Aaron]() - <time datetime="2012-01-26 02:14:57">Jan 4, 2012</time>

Moving from Amazon SQS to Beanstalkd. Does $pheanstalk->watch() return the oldest item added to the tube? Or does it return multiple items? Also, can $priority be null? Will this effect FIFO order? If all items have the same priority are they delivered to the worker FIFO? Thanks. I would love to see more examples of pheanstalkd.
<hr />

[alister](http://abulman.co.uk/) - <time datetime="2012-01-26 10:45:23">Jan 4, 2012</time>

You don't have to actively use priority, but in that case, just default it to some number (I'd personally say a fairly high number, and so make it a low-priority, in case you did want to set something as more important later). If there aren't priorities being used, then it would be plain FIFO. Different priorities will then override the first-in-first-out order and be returned earlier. `reserve()` only returns one at a time (`watch()` just says where you will look for the items), but you can keep calling it to get more. In my last project, I collected 100 items from the queue, fetched data about them all at the same time (from the Twitter API), and then went back to delete all 100 (via the job-ID I stored when I had read them originally). I hope this is useful to you. If you have a request for another post, Aaron, then please let me know what you would like to know more on, and I'll see what I can do!
<hr />

[Michael](http://michaelhasselbring.com) - <time datetime="2012-06-10 04:00:05">Jun 0, 2012</time>

If I have multiple workers watching the same tube, can the same job overlap? Such as if one worker grabs a job, the other worker won't grab the same job right, even if the first worker hasn't finished?
<hr />

[alister](http://abulman.co.uk/) - <time datetime="2012-06-10 11:47:44">Jun 0, 2012</time>

The same job won't be given to two (or more) workers unless the connection to the worker is dropped before it has been deleted, or unless the time allocated (the TTR) to run the particular job has expired. If it's taking a long time, but the worker is handling it, you can also 'TOUCH' the job, to reset the clock. There's also a writeup of this on the FAQ - https://github.com/kr/beanstalkd/wiki/faq - under "How does TTR work?".
<hr />

[LornaJane](http://lornajane.net) - <time datetime="2014-03-11 10:04:27">Mar 2, 2014</time>

Thanks for this, I'm doing some beanstalk stuff now and finding a LOT of your resources around. Thanks for taking the time :)
<hr />

[alister](http://abulman.co.uk/) - <time datetime="2014-03-11 21:38:59">Mar 2, 2014</time>

Happy to help!
<hr />
