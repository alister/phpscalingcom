---
title: 'PHP-in-London list'
date: Tue, 29 Aug 2017 10:27:18 +0000
draft: false
tags: ['php', 'jobs']
---

While Twitter can be _really_ annoying, sometimes it can help to promote some wonderfully simple ideas.

One of these came from [Andrew Woods (@awoods)](https://twitter.com/awoods/status/652204250408161280) - a github repo called [php-in-seattle](https://github.com/andrewwoods/php-in-seattle).  It's a simple idea - just a list of companies around a geographical area that use PHP.

What it can enable is of mutual advantage to the companies, and developers that might be looking for a new job. So, I started a similar list for the London (UK) area.

[https://github.com/alister/php-in-london](https://github.com/alister/php-in-london)

### PHP-In-London, (UK)

There are a lot of companies in London that use PHP, and it is hard to keep track. If you know of any PHP companies with offices in the (Greater) London area (within ~20 miles), add them with a quick pull request (you can also edit directly from within the Github website if you want).

I've also added another list for recruiters that deal with a lot of PHP roles - though as they are somewhat less technically savvy, there are only a couple of entries, one of which is a recruitment startup.

[![Hacktoberfest T-shirt](https://phpscaling.com/files/2017/08/hacktoberfest.300x200.jpg)](https://phpscaling.com/files/2017/08/hacktoberfest.300x200.jpg)Starting the list actually helped to earn me a T-Shirt - for [DigitalOcean Hacktober](https://hacktoberfest.digitalocean.com/) (which I'm actually wearing as I write this post :D).

To help spread the PR and Github Karma, when someone asked to add a new item to the list, I also added them as a full collaborator on the project - allowing them to add more - something that a some of them have already done - like these fine developers.

*   Tristan Bailey, [@tristanbailey](https://github.com/tristanbailey)
*   Ross Motley, [@rossmotley](https://github.com/rossmotley)
*   Scott (Fatbeehive) [@scottfatbeehive](https://github.com/scottfatbeehive)
*   kungfuchris, [@kungfuchris](https://github.com/kungfuchris)
*   Chris Padfield, [@chrispadfield](https://github.com/chrispadfield)
*   Richard George, [@parsingphase](https://github.com/parsingphase)
*   Craig Willis, [@craigwillis85](https://github.com/craigwillis85)

### Awesome_Bot

As a developer, I know well that anything worth doing, is worth testing, so I looked around and realised that there was indeed a way to test the URLs that were in the list. It's called 'Awesome_Bot', and it is mostly aimed at the various 'Awesome ' lists that helped to inspire the list in the first place. Setting it up with Travis isn't hard, and so I've also enabled that.

{{< gist alister 16e206eeed6cdf48c30d6099d1a858d3 ".travis.yml" >}}

{{% notice tip "Updated" %}}
I'm now using [Github Workflows](https://github.com/alister/php-in-london/blob/master/.github/workflows/is_awesome.yml), rather than Travis.
{{% /notice %}}


It has taken a little tweaking of the configuration (even this morning, when I added Travis-CI to the whitelist to allow for redirects of the SVG build status button), but it has also helped to catch a couple of problems.

### HacktoberFest 2017? Don't wait till then, though!

Although HacktoberFest 2017 hasn't been announced yet, if it does happen, I'm aiming to plug the project more in early October - but that doesn't mean you have to wait until then. If you work at a company based in London (or at least easily commutable), take a look, and throw me a pull request! If you've not used Git/Github before, then let me know, and I can walk you through what to do, or worst case, just drop me an email with the relevant details and I'll add them myself on your behalf.

### One final question for you:

I'm considering a wider UK list of companies that use PHP, probably broken down by region and then larger towns/cities. If you think it's a good idea - let me know with a comment, and/or Thumbs-Up (or down) on the issue I've created for the idea.

[Should I create a separate PHP-in-UK repo?](https://github.com/alister/php-in-london/issues/27)
