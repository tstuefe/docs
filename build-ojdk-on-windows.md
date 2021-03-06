## How to build OpenJDK on Windows

This is a short description of how I build OpenJDK on Windows, as of today (2018-06-28).

_Disclaimer: This setup I have worked with and know it will work. Other variants - different toolchains, Windows versions, Visual Studio versions - may and probably will work too but are outside the scope of this document._

### Prerequisites

- Windows 7 x64 or Windows 10 x64
- Visual Studio 2013 Professional or Visual Studio 2017 Community Edition (_see Disclaimer above_)

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

Note: Since updating Cygwin is cumbersome, now would be a good time to add any more tools - vim, rsync, emacs, git, ... - you'd want in your setup.

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
               | (any number of build outputs:)
               |--output-fastdebug
               |--output-slowdebug
               |--output-fastdebug-nonpch
               |--output-fastdebug-zero
               |--output-fastdebug-32
               .....
```
It is of course fine to use another layout - just make the mental adjustments while reading this document.

### Get build jdk

To build the jdk, we need a jdk. To build OpenJDK 11, we need JDK 10. 

At the time of this writing, the only way to get a JDK 10 for Windows is to download the official oracle JDK:
 - http://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html

(_For Linux, one has a number of options beside the official Oracle JDK binaries, among others https://sap.github.io/ or https://adoptopenjdk.net/. But they are not there yet for Windows._)

Put the downloaded and extracted JDK into openjdk/jdks.

### Get sources

#### The official _slow_ way: hg clone

The standard way to get the sources is to clone the corresponding mercurial repository from hg.openjdk.java.net.

Open a cygwin shell:

````
cd openjdk/jdk-jdk
hg clone http://hg.openjdk.java.net/jdk/jdk/
````

To follow my layout scheme, I rename the jdk to "source":
````
mv jdk source
````

Once this is done, it may be a good idea to create a tarball from the .hg folder in case you need to resurrect the unmodified sources:

````
cd source
tar -czf ../jdk-source.tar .hg
````

Unfortunately mercurial servers at openjdk.java.net are slow. In Europe, cloning a full repository takes ~40 minutes. On Windows, this process is also plagued by timeout errors.

#### The _faster_ alternative: copy sources and update

A faster alternative is to copy the repository from somewhere else and just update it. There are many ways to get a copy of the repository. One popular way is to download the source tarballs from: https://builds.shipilev.net/workspaces . 

These tarballs are maintained by _Aleksey Shipilev_ from Red Hat, thanks to him for this big time saver.

```
cd openjdk/jdk-jdk
wget https://builds.shipilev.net/workspaces/jdk-jdk.tar.xz
tar -xf jdk-jdk.tar.xz
mv jdk-jdk source        (note: rename folder to "source")
cd source
hg pull
hg update
```

It is a good idea to keep Alekseys tarball around. Even if the tarball is weeks old, it is still faster to extract it and pull updates instead of re-cloning the whole directory.

#### Resurrecting the sources

If you later want to restore the sources from scratch, extract the source tarball and restore the repository using `hg update`, then pull changes using `hg pull -u`. 


### Build

Open the cygwin shell. Create output directory:

```
mkdir openjdk/jdk-jdk/output
cd openjdk/jdk-jdk/output
```

Run configure:

Fastdebug:
```
bash ../source/configure --with-boot-jdk=/cygdrive/c/openjdk/jdks/jdk10 --with-debug-level=fastdebug
```

Release:
```
bash ../source/configure --with-boot-jdk=/cygdrive/c/openjdk/jdks/jdk10 --with-debug-level=release
```

_(Note: boot jdk path has to be absolute due to a [bug](https://bugs.openjdk.java.net/browse/JDK-8206125) in configure.)_

Run make:
```
make images
```

### Tips and Tricks

Re-run configure and rebuild from scratch without having to reissue the configure command:

```
make reconfigure clean images
```

Build 32bit:
```
--with-target-bits=32
```

Build Non-PCH:
```
--disable-precompiled-headers
```

Choose which Visual Studio installation to use:
```
--with-toolchain-version={2010|2013|2017}
```

Check which configure options a build was built with:

```
grep CONFIGURE_COMMAND spec.gmk
```

Speed up builds: During builds, the JDK being created is also used - to run jmod.exe, jlink.exe and other tools which need a close knowledge of the JDK source level to work correctly.

But this means that if you build naturally slow JDK variants (e.g. zero and slowdebug), the build itself can become slow. To counter that, one can specify a separate build JDK which is used instead of the just-built JDK. 

```
--with-build-jdk=<path to hopefully fast build jdk>
```

**Warning: In contrast to the boot jdk, this JDK has to be close to the sources!** A good practice would be to use a release build of the same source tree and source level.





