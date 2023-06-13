---
title: 'Code Tombstones'
date: Mon, 28 Aug 2017 18:43:06 +0000
draft: false
tags: ['php', 'tools']
---

> Version 0.9 of [scheb/tombstone](https://github.com/scheb/tombstone) autoloads a file with a `tombstone()` function. See the bottom of the post for a fix to override that in your own code.

In a large project - particularly one in a dynamic language like PHP, as a project gets bigger maintaining full control of the code can be difficult. New features are written, old ones are changed or deprecated. Sometimes code is left behind, unneeded in later versions, but still in the code-base.

This can be even more difficult in a full-stack framework like Symfony or Laravel, where some of the code is only run via the framework - such as Event Listeners/Subscribers, or the authentication layer.

Code test-coverage is good, but unit tests can't tell you if the code *isn't* being used elsewhere. You can spend time updating code, and the tests that just aren't useful.

In late 2015, I came across an interesting potential solution - '**Code Tombstones**'. They are a more refined version of an exception or simple echo/die. In production all they do is make a log entry - 'I was here'. In a development environment, you might choose to immediately quit and complain - having the developer investigate if the code is genuinely useful or not..

Using a tombstone():
--------------------

1.  Something that looks like a date - it's just a string but it's useful to note when you put the tombstone call in
2.  Your name, or initials - who put it there
3.  An (optional) label

They are all just simple strings that will just be output as-is if the function is called, but knowledge is power when it comes to tracking down what is happening.

{{< gist alister 8e3aa440c08b16c6ed48a0e52ff977d5 example.php >}}

You could also put the call into a branch (one choice of an \`if\` statement, for example). You can even put in into a PHP file outside of a class - though this will risk a large number of 'ordinary failures' because PHPunit (for example) will often read many files to find testing code, and so any code outside of a class in a file will be run, showing an error.

Defining the tombstone() function
---------------------------------

This gets a little more interesting, because you may well want to have different configurations in different environments. In development or testing (including, for web-frameworks, like the Symfony _app\_test.php_ file) - a simple no-op - or you might want to send the messages produced to the console and then stop execution entirely.

In a production environment - the whole point is to just log and then continue.

{{< gist alister 8e3aa440c08b16c6ed48a0e52ff977d5 app.php >}}

In my prod-ready tombstone(), you see I also take the further step of catching any errors to then swallow them - you would not want testing code to break your live site!

There is one final thing I would suggest though - make sure you test that the tombstone will in fact write to the logfile on production. After a server move, I went several months not realising that any writes to the log was trying to write to a file that could not exist, but that error was being discarded. I put a quick failing-test URL that would force running the tombstone confirmed that the logfile was created as I had planned.

The library I use to write the logs comes from [Christian Scheb](https://github.com/scheb/tombstone), [(scheb/tombstone)](https://packagist.org/packages/scheb/tombstone), it takes care of the fiddly work of skipping enough of the [debug\_backtrace](https://php.net/debug_backtrace) to figure out exactly where the function was called, and to log it, by as many log-handlers as you care to set.

* * *

{{< youtube 29UXzfQWOhQ >}}

Video from [http://devblog.nestoria.com/post/115930183873/tombstones-for-dead-code](https://web.archive.org/web/20220111111029/https://devblog.nestoria.com/post/115930183873/tombstones-for-dead-code)

David Schnepper's Ignite talk, "Isn't That Code Dead?" - Velocity Santa Clara 2014

### Update: Dec 10th 2018 for version 0.9

As the [scheb/tombstone](https://github.com/scheb/tombstone) library now autoloads a default `tombstone()` function, if you want to override that with a custom version (per environment for example), you'll need to (potentially) move your versions in the web/app\*.php files to above the `require '../vendors/autoload.php';` line.
