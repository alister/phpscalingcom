---
title: 'Investigating RedisGraph'
date: Thu, 06 Dec 2018 13:31:44 +0000
draft: false
tags: ['advanced', 'graphdb', 'redis', 'tools']
---

<img src="/images/RedisGraph-logo.png" title="RedisGraph logo" align="right"> Thus far, I've not done anything serious with much more with database like Mysql, some Postgres and on the NoSql side MongoDB (with a frisson of some use of Redis for some barely-more-than basic things), but I saw some mention of using [RedisGraph PHP Client](https://github.com/kjdev/php-redis-graph) back in late October, as part of my regular scan of the [packagist feed](https://packagist.org/feeds/) for new PHP/Composer packages.

The ['kjdev/redis-graph'](https://packagist.org/packages/kjdev/redis-graph) package is the first example of an interface library to [RedisGraph](https://github.com/RedisLabsModules/RedisGraph) - an [extension module to Redis](https://redislabs.com/community/redis-modules-hub/) that became possible with Redis 4.0's release. Other modules now include Bloom filters, rate-limiting and a JSON type.

This caught my interest. I've been a fan and user of Redis for several years, and it's now a go-to tool that I use for most of my caching needs. I've not used it as a 'serious' database yet though, considering it more as a transient cache. That opinion is changing fast though.

I was curious about a few things:

1.  How fast to run a useful query?
2.  How fast is it to insert new (bulk) data?
3.  How much space does data take up?

Getting RedisGraph running / Why its different to other GraphDBs
----------------------------------------------------------------

The first part was easy enough. Although I didn't have a version of Redis at that time (a couple of weeks ago) that could use Redis Modules, they have a [Docker image](https://oss.redislabs.com/redisgraph/#docker) that can easily be used for testing.

```
docker run -p 6380:6379 -it --rm redislabs/redisgraph
```

As I already had an instance of Redis-server running, I'm exporting this new version on a different port - port 6380 outside of Docker, from the usual 6379 inside. That allowed me to keep running my current (old) version, but try the new version with `redis-cli -p 6380`. You can [try the same examples](https://oss.redislabs.com/redisgraph/#give-it-a-try) and get a feel for it.

![Example sparse matrix.](https://wikimedia.org/api/rest_v1/media/math/render/svg/80a67bed613b4fe0f0d8ee542e8a599c288d5d0c "Sparsity is 74%, and its density is 26%") A (small) part of the reason I'd not tried much with GraphDBs is because I heard that they can be slow - but that RedisGraph used a set of new techniques that were previously far more difficult. A sparse matrix is a data-structure that could have thousands of items in a grid, but only a small number of the intersections between them might be used. A naive storage mechanism for thousands of possible points could take up hundreds of megabytes of RAM - far from ideal when most of the points that could be connected, are not.

RedisGraph uses a sparse matrix for each possible combination meaning that the simple query of finding out that a connection (an 'edge' in graph-parlance) has been made between items (a 'node'). Each node, and edge can also have data associated between them - not just a simple name, but a group of properties - any of which can be used to search for the node, or edge (item/record, or connection). Even more interestingly, those matrices can have various mathematical operations performed on them to calculate some very complex potential relationships. This avoids entirely the need to search and traverse the nodes to be able to make queries.

Pushing data
------------

So - I set about writing '[Caxton](https://github.com/alister/caxton)' as a personal test and a basic example of creating and querying a large number of items in a graph. This is deliberately simple for now - just a set of people, with some randomly generated data attached ('username', 'full name' and a couple of dates, stored as an integer), that can be linked to another user. The links also have a date ('createdAt' time, and a simple key:value).

> The project is online at github - [https://github.com/alister/caxton](https://github.com/alister/caxton) written in PHP, and as a simple console command. \`bin/console app:create-graph 750 --prime=15\`. It is written as a single simple class, called from a Symfony/Console command, in a small Symfony application and using the [kjdev/redis-graph](https://packagist.org/packages/kjdev/redis-graph) library to talk to the Redis Module.

`buildPersons` is called to create as many 'node' items as is needed - I call each one a 'Person'. An internal loop creates a single Node, and then '[yield](http://php.net/manual/en/language.generators.syntax.php#control-structures.yield)'s it to the outside loop, where it's stored in an array for now.

{{< gist alister 3ecc9d8483b8b3a85b0cde8f04761508 SampleGraphBuilder.BuildPersons.php >}}

Each node added is passed to the library to store, and then is committed to the database in groups of up to 128. The first 1% of the users are marked as a little special - they can connect to the rest. I pick a random number of the other 'Persons' (up to 2,500), and then they are linked:

```
MATCH (r:Person),(c:Person) WHERE r.username = '{$username}' AND c.username = '{$linkToPerson}'
CREATE (r)-\[:link\]->(c)
# Adding an index on 'username' speeds up this search immensely.
```

Where I create the `[link]` clause, I also add some other properties. I'm not currently using it, but with them in place, I could ask later (for example) 'When was Person r, connected to person c?', or search for links dated before, or after a particular time.

> The variables inside the query are having to be interpolated in place, and this does sadden me somewhat from a security point of view, and to a lesser extent, ease of use. As Neo4j has a great deal more time of development, for some years it has had the ability to use parameterised queries - and that also helps for an impressive speed boost. I hope that RedisGraph could add them as well in the future (though I wonder if an optional binary format would have to be added to enable it with the mostly-text-oriented Redis commands). It has also one of the techniques that makes SQL injection in PHP, or any other language, much more difficult to even allow to happen accidentally.

```
\# bin/console app:create-graph 75000 --prime=150

\> Building person nodes.
>  75000/75000 \[============================\] 100%
> Build connections.
>  104832/196641 \[==============>-------------\]  53%
```

Running queries
---------------

Finding the number of links to each user (even with over 195,000 such links) is surprisingly fast - 1411.8ms, just under 1.5 seconds.

```
$query = 'MATCH (r:Person)-\[x:link\]->(:Person) RETURN r.username,COUNT(x)';
$result = $graph->query($query);
```

Total number of links

| r.username           | COUNT(x)    |
| :--------------------| ----------: |
| alexandre44          | 1679.000000 |
| alejandra68          | 172.000000  |
| alison37             | 1302.000000 |
| alivia.oconner       | 1300.000000 |
| alverta32            | 2162.000000 |
| amalia.orn           | 306.000000  |
| astehr               | 809.000000  |
| avis.hand            | 2121.000000 |
| annabell94           | 806.000000  |
| annetta08            | 157.000000  |
| acole                | 690.000000  |
| etc... |

Finding all the links from a particular person took just 20.9ms in this example - including retrieving and parsing the results to something that can be easily displayed, or used.

```
MATCH (r:Person)-\[:link\]->(c:Person) WHERE r.username = '{$username}'
RETURN r.username,c.username,c.name,c.setDate";
```

All links from cartwright.malika

| r.username        | c.username    | c.name          | c.setDate         |
| ------------------|---------------|-----------------|:------------------|
| cartwright.malika | oherman       | Gerard Haley    | 0.000000          |
| cartwright.malika | arch54        | Frederic Koss   | 0.000000          |
| cartwright.malika | boyle.glennie | Aliza Anderson  | 1530409683.000000 |
| cartwright.malika | malvina11     | Shanie Lindgren | 0.000000          |
| cartwright.malika | abigail42     | Cleta Lind MD   | 1484453700.000000 |
| etc....           |

Even with a more complex search, (between two dates, stored as INTs) and an `ORDER BY`, returning 273 results out of 901 links from this user, the total time to search and return the data is just 5.4ms, though the library also reports an internal time taken of just over 3.85ms.

```
$today = 1544031124; // epoch, Dec 05 2018 17:32:04
$query = "MATCH (r:Person)-\[:link\]->(c:Person)
    WHERE (r.username='{$username}')
        AND (c.setDate > 0) AND (c.setDate < {$today})
    RETURN r.username,c.username,setDate,c.updatedAt
    ORDER BY r.username,c.username,c.setDate";

| r.username | c.username     | c.setDate         | c.updatedAt       |
|------------|----------------| :-----------------| :-----------------|
| ibernhard  | adriel.gleason | 1524731283.000000 | 1526763937.000000 |
| ibernhard  | aheidenreich   | 1525008374.000000 | 1496206048.000000 |
| ibernhard  | aileen.schumm  | 1503427894.000000 | 1504939394.000000 |
| ibernhard  | alberto90      | 1540786524.000000 | 1487852712.000000 |
| ibernhard  | alexys.stokes  | 1494405825.000000 | 1498136432.000000 |
| ibernhard  | allene.weber   | 1503973246.000000 | 1500810582.000000 |
# etc
```

Memory use
----------

My third question was "How much space does data take up?", and just after the script has run, Redis reports `"used_memory" => 65,966,216` or about ~63MB.'. (My almost empty Redis instance - just after a restart - reports `used_memory:876704`, 856.16K in comparison).

A little more interesting is after the server is restarted with the sample data inserted - all the data is still there, thanks to the regular dumps to disk, but now the server reports a lot less memory used: `used_memory_human:32.82M`.

Speeding up insertion
---------------------

There is no doubt at all that adding indexes to at least the main data-node properties speeds up searching for the data. If you are, as I am at the moment, building all the nodes and then adding links to them, with a quick test of adding ~6,200 connections/edges, the amount of time spent went from 8.8 seconds to 4.65 secs.

*   CREATE INDEX ON :Person(username)'
*   CREATE INDEX ON :Person(setDate)

As with other databases indexes should be used carefully - as if they aren't being regularly used, they will have an overhead, and that means a cost.

There is a bulk-import command, though the format isn't well described and it does have a number of downsides (such as some complexity to setup) and it can't, at least currently, add properties to nodes or edges between them. I'll be trying that soon.

Summary
-------

<img src="/images/RedisGraph-logo.png" title="RedisGraph logo" align="left"> With my tests, RedisGraph looks to be more than fast enough to create useful graph database today, and to query them in some potentially complex ways. Even running on a low-spec home server, it's adding several thousands of nodes, and then connecting them with more than a thousand edges per second. With a higher-spec, more production ready system, I'd expect to see many multiples of that. The limitations are well understood, with currently incomplete support for the Cypher query language, and the simple fact that Redis runs from memory (but can easily save that to disk for persistence).
<!-- [![](https://phpscaling.com/files/2018/12/RedisGraph-logo.png)](https://phpscaling.com/files/2018/12/RedisGraph-logo.png) -->

```

|                        | Counts/Duration |
| -----------------------| ----------------|
| persons built          |     75,000      |
| total 'edges'          |    181,215      |
| build persons duration |     13,848.4 ms |
| build persons per/sec  |      5,415.79   |
| build edges per/sec    |      1,287.92   |
| build-edges duration   |    140,703.5 ms |
| search links duration  |         18.7 ms |


This looks to be a great tool, mixing the flexibility of graph databases, NoSQL and Redis for the in-memory databases for a surprising amount of data in a quite small amount of (RAM) space - while keeping the functionality that Redis is so well known for with data-structures like the hash for 'NoSql' style schema-less data storage and retrieval.

I'm looking forward to using it for some interesting things.
