---
layout:   post
title:    "Making of Patch"
date:     2018-11-11 04:00:00 +0200
tags:     post development git github
comments: true
---

Today I'm going to talk about how I make beautiful commits to public projects.
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
but other people's issues are fine too,
everybody likes when their problems go away.
If you have no idea then try looking for or in the issue tracker.
Usually the software we write is terrible,
it always has some issues.
Somebody might even be annoyed enough to write these issues down
or at least to voice their complaints.

That was the case for me:
I was shuffling through my inbox,
stumbled upon [this person](https://github.com/FreeRDP/FreeRDP/issues/4984#issue-377672013)
and felt sorry for them.

[![RAIL issue screenshot from FreeRDP's bugtracker](/images/2018-11-11/01-issue.png)](https://github.com/FreeRDP/FreeRDP/issues/4984)

For real,
FreeRDP should have had this feature years ago
but it doesn't.
Displaying remote application icons is important,
but _no one cared about it_
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
It's just a matter of reading some specs and implementing a minor feature.

Initially I did not intend to fix that issue myself
so I left [a comment](https://github.com/FreeRDP/FreeRDP/issues/4984#issuecomment-436786068)
with some hints for whoever is going to tackle that issue.
It's very important to leave this sort of message trail,
especially in open-source work done during you free time,
because of the immense help such notes provide
in case you flake and someone else has to deal with this later.

### Starting my work

Now comes the hard part (at least for me):
starting.
Once I've done that it's infinitely easier to continue.
So I patiently wait for desire to get to the code.
This time it did not take long,
I was kinda frustrated with another project
and seemed to be a good idea to C some Rust off my X11 skills
(pun intended).

Make a fork, checkout a new branch, start hacking.
Sounds easy.
I like to name my first-contact branches with a `wip` prefix
so this gets called `wip/rail-icons`.
I know I'll need an icon cache
so [I build something like that](https://github.com/ilammy/FreeRDP/commit/e44a3e1c29cb17e08ea2cff5f5e998905454ad2f),
[improving it](https://github.com/ilammy/FreeRDP/commit/8696117a0c478c825892f07919b20f19cdb6b564)
[along the way](https://github.com/ilammy/FreeRDP/commit/9f473d89cd63c7cf3a08c6155cbf3a71f3c3c5a8).
At some point I want to test things out so [I add some debug logging](https://github.com/ilammy/FreeRDP/commit/5a2c6359da9695640d7dd9c038cfb0125724adb3),
marking it as such so it's easy to remove later.
Then [more code for color conversion follows](https://github.com/ilammy/FreeRDP/commit/f2c70c7b068d8f8938cc04bacc389ddb5e54383e),
with [experiments](https://github.com/ilammy/FreeRDP/commit/f5dd034a5dbd389ab37a20baf9cd741c87c03226),
[curses](https://github.com/ilammy/FreeRDP/commit/c9c7f3f8813c2f5b0c8a417e8faed4997a0db604#diff-88e1759a07bca843ce780d306c5ae6b5R769)
and [TODOs](https://github.com/ilammy/FreeRDP/commit/93d984a0a231df0e423c4436a8e34c7add117f60)
all around it.

### Hacking around

I make rather crappy commits while developing and experimenting:

```console
$ git log --oneline --decorate --graph
* 45daf77d0 (HEAD -> wip/rail-icons) palette
* 42a05aa67 fix order reading
* 016ffb295 fix alpha
* cbff68a9f fix comment
* c9c7f3f88 better convesions
* f5dd034a5 pixel formats
* 444d8f301 icons are appended
* 93d984a0a todo
* 295fa4165 setting icons
* f2c70c7b0 color conversion
* 9355a0be8 [debug] flush on output
* e7fdb6295 [debug] output
* ec352a211 cache: actually, check 0xFF first
* 5a2c6359d [debug] output
* 60fbab6d2 using icon caches
* 77012fff3 introduce scratch icon
* f3a070680 rename pixel field
* 9f473d89c icon cache values, cleanup and lookup
* 8696117a0 allocating caches
* e44a3e1c2 prepare icon cache structs
*   c5c1bac31 (upstream/master) Merge pull request #4960 from akallabeth/interleaved_fix
|\
| * fff2454ae Make VS2010 happy, reworked UNROLL defines.
```

And this is fine because _they are not meant to go public_.
Some people would submit a code like that for review as soon as it works,
but I find it disgustingly uncultured practice.
That's like when you come to a restaurant
and instead of getting your food served on a plate by a waiter
you are invited to eat the meal straight from the oven in the kitchen.
I like keeping notes in TODOs when I work on the code,
occasionally swearing in them.
I split the commits into very small parts
so I can get back to them later,
or revert to a particular state of the code.
I use [debug] tags for commits that need to be removed later.
You're free to do anything you want
as long as you're comfortable with it,
and you do it privately
without involving other people into it
(such as your code reviewers).

Note, however, that while my commits seem to be incoherent
they are not made by simply binding Ctrl-S hotkey to `git commit`.
While I keep most of the information in my head
the commits are still self-contained pieces of changes.
Each of them introduces some _small_ atomic improvement.
It is important to keep them small but meaningful.
Commit early and often,
do not postpone it until committing at the end of the day.

### Wrapping up for the day

It's important to have something working at the end of your hacking session.
This feeling of accomplishment keeps you motivated and preserves momentum.
In my case I've managed to get the icons somewhat displayed.
At least on my machine.
In one environment.
Sometimes.
If you know where to look.

![Almost-working icon of notepad.exe](/images/2018-11-11/02-notepad.png)

Now all that remains is to do more testing,
fix some bugs
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
In order to avoid that I prune my git trees at least once per day.

`git rebase --interactive` is an amazing tool for that.
It lets you rearrange, split and squash, and meld commits
into whatever shape you need.

Another tool that I use for rebasing is `gitk`.
Despite its simplistic Tcl/Tk interface,
it does a great job at displaying branch history
and all relevant information about commits.

![Viewing a branch history via gitk](/images/2018-11-11/03-gitk-branch.png)

So I start with [this tree](https://github.com/ilammy/FreeRDP/commits/45daf77d08c55901f02ac42c7c2218dca0410345).
Then I stare at commits in gitk,
review their contents
and think how they can be grouped up.
The following four groups seem to emerge:

- temporary debugging commits that need to be removed
- bug fixes in code that is not directly related to my feature
- mainaining the icon cache and interacting with RDP stack
- processing images and actually setting the icons

To regroup the commits I run `git rebase -i master`
and edit the rebase worklist like this,
constructing four commits out of that mess:

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

`:wq`, enjoy.
I don't bother with better commit messages for now,
it's good enough to just keep the commit count at bay
and get rid of some failed experiments I made on the way.
Plus, reviewing the commit messages reminds me what the hell did I do yesterday.

Also, I'm happy to avoid having to resolve any rebase conflicts this time.
That's usually the case if you rebase and prune commits often.
However, if you neglect this care for too long then you're bound to encounter conflicts.

So I start my next coding session with [this tree](https://github.com/ilammy/FreeRDP/commits/1cac1cf5a6f6c9c1f8e256826660fa78332b30c7)
which contains only four commits.
I expect to do only bug fixes now
so it seems the final patch set will contain around this number of commits.

### Finishing up the code changes

Now, there were a few bugs that I'm aware of
and a couple that I discovered along the way.
This post is not a guide on debugging C code
and not a rant about hysterical raisins
which caused the engineers of DIB format
to make counterintuitive decisions.
Let's just say that
I had to tweak some things
to make these icons:

![Icons with color corruptions](/images/2018-11-11/04-broken-icons.png)

look like actully normal ones:

![Correctly displayed icons](/images/2018-11-11/05-right-icons.png)

After that the patch set is feature-complete
and I can start cleaning up the code.
Like, for example,
[remove all the swearing from TODOs](https://github.com/ilammy/FreeRDP/commit/5fcf43b59f1effc416c738c3d57e8d3112cba011),
add [actually useful comments](https://github.com/ilammy/FreeRDP/commit/7000dbd0d52e2bc6eae7c949161942ea2caa96d5),
[make some code style changes](https://github.com/ilammy/FreeRDP/commit/14ce1752b7f5bbf0e95522fbd7c7b6f86739f0ed)
and [remove temporary debug logs which are no longer needed](https://github.com/ilammy/FreeRDP/commit/2d39ffc5f31a835856279e4c96c7f81be32bc7c1).

All of this results in another bunch of commits
slapped on top of those four from the day before:

```console
* 2dd6bd7d6 (HEAD -> wip/rail-icons) add xflush for icon set
* e7ad0303e drop dead code
* eecba32bc spec reference
* 0e466bd8b drop comment just use i in the commit message:
* ff1d47d84 return values
* 564490a45 code style
* 4aa3638fa drop more debug logs
* 2d39ffc5f drop debug logs
* 7803a526c dont use malloc
* d3541cba1 improve naming
* 5e07c098f improve naming
* 14ce1752b simplify error handling
* 7000dbd0d cleanup comments, lower amount of smug in them
* 5fcf43b59 use functions instead of swearing about formatting
* 31d9bdfd8 fix 8-bit palettes (RGBQUAD does not include alpha)
* b8f027d6f alpha bitmask should actually be a *mask*
* db30910fb better explanation
* 1dbc6dd55 fun bugs with color formats
* f3c5008fe fix a bug with 16x16 icons
* 1cac1cf5a (origin/wip/rail-icons) color conversion
* 2d6c8d825 prepare icon cache structs
* 3619a2c6a fix order reading
* a9d40cb18 [debug] flush on output
*   c5c1bac31 (upstream/master) Merge pull request #4960 from akallabeth/interleaved_fix
|\
| * fff2454ae Make VS2010 happy, reworked UNROLL defines.
```

The content of the commits or their message do not really matter at this point.
The one thing that matters is that I'm happy with the final state of the code.
We'll clean up the commits later to make sure they are readable.
Now I can go and report the status [on GitHub](https://github.com/FreeRDP/FreeRDP/issues/4984#issuecomment-437681547)
to keep interested people updated on the progress.

### Preparing the patch set

Once again,
I start the day with a bunch of commits from yesterday
which I want to regroup and squash,
melding the bug fixes with new code
as if I never made those mistakes.

![Example of gitk highlighting](/images/2018-11-11/06-gitk-log-s.png)

gitk can highlight commits
that add or remove a particular string
(like `git log -S` from the terminal),
it helps identifying commits that can be grouped.
I end up with the following rebase worklist.
Note that I remove the debugging commit
and change the relative order of some of the commits.

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

This time the rebase did not apply cleany and caused merge conflicts.
Commits `5e07c09 improve naming` and `ff1d47d return values`
failed to apply because some functions were renamed.
These kinds of conflicts are easy to resolve:
you look at the conflicting commit,
remember its intent,
and try redoing the same change to the old code.
Keep in mind that
fixes you do can cause more conflicts in the following commits.
Merge conflicts are bound to happen
if you reorder patches during an interactive rebase.
That's why you should prune commits early and often,
lowering the possibility of conflicts in the future.

I end up with the following [tree](https://github.com/ilammy/FreeRDP/commits/9536a2c7a96f9ef9c1293cba1203b4e8a20a7cba)
after the rebase.
It's always a good idea to retest your code after rebasing,
especially after resolving merge conflicts.
You can also run a `git diff` between pre- and post-rebase states
to make sure that you did not drop something you did not intend to.

### Editing patches

When I feel that the patch set is ready
I focus my attention on the diffs those patches introduce.
Each patch should be atomic and solve one specific task.
In my case I implement a new feature
so I'd expect the diffs to be mostly green (additions).

I review the commits I currently have
and see what can be improved in them:

- [fix order reading](https://github.com/ilammy/FreeRDP/commit/1eeff8968a0a5c98058c2f47e355f3e0de40f845)
- [prepare icon cache structs](https://github.com/ilammy/FreeRDP/commit/91036c6fa5c7737e01c83f69d190e3f6feeab3dc)
- [color conversion](https://github.com/ilammy/FreeRDP/commit/9536a2c7a96f9ef9c1293cba1203b4e8a20a7cba)

The first one looks good:
it's an easy bug fix.
The second one also seems okay:
nice, self-contained additions.
The last commit is also mostly good
but there are some weird diff hunks in it.
It seems that some these changes should be a part of the second commit.

Here we see changes in `struct xf_rail_icon`
which is introduced in the previous commit.
It's an artifact of a failed experiement.
This struct can be defined correctly from the start.

```diff
@@ -63,9 +64,8 @@ static const char* movetype_names[] =

 struct xf_rail_icon
 {
-        UINT16 width;
-        UINT16 height;
-        BYTE* argbPixels;
+        long *data;
+        int length;
 };
 typedef struct xf_rail_icon xfRailIcon;
```

Then there's this weird diff
which is actually another artifact of renaming `xf_rail_convert_icon()` to `convert_rail_icon()`.
Let's use the correct name right away.

```diff
@@ -611,15 +611,206 @@ static xfRailIcon* RailIconCache_Lookup(xfRailIconCache* cache,
         return &cache->entries[cache->numCacheEntries * cacheId + cacheEntry];
 }

-static void xf_rail_convert_icon(ICON_INFO* iconInfo, xfRailIcon *railIcon)
+static inline UINT32 read_color_quad(const BYTE* pixels)
+{
+        return (((UINT32) pixels[0]) << 24)
+             | (((UINT32) pixels[1]) << 16)
+             | (((UINT32) pixels[2]) << 8)
+             | (((UINT32) pixels[3]) << 0);
+}
...
```

Here's some debug logging that was not removed in time.
It is introduced in the previous commit but it's not needed anymore.

```diff
+static inline UINT32 round_up(UINT32 a, UINT32 b)
+{
+        return b * div_ceil(a, b);
+}
+
+static void apply_icon_alpha_mask(ICON_INFO* iconInfo, BYTE* argbPixels)
 {
-        WLog_DBG(TAG, "convert icon: cacheEntry=%u cacheId=%u bpp=%u width=%u height=%u",
-                iconInfo->cacheId, iconInfo->cacheEntry, iconInfo->bpp, iconInfo->width, iconInfo->height);
+        BYTE nextBit;
+        BYTE* maskByte;
+        UINT32 x, y;
+        UINT32 stride;
+
+        if (!iconInfo->cbBitsMask)
+                return;
```

Finally,
here we add an argument to the function
which is first added in the previous commit.
We can define the stub correctly earlier.

```diff
 static void xf_rail_set_window_icon(xfContext* xfc,
-                                    xfAppWindow* railWindow, xfRailIcon *icon)
+                                    xfAppWindow* railWindow, xfRailIcon *icon,
+                                    BOOL replace)
```

```diff
-        xf_rail_set_window_icon(xfc, railWindow, icon);
+        replaceIcon = !!(orderInfo->fieldFlags & WINDOW_ORDER_STATE_NEW);
+        xf_rail_set_window_icon(xfc, railWindow, icon, replaceIcon);
```

All these changes rectify diffs
which effectively revert changes made by the previous diffs.
What would your code reviewer think if they saw such patches?
“Hm... weird...”
That's exactly how you pinpoint bad spots in your patches.

So how do I iron out these wrinkles?
That's a job for `git rebase --interactive` again!
I start with `git rebase -i master`
and edit the worklist like this:

```
pick 1eeff89 fix order reading
edit 91036c6 prepare icon cache structs
pick 9536a2c color conversion
```

This tells git to stop at commit `91036c6` and let us edit it.
I won't edit it right away but rather insert some changes
between `91036c6` and `9536a2c` that follows it.
If `9536a2c` were smaller then splitting it directly with `git reset`
(as [described](https://git-scm.com/docs/git-rebase#_splitting_commits) in manpage for rebase)
might have been easier to do,
but in this case I won't do it.

I stop at `91036c6` and re-do the changes from `9536a2c` on top of that commit.
I use `git commit --fixup=91036c6` to create specially named commits
which are compatible with `--autosquash` option of `rebase`.
(I like to add comments to fixup commits to note what the fix.
You can do that with an `--edit` flag.)

> By the way, did I tell about the awesome `git add -p` mode
> where you can review and select the patch hunks that go into commit?
> Now you know.

After I'm done I do `git rebase --continue` to resume rebasing
and reapply `9536a2c` on top of split changes.
As expected, the patch does not apply cleanly,
there are some conflicts but they are rather minor.
Fix them, `git add` then `git rebase --continue` again.
I end up with
[this new commit](https://github.com/ilammy/FreeRDP/commit/5a97269f95d82220ace055d2926493a6e2b83e5f)
which looks much nicer.

Now I do a final `git rebase -i --autosquash master`.
Note how `fixup!` commits are already in the right place:

```
pick 1eeff89 fix order reading
pick 91036c6 prepare icon cache structs
fixup f36c0e8 fixup! prepare icon cache structs
fixup a715365 fixup! prepare icon cache structs
fixup b696670 fixup! prepare icon cache structs
fixup ff5e5f4 fixup! prepare icon cache structs
pick 5a97269 color conversion
```

And here we have our complete patch set.
I usually review the patches once again to make sure that I like the diffs.
It's also a good moment to re-test your work,
just in case you messed up during all the rebases.
Another thing that I can recommend testing is _bisectability_.
Make sure that each of your commits at least compiles and passes unit-tests.

Now it's time to do something about these awful commit messages.

### Writing good commit messages

I use `git rebase -i` again to change commit messages
(yes, it's an amazing multitool).
This time the `reword` command is used in rebase worklist:

```
reword 1eeff89 fix order reading
reword aec29ec prepare icon cache structs
reword 5c2a1f4 color conversion
```

After which git opens a text editor three times to ask for new commit messages.

How does a good commit message look like?
Chris Beams [has summarized this for me](https://chris.beams.io/posts/git-commit/)
so I won't repeat the points which are made over and over.
In short, use the following format:

```
Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequences of this
change? Here's the place to explain them.

Further paragraphs come after blank lines.

 - Bullet points are okay, too

 - Typically a hyphen or asterisk is used for the bullet, preceded
   by a single space, with blank lines in between, but conventions
   vary here

If you use an issue tracker, put references to them at the bottom,
like this:

Resolves: #123
See also: #456, #789
```

You can see how this advice is applied in the commit messages I came up with:

- [libfreerdp-core: fix reading TS_ICON_INFO](https://github.com/ilammy/FreeRDP/commit/158ac195a343936c43a924544776fc51eb2dd478)
- [xfreerdp: add RAIL icon cache](https://github.com/ilammy/FreeRDP/commit/d46bde785c5ea0ee32b1a0c49002b61f60b767d5)
- [xfreerdp: set \_NET_WM_ICON to RAIL app icon](https://github.com/ilammy/FreeRDP/commit/530df917c2e534d9defff9bf0aaa068d98221b59)

Each of them describes the changes,
provides context for them,
and explains some non-obvious quirks.

The formatting and line-breaking are quite important
so that `git log` output looks nice:

```console
$ git show 158ac195a
commit 158ac195a343936c43a924544776fc51eb2dd478
Author: ilammy <a.lozovsky@gmail.com>
Date:   Sat Nov 10 22:09:20 2018 +0200

    libfreerdp-core: fix reading TS_ICON_INFO

    The spec says that CbColorTable field is present when Bpp is 1, 4, 8.
    Actually, bpp == 2 is not supported by TS_ICON_INFO according to the
    spec (though, DIB definitely supports 16-color images).

        MS-RDPERP 2.2.1.2.3 Icon Info (TS_ICON_INFO)

        CbColorTable (2 bytes):
            This field is ONLY present if the bits per pixel (Bpp)
            value is 1, 4, or 8.

    Omitting 8-bit value breaks 256-color icons which are incorrectly
    read with color and alpha data mixed up.

diff --git a/libfreerdp/core/window.c b/libfreerdp/core/window.c
index a9b3c00b8..74281a6ff 100644
--- a/libfreerdp/core/window.c
+++ b/libfreerdp/core/window.c
@@ -86,8 +86,8 @@ BOOL update_read_icon_info(wStream* s, ICON_INFO* iconInfo)
        Stream_Read_UINT16(s, iconInfo->width); /* width (2 bytes) */
        Stream_Read_UINT16(s, iconInfo->height); /* height (2 bytes) */

-       /* cbColorTable is only present when bpp is 1, 2 or 4 */
-       if (iconInfo->bpp == 1 || iconInfo->bpp == 2 || iconInfo->bpp == 4)
+       /* cbColorTable is only present when bpp is 1, 4 or 8 */
+       if (iconInfo->bpp == 1 || iconInfo->bpp == 4 || iconInfo->bpp == 8)
        {
                if (Stream_GetRemainingLength(s) < 2)
                        return FALSE;
```

### Submitting a pull request

Finally it's time to make a pull request.
In this case
I'm implementing a simple feature to solve a specific issue
so the description can be quite basic
and just reference the issue.
It's also a visual feature so I can add an eye-catcher image.

[![Submitted pull request #5003](/images/2018-11-11/07-pr.png)](https://github.com/FreeRDP/FreeRDP/pull/5003)

Always describe your work
and why you think it would be a good addition to the project.
Read and respect
the project's contribution guidelines
[(here are ones for FreeRDP)](https://github.com/FreeRDP/FreeRDP/blob/master/.github/PULL_REQUEST_TEMPLATE.md)
before submitting a pull request.
These rules are there for a reason.

After creating a pull request
I've also asked the person having an issue to test the fix,
which they gladly did and confirmed that it works for them too.

### Working with maintainers

Soon the code is going to be reviewed by project's maintainers.
It's very likely to receive feedback
and some change requests.
In my case these were mostly nitpicks and some clarifications
which I gladly replied to.
Furthermore,
the maintainer (Armin) was exceptionally nice
and fixed all the nitpicks himself.

I'm also very glad that my humor was not wasted.
Code comments are a vessel for oral tradition
and they a good opportunity to occasionally have fun and make fun,
for example,
of stupid things we have to do for the sake of backwards compatibility.
Don't be so serious.

### Conclusions

Yay!
Now I have (or rather, maintain) this nice [Contributor] badge on GitHub:

![Comment with a contributor badge](/images/2018-11-11/08-thanks.png)

Hopefully, the world is on a way to be a better place now.
One reason for that is the pull request discussed here
which hopefully makes FreeRDP a bit nicer.
The other reason is you,
who hopefully have read until this point
and learned yourself a trick or two about **git**.
Please use this knowledge wisely
to increase the quality of your submissions.

Cheerio~

[![Yuyuko picture by @sasamorichou](/images/2018-11-11/09-yuyuko.png)](https://twitter.com/sasamorichou/status/1054729930683138050)

<hr/>

2018-11-10 13:00:00 - started work on a patch
2018-11-11 03:50:00 - idea of a post
2018-11-11 04:05:00 - draft of points
2018-11-12 21:30:00 - done the patch more or less
2018-11-13 00:15:00 - PR submitted
2018-11-14 22:00:00 - almost completed draft post

<hr/>

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
