---
layout: post
title:  "Packaging Python apps for Debian"
date:   2020-09-20 12:00:00 +0300
tags:   post development debian pyfa
---

Recently I had lost self-control and started thinking about going back to playing EVE again (for the third time?)
After getting all hyped up on various streams and patch notes,
I have realized that my OS habits have changed a bit since the time.
No longer I own any physical machines with Windows (and I don't want to).
Thankfully, EVE has been supporting Linux since like forever via Wine.

After installing the client and checking in on my characters, naturally, I have reached for familiar tools.
EVE Fitting Tool has been already dead when I left.
Pyfa already existed then and was (and is) great.
What surprised me is that Pyfa was not packaged for my beloved Debian/Ubuntu boxes.
While running it from source is pretty much doable
(especially if you're not as clueless as I am and will avoid compiling wxWidgets from source on your first install),
having it all nicely packaged would be much better.

I have some experience in Debian packaging thanks to Chibi Scheme,
so I thought,
"How hard could it be to package some Python software for a change?"

Well, not as hard as it may seem to, but definitely not a 10-minute hack.
As Göran Weinholt aptly put it,

> The "Debian magic" is hours and hours of manual work.

For what it's worth, *never* in my experience I had encountered packaging issues with anything coming from Debian's repos.
Software bugs? Sure thing!
But having `apt` bork my system directories? Never.
Package fails to install? Nope.
Package leaves crap in the the system after uninstallaton? Not eve— well, *rarely*.
Package fails to start after installation? Seriously?

So... Instead of playing the damn game, I'm off wasting my life fiddling with computers again.
I hope I'll lose interest partway and forget about it all eventually, but at least I'll learn something new.
