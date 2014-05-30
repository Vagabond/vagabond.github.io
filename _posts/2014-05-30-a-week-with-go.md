---
layout: post
title: "A week with Go"
description: ""
category: rants
tags: []
---
{% include JB/setup %}

OK so, I've been working with Go (the programming language from Google) for about a week now, and I have some initial thoughts. Now I'm far from an expert on Go, so if I get something wrong well, it would not be the first time someone was wrong on the internet.

So Go is kind of a better C, it has nice things like type inference:

```
var x int = 5
```

Can, and usuallu should be written as:

```
x := 5
```

That's nice.

The For loop is sort of like a generic C for loop on steroids. The switch statement doesn't have fallthrough, the if statement doesn't need parentheses (and *requires* curly braces, to prevent those stupid braceless oneliners C allows). It has a native hash table, which is handy. These are all nice things.

However, now things start to get a little weird. Function heads are pretty wacky (from a C perspective), really in general type declarations feel 'backwards'. Looking at it objectively they do sort of flow more logically, but it feels like bucking a 50 year trend is a little silly, given all the other borrowed syntax.

Multiple returns are nice (although you could just have tuples and destructuring/pattern matching), closures are handy (although C function pointers usually are good enough, really). I like the Struct/Method stuff better than C++ style insanity. Go doesn't have tail call optimization (as far as I can tell) which is kind of unfortunate. The error/exception handling is kind of annoying, but I guess it works...

Goroutines are neat, although while they are concurrent, their level of parallelism is unclear (GOMAXPROCS seems to deal with goroutines blocked in system calls). Channels, from an Erlang perspective, look a bit dangerous, especially the synchronous aspect of them. Erlang's mailboxes suffer from some opposite problems, though, so maybe I should not pick on channels too much.

Packages seem OK, definitely an improvement over C/C++. I'm not really thrilled with the compiler and the tooling. They work, but some of the error messages are pretty obtuse. I'm also not a convert of the GOPATH stuff, I can't tell if it supposed to be like a virtualenv, and how the heck do you pin something to a particular git sha when using 'go get'? Are reproducible builds even possible? How about a static analyzer? The compiler is evidently not infallible.

Where it got *really* ugly for me is when I found out it was a garbage collected language. I actually enjoy programming in C and I don't mind managing my own memory there. I actually expected Go would be manually memory managed because it aims to be a 'systems' programming lanauge. I had a nasty shock. Then I found out that goroutines don't have any isolation of their memory space, so garbage collection is of the much-maligned 'stop the world' variety. Lame.

Because goroutines don't have isolated memory spaces, that also means that one goroutine crashing takes down the whole system. Now you might say that the compiler makes that unlikely, but I was able to make it happen in my dabbling (the compiler said the code was OK, but it had a runtime error). Not good. If I was writing simple shell commands or single-use programs, that would be fine, but for something like a webserver, yuck. Shouldn't new languages like Go be embracing the multicore era? To an extent it does, but the lack of fault tolerance, for me, is a big sign saying 'don't write big servery things that deal with lots of independant tasks in Go'.

I don't know. Go currently feels to me like a missed opportunity. Mozilla's Rust looks like a much more thoughtfully designed language, especially with the idea that one task can provide read-only access to a variable to another, or transfer ownership entirely. I just wish they'd stop fiddling with it and ship a 1.0. Granted I have not actually used Rust for anything, so it might be horrible, too.

Now, gentle reader, there IS a language that is well suited for parallel, independant, fault-tolerant task execution: Erlang. I'm clearly biased (although I've tried most of the 'cool' languages at this point, so I'm at least informed as well), but Erlang's process model makes it almost a joy to deal with both parallel execution and fault tolerance. I built a (albeit simple) server in 20 minutes once that ended up in production for *years*. Because I wrote with an eye towards fault tolerance, it was tolerant of all sorts of stupid invalid inputs that came its way, without crashing the server itself, just the particular process handling that connection. In Go, from what I can tell, you'd end up with tons of defensive programming and still no gurantees you handled all the edge cases. I've been there, I know how to program like that, and how long it takes to flush all the bugs out. Alternatively I have sat on an Erlang shell, watching processes crash, writing the patch (if needed) and hot-code reloading it. New connections hitting that same bug magically start to work.

I don't expect this rant to stem the tide of "we rewrote our <core service> in Go and made it 65535% faster with 1% of the lines of code", but knowing what I know now, I'll probably treat them with even less creduility than before. Speed and LOC are not all a service needs to provide (usually).
