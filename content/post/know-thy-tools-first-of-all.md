---
title: 'Know thy tools first of all'
date: Mon, 17 Mar 2008 20:09:22 +0000
draft: false
tags: ['php', 'zend framework']
aliases: ['/2008/03/17/know-thy-tools-first-of-all/']
---

When you have a library, like [PEAR](http://pear.php.net) or [Zend Framework](http://framework.zend.com) – or even just the whole [PHP](http://www.php.net) language library – it’s absolutely vital you know what it can do.

What you don’t know can cost you weeks of effort and pain. I found this out (again) today, but it’s not my pain – it’s an employee who was too busy deciding that the Zend Framework wasn’t suitable for a simple cron-script task, he has spent most of the last few weeks duplicating something that is not as good as what I could write – with ZF – in about an hour.

I'd set him a fairly task - write a PHP script to loop through a maildir, deleting the messages marked (by headers) as spam, and extracting email addresses from a few different kinds of email bounces.  This, I gave to a senior ZCE qualified developer, but I had figured that even for a junior developer it would be no more than a couple of days job to help them get into the groove of developing code in a new position.

It's been a few weeks since I set him that task, and for the last week - with him working almost full time on it, it's been a couple of hours from completion.  He's not impressing me.

This afternoon, I got curious, so decided to have a go at it myself.  I'd already written the basics of the main loop:

{{< gist alister 1385998 >}}

That was the crux of the script.  There was a little more in the ParseBounces class to look at the headers and body to return the appropriate email addresses, but it's about that simple.

I didn't know enough about the `Zend_Mail_Storage_Maildir` class to be able to delete the message - but then, I expected a smart guy to go and fill in the blanks - and here there was just two - delete a message from a Maildir, and update a database record (which is pretty easy).

What I have from him, so far, includes an extention to `Zend_Mail_Storage_Maildir` to peer inside the protected class and another that decodes all the mails in the Maildir (and currently, there's over 20,000) before passing the emails and filenames into the loop where they are acted upon.

## Know thy tools first

Between 5pm and 5:50pm, I updated that script, shown above, to do what it required, including parsing a couple of the message formats (checking for SpamAssassin headers, a header made by the local MTA -Exim - and parsing a multi-part mime delivery failure message).

Spot the differences:

{{< gist alister 1386002 >}}

That didn't actually take me 50 minutes - I spent the first 15 minutes reading the APIs to see what '`Zend_Mail_Storage_Maildir_Writeable`' could do, that '`Zend_Mail_Storage_Maildir`' could not.  It was fairly safe to assume it had to do with change the contents of a maildir.  Indeed  it can - create new folders, rename them, move messages around, and delete them.

So, I've spent less than an hour and changing a few lines to code to do what has taken him weeks to do.  While there are some thing I can add - parsing more formats of bounce messages would be high on the list, what is there right now solves probably 80% of the task, and can be added to pretty trivially to do the rest.  because it's not parsing the entire directory full of messages first, it's going to take less time, and less memory - and if there's a problem, I can simply kill it part way, and it will have achieved something useful - and then will be able to pick up from where it left off.

I've not told him yet.  But it's amazing what you can pickup from reading the manual, and the code.
