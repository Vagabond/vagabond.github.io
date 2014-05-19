---
layout: post
title: "OpenSSL is dead, long live LibreSSL"
description: ""
category: rants
tags: []
---
{% include JB/setup %}

So, the OpenBSD people have just given their first public talk on LibreSSL, their fork of OpenSSL. View the [slides](http://www.openbsd.org/papers/bsdcan14-libressl/index.html) or the [video](https://www.youtube.com/watch?v=GnBbhXBDmwU).

Now, I have massive respect for the OpenBSD team. They are certainly a spiky bunch, but you can't argue with their results. So, when I saw them decide that enough was enough and OpenSSL needed forking, I was elated. They've already made great strides and their plans for the future look good as well.

However, the linux foundation has announced the [core infrastructure initiative](http://www.linuxfoundation.org/programs/core-infrastructure-initiative) which solicits donations from large companies to be used for the improvement of software projects considered fundamental to the internet. This is all well and good, except for one thing. I think their plans are to donate to OpenSSL, not LibreSSL.

I think this is a mistake and will be throwing good money after bad. Let me explain why.

One of the big reasons given for the endless stream of OpenSSL failures (heartbleed was just the best publicised) is lack of funding. I can excuse lack of progress due to lack of funding, but I can't excuse lack of quality. If you don't have enough money to do something right, don't do it at all.

The OpenSSL developers apparently don't agree with me and apparently just layered on more crap in response to the funding they got, rather than going back to revisit the previous layers of dreck. This is unacceptable behaviour on the part of people who work on something like OpenSSL.

So, I call on anyone thinking of bailing the OpenSSL devs out yet again to instead consider donating to LibreSSL or the OpenBSD project (they also make OpenSSH and a bunch of other cool stuff you might be using without realizing it). It'll do a lot more good.
