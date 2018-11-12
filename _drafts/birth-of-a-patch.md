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
Somebody eveb might be annoyed enough to write these issues down
or at least to voice their complaints.

That was the case for me:
I was shuffling through my inbox,
stumbled upon [this person](https://github.com/FreeRDP/FreeRDP/issues/4984#issue-377672013)
and felt sorry for them.

> [screenshot]

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
so I knew more or less where and what to look for.
We just need to look at some specs and implement a minor feature.

Initially I did not intend to fix that issue myself
so I left [a comment](https://github.com/FreeRDP/FreeRDP/issues/4984#issuecomment-436786068)
with some hints for whoever is going to tackle that issue.
It’s very important to leave this sort of message trail,
especially in open-source work done during you free time,
because of the immense help such notes provide
in case you flake and someone else has to deal with this later.

### Starting my work

Now comes the hard part (at least for me):
starting.
Once I’ve done that it’s infinitely easier to continue.
So I patiently wait for desire to get to the code.
This time it did not take long,
I was kinda frustrated with another project
and seemed to be a good idea to C some Rust off my X11 skills
(pun intended).

Make a fork, checkout a new branch, start hacking.
Sounds easy.
I like to name my first-contact branches with a `wip` prefix
so this gets called `wip/rail-icons`.
I know I’ll need an icon cache
so I build something like that,
improving it along the way.
At some point I want to test things out so I add some debug logging,
marking it as such so it's easy to remove later.
Then more code for color conversion follows,
with experiments, curses and TODOs all around it.

### Hacking around

I make rather crappy commits while developing and experimenting:

> [screenshot]

And this is fine because _they are not meant to go public_.
Some people would submit a code like that for review as soon as it works,
but I find it disgustingly uncultured practice.
This is my workplace,
I keep it comfortable to me.
But other developers do not need to know about my habits,
and they honestly should not care.
(“Hey, so why are you writing this post then?”)

Keeping TODOs in the code,
occasionally swearing,
marking commits with `[tags]`...
All this works for me
but maybe it won‘t for you.
You’re free to do anything you want as long as you’re comfortable with it,
just hide the ‘sausage making’.

Note, however, that while my commits seem to be incoherent
they are not made by simply binding Ctrl-S hotkey to `git commit`.
While I keep most of the information in my head
the commits are still self-contained pieces of changes.
Each of them introduces some _small_ atomic improvement.
It is important to keep them small but meaningful.
Commit early and often, do not postpone committing until the end of the day.
If you feel that you‘re not ready to commit then you won’t _ever_ be ready.

### Wrapping up for the day

It's important to have something working at the end of your hacking session.
This feeling of accomplishment keeps you motivated and preserves momentum.
In my case I've managed to get the feature more or less working.
At least on my machine.
In one environment.
Sometimes.

> [screenshot]

Now all that remains is to do more testing,
fix remaining known bugs
(there _will_ be bugs),
cleanup the code,
prepare the patches
and submit a pull request.

I like to leave incomplete tasks slightly broken by the end of the day.
This gives you a good anchor point to resume your work next day.
In this case I left the commits and remaining bugs as-is,
knowing that they will be the first thing I notice tomorrow.
Just don't forget to `git push` them to some remote repository.

### Keeping commits organized with `git rebase`

Remember that ugly stream of incoherent commits?
That's obviously _not pretty_.
If you keep pumping out commits like that
then you're digging your own grave
when the only remaining option will be
to squash them all into one giant ball of mud.
In order to avoid that I like to prune my git trees
at least once per day.

`git rebase --interactive` is an amazing tool for that.
It lets you rearrange, split and squash, and meld commits
into whatever shape you need.

Another tool that I use for rebasing is `gitk`.
Despite its simplistic Tcl/Tk interface,
it does a great job at displaying branch history
and all relevant information about commits.

> [screenshot]

So I start with [this branch tip](45daf77d08c55901f02ac42c7c2218dca0410345),
run `git rebase -i master`
and get the following commit list:

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

Then I stare at these commits in gitk,
review their contents
and think how they can be grouped up.
The following four groups seem to emerge:

- temporary debugging commits that need to be removed
- bug fixes in code that is not directly related to my feature
- mainaining the icon cache and interacting with RDP stack
- processing images and actually setting the icons

Which gives us the following rebase worklist:

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

Bamf! Save, quit, enjoy.
I don't bother with better commit messages for now,
it's good enough to just keep the commit count at bay
and get rid of some failed experiments I made on the way.
Plus, reviewing the commit messages reminds me what the hell did I do yesterday.

Also, I'm happy to avoid having to resolve any rebase conflict this time.
That's usually the case if you rebase and prune commits often.
However, if you neglect this care for too long then you're bound to encounter conflicts.

So, at the start of the next day
I'm at [this tip](1cac1cf5a6f6c9c1f8e256826660fa78332b30c7)
which contains only four commits.
I expect to do only bug fixes now
so it seems the final patch set will contain around this number of commits.

### Finishing up the code changes

Now, there were a few bugs that I'm aware of
and a couple that I discovered along the way.
This post is not a guide on debugging C code
and not a rant about hysterical raisins
which caused the engineers of DIB format
to make counterintuitive decisions about row order in bitmaps
and strange rounding of stride lengths.
Let's just leave it at that
I had to tweak some things
to make these icons:

> [screenshots]

look like actully normal ones:

> [screenshots]

After that the patch set is feature-complete
and I can start cleaning up the code.
Like, for example,
remove all the swearing from TODOs,
add actually useful comments,
make some code style changes
and remove temporary debug logs which are no longer needed.

All of this results in another bunch of commits
slapped on top of those four from the day before:

> [screenshot]

The content of the commits or their message do not really matter at this point.
The one thing that matters is that I'm happy with the final state of the code.
We'll clean up the commits later to make sure they are readable.
Now I can go and report the status [on GitHub](https://github.com/FreeRDP/FreeRDP/issues/4984#issuecomment-437681547)
to keep interested people updated on the progress.

### Preparing the patch set

### Submitting your work

### Conclusions

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
2018-11-12 21:30:00 - done the patch more or less

It was exceptionally hard to capture the exact process I follow
without degenerating into a raw terminal log.
This is because I do most of these actions automatically,
so it feels a bit like trying to explain
how to quickly type on a keyboard
to an adult that sees it for the first time.
