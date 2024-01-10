---
title: "It's All About The Data (part 1)"
date: 2024-01-02T15:06:25Z
draft: false
tags: ['PHP','data','optimisation','best-practice']
description: "Arrays in PHP."
codeLineNumbers: false # Override global value for showing of line numbers within code block.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
lastmod: 2024-01-10T16:40:00Z
---

{{% notice info "Comments?" %}}
You can comment on this post on my [Mastodon post](https://mastodon.cloud/@Alister/111728279387022221) for "It's All About The Data"
{{% /notice %}}

## ... and *arrays* aren't great for data

In my career so far (now over 20 years programming sites with PHP - half of that with Symfony), I've worked on a number of projects that dealt with a great deal of data - a couple of recent examples include menus for many restaurants and hotels (and each with hundreds of menu-items and then with more options available for each dish), and more recently, property-management which came with the issue of a 3rd-party API-based data-source with complex, sometimes inconsistent, and redundant data-structures, that would be returned and had to be normalized, simplified, processed and then output for the front-end JavaScript-based website.

What was common between both of these projects - as well as so many others I've seen - was that they were all made far more complicated and difficult by how the data was being fetched, passed around the system, and processed for use (invariably being JSON encoded for output as an API). The default method of moving data around to be processed, quickly becomes a matter of large structures with complex arrays of nested associated-array data, and that is only the *start* of the problems.

When you are dealing with a lot of data - and particularly when there are lots of disparate pieces within it - then knowing what is there, and what needs to happen to it next in any given situation quickly tends towards being **the most difficult** part of dealing with that data. Arrays don't need to have a particular structure, and they barely have specific types - especially when they come from an API - or when they are being output as an API.

When you start a new job - what do you find more complicated - the code that is right there in front of you in your IDE? Or is the bigger issue not knowing what data exists, and how it's structured - even (sometimes especially) with the code in front of you that processes the data? I know my answer.

<img src="/images/20240102-fellow-humans_300x170.jpg" width="300" height="170" align="right">

Computers are *really good* at doing what they are told - the problem is figuring out what is needed (and writing it down, or as we call it - programming) and for that, you need a human in the loop. But - there is only so much that we can fit into our brains at any one time. For computers, lots of data isn't a problem, and RAM is really pretty cheap - but the most expensive RAM is the grey matter between the developers ears, and the most expensive computational cycles, our time. We need to optimise the code and data that we deal with - to simplify for the developer.

<hr width="100%">

We'll get back to structure and types in the [next post](/post/its-all-about-the-data-2), but first lets see how even the simplest things can go wrong - like, what is something even called?

```json
{
	"hex_color": "#BADBED"
}
```
```json
{
	"hex_colour": "#BADBED"
}
```
It's clear what the problem is here - two different spellings of 'colour' - but now imagine it in the midst of a 10+KB unformatted JSON payload, or in a deeply nested set of arrays that have already been decoded from a JSON source.  Imagine how much more frustrating it would be to have a *different* property elsewhere in the payload for something else, and it was called <nobr>`rgb_colour`</nobr> there! Tooling and documentation, such as an OpenApi specification can help - but it will always be far from perfect. Most of the APIs I've seen recently are manually coded and then separately, the documentation - so that either could be wrong, and even if the *output* was was 'correct', the reference documentation wasn't ... or vice versa.

Systems like [API-Platform](https://api-platform.com/) can be a lot better - so much of the output and even documentation is driven directly from the actual code (the specific configurations of the data types, definitions and DocBlocks), so the number of times each item is mentioned and processed is also minimised, properties (and their types) are far more likely to be consistent throughout.

Are you creating the output manually though? I mean assembling the arrays that are being output, either with some model layer, or directly inside the controller action (you are using a MVC or ADR pattern, right?)  In so many projects it is simply not even specifically listed in any documentation - what is being output is what is used - probably. In others, the API documentation is written, after the fact, and then it's rarely looked at, or updated, again - even as updates and fixes are put in place.

Within PHP code, static analysis tools like PHPStan & Psalm have [array-shapes](https://phpstan.org/writing-php-code/phpdoc-types#array-shapes) that are great for checking PHP code is consistent, but they can be difficult to setup if the data is even *slightly* complicated - to say nothing of the sheer size that API payloads will often be (even without arrays of similar types of data), and keeping it up to date with any changes only adds to the difficulty.

### Comparing the memory size of PHP types with other dynamic languages

It's well known that PHP (like almost all other computer languages) isn't "designed, complete and perfect", it [grew and evolved](https://www.php.net/manual/en/history.php.php) - originally from a [C-based CGI script](https://web.archive.org/web/20210124025138/https://groups.google.com/g/comp.infosystems.www.authoring.cgi/c/PyJ25gZ6z7A/m/M9FkTUVDfcwJ?pli=1) script. One early choice that may have had a small part in helping it gain popularity was the simplicity of dealing with data - the PHP array.

In Perl, there is an array, and separate, specific, hash-type.

```perl
# An array represents a list of values:
my @animals = ("camel", "llama", "owl");
my @numbers = (23, 42, 69);
my @mixed   = ("camel", 42, 1.23);

# A hash represents a set of key/value pairs:
my %fruit_color = ( apple => "red", banana => "yellow");
```

```javascript
// JavaScript has essentially the same types available
cars = ["Saab","Volvo","BMW"];

// as a more 'object' oriented language, it stores a hash-equivalent on an object
person = {firstName:"John", lastName:"Doe"}; // anonymous object, with properties
//  ECMAScript 6 (released June 2015) adds distinct Map & Set objects into the language
```

PHP combined the two types - and can use them interchangeably, even in the same underlying array.
```php
$fruitsAndMore = ['apple', 1, 'lastName'=>'Doe'];
```
It's not typical (or even usually a good idea) to mix values & key/values like this - but it's quite possible.

Here is a (simplified) result (with types) of that `$fruitsAndMore` array:
```php
array(3) { 0=> "apple", 1=> int(1), "lastName"=>"Doe" }
```

This does make arrays (or associative arrays - depending on what you are normally storing in them) incredibly flexible and easy to use. There are inevitably significant technical downsides though - beyond the issues already mentioned.

### Behind the scenes - how do arrays work inside PHP?

**1. Memory - and the variables that use it - aren't free.** There is overhead that is required to store anything - as we need to know what type of thing we are storing, what it's called, and how large it may be.  'Small' values, like integers, floats and boolean's can be stored as part of the actual zval, so they are.

{{% notice tip "Zvals" %}}
Within the code that runs PHP, this internal structure to keep track of variables, and the value they are set to, is called a '[Zval](https://www.npopov.com/2015/05/05/Internal-value-representation-in-PHP-7-part-1.html#zvals-in-php-7)' and other dynamic languages have similar things.  In PHP v7, one of the significant upgrades from v5.6, was a change in how these were created and stored - and for other types such as associative arrays (AKA [hashtables](https://www.npopov.com/2014/12/22/PHPs-new-hashtable-implementation.html)) that uses the underlying zvals. These changes of how memory was used, were part of why PHP v7 was significantly faster than previous versions.
{{% /notice %}}

{{% notice note  "...other dynamic languages" %}}
This overhead/structure is also the case for other dynamic languages - like Ruby: [Optimizing Ruby’s Memory Layout: Variable Width Allocation - Object Allocation](https://web.archive.org/web/20221225051939/https://shopify.engineering/ruby-variable-width-allocation#Object)
{{% /notice %}}

**2. Each array - including each nested array - have their own overhead** - beyond the items that are held in the array. This overhead is only compounded with arrays within arrays - and an API can quickly end up returning a lot of embedded arrays.

{{% notice tip "pre-allocating space to grow" %}}
Although it doesn't help with memory use, PHP optimises the speed of using dynamic arrays by pre-allocating the internal space for them to be able to grow.  So, when you create a new array and have put 3 items in it, it already has room for a 8th. Adding the 8th item means that the array is then re-created with 16 slots, with 8 more available for use. As the number of items (or key/values) in a array increases, the number of unused array-slots is an overhead.
{{% /notice %}}


**3. On the bright side - Repeated key-names *should* be *nearly* free.** Simple strings as key-names will be very common when using associative arrays so PHP can store the strings and avoid the repetition - just the reference to the unique copy that was kept.

{{% notice tip  "Opcache required" %}}
Many of the optimisations that are available are only really available via Opcache. De-duplicated or 'Interned' strings, like the main task of the PHP Opcache module (compiling the PHP source code to reuse later), really come into their own when it is stored into shared memory and used between different calls to PHP.
{{% /notice %}}


{{% notice warning  "Opcache" %}}
If you don't already **know** that your PHP runtime has Opcache installed and being used then ***stop reading now,** and make sure it is being enabled & used for all your websites*. Cronjob or queue-runners, along with other CLI-based uses of PHP, generally can't use Opcache to its best (if at all), but that's an acceptable trade-off. Any website that is expected to get any useful number of hits or visitors per day is wasting the as-close-to-free performance boost if the Opcache isn't enabled.
{{% /notice %}}


### Hmmm, arrays are bad, OK?

OK - what should I use?   **Objects**.

I'll go into more details - how, and why, in my [next post](/post/its-all-about-the-data-2).

----

Extra reading:

By [Larry Garfield](https://www.garfieldtech.com/) / [Crell](https://phpc.social/@Crell)
* [PHP: Use associative arrays basically never](https://steemit.com/php/@crell/php-use-associative-arrays-basically-never)
* [PHP: Never type hint on arrays](https://steemit.com/php/@crell/php-never-type-hint-on-arrays)
* https://presentations.garfieldtech.com/slides-never-use-arrays/phpkonf2021/

By [Nikita Popov](https://www.npopov.com/)  / [nikic](https://www.npopov.com/aboutMe.html)
* [PHP's new hashtable implementation - Memory utilization](https://www.npopov.com/2014/12/22/PHPs-new-hashtable-implementation.html#memory-utilization)
* [How big are PHP arrays (and values) really? (Hint: BIG!)](https://www.npopov.com/2011/12/12/How-big-are-PHP-arrays-really-Hint-BIG.html)
