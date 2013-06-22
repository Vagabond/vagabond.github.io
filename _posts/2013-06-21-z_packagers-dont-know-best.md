---
layout: post
title: "Packagers don't know best"
description: ""
category: 
---
{% include JB/setup %}

A favorite topic between [Jared](https://github.com/jaredmorrow) and myself at
Basho (right behind how much we hate Solaris) is how package maintainers like to
package Riak.

I just don't get it. They have some kind of OCD that insists that if software
can be split into multiple pieces, it should be, regardless of the impact or the
logic of such a choice. Back when I worked on the
[FreeSWITCH](http://freeswitch.org) project, they had
this problem as well; FreeSWITCH used a TON of 3rd party libraries (the sofia
SIP stack, spidermonkey portaudio, a bunch of codec libraries, etc). They
include these in the tree because often they have custom patches or require
specific versions. These choices are not made lightly. However, everytime
someone volunteered to package FreeSWITCH for $OS_NAME they'd always start
by patching the build system to support pulling in spidermonkey from the package
manager, instead of usiing the in-tree one (which was installed in a custom
prefix that could never pollute the system, hell it may have even been
statically linked).

This invariably caused problems, the versions from the package manager were too
new/old or they were missing the custom patches needed. Yet, people persisted in
the belief that 'one dependency to rule them all' was the way to go.

Fast forward a few years. Now I work at Basho on Riak, and we see the same
mindset at work. We provide binary packages that are self-contained; an erlang
'release' with a erlang virtual machine binary and all the required libraries,
compiled to bytecode, in one tidy package (that again installs to a place that
won't pollute the system). Yet people 'packaging' Riak insist on splitting
things up again, just because they can. It is even more ridiculous in Riak's
case, however, as some of the 'dependencies' Riak has have almost 0 value as
independent packages, they're only split up like that for organizational
reasons. Yet, packagers see these different dependencies, each in their own git
repo, and get that insane gleam in their eye.

Long ago, Riak *was* developed as one enormous erlang application. We changed
that for reusability and organizational reasons, but if we had not, I doubt if
the packagers would have gone in and done it for us. Packagers don't understand
the systems they package, they just seem to pattern-match on obvious boundaries
and that's where they apply the knife.

As an aside, a lot of this is fallout from dynamic linking. Dynamic linking lets
2 programs indicate they want to use library X at runtime, and possibly even
share a copy of X loaded into RAM. This is great if it is 1987 and you have 12mb
of ram and want to run more than 3 xterms, but we don't live in that world
anymore. Dynamic linking is what brought you 'DLL Hell' on Windows (UNIX has the
same problem, too). Because you defer loading the library until execution time,
if the system has upgraded the version of library X (to satisfy shiny new 
application Z), you may or may not encounter a problem.

One often touted benefit of dynamic linking is security, you can upgrade library
X to fix some security hole and all the applications that use it will
automatically gain the security fix the next time they're run (assuming they
still can run). I admit this benefit, but I think that package managers could
work around this if they used static linking (Y depends on X, which has a
security update, rebuild X and then rebuild Y and ship an updated package). If
you don't believe me about the marginal (at best) benefits of dynamic linking,
maybe you'll believe [Rob
Pike](http://harmful.cat-v.org/software/dynamic-linking/).

Anyway, this is effectively the mess that package maintainers impose on the
carefully curated erlang libraries we ship with each Riak release. However, it
gets even better. With C you can link to a *specific* version of a libary, so
you can say your application depends on libfoo-1.0.2, and even if the user
*also* installs libfoo-1.5.7, you'll probably be ok. Erlang has no mechanism for
versioned code loading, you get whatever Erlang finds first in the code path.

This means that if we ship Riak with lager 1.2.2, but the latest upstream
release is 2.0.0 (yes, Riak does not always use the latest version of even some
of the Basho developed libraries) what does the packager do if he also wants to
package some other erlang application that depends on lager 2.0.0 (which is
backwards incompatible with 1.2.2)? Erlang releases handle this natively, this
is the whole point of them, but packagers blithely decide that we're doing it
wrong and don't know how to package our own software and give us a lager package
for our package manager.

We have the same problem with one of our backend libraries, leveldb. Leveldb is
a key/value database originally developed by Google for implementing things like
HTML5's indexeddb feature in Google Chrome. Basho has
invested some serious engineering effort in adapting it as one of the backends
that Riak can be configured to use to store data on disk. Problem is, our
usecase diverges significantly from what Google wants to use it for, so we've
effectively forked it (although we still import upstream changes). This is fine
the way *we* package it, but again, the package maintainer gets that gleam in
their eye and does one of two things; they either import Google's leveldb as a
package, and hack Riak to use that, or they import Basho's leveldb and make that
the system leveldb package. Both of these solutions are bad. Either users get a
broken Riak, or they get a leveldb lib tuned in a suprising way. Who wins here?

And the madness doesn't even stop with applications. Programming languages are
subject to it as well. Look at this [ubuntu erlang
package](http://packages.ubuntu.com/lucid/erlang), it depends on 40 other
packages, as well. That isn't even the worst of it, if you type 'erl' it tells
you to install 'erlang-base', which only has a handful of dependencies, none of
which are any of these erlang libraries! So you get an installed erlang where
the standard library isn't provided as *standard*. This is madness!

Another variant of this is having -dev packages or -man packages which install
the headers or man pages, respectively. I can understand if you're trying to
build an embedded system, but to strip this stuff out by default is crazy. On my
arch linux machine, which does not split development headers or man pages into
other packages, my /usr/include is a whopping 158mb spread across some 16
thousand files. Nowadays that is nothing, even on a SSD, like this machine has.
My man pages are similarly massive, with 76mb spread across another 16 thousand
files. Even if SSDs are $1/Gb this is still ridiculous, since we're barely using
a fifth of that. $0.20 for the life of the machine to deliver software as the
authors intended it? What heresy!

So package maintainers, I know you have your particular package manager's bible
codified in 1992 by some grand old hacker beard, and that's cool. However, that
was twenty years ago, software has changed, hardware has changed and maybe it is
time to think about these choices again. At least grant us, the developers of the
software, the benefit of the doubt. We know how our software works and how it
should be packaged. Honest.

Update: There's a follow up post [here](http://localhost:4000/2013/06/21/zz_packaging-and-the-tide-of-history/) and a suprisingly insightful HN discussion [here](https://news.ycombinator.com/item?id=5920921).
