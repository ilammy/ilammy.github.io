---
layout:   post
title:    "CreateRemoteThread for Linux"
date:     2019-XX-XX XX:XX:XX +0200
tags:     post linux
comments: true
---

WinAPI has an interesting function called [CreateRemoteThread]
which can launch a new thread in a different process address space.
It can be used for various _DLL injections_ with various goals,
like malware implementation (videogame cheats, keyloggers, etc.)
or to fix a bug in a non-stop production environment,
or to add plugins to software which is not extensible enough.

This is a fairly questionable feature
so it's not surprising that Linux does not have a drop-in replacement for it.
However, I was interested in how it could be implemented.

Brace yourself for a ride,
I'm going to tell you how to implement a tiny debugger.
It will be able to inject and execute arbitrary code
into a live and kicking external process.

You'll need some basic skills and knowledge about Linux system programming:

  - how to read, write, and debug C programs
  - what role machine code and memory play in computers
  - what are system calls
  - familiarity with standard UNIX libraries
  - ability to read the docs

<!-- eyecatch here -->

[CreateRemoteThread]: https://msdn.microsoft.com/en-us/library/windows/desktop/ms682437(v=vs.85).aspx

<!-- table of contents here -->
