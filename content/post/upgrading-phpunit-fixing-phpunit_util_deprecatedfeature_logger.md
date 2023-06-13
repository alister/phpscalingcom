---
title: 'Upgrading PHPunit - fixing PHPUnit_Util_DeprecatedFeature_Logger'
date: Thu, 11 Jun 2015 13:04:07 +0000
draft: false
tags: ['best-practice', 'phpunit', 'testing', 'tools']
series: phpunit
---

Having just watched Sebastian Bergmann's ["The State of PHPUnit" presentation](https://ftp.belnet.be/mirror/FOSDEM/video/2015/devroom-php_and_friends/thestateofphpunit.mp4) from Fosdem 2015, I was inspired to install and test a project of mine with the latest stable PHPUnit - v4.7. It was easily installed on the command line.

`composer global require "phpunit/phpunit"`

I installed it as a new, global, tool because in my project I am using the "[ibuildings/qa-tools](https://packagist.org/packages/ibuildings/qa-tools)" repository to install and help run a number of QA tools - and the stable 1.1.\* versions lock PHPunit to v3.7 - the last released version of which was in April 2014.

A good part of the reason to do so - beyond using the latest version - was also to enable the strict tests

```
<phpunit
    beStrictAboutTestsThatDoNotTestAnything="true"
    checkForUnintentionallyCoveredCode="true"
    beStrictAboutOutputDuringTests="true"
    beStrictAboutTestSize="true"
    ... more parameters ...
    colors="false"
    verbose="true"
>
```

This blogpost is to help someone else that tries it - and comes across the same issue I did:

> PHP Fatal error: Class PHPUnit\_Util\_DeprecatedFeature\_Logger contains 1 abstract method and must therefore be declared abstract or implement the remaining methods (PHPUnit\_Framework\_TestListener::addRiskyTest) in ...vendor/phpunit/phpunit/PHPUnit/Util/DeprecatedFeature/Logger.php on line 201

My fix was simple - it took some systematic editing of the phpunit.xml file to figure it out. At first, I tried commenting out the various add-on I've got for PHPunit, tools to report slow tests, and to automatically close and check the results of any Mockery expectations. None of them helped, and so I started on the parameters in the opening XML tag of the file.

The actual fix was simple - the problematical line for me was:

`colors = "false"`

Removing that from the top of the phpunit.xml file, solved my issue, and now I've also gone on to update the "[ibuildings/qa-tools](https://packagist.org/packages/ibuildings/qa-tools)" package to dev-master to get the latest-and-greatest (including automatically pulling in PHPUnit v4.\* and Behat v3, among others). It was reassuring to know that I had the previous configuration safely stored in version control - so I could always just revert back to something that had worked. Running a separate copy of PHPunit installed outside of the project didn't hurt either.

I've said for a long time that "you don't get paid the big bucks for knowing what to do - it's for knowing how to fix it when you make the inevitable screw-ups".

Now, when I run my PHPunit-tests, I get a lot more warnings about 'risky' tests (all of it "This test executed code that is not listed as code to be covered or used") - but those aren't big issues for me right now.

The take-away is, don't be afraid to upgrade, and if there is a problem, systematically (temporarily) commenting, or removing configuration, or code, can find the issues surprisingly quickly.
