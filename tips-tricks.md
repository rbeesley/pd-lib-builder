pd-lib-builder cheatsheet
=========================

# Creating special builds

## Building for non-native platform

Using pd-lib-builder >=0.6.0 we can define variable `PLATFORM` to specify a
target triplet for cross-compilation. Assuming a W32 package for Pd is unzipped
into path `${PDWIN32}`, to build for Windows 32 bit:

    make PLATFORM=i686-w64-mingw32 PDDIR="${PDWIN32}"

#### Older pd-lib-builder versions

Using pd-lib-builder < 0.6.0, in the absence of variable `PLATFORM`, you would
instead override variables `system`, `target.arch`, `CC` and / or `CXX`,
`STRIP`. Example:

    make system=Windows target.arch=i686 CC=i686-w64-mingw32-gcc STRIP=i686-w64-mingw32-strip PDDIR="${PDWIN32}"

#### Toolchains

To build for non-native OS and/or architecture you need a cross toolchain. On
Linux such toolchains are relatively easy to get. For example Debian Buster
amd64 provides them for the following platforms (install g++ with dependencies
for a given platform to get the whole toolchain):

- `arm-linux-gnueabihf`
- `aarch64-linux-gnu`
- `i686-linux-gnu`
- `i686-w64-mingw32` and `x86_64-w64-mingw32` (install `mingw-w64`)

Cross toolchains for OSX/MacOS are not generally distributed. Project
`osxcross` from Thomas Poechtraeger can create them for Linux.

## Universal binaries on macOS

The compiler, by default, builds for the native architecture of the build
machine. To make a "universal" multi-arch build, specify the desired
archtectures on the command line using the "arch" pd-lib-builder Makefile
variable.

For example, to build a "fat" external for both 64-bit Intel and Arm (Apple
Silicon):

    make arch="x86_64 arm64"

If the build is successful, the compiled architectures in the built external can
be confirmed via the `file` command:

~~~sh
% file vbap.pd_darwin
vbap.pd_darwin: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit bundle x86_64] [arm64:Mach-O 64-bit bundle arm64]
vbap.pd_darwin (for architecture x86_64):   Mach-O 64-bit bundle x86_64
vbap.pd_darwin (for architecture arm64):    Mach-O 64-bit bundle arm64
~~~

Note: The available architectures depend on which macOS version & command line
tools/Xcode combination the build system has. For example, any newer macOS
10.15+ will support both x86_64 (Intel 64-bit) and arm64 (Apple Silicon) while
OSX 10.6 - macOS 10.14 can build for x86_64 and i386 (Intel 32-bit).

## Building double-precision externals

At the time of writing (2023-07-06) there is no official Pd that supports
double-precision numbers yet.
However, if you do get hold of an experimental double-precision Pd, you can
easily build your externals for 64-bit numbers, by passing `floatsize=64`
as an argument to `make`.
Starting with Pd>=0.54, double precision externals use different extensions
from traditional (single-precision) externals.
The extension consists of the OS ("linux", "darwin", "windows"), the CPU
architecture ("amd64" (x86_64), "i386" (x86), "arm64",...) and the floatsize
in bits ("64" for double-precision), followed by the system's native extension
for dynamic libraries (".dll" on Windows, ".so" on macOS/Linux/un*xes).
As of pd-lib-builder==0.7.0, you have to manually pass this extension:

    make floatsize=64 extension=windows-amd64-64.dll
    make floatsize=64 extension=linux-arm64-64.so
    make floatsize=64 extension=darwin-fat-64.so arch="x86_64 arm64"


# Project management

In general it is advised to put the `Makefile.pdlibbuilder` into a separate
subdirectory (e.g. `pd-lib-builder/`).
This makes it much easier to update the `Makefile.pdlibbuilder` later

You *should* also use a variable to the actual path of the Makefile.pdlibbuilder
(even if you keep it in the root-directory), as this allows easy experimenting
with newer (or older) (or site-specific) versions of the pd-lib-builder
Makefile.

~~~make
PDLIBBUILDER_DIR=pd-lib-builder/
include $(PDLIBBUILDER_DIR)/Makefile.pdlibbuilder
~~~

## Keeping pd-lib-builder up-to-date

