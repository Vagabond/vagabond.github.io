---
layout: post
title: "A History of OpenOMF"
description: ""
category: programming
tags: []
---
{% include JB/setup %}

As we are about to release 0.8.0 of [OpenOMF](http://openomf.org), I wanted to
look back a bit on my involvement with the project, and its predecessor, which
go back to late 2004, or really to 1994. I am going to recount the story mostly
from memory, so there may be some errors or misconceptions in what follows.

One Must Fall 2097 was a DOS fighting game for the IBM PC. It was developed by a
small Florida game developer company called Diversions Entertainment, and it was
published by Epic Megagames. The game was the commercial version of an earlier
shareware fighting game (which we call omf 1) which a young programmer named Rob
Elam had released. For 2097 the game was massively expanded to include 10 unique
fighting robots (called Human Assisted Robots or HARs in the game's lore), 10
single player pilots for those HARs, a single player boss character, a
tournament mode with RPG elements and a remarkable amount of game options and
secrets.

I was first exposed to the game via the shareware demo, which I believe we got
on a CD or floppy taped to the front of a computer magazine (this was the era in
which downloading more than a few hundred kilobytes from the internet was an all
day affair). My brother and I, having never really played a fighting game
outside an arcade before, were enthralled. We played the heck out of the demo
and quickly convinced our parents we needed the full copy. My parents did
whatever bizarre ordering procedure the time called for, and a few weeks later a
box edition of the game arrived, complete with the manual, a poster and a
strategy guide (all of which I still have). We then proceeded to play the game
obsessively for most of a summer vacation.

I think everyone has some encounter with media that hits them at just the right
time, whether a book, a movie, a song or a video game. You're receptive to it in
some way that makes it hard to explain to others because in consuming the media
you are yourself changed by it. This was one of those pieces of media for me.
When I taught myself 3D modeling some of my first ever 3D models were HARs from
2097.

Once the Internet was more of a thing, in the late 90s and very early 00s I
discovered Diversions Entertainment was working on a 3D sequel to OMF, called
One Must Fall:Battlegrounds. I dabbled a bit in the online community that had
formed around the community, and I tried Battlegrounds when it came out, but I
found it a bit underwhelming and clunky compared to the original.

Several years later, I had graduated high school (barely), dropped out of
college (more school just wasn't what I could do), had spent a year abroad
living in Germany, and then returned home to Ireland, at a bit of a loss with
what to do next. For some reason I decided to pick up OMF2097 again. I found
that, while the game had had networking support added and the game itself had
been made freeware in 1999, it no longer ran well under Windows 2000 and you
had to use something called "DOSBox" to run it. However, I could never get the
game to "feel" right under DOSBox, no matter how much I tweaked the cycles or
the settings. I had also, in the intervening years, learned how to program,
primarily in a "new" language called Ruby. I decided I was going to try to
[recreate the game](https://github.com/Vagabond/rubyomf2097) using
Ruby and a game engine called [Gosu](https://www.libgosu.org/). I had done a bit
of OpenGL and C++ programming before this, and decided I wanted nothing to do
with it, so Ruby/Gosu let me focus on the parts that I found interesting.

I had found there were some fan-made tools for unpacking/repacking some of the
game assets, especially the "AF" files, which is where the HAR information was
stored. These tools also documented the binary file format, and how to extract
it. I then had to teach myself how to work with binary files from Ruby (turns
out String.unpack/pack support some pretty complex specification strings). I
then wrote some tools to decompile the assets into sprites and giant XML files
of the known data. This proved to be a mistake, as I spent a lot of time messing
around with updating the representation of the data as I learned more about it.

After a little while, I had something that looked a bit like a game (although it
didn't really act like one). I created a RubyForge (RIP) project for it called
rubyomf2097 and posted about it to the [OMF
forums](https://web.archive.org/web/20081231224855/http://www.omf2097.com/~forum/viewthread.php?tid=193).
People were interested, but cynical it was going to lead anywhere (apparently I was not the first person to
tilt at this windmill, although I believe I got the furthest). Eventually life
got in the way, and I sort of stalled out on the project (although I had
developed some tools for editing the asset files and learned a whole bunch along
the way). There was just too much unknown about how the game worked, and things
seemed to be much more complex than they might appear at first glance. I did
remain around the community, and in the #omf IRC channel on Freenode (RIP).

Then, sometime in 2012, someone called "katajakasa" posted about
[their OMF2097 remake](https://github.com/katajakasa/old-omf2097-engine-remake),
this time in C++. I had been programming professionally for several
years by then, and had done a fair amount of C programming. I also had done
enough C++ to realize I really didn't like it, so I proposed joining forces if
he agreed to switch to C. He agreed so he and I and another OMF2097 fan from
Australia, "animehunter", joined forces and started on another remake. We ported
over what we had from the 2 previous codebases and started on implementing
libraries to implement encoders/decoders for the various game formats. As this
progressed we also started building a new game engine from scratch, using SDL2
as the base to give us basic things like window handling, input, etc.

We made pretty good progress for the next couple years, but after about 2014 the
pace of the project slowed. It turned out the game we had decided to reimplement
was vastly more complex and confusing than we had expected. The game had its
own internal scripting language that was used to control what effects would
happen on each frame of animation. This scripting language was difficult to
understand and reverse engineer given our tools and skillset. Katajakasa did
some decompilation using IDAPro, and I would use our tools to decompile the
assets, edit them and recompile them to see what would change in the original
game. This was extremely tedious and error prone, although we did manage to
solve several mysteries, like how collision detection worked, and a bunch of
other game mechanics (move types, how moves chain together, etc).

I also implemented a version of network play, using somewhat more modern
methods (the original used IPX/SPX in lockstep mode, where nothing could happen
until the other side acknowledged it), although I learned the hard way that
fighting games are notorious for being the hardest game type to write netcode
for. The approach I took ended up being very brittle and flawed, but I lacked
the energy to try again.

So the project went somewhat dormant. We had some contributions from the
community, katajakasa kept working on things here and there, but I had
essentially stepped away from doing anything, as had animehunter. I returned
briefly in early 2023 to implement the majority of Tournament mode, but then I
went dormant again. katajakasa had been working on a rewrite of the rendering
layer for a few years slowly (turns out simulating a VGA video buffer in modern
OpenGL is a bit tricky), but progress was pretty slow.

Then, miraculously, things started to come back to life around January of 2024.
A few new contributors arrived; martti, Nopey, Insanius and nopjne.
We also started using Ghidra for reverse engineering (we had been using it a
little during the lull as well). In August I left my job to take a break, and I
decided to spend some of my programming energy on OpenOMF. I started with
rewriting the network code from scratch, implementing a proper
[GGPO](http://ggpo.net) rollback style netcode, which ended up being as
difficult as expected. I also implemented a
[network lobby](https://github.com/omf2097/openomf_lobby), NAT support and UDP
hole punching support for the network client.

We finally made an official release for the first time in over 10 years, 0.7.0
(and a couple followup bugfix releases), and we've even packaged the game for
[Flatpak](https://flathub.org/apps/org.openomf.OpenOMF).

An intrepid contributor managed to port the game to the Nintendo 64 using
libdragon. A very impressive achievement, and one we intend to support in the
mainline codebase. This has proven the efficiency and portability of our engine,
and hopefully will help lay the groundwork for further ports.

We also finally landed the new rendering code, and have been rapidly progressing
on features and bugfixes since. We've restored and repaired support for the game
recordings from the original engine, and we've figured out how to use them both
as a way to inspect behaviour in the original engine, but also to embed
assertions into them our engine can check, so we can also use them as unit
tests.

We (mostly Insanius) also documented the memory layout of the original game
enough that we can dump player position/velocity/health/endurance/etc at
runtime. I wrote a simple C utility called
[OneMustSee](https://github.com/omf2097/OneMustSee) that can be
pointed at a dosbox pid. This allows us to play back a known recording in the
game, use the memory dumper to dump the memory values, then use those values to
annotate the REC for playback in our engine. This currently reveals a LOT of
small incompatibilities, but we have finally developed a pretty robust suite of
tools for interrogating the original engine and ensuring our own complies.

With the release of 0.8.0, we are considering the game to be in "alpha" state,
meaning that all the major features are implemented. Minor features may not be
implemented, and there may be some bugs or incompatibilities. The next focus
will be on getting all the smaller features implemented and correcting whatever
bugs we find along the way. Once we are confident that *all* features are
implemented, we will tag a 0.9.0 and then work on fixing all remaining known
incompatibilities until we reach 1.0.

We are also exploring a mod framework for the engine, to allow for things like
higher resolution assets, rebalancing, new arenas, enhanced features for
tournament mode, etc. Our project is actually one of the only open source
fighting game engines, and it has a unique lineage to all the other ones
(because OMF2097 itself was a bit of a weird fighting game), so the idea of
total conversions or other changes for the engine would also be possible.

If any of this sounds interesting, you're welcome to swing by our
[Discord](https://discord.gg/7CPPzab) or
[GitHub](https://github.com/omf2097/openomf). We could always use more people to
test, report bugs, play around with reverse engineering or C code, or just hang
out. Community engagement is all that keeps projects like this going, so if you
know of a similar project you'd like to see continue on, make sure to let them
know you appreciate the work they're doing.

Looking back on 20 years of this project, in one form or another, maybe I can
distill some lessons from it all. I think had we known then what we know now
about what the scope of this project entailed, we probably would not have tried.
This game turned out to be much more complex to implement than we expected, and
have a lot of unique features and quirks. I do think, however, that I've learned
a lot of useful things as a result. It taught me how to work with binary files,
helped improve my C programming skills, my network programming skills, my
ability to reverse engineer systems, how to use a debugger, etc. So if anyone
out there is considering a similar project, do not be dissuaded, just prepare
for it to take a bit longer than you expect. I do think we are finally in the
home stretch, but we just don't know exactly how far away the finish line is,
still.

Finally, I'd like to thank everyone who HAS participated or contributed over all
these long years. Every little spark of interest has helped us keep going.
