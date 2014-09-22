---
layout: post
title: "Optimizing egitd  - Part 1"
description: ""
category: programming
tags: [erlang, egitd]
---
{% include JB/setup %}

Alright, here we go. The first thing is to get the code on my machine and get it to run. Since I'm going to be committing my changes, I'm going to go ahead and [fork](https://github.com/Vagabond/egitd) egitd on github.

Now that I have my own copy of egitd to hack on, time to get it on my local machine:

    git clone git://github.com/Vagabond/egitd.git
    cd egitd

So, looking in the folder we just checked out we can see a Rakefile, that means we use rake to compile this project. When I run rake, I get this output (on R14B):

    (in /Users/andrew/egitd)
    cd elibs
    ./reg.erl:821: Warning: list/1 obsolete
    ./server.erl:70: Warning: regexp:match/2: the regexp module is deprecated (will be removed in R15A); use the re module instead
    ./server.erl:78: Warning: regexp:match/2: the regexp module is deprecated (will be removed in R15A); use the re module instead
    ./upload_pack.erl:24: Warning: regexp:match/2: the regexp module is deprecated (will be removed in R15A); use the re module instead

Compile warnings are always a good place to start, but first I want to figure out how to *use* egitd, so I can test it to make sure I don't break stuff when I make changes. We'll come back to these in a few minutes.

The README tells me how to run egitd, it uses a config file to sorta-virtualhost github repos and then use the path information to route to a specific repository. Since I'm just testing, I'll make my own config file that looks like it might work:

    localhost    (.+)    "/Users/andrew/egitd-repos/" ++ Match1.

In theory, this should make git://localhost/myrepo.git clone the repo at /Users/andrew/egitd-repos/myrepo.git. I'm actually going to test with egitd's own repo because I have that handy.

    cd ~/egitd-repos
    git clone --bare git://github.com/Vagabond/egitd.git

I did a bare clone because git-daemon likes to work with bare repos, and I assume egitd does too. Now lets try to actually run egitd and see what happens when we try to clone

    cd ~/egitd
    ./bin/egitd -c egitd.conf -l egitd.log

It spews a lot of [SASL log messages](http://www.erlang.org/doc/apps/sasl/error_logging.html), but everything looks OK. In another terminal lets try to clone from this repo over the git protocol:

    git clone git://localhost/egitd.git
    Cloning into egitd...
    localhost[0: ::1]: errno=Connection refused
    localhost[0: fe80::1%lo0]: errno=Connection refused
    fatal: protocol error: expected sha/ref, got '*********'
    Permission denied. Repository is not public.
    *********'

Well, that didn't go well. Using 'git grep' leads me to this line which leads me to believe, from the comment right before the function, that I need some sort of magic file in the repo to tell egitd that it is allowed to serve this repo to me. So I try:

    touch ~/egitd-repos/egitd.git/git-daemon-export-ok

And voila, I can clone! So I know that egitd works, at least. Now we can actually start looking at the codebase a little. The obvious place to start is on the compile warnings. They were caused by an obsolete guard and the use of the old, deprecated, regexp module. The re module is the replacement and instead of being in pure-erlang is a wrapper for PCRE. You can see the changes I made to eliminate the warnings [here](https://github.com/Vagabond/egitd/commit/98b443a41f3d2043f8a05b94382012019d53535b).

Now, if you're following along at home, you may have seen an error about 'read socket timeout' in your egitd shell. I dug into the code a little and found it was in uploads_pack.erl. With some more digging it looks like this is the issue that github was running into.

The core issue seems to be that the git client sends the server the list of refs it already has and egitd sends this list to [git upload-pack](http://www.kernel.org/pub/software/scm/git/docs/git-upload-pack.html) which generates a packfile containing any missing refs back to the client. upload_pack.erl is opening an [erlang port](http://www.erlang.org/doc/tutorial/c_port.html) to the git command and then basically connecting the client socket to the stdin/out of the erlang port. The problem here is that the code is doing a bunch of synchronous reads on both the port and on the socket. This isn't very erlangish, the erlang way to do this is to let the TCP driver and the port send you messages when there's data waiting on them, and your erlang process can be idle in the meantime. Doing a bunch of blocking receives is just going to slow things down. The offending functions are [send_socket_to_port](https://github.com/Vagabond/egitd/blob/98b443a41f3d2043f8a05b94382012019d53535b/elibs/upload_pack.erl#L110-L121) and [send_port_to_socket](https://github.com/Vagabond/egitd/blob/98b443a41f3d2043f8a05b94382012019d53535b/elibs/upload_pack.erl#L83-L107).

So, the next step is to fix that. The reason that this is even a problem is that egitd wasn't written to [OTP principles](http://www.erlang.org/doc/design_principles/des_princ.html). Using OTP, upload_pack would be an asynchronous gen_server which would receive events both from the port and the socket and proxy them across. We'd also gain better error handling, hot code reloading, etc. You *could* write upload_pack like this without gen_server, but you should have a damn good reason to do so because gen_server has been battle-tested for 20 odd years and is very reliable.

I'm going to go do that now. Once I've got that done, we'll take a look at whether it helped or not.
