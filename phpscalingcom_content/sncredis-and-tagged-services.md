---
title: 'SncRedis and tagged services'
date: Wed, 27 Sep 2017 14:17:47 +0000
draft: false
tags: ['advanced', 'php', 'symfony']
---

In early 2016, I suggested an addition to the SncRedis-bundle. The project itself is an fully-featured add-on ('bundle') for Symfony framework projects to easily do a number of very useful interface functions between Symfony and the Redis database/cache.  It can, for example, quickly enable all the sessions to be put into a Redis server, and also cache the Doctrine meta-information as well as anything else that the developer would like to cache.

[My additions were to do some relatively simple things](https://github.com/snc/SncRedisBundle/commits?author=alister) - adding tags and aliases to the services it created from the configuration - in effect, really just adding some variables to the internal Symfony environment that could be used elsewhere, if required.

Now, it's time to show you how why I thought it was useful to add, and what you can do with it.

You can do a lot with Redis, and when I'm using it, I prefer to keep things as separate as I can - internal (admin) caches, vs data that would be used on the public website, keeping the website sessions data apart from emails that are waiting to send, and background jobs that are being run.

With Redis, and the SncRedis bundle, it's easy to keep them apart. While databases like MySql and Postgres give databases names, Redis prefers some simplicity - by default, it will create 16 numbered databases (0-15) - although telling the server to use more is just a quick config change away.

Recording which database is being used for which purpose though, it something that usually only happens in the configuration though - and so to show how much was in each numbered database, I kept a list to be able to refer to:

```
sncRedisDBs:
    0: Default
    1: Cache
    2: Admincache
    3: Swiftmail
    4: DoctrineMeta
    5: Sessions
    7: Queue
``` 

That's easy enough to refer to or use, and also to keep up to date since it's right next to the configuration. But, I'm a developer that loves to write code, and hates to keep documentation up to date, so I spent a few hours coding to allow the code to document the configuration.

That's what the Symfony Aliases and tags allow for - it's easy to fetch the data, and get the information about the database from Redis to be shown with what I've named database  - the 'alias' I've set in the configuration, and so now I can use that name to be able to find out more about the contents of the database. [![](https://phpscaling.com/files/2017/09/redis-list-keys.png)](https://phpscaling.com/files/2017/09/redis-list-keys.png)

Here's sample output of a fairly simple Symfony command I've written. If there's no specific parameters of something to look at - it shows the list of databases and their aliases. Alternatively, I can use the name (here, 'default' or Redis dbNum:1) to get an item from the Redis database. Currently, that's just an index into a dumped list - but it's just as easy to get a named entry.

So, how can we get the details of the Redis clients, which refer to the databases inside Redis itself?  There are a few moving pieces:

1.  Add a CompilerPass - this sends the container, to some code that can collect data from it, before it is finished which will remove unused services and information.
2.  In the CompilerPass, find all the services tagged with the 'snc\_redis.client' tag, and store the data required (in this instance, the Redis client, and the 'alias' that it is known as).
3.  Create a service to hold the data that the CompilerPass finds, and be able to retrieve it on demand

Lets go through them step class by class.

1.  Adding the CompilerPass - This is quite easy - we can create and add the relevant class right in the Symfony Framework's core AppKernel.php class. You can also add the CompilerPass from within a bundle configuration, if you have any.

\[gist id="6d149871170cdf0c769fa4f833aeb112" file="AppKernel.php" /\]

1.  The CompilerPass itself finds the tagged services (themselves created in SncRedis from the configuration), and arranges for them to be added to a holding service (the \`AppBundle\\SncRedis\\ClientsList\` class/service). Because each service could have more than one tag attached, we loop the list of potential tags, and have them added to the ClientList.

\[gist id="6d149871170cdf0c769fa4f833aeb112" file="ClientsListPass.php" /\]

1.  Finally, the \`AppBundle\\SncRedis\\ClientsList\` class/service itself. A very simple store and retrieve. We can also get all the data, or just for a specifically named service, if it exists.

\[gist id="6d149871170cdf0c769fa4f833aeb112" file="ClientsList.php" /\]

So - now we have a service that can store the data - how to use it? Get the service (which only then fills the information), get the clients and loop around to collect the parts of the information we need!

\[gist id="6d149871170cdf0c769fa4f833aeb112" file="DisplayRedisClients.php" /\]

Once you have the name of the alias name of the queue, it's simple to use that as part of the name of the snc\_redis client service name, and so collect data about, or from the Redis Database service itself.