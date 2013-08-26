---
layout: post
title: "Packaging and the tide of history"
description: ""
category: Rants
tags: []
---
{% include JB/setup %}

A quick follow up to my
[previous post](http://vagabond.github.io/2013/06/21/z_packagers-dont-know-best/)
because I forgot to mention some things as part of the conclusion (it was 5am,
it happens).

The observation I wanted to make was, that developers are *already* rejecting
the kind of packaging principles that package maintainers cling to. Ruby has
[bundler](http://gembundler.com/), node has (well, a bunch of things, [npm
shrinkwrap]((https://npmjs.org/doc/shrinkwrap.html) seems to be the
latest hotness). There's even this thing called [docker](http://www.docker.io/)
that lets you build a whole mini environment with tailored versions of anything,
for deploying polyglot applications. I could probably find more examples. The
point is, all this stuff has emerged in the past few years (with the exception
of erlang releases, which have been around for a long time, but have recently
come into vogue).

I think this trend reflects the explosion in the open source ecosystem; there's
libraries for everything now. The problem is, most of these libraries are
maintained by different people with varying levels of experience, knowledge
about compatability issues and ideas of versioning (not to mention testing
methodology). I regularly see backwards incompatible changes pushed in minor
releases, [semantic versioning](http://semver.org) be damned, and that's fine.
If the project's code is solid, I'm happy to let the maintainer run it their
own way. Even Riak isn't terribly good at this, some of our libraries are
semver, some are versioned for marketing reasons (riak 1.0 sells better than
riak 0.15). Also remember, that in this era of github, lots of good libraries
don't even *do* versioning (at least not in their early stages).

However, this shift to many small, independently maintained libraries means that
the old approach of installing a library as its own package becomes increasingly
complicated and failure prone. A common library being bumped now means that all
the packages that depend on it need to be re-verified and checked for subtle
breakage. Back in the day, the gAIM developers refused to accept bugreports from
Gentoo users because of the packaging changes Gentoo made.

Another parallel is to look at operating system kernels and the userland. Many
operating systems ship with a 'world' which is a small bare minimum set of
applications to provide a useful environment. Some 'world' installs are larger
than others (OSX is particularly bloated, bundling things like stale versions of
ruby, which impact applications needing a newer version). For operating systems
with a reasonable policy on what is included in the world, the kernel and the
world can be upgraded in lockstep. The BSDs are a particularly good example of
this, they provide a minimal set of useful things and then provide package
management on top of it. Many linux distributions provide a smaller set of
essential packages, so they have the risk that updating one core dependency can
break everything. I remember all too well breaking my Gentoo install by
upgrading libstdc++ and breaking gentoo's 'emerge' tool, which was written in
python (this is really fun to fix). The BSDs usually provide a compiler and a
libc as part of the world, so that kind of breakage is very hard to do by
accident (of course, other compilers are often available via the package
manager).

Now, I'm not saying that a rails application should bundle a postgres install
(but maybe it could, if you had good reason) but that the idea that libraries
can be easily shared between applications in this modern era of large, fast
moving, differently maintained library ecosystems is kind of a fallacy. Maybe
this is some manifestation of [the tragedy of the
commons](http://en.wikipedia.org/wiki/Tragedy_of_the_commons), but it is still
the world we live in and our packaging should reflect that, not ignore it.

So, package managers, take note of what developers are doing and try to think of
ways to adapt, lest you find yourselves on the wrong side of history (and having
us reject all the bugreports from your packages).

As some further reading, check out Jared's
[slides](https://speakerdeck.com/jaredmorrow/packaging-erlang-applications)
on node_package, the tool we use at Basho to package erlang releases as
operating system packages (for 6 different platforms, no less). This is the
future of packaging, I believe, where the package contains the library ecosystem
needed to run the application as the maintainer has intended (and QAed). I know
it might use more disk space, but storage is cheap, and compromising reliability
for a few megabytes on disk is crazy.
