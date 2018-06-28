## How to build OpenJDK on Windows

This is a short description of how I build OpenJDK on Windows, as of today (2018-06-28).

_Big Disclaimer: these are the variants I personally have worked with and know will work. Many more variants - different toolchains, windows versions, Visual Studio versions - may work too but I do not know for sure nor do I care._

### Prerequisites

- Windows 7 x64 or Windows 10 x64
- Visual Studio 2013 Professional or Visual Studio 2017 Community Edition

### Install Cygwin
Install 64bit version of Cygwin. 

The following tools are needed (beyond the base tools Cygwin installs automatically):
- mercurial
- diffutils
- binutils
- make (The GNU version)
- m4 (Interpreters)
- cpio 
- gawk
- file ("file: determine file types")
- zip, unzip
- procps-ng
- autoconf, automake

The following tools may not be strictly needed but are useful for contributors:
- ksh (for webrev)
- patch, diff
- ssh
- wget

Note: Since updating cygwin is cumbersome, now would be a good time to add any more tools - vim, rsync, emacs, ... - you may need.

### Directory structure

Completely arbitrary, but this is my folder setup and the rest of the document will assume it is:

```
openjdk
  |--jdks
  |    |--<buildjdk, e.g. openjdk10>
  |
  |--<repository, e.g. jdk-jdk>
               |--source
               |--output
```

### Get build jdk

To build the jdk, we need a jdk - and a recent one at that. Download a build jdk from somewhere. For openjdk 11, one needs a jdk10 to build. 

At the time of this writing, the only way to get a jdk 10 as build JDK on Windows was to download the official oracle JDK:
 - http://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html

_(Note: I prefer to download from https://adoptopenjdk.net/ or from https://sap.github.io/, but no Windows JDK 10 variants available there yet.)_

Put the downloaded and extracted JDK into openjdk/jdks

### Get sources

#### The official slow way: hg clone

The standard way to get the sources is to clone the corresponding mercurial repository from hg.openjdk.java.net.

Open a cygwin shell, and in the parent directory of the future source folder, call:

````
hg clone http://hg.openjdk.java.net/jdk/jdk/
````

Unfortunately this may lag since download speed from mercurial servers at openjdk.java.net is limited. In Europe, cloning a full repository takes ~40 minutes.

#### The faster alternative: copy sources and update

A faster alternative to cloning is to copy the sources from somewhere else and just update the repository.

One way to get fresh source tarballs is to download them from: https://builds.shipilev.net/workspaces . These tarballs are maintained by _Aleksey Shipilev from Red Hat_, thanks to him for this time saver.

```
mkdir /cygdrive/c/openjdk/jdk-jdk
cd /cygdrive/c/openjdk/jdk-jdk
wget https://builds.shipilev.net/workspaces/jdk-jdk.tar.xz
tar -xf jdk-jdk.tar.xz
mv jdk source
hg pull
hg update
```

(Note: I usually rename the expanded source tree root to "source").

### Build

Open the cygwin shell. Create an output directory and start configure.

```
mkdir /cygdrive/c/openjdk/jdk-jdk/output
cd /cygdrive/c/openjdk/jdk-jdk/output
```
Now start the build:

Debug:
```
bash ../source/configure --with-boot-jdk=/cygdrive/c/openjdk/jdks/openjdk10 --with-debug-level=fastdebug --with-native-debug-symbols=external
make images
```
Release:
```
bash ../source/configure --with-boot-jdk=/cygdrive/c/openjdk/jdks/openjdk10 --with-debug-level=release
make images
```

### Notes

Scratch build into an existing previous build without re-issuing the configure command:

```
make reconfigure clean images
```

Build 32bit:
```
--with-target-bits=32
```

Non-PCH:
```
--disable-precompiled-headers
```

Choose which Visual Studio installation to use:
```
--with-toolchain-version={2010|2013|2017}
```

