# General Linux Guide

To build Cataclysm from source you will need at least a C++ compiler, some basic developer tools, and necessary build dependencies. The exact package names vary greatly from distro to distro, so this part of the guide is intended to give you higher-level understanding of the process.

## Compiler

You have three major choices here: GCC, Clang and MXE.

  * GCC is almost always the default on Linux systems so it's likely you already have it
  * Clang is usually faster than GCC, so it's worth installing if you plan to keep up with the latest experimentals
  * MXE is a cross-compiler, so of any importance only if you plan to compile for Windows on your Linux machine

(Note that your distro may have separate packages e.g. `gcc` only includes the C compiler and for C++ you'll need to install `g++`.)

Cataclysm is targeting C++11 standard and that means you'll need a compiler that supports it. You can easily check if your version of `g++` supports C++11 by running:

    $ g++ --std=c++11
    g++: fatal error: no input files
    compilation terminated.

If you get a line like:

    g++: error: unrecognized command line option ‘--std=c++11’

This means you'll need a newer version of GCC (`g++`).

The general rule is the newer the compiler the better.

## Tools

Most distros seem to package essential build tools as either a single package (Debian and derivatives have `build-essential`) or a package group (Arch has `base-devel`). You should use the above if available. Otherwise you'll at least need `make` and figure out the missing dependencies as you go (if any).

Besides the essentials you will need `git`.

If you plan on keeping up with experimentals you should also install `ccache`, which  will considerably speed-up partial builds.

## Dependencies

There are some general dependencies, optional dependencies and then specific dependencies for either curses or tiles builds. The exact package names again depend on the distro you're using, and whether your distro packages libraries and their development files separately (e.g. Debian and derivatives).

Rough list based on building on Arch:

  * General: `gcc-libs`, `glibc`, `zlib`, `bzip2`
  * Optional: `lua51`, `gettext`
  * Curses: `ncurses`
  * Tiles: `sdl2`, `sdl2_image`, `sdl2_ttf`, `sdl2_mixer`, `freetype2`

E.g. for curses build on Debian and derivatives you'll also need `libncurses5-dev` or `libncursesw5-dev`.

Note on optional dependencies:

  * `gettext` - for localization support; if you plan to only use English you can skip it
  * `lua` - for full-fledged mods; you'll probably prefer to have it

You should be able to figure out what you are missing by reading the compilation errors and/or the output of `ldd` for compiled binaries.

## Make flags

Given you're building from source you have a number of choices to make:

  * `NATIVE=` - you should only care about this if you're cross-compiling
  * `RELEASE=1` - without this you'll get a debug build (see note below)
  * `TILES=1` - with this you'll get the tiles version, without it the curses version
  * `SOUND=1` - if you want sound; this requires `TILES=1`
  * `LOCALIZE=0` - this disables localizations so `gettext` is not needed
  * `LUA=1` - this enables Lua support; needed only for full-fledged mods
  * `CLANG=1` - use Clang instead of GCC
  * `CACHE=1` - use ccache

There is a couple of other possible options - feel free to read the `Makefile`.

If you have a multi-core computer you'd probably want to add `-jX` to the options, where `X` should roughly be twice the number of cores you have available.

Example: `make -j4 CLANG=1 CCACHE=1 NATIVE=linux64 RELEASE=1 TILES=1`

The above will build a tiles release explicitly for 64 bit Linux, using Clang and ccache and 4 parallel processes.

Example: `make -j2 LOCALIZE=0`

The above will build a debug-enabled curses version for the architecture you are using, using GCC and 2 parallel processes.

**Note on debug**:
You should probably always build with `RELEASE=1` unless you experience segfaults and are willing to provide stack traces.

# Debian

Instructions for compiling on a Debian-based system. The package names here are valid for Ubuntu 12.10 and may or may not work on your system.

Building instructions, below, always assume you are running them from the Cataclysm:DDA source directory.

## Linux (native) ncurses builds

Dependencies:

  * ncurses or ncursesw (for multi-byte locales)
  * build essentials

Install:

    sudo apt-get install libncurses5-dev libncursesw5-dev build-essential

### Building

Run:

    make

## Cross-compiling to linux 32-bit from linux 64-bit

Dependencies:

  * 32-bit toolchain
  * 32-bit ncursesw (compatible with both multi-byte and 8-bit locales)

Install:

    sudo apt-get install libc6-dev-i386 lib32stdc++-dev g++-multilib lib32ncursesw5-dev

### Building

Run:

    make NATIVE=linux32

## Cross-compile to Windows from Linux

Dependencies:

  * [mxe](http://mxe.cc)

Install:

    sudo apt-get install autoconf bison flex cmake git automake intltool libtool scons yasm
    mkdir -p ~/src/mxe
    git clone -b stable https://github.com/mxe/mxe.git ~/src/mxe
    cd ~/src/mxe
    make gcc glib

### Building

Run:

    PATH="${PATH}:~/src/mxe/usr/bin"
    make CROSS=i686-pc-mingw32-

## Linux (native) SDL builds

Dependencies:

  * SDL
  * SDL_ttf
  * freetype
  * build essentials

Install:

    sudo apt-get install libsdl1.2-dev libsdl-ttf2.0-dev libfreetype6-dev build-essential

### Building

Run:

    make TILES=1

## Cross-compile to Windows SDL from Linux

Dependencies:

  * [mxe](http://mxe.cc)

Install:

    sudo apt-get install autoconf bison flex cmake git automake intltool libtool scons yasm
    mkdir -p ~/src/mxe
    git clone -b stable https://github.com/mxe/mxe.git ~/src/mxe
    cd ~/src/mxe
    make sdl sdl_ttf

### Building

Run:

    PATH="${PATH}:~/src/mxe/usr/bin"
    make TILES=1 CROSS=i686-pc-mingw32-

# Mac OS X

To build Cataclysm on Mac you'll need [Command Line Tools for Xcode](https://developer.apple.com/downloads/) and the [Homebrew](http://brew.sh) package manager. With Homebrew, you can easily install or build Cataclysm using the Cataclysm forumla on Homebrew Games.

## Simple build using Homebrew

Once you have Homebrew installed, open Terminal and run one of the following commands.

For a curses build:

    brew install homebrew/games/cataclysm

For a tiles build:

    brew install homebrew/games/cataclysm --with-tiles

For an experimental curses build:

    brew install homebrew/games/cataclysm --HEAD

For an experimental tiles build:

    brew install homebrew/games/cataclysm --with-tiles --HEAD

Whichever build you choose, Homebrew will install the appropriate dependencies as needed. The installation will be in `/usr/local/Cellar/cataclysm` with a symlink named `cataclysm` in `/usr/local/bin`.

To launch Cataclysm, just open Terminal and run `cataclysm`.

To update an experimental build, you must uninstall Cataclysm, then reinstall using one of the above commands. If you want to keep your saved games, be sure to backup the folder `/usr/local/Cellar/cataclysm/HEAD/libexec/save` first, then uninstall Cataclysm using the command:

    brew rm cataclysm

## Advanced info for Developers

For most people, the simple Homebrew installation is enough. For developers, here are some more technical details on building Cataclysm on Mac OS X.

### SDL

SDL2, SDL2_image, and SDL2_ttf are needed for the tiles build. Cataclysm can be built using either the SDL framework, or shared libraries built from source.

The SDL framework files can be downloaded here:

* [**SDL2**](http://www.libsdl.org/download-2.0.php)
* [**SDL2_image**](http://www.libsdl.org/projects/SDL_image/)
* [**SDL2_ttf**](http://www.libsdl.org/projects/SDL_ttf/)

Copy `SDL2.framework`, `SDL2_image.framework`, and `SDL2_ttf.framework`
to `/Library/Frameworks` or `/Users/name/Library/Frameworks`.

Alternatively, SDL shared libraries can be installed using a package manager:

For Homebrew:

    brew install sdl2 sdl2_image sdl2_ttf

For MacPorts:

    sudo port install libsdl2 libsdl2_image libsdl2_ttf

### ncurses and gettext

ncurses (with wide character support enabled) and gettext are needed if you want to build Cataclysm with localization.

For Homebrew:

    brew tap homebrew/dupes
    brew install gettext ncurses
    brew link --force gettext ncurses

Then, after compiling, be sure to unlink these libraries to prevent conflicts with the OS X shared libraries:

    brew unlink gettext ncurses

For MacPorts:

    sudo port install gettext ncurses
    hash -r

### Compiling

The Cataclysm source is compiled using `make`.

### Make options

* `NATIVE=osx` build for OS X. Required for all Mac builds.
* `OSX_MIN=version` sets `-mmacosx-version-min=` (for OS X > 10.5 set it to 10.6 or higher); omit for 10.5.
* `TILES=1` build the SDL version with graphical tiles (and graphical ASCII); omit to build with `ncurses`.
* `FRAMEWORK=1` (tiles only) link to SDL libraries under the OS X Frameworks folders; omit to use SDL shared libraries from Homebrew or Macports.
* `LOCALIZE=0` disable localization (to get around possible `gettext` errors if it is not setup correctly); omit to use `gettext`.
* `LANGUAGES="<lang_id_1>[lang_id_2][...]"` compile localization files for specified languages. e.g. `LANGUAGES="zh_CN zh_TW"`
* `RELEASE=1` build an optimized release version; omit for debug build.
* `CLANG=1` build with [Clang](http://clang.llvm.org/), the compiler that's included with the latest Command Line Tools for Xcode; omit to build using gcc/g++.
* `MACPORTS=1` build against dependencies installed via Macports, currently only `gettext` and `ncurses`.
* `USE_HOME_DIR=1` places user files (config, saves, graveyard, etc) in the user's home directory. For curses builds, this is `/Users/<user>/.cataclysm-dda`, for SDL builds it is `/Users/<user>/Library/Application Support/Cataclysm`.

In addition to the options above, there is an `app` make target which will package the tiles build into `Cataclysm.app`, a complete tiles build in a Mac application that can run without Terminal.

For more info, see the comments in the `Makefile`.

### Make examples

Build a release SDL version using Clang without gettext:

    make NATIVE=osx OSX_MIN=10.7 RELEASE=1 TILES=1 LOCALIZE=0 CLANG=1

Build a release SDL version using Clang, link to libraries in the OS X Frameworks folders, don't use `gettext`, and package it into `Cataclysm.app`:

    make app NATIVE=osx OSX_MIN=10.7 RELEASE=1 TILES=1 FRAMEWORK=1 LOCALIZE=0 CLANG=1

Build a release curses version with gettext supplied by Macports:

    make NATIVE=osx OSX_MIN=10.7 RELEASE=1 LOCALIZE=1 MACPORTS=1

### Compiling localization files

If you just want to compile localization files for specified languages, you can add `LANGUAGES="<lang_id_1>[lang_id_2][...]"` option to make command:

    make LANGUAGES="zh_CN zh_TW"

You can get the language ID from the filenames of `*.po` in `lang/po` directory. Setting `LOCALIZE=1` may not tell `make` to compile those localization files for you.

### Running

For curses builds:

    ./cataclysm

For SDL:

    ./cataclysm-tiles

For `app` builds, launch Cataclysm.app from Finder.

## Troubleshooting

### ISSUE: crash on startup due to libint.8.dylib aborting

If you're compiling on Mountain Lion or above, it won't be possible to run successfully on older OS X versions due to libint.8 / pthreads version issue.

From https://wiki.gnome.org/GTK+/OSX/Building:

> "There's another issue with building on Lion or Mountain Lion using either "native" or the 10.7 SDK: Apple has updated the pthreads implementation to provide recursive locking. This would be good except that Gettext's libintl uses this and if the pthreads implementation doesn't provide it it fabricates its own. Since the Lion pthreads does provide it, libintl links the provided function and then crashes when you try to run it against an older version of the library. The simplest solution is to specify the 10.6 SDK when building on Lion, but that won't work on Mountain Lion, which doesn't include it. See below for how to install and use XCode 3 on Lion and later for building applications compatible with earlier versions of OSX."

Workaround: install XCode 3 like that article describes, or disable localization support in Cataclysm so gettext/libint are not dependencies. Or else simply don't support OS X versions below 10.7.

### ISSUE: Colors don't show up correctly

Open Terminal's preferences, turn on "Use bright colors for bold text" in "Preferences -> Settings -> Text"


# Windows

## MinGW Guide
To compile under windows MinGW you first need to download mingw. An automated GUI installer assistant called mingw-get-setup.exe will make everything a lot easier. I recommend installing it to `C:\MinGW`
https://sourceforge.net/projects/mingw/files/latest/download

### MinGW setup
once installed we need to get the right packages. In "Basic Setup", mark `mingw-developer-toolkit`, `mingw32-base` and `mingw32-gcc-g++`

Then install these components using `Installation -> Apply Changes`.

#### Localization
If we want to compile with localization, we will need gettext and libintl. In "All Packages -> MinGW -> MinGW Autotools" ensure that `mingw32-gettext` and `mingw32-libintl` are installed.

### Required Tiles(SDL) Libraries
If we want to compile with Tiles (SDL) we have to download a few libraries.
* `SDL2` http://www.libsdl.org/download-2.0.php chose `SDL2-devel-2.0.X-mingw.tar.gz`.
* `SDL_ttf` https://www.libsdl.org/projects/SDL_ttf/ chose `SDL2_ttf-devel-2.0.12-mingw.tar.gz`.
* `SDL_image` https://www.libsdl.org/projects/SDL_image/ chose ` SDL2_image-devel-2.0.0-mingw.tar.gz` 
* `freetype` http://gnuwin32.sourceforge.net/packages/freetype.htm chose `Binaries` and `Developer files`  

#### Bundled Libraries
The following archives were pre-bundled for convienience and reduction of head-aches, simply download and extract directly to the root directory of the CDDA source:
* `64-bit SDL \ Tiles \ Sound \ Lua \ Localization` http://dev.narc.ro/cataclysm/cdda-win64-codeblocks.7z

#### Installing Tiles(SDL) libraries.
For the first 3 (`SDL2`, `SDL_ttf` and `SDL_image`) you want to extract the include and lib folders from the `i686-w64-mingw32` folders into your MinGW installtion folder. (Reccomended `C:\MinGW`). And the `SDL2_image.dll` and `SDL2_ttf.dll` into your cataclysm root folder.

For freetype you want to grab the include and lib folders from the `freetype-2.X.X-X-lib.zip` and move them into your your MinGW installation folder. Then you want to get the freetype6.dll from the `freetype-2.X.X-X-bin.zip` and move it into your cataclysm root folder.

#### ISSUE - "winapifamily.h" no such file or directoyr
There seems to be at the moment of writing that a file in SDL is broken and needs to be replaced. 
https://hg.libsdl.org/SDL/raw-file/e217ed463f25/include/SDL_platform.h 
Replace SDL_platform.h in the MinGW/include/SDL2 folder and it should be fine.

### Makefile changes
This probably not the best way to do it. But it seems that you need to remove a few dependenceis from the makefile or it will not build.
change the line `LDFLAGS += -lfreetype -lpng -lz -ljpeg -lbz2` to `LDFLAGS += -lfreetype`

### Compiling
Navigate to `MinGW\msys\1.0` and run `msys.bat`. This will start a cmd-like shell where the following entries will be made.

Add the MinGW toolchain to your PATH with `export PATH=$PATH:/c/MinGW/bin`. Replace /c/MinGW/ with the directory into which you installed MinGW (/c/ stands for drive C:, so if it's in F:/foo/bar, you'd use /f/foo/bar).

Navigate to the CDDA source code directory.

Compile using `make TILES=1 NATIVE=win32 LOCALIZE=1` and unless there are problems, it should produce a CDDA binary for you.

If you dont want tiles you can change `TILES` to 0.

If you dont want localization you can change `LOCALIZE` to 0.

## Rough guide to building with only MSYS2

This is a tentative step-by-step guide to building your own CDDA with Tiles, Localization and Lua using only MSYS2. You may want to follow it if the MinGW guide above doesn't work for you or you just feel adventurous. Feedback is very much welcome in terms of issues and/or pull-requests.

This guide assumes you're building on a x86_64 build of Windows. If not adjust the invocations appropriately. It has been tested and proven to work on Windows XP, Windows 7 and Windows 10. Your mileage may vary.

#### 1. Go to https://msys2.github.io/ and download appropriate MSYS (top of the page).

#### 2. Install MSYS2 and leave the Run ticker on. You should end up with a MSYS2 terminal.

#### 3. In the open terminal:

```bash
pacman --needed -Sy bash pacman pacman-mirrors msys2-runtime
```

Note: You may close the terminal now and reopen it from the Start menu (MSYS2 Shell, just to be on the safe-er side).

Note: You may need to bash the close button repeatedly. Or use the task manager to kill it.

#### 4. Open an editor that preserves line-endings

Note: Wordpad should do. Or Notepadd++.

#### 5. Open `C:\msys64\etc\pacman.conf` and change:

```bash
# By default, pacman accepts packages signed by keys that its local keyring
# trusts (see pacman-key and its man page), as well as unsigned packages.
#SigLevel = Never
SigLevel    = Required DatabaseOptional
LocalFileSigLevel = Optional
#RemoteFileSigLevel = Required
```

To:

```bash
# By default, pacman accepts packages signed by keys that its local keyring
# trusts (see pacman-key and its man page), as well as unsigned packages.
SigLevel = Never
#SigLevel    = Required DatabaseOptional
LocalFileSigLevel = Optional
#RemoteFileSigLevel = Required
```

(Exchange the # on SigLevel). This disables signature checking as it is currently borked.

#### 6. Save the file

#### 7. Run in MSYS2 terminal:

```bash
pacman -Su
pacman -S mingw-w64-x86_64-gcc
pacman -S mingw-w64-x86_64-SDL2 mingw-w64-x86_64-SDL2_image mingw-w64-x86_64-SDL2_mixer mingw-w64-x86_64-SDL2_ttf
pacman -S mingw-w64-x86_64-pkg-config mingw-w64-x86_64-libwebp
pacman -S git make
```

If you wish to build with Lua also run:

```bash
pacman -S mingw-w64-x86_64-lua
```

#### 8. Close MSYS2 terminal and open MinGW-w64 Win64 Shell from Start menu and run:

Note: This will download whole CDDA repository. If you're just testing you should probably add `--depth=1`.

```bash
git clone https://github.com/CleverRaven/Cataclysm-DDA.git
cd Cataclysm-DDA
```

#### 9. Open `Makefile` (it's located at `C:\msys64\home\<Your_Login>\Cataclysm-DDA\Makefile`) in an editor that worked before and change:

```Makefile
   ifeq ($(NATIVE), osx)
     CXXFLAGS += -O3
   else
     CXXFLAGS += -Os
     LDFLAGS += -s
   endif
```

To:

```Makefile
   ifeq ($(NATIVE), osx)
     CXXFLAGS += -O3
   else
     #CXXFLAGS += -Os
     LDFLAGS += -s
   endif
```

(Comment out `CXXFLAGS += -Os`). Optimizations break `gcc 4.9.2` you get with MSYS2.

Also change:

```Makefile
   ifeq ($(TARGETSYSTEM),WINDOWS)
     ifndef DYNAMIC_LINKING
       # These differ depending on what SDL2 is configured to use.
       LDFLAGS += -lfreetype -lpng -lz -ljpeg -lbz2
     else
```

To:

```Makefile
   ifeq ($(TARGETSYSTEM),WINDOWS)
     ifndef DYNAMIC_LINKING
       # These differ depending on what SDL2 is configured to use.
       LDFLAGS += -lfreetype -lpng -lz -ltiff -lbz2 -lharfbuzz -lglib-2.0 -llzma -lws2_32 -lintl -liconv -lwebp -ljpeg -luuid
     else
```

(Add `-lharfbuzz -lglib-2.0 -llzma -lws2_32 -lintl -liconv -lwebp -ljpeg -luuid`). You'll need these libs for it to link.

#### 10. Compile your CDDA by running:

```bash
make RELEASE=1 TILES=1 LOCALIZE=0 NATIVE=win64
```

Note: Add `-jX` where X should be the number of threads/cores your processor has (for speeding the build up).

For:
- Lua:
    You'd need to first run:
    
    ```bash
    cd src/lua && lua generate_bindings.lua && cd ../../
    ```

    Then add `LUA=1` to make invocation
- Localization: Use `LOCALIZE=1`

That's it. You should get a `cataclysm-tiles.exe` binary in the same folder you've found the `Makefile` in.

# BSDs

There are reports of CDDA building fine on recent OpenBSD and FreeBSD machines (with appropriately recent compilers), and there is some work being done on making the `Makefile` "just work", however we're far from that and BSDs support is mostly based on user contributions. Your mileage may vary. So far essentially all testing has been on amd64, but there is no (known) reason that other architectures shouldn't work, in principle.

### Building on FreeBSD/amd64 10.1 with the system compiler

FreeBSD uses clang as the default compiler as of 10.0, and combines it with libc++ to provide C++11 support out of the box. You will however need gmake (examples for binary packages):

`pkg install gmake`

Tiles builds will also require SDL2:

`pkg install sdl2 sdl2_image sdl2_mixer sdl2_ttf`

Then you should be able to build with something like this (you can of course set CXXFLAGS and LDFLAGS in your .profile or something):

```
export CXXFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib"
gmake # ncurses builds
gmake TILES=1 # tiles builds
```

The author has not tested tiles builds, as the build VM lacks X; they do at least compile/link successfully.

### Building ncurses version on FreeBSD/amd64 9.3 with GCC 4.8.4 from ports

For ncurses build add to `Makefile`, before `VERSION`:

```Makefile
OTHERS += -D_GLIBCXX_USE_C99
CXX = g++48
CXXFLAGS += -I/usr/local/lib/gcc48/include
LDFLAGS += -rpath=/usr/local/lib/gcc48
```
Note: or you can `setenv` the above (merging `OTHERS` into `CXXFLAGS`), but you knew that.

And then build with `gmake LOCALIZE=0 RELEASE=1`.

### Building on OpenBSD/amd64 5.7 with GCC 4.9.2 from ports/packages

First, install g++ and gmake from packages (g++ 4.8 or 4.9 should work; 4.9 has been tested):

`pkg_add g++ gmake`

Then you should  be able to build with something like:

`CXX=eg++ gmake`

Only an ncurses build is possible, as SDL2 is currently broken on OpenBSD.

Note that testing effort has been focused on -current, and these instructions applied to it at the time of writing and will probably be maintained for it in the future.

### Building on NetBSD/amd64 7.0RC1 with the system compiler

NetBSD has (or will have) gcc 4.8.4 as of version 7.0, which is new enough to build cataclysm. You will need to install gmake and ncursesw:

`pkgin install gmake ncursesw`

Then you should be able to build with something like this (LDFLAGS for ncurses builds are taken care of by the ncurses configuration script; you can of course set CXXFLAGS/LDFLAGS in your .profile or something):

```
export CXXFLAGS="-I/usr/pkg/include"
gmake # ncurses builds
LDFLAGS="-L/usr/pkg/lib" gmake TILES=1 # tiles builds
```

SDL builds currently compile, but did not run in my testing - not only do they segfault, but gdb segfaults when reading the debug symbols! Perhaps your mileage will vary.
