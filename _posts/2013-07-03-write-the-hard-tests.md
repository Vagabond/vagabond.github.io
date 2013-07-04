---
layout: post
title: "Write the hard tests"
description: ""
category: 
tags: []
---
{% include JB/setup %}

I just saw the post [Your test suite is tring to tell you
something](http://blog.jgc.org/2013/07/your-test-suite-is-trying-to-tell-you.html)
on Hacker News
and it eerily echoed my own experience, so I wanted to throw in some
war-stories of my own.

At Basho, we try to value release quality over release quantity. We've slipped
releases, sometimes by months, to resolve issues we felt were too serious to
ignore. As an attempt to improve our release times, we've been trying to write
some better tests, specifically using [EQC](http://www.quviq.com/) (which I
cannot recommend enough - they have great software and a great team), our own
home-grown [riak-test](https://github.com/basho/riak_test) and that old standby
of EUnit.

Each of these tools is well suited to particular kind of test, EUnit is good for
testing simple, pure functions (although EQC can arguably do it better, if you
can express the function's behaviour as a property), EQC is great for generating
sequences of commands, and reporting when a particular sequence breaks your
expectations. riak_test really shines if you need to test how a riak cluster
behaves, which is a real pain to do from an eunit test (we do have some older
eunit tests that stand up riak nodes, but they're extremely annoying and need to
be rewritten as riak_tests).

Now, the simplest of these tests to write is undoubtedly EUnit (unless you have
to figure out how EUnit test timeouts work) but arguably they're also the least
interesting. A new EUnit test will often expose obvious or expected bugs, the
other two tools often expose *unexpected* bugs, or bugs that don't even look
like bugs initially.

For example, the latest incarnation of Riak Enterprise's Multi-Datacenter
Replication features a nifty multi-consumer bounded queue. This is used to allow
realtime replication to multiple clusters to each be a pointer into a shared
queue. Now, I had written an EQC test for this that tested the queue in
unbounded mode as well as an eunit test that checked that bounded mode worked. I
didn't model trimming in the EQC test because the implementation relied on
calculating ETS overhead, which is not terribly easy to model. Both tests passed
fine.

However, I finally decided to bite the bullet and extend the EQC test to model
trimming (I sort of cheated by #ifdefing a different size calculation function
when the module was compiled for testing). This was kind of a pain, but it
exposed a new bug! Turns out, if a consumer registered, disconnected and then
re-registered AFTER a trim had happened, the sequence ID the consumer would be
given had a chance of being a trimmed entry. This would crash the whole queue
process, dropping all your realtime information. This is the power of EQC, it
will generate test cases you'll never think to test yourself.

There's other ways to hunt bugs too, more reminiscent of the blog post above.
Riak MDC has some [very extensive
riak_tests](https://github.com/basho/riak_test/blob/master/tests/replication2.erl)
which, although ugly, test a LOT of functionality. When I first wrote these
tests, they used to fail a lot. There were race conditions everywhere. For a
while, I just sort of blew the intermittent failure off. I mean if the test
passes most of the time, it must be pretty good, right?

No, it is not good. About once a release, I went on a crusade trying to increase
the reliability of the tests. Often it was just additional checking/waiting in
the test, but occasionally it was a legitimate bugfix, and boy did we find some
nasty bugs. Now, this work can be *exhausting*, running the same test over and
over again waiting for it to fail the same way as it did last time, adding debug
prints to figure out what is happening, etc. I often end up burning myself out
on testing trying to ferret these issues out, but it is absolutely worth it.

The riak_tests still aren't perfect, but they're much *better* and hopefully
they'll continue to improve. I know other people at Basho are being similarly
stubborn about hunting down the source of test failures, whatever the cause, and
it has paid off for them as well.

So, next time you code up a new bit of your software, write that easy unit test,
sure, but try to think outside the box and either have something like EQC
generate test cases for you or code up a big old integration test for it. It
won't be fun, it won't be glamorous, but you'll find the kind of bugs you'd
previously blow off as 'impossible' or 'memory corruption' or 'a bug in the VM'.
Hell, you might even find a bug in the [standard
library](http://erlang.org/pipermail/erlang-questions/2012-September/069039.html)
or in the [testing
framework](http://erlang.org/pipermail/erlang-bugs/2013-May/003601.html).

Also, if you're handed a a bit of important code to maintain, the *best* thing
you can do is try to beef up the tests. You'll gain understanding of the
codebase, you'll probably find bugs, and you'll have a much stronger safety net
when the inevitable urge to do some re(write|factoring) strikes. There is
nothing worse than an ill-informed rewrite that discards the history encoded in
its ancestor.

So, yes, your test suite may well be trying to tell you something but only if
you invest enough time in it (initially and on an ongoing basis).
