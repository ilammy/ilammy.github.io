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

`dh_lintian` is responsible for installing override files.
It looks at the following paths:

  - `debian/source/lintian-overrides` — for the whole source package
  - `debian/${package}.lintian-overrides` — for individual binary packages

Overrides for binary packages are a part of the package
and get installed into `/usr/share/lintian/overrides`.
Source package overrides are not installed, they are only used at build time.

Obviously, you shouldn't use this feature lightly and it's better to fix the warnings
instead of just telling Lintian to shut up.
Each suppression should be documented with a good enough reason.

In our case, here's what goes into `debian/pyfa.lintian-overrides`:

```
# This package does not go into Debian archive, and thus does not use BTS.
pyfa: new-package-should-close-itp-bug
```

### Writing manpages

Every top-level program in Debian should have a manpage.
Manual pages are normally written in **nroff** format.
This is a... *venerable* format which is described in
[`man(7)`](https://manpages.debian.org/man/7/man).
You may like it or not but that's *the* format used by `man`.

Manpages for programs usually go into `debian/${name}.1` files.
(The `1` is the number of manual section for programs.)
Then `debian/${package}.manpages` file enumerates which manpages to install for which package
(`dh_installman` takes care of it).

```
debian/pyfa.1
```

Since pyfa is primarily GUI application, there aren't many things to write in the manpage.
The most important parts are in the header.
Here's a excerpt:

```
.TH PYFA 1 "2020-09-27" "2.28.2" "General Commands Manual"
.SH NAME
pyfa \- EVE Online fitting assistant

.SH SYNOPSIS
.SY pyfa
.RB [ --root ]
.YS

.SH DESCRIPTION

.BR Pyfa ,
short for Python fitting assistant,
allows you to create, experiment with, and save ship fittings without being in game.
```

I won't go into roff syntax here, just read `man(7)`.

### Updating copyright information

Debian treats licensing and copyright as a serious business,
making sure that core repositories contain only free software.
This translates into packaging process too.

The `debian/copyright` file should describe the copyright information:
who owns the copyright over which files and what is the license
under which Debian is allowed to use this content.
Sometimes this is easy to do,
such as with applications written by one person,
or with organizations demanding copyright assignment to them.
However, with modern open-source development practices it's often a legal nightmare.
*Technically*, the source code is the combined work of the authors
(which pyfa has 83 as of time of writing this),
and you should accurately track who edited which files and when.

I don't really have time for this
but I did some basic authorship analysis to single out core contributors.
The result looks like this:

```
Format: https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/
Upstream-Name: pyfa
Upstream-Contact: DarkPhoenix <xxxxxx@example.com>
Source: https://github.com/pyfa-org/pyfa
License: GPL-3.0+
Comment: accurate authorship information is available in git history

Files: config.py
       pyfa.py
Copyright: 2010 DarkPhoenix
           2010 Diego Duclos
           2010 HomeWorld
           2014 Ryan Holmes
           2016 Ebag333
           other minor contributors, see git history
License: GPL-3.0+

License: GPL-3.0+
 pyfa is free software; you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published
 by the Free Software Foundation, either version 3 of the License,
 or (at your option) any later version.
 .
 pyfa is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.
 .
 You should have received a copy of the GNU General Public License
 along with pyfa.  If not, see <http://www.gnu.org/licenses/>.
 .
 On Debian systems, the full text of the GNU General Public License
 Version 3 can be found in `/usr/share/common-licenses/GPL-3'.
```

The `debian/copyright` file is expected to be readable by machines and humans alike.
Its format is documented [in the policy](https://www.debian.org/doc/packaging-manuals/copyright-format/1.0/).

pyfa is primarily licensed under GPL 3, with some parts (eos) released under LGPL.
There are also some original game images used with CCP's permission.

### Tweaking pyfa functionality

#### Disabling unneeded features

#### Playing nicely with updates

### Desktop shortcuts
