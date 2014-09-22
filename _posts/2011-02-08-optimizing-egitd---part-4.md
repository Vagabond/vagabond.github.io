---
layout: post
title: "Optimizing egitd  - Part 4"
description: ""
category: programming
tags: [erlang, egitd]
---
{% include JB/setup %}

So, I did some concurrent cloning benchmarks (protip: disable spotlight if you're on OSX if you're benchmarking something using the disk) and it looks like egitd and git-daemon are now pretty much as fast as each other (git-daemon is a tad faster, but not enough that I really care).

So now we're as fast as the competition (took about 4 hours from never having looked at the code before). I'm going to do some housekeeping. There's a lot of files in elibs and a quick bit of git-grepping indicates that pipe.erl isn't used anywhere and reg.erl is used in *one* place. reg.erl seems to be a home-rolled regular expression engine. I don't see any reason to keep it since we have the [re module](http://erldocs.com/R14B01/stdlib/re.html) now, so why use some weird home-rolled pure-erlang one?

Also, I've duplicated all the functionality in upload-pack.erl and receive-pack.erl, so kill those too. Here's the [cleanup commit](https://github.com/Vagabond/egitd/commit/543f6a780a86c1ef51e424f9d1fea169cfe9650c). This cuts the size of the source tree by ~1500 lines to just over 300. That's much more manageable. log.erl is used in like 2 places and md5 isn't very used either, but I'll leave them be for now, at least.

So, I'm running out of things to do a lot faster than I expected. server.erl needs to become a gen_server, but beyond that I'm not really sure what else needs doing. I'm not a big fan of the file layout or the build system, or the lack of unit tests, but its a big improvement over what I started with. I wasn't really aiming to polish egitd into a finished application, just trying to make it fast enough to be a viable git-daemon competitor and prove that erlang wasn't slow.

I'll probably do at least one more post to wrap this up before I move on to something else. Hopefully there was something useful in all this.

