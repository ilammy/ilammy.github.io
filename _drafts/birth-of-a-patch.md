---
layout:   post
title:    "Making of Patch"
date:     2018-11-11 04:00:00 +0200
tags:     post development git github
comments: true
---

Today I’m going to talk about how I make beautiful commits to public projects.
I will be focusing more on matters and processes around **git** usage,
rather than technical details of the contribution
or social aspects of interacting with random people over Internet.

Note that while this will be about an open-source project
— which is definitely a labor of love for me —
I strive to maintain the same process at my day job as well.
Quality is rarely a matter of budget constraints,
_actually caring_ about it and setting the bar high enough
is way more important.

### How it begins

You start off with an issue.
Ideally, one that affects yourself
(aka scratch your own itch)
but other people’s issues are fine too,
everybody likes when their problems go away.
If you have no idea then try looking for or in the issue tracker.
Usually the software we write is terrible,
it always has some issues.
And somebody might be annoyed enough to write these issues down
or at least to voice their complaints.
That was the case for me:
I was shuffling through my inbox,
stumbled upon [this person](https://github.com/FreeRDP/FreeRDP/issues/4984#issue-377672013)
and felt sorry for them.
For real,
FreeRDP should have had this feature years ago
but it doesn’t.
Because _no one cared about it_
aside from that person.
Do _you_ care about it?
Are you bad enough to rescue the President?

Okay, you have your problem,
now we need a solution.
Experience definitely helps here
as solving similar problems should be easier next time you have them.
Thankfully,
I do have enough experience with FreeRDP and X11 under my belt
so I knew where and what to look for.
(If that wasn’t the case then the journey would be even more fun.)
Initially I did not intend to fix that issue myself
so I left [a comment](https://github.com/FreeRDP/FreeRDP/issues/4984#issuecomment-436786068)
with some hints for whoever is going to tackle that issue.
It’s very important to leave this sort of message trail,
especially in open-source work done during you free time,
because of the immense help such notes provide
in case you flake and someone else has to deal with this.

### Starting your work

> wait for it... the desire to do the deed

> fork it, make a branch
> i like wip/ prefixes

### Hacking around

> start hacking
> i do rather crappy commit messages
> keep in my head
> and todos
> and [tags] in messages
> but note that changes are still self-contained, these are not commits created every N minutes

### Wrapping up for the day

> see something working at the end of hacking session
> [post screenshots] - not *quite* working, but we'll fix bugs later
> get satisfaction and feeling of accomplishment
> don't forget to push

### Keeping commits organized

45daf77d08c55901f02ac42c7c2218dca0410345 - 2018-11-10-end - start
1cac1cf5a6f6c9c1f8e256826660fa78332b30c7 - 2018-11-11-begin - end

start with this:

```
pick e44a3e1 prepare icon cache structs
pick 8696117 allocating caches
pick 9f473d8 icon cache values, cleanup and lookup
pick f3a0706 rename pixel field
pick 77012ff introduce scratch icon
pick 60fbab6 using icon caches
pick 5a2c635 [debug] output
pick ec352a2 cache: actually, check 0xFF first
pick e7fdb62 [debug] output
pick 9355a0b [debug] flush on output
pick f2c70c7 color conversion
pick 295fa41 setting icons
pick 93d984a todo
pick 444d8f3 icons are appended
pick f5dd034 pixel formats
pick c9c7f3f better convesions
pick cbff68a fix comment
pick 016ffb2 fix alpha
pick 42a05aa fix order reading
pick 45daf77 palette
```

first step is to just bunch up changes, split into three changesets:

- patching bugs in 'external' code
- mainaining icon cache and interacting with RDP
- processing and setting icons

leave with this file:

```
pick 9355a0b [debug] flush on output

pick 42a05aa fix order reading

pick e44a3e1 prepare icon cache structs
s 8696117 allocating caches
s 9f473d8 icon cache values, cleanup and lookup
s f3a0706 rename pixel field
s 77012ff introduce scratch icon
s 60fbab6 using icon caches
s 5a2c635 [debug] output
s ec352a2 cache: actually, check 0xFF first
s e7fdb62 [debug] output

pick f2c70c7 color conversion
s 295fa41 setting icons
s 93d984a todo
s 444d8f3 icons are appended
s f5dd034 pixel formats
s c9c7f3f better convesions
s cbff68a fix comment
s 016ffb2 fix alpha
s 45daf77 palette
```

and that's more of less clean, leave it
just check in gitk that changes kinda make sense and keep commit count at bay
(it just happens that i don't need to remove changes of resolve conflicts)

> next day make a cleanup
> don't leave *too* long stream of incoherent commits, it is hard to rebase
> still don't bother with messages, just prune the intermediate changes and failed experiments

### Getting through it

### Submitting your work

### xxx

next:

> iterate development
> review patches, rehash changes between them
> make good commit messages
> what good messages are
> test your changes
> test bisectability
> make a PR
> write a good PR message
> wait for maintainers
> fix issues
> now you have a nice [contributor] badge, yay!

2018-11-11 03:50:00 - idea of a post
2018-11-11 04:05:00 - draft of points
