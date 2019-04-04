---
layout:   post
title:    "Organizing C library code"
date:     2019-04-05 00:45:00 +0300
tags:     post imp c layout headers
comments: false
---

Currently I work on [a library](https://github.com/cossacklabs/themis)
that is mostly written in C.
I feel that writing good C code is somewhat arcane art by now
so I want to document it for posterity,
and for myself to revisit when I doubt greybeards' decisions.
Let's start with directory and file layout.

## Model problem

Just to be specific,
let's say we are writing a library for arbitrary-precision arithmetic:
_ilammy's multi-precision library_ aka IMP.
(And pretend that [GMP](https://gmplib.org/) does not exist.)

## Questionable layouts

### Dumping everything into the root directory

The easiest way is to just dump all files into the root directory:

```
.
├── Makefile
├── number.h
├── number.c
├── allocation.c
└── allocation.h
```

This is easy to do and has all the code readily accessible.
However, there is an issue with this layout:
it's hard to tell which headers files are public interface
and which ones are your implementation details.

### Separate `sources` and `headers`

Another common approach is to put header files and source files
into separate directories:

```
.
├── Makefile
├── headers
│   ├── number.h
│   └── allocation.h
└── sources
    ├── number.c
    └── allocation.c
```

It's a bit better because now
'the interface' is clearly separated in the `headers` directory.
However, the header files are still a mess:
we should not really export `allocation.h` file to the users.

## Preferred code layout

It's better to have the code laid out as follows:

```
.
├── Makefile
├── build
├── include
│   └── imp
│       └── number.h
├── src
│   └── imp
│       ├── number.c
│       ├── allocation.c
│       └── allocation.h
└── test
    └── imp
        ├── number.c
        └── allocation.c
```

Note the crucial points:

- **Separate public headers.**
  These go into `include` directory (a customary name).
  Now it's obvious which headers are to be exported to the user,
  and which ones are for internal consumption only.

- **Namespacing.**
  Using library name as a namespace makes life a bit easier
  as now you import the files as
  ```c
  #include <imp/number.h>
  ```
  and can name them however you want.
  It also makes it easy to split the library in two if necessary.
  Note that the source code is namespaced as well.

- **Default `build` directory.**
  I personally prefer in-source builds for regular development
  because it limits 'the project' to its working directory.
  It's a good idea to keep all generated files in a single place:
  then you can `rm -r` it and you're done.
  However, that should be only _a default_:
  sometimes out-of-source builds are necessary,
  and they are not that hard to do with Make.

## On header files

### Include guards

It's totally okay to use

```c
#pragma once
```

Any reasonable compiler supports it by now.

However, I find the old school include guards somewhat appealing.
I'm not always using include guards, but when I do, I write them like this:

```c
#ifndef IMP_NUMBER_H
#define IMP_NUMBER_H

// your declarations go here

#endif /* IMP_NUMBER_H */
```

Note that the macro name is _namespaced_ to prevent accidental shadowing.
The most common culprit is the ubiquitions `UTILS_H`
included from two totally unrelated libraries.

### Umbrella header file

For libraries, it makes very much sense to provide an _umbrella_ header:

```
.
└── include
    └── imp
        └── imp.h
```

which simply includes all your public headers:

```c
#ifndef IMP_IMP_H
#define IMP_IMP_H

#include <imp/number.h>

#endif /* IMP_IMP_H */
```

Umbrella headers make it easy for the users to import your library:

```c
#include <imp/imp.h>
```

Yes, this may bring in some symbols that are not really needed,
but with this approach users don't have to track new module headers.
And there's always an option to include only what you need.

### C++ compatibility

Take care to add `extern "C"` to your headers
so that they can be used with C++ programs.
This makes sure that function names are not _mangled_.

```c
#ifndef IMP_ADDITION_H
#define IMP_ADDITION_H

#include <imp/number.h>

#ifdef __cplusplus
extern "C" {
#endif

// your declarations go here

#ifdef __cplusplus
}
#endif

#endif /* IMP_ADDITION_H */
```

Note that includes are not wrapped into `extern "C"`
while the declarations are:
`extern "C"` don't nest well.
