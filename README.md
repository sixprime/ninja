# Ninja

Ninja is a small build system with a focus on speed.
https://ninja-build.org/

See [the manual](https://ninja-build.org/manual.html) or
`doc/manual.asciidoc` included in the distribution for background
and more details.

Binaries for Linux, Mac and Windows are available on
  [GitHub](https://github.com/ninja-build/ninja/releases).
Run `./ninja -h` for Ninja help.

Installation is not necessary because the only required file is the
resulting ninja binary. However, to enable features like Bash
completion and Emacs and Vim editing modes, some files in misc/ must be
copied to appropriate locations.

If you're interested in making changes to Ninja, read
[CONTRIBUTING.md](CONTRIBUTING.md) first.

## Building Ninja itself

You can either build Ninja via the custom generator script written in Python or
via CMake. For more details see
[the wiki](https://github.com/ninja-build/ninja/wiki).

### Python

```
./configure.py --bootstrap
```

This will generate the `ninja` binary and a `build.ninja` file you can now use
to build Ninja with itself.

### CMake

```
cmake -Bbuild-cmake
cmake --build build-cmake
```

The `ninja` binary will now be inside the `build-cmake` directory (you can
choose any other name you like).

To run the unit tests:

```
./build-cmake/ninja_test
```

## Usage

### Environment variables

Using environment variables allows for more flexible design of Ninja build files. This fork supports environment variables when starting with `NINJA_`, and look for them only if normal lookup failed.

Usage example :

``` bash
$ tree /F
Folder PATH listing
Volume serial number is D407-7C4F
D:.
│   .gitignore
│   build.clang.ninja
│   build.msvc.ninja
│   build.ninja
│   convoy.manifest
│   LICENSE
│   README.md
│
├───.github
│   └───workflows
│           ci-ninja.yml
│
├───docs
├───external
├───src
│       convoy.cpp
│
└───tests
```

Notice 3 Ninja build files :

- `build.ninja` : main entry point and responsible for listing all C++ files to compile. Also includes the compiler target `msvc` or `clang` Ninja file accordingly, depending on the value of the environment variable `$NINJA_CC`.

``` bash
$ cat build.ninja
ninja_required_version = 1.10

root = .
builddir = build

include build.$NINJA_CC.ninja

build $builddir/convoy.o: cxx $root/src/convoy.cpp

build convoy: link $builddir/convoy.o

default convoy

build all: phony convoy
```

- `build.msvc.ninja` : specific build instructions when choosing the MSVC compiler.

``` bash
$ cat build.msvc.ninja
cxx = cl
cflags = /nologo /showIncludes /Ox /EHsc
ldflags =

rule cxx
  deps = msvc
  command = $cxx $cflags /c $in -Fo$out

rule link
  command = link /OUT:$out.exe $in $ldflags
```

- `build.clang.ninja` : specific build instructions when choosing the Clang compiler.

``` bash
$ cat build.clang.ninja
cxx = clang
cflags = -x c++ -std=c++17 -Wall -Wextra -Wpedantic
ldflags = -L$builddir

rule cxx
  command = $cxx $cflags -c $in -o $out

rule link
  command = $cxx $ldflags $in -o $out
```

To start compilation using Clang (on Windows),

``` bash
$ set NINJA_CC=clang && ninja
[2/2] clang -Lbuild build/convoy.o -o convoy
```

To start compilation using MSVC (on Windows),

``` bash
$ "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat" x86
**********************************************************************
** Visual Studio 2022 Developer Command Prompt v17.3.6
** Copyright (c) 2022 Microsoft Corporation
**********************************************************************
[vcvarsall.bat] Environment initialized for: 'x86'

$ set NINJA_CC=msvc && ninja
[2/2] link /OUT:convoy.exe build/convoy.o
Microsoft (R) Incremental Linker Version 14.33.31630.0
Copyright (C) Microsoft Corporation.  All rights reserved.
```

The default Ninja pattern would force us to explicitly specify the Ninja build file to execute :

``` bash
ninja -f build.clang.ninja
ninja -f build.clang.ninja -t clean
```

Not with this pattern change :

``` bash
ninja
ninja-t clean
```

This is an experimental feature that has pros and cons, but allows for a more elegant usage of Ninja by default and without compromising speed, at least in this use case.
