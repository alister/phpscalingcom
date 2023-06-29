---
title: 'Speeding up your tests, and also your code coverage!'
date: Wed, 19 Sep 2018 21:42:44 +0000
draft: false
tags: ['php', 'testing', 'tools', 'phpunit']
series: phpunit
aliases: ['/2018/09/19/speeding-up-your-tests/']
lastmod: 2023-06-28
---

Slow running tests are a bug - they stop you from doing as much as you can with your code, & its quality. Spend a little time working on making your tests better, clearer, and faster - and you'll reap rewards from your work.

I've had a couple of useful improvements in the time spent running my PHPunit tests recently.

First was avoiding setting up the database in each test, and using the Alice library (via [AliceBundle](https://github.com/hautelook/AliceBundle)) to pre-create at least the majority of the data my tests will need. The data itself is mostly produced with the [Faker Library](https://github.com/fzaninotto/Faker).

One issue to be careful to avoid with this technique is to make sure that any new records that are written are unique - often dynamically creating such things as email addresses using either random numbers, or possibly an element of the current time. Reading those records back needs to know what you expect, but that is easily solved by storing the original value into a variable to check later.

Its all well having a clean and defined set of data when you start a test-run, but isolating the data between tests is also important - starting from the same place for everything means the underlying state is consistent. Now, it's easy enough to clear the database down before every test and rebuild it fresh, but most databases (certainly relational DBs like MySQL, Postgres and Sqlite) all have transactions, and the [dama/doctrine-test-bundle](https://github.com/dmaicher/doctrine-test-bundle) for doctrine plugs into PHPUnit to begin a transaction before a test-run, and rolls-back to undo everything from the test. This keeps most, if not all the changes in memory, and serves as a quick 'undo' for what would be the database changes from each test. For debugging the tests, and seeing the state of the database during, or after the changes, you can add an explicit call to have the database \`COMMIT\` the changes from the transaction to examine the state of the database, knowing that a fresh-run of the tests will empty the database and recreate it with more data to your specification.

Another issue with that is being unsure as to find something by (usually a primary key) - emptying the table before you start creating test data is usually done with the MySQL command \`TRUNCATE\` command, but this doesn't reset the primary key IDs. I think this is actually something of a hidden positive - and easily solved in your tests by searching for the known data before updating data and writing it back. I call it as a hidden positive because in the live environment you'll not usually have full control over primary key IDs and the matching data, so it's good to be aware of that limitation. The same search-first technique will also be required for other dynamic data if you are using the Faker library for generating data that is going to be used to search on. For example, I've got booleans in the database that might have a 50/50, or 80/20 chance of being set - with enough potential rows the the chance of none of them being set to true, (or false) is almost none, but I can run the test to check that at least some exist - and that checks the live code.

Writing all my test-data up front saved me a lot of effort within the individual tests, time taken to run the tests, and as importantly a great deal of memory. In fact, before writing all the data up front, my unit tests used over 1 gigabyte of RAM, afterwards with database rollbacks, that has halved.

An even bigger win in reducing the time taken came with an improvement to XDebug v2.6.0 - extension side code-coverage-filtering. Before that, the extension was collecting all the coverage for all PHP that was being run, if it was needed or not, and then PHPunit would filter the data - after it was already collected.

> The code coverage filter feature of [@xdebug](https://twitter.com/xdebug?ref_src=twsrc%5Etfw) works really well! Enabling it almost cut the build time of one of our CI jobs in half. [pic.twitter.com/v3VvZiNWC4](https://t.co/v3VvZiNWC4)
>
> — Arnout Boks (@arnoutboks) [August 14, 2018](https://twitter.com/arnoutboks/status/1029334090363879424?ref_src=twsrc%5Etfw)

For a few years, I'd avoided using Xdebug on my development machines because of the overhead - even when it was turned off (but it still had to check its status before not doing anything more substantial). I had instead used the 'phpdbg' CLI to run the tests with code coverage. I'm still not enabling Xdebug all the time within the PHP.ini (or more likely, the /etc/php.d/cli/ directory). Instead, I load it only for code-coverage test runs. You can see how in the code below, part of my build.xml configuration.

{{< gist alister 5a2da51a88cc5e8644e2c885a5c1319e build-snippet.xml >}}

As I've already said though, the biggest single win was the extension-side coverage filtering, and enabling that was just a few lines in my test-bootstrap file:

{{< gist alister 5a2da51a88cc5e8644e2c885a5c1319e phpunit-snippet.xml.dist >}}

{{< gist alister 5a2da51a88cc5e8644e2c885a5c1319e bootstrap_test.php >}}

My bootstrap also adds other code, like setting up a fresh database (see 'dama/doctrine-test-bundle', above) or test-specific code, like a [tombstone()](/post/code-tombstones/).

My entire full-build test can now take a little as 5 minutes - which includes 2 minutes for a large Behat test-suite (210 scenarios, over 1000 steps), 90 seconds for a full PHPunit test-coverage run (with more than 1,250 tests running 3,600 assertions) as well as a number of code-quality tools such as [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer), [PHP Copy/Paste Detector](https://github.com/sebastianbergmann/phpcpd) and outputting a range of reports, including the coverage in HTML and XML, [PHP_CodeBrowser](https://github.com/mayflower/PHP_CodeBrowser) and API/php documentation generated by [Sami](https://github.com/FriendsOfPHP/Sami). I should also add that this ~5 minute run-time happens on a lower-powered, spinning magnetic disc server.

> I started using phpdbg so I didn't have to have xdebug loaded all the time. I've now set my code coverage to just add the extension as needed, and the loader adds xdebug_set_filter for 'src/' - my coverage-run times have gone from 2m 52s for phpdbg to 1m 28s for [@xdebug](https://twitter.com/xdebug?ref_src=twsrc%5Etfw)!
>
> — Alister 'Blockchain emitting diode' Bulman (@alister_b) [August 29, 2018](https://twitter.com/alister_b/status/1034906997504843778?ref_src=twsrc%5Etfw)
