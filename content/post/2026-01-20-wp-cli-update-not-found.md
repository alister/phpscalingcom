---
title: "Solving 'WP-CLI Updates Not Found'"
date: 2026-01-20T12:41:06Z # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true
draft: false
toc: false
#menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
featureImage: "/logos/wp-cli-logo-inverted.png"
featureImageAlt: 'WP-CLI logo'
#featureImageCap: 'This is the featured image.' # Caption (optional).
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - tips
tags:
  - wordpress
  - wp-cli
comment: false
linenos: table
---

{{% notice info "Comments?" %}}
You can comment on this article on my [Mastodon post](https://hachyderm.io/@alister/115927510278696934) for "Solving 'WP-CLI Updates Not Found'"
{{% /notice %}}


That's one (slightly annoying) problem solved - I've managed to upgrade the WordPress version and plugins on <https://abulman.co.uk/> on the second attempt.

I use [WP-CLI](https://wp-cli.org/) to do it all from the command line (and then commit all the changes into Git), but when I last tried to update, it had been reporting that everything was already up to date - even though the dashboard was lit up like an Xmas tree with new version notifications.

```bash
$ wp core check-update
Success: WordPress is at the latest version.
$ wp theme update --all --dry-run
No theme updates available.
$ wp plugin update --all --dry-run
No plugin updates available.
```

Initially, I forced a version update with `wp core update --version=6.9 --force`, but wanted to find the general solution. Searching found a couple of posts saying that security or caching plugins could be the issue, so I ended up making a quick edit in the W3TC drop-in plugin file at `wp-content/object-cache.php`, just adding a `return;` at the top so it wasn't used.  That solved it, and I could upgrade all the themes and plugins, before removing the line again.

As that WordPress site is the only thing I've got on the server, I'll take another look around for a good looking theme for Hugo to replace it with a Github pages version and setup all the redirects in [LuaDNS](https://www.luadns.com/) to send them there. Or maybe I can just add some things to my Github front-page at <https://github.com/alister> instead?  I'll have a think and see what I can do.
