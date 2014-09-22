---
layout: post
title: "Optimizing egitd  - Introduction"
description: ""
category: programming
tags: [erlang, egitd]
---
{% include JB/setup %}

I was thinking the other night about [egitd](https://github.com/mojombo/egitd), the erlang git-daemon that github
wrote because they didn't like the one included with git. They had some neat
stuff like pattern matching the URLs to repo paths, better error messages and
better logging. It all sounded really cool back in mid-2008 when it was
[announced](https://github.com/blog/112-supercharged-git-daemon), they even deployed it for a while but then I never heard any more
about it.

So, I looked it up. It turns out that they had to abandon it because of
performance issues:

> This software was in production use at github.com for a short time until it
> became obvious that the communications model was flawed. To be specific, if the
> upload-pack takes a long time to respond (for big repos), either the timeouts
> have to be increased to unreasonable values (slowing the entire transfer down),
> or some connections will timeout and fail.

Well, that's not so cool. I didn't really see why Erlang wasn't suitable for
this task so I glanced over the code (very briefly). I saw a fair amount of
scope for optimization and I decided to see what the problems with egitd were
and if they could be solved. The main reasons I'd like to do this are:

 * Prove that Erlang was suitable for this task
 * Illustrate some Erlang best-practices
 * Document how to optimize an Erlang project
 * Maybe learn some more tricks along the way

Things I'm  *not* trying to do:

 * Make mojombo and/or github look bad
 * Advocate anyone actually *use* egitd

egitd is just a good example of an Erlang codebase that has some problems and I
have no familiarity with. I learned a lot doing optimization on [gen_smtp](https://github.com/Vagabond/gen_smtp) and I
didn't think to document that knowledge at the time, hopefully this time around
I can.

I plan to try to write a series of articles where I explore the egitd codebase
and explain what I'm fixing and why, I have no idea how long it'll take or
when/if it'll be done. I'm not even sure what exactly the 'upload-pack' problem
is, but I guess I'll be done when I can understand what the root issue was and
if/how it can be fixed.
