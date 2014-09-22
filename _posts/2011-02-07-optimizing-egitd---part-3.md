---
layout: post
title: "Optimizing egitd  - Part 3"
description: ""
category: programming
tags: [erlang, egitd]
---
{% include JB/setup %}

Alright, time to do some benchmarking against git-daemon itself. This time we're cloning the linux-kernel repo, which is ~500mb or so, the largest public git repo I'm aware of.

To run git-daemon for this test I used this command:

    git daemon --verbose --base-path=/Users/andrew/egitd-repos

classic egitd:

    git clone git://localhost/linux-2.6.git  105.97s user 19.21s system 18% cpu 11:00.86 total
    git clone git://localhost/linux-2.6.git  106.01s user 19.07s system 19% cpu 10:53.45 total
    git clone git://localhost/linux-2.6.git  104.69s user 18.98s system 18% cpu 11:03.95 total

new egitd:

    git clone git://localhost/linux-2.6.git  105.25s user 16.39s system 68% cpu 2:58.54 total
    git clone git://localhost/linux-2.6.git  104.35s user 15.81s system 72% cpu 2:46.85 total
    git clone git://localhost/linux-2.6.git  104.49s user 15.92s system 71% cpu 2:48.21 total

git-daemon:

    git clone git://localhost/linux-2.6.git  101.49s user 14.86s system 71% cpu 2:42.34 total
    git clone git://localhost/linux-2.6.git  101.01s user 14.80s system 70% cpu 2:45.08 total
    git clone git://localhost/linux-2.6.git  103.82s user 15.48s system 71% cpu 2:46.59 total

So old egitd takes 11 minutes, new egitd is at 2:50 or so and git-daemon is at 2:45. So egitd is now comparable in speed to git-daemon, rather than being ~3.5x slower.

The next thing to test is lots of simultaneous clones to see how things compare there. I think I'm going to stop benchmarking the old egitd, it just takes too damn long to do anything.

