---
layout: post
title:  "Memory dumps and Java debugging tools on Debian"
date:   2021-01-27 00:02:06 +0200
---

Note: This is a generally straight forward description of adding a new package repo and authenticating it, no revelations here. 
I just wanted to save you the time that I spent looking and constructing a solution (because no complete workaround exists on the web), and write a generic end-to-end process. 

---

The official `openjdk8` Docker image (`FROM openjdk:8-jre`) is built on Debian. This base layer includes a minimal runtime environment for Java 8 applications. When trying to debug a Java application we will sometimes need to extract a Heap memory dump to analyze and debug. This can be done easily using `jmap`:

```
jmap -dump:format=b,file=memdump.hprof JAVAPID
```
Starting from the official `openjdk-8` base image would probably result in:

```
jmap: command not found
```
Indeed, looking at the `$JAVA_HOME` folder reveals that:

```
>> ls $JAVA_HOME/bin/
java  jjs  keytool  orbd  pack200  policytool  rmid  rmiregistry  servertool  tnameserv  unpack200
```
Browsing to the Debian [man pages](https://manpages.debian.org/unstable/openjdk-8-jdk-headless/jmap.1.en.html) will tel us that `jmap` is included in the `openjdk-8-jdk-headless` package. But trying to install it results in:

```
>> apt install openjdk-8-jdk-headless
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package openjdk-8-jdk-headless
```
The package has apparently removed from the official Debian repositories.
The solution would be to add some external repository which hosts the `headless` package. One such repository is the [MXLinux repo](http://mxrepo.com/):

1. Adding the MXLinux repo to the sources list. Create a new file under `/etc/apt/sources.list.d` named `mx.list` and add the following line to it:
```
deb http://mxrepo.com/mx/repo/ buster main non-free
```
2. Updating `apt` will result in:
 ```
 >> apt-get update
 Err:4 http://mxrepo.com/mx/repo buster InRelease
  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 276ECD5CEF864D8F
Reading package lists... Done
W: GPG error: http://mxrepo.com/mx/repo buster InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 276ECD5CEF864D8F
```
3. We need to import the PGP Key:
```
gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv  276ECD5CEF864D8F
gpg --export --armor 276ECD5CEF864D8F | apt-key add -
```
4. Now we can successfully `apt-get update`
5. And install: `apt-get install openjdk-8-jdk-headless`

Now we have `jmap` among other useful java debugging tools like `jcmd`.

