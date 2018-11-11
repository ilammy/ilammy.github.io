---
layout:   post
title:    "Making of Patch"
date:     2018-11-11 04:00:00 +0200
tags:     post philosophy
comments: true
---

> talk about how I make patches to public projects
> this is about open-source which is labor of love
> I strive to keep the same process at my day job

> start with an idea
> look at an issue
> devise a way to resolve it
> (experience helps here)
> leave some message trail behind, useful in case you fluke and someone else has to deal with this

> wait for it... the desire to do the deed

> fork it, make a branch
> i like wip/ prefixes

> start hacking
> i do rather crappy commit messages
> keep in my head
> and todos
> and [tags] in messages
> but note that changes are still self-contained, these are not commits created every N minutes

> see something working at the end of hacking session
> [post screenshots] - not *quite* working, but we'll fix bugs later
> get satisfaction and feeling of accomplishment
> don't forget to push

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
