---
layout: post
title: "Hot takes on Elixir"
description: ""
category: rants
tags: [erlang]
---
{% include JB/setup %}

So, Elixir has been a thing for a while now, and on the whole it seems like a great thing. People who get all hung up on Erlang's syntax have an alternative, we have Hex for Erlang packages in rebar 3 and they've come up with some cool syntax like the pipe operator that might make it back into Erlang one day.

However, I do have a bit of a problem with Elixir: people are using my Erlang libraries from Elixir.
I'm the author of 2 fairly popular libraries for Erlang; lager for logging and gen_smtp for SMTP. Both have become arguably the de-facto libraries for those tasks in Erlang. Obviously the Elixir community would use that battle tested code in their own ecosystem as well, and they do. This is all fine and well, and I'm very happy my code is making the world a better place. The problems are two fold: support and credit.

I've been getting enough Elixir GitHub issues filed that it is getting annoying. Almost always it has to do with incorrectly invoking my Erlang code from Elixir. When it is a legitimate bug I'm stuck trying to understand what the hell the Elixir code actually is doing (I don't use Elixir and so I'm not very familiar with it). Essentially every time I see a Github email come in and it mentions Elixir, my heart sinks. I'm already neglecting my open source maintainerships (free code doesn't pay well), and this isn't helping.

The second issue is credit. Some of the Elixir wrappers for my libraries don't actually acknowledge they're wrappers around my code. There's nothing in the license that requires that, but it feels a bit... icky. Whenever I wrap code, or use some code to derive something, I try to give credit. Open source as a resume booster is a thing (it's happened to me), but also if you don't actually know what code you're using in your project, because the wrapper hid it from you, you have no way to know if a security vulnerability or a bugfix applies to you.

I'm sure people who write Java or .NET libraries see the same problems with Clojure/Scala/F# etc. It is just interesting to see it play out in Erlang land.
