# Jotunheim SDK Design
**Goals**

Jotunheim SDK aims to be a one stop shop for the development of custom software and Operating Systems.  Taking a set of input Jotunheim SDK (The SDK) seeks for anyone to be able to quickly setup a framework for their custom software and then execute a build of the software on their current machine.  It also seeks to be able to allow a creator to pull in external sources of custom software and build a distribution.  It seeks to borrow heavily from existing package managers in concept but will be a complete overhaul of the current environment by allowing you to combine sources from different locations (git, svn, hg, source, or binary to name a few), catalog them all and combine them into a working piece of software.

**Background**

Primarily the system will be written in Rust, with a possible Flutter frontend (*still debating if it is going to be a mono-language program or if we want to build it in multiple languages*).

Rust will allow us to provide a stable base for all of our development needs and will provide us with the opportunity to develop low-level code and high-level code in tandem.  Golang was floated as a possible language to accomplish this task too but ultimately we settled on Rust as the de facto standard for the SDK.

The SDK seeks to provide a shell independent of the current Operating System in a sandboxed environment that can act as a build system.  It will come with a standard set of commands that will be familiar to anyone who comes from a Unix based or Unix like system.

There is also a planned Graphical User Interface (GUI) for people who are not comfortable working within a command line interface (CLI), however it should be noted that not all CLI options may be available within the GUI as quickly as the CLI as the CLI will be the one serving as the backbones for the GUI and will be developed at a slightly faster pace.

This will not prevent the GUI from suffering from a lack of features but rather that we seek to stabilize the CLI version updates and work out any issues before releasing the GUI version with the updated features.

**Overview**