### `git subtree`

With git-subtrees, you make the pd-lib-builder repository (or any other
repository for that matter) part of your own repository - with full history and
everything - put nicely into a distinct subdirectory.

Support for *manipulating* subtrees has been added with Git-v1.7.11 (May 2012).
The nice thing however is, that from "outside" the subtree is part of your
repository like any other directory. E.g. older versions of Git can clone your
repository with the full subtree (and all it's history) just fine.
You can also use git-archive to make a complete snapshot of your repository
(including the subtree) - nice, if you e.g. want self-contained downloads of
your project from git hosting platforms (like Github, Gitlab, Bitbucket,...)

In short, `git subtree` is the better `git submodule`.

So here's how to do it:

#### Initial setup/check-out

This will create a `pd-lib-builder/` directory containing the full history of
the pd-lib-builder repository up to its release `v0.5.0`

~~~sh
git subtree add --prefix=pd-lib-builder/ https://github.com/pure-data/pd-lib-builder v0.5.0
~~~

This will automatically merge the `pd-lib-builder/` history into your current
branch, so everything is ready to go.

#### Cloning your repository with the subtree

Nothing special, really.
Just clone your repository as always:

~~~sh
git clone https://git.example.org/pd/superbonk~.git
~~~

#### Updating the subtree

Time passes and sooner or later you will find, that there is a shiny new
pd-lib-builder with plenty of bugfixes and new features.
To update your local copy to pd-lib-builder's current `master`, simply run:

~~~sh
git subtree pull --prefix pd-lib-builder/ https://github.com/pure-data/pd-lib-builder master
~~~

#### Pulling the updated subtree into existing clones

Again, nothing special.
Just pull as always:

~~~sh
git pull
~~~

#### Further reading

More on the power of `git subtree` can be found online
- https://medium.com/@v/git-subtrees-a-tutorial-6ff568381844
- https://www.atlassian.com/blog/git/alternatives-to-git-submodule-git-subtree
- ...

### ~~`git submodule`~~ [DISCOURAGED]

#### Initial setup/check-out

To add a new submodule to your repository, just run `git submodule add` and
commit the changes:

~~~sh
git submodule add https://github.com/pure-data/pd-lib-builder
git commit .gitmodules pd-lib-builder/ -m "Added pd-lib-builder as git-submodule"
~~~

#### Cloning your repository with the submodule

When doing a fresh clone of your repository, pass the `--recursive` option to
automatically fetch all submodules:

~~~sh
git clone --recursive https://git.example.org/pd/superbonk~.git
~~~

If you've cloned non-recursively, you can initialize and update the submodules
manually:

~~~sh
git submodule init
git submodule update
~~~

#### Updating the submodule

Submodules are usually fixed to a given commit in their repository.
To update the `pd-lib-builder` submodule to the current `master` do something
like:

~~~sh
cd pd-lib-builder
git checkout master
git pull
cd ..
git status pd-lib-builder
git commit pd-lib-builder -m "Updated pd-lib-builder to current master"
~~~

#### Pulling the updated submodule into existing clones

After you have pushed the submodule updates in your repository, other clones of
the repository can be updated as follows:

~~~sh
git pull
~~~

The above will make your repository aware, that the submodule is out-of-sync.

~~~sh
$ LANG=C git status pd-lib-builder
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   pd-lib-builder (new commits)
$
~~~

In order to sync the submodule to the correct commit, run the following:

~~~sh
git submodule update
~~~

#### Drawbacks

`git submodule` has a number of drawbacks:
- it requires special commands to synchronize the submodules, in addition to
  synching your repository.
- you must make sure to use an URL for the submodule that is accessible to your
  potential users. e.g. using `git@github.com:pure-data/pd-lib-builder` is bad,
  because it requires everybody who wants to checkout your sources to have a
  github-account - even if they could checkout *your* repository anonymously.
- submodules will be excluded from `git archive`. This means, that if you use a
  mainstream git provider (like Github, GitLab, Bitbucket,...) and make releases
  by creating a `git tag`, the automatically generated zipfiles with the sources
  will lack the submodule - and your users will not be able to compile your
  source code.

In general, I would suggest to **avoid** `git submodule`, and instead use the
better `git subtree` (above).
