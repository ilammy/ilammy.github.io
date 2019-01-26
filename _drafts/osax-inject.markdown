---
layout: post
title:  Fun Injection via Scripting Additions
date:   2019-02-14 00:00:00 GMT+2
labels: post code-injection macos
---

One day divine providence spoke to me in a form of an old screenshot.
The screenshot depicted _screen mates:_
an obsolete practice of decorating your desktop and windows with critters.
I found this to be quite cute and wanted to have my own.
And to finally build something for this macOS of yours.

<!-- Maybe put an eye-catch here: that screenshot or yours. -->

In short,
I needed to have something similar to _drawers_
(which are deprecated on the latest macOS).
However, the drawer itself must be provided by a third-party application.

This turned out to be apparently impossible to achieve on macOS.

<!-- - - 8< - - cut here - - 8< - - -->

## Why code injection?

Some windowing systems have a built-in feature of _window reparenting_
which allows one to make their window a child of another window.
The API for this is just one function call:
`XReparentWindow()` with Xlib,
`SetParent()` with Win32.
But one thinks different in Apple land.
After some research I have found that there is no such feature in Quartz.
(Presumably, this is the same for Wayland then.)

Initially I thought that it might not be that bad.
After all, it should still be possible to create a window,
track the parent window position and Z-order,
and then reposition and reorder my window accordingly.
Right?
You guessed it.
Available API for these kinds of things turned out to be limited as well.

One can use _Accessibility API_ to track window position
and receive notifications about movements of a particular window.
The bad news:
it has extremely low resolution, around 1–2 FPS.
This is unacceptable for live motion tracking.
(The API also requires manual approval from the user.
It's irrelevant to my goals
but may disappoint people who would like some stealth.)

Another approach uses _Quartz Window Services_
in the form of the `CGWindowListCopyWindowInfo()` function.
The bad news:
that's a polling API and it's quite CPU-intensive.
There is another function `CGWindowListCreateDescriptionFromArray()`
which can provide information only about specified windows,
but you still need to keep track of all the other ones
in order to get accurate information about Z-order
and visible regions.
Furthermore,
the latency is quite visible.
I assume that it can be mitigated to some degree though.

A-a-and... that's about it
for the ways to track alien window positions on macOS.
Not very useful and easy thing to do.
And this does not even handle various other things
that one can do with windows:
minimization,
zoom,
live resize,
moving around desktops,
non-rectangular window geometry,
etc.

Research on the Internet suggested that
it should be possible to use code injection
for extending third-party applications with my windows.
Indeed, this should greatly simplify window tracking
as Cocoa already has a notion of _child windows_.
However, code injection is a kind of black art
and I was sure that Apple does not like it for security reasons.
My guess turned out to be right.

## History and background

### Injection techniques

### Apple Script terminology

## Building OSAX bundle

## Invoking via Scripting Bridge

## Caveats

### System Integrity Protection

### Code signature verification

### Dynamic linkage and loading

## References

- shoumikhin

<hr/>

- why I needed it
- alternatives considered
- known thing (if you know the right words to ask google for)
  but not one-stop tutorial
- apple script history
- apple script terminology
- osax bundle structure
- invoking via scripting bridge
- system integrity protection
- code signing
- linkage, @rpath, Swift support
- invoking via apple script
- literature
