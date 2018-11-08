# How to set up Intellij for working on the OpenJDK java sources

This document describes how I set up Intellij for OpenJDK. Note that I mainly work on Linux and have not tested this on other platforms. I assume this works the same way anywhere but no guarantees.

## Run the generator script

You need the OpenJDK sources cloned and an OpenJDK freshly built. I usually use a fastdebug build for that - not sure if that makes any difference.

Before letting the generation script run: the script assumes that the JDK is built into the source directory itself (so, configure and make have been called from within the source root). This seems to be a common setup at Oracle.

But if you, like me, do not follow that setup and instead prefer your output directory to be separated from the source directory, you need to create a link in the source directory to the output directory. For example, assuming this setup:

```
  |--<repository, e.g. jdk-jdk>
               |--source
               |--output-fastdebug
               |--output-slowdebug
               .....
```
we would need to create the link the generator expects like this:
```
cd source
mkdir build
cd build
ln -s ../../output-fastdebug linux-x86_64-normal-server-fastdebug
```
Then from within source root let the idea.sh script run (need ANT for that):
```
cd source
export ANT_HOME=<ant home>           # ANT: I currently use 1.10.3
bash ./bin/idea.sh 
```
That's it. You should now have a .idea directory in the source root:
```
thomas@mainframe /shared/projects/openjdk/jdk-jdk/source $ ls -ald .idea
drwxrwxr-x 6 thomas thomas 4096 Nov  8 07:57 .idea
```

## Open the project in Intellij

Start intellij and open the project ("Open" -> choose source root folder, should have a little black mark indicating it contains an .idea folder).

Setup "Project SDK" aka JDK to use: 

Open Project Structure Dialog (File->Project Structure or [ctrl+shift+alt+s]). Switch to "Project" pane. Select the JDK to use from the dropdown menu: For openjdk12 development, at the time of this writing, this should be a JDK11.

If no JDK is available in the Dropdown box, add a new JDK ("New...").

Once the indexer did run, we are done. 

Java sources are parsed and indexed, and you should be able to work fine with the majority of java files - code completion and -analysis should all work now.

## Known issues:

- There are gray crosses at the source directory icons - they can be ignored. They indicate that the source directories are excluded from building and this is fine since Intellij shall not attempt to build the packages itself.

- Openening two module-info.java files will result in red sqiggles - "module-info.java already exists in the module". That is a known issue and being worked on.

