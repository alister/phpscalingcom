---
title: "It's All About The Data (part 2)"
date: 2024-01-04T12:06:21Z # Date of post creation.
draft: false
tags: ['PHP','data','optimisation','best-practice']
description: "Using objects for structuring and storing data."
codeLineNumbers: false # Override global value for showing of line numbers within code block.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
codeMaxLines: 25 # Override global value for how many lines within a code block before auto-collapsing.
---

## ... What should I use? Objects.

![Thanks for coming to my TED talk:inline](/images/20240104-wave-goodbye_276x183.jpg)

There are _two_ reasons. An obvious reason - and then the actual reason that is more subtle - and is the best reason to use objects (pretty much everywhere).

### Objects use less memory

If you only need to deal with a few small, and uncomplicated things - pieces of data - then almost anything will work - especially if you won't be using it more than once. When there is a lot of data, then using more memory for each part of your data is not your friend.

I've already mentioned the [memory overhead of arrays](/post/its-all-about-the-data-1/#comparing-the-memory-size-of-php-types-with-other-dynamic-languages), and I won't go into that again, but I do want to show one more example - with some [PHPBench](https://github.com/phpbench/phpbench) results.


```
$ vendor/bin/phpbench run --report=default --group=array  --retry-threshold=5
PHPBench (1.2.15) running benchmarks... #standwithukraine
with configuration file: /home/alister/repo/tmp/php-objects-vs-arrays-benchmark/phpbench.json
with PHP version 8.2.14, xdebug ❌, opcache ✔

\Php\Bench\ObjectVsArrayBench

    benchArray..............................R1 I9 - Mo30.36000ms (±0.97%)
    benchObject.............................R1 I19 - Mo28.40263ms (±1.10%)
    benchObjectCollection...................R1 I11 - Mo29.21068ms (±1.63%)
    benchObjectSetters......................R1 I11 - Mo52.69392ms (±1.60%)
    benchObjectSettersConstruct.............R1 I19 - Mo43.92354ms (±1.59%)

Subjects: 5, Assertions: 0, Failures: 0, Errors: 0
+--------------------+-----------------------------+-----+------+-----+----------+------------+--------+
| benchmark          | subject                     | set | revs | its | mem_peak | mode       | rstdev |
+--------------------+-----------------------------+-----+------+-----+----------+------------+--------+
| ObjectVsArrayBench | benchArray                  | 0   | 1    | 20  | 43.367mb | 30.36000ms | ±0.97% |
| ObjectVsArrayBench | benchObject                 | 0   | 1    | 20  | 18.007mb | 28.40263ms | ±1.10% |
| ObjectVsArrayBench | benchObjectCollection       | 0   | 1    | 20  | 18.008mb | 29.21068ms | ±1.63% |
| ObjectVsArrayBench | benchObjectSetters          | 0   | 1    | 20  | 18.008mb | 52.69392ms | ±1.60% |
| ObjectVsArrayBench | benchObjectSettersConstruct | 0   | 1    | 20  | 18.008mb | 43.92354ms | ±1.59% |
+--------------------+-----------------------------+-----+------+-----+----------+------------+--------+
```

The important part in the table are the values in the **'mem_peak'** column - particularly for the '**benchArray**' and '**benchObject**' rows, indeed all of the object-based rows. While the constructor and setters do take additional time, there is almost no more memory use. With this example, the objects take just 42% of the total amount of memory as the array version. Alternatively, you could be storing more than twice as much data in the same amount of space.

All of these tests put 100,000 items (either into a small associated-array, or an object that contains the same data) into a larger array - except the <nobr>`benchObjectCollection`</nobr>, which puts the objects into an `\ArrayObject` instance instead of a normal array.

{{% notice note "Benchmark sources" %}}
See [alister/php-objects-vs-arrays-benchmark](https://github.com/alister/php-objects-vs-arrays-benchmark) (updated with an up to date version of [PHPBench](https://github.com/phpbench/phpbench), from the original by [dbalabka](https://github.com/dbalabka/php-objects-vs-arrays-benchmark)). The specific class that is being shown above is the [ObjectVsArrayBench](https://github.com/alister/php-objects-vs-arrays-benchmark/blob/60621615c787ef1ca082d06eebdfe35184b9c116/benchmarks/ObjectVsArrayBench.php#L49-L78), if you want to see exactly what is being done.
{{% /notice %}}

***It isn't even about the memory size...*** - though it can be incredibly useful to do it with less memory, or to be able to work with more.

I've dealt with databases with just a few useful rows, all the way up to hundreds of millions and more. The potential problems with such large amounts of data are well understood, and I've already written about one such '[MySQL optimisation Story](https://alister.github.io/blog/2013/01/01/a-mysql-optimisation-story/)' before.

{{% notice tip "Large scale databases" %}}
You'd be surprised how much can be done by a decently configured off-the-shelf database server. With a couple of terabytes worth of good-quality SSDs and 128GB+ of RAM will solve most problems before even considering making some proper (and not so complicated) improvements like optimising indexes, simplifying data, pre-generating what might be complex searches, or beginning to partition data.
{{% /notice %}}

**None** very few of the potential fixes of large-scale data will help you when you are dealing with the results of a single (complex) database query though - or worse, a large API payload that needs to be processed and returned out to a browser as a JSON API.

### Objects have types, a well defined structure & built-in methods

Data: useful information is three (or four) things.

- The context and structure
- The values
- The types

#### Context and Structure

What does `20240109` mean? It could be a date, how long since something happened, or the number of widgets sold in the last month. Without a type (integer or date, or something else) we don't know. Even with a specific type (say a Date), without context and the structure of the other properties around it, it has no useful meaning.

Our job as developers is to keep the structure and context (the meaning) of the data we get, and to process and display what has been derived from its processing to add further information to summarise, or otherwise add to the usefulness of that information.

We need to keep the data we are dealing with, the structure of it, and the specific types that both help to make that collected data understandable and useful. There will be an inevitable overhead to the process, but that (really quite minor) price is well worth the cost in 99.99% of the time we spend on clarifying what it all means.

{{% notice warning  "but in this case...." %}}
When you think you might just be in the 0.01% of the time when spending time and effort - especially CPU time - on doing it a 'faster' way - know that _you are invariably & definitely **incorrect**_. Your time is multiple orders of magnitude more expensive than buying, or renting a bigger, faster computer for all but the most extreme tasks and amount of data. Optimise for a human to understand the system, and the code&nbsp;first.
{{% /notice %}}


Keep the data structured - build objects at multiple levels and map data into them. With technologies such as ORMs (like Doctrine), this can already be taken care of - the data was written into an object, a hierarchy (or list) of classes somewhere and will usually be returned in the same way from the database - but other systems like a call to (or from) an external API often doesn't and especially if we are responsible for fetching the data ourselves. At every level, take the raw data and give it the appropriate meaning and context within the entirety of the data we are working with. Hopefully the original source has some documentation and thought about what each part of the data means, but if it's over-complicated then simplifying is an option.

Maybe is it better to keep the fidelity of the original data (if not the total structure if it can be improved), but add the flexibility by how your classes and objects retrieve that data. If you wanted a full name, but the data only has first-name & last-name - a <nobr>`getFullName()`</nobr> method can return a simple concatenated version (though you will probably be [wrong](https://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/)). Dates can default to 'now', or to another value (so <nobr>`updatedAt`</nobr> matches <nobr>`createdAt`</nobr>, for example). The data available and what is needed, dictates what needs to be done within the code and we help define the meaning of the data where it is first used. Defining the meaning of a piece of data is far from the simplest part of using it.

In my original example of a property-management system, there is a multitude of information being held about a single property - potentially multiple owners, agents, managers, and service staff. There would also be current (or former) tenants - or someone living in the property they own. There are the required utilities (Gas, electric, water etc), with all the data being stored and updated, and all the bank-details for everyone that was relevant. Many similar 'shaped' pieces of data for people, organisations and other information, but with some subtle differences between them too. Keeping track of what each level or group of people or organisations required would be (and was) a nightmare. Without the context that a well-defined structure of data objects can help to provide, confusion is inevitable. Instead, with a hierarchy of related objects, the structure can, and certainly should be, as self-describing as possible based on what exists within each object's properties.

#### The Types

Importantly, define and convert the data into specific types (and then moving that into the structure) as early as possible after it has been fetched - and do that conversion only once and in only one place. Having to re-convert or check the same data (or a subset) repeatedly, results in spreading the knowledge of what of the data _means_ around the system. That repetition adds complexity, and makes it harder for the developers (yourself, and others) to understand the system as a whole, and more specifically the data that the system is dealing with in general. Unless you can '[grok](https://www.merriam-webster.com/dictionary/grok)' the data, at least the part that you are dealing with, then understanding the wider system becomes vastly more difficult.

Consistency is key. All dates with a time should be a DateTime Object - and if there's any chance of being used in a different place on earth (so, a different timezone), then that should be set as well, at least to a reasonable default. In PHP using the DateTimeImmutable type is highly preferable, to avoid any calculations from affecting something else unexpected. If a date doesn't need a time attached to it - then make it a more specialised Date object that doesn't even _have_ a time.

DateTimes, with a timezone, are a simple example of a wider idea - compound types - where a collection of data goes together naturally. Another fairly simple example - an address. The 2nd line of an address is essentially meaningless without the rest, and the zipcode or postal code is also pretty plainly part of the same group, so keep it together in the same object to store data.

#### The Values

Some (including myself) would also advocate for going further than most to even avoid potentially any 'simple scalars', strings, boolean or integers. Back to '20240109' - what if it was part of this object?

```php
class MeterReading
{
  public function __construct(
    MeterIdentifier $identifier,   // eg (MeterType::Electric, 'electric_company_meter_id')
    DateTimeImmutable $whenRecorded,  // Jan 2nd 2024, 3:14:15pm
    int $readingValue,  // 20240109
  {
    // .....
  }
}
```

So the value `20240109` is now plain as to what it means in this instance - not some kind of a date, but the amount of electricity that has been recorded on a specific meter at a given date & time.  We are using code and the structure of the data to help us show exactly what these values mean, and also grouping the commonly collected (and relevant data) together.  If you do worry that `20240109` is a mistake, and it is *actually* a date - then validate it. Is today the 9th of January 2024 (or was it very recently)? Even better - does it make sense compared to previous meter readings?

There's a strong case to be made, while we are here, to strongly type everything - not even having an integer `$readingValue` from a meter for example, instead embedding that number into its own object. Better examples could be an email-address, or a username - Being able to explicitly create a `new EmailAddress('johndoe@example.com')` allows for embedded checks - is it a valid email address? Are `@example.com` hostnames even allowed? Dealing with something that can **only** be a _known to be valid_ `EmailAddress` does make it very clear what it is supposed to be. Ultimately, things will inevitably be a string or an integer, floating-point-number or a boolean, but that higher level of abstraction - a more specific type (and compound types, like a <nobr>`MeterReading`</nobr>) helps clarify exactly what information you are dealing with at any time. I'm still waiting to work on an all-object, no 'naked' scalars system, in the meantime, I do keep hoping.

### No easy answers

The hard part will always be that initial conversion to a well defined structure of objects. If the data being sent to you is not too complex, but is well defined (and is being sent to a URL/endpoint), and you are using advanced frameworks like Symfony, then tools such as [MapRequestPayload](https://symfony.com/doc/current/controller.html#mapping-request-payload) (or the simpler [MapQueryString](https://symfony.com/doc/current/controller.html#mapping-the-whole-query-string)) will help a great deal. However it's usually a great deal more complicated, or we need to fetch and decode the data ourselves. As usual, existing libraries, or at least good documention, can make things easier, with pre-built objects that are filled by a library or a well-defined payload that allow for automatically generating such objects from an OpenAPI specification  ... but I've rarely seen good examples of that.

So, that initial conversion, and creating the underlying set of classes/objects to represent the data will generally be down to you. All I can offer is some suggestions for tools to perform the bulk of the generic conversions.

* Symfony has its [Serializer Component](https://symfony.com/doc/current/components/serializer.html), along with the [validator](https://symfony.com/doc/current/validation.html). Both take an class-defination, with the property types defined alongside extra validation attributes, as well as the ability for more complex checks that can be added, or explicitly coded.
* Crell has also written a similar tool - '[Robust Serde (serialization/deserialization) library for PHP 8](https://github.com/Crell/Serde)', with his presentation at Longhorn PHP 2023 [Serializing PHP with Crell/Serde](https://www.youtube.com/watch?v=GB5Vfi68ToY) marking the first public release of it.

### Some final thoughts

* Don't be afraid to create new classes/objects - but use them consistently
* Complex objects should be built of simpler objects
* Composing and extending classes to build a structure can both be valuable
* Convert simple associative data to strong object types
* Merge related information that will be used together to a compound type
* Pass objects around - not associative arrays
* An array can still have a place, for zero or more of the same type of specific object

Strong, well-defined types, are the foundation of a good software system. When you have that in place, the structure that is built on them will be stronger, and more easily understandable.


Alister Bulman.

----

Extra reading:

* Nikita Popov: [Why objects (usually) use less memory than arrays in PHP](https://gist.github.com/nikic/5015323) - PHP 5.4 and newer
* Tomas Votruba: [3 Signs Your Project is Becoming Legacy - Arrays Creep](https://tomasvotruba.com/blog/3-signs-your-project-is-becoming-legacy-arrays-creep#rule-of-single-place)
* thecodingmachine [TCM Best practices](https://bestpractices.thecodingmachine.com/php/organize_your_code.html#services-should-not-return-encoded-data) - Services should not return encoded data

<!--
* Benjamin Eberlei - Doctrine ORM and NoSQL, 2016 [Avoiding 'array soup' from Doctrine ORM from a JSON custom type](https://youtu.be/E8w1y1Jo7YI?feature=shared&t=2147)
 * https://github.com/dunglas/doctrine-json-odm
-->
