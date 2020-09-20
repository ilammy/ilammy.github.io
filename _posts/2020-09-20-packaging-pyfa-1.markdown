---
layout: post
title:  "Packaging Python apps for Debian"
date:   2020-09-20 12:00:00 +0300
tags:   post development debian pyfa
---

Recently I had lost self-control and started thinking about going back to playing EVE again (for the third time?)
After getting all hyped up on various streams and patch notes,
I have realized that my OS habits have changed a bit since the time.
No longer I own any physical machines with Windows (and I don't want to).
Thankfully, EVE has been supporting Linux since like forever via Wine.

After installing the client and checking in on my characters, naturally, I have reached for familiar tools.
EVE Fitting Tool has been already dead when I left.
Pyfa already existed then and was (and is) great.
What surprised me is that Pyfa was not packaged for my beloved Debian/Ubuntu boxes.
While running it from source is pretty much doable
(especially if you're not as clueless as I am and will avoid compiling wxWidgets from source on your first install),
having it all nicely packaged would be much better.

I have some experience in Debian packaging thanks to Chibi Scheme,
so I thought,
"How hard could it be to package some Python software for a change?"

Well, not as hard as it may seem to, but definitely not a 10-minute hack.
As Göran Weinholt aptly put it,

> The "Debian magic" is hours and hours of manual work.

For what it's worth, *never* in my experience I had encountered packaging issues with anything coming from Debian's repos.
Software bugs? Sure thing!
But having `apt` bork my system directories? Never.
Package fails to install? Nope.
Package leaves crap in the the system after uninstallaton? Not eve— well, *rarely*.
Package fails to start after installation? Seriously?

So... Instead of playing the damn game, I'm off wasting my life fiddling with computers again.
I hope I'll lose interest partway and forget about it all eventually, but at least I'll learn something new.

### Disclaimer

I'm not a Python developer or anything.
I know some Python, occasionally use it write scripts,
I maintain a wrapper library, and once in a blue moon debug something,
but that's about it.
I'm totally clueless about best practices, frameworks, eggs, wheels, pips, condas, and all other poetry.
Therefore, I fell entitled to be mad about "compromises" that the community has accepted long ago.

### Running Pyfa

The first step to packaging software is to make sure it even works on your machine.
Pyfa [installation instructions](https://github.com/pyfa-org/Pyfa#installation) for Linux
boil down to "well, if it's not packaged then you'll figure it out somehow".
Sure thing I will, it's desktop Linux after all!

You may be tempted to follow to [the guide for developers](https://github.com/pyfa-org/Pyfa/blob/master/CONTRIBUTING.md)
which is basically

```shell
python3 -m venv containment-zone
. containment-zone/bin/activate
pip3 install -r pyfa/requirements.txt
python3 pyfa/pyfa.py
```

But **don't do it**.
One of the dependencies is wxPython, installing which via `pip` will build the entire wxWidgets from source.
[This is silly](https://wxpython.org/blog/2017-08-17-builds-for-linux-with-pip/index.html), but that's the way it is.
Obviously, a GUI toolkit takes ages to build and you don't need that.

Instead, you should follow the Debian way and install dependencies using the system package manager.
You'll need to do it anyway.

Here are the dependencies for Pyfa 2.28.2:

| `requirements.txt`  | Debian package |
| ------------------- | -------------- |
| wxPython == 4.0.6   | python3-wxgtk4.0 |
| logbook >= 1.0.0    | ⚠ *missing* |
| matplotlib == 3.2.2 | python3-matplotlib |
| python-dateutil     | python3-dateutil |
| requests >= 2.0.0   | python3-requests |
| sqlalchemy >= 1.3.0 | python3-sqlalchemy |
| cryptography >= 2.3 | python3-cryptography |
| markdown2 >= 2.3.5  | python3-markdown2 |
| packaging >= 16.8   | python3-packaging |
| roman >= 2.0.0      | python3-roman |
| beautifulsoup4 >= 4.6.0 | python3-bs4 |
| pyyaml >= 5.1       | python3-yaml |

Well, the good news is that almost everything is already present.
[`python3-logbook` package exists](https://tracker.debian.org/pkg/logbook)
but not in all distribution versions.
In particular, it is missing from the latest Ubuntu 20.04 repositories
since it's not needed by any Ubuntu-packaged software.

You can either temporarily install it from Debian's repositories,
or use `pip3 install --user`, or something.
However, we will need to package it properly for Pyfa to depend on it in Focal Focca.

### Repackaging logbook

Repackaging a Debian package for another distribution is relatively easy.
Most of the hard work has been already done by the original maintainer
and in easy cases all you need to change is basically branding.
This turned out to be the case with `logbook`.

Here is [my repo with repack](https://git.sr.ht/~ilammy/logbook),
and here is [the original salsa repo](https://salsa.debian.org/debian/logbook).
The changes between them are minimal.

#### Updating package version

The `debian/changelog` file needs to be updated to change the version and distribution:

```diff
+logbook (1.5.3-4~ppa1~ubuntu20.04) focal; urgency=medium
+
+  * Repackage for Ubuntu 20.04 Focal Focca.
+
+ -- Alexei Lozovsky <nospam@example.com>  Sat, 19 Sep 2020 18:23:20 +0300
+
 logbook (1.5.3-4) unstable; urgency=medium
```

The package version is changed from the original **1.5.3-4** into **1.5.3-4\~ppa1\~ubuntu20.04**
to keep my repackaging distinct and future-compatible.
[Debian versioning policy](https://www.debian.org/doc/debian-policy/ch-controlfields.html#version)
has a peculiar comparison algorithm which has a number of useful features.

Since I'll be uploading this package to a private archive,
I would like to be able to update it independently of the original one
*and* to let the version from the main archive to override mine, if one should appear there.
The version parts help with that:

  - `1.5.3` is the "upstream version" of the logbook library itself
  - Debian revision is split into several parts:
    - `-4` is the main Debian revision, usually versioning the packaging
    - `~ppa1` is my extension to version the private packaging modifications
    - `~ubuntu20.04` is my extension to version the repacks for Ubuntu distros

The PPA extensions are [conventional in Ubuntu distro](https://help.launchpad.net/Packaging/PPA/Uploading#Using_packages_from_other_distributions)
and allow the main Debian package archive to override the repacks.

The magic here is in the tilde `~` delimiters which order the suffix *after* the main package.
That is, `1.5.3-4` is newer than `1.5.3-4~whatever`,
so when the *real* `1.5.3-4` appears in the main repos it will override my repack.
However, the versions after tilde are compared as usual, so `1.5.3-4~ppa2` is newer than `1.5.3-4~ppa1`.
The distro-specific suffix also uses tilde to allow a generic `1.5.3-4~ppa1`, if it is possible,
to override any distro-specific `1.5.3-4~ppa1~ubuntu20.04`.

#### New package maintainer

The `Maintainer` field should be updated in the `debian/control` file as well:

```diff
 Source: logbook
 Section: python
 Priority: optional
-Maintainer: Agustin Henze <nospam@example.com>
-Uploaders:
- Iñaki Malerba <nospam@example.com>,
+Maintainer: Alexei Lozovsky <nospam@example.com>
+XSBC-Original-Maintainer: Agustin Henze <nospam@example.com>
```

This is mandated by [Debian derivative guidelines](https://wiki.debian.org/Derivatives/Guidelines).
Technically, since I'm repackaging the software, I will be responsible for any packaging issues.
That is, I am the maintainer here and `reportbug` should address issues to me.

### On naming branches

Note that branch names in packaging repositories follow a particular pattern.
Only recently I had learned that there is a policy for that too: [DEP-14](https://dep-team.pages.debian.net/deps/dep14/).
According to DEP-14, the master packaging branch for Ubuntu distro should be named `ubuntu/master`,
and the version tag should be translated into `ubuntu/1.5.3-4_ppa1_ubuntu20.04`.

### Uploading to Launchpad PPA

Now that `logbook` has been adjusted,
we can check it by building the binary package (`-b`),
without signing the changes (`-uc`):

```shell
dpkg-buildpackage -b -uc
```

Then install it:

```shell
sudo dpkg -i ../python3-logbook_1.5.3-4~ppa1~ubuntu20.04_amd64.deb
```

and check that the module can be imported:

```
$ python
Python 3.8.2 (default, Jul 16 2020, 14:00:26)
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import logbook
>>>
```

Now, Launchpad is meant for open-source software so you don't upload binaries.
Instead, it expects the *source package* do be uploaded and binary packages will be built on Launchpad.

First of all, you will the *original tarball* with upstream sources: `logbook_1.5.3.orig.tar.gz`.
You can get it [from Debian](https://packages.debian.org/source/sid/logbook), for example.
Put it next to the package source tree, which is effectively the unpacked tarball with `debian` directory.

Build the source package (`-S`) and *sign* the changes (`-k...`) like this:

```shell
debuild -S -k2820EB472F198B25C27253145F338CB3B7203B42
```

The signing key needs to be the one registered on Launchpad,
[here's mine](https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=index&search=0x2820EB472F198B25C27253145F338CB3B7203B42),
for example.
(I need to specify it explicitly since the email address is different.)

And then the source package can be easily uploaded to the PPA:

```shell
dput ppa:ilammy/pyfa ../logbook_1.5.3-4~ppa1~ubuntu20.04_source.changes
```

After an hour or so binary packages are built and added to the repository.
Check them out like this:

```shell
sudo add-apt-repository ppa:ilammy/pyfa
sudo apt install python3-logbook
```
