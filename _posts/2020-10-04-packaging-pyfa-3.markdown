---
layout: post
title:  "Packaging Python apps for Debian (part 3)"
date:   2020-10-04 12:00:00 +0300
tags:   post development debian pyfa
---

[In the previous part](https://blog.ilammy.net/2020/09/27/packaging-pyfa-2.html)
we have managed to get **pyfa** package working.
Now we'll improve it a little bit.

...while still not playing the game.

### Cleaning up

First of all, let's clean up obvious things.

  - Removed unnecessary `debian/README.debian` file.

  - Replaced `debian/compat` file with `Build-Depends: debhelper-compat (= 12)` in `debian/control`.
    This is a new way to specify debhelper compatibility level.

  - Removed `Depends: ${shlibs:Depends}` in `debian/control`.
    Since our package is architecture-independent,
    it does not depend on native shared libraries either
    and does not need this macro.

  - Removed unused boilerplate comments in `debian/rules`,
    like the ones setting `DEB_BUILD_MAINT_OPTIONS`.
    Normally these are useful to tweak C compiler options but we don't need them for pyfa.

Then I moved on to filling other fields in `debian/control`:

```diff
-Section: unknown
+Section: misc
 Priority: optional
-Maintainer: ilammy <>
+Maintainer: Alexei Lozovsky <xxxxx@exaple.com>
 X-Python3-Version: >= 3.6
-Standards-Version: 4.1.4
-Homepage: <insert the upstream URL, if relevant>
+Standards-Version: 4.5.0
+Homepage: https://github.com/pyfa-org/pyfa
+Vcs-Browser: https://git.sr.ht/~ilammy/pyfa
+Vcs-Git: https://git.sr.ht/~ilammy/pyfa
```

as well as make the package architecture-independent:

```diff
 Package: pyfa
-Architecture: any
-Multi-Arch: foreign
+Architecture: all
```

`all` means that the package can be built for *all* architectures –
but still, has to be built for each individual architecture independently.
`any` means the package works on *any* architecture as is.

Most importantly, the `debian/changelog` file needs to be correct.

```diff
 pyfa (2.28.2-1) UNRELEASED; urgency=low

-  * Initial release. Closes: #nnnn
-    <nnnn is the bug number of your ITP>
+  * Initial release.

- -- ilammy <>  Sat, 19 Sep 2020 12:42:15 +0300
+ -- Alexei Lozovsky <xxxxx@exaple.com>  Sat, 19 Sep 2020 12:42:15 +0300
```

This file has [quite strict format](https://www.debian.org/doc/debian-policy/ch-source.html#s-dpkgchangelog)
and is the main source of versioning information for the package.

### Using lintian
