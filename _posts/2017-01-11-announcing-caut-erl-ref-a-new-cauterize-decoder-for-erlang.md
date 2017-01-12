---
layout: post
title: "Announcing caut erl ref; a \"new\" Cauterize decoder for Erlang"
description: ""
category: programming
tags: [erlang, networking]
---
{% include JB/setup %}

### What?

I just tagged [1.0.0](https://github.com/cauterize-tools/caut-erl-ref/tree/1.0.0) of [caut-erl-ref](https://github.com/cauterize-tools/caut-erl-ref) which is a Cauterize encoder/decoder implementation for Erlang. This isn't actually a 'new' library, it is almost a year old, but it has been in use for most of that time and I finally took the time to clean up some stuff and add some documentation.

"What the heck is Cauterize" I hear you cry, dear reader. [Cauterize](https://github.com/cauterize-tools/cauterize) is yet another serialization format, like msgpack, thrift, protocol buffers, etc. Cauterize, however, is targeted at hard real-time embedded systems. This means that it focuses heavily on things like predictable memory usage, small overhead and simplicity. At [Helium](https://helium.com) we use Cauterize extensively to shuttle our data around, especially on the wireless side, where smaller packets mean less transmit power used and more transmit range (because you can operate at a lower bitrate). Cauterize is an invention of my colleague, [John Van Enk](https://github.com/sw17ch), and he's provided implementations for C and Haskell. Another Helium colleague, [Jay Kickliter](https://github.com/JayKickliter) has a [Rust implementation](https://github.com/JayKickliter/caut-rust-ref).

John and I, last February at a Helium meetup in Denver, implemented the first versions of the Erlang implementation in about 4 hours. Since then I've been tweaking and refining it to better suit my usage. It is a little different than the other implementations, because the Cauterize code generator doesn't generate an encoder/decoder directly, it generates an abstract representation of the schema and uses a generic library (cauterize.erl) for the encoding/decoding. This probably means it is not the fastest implementation, but it did keep the code generator simple and I've mostly focused on making the library very powerful and easy to use.

### Features

In addition to being able to (obviously) encode/decode Cauterize, the Erlang implementation has a couple neat features:

#### Key value coding

The library is compatible with Bob Ippolito's [kvc](https://github.com/etrepum/kvc) library, which provides key-value coding for Erlang. This makes it very easy to traverse decoded Cauterize structures, rather than writing complicated pattern matching expressions.

#### Decode stack traces

When a Cauterize decode fails, erl-caut-ref will show you how far it managed to get before the parsing hit an error. This has been helpful in chasing down some packet corruption issues we've seen. This was quite a bit trickier than I expected to implement.

#### Lots of testing

The library has been in use for almost a year, it has a pretty comprehensive unit test suite and it's also been checked with [Crucible](https://github.com/cauterize-tools/crucible) which generates random schemas and random messages based on that schema and checks they can be decoded.

### Conclusion

Cauterize is pretty neat, it just gives you a very tiny serialization format. There's no [RPC bullshit](https://christophermeiklejohn.com/pl/2016/04/12/rpc.html), there's no fancy, brittle pieces, you can probably make it work anywhere (we use it on a bare-metal Cortex M0) and you can probably implement it for your own pet language yourself.
