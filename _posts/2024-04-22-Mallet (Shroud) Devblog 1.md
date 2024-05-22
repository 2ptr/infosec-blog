---
title: Mallet - Devblog 1
date: 2024-04-22 09:08:01 +0800
categories: [Malware Development]
tags: [maldev,windows,shellcode]     # TAG names should always be lowercase
pin: true
---
## Project Overview

If you play CTFs a lot, you may run into the occasional machine that has Windows Defender enabled. Maybe you use this opportunity to learn a bit more about malware development and evasion, as I did. You can craft a little encrypted payload dropper, compile, and then execute and move on with your life.
But then, later down the road, you run into it again. This time it isn't as exciting. You'll have to painstakingly re-assemble your dropper with new shellcode and re-compile, just to get a foothold. 

I got tired of doing this and I am sure many of you are tired of doing it as well. I decided to make a one-stop shop tool that will take any shellcode you've got and turn it into an evasive dropper tailored to your specific needs.

The catch is that I barely know what I am doing. Mallet is certainly going to be a useful tool, but it is also intended to be an educational 'framework' for me to explore new techniques with. I plan to make a new devlog post every time I manage to get a new feature or technique added (HTTP staged payloads, for instance).

## Where We're At

As of right now, this is what we have - a project written primarily in C++ using the Qt framework for a GUI and only basic evasion features:

![Main menu](/assets/img/mainmenu.png)

DLLs aren't supported. RC4 and XOR aren't implemented. The HTTP staged payload isn't written at all. I don't even know what to replace the Win32 calls with for the NT API feature.

We have a long, long way to go. But that doesn't mean it was easy to get to version 0.2 (current). 

## How We Got There
As someone who has only ever had experience with C (NOT C++), Qt was an interesting weekend learn. The designer software is incredible and the framework is unbelievably powerful, but only after you spend 7 hours trying to learn how it works.

After you spend a weekend afternoon making a message box say "HI :)", you may wonder why your generated PE doesn't seem to work. Qt generated projects require special dependencies. You might think these are only the DLLs it says it can't find upon attempting to launch, like `QtGuidh6.dll`. Nope. You're missing way, way more.

I tried for a couple hours to statically link necessary libraries during compilation. I looked at building a [static Qt](https://wiki.qt.io/Building_a_static_Qt_for_Windows_using_MinGW), but just gave up after a while and finding no support online.

Instead, I gave in and decided to go with dynamic linking, but even then I didn't know how to get my PE to work. Apparently, despite no indication of this on Qt's documentation (that I could uncover), there is an included binary called `windeployqt.exe` in the Qt source used for this exact purpose. [This](https://www.youtube.com/watch?v=rFHPOZoqzcg&pp=ygUMUXQgd2luZGVwbG95) YouTube video saved my life.

After getting the Qt issue resolved, I had a nice looking GUI with absolutely no functionality. I had previously written the basic features of Mallet in a separate C project. Porting existing C code to a C++ project should be no problem, right? Just add in a parsed struct for user options from the GUI and keep application logic the same! No problem!

Wrong. Like I said, I have no experience with C++. At first, I just tried to keep the project as a mix of C and C++ files with `extern "C"` for include headers. I clicked build solution and got pretty much every type of error there is. I spent 30 minutes typecasting random function parameters (as apparently there is a data type called "Qstring"...) and learning how to convert C++ std::String to char*. Then I had to consolidate and repopulate my `Common.h` header file. And still, it did not work.

Once all errors were addressed, a new one popped up. "Unresolved external symbol". I tried for hours, but I couldn't fix this one. Something about mixing C/C++ projects, compiler flags, includes, I don't know. 

I decided to just port the entire project to C++ and make necessary updates. This turned out to be a good move, as working with `string` and C++ libraries is a lot nicer than raw C. Application logic worked after some tuning. I decided that instead of having the user fish for `Mallet.exe` in the sea of files under `Release`, I would just start the program from a bat file. This works well, but is a little disappointing compared to the one-stop-shop I had in mind.

## Where We're Going

My end goal is a statically linked, standalone PE toolkit. I can work on this eventually, but right now I want to finish the framework-y aspect of this project and actually learn more about malware development. The produced dropper files are completely fine right now but could be significantly improved. I want to be pumping out VirusTotal 0/70 detection droppers at the click of a button.
