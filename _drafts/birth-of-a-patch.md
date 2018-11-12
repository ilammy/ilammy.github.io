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
s    8696117 allocating caches
s    9f473d8 icon cache values, cleanup and lookup
s    f3a0706 rename pixel field
s    77012ff introduce scratch icon
s    60fbab6 using icon caches
s    5a2c635 [debug] output
s    ec352a2 cache: actually, check 0xFF first
s    e7fdb62 [debug] output

pick f2c70c7 color conversion
s    295fa41 setting icons
s    93d984a todo
s    444d8f3 icons are appended
s    f5dd034 pixel formats
s    c9c7f3f better convesions
s    cbff68a fix comment
s    016ffb2 fix alpha
s    45daf77 palette
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

> patching

from

```
pick a9d40cb [debug] flush on output
pick 3619a2c fix order reading
pick 2d6c8d8 prepare icon cache structs
pick 1cac1cf color conversion
pick f3c5008 fix a bug with 16x16 icons
pick 1dbc6dd fun bugs with color formats
pick db30910 better explanation
pick b8f027d alpha bitmask should actually be a *mask*
pick 31d9bdf fix 8-bit palettes (RGBQUAD does not include alpha)
pick 5fcf43b use functions instead of swearing about formatting
pick 7000dbd cleanup comments, lower amount of smug in them
pick 14ce175 simplify error handling
pick 5e07c09 improve naming
pick d3541cb improve naming
pick 7803a52 dont use malloc
pick 2d39ffc drop debug logs
pick 4aa3638 drop more debug logs
pick 564490a code style
pick ff1d47d return values
pick 0e466bd drop comment just use i in the commit message:
pick eecba32 spec reference
pick e7ad030 drop dead code
pick 2dd6bd7 add xflush for icon set
```

to

```
#### a9d40cb [debug] flush on output

pick 3619a2c fix order reading
s    eecba32 spec reference

pick 2d6c8d8 prepare icon cache structs
s    5e07c09 improve naming
s    d3541cb improve naming
s    4aa3638 drop more debug logs
s    564490a code style
s    ff1d47d return values
s    0e466bd drop comment just use i in the commit message:

pick 1cac1cf color conversion
s    f3c5008 fix a bug with 16x16 icons
s    1dbc6dd fun bugs with color formats
s    db30910 better explanation
s    b8f027d alpha bitmask should actually be a *mask*
s    31d9bdf fix 8-bit palettes (RGBQUAD does not include alpha)
s    5fcf43b use functions instead of swearing about formatting
s    7000dbd cleanup comments, lower amount of smug in them
s    14ce175 simplify error handling
s    7803a52 dont use malloc
s    2d39ffc drop debug logs
s    e7ad030 drop dead code
s    2dd6bd7 add xflush for icon set
```

gitk definitely helps here,
with its abilility to highligth commits
that mention a string

this time the rebase caused some conflits, but they were easy to solve
patch 5e07c098f41a2d73cd047fda8802810097711ea5 did not apply cleanly:
the rename from xf_rail_window_set_icon() to xf_rail_set_window_icon()
caused some unrelated context change which were not there,
so I had to resolve the conflict by redoing the renaming in the old code
(as if the function has been named correctly from the start)
and then fix the rippled conflicts in the newer patches which were
made without the rename

and another one when applying ff1d47d8460f5c04d28dac1c75923172458ba305
which was caused mainly by xf_rail_convert_icon() having a new name

this can always happend if you reorder patches during interactive rebase,
when resolving conflicts always review the patch that did not apply
and fix the code so that it does the same thing the patch intended to do

so I ended up with 9536a2c7a96f9ef9c1293cba1203b4e8a20a7cba after rebase

after you resolve some conflicts it's important to test your code
and review patches as git rebase might have done something wrong

it's also a good idea to run a `git diff` before and after the rebase
to make sure that at least the final state of the code is identical
and you did not lose some imporant part in the midway

reviewing the patches:
- 1eeff8968a0a5c98058c2f47e355f3e0de40f845
- 91036c6fa5c7737e01c83f69d190e3f6feeab3dc
- 9536a2c7a96f9ef9c1293cba1203b4e8a20a7cba

the first one is easy
the second one looks good: almost all additions, self-contained,

the third one is fine, but I'd move some lines in the previous commit
because these diffs look ugly:

```
@@ -63,9 +64,8 @@ static const char* movetype_names[] =

 struct xf_rail_icon
 {
-	UINT16 width;
-	UINT16 height;
-	BYTE* argbPixels;
+	long *data;
+	int length;
 };
 typedef struct xf_rail_icon xfRailIcon;
```

where `struct xf_rail_icon` is introduced in the previous commit,
it should be defined correctly from the start
(+ related changes, like free()ing `data` instead of `argbPixels`)

