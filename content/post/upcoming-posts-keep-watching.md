---
title: 'Upcoming posts - keep watching'
date: Tue, 02 Jun 2009 10:28:55 +0000
draft: false
tags: ['php', 'quick']
---

Just a quick note on what is going to be posted in the next few weeks – I've got a few significant pieces in mind for various topics – including:

*   Doing the work elsewhere – asynchronous queues

This is going to be a series of articles – and to support it, I'm rewriting some code that I had originally wrote for my last job (v2, and so significantly improved over the original). First though, I'll tell you what is planned, and just how asynchronous queues are used and how they can be incredibly useful for scaling up any significant website, and not just in the obvious ways.

*   Mail queuing, on a vast scale

Dave Marshall has just posted an entry on [Using message queues to improve user experience](http://www.davedevelopment.co.uk/2009/06/01/using-message-queues-to-improve-user-experience/) where he queues up some emails in order to spool them out over the course of a few minutes. For the last 18 months, I'd been doing something very similar, on a far larger scale, with PEAR's Mail\_Queue. I'll show you how I did it and I'll show you how the messages were generated quickly, and how they could be sent out – and more importantly without destroying the system it was running on. As a bonus, I'll show you how if you run some form of internal mail system, you could save gigabytes of database space and give yourself vastly more flexibility.

*   Self stubbing mocks

Using the Mocking functionality in PHPunit & Simpletest can be complicated with the various calls that are required. There's just not much documentation around the PHP-world on how to run it. Another method, which can be easier to understand, is self-stubbing – putting your code into the test class. I'll show some examples of how to do that.

Finally, I'm going to be doing my ZCE exam in the next week or two – quite possibly on Thursday 3rd June (2009). Keep a close eye here for the results, and a follow-up.