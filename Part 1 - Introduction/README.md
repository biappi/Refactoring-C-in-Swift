# Introduction

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


# Our "Legacy" Project

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

