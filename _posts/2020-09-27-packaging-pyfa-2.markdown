---
layout: post
title:  "Packaging Python apps for Debian (part 2)"
date:   2020-09-27 12:00:00 +0300
tags:   post development debian pyfa
---

[In the previous part](https://blog.ilammy.net/2020/09/20/packaging-pyfa-1.html)
we have managed to get **pyfa** working.
Now we'll get to actual packaging.

And yes, if you're interested, I have *not* played the game since :)

### Breaking in

To get things started, let's get a basic package layout with `debmake`.
That way I won't forget about some important file that needs to be under the `debian` directory.
Here's what a default looks like:

```
$ tree debian
debian
├── changelog
├── compat
├── control
├── copyright
├── patches
│   └── series
├── README.Debian
├── rules
├── source
│   ├── format
│   └── local-options
└── watch
```

For the uninitiated, here is what each file does:

| File | Purpose |
| ---- | ------- |
| `debian/changelog`     | Package changelog |
| `debian/compat`        | Debhelper version |
| `debian/control`       | Package metadata is stored here |
| `debian/copyright`     | Per-file copyright and licensing information |
| `debian/patches/...`   | Maintainer patches to apply to the original source |
| `debian/rules`         | Master Makefile, entry point for packaging system |
| `debian/README.Debian` | Information to package maintainers, if necessary |
| `debian/source/format` | Source package format version |
| `debian/watch`         | Automatching watching for upstream updates |

Read [Debian Policy](https://www.debian.org/doc/debian-policy/ch-source.html) to learn more.

Most of the default boilerplate is not suitable for our package,
but that's a good start nevertheless.

### Using pybuild

Debian packaging support for included an amazing **debhelper** tool
which really helps with typical packaging and does most of the work.
Python software can be easily packaged with **pybuild** as a 'build system'.

(Note that the accent here is the word *typical*.
If the software is not typical – for Debian – then you might need to do way more work.)

It's really easy to start using pybuild,
just write the following in `debian/rules`:

```makefile
export PYBUILD_NAME = pyfa

%:
	dh $@ --with python3 --buildsystem=pybuild
```

and add in some build dependencies to `debian/contol`:

```diff
-Build-Depends: debhelper (>= 11~)
+Build-Depends: debhelper (>= 11~), dh-python, python3, python3-setuptools
```

### Configuring pybuild

pybuild supports setuptools/distutils projects out of the box.
Unfortunately, pyfa is not setuptools-based project.

While pyfa is easy to install manually (just copy some Python files),
pybuild is not only about installation.
There are other tasks which are better left to Debian automation:

  - Python bytecode compilation
  - Python version selection
  - autofilling dependency information

So I'd really like to piggyback on pybuild.
For that, we'd need to cheat a little and make it *look* like pyfa uses setuptools.

### setup.py content

Here's what we need for the build:

```python
from setuptools import setup, find_packages

def read_version_yml():
    with open('version.yml') as f:
        return f.read().split(':')[1].strip().strip('v')

def read_requirements_txt():
    with open('requirements.txt') as f:
        return [line.rstrip('\n') for line in f]

setup(
    name='pyfa',
    version=read_version_yml(),
    description='EVE Online fitting assistant',
    long_description='Pyfa, short for python fitting assistant, allows you to create, experiment with, and save ship fittings without being in game.',
    url='https://github.com/pyfa-org/Pyfa',
    license='GPL-3.0-only',
    author='DarkPhoenix',
    author_email='no-spam@example.com',
    platforms='any',
    install_requires=read_requirements_txt(),
    packages=find_packages(exclude=['_development', 'tests']),
    scripts=['pyfa.py'],
)
```

Nothing special, just some minimum information about the project:
some static information, with version and dependencies from the source code.
Several directories are also excluded from the build as they don't have to be packaged.
You can read more about setup.py files in
[setuptools docs](https://setuptools.readthedocs.io/en/latest/userguide/index.html).

### Using setup.py

But where do we put the `setup.py` file?
You can't just drop it into the root directory – where setuptools expect it.
That will make the pristine source code in the repo to be different from the pristine source taball.
Debian package building system detects this error.

The solution is to modify the source tree *after* the build has started.
Normally, you use `debian/patches` for that,
but since we're not stricly patching existing source code
it's more convenient to store `setup.py` as is, not as a patch.

So we keep it under `debian/setup.py` and just hack our way out of this situation
by telling debhelper to copy the file before doing the build:

```makefile
# Use our custom "setup.py" to fake a distutils build system
execute_after_dh_testdir:
	cp debian/setup.py setup.py
```

This target will be executed before the `dh_testdir` step –
the very first step debhelper does in a build directory.

### Installing into the right place

By default pybuild will install Python code as a *library*.
Python modules go under `/usr/lib/pythonX.Y/dist-packages` on Debian.
However, pyfa is not a library and its modules should not be accessible to everyone in the system.
This can be tweaked with a couple of flags passed to pybuild:

```makefile
# Install private packages into a private directory
export PYBUILD_INSTALL_ARGS += --install-lib=/usr/share/$(PYBUILD_NAME)
export PYBUILD_INSTALL_ARGS += --install-scripts=/usr/share/$(PYBUILD_NAME)
```

Now everything will go under `/usr/share/pyfa`, where it should be according to the policy.

### Installing static files

In addition to the files installed by `setup.py`,
pyfa needs to have item icons and other static data to be available.
These can be installed with an `*.install*` and `*.links` files.

The `pyfa.install` file lists files which need to be installed
in addition to what is installed at the project install step (controlled by `setup.py`):

```
config.py    usr/share/pyfa/
db_update.py usr/share/pyfa/
imgs/        usr/share/pyfa/
service/     usr/share/pyfa/
version.yml  usr/share/pyfa/
```

The `pyfa.links` file manages creation of symbolic links.
We use it to create a 'top-level executable' `pyfa`
so that the users don't have to `/usr/share/pyfa/pyfa.py` all the time.

```
usr/share/pyfa/pyfa.py usr/bin/pyfa
```

You can learn more about both special files
in [`dh_install(1)`](https://manpages.debian.org/testing/debhelper/dh_install.1.en.html)
and [`dh_link(1)`](https://manpages.debian.org/testing/debhelper/dh_link.1.en.html)
as both features are provided by debhelper.

### Prebuilding EVE item database

On the first start pyfa checks for the item database file `eve.db`
and builds it from the static data if it's not available.
However,
1) this takes about 20 seconds,
2) we don't install the static data,
3) pyfa doesn't have write access to `/usr/share/pyfa` when running as a regular user.

Instead, let's prebuild the database to make startup faster.
Add a step after the build in `debian/rules`:

```makefile
# Prebuild EVE item database to be included into the package
execute_after_dh_auto_build:
	python3 -B db_update.py
```

and install the file in `pyfa.install`:

```diff
 db_update.py usr/share/pyfa/
+eve.db       usr/share/pyfa/
 imgs/        usr/share/pyfa/
```

Now, the tricky part here is that now we need to be able to *run* pyfa on the build machine.
Of course, the maintainer will probably have all the dependencies already installed,
but the build environment in Debian is required to be clean.
We need to all the necessary packages to the build dependencies explicitly:

```diff
-Build-Depends: debhelper (>= 11~), dh-python, python3, python3-setuptools
+Build-Depends: debhelper (>= 11~), dh-python, python3, python3-logbook, python3-setuptools, python3-sqlalchemy, python3-wxgtk4.0
```

Thankfully, there are not many of them and we don't have to replicate the `requirements.txt`.

### Tweaking the control file

Speaking of the dependencies, let's tweak the `debian/control` further
for the dependencies to be accurate.

pyfa requires at least Python 3.6 so let's tell pybuild about that:

```diff
 Build-Depends: debhelper (>= 11~), dh-python, python3, python3-logbook, python3-setuptools, python3-sqlalchemy, python3-wxgtk4.0
+X-Python3-Version: >= 3.6
 Standards-Version: 4.1.4
```

Also, tell it to add all the Python dependencies it figures out:

```diff
 Package: pyfa
-Depends: ${misc:Depends}, ${shlibs:Depends}
+Depends: ${misc:Depends}, ${shlibs:Depends}, ${python3:Depends}
```

### Building the package

Finally we have reached the step when the package can be built and it should run.
Build a binary package (`-b`) without signatures (`-uc`):

```shell
dpkg-buildpackage -b -uc
```

Yay, it seems to work!

```
$ dpkg-deb --info ../pyfa_2.28.2-1_amd64.deb
 new Debian package, version 2.0.
 size 10611268 bytes: control archive=80868 bytes.
     619 bytes,    13 lines      control
  302513 bytes,  4061 lines      md5sums
     280 bytes,    12 lines   *  postinst             #!/bin/sh
     383 bytes,    12 lines   *  prerm                #!/bin/sh
 Package: pyfa
 Version: 2.28.2-1
 Architecture: amd64
 Maintainer: ilammy <>
 Installed-Size: 32827
 Depends: python3-bs4, python3-cryptography (>= 2.3), python3-dateutil, python3-logbook, python3-markdown2, python3-matplotlib, python3-packaging, python3-requests, python3-roman, python3-sqlalchemy (>= 1.3.0), python3-wxgtk4.0, python3-yaml, python3:any (>= 3.6~)
 Section: unknown
 Priority: optional
 Multi-Arch: foreign
 Homepage: <insert the upstream URL, if relevant>
 Description: auto-generated package by debmake
  This Debian binary package was auto-generated by the
  debmake(1) command provided by the debmake package.
```

I can even install it and run it!

```
$ sudo dpkg -i ../pyfa_2.28.2-1_amd64.deb
$ pyfa
```
