---
title: 'Replacing @expectedException with $this->expectException()'
date: Tue, 08 Aug 2017 18:59:44 +0000
draft: false
tags: ['php', 'testing', 'tools', 'phpunit']
series: phpunit
---

One of the advantages of a side-project is that you can be a little extra passionate about getting things _just right_. If you want to increase code coverage because you think that it's good, you can - after all, it's just some time now doing things that you like.

So, earlier in the year, when I saw Sebastian Bergmann's article on ['Questioning PHPUnit Best Practices'](https://thephp.cc/news/2016/02/questioning-phpunit-best-practices), I added it to a little (well, it currently stands at a count of 20 items...) list of clean-ups and improvements.

Well, a couple of days ago, I had the time to take a look at it, just a simple - almost relaxing - piece of work to convert from annotations-driven Exception testing to code-driven expectation tests.

The idea itself is quite simple - take a piece of code like such:

```php
use App\CanThrowAnException;
use App\Exception\Complicated;

/**
 * Pass the test if an exception is thrown
 *
 * Assert what should be happening
 * @expectedException \App\Exception\Complicated
 */
function testException()
{
    // setup the situation
    $systemUnderTest = new CanThrowAnException();`

    // act
    $systemUnderTest->shouldThrowComplicated();
}
```

... to the newer style, `$this->expectException(...)`, which came in with [PHPUnit 5.2](https://github.com/sebastianbergmann/phpunit/wiki/Release-Announcement-for-PHPUnit-5.2.0).

```php
use App\CanThrowAnException;
use App\Exception\Complicated;

/**
 * Pass the test if an exception is thrown
 */
function testException()
{
    // setup the situation
    $systemUnderTest = new CanThrowAnException();

    // assert what should be happening
    $this->expectException(Complicated::class);`

    // act
    $systemUnderTest->shouldThrowComplicated();
}
```

This gives a couple of advantages -

1.  The test is contained entirely within the code. This means that the expectation of what should happen can be set after the initial setup has happened, giving a setup/assert/act direction of code
2.  Since it is now in PHP code, and no longer a annotation comment, it can use the namespace functionality and the `::class` name resolution for shorter names, that will also still easily usable by automated PHP-parsing tools (For example, PHPStorm reads and understands the `use` lines to find all instances of the classes, even if they are aliased).

The rewriting from the annotation to function call is almost trivial, and there are also some new (related) function calls for the other `@expectedException...` annotations

```php
@expectedExceptionCode -> $this->expectExceptionCode(int $code);
@expectedExceptionMessage -> $this->expectExceptionMessage($msgToExpect);
@expectedExceptionMessageRegExp -> $this->expectExceptionMessageRegExp($regexp);
```

The last thing I did before moving on was make sure that new `@expectedException` annotations didn't creep back into the codebase. For that, I got a little 'hack-y'. There are probably better ways to do it (with a [PHPCS](https://github.com/squizlabs/PHP_CodeSniffer) sniff), but I couldn't find an easy way with a quick search - so I built my own.

First of all, a little history of how I develop software, and run tests. Back in 2013 I published what I called '[personal-ci](https://github.com/alister/personal-ci)'. It is, at it's core an `ant` `build.xml`control file that runs a whole suite of tests. I'm still using the same system now, extending it very occasionally with new tools.  I'll run the full set of tests maybe a couple of times a day while a system is in frequent development and that will take about five to six minutes to run. A slightly smaller subset of the unit tests will only take two seconds though!

The difference in, in 5.5 minutes, it will also run a full php\_codesniffer report, [phpcpd](https://github.com/sebastianbergmann/phpcpd), over 170 Behat integration scenarios, PHPUnit with code-coverage reports, [PHPMetrics reports](http://www.phpmetrics.org/) and [Sami](https://github.com/FriendsOfPHP/Sami) for documentation.

So, in my main build.xml file - I setup a couple of searches in the src/ and tests/ directories - if the string '@expectedException' appears - fail. If one did stray back in, actually finding them is easy with [ack](https://beyondgrep.com/) or my own new favourite - [ag](https://github.com/ggreer/the_silver_searcher).

```xml
<!-- If there are any '@expectedException...' annotations, fail. -->
<target name="noExpectedException">
    <property name="fail.string" value="@expectedException"></property>
    <fileset id="hasexpectedException_in_src" dir="src">
        <contains text="${fail.string}"></contains>
    </fileset>
    <fileset id="hasexpectedException_in_tests" dir="tests">
        <contains text="${fail.string}"></contains>
    </fileset>
    <fail status="1" message="One or more '@expectedException' detected">
        <condition>
            <or>
                <resourcecount when="greater" count="0" refid="hasexpectedException_in_src"></resourcecount>
                <resourcecount when="greater" count="0" refid="hasexpectedException_in_tests"></resourcecount>
            </or>
        </condition>
    </fail>
</target>
```

I've added that check to my php-linting action, which is run by default when `ant` is run.

```xml
<target name="php-lint-ci" depends="get-changeset.php.spacesep,noExpectedException"
    if="changeset.php.notempty"
    description="Perform syntax check of sourcecode files in parallel">
    <!-- run `php -l` on the files for a quick safety check -->
</target>
```

So, that's the latest addition to my 'personal-ci' workflow, updating to the latest best-practices in testing exceptions with PHPUnit, and making sure that the old-style annotations can't be used in future.
