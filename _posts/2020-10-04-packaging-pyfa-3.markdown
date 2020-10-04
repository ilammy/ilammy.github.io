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

### Using Lintian

**Lintian** is a linter for Debian packages.
It makes hunderds of checks for policy violations, best practices, common pitfalls, etc.
It's highly recommended for maintainers to use it (though not a requirement).

Lintian can work with binary packages (`*.deb`) as well as source ones (`*.dsc`).
Here's what it outputs for our binary package at the moment:

```
$ lintian ../pyfa_2.28.2-1_all.deb
E: pyfa: copyright-file-contains-full-gpl-license
W: pyfa: binary-without-manpage usr/bin/pyfa
W: pyfa: copyright-has-url-from-dh_make-boilerplate
W: pyfa: executable-not-elf-or-script usr/share/pyfa/service/port/efs.py
W: pyfa: new-package-should-close-itp-bug
```

More information is available with `--info`:

```
$ lintian --info ../pyfa_2.28.2-1_all.deb
[...]
W: pyfa: new-package-should-close-itp-bug
N:
N:    This package appears to be the first packaging of a new upstream
N:    software package (there is only one changelog entry and the Debian
N:    revision is 1), but it does not close any bugs. The initial upload of a
N:    new package should close the corresponding ITP bug for that package.
N:
N:    This warning can be ignored if the package is not intended for Debian or
N:    if it is a split of an existing Debian package.
N:
N:    Refer to Debian Developer's Reference section 5.1 (New packages) for
N:    details.
N:
N:    Severity: warning
N:
N:    Check: debian/changelog
N:
```

Built-in descriptions are typically enough to understand the issues with the package.

Let's sum up outstanding ones:

  - `copyright-file-contains-full-gpl-license` and `copyright-has-url-from-dh_make-boilerplate`

    These stem from the fact that `debian/copyright` is still a boilerplate.
    We'll get back to fixing that file soon.

  - `binary-without-manpage usr/bin/pyfa`

    The policy requires each top-level binary to have a manpage.
    I.e., if you can launch something as `foo` via shell, `man foo` must be available.
    We'll write a simple manpage to check this item off the list.

  - `executable-not-elf-or-script usr/share/pyfa/service/port/efs.py`

    This seems to an artifact of the upstream repository which has some files executable
    when they are not intended to be executable.
    This can be easily fixed by the maintainer scripts:

    ```make
    execute_after_dh_install:
            chmod -x debian/pyfa/usr/share/pyfa/service/port/efs.py
    ```

  - `new-package-should-close-itp-bug`

    Each new package in Debian should close the 'ITP bug' (intent to package)
    in the Debian BTS (bug tracking system).
    Since we're not aiming to get into the main Debian repository (yet...)
    we can't do much about this one.

    The only choice here is to *override* the warning.

### Overriding Lintian warnings

Sometimes Lintian warnings are false positives, or it is impossible to easily fix them.
Such items may be suppressed to not show up in the usual reports.
For example, I won't be filing an ITP bug for pyfa into Debian
so the `new-package-should-close-itp-bug` warning has to be suppressed.

...types of overrides

Obviously, you shouldn't use this feature lightly and it's better to fix the warnings
instead of just telling Lintian to shut up.
Each suppression should be documented with a good enough reason.

In our case, here's what goes into `debian/pyfa.lintian-overrides`:

```
# This package does not go into Debian archive, and thus does not use BTS.
pyfa: new-package-should-close-itp-bug
```

You can read more about the override format [in the documentation](...).
It's very similar to the output of `lintian` itself.

### Writing manpages

### Updating copyright information

### Tweaking pyfa functionality

#### Disabling unneeded features

#### Playing nicely with updates

### Desktop shortcuts