So here we seek to establish the basic controls of the framework.  The general framework can be found at [https://github.com/Aiatar/jotunheim_sdk_docs](URL) within the Framework directory, and will be updated with new features and the general *working* direction for the SDK.

The controls for the SDK, as metioned earlier, will center around the CLI.  The SDK will provide a sandboxed environment with a custom set of commands and controls in order to better serve the user.  It should allow you to do everything that you need to be able to do in order to build the specific software that you seek to buiild.

The custom OSes that will be available with the 1.x release will be no less than custom Linux and BSD  OSes.  Out of the box day one you should be able to spin up the software and create a simple config recipe using a set of prompts.  The goal being that you will be able to enter a command like: `new-recipe jotuheim-sdk` and the system will open a base config that you customize with all of your options.

Once we formalize how the config file will be structured, we will publish a best practice and you will be able to use a command similar to: fire jotunheim-sdk -C jotuheim-sdk.rec and it will be able to build it using the recipe file stored at jotunheim-sdk.rec.

Now comes the fun part, how does the system know how to build what you want it build?

The magic is in the batter that you use.  So cooking is a passion for the creator of this software.  Recipes are a way to make sure that you get the same relative results everytime that you make the recipe.  Everyone has an idea of a way to make a recipe better, and in this thought we encourage people to share their recipes with friends, family or anyone that you might feel inclined to so that as a community we can grow.

As a sign of goodwill, anything and everything created by the Alfheim Software Initiative team, in conjunction with the SDK, will be released under the MIT license and will be available to all to use and build your own custom software.

Ok but you didn't tell us how the SDK works, how are you going to be able to produce an SDK that will be able to create custom software?

By utilizing modern technologies, the idea is to be able to create enough use cases that we will be able to fill the major areas of modern software development needs.  That is a whole lot of mumble jumble meant to impress so let's break down the concept using an example recipe (mind you this is not the final format and will be heavily modified going forward, for more in depth concepts about the framework please visit the aforementioned Framework documetation on Github.)

```
Example Recipe:alfheim_linux

name: alfheim_linux
developer: Alfheim Software Initiative
maintainer: ASI Developers <alfheim(dot)software(dot)initiative(at)gmail(dot)com>
repo: https://github.com/Aiatar/alfheim_linux
release: master
prog_lang: rust
lang: en
base: endeavour_os
ver: nightly_build_0.0.1
build_dir: .build/($name)
dep_dir: ($build_dir)/depends
conf_dir: ($build_dir)/config
debug_dir: ($build_dir)/debug
rel_dir: ($build_dir)/release
depends:
    - jotunheim_sdk = nightly
    - rust = nightly
    - chrome = dev
    - retext >= 8.0.0-2
    - github-desktop >= 3.2.7-2
    - pulsar >= 2.11.0-1
    - guake >= 3.10-1
config:
    pull.base
    dl.depends: ($depends)
    bake: ($base)+($depends) -> ($name)_($ver).iso
conf_sec:
    chktmp: sec_hash ($build_dir)/release/($name)_($ver).iso->($rel_dir)/sec_hash : comp $1 $2 : ok=[rm ($debug_dir)/($name)_($ver).iso] else=[($red) :: "Software didn't pass security checks, removing release file"] ;[rm ($rel_dir)/($name)_($ver).iso]
fire:
    create.dir: ($build_dir) ($dep_dir) ($conf_dir) ($debug_dir) ($rel_dir)
    get.depends: ($dep_dir)
    copy.config: ($conf_dir)/($name).rec
    cook: cd ($debug_dir) : hel-fire -c ($conf_dir)/($name).rec -gen sec_hash->sec_hash : cp ($debug_dir)/($name)_($ver).iso -> ($rel_dir)/($name)_($ver).iso : ($conf_sec) ($debug_dir)/sec_hash
sale:
    git_rel alfheim_linux_latest.iso ($rel_dir)/($name)_($ver).iso
```
This is just a rough outline of how the recipes are currently being imagined.  Let's take a look at the many moving parts of this one at a time:

```
name:
    This is the name of the project, it can also be the name
of the resulting package but that is entirely up to the person
designing and writing the recipe.  If you wish to have a
different release name you can use the *name_rel* tag.
```

```
developer:
    This is the developer of the project.  Sometimes like in
the case of Arch Linux AUR packages the developer is different
from the maintainer, if they are the same you can use the same 
name in both spots.
```

```
maintainer:
    This is the maintainer of the recipes.  Note: the standard
is for an email address to be included after the name of the
maitainer in case someone needs to get ahold of you for a bug.
```

```
repo:
    This is where the code is maintained for the recipe.
```

```
release:
    This refers to the release of the repo that you are
pulling from and does not refer to the release version of the
created software.
```

```
prog_lang:
    This is the programming language that you wish the final
product to be written in.
```

```
lang:
    This is an optional field and can be set to target certain
spoken/written languages.
```

```
base:
    This is a setting for custom OSes particularly.  It sets
the base software that you wish to modify using the current
recipe.  In the future it is planned that it will expand to
include other software types.  It is of particular interest
that you can include your own software as a base by using the
following format: base: <software_name> = <download_location>
Note: due to the nature of closed source software it will be
entirely impossible to include it in the list of options for
customizing.
```

```
ver:
    This is the release version number.
```

```
build_dir \
dep_dir \
conf_dir \
debug_dir \
rel_dir:
    These are the directories where the software is built,
these can be changed based on the particular developer needs
for the build, however, it is recommended that you keep
build_dir the same.
```

```
depends:
    These are all of the dependencies for the particular
recipe to successfully complete.  In the example the base
system is Endeavour OS and that comes standard with a lot of
goodies, we are only interested in really adding the following 
packages in addition to what is already offered.  Note: here
instead of including version numbers it would be better to
include download locations with version numbers. This would
look like: depends: chrome = www.google.com/chrome/download.
```

```
config:
    Is the way in which the software is to be built.  This is
a very BASIC config, when you are actually building your
software your config will and should be much more advanced
than this one.
    pull.base is a built-in command to download the base.
    dl.depends is a built-in command to download the
dependencies of the project.
    bake is the built-in command to combine the base and
depends and output the file on the right.
```

```
conf_sec:
    This checks to make sure that the security hash of both
the debug version and the release version are the same before
releasing the software.  Also this security hash will be
uploaded with the release so that others can check to make
sure that they have the correct software.
```

```
fire:
    Here is where all of the before work gets all mixed
together and made into a release.
    create_dir creates the necessary directories for build
and release.
    get.depends downloads the dependencies into the dep_dir.
    copy,config copies the $(config) into a usuable recipe.
    cook is the command to actually build the software.
    hel-fire is a built in command that takes the config and
builds the software according to the config, and generates a
security has (using sec_hash of the created file.
    Then the file gets copied from the debug directory to the
release directory and we run conf_sec to confirm that the two
hashes match.
    If the match the debug file is removed, if they dont the
release file is removed.
```

```
sale:
    This is the final step and here the SDK can release the
newly created software for you.
```

This is a very simple example but it outlines the basics of the SDK recipes for you.  You are warned again, this is not the final version of this specification, and as such this can all change as needed.  There will also be a test suite that will be implemented between fire and sale, but as of the creation of this document has not currently been completely formulated.

Now I have the SDK installed and running and I have imported my recipe into the system, how do I run the recipe in the SDK?

***WARNING: This is a proof of concept section and is not intended to inform you how to use any of the releases of the SDK, for that please visit: [https://github.com/Aiatar/jotunheim_sdk_docs](URL) and view the documents in the manuals folder.***

Ok so let's say you have the SDK CLI version installed and everything seems to be running well.  First thing you want to do is a test build.  To do this the SDK will provide a group of test recipes built into it.  You will simply type `preheat` and it will test your setup using three different recipes.  If you want to test for a certain programming language you can use `preheat <prog_lang>` and it will run three specific tests for that language.
