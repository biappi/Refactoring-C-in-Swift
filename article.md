# Refactoring a C application into Swift 


## Introduction

One of the most overlooked features of Swift is it's fantastic
inter-operability with C. So powerful that it should be possible to
convert an existing "legacy" codebase gradually, without compromising
the functionality of the application at any point of the process.

Of course, a complete rewrite of a codebase is never a goal for itself,
however, I believe that the ideas in this article are useful not only to
any Swift programmer, but also to C programmers that would like to approach
a more "modern" language, while retaining most of their knowledge and
toolset.

Even when using Swift exclusivly in the Apple ecosystem, at the time of
writing, all the major platform APIs are Objective-C or C, so understanding
some of the C inter-operability will help in using, for example Core
Graphics, or the Keychain API.

For the people more present in Open Source communities, the
inter-operability will allow using any of the C software from Swift,
and people that are from, ad example, Linux or BSD communities might
over-estimate the difficuitly of using all the software they are familiar
with, with a modern and safe language.

Closing this already long introduction, to longtime C programmers that
are curious about Swift but are not familiar with the Apple ecosystem, I
would say, with my funny teaser voice, "And all this comes for free.
At link time. You are welcome".


## Our "Legacy" Project

The codebase we are going to work with is the source code of "Tabboz
Simulator", a simple and "fun" tamagotchi-style game, where "fun" is
a quintessential Italian trash humor`from the late nigthies, as gross and
juvenile as a teenager of the era in full hormonal swing could be.

Simply put, I love it. While recognizing some might not share my
enthusiasm. And I have recently noticed it was released as GPLv3 by
one of the original authors, Andrea Bonomi, on github.

Grazie, Andrea!

  * http://www.aabo.it/house/katzate/tabboz/tabboz.htm
  * https://github.com/andreax79/tabboz

To share and revive a personal piece of history, I'm going to try to
port it from its current form of a Windows C application to a runnable
Swift program that implements the game.

The compromise is, while I will not attempt to port the UI, which is as
important as any other feature of the game, I will try to document how
wide the spectrum of interaction between C and younger languages like
Swift, and its Rust, Go and other brothers and sisters.


## Takeoff! Clone and first impressions

Lets clone our upstream repository and take a look at what we're working
with.

    willy@Thala  ~$ git clone https://github.com/andreax79/tabboz.git
    Cloning into 'tabboz'...
    remote: Enumerating objects: 70, done.
    remote: Total 70 (delta 0), reused 0 (delta 0), pack-reused 70
    Unpacking objects: 100% (70/70), done.
    willy@Thala  ~$ cd tabboz/
    willy@Thala  tabboz master$ ls
    BWCC.DLL      lavoro.c      tabimg.c      tabs0402.wav  tabs1300.wav
    BWCC32.DLL    net.c         tabs0000.wav  tabs0501.wav  tabs1500.wav
    LICENSE       net.h         tabs0101.wav  tabs0502.wav  tabs1501.wav
    README.md     newproteggi.c tabs0102.wav  tabs0503.wav  telefono.c
    REGIS32.DSW   obsolete      tabs0201.wav  tabs0504.wav  tempo.c
    TABBOZ.DSW    os.h          tabs0202.wav  tabs0602.wav  tipa.c
    TABBOZ.IDE    prompt.c      tabs0203.wav  tabs0603.wav  vestiti.c
    TEXT.RES      proteggi.c    tabs0204.wav  tabs0604.wav  zarrosim.c
    ZARRO32.RES   readkey.c     tabs0205.wav  tabs0701.wav  zarrosim.def
    bwcc.lib      resource.h    tabs0302.wav  tabs0801.wav  zarrosim.h
    bwcc32.lib    scooter.c     tabs0303.wav  tabs0900.wav
    disco.c       scuola.c      tabs0304.wav  tabs1100.wav
    eventi.c      tabboz.exe    tabs0305.wav  tabs1200.wav
    willy@Thala  tabboz master$ 

This codebase is "deceivefully" well written, so much that I would like
to underline a few of its positive features, while disclosing -- not that
it would matter -- that I do not personally now the original author.

Squinting in what at first glance might see a lazy dump of all files in
no particular order, we might find the usual suspects of an application:

  * a binary build
  * distribution files (README, LICENSE, et similia)
  * build system support files
  * platform support source files
  * application source files
  * resources

There are some features that warrant highlighting. We can easily discern
by the filename alone between the support source files and the familiar
scenarios of the game. And if, as I suspect, you're oblivious to obscure
Italian pop-trash references, can easily serve as a description of
what kind of tamagotchi game we are talking about.

  * `Tabboz`, `Zarro` Being impartial, I will only quote the current Urban
    Dictionary definition:

    > The zarro, always uneducated, impolite and probably a fan
    > of soccer/football, loves to drive a scooter and goes around
    > with other zarri making noise, smoking pot and looking for
    > cocaine.

  * `disco.c` Club
  * `eventi.c` Events
  * `lavoro.c` Job
  * `scooter.c`
  * `scuola.c` School
  * `telefono.c` Mobile
  * `tempo.c` Time
  * `tipa.c` Girlfriend
  * `vestiti.c` Clothing

As you can see, not only we are in for a treat, but let's look at how
much did we gather from a single directory listing that easily fits a
standard terminal. The structure of the distribution, what files contains
which business logic, a small synopsis of the domain model of the
application.

A notable absent is any mention of "middleware" platforms, and no
mention of a "patternized" architecture, suggesting the author chose its
targeting platform wisely, and used it directly for his implementation,
substantially reducing the amount of code to maintain for him, and to
understand for us.

In case you only speak English, my reader friend, let me tell you you are
missing an entire universe of funny holy wars about when, how and why
use what language for what filename, function or variable name, ask your
nearest multi-lingual coder friend about it!


Tools of the trade, and plan of action
======================================

Some disagree, but I belive that the advantages of using an IDE fairly
outweight the lack of "integration" of using separate tools, especially
in a code refactoring excercise like the one at hand, in which we'll have
to quickly wrangle the creation of a build system, bridging between two
languages, and troubleshooting building and linking errors.

The obvious choice falls on the first-party Xcode, which is excellent and
free. With excuses to people who not have access to Xcode, but the
knowledge required to troubleshoot an installation of a Swift runtime
can overlap to the knowledge of translating what the IDE will do under
the hood.

So, let's finally begin for real by addressing one of the elephants in the
room. This is a Windows application, I do not have access to a runtime
to test the build, nor a build environment to try and build the default
configuration.

This might be a crazy idea, but a cursory look at the files show that
the content of the files look exactly like their directory structure:
very well made, very understandable, no "middleman" layers of
abstractions, again a testament of how much simplicity always pays off,
and kind of a lost art, as one would cynically say.

For now, we can envision a fuzzy goal of a text based version of the game,
or maybe a more generic abstraction of the game logic, written somewhat
in Swift.

If that's the fuzzy goal, our first plan is almost forced: we need to get
the codebase to compile, maybe link, with any means necessary. Then,
optimistically, future us will be closer to a fuzzy goal.


Project Scaffolding
===================

It is good form, if overkill for this exercise, also because upstream
looks like a "dead" dumped artifact, so:

    willy@Thala  tabboz master$ git tag original

It is a Swift naming convention that namespaces like `class`, `struct`,
and `enum` are capitalized, so I believe it follows that other namespaces
like files and directories should be named accordingly.

This means we're going to change the directory structure that's served
so well in all those years to something that's more conventional in its
new home.

Let's archive it in a folder and keep it there as a source to cherry-pick
the files we will use.

    willy@Thala  tabboz master$ mkdir Original
    willy@Thala  tabboz master$ mv * Original/
    willy@Thala  tabboz master$ git add .
    willy@Thala  tabboz master$ git commit -m'Archiving original source'

Finally opening Xcode, let's create a new project: macOS, Command Line Tool
will do just fine for now.

    willy@Thala  tabboz master$ git commit -m'Adding Xcode project'
    [master 9d51fee] Adding Xcode project
     4 files changed, 315 insertions(+)
     create mode 100644 Tabboz Simulator.xcodeproj/project.pbxproj
     create mode 100644 Tabboz Simulator.xcodeproj/project.xcworkspace/contents.xcworkspacedata
     create mode 100644 Tabboz Simulator.xcodeproj/project.xcworkspace/xcshareddata/IDEWorkspaceChecks.plist
     create mode 100644 Tabboz Simulator/main.swift
    willy@Thala  tabboz master$ 

And here's a familiar starting point, our default Hello World template.

As the last "administration" task, let's fork the original upstream repo
and do all the favorite git gymnastic, and pronto, we can push to our
forked repo.

    willy@Thala  tabboz master$ git push
    Counting objects: 2, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (2/2), done.
    Writing objects: 100% (2/2), 239 bytes | 239.00 KiB/s, done.
    Total 2 (delta 1), reused 0 (delta 0)
    remote: Resolving deltas: 100% (1/1), completed with 1 local object.
    To https://github.com/biappi/Tabboz-Simulator.git
       f0c4b93..420c8e0  master -> master
    willy@Thala  tabboz master$ git lg

This, of course, means that the repository I am going to work with is
going to be at: https://github.com/biappi/Tabboz-Simulator


Importing the C Files
=====================

In Xcode import a C implementation file into a group, this is one way to
create a bridging header.

From the point of view of a C programmer, this header is a C file the
resulting object will be part of an imaginary translation unit spanning
the entire Swift program. For a Swift programmer, it is the file used to
import C and Objective-C libraries to be used in Swift.

I will continue mechanically importing all the C files in Xcode, resisting
the temptation of cherry-picking only the business-logic files, as this
will present to the Xcode indexer a better view of the code.

    willy@Thala  tabboz master$ git commit -am'Adding C files'
    [master 83cda99] Adding C files
     22 files changed, 9125 insertions(+)
     create mode 100644 Tabboz Simulator/C/Tabboz Simulator-Bridging-Header.h
     create mode 100644 Tabboz Simulator/C/disco.c
     create mode 100644 Tabboz Simulator/C/eventi.c
     create mode 100644 Tabboz Simulator/C/lavoro.c
     create mode 100644 Tabboz Simulator/C/net.c
     create mode 100644 Tabboz Simulator/C/net.h
     create mode 100644 Tabboz Simulator/C/newproteggi.c
     create mode 100644 Tabboz Simulator/C/os.h
     create mode 100644 Tabboz Simulator/C/prompt.c
     create mode 100644 Tabboz Simulator/C/proteggi.c
     create mode 100644 Tabboz Simulator/C/readkey.c
     create mode 100644 Tabboz Simulator/C/resource.h
     create mode 100644 Tabboz Simulator/C/scooter.c
     create mode 100644 Tabboz Simulator/C/scuola.c
     create mode 100644 Tabboz Simulator/C/tabimg.c
     create mode 100644 Tabboz Simulator/C/telefono.c
     create mode 100644 Tabboz Simulator/C/tempo.c
     create mode 100644 Tabboz Simulator/C/tipa.c
     create mode 100644 Tabboz Simulator/C/vestiti.c
     create mode 100644 Tabboz Simulator/C/zarrosim.c
     create mode 100644 Tabboz Simulator/C/zarrosim.h
    willy@Thala  tabboz master$ 

After a build, the error is a single one, but quite dramatic:

    fatal error: 'windows.h' file not found

Since it seems to be included from the `os.h` file only, let's hijack
it as our interface file. It was clearly its own intended use, even if
the syntax errors in the `#ifdefs` betrays the fact it was probably never
used.

Taking it as an invite, not daunted by the task, and basking in nostalgia
for `AMIGA` and `WIN_16`, let's jank the file.

This is also a nice moment to check if the `General` >
`Continue building after errors` preference is enabled in Xcode, as it
will be useful during the whack-a-mole game of stubbing out the
interface the application depends on.

Yay, Xcode is now lighting with a glowing red "21 issues" mark. This is
progress.

    willy@Thala  tabboz master$ git commit -m'Removing windows.h' -a
    [master 438fc11] Removing windows.h
     1 file changed, 1 insertion(+), 52 deletions(-)
     rewrite Tabboz Simulator/C/os.h (99%)
    willy@Thala  tabboz master$



