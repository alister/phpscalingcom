---
title: 'Recently....'
date: Tue, 03 Apr 2012 11:03:33 +0000
draft: false
tags: ['advanced', 'contracting', 'funemployment', 'linux', 'puppet', 'tools']
---

It's been one of those quiet spots around here for a while, so here's the catch-up on what has been happening while I was not posting.

I've recently finished a short contract working with an agency, Transform (part of the Engine group) working with a couple of government departments. The Office Of The Public Guardian receives, checks and stores Lasting Powers of Attorney - a legal document that you write while still mentally compentant to say what you would like to happen should the worst occur, and by whom you want to do it. The simpler cases aren't actually very complicated, but there is a lot of work to get the form completed - and the information can have to be written in triplicate over two or three different forms.

The project was to work with (and at the offices of) the new Government Digital Services (GDS) who are building the [http://Gov.uk](http://Gov.uk) project, and I helped write the first-draft (but otherwise basically complete) prototype to put the form online. If nothing else, it allows someone to step through and only have to put information in once. Myself and one other developer, with a project manager and many others from the OPG and GDS took nearly all of the 37-pages of duplicated paper forms and created a PHP/Zend Form based system, that in the end produced PDFs, ready to check and then have signed by everyone involved.

It was an interesting project - and it will be a valuable service to make it easier to handle, and eventually also process from the back-office perspective. It's not quite what I would normally do, I'm far more infrastructure and back-end oriented - not so used to building a large and complex flowing form, and so I elected to move on at the end of the prototype, rather than continue with the alpha/beta phases. With a little luck, it will go live later this year after some extensive user-testing to make it as useful, and easy as possible to fill in this important legal document. The end result, in a few years, should be the ability to have many times more people making an LPA for themselves, often as part as something as routine as making a will, or buying a new house.

Rest assured, there's been plenty of relaxing since I finished that project a couple of weeks ago (especially since I spend a good portion of the time working while distinctly under the weather).

### Puppet and Github

In the last couple of weeks, I've been a software developing machine. I've also been looking for my next contract role, hopefully something to start just after Easter - though, as I write this, exactly what that will be is still up in the air.

First, I've been putting together a development VM - currently based on a beta of Ubuntu 12.04. There are a few things that go into making it.

Puppet config [https://github.com/alister/puppet-ab](https://github.com/alister/puppet-ab) I've been occasionally tweaking this since before Christmas when I took some time to do a deeper dive into Puppet, after using it last year. Many of the modules I use, are actually pulled in from other github-based projects, especially from a number by [https://github.com/saz](https://github.com/saz) (Steffen Zieger).

[Puppet-dropbox](https://github.com/alister/puppet-dropbox): (forked from [https://github.com/cwarden/puppet-dropbox](https://github.com/cwarden/puppet-dropbox)) Dropbox is a very useful tool on any desktop to copy files around your own machines, and shared folders enable easy access to others working on the same project - I found it nearly invaluable in my last contract. It was also good to be able to improve the code that was upstream - a small fix to allow for Ubuntu to be also set as a destination for the installation.

In the end, I have elected to use it to just install the basic command line tool (rather than the full client), and then that can be used to install the main client, if required. It saves having to store the username and password in the repository, and it is also useful from other security standpoints not having copies of your files on all machines where the puppet manifests might be run.

[Dotfiles](https://github.com/alister/dotfiles) is more of a meta-project, many appear to have a repo by that name, but a large number of them are hand-rolled. I forked one of the more common bases by Ryan Bates [https://github.com/ryanb/dotfiles](https://github.com/ryanb/dotfiles) who also runs the excellent [http://railscasts.com/](http://railscasts.com/) (which are not all about Ruby On Rails). I have yet to find a good way to integrate it with another shell-oriented project - Oh-My-Zsh, [https://github.com/robbyrussell/oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) which is an excellent improvement over a standard Bash shell, that I've been using for more than a dozend years.

The common thread between both of these is to take a basic machine, with git, an SSH key and Puppet installed and bring it quickly up to a full spec development desktop/server. It's a continuing project, but a valuable one, and not just as a learning tool.

There are two other projects that I've been working on.

[guard-puppet-lint](https://github.com/alister/guard-puppet-lint): Guard (see the railscast episode [http://railscasts.com/episodes/264-guard](http://railscasts.com/episodes/264-guard)) is a ruby-based project that will watch for file changes in a subdirectory hierarchy. There are a lot of plug-ins for it [https://rubygems.org/search?utf8=%E2%9C%93&query=guard-](http://railscasts.com/episodes/264-guard) including PHP-oriented ones for PHPunit and PHP\_CodeSniffer. The project itself can be downloaded from [https://rubygems.org/gems/guard-puppet-lint](https://rubygems.org/gems/guard-puppet-lint).

As the name suggests, this small ruby gem adds a slightly easier way to run Puppet-Lint through Guard. As my first released ruby-code, there's not much to it, and in fact, it's really just a hack of guard-shell, that will run puppet-lint on the changed manifests. it does make it slightly cleaner though, and so I'm happy enough. I'm also very pleased to have had a couple of (very minor) issues raised - literally one word missing from the readme file and a single character to reduce the number of false-positive files that might be processed. There are some ideas I can add to it to make it even more useful, but that can wait for a little while, and besides, I have to figure out how to better use Guard in the first place, to be able to to do so.

My final, and latest, project is [https://github.com/alister/QR-Generator-PHP](https://github.com/alister/QR-Generator-PHP) - a refactoring of the QR code library at [https://github.com/edent/QR-Generator-PHP](https://github.com/edent/QR-Generator-PHP).

The code itself works fine, but only as a URL destination. One of the ideas that came up while working with the Office Of The Public Guardian on their new LPA form was to put QR codes onto the final output PDF pages to help verify automatically that all the pages that have been produced, have been recieved at the back office - and also being able to refer the paper form to a digital version stored in the database.

It's a classic refactoring though - taking a piece of code and, without changing the end results, make it possible to use in a slightly different, but useful, context. Eventually, the qr.php webpage would be a thin wrapper around the class - and the class itself could be used from backend code to, for example, generate an image that can be placed into a PDF.