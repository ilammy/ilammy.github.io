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

## Basic ideas

We could have taken a massive shortcut by using `LD_PRELOAD`
if it weren't for that pesky requirement
to inject code into a currently running process.
`LD_PRELOAD` environment variable instructs the dynamic loader
to load an additional shared library during process startup.
Shared libraries can contain _constructor functions_
that are executed when a library is loaded.

`LD_PRELOAD` with constructors can be used to execute arbitrary code
in (almost) any process that uses dynamic loader.
It's a widely known feature which is often used for debugging.
For example, one can implement a wrapper library
that overrides `malloc()` and `free()` functions
for tracking memory leaks.

Unforuntately, `LD_PRELOAD` works only at load-time.
You cannot use it to load a library _after_ a process has been launched.
Libraries can be loaded at run-time with `dlopen()`
but the process has to call it itself,
for example when explicitly loading plugins.

> **On static linkage**
>
> `LD_PRELOAD` works only for dynamically-linked applications.
> If you build a binary with the `-static` option
> then it will simply embed all required libraries.
> Dependency resolution happens statically, at compile-time in this case.
> The resulting application will be not ready and not able
> to load more libraries dynamically, at run-time.
>
> It is possible to inject code at run-time into statically-linked processes.
> But it's much more tricky and not quite safe
> as it involves live-patching executable code of the application.

To sum it up, there is no readymade solution,
we need to invent something new here.
If there was a widely known official way
then you wouldn't be reading this text :)

Conceptually, we'll need to perform the following steps
to coerce an alien process into executing arbitrary code:

 1. Assume control over the target process.
 2. Load the code into the process memory.
 3. Prepare the loaded code for execution.
 4. Execute the code and wait for results.

Full speed ahead!

### Subvert control over a process

### Load executable code into memory

### Prepare the code for execution

### Transfer control to the new code
