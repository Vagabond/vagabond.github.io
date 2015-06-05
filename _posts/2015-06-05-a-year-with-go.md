---
layout: post
title: "A year with Go"
description: ""
category: rants
tags: [golang]
---
{% include JB/setup %}

So, it has [been a year](rants/2014/05/30/a-week-with-go/) I've been working with Go. Last week I removed it from production.

Re-reading my impressions after just a week, I pretty much stand by what I said back then, but there's a few other things that I'd like talk about, and amplify some points from the previous post.

Now, I'm writing this up because people have asked me about my thoughts on Go several times over the past year, and I wanted to go into a little more depth than is possible over Twitter/IRC before all the details fade from memory. If you're not interested in my opinion, or are ending up here via some Go news aggregator or something and want to show me the error of my ways, you probably needn't bother. I'm going to put Go (alongside C++, Java and PHP) in the weird drawer under the microwave where all the stuff you can't find a good use for gravitates.


So, lets talk about the reasons I don't consider Go a useful tool:

The tooling
===========

Go's tooling is really weird, on the surface it has some really nice tools, but a lot of them, when you start using them, quickly show their limitations. Compared to the tooling in C or Erlang, they're kind of a joke.

Coverage
--------

The Go coverage tool is, frankly, a hack. It only works on single files at a time and it works by inserting lines like this:

```golang
GoCover.Count[n] = 1
```

where `n` is the branch id in the file. It also adds a giant global struct at the end of the file:

```golang
var GoCover = struct {
        Count     [7]uint32
        Pos       [3 * 7]uint32
        NumStmt   [7]uint16
} {
        Pos: [3 * 7]uint32{
                3, 4, 0xc0019, // [0]
                16, 16, 0x160005, // [1]
                5, 6, 0x1a0005, // [2]
                7, 8, 0x160005, // [3]
                9, 10, 0x170005, // [4]
                11, 12, 0x150005, // [5]
                13, 14, 0x160005, // [6]
        },
        NumStmt: [7]uint16{
                1, // 0
                1, // 1
                1, // 2
                1, // 3
                1, // 4
                1, // 5
                1, // 6
        },
}
```

This actually works fine for unit tests on single files, but good luck getting any idea of integration test coverage across an application. The global values conflict if you use the same name across files, and if you don't then there's not an easy way to collect the coverage report. So basically if you're interested in integration tests, no coverage for you. Other languages use more sophisticated tools to get coverage reports for the program as a whole, not just one file at a time.


Benchmarking
------------

The benchmarking tool is a similar thing, it looks great until you actually look into how it works. What it ends up doing is wrapping your benchmark in a for loop with a variable iteration count. Then the benchmark tool increments the iteration count until the benchmark runs 'long enough' (default is 1s) and then it divides the execution time by the iterations. Not only does this include the for loop time in the benchmark, it also masks outliers, all you get is a naive average execution time per iteration. This is the actual code from benchmark.go:

```golang
func (b *B) nsPerOp() int64 {
    if b.N <= 0 {
        return 0
    }
    return b.duration.Nanoseconds() / int64(b.N)
}
```

This will hide things like GC pauses, lock contention slowdowns, etc if they're infrequent.


Compiler & go vet
-----------------

One of the things people tote about Go is the fast compile speed. From what I can tell, Go at least partially achieves this by simply not doing some of the checks you'd expect from the compiler and instead implementing those in `go vet`. Things like [shadowed variables](http://www.qureet.com/blog/golang-beartrap/) and bad printf format strings aren't checked by the compiler, they're checked with `go vet`. Ugh. I've also noticed `go vet` actually regress between 1.2 and 1.3, where 1.3 wasn't catching valid problems that 1.2 would.

go get
------

The less said about this idea the better, the fact that Go users now say not to use it, but apparently are making no move to actually deprecate/remove it is unfortunate, as is the lack of an 'official' replacement.

$GOPATH
-------

Another idea I'm not enthralled with, I'd rather clone the repo to my home dir and have the build system put the deps under the project root. Not a major pain point but just annoying.

Go race detector
----------------

This one is actually kind of nice, although I'm sad it has to exist at all. The annoying thing is that it doesn't work on all 'supported' platforms (FreeBSD anyone?) and it is limited to 8192 goroutines. You also have to manage to hit the race, which can be tricky to do with how much the race detector slows things down.

Runtime
=======

Channels/mutexes
-----------------

Channels and mutexes are SLOW. Adding proper mutexes to some of our code in production slowed things down so much it was actually better to just run the service under daemontools and let the service crash/restart.

Crash logs
-----------

When Go DOES crash, the crap it dumps to the logs are kind of ridiculous, every active goroutine (starting with the one causing the crash) dumps its stack to stdout. This gets a little unwieldy with scale. Also, the crash messages are extremely obtuse, including things like 'evacuation not done in time', 'freelist empty' and other gems. I wonder if the error messages are a ploy to drive more traffic to Google's search engine, because that's the only way you'll figure out what they mean.

Runtime inspectability
----------------------

