---
layout: post
title: "Field notes on extending the Erlang packet parser"
description: "Another dispatch from the depths of the BEAM"
category: programming
tags: [erlang, networking]
---
{% include JB/setup %}

It's that time again, dear reader, in which I get caremad about something and go
off on a Quixotic adventure to do something about it. The target of my ire this
time is binary network protocols that are not length prefixed and how to handle
them in Erlang.

One of the great things in Erlang is `active` mode for sockets and the
`{packet, N}` option. Setting options like `{active, true}, {packet, 4}` tells
Erlang to send the owner of the socket a message that looks like `{tcp, Socket,
Payload}` every time it receives a 4-byte big-endian length-prefixed packet.
Even better, sending on that socket automatically prefixes the payload with the
4 byte prefix. This makes framing and deframing streams of data on sockets in
Erlang trivial, so long as both sides support and use this simple framing format.
It also allows the Erlang process owning the socket to do other things while the
packet is being accumulated by the runtime system. This is helpful because your
gen_server or whatever can just define a
[handle_info](http://erlang.org/doc/man/gen_server.html#Module:handle_info-2)
clause for packets instead of having to periodically read the socket for any
pending data.

This kind of length prefixed packet framing is reasonably common, thankfully
(endianness aside), but it's not universal. Herein lies the rub.

Consider, for example, the
[Yamux](https://github.com/hashicorp/yamux/blob/master/spec.md#framing) packet
format. It consists of 4 header fields followed by a length byte. What's wrong
with this you ask? Well, consider how you have to receive this protocol. First
you'd read 12 bytes to get the header, then read an additional N bytes to
receive the payload. This is fine, but it involves more tracking and buffering
as compared to the `packet,N` approach, despite being essentially identical.

It gets even worse, consider the
[mplex](https://github.com/libp2p/specs/tree/master/mplex#message-format) muxer
protocol. The protocol messages begin with 2 *varints*, one is the header flags
and the second is the payload length. This is a real pain in the ass because now
you can't even do a fixed receive to read the packet length (I mean, technically
you can because the varints have a maximum length). Again though that's a lot of
extra work as compared to `packet,N`, you have to do a blocking recv of at least
whatever the maximum varint size is multipled by 2, or you can read it bytewise
and accumulate until you have all of both varints.

Another example is the [UBX binary
protocol](https://www.u-blox.com/sites/default/files/products/documents/u-blox8-M8_ReceiverDescrProtSpec_%28UBX-13003221%29_Public.pdf)
(see section 33.2) used on u-blox GPS receivers. It has 2 bytes of sync word, 1
byte of message class, one byte of message ID and a 16 byte little-endian length
field. It's not a bad protocol and, in fact this is a good structure because it
can be sent over transports where bytes can be dropped if they're not received
so the sync word is very necessary, but it again can be clumsier to work with
than desired.

What if there was a better way? How does Erlang do its magic with `packet,N` and
what other packet types are there? It turns out that it's done with something
called the
[packet parser](https://github.com/erlang/otp/blob/master/erts/emulator/beam/packet_parser.c)
and it supports quite a few packet types:

 * `raw` - No packet parsing
 * `1`, `2`, `4` - The packet,N mode described above
 * `asn1` - ASN.1 BER
 * `sunrm` - [SUN RPC encoding](http://www.rhyshaden.com/rpc.htm), another classic
 * `cdr` - CORBA, nuff said
 * `fcgi` - [Fast
   CGI](http://www.mit.edu/~yandros/doc/specs/fcgi-spec.html#S3.1)
 * `tpkt` - [TPKT format from
   RFC1006](https://tools.ietf.org/html/rfc1006#section-6)
 * `line` - Newline terminated
 * `http` - HTTP 1.x response packet
 * `httph` - HTTP 1.x headers (used by `http` as well)

This is actually a surprisingly rich selection of packet types (although with a
distinctly 90s vibe). Each of these packet types has code that checks if the
packet is complete or if more bytes are needed. The packet parser is actually
used in 2 places, in the TCP receive path, and in
[erlang:decode_packet/3](http://erlang.org/doc/man/erlang.html#decode_packet-3)
which takes a packet type, some binary data, and some packet options. Thus you
can decode from a TCP (or TLS) socket or from a file or from memory.

Now, as you'll no doubt have noticed, this is a fairly arbitrary selection of
protocols. For example websockets (which has a framing mechanism) is nowhere to
be found, likely because it was invented long after 1995. Similarly none of the
protocols I mentioned above appear, which is not surprising.

Having hit the limits of Erlang's packet parser in the past, I finally decided
yesterday to try to support a new packet type. However, I didn't want to add
just any packet type, but rather a way to describe many common binary framing
schemes so I could support yamux, mplex, UBX and anything else that was
relatively simple (websocket framing is more complicated so it's beyond what
I've implemented below).

The result I came up with can be found
[here](https://github.com/erlang/otp/compare/maint-21...helium:adt/packet-match-spec)

It enables functionality like this:

```
4> erlang:decode_packet(match_spec, <<16#deadbeef:32/integer-unsigned-big, 2:16/integer-unsigned-little, "hithisisthenextpacket">>, [{match_spec, [u32, u16le]}]).
{ok,<<222,173,190,239,2,0,104,105>>,
    <<"thisisthenextpacket">>}
```

And more broadly things like this:

```erlang
test() ->
    {ok, LSock} = gen_tcp:listen(5678, [binary, {packet, raw},
                                        {active, false}, {reuseaddr, true}]),
    spawn(fun() ->
                  {ok, SSock} = gen_tcp:accept(LSock),
                  gen_tcp:send(SSock, <<16#deadbeef:32/integer, 2:8/integer, "hi",
                                        16#c0ffee:32/integer, 3:8/integer, "bye">>),
                  timer:sleep(infinity)
          end),
    {ok, S} = gen_tcp:connect("127.0.0.1", 5678, [binary, {active, true},
                                                  {packet, match_spec}, {match_spec, [u32, u8]}]),
    io:format("connected~n"),
    receive
        {tcp, S, <<16#deadbeef:32/integer,Length:8/integer, Data:Length/binary>>} ->
            io:format("Got data ~p~n", [Data]) %% Data is 'hi' here
    end,
    receive
        {tcp, S, <<16#c0ffee:32/integer,Length2:8/integer, Data2:Length2/binary>>} ->
            io:format("Got data ~p~n", [Data2]) %% Data2 is 'bye' here
    end.
```

Essentially it allows you to define a list of fields (available types are `u8`,
`u16`, `u16le`, `u32`, `u32le` and `varint`) the *last* of which is the payload
length field. Thus the yamux spec would be `[u8. u8, u16, u32, u32]` and the
mplex spec would be `[varint, varint]`. Annoyingly the UBX protocol doesn't
work with this scheme because 2 checksum bytes appear after the payload, but are
not included in the length. I will try to think of a way to support this
relatively common pattern as well. Perhaps something like `[u8, u8, u8, u8, u16,
'_', u16]` and have the `_` indicate the variable-length payload immediately
following the length byte (non-payload-adjacent length fields is probably
pushing the limits of what this feature should do).

So, how the hell does all this work? Well, it's remarkably complicated and has
to touch some rather gritty corners of the BEAM. Essentially, as noted above,
there's 2 ways to invoke the packet parser. Decode packet goes through
`erl_bif_port.c` which implements all the built-in-functions (before NIFs there
were BIFs, but only OTP was allowed to implement them) for dealing with ports.
Like NIFs, BIFs get passed some C version of Erlang terms which they have to
destructure and interpret to control the behaviour of the C code. Annoyingly,
this is not the same [enif](http://erlang.org/doc/man/erl_nif.html) API as NIFs
use; it appears to be some distant ancestor of it. Anyway, once we've parsed the
arguments to erlang:decode_packet and decoded the options, we call
[packet_get_length](https://github.com/erlang/otp/blob/master/erts/emulator/beam/packet_parser.c#L255)
which returns -1 on error, 0 on 'not enough bytes' or a
positive integer (that is the length of the packet)
when it has a complete packet for whatever the selected packet type is.
This is the simpler path.

For sockets, we first have to traverse gen_tcp which yields the parsing of
packet options to
[inet.erl](https://github.com/erlang/otp/blob/master/lib/kernel/src/inet.erl)
, which quickly calls into
[prim_inet](https://github.com/erlang/otp/blob/master/erts/preloaded/src/prim_inet.erl)
which constructs the actual port commands to the
[inet_drv](https://github.com/erlang/otp/blob/master/erts/emulator/drivers/common/inet_drv.c)
port. In Erlang, ports are essentially sub-programs that communicate with the
host BEAM via (usually) stdin/stdout/stderr (or other file descriptors).
Sometimes, in the case of the ODBC port, the port opens a TCP connection back
to the BEAM for performance. Ports are one of the oldest mechanisms the BEAM has
for interoperating with the operating system or underlying hardware, and their
process isolation means they remain the safest.

However, because data now has to cross a process boundary, we have to
marshal/unmarshal it to get it across. Again, inet_drv probably predates
[erl_interface](http://erlang.org/doc/tutorial/erl_interface.html)
which provides some nice support for this (including a way to
un-marshal the erlang binary term format) and it does all its communication with
a fairly simple binary 'protocol'. Essentially each 'command' is prefixed by
some kind of `INET_OPT` shared constant followed by some optional data. For
example setting the reuseaddr is done via the `INET_OPT_REUSEADDR` constant
(defined as 0). prim_inet handles turning `{reuseaddr, true}` into something
that looks like `<<?INET_OPT_REUSEADDR:8, Value:32/integer>>` and sending it
down to inet_drv where it is parsed in a giant switch statement and then somehow
actually applied using setsockopt.

This is mostly fine, although the big snag is the `prim_inet` module is special
in that it's
[preloaded](https://github.com/erlang/otp/blob/master/HOWTO/BOOTSTRAP.md#preloaded-code).
Preloaded modules are BEAM bytecode that is
essentially compiled into the BEAM when the BEAM is built and cannot be reloaded
or changed without rebuilding the BEAM. Even more interestingly the preloaded
modules are not normally compiled when you build OTP from source, the OTP
distribution, and the git repo, contain the precompiled beams. If you wish to
perform the dark-art of recompiling a preloaded beam you must use `make
preloaded`, which re-compiles any changed preloaded beams (but does not put them
in the right place for the BEAM build process to pick them up). If the
compilation looks like it worked, you can then use `./otp_build
update_preloaded` which will recompile the preloaded beams and put them in the
right place (note that this will recompile ALL the precompiled beams and also
make a git commit on your behalf(???), so use with caution). You can also simply
copy the beam file you've recompiled into the right place by hand.

Precompiled beams also have some restrictions. For example you probably don't
want to call io:format() from inside them, because precompiled beams can run
before the BEAM is fully booted and some things like the io service might not be
available yet. Happily debug macros are provided to ease the pain a bit.

So, to get my new packet type and options to work, I had to work my way down
through the layers of parsing, serialization, deserialization and usage to
actually get my new options to make it all the way to inet_drv's use of the
packet parser. This was not easy, and I might not have done it the right way,
but I eventually did get it to work.

To summarize, in less than a day's work and less than 200 lines of (only
somewhat horrible) code I was able to add what I think is a useful feature to
Erlang despite having touched hardly any of these parts of Erlang system before.
I hope to clean this up some more and submit it to the OTP team for inclusion. I
will probably change the name from `match_spec` to `packet_spec` or something
and maybe try to support the UBX use-case better. I don't know how much longer
inet_drv will be around (the file driver was rewritten to be a NIF that uses
dirty schedulers for OTP 21, maybe the inet driver is next?) but maybe we can
think about keeping the idea of powerful packet parsing down in the VM and
evaluate approaches like this to make it more flexible (and less 90s themed).
Longer term it might be nice to have something like BPF programs you pass down
into the packet parser, but that would be a lot more work.

Finally, I'd like to thank [Marc Nidjam](https://twitter.com/madninja) for
pitching in on the varint support and the tests (not all his code is in there
yet). Any other suggestions or assistance is most welcome.
