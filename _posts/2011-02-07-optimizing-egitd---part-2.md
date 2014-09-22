---
layout: post
title: "Optimizing egitd  - Part 2"
description: ""
category: programming
tags: [erlang, egitd]
---
{% include JB/setup %}

I've started moving the handing of the individual socket connections out into a gen_server. I have it doing basic 'git method' packet parsing, but I'm doing it with binaries and the bit syntax, not strings and regular expressions. The reason I'm doing this is that its a lot faster, and it uses a lot less memory (strings in erlang are linked-lists of integers (32 or 64 bit, depending on your machine)). Also, you can split binaries which essentially gives your a pointer into a sub-binary, instead of copying all the data into a new variable (you remember that erlang is single assignment and all data is immutable, right?).

The commit with this initial work is [here](https://github.com/Vagabond/egitd/commit/471b4a7a492761d6b272cf5a46d197fb06e5e6bf), its not finished yet, so I haven't switched server.erl over to using it yet, but contrast the handle_info clause doing the pattern match with all the code server.erl is doing before it extracts the method name.

Then I [add support](https://github.com/Vagabond/egitd/commit/8db0f4303cbaad81dd38ffb46ae5245f4641674f) for actually dispatching based on the git method requested. The old egitd only supported 'upload-pack' and 'receive-pack', so that all I'm going to do. 'receive-pack' is actually disallowed so the only *real* operation is 'upload-pack'. I also move the packet pattern matching up into the function clause for tidiness. The validation on the 'upload-pack' is also added (it gets a little hairy there) and then we open the port to git upload-pack, but we don't use it.

The code still doesn't work, because the messages on the port and the socket aren't exchanged. So now I actually start exchanging the port messages and the socket messages. Basically once there's a port created, any messages on the socket go to the port and any on the port go to the socket.

I actually got stuck for a while on this bit because while I was relaying messages from the port to the socket, the socket never sent me data back. This was because I was forgetting to set {active, once} on the socket after every packet I consumed. This is something you MUST remember to do, or you'll never get any more packet messages (unless you want to switch into passive mode or something).

So, I fixed that and it WORKS. Here's the [changes needed](https://github.com/Vagabond/egitd/commit/90fdbb761f80dc35d199a86af51159cf7c5545b9). Really we just have 2 handle_info clauses to handle incoming packet data and forward it to the socket, one for the other direction and one to exit when the socket closes.

Now, lets look at some numbers. Here's three runs cloning the [FreeSWITCH](http://freeswitch.org/) repo with 'classic' egitd. This is a good repo as its fairly large and has a long commit history. I did several clones before this to warm the disk-cache up. The client and server are on the same machine, but its a quad core i7, so I don't think that's too significant.

    git clone git://localhost/FreeSWITCH.git  13.23s user 3.01s system 14% cpu 1:52.91 total
    git clone git://localhost/FreeSWITCH.git  13.23s user 2.95s system 14% cpu 1:48.64 total
    git clone git://localhost/FreeSWITCH.git  12.39s user 2.91s system 13% cpu 1:53.04 total

Here's the same test with the new egitd:

    git clone git://localhost/FreeSWITCH.git  12.65s user 2.72s system 70% cpu 21.721 total
    git clone git://localhost/FreeSWITCH.git  12.48s user 2.62s system 71% cpu 21.036 total
    git clone git://localhost/FreeSWITCH.git  12.52s user 2.64s system 72% cpu 20.829 total

So, I think I found the problem. With this simple rework we're 7x faster on the same repo on the same hardware. The numbers are also more consistant (I think because we're not blocking on socket timeouts).

So here's the takeways from this:

 * Use OTP, OTP is your friend and it makes writing erlang processes like this trivial. Even if you aren't going to interact with other OTP processes, the handle_info callback is great for stuff like this.
 * Use binaries, we didn't get a big win from that in this case, since we weren't doing a lot of processing, but the new way the git packets are parsed is a lot more efficient than regexing on strings.
 * Use {active, once} mode on sockets, it fits great into erlang's async nature. Don't do a gen_tcp recv unless you have a good reason (you want to block on a packet, you want to do a tight-receive loop for lots of data).
 * Don't forget to keep setting {active, once} on a socket EVERY SINGLE TIME you are ready to get another packet.

That's all for now. I think I've already solved the real issue with egitd, but I'm going to look at the code some more, benchmark it against git-daemon itself and test it with a really big repo, like the linux kernel.
