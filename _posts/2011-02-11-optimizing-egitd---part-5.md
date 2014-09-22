---
layout: post
title: "Optimizing egitd  - Part 5"
description: ""
category: programming
tags: [erlang, egitd]
---
{% include JB/setup %}

Alright, I'm just going to fix some miscellaneous stuff in egit that bother me. First up is the build system and the project layout. Rake is great and all, but it introduces a dependency on Ruby which isn't really necessary. Erlang has several native build systems but I prefer [rebar](https://github.com/basho/rebar). Rebar has its flaws but its probably the most capable build system for erlang at this point. So now, to compile egitd instead of running 'rake', you run 'make' (I added a really simple Makefile to wrap rebar in), or './rebar compile'. Commit is [here](https://github.com/Vagabond/egitd/commit/13f2993ceee691307324cf88985ca33a42906b0d).

The next thing I didn't like that caught my eye was the naming of some of the files, 'server.erl', 'conf.erl', 'log.erl', these are just asking to cause a clash. So, I [renamed a bunch of things](https://github.com/Vagabond/egitd/commit/004ae965d69771fb3aeedbee262cb98b48f0b607) around and fixed the references to them. I left log.erl and md5.erl alone, since I need to figure out if I even want to keep them (log.erl is used to log precisely 1 message in the entire codebase).

I also wanted to rework egitd_server, the socket accept() loop as a OTP behaviour, but short of resorting to the prim_inet:async_accept trick (an undocumented function that's not guaranteed to not be randomly removed) there's not a clean way to do it. [gen_nb_server](https://github.com/kevsmith/gen_nb_server) does look pretty nice, though. OTP team, if you read this, please consider making a documented and supported way of doing async accept in Erlang.

What egitd_server does is it uses [proc_lib:spawn_link](http://erldocs.com/R14B01/stdlib/proc_lib.html?i=12&search=proc_li#spawn_link/3) to start the process and then [proc_lib:init_ack](http://erldocs.com/R14B01/stdlib/proc_lib.html?i=4&search=proc_li#init_ack/2) to return control to the parent process before the init() function returns. This means that from the end of init, it call call into its own event loop in which it constantly calls accept() and blocks waiting for a connection. Its not ideal because you can't do stuff like hot code reloading or really have the process do *anything* other than accept() but that's acceptable. So, after looking at it, I think I'm going to mostly leave this code alone.

The next thing I'm going to do is feed the codebase through [tidier](http://tidier.softlab.ntua.gr/mediawiki/index.php/Main_Page), which is a nice online tool for refactoring that is provided free for open-source erlang projects. You can tar.gz all your erlang files and upload the whole thing and it'll give you suggestions on making your code prettier and in some cases faster, too. In the case of egitd, it didn't really complain about anything but a single call to lists:append. Its purely cosmetic, but I fixed it [anyway](https://github.com/Vagabond/egitd/commit/dcbd2259b18524549626dc790b686bbbce6490cb). Often tidier will have more suggestions but since most of the remaining egitd code is so simple, it didn't find a lot to complain about.

Then I got sick of the hardcoded error messages that didn't include the actual information submitted so I wrote a [little function](https://github.com/Vagabond/egitd/commit/d3b8a83e4eaa0d496546b52322128cbbac2e7dd5) that puts the 4 byte hex-length header on the message.

I'm going to call this done now, since there's not a lot more I really think needs to be done. egitd is now fast, small and (fairly) readable now. I've updated the README with a link to these rewrite notes and I'm going post this to erlang-questions so hopefully someone can learn from this.