This isn't really a thing, you're better off just writing in a real systems language and using gdb/valgrind/etc or use a language with a VM that can give you a way to peek inside the running instance. I guess Go keeps the idea of printf debugging alive. You can use GDB with Go, but you probably don't [want to](https://groups.google.com/forum/?hl=de#!searchin/golang-dev/gdb/golang-dev/UiVP6F-9-yg/lqS3sbyfTZMJ).

The language
============

I genuniely don't enjoy writing Go. Either I'm battling the limited type system, casting everything to interface{} or copy/pasting code to do pretty much the same thing with 2 kinds of structs. Every time I want to add a new feature it feels like I'm adding more struct definitions and bespoke code for working with them. How is this better than C structs with function pointers, or writing things in a functional style where you have smart data structures and dumb code? Don't even get me started on the anonymous struct nonsense.

I also, apparently, don't understand Go's pointers (C pointers I understand fine). I've literally had cases where just dropping a * in front of something has made it magically work (but it compiled without one). Why the heck is Go making me care about pointers at all if it is a GC'd language?

I also tire of casting between byte[] and string, and messing with arrays/slices. I understand why they're there, but it feels unnecessarily low level given the rest of Go.

There's also the whole nonsense of [:], ... and append, check this out:

```golang
iv = append(iv, truncatedIv[:]...)
```

This converts the array 'truncatedIv' into a slice of all the elements, explodes the slice to be an argument list, and appends those arguments to 'iv'. append() here is a special magic builtin that works for any slices (you might even say it was generic). You have to reassign the result of the append() call to the variable being appended to because append *sometimes*, depending on the size of the array underlying the slice, will append in-place and sometimes will allocate a new array and return that. It is basically realloc(3) for Go.

The Stdlib
----------

Some of Go's stdlib is pretty nice, the crypto stuff is a lot less clumsy than the shitty OpenSSL wrapper lots of languages give you. I don't really enjoy the Go documentation though, especially when interfaces are involved. I usually have to go read the source code to figure out what is actually going on. "Implements the X method" isn't that useful if I don't know what X is supposed to do.

I *do* have quite a big problem with the 'net' package. Unlike regular socket programming, you don't get to configure the socket the way you want. Want to toggle an arbitrary sockopt like IP_RECVPKTINFO? Good luck. The only way to do that is via the 'syscall' package, which is the laziest wrapper around the POSIX interface I've seen in a while (reminds me of some old PHP bindings). Even better, you can't get the file descriptor out of a connection initiated with the 'net' package, you get to standup the socket *entirely* with the syscall interface:

```golang
    fd, err := syscall.Socket(syscall.AF_INET6, syscall.SOCK_DGRAM, 0)
    if err != nil {
        rlog.Fatal("failed to create socket", err.Error())
    }
    rlog.Debug("socket fd is %d\n", fd)

    err = syscall.SetsockoptInt(fd, syscall.IPPROTO_IPV6, syscall.IPV6_RECVPKTINFO, 1)
    if err != nil {
        rlog.Fatal("unable to set IPV6_RECVPKTINFO", err.Error())
    }

    err = syscall.SetsockoptInt(fd, syscall.IPPROTO_IPV6, syscall.IPV6_V6ONLY, 1)
    if err != nil {
        rlog.Fatal("unable to set IPV6_V6ONLY", err.Error())
    }

    addr := new(syscall.SockaddrInet6)
    addr.Port = UDPPort

    rlog.Notice("UDP listen port is %d", addr.Port)

    err = syscall.Bind(fd, addr)
    if err != nil {
        rlog.Fatal("bind error ", err.Error())
    }
```

And then you get the joy of passing/receiving byte[] parameters to/from the syscall functions. Constructing/destructuring C structures from Go is super-fun.

Apparently the reason for this madness is the 'net' package assumes the sockopts are set up a specific way so the socket polling can work? I don't know for sure but I know it makes any 'fancy' network programming pretty annoying and dubiously portable.

Conclusion
==========

I just don't understand the point of Go. If I wanted a systems language, I'd use C/D/Rust, if I wanted a language built around concurrency I'd use Erlang or Haskell. The only place I can see Go shining is for stuff like portable command line utilities where you want to ship a static binary that Just Works(tm). For interactive tasks I think it would be fine, I just don't think it is particularly well suited to long-running servery things. It also probably looks attractive to Ruby/Python/Java developers, which is where I think a lot of Go programmers come from. Speaking of Java, I wouldn't be surprised to see Go end up as the 'new Java' given the easier deploy story and the similar sort of vibe I get from the language. If you're just looking for a 'better' Ruby/Python/Java, Go might be for you, but I would encourage you to look further afield. Good languages help evolve your approach to programming; LISP shows you the idea of code as data, C teaches you about working with the machine at a lower level, Ruby teaches you about message passing & lambdas, Erlang teaches you about concurrency and fault tolerance, Haskell teaches you about real type systems and purity, Rust presumably teaches you about sharing memory in a concurrent environment. I just don't think I got much from learning Go.
