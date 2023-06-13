---
title: 'A useful idea for helping to enforce PHP code standards'
date: Tue, 11 Mar 2008 17:16:57 +0000
draft: false
tags: ['php', 'PHP_CodeSniffer', 'standards']
---

[Extending PHP\_CodeSniffer](http://raphaelstolt.blogspot.com/2008/03/sniffing-refactoring-needs.html) by Raphael Stolt shows how to quite easily add to a tool that will report what parts of your PHP source needs a clean-up, from the built in 'sniffs' for coding standards, and now adding to that for some slightly more opinionated choices on the maximum number of lines per function, or functions per class.

This could be a start of a whole collection of additional classes for additional checks.

The only challenge I see at the moment (and I don't think it's a big one, though I've not tried it, so it might not even exist, I've just not had the opportunity to look yet) is having to put code into the PEAR directory path, since that is where PHP\_CodeSniffer is looking for the base library. I do see that they use long PEAR class-names, so it may just a matter of having the phpcs tool look elsewhere for the base coding style class.

**[edit: 18:06]** Ahah, some investigation - and copious use of echos in the phpcs script, and /usr/share/php/PHP/CodeSniffer.php class:

Creating your own standards, and using them from anywhere on the filesystem - not having to link from inside the PEAR/PHP/CodeSniffer directory:

The coding standard class with your extension: The class is called PHP\_CodeSniffer\_Standards\_Example\_ExampleCodingStandard - the two <em>'Example'</em>s being the name of the coding standard. The file is called 'Example/ExampleCodingStandard.php'.

{{< gist alister 1385932 >}}

And in the subdir: .../Example/Standards/ - Raphael's ToManyMethodsSniff.php and MethodLengthSniff.php files from his post.

To call it:

{{< gist alister 1385936 >}}
