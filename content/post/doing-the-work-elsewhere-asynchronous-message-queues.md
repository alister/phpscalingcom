---
title: 'Doing the work elsewhere - Asynchronous Message Queues'
date: Wed, 10 Jun 2009 15:43:29 +0000
draft: false
tags: ['advanced', 'beanstalkd', 'php', 'queues', 'scaling', 'workers']
series: queues
aliases: ['/2009/06/10/doing-the-work-elsewhere-asynchronous-message-queues/']
---

## The use of Beanstalkd as a queueing system

### What is an asynchronous queue

The classic [wikipedia quote (Message queue)](http://en.wikipedia.org/wiki/Message_queue)

> In computer science, message queues and mailboxes are software-engineering components used for interprocess communication, or for inter-thread communication within the same process. They use a queue for messaging - the passing of control or of content. Group communication systems provide similar kinds of functionality.

So one part of a system puts a message into a queue for another part to read from, and then act upon. The asynchronous nature means that each side is otherwise independent from the other, and does not wait for a response. That independence is an important part of the nature of the system though - and we'll see later how some of the more advanced functionality for our software of choice here can give some extraordinary flexibility to what can be done.

### Why use a queuing system?

You'd be surprised how few things need to happen right now - you go and buy a fancy coffee, and they write your order down, and put it into the queue for the Barista to make it. That disconnected set of actions works exceeding well for such distributed system (see [Starbucks Does Not Use Two-Phase Commit](http://www.enterpriseintegrationpatterns.com/ramblings/18_starbucks.html))

In much the same way as you not getting your coffee till it's made, what about web-sites that have to fetch (or produce) information. A couple of the simpler examples are when you've uploaded an image onto Flickr.com. That image has to be stored, and then resized into several files. If it's a large image though, it would take some time, and a lot of resources to be able to do that while you waited - time that you're left twiddling your thumbs. Instead, it returns immediately, and tells you that the image is being handled in the background - and in a few seconds, or maybe minutes, it shows up on your page.

How about waiting a few seconds for other information? How about, when you login to a social media website, it returns a simple webpage immediately with what it's got to hand, but then in the background, checks how many new messages you have, and displays them either by updating the page (with ajax), or when you view a different page. Is it so vital you find out that you have thirty old messages, and a few new ones - right now? For a web-mail system like Gmail, or Yahoo Mail, that is the point - but what about on another kind of site?

### BeanstalkD

Beanstalkd is a big to-do list for your distributed application. If there is a unit of work that you want to defer to later (say, sending an email, pushing some data to a slow external service, pulling data from a slow external service, generating high-quality image thumbnails) you put a description of that work, a "job", into Beanstalkd. Some processes (such as web request handlers), "producers", put jobs into the queue. Other processes, "workers", take jobs out of the queue and run them. From the [BeanstalkD FAQ](http://wiki.github.com/kr/beanstalkd/faq)

#### What can it do?

I've already mentioned a few ideas for things to have an asynchronous worker do, via a BeanstalkD queue, but there are a number of ways that it can be run, and a number of very useful facilities that BeanstalkD gives a producer of tasks.

#### Priorities

Simple enough to describe - given more than one task that could be run at a particular time, run the more important. The most urgent priority is 0; the least urgent priority is 4,294,967,295 (2^32).

#### Tubes

This is, in my mind one of the two secret weapons of Beanstalkd - together with a delayed job. Tubes, or 'named queues' can be created at will, and you can use as many different tubes as you want to put jobs into, but those jobs would only be returned to workers that were watching a given tube. Each worker could be watching many, but a single job can only be in a particular tube.

If you don't use a particular tube-name, it goes into 'default', but there's a lot of flexibility in sending particular jobs to specific workers, or groups of workers. For example, you could create a tube called 'sql' watched by workers on a database server, or even further limited by role.

File uploads can create special problems, unless you have some significant back-end systems, they will generally be uploaded to a front-end webserver and then have to be processed there, or moved on to somewhere else before they can be processed. This is a common event, so how do you make sure that any request to process an image can only be picked up by a particular machine? Send it to a tube named after the hostname of the server! As long as there is a worker process there, it will be picked up, and run. What it does from there, is up to it - it could resize the image, and save it to a local file system, or arrange for the file to be moved to a central file-storage area, and then fire another message into the queue for further processing there.

Although BeanstalkD doesn't (yet) have persistent queues saved to disk, you could also use a tube as a long-term hold. For example, throw a message into a tube called 'overnight-reports' - but don't have a worker pick it up immediately, instead one is only brought up to run the queue tasks in the quiet overnight hours.

The potential flexibility is enormous.

#### Delays

Another of the secret weapons, or killer features of BeanstalkD, is the ability to hold a message within the queue for a defined period before allowing it to be collected, and acted upon. If you have an action that has to be checked repeatedly, for example, has a particular person come online? then you can fire a number of identical tasks into the queue and allow them to slowly come out as the time passes.

It can also be useful to not do everything at once - maybe setting a lower-priority task that would run a few seconds after someone logs in - for example, updating an internal status or record - or checking for lesser-requested information.

#### How to use

Although BeanstalkD allows a large amount of information to go into the job-specification (the information that is held in the queue and passed between the producers and workers), I find that a simple string can hold at least a reference to what is required. I take my lead from URLs - and use them to direct the action to be run, and a few parameters as needed. For example - imagine the following strings being sent to a BeanstalkD worker, which it decodes and runs as a task:

 * /tasks/image/resize/filename/example.jpg
 * /tasks/image/resize/filename/example.jpg/sizeX/640/sizeY/480
 * /tasks/image/move/from/web1/to/centralstore/filename/example.jpg
 * /tasks/member/logintasks/id/12345
 * /tasks/event/add/id/12345/event/27
 * /tasks/mail/fetchcounts/id/12345
 * /tasks/mail/check-for-disallowed/id/596583405

Sending simple messages like these would require very little setup from the producer's side, and can be quite easily parsed by any worker process to pass on to a given function. In these examples (some of which I've used myself in live code), the path refers to a Zend Framework layout of module/controller/action & parameters. Rather than sending large amounts of text for the actual contents of a mail message (in the last example path), we simply refer to a record in the database for simplicity. Similarly for an image filename in the first item.

#### Next time:

Following articles in this series will show code to insert some messages into the queue. From there, I'll show you how to have a worker keep running reliably and pick and run the jobs as required.
