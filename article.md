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


