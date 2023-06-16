---
title: 'Improve your code coverage percentage - delete code!'
date: Mon, 04 Sep 2017 12:14:17 +0000
draft: false
tags: ['php', 'testing']
aliases: ['/2017/09/04/improve-your-code-coverage-percentage-delete-code/']
---

A recent post showed how to setup [Code Tombstones](https://phpscaling.com/2017/08/28/code-tombstones/) - but there are other , even more insidious pieces of code in a project. The code you know you aren't using now, but you wrote ahead of time - because  you think it will be useful, or you have plans for it, or any one of a dozen more reasons.

Chances are - you might never get back to it, and it's just taking up important space. Disk space, or space in version control isn't worth worrying about though - it's the space in your brain that is most important for a developer.

You have to [let it go](https://www.youtube.com/watch?v=L0MK7qz13bU&t=1m4s).

Here's the results from one of my own projects:

Before deleting unused code:
* Classes: 30.22% (55/182)
* Methods: 51.65% (515/997)
* Lines: 40.99% (2080/5075)

After deleting code, and adding a few new tests

* Classes: 34.52% (58/168)
* Methods: 58.57% (523/893)
* Lines: 52.18% (2299/4406)

... and all it took was deleting 2,615 lines of PHP classes, tests, Twig templates and yaml configuration - none of which were being used, or were in a state to use right now. The number of unit tests also dropped from **1,003** with 4,026 assertions, down to **984** and 3,966 assertions.

### Because of the wonders of Git, and version control...

None of that code need ever be forgotten - I made a quick and simple branch - `deleting-code` branch and removed a block of related code (at least what I could find) per commit.

* 74a2e09 - (deleting-code) Removed unused commands
* 622295f - Remove deprecated code
* 4783f79 - Removing unused ContractorsList service
* 822e57b - Datatable & DemoTable routing and Javascript
* e28e535 - Remove RecruitersDemo - generated fake data on the fly
* 131864b - Removing the Mailable & Mailing code.
* e5b53b5 - Removing most of the 'Fakes' code
* ce8f717 - Start deleting unused code

I've also matched those commits with some issues, and plenty of searchable text that I can use to find them in the git log.

I, for example, I decide to come back and take another look at the 'Mailable' code (which is intended to make configuring and sending emails easier) - I can just search for 'emails' or 'mailable' and revert just a single commit - and I'm right back where I need to be.  But for now, I want to just do it the 'hard way' with simple SwiftMailer-related code before I decide what _can_ be done in an easier way.