```
@@ -611,15 +611,206 @@ static xfRailIcon* RailIconCache_Lookup(xfRailIconCache* cache,
 	return &cache->entries[cache->numCacheEntries * cacheId + cacheEntry];
 }

-static void xf_rail_convert_icon(ICON_INFO* iconInfo, xfRailIcon *railIcon)
+static inline UINT32 read_color_quad(const BYTE* pixels)
+{
+	return (((UINT32) pixels[0]) << 24)
+	     | (((UINT32) pixels[1]) << 16)
+	     | (((UINT32) pixels[2]) << 8)
+	     | (((UINT32) pixels[3]) << 0);
+}
+
+/*
+ * DIB color palettes are arrays of RGBQUAD structs with colors in BGRX format.
+ * They are used only by 1, 2, 4, and 8-bit bitmaps.
+ */
+static void fill_gdi_palette_for_icon(ICON_INFO* iconInfo, gdiPalette *palette)
+{
...
```

this weird diff is an artifact of renaming xf_rail_convert_icon()
to convert_rail_icon(), let's use the correct name from the start

```
+static inline UINT32 round_up(UINT32 a, UINT32 b)
+{
+	return b * div_ceil(a, b);
+}
+
+static void apply_icon_alpha_mask(ICON_INFO* iconInfo, BYTE* argbPixels)
 {
-	WLog_DBG(TAG, "convert icon: cacheEntry=%u cacheId=%u bpp=%u width=%u height=%u",
-		iconInfo->cacheId, iconInfo->cacheEntry, iconInfo->bpp, iconInfo->width, iconInfo->height);
+	BYTE nextBit;
+	BYTE* maskByte;
+	UINT32 x, y;
+	UINT32 stride;
+
+	if (!iconInfo->cbBitsMask)
+		return;
```

this random debug log removal in the middle of a bunch of new functions,
let's remove the log (added in previous commit)

```
 static void xf_rail_set_window_icon(xfContext* xfc,
-                                    xfAppWindow* railWindow, xfRailIcon *icon)
+                                    xfAppWindow* railWindow, xfRailIcon *icon,
+                                    BOOL replace)
```

```
-	xf_rail_set_window_icon(xfc, railWindow, icon);
+	replaceIcon = !!(orderInfo->fieldFlags & WINDOW_ORDER_STATE_NEW);
+	xf_rail_set_window_icon(xfc, railWindow, icon, replaceIcon);
```

this addition of an argument to a function stub
that does nothing in the previous commit
let's add it right away

Always think from the standpoint of code reviewer.
What would they think if they saw this patch?

So how do I reorder these changes?
`git rebase --interactive` again.
We start with `git rebase -i master`
and edit the commit list like this:

```
pick 1eeff89 fix order reading
edit 91036c6 prepare icon cache structs
pick 9536a2c color conversion
```

This tells git to stop at commit `91036c6` and let us edit it.
We would not edit it right away but rather insert some changes
between `91036c6` and `9536a2c` that follows it.
If `9536a2c` were smaller then splitting it directly with `git reset`
(as described in manpage for rebase) might have been easier to do,
but in this case we won't do it.

So we stop at `91036c6`
and re-do the changes from `9536a2c`
on top of that commit.
We use `git commit --fixup=91036c6` to create specially named commits
which are compatible with `--autosquash` option of `rebase`.

xxx: ff5e5f43d26944318fbd2d652a5e996d3f396c16

> [screenshot]

Note that I like adding comments in fixup commits to note what they fix.

Also,
did I tell about the awesome `git add -p` mode
where you can review and select the patch hunks that go into commit?
Now you know.

After we're done
we do `git rebase --continue` to resume rebasing
and reapply `9536a2c` on top of our work.
As expected, the patch does not apply cleanly.
We have to fix the conflicts, but they are rather minor.
Fix them, `git add`, `git rebase --continue` again.
We end up with this new commit which looks much nicer.

xxx: 5a97269f95d82220ace055d2926493a6e2b83e5f

Now we do a final `git rebase -i --autosquash master`,
immediately see the correct commit list:

```
pick 1eeff89 fix order reading
pick 91036c6 prepare icon cache structs
fixup f36c0e8 fixup! prepare icon cache structs
fixup a715365 fixup! prepare icon cache structs
fixup b696670 fixup! prepare icon cache structs
fixup ff5e5f4 fixup! prepare icon cache structs
pick 5a97269 color conversion
```

And here we have out complete patch set.
I usually review the patches once again to make sure that I like the diffs.
Now it's time to do something about these awful commit messages.

It's also a good moment to re-test your work,
just in case you messed up that rebase.

I absolutely love to juggle the changes between commits like this.

### Writing good commit messages

you do `git rebase -i` again (yes, it's an amazing multitool)

```
reword 1eeff89 fix order reading
reword aec29ec prepare icon cache structs
reword 5c2a1f4 color conversion
```

and start writing
this is what i came up with

- 1
- 2
- 3


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
I had to monitor myself very closely,
try to not cut corners,
remember to not get carried away hacking
and to write down some thoughts at the same time.
If it wasn't for this idea of a post
I would have probably coded and submitted the pull request on the same evening.
But now you may see from commit timestamps how much time it actually took.
