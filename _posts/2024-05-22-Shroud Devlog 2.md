---
title: Shroud - Devlog 2
date: 2024-05-22 09:08:01 +0800
categories: [Malware Development]
tags: [maldev,windows,shellcode]     # TAG names should always be lowercase
pin: true
---

## Scope Updates

I've been extremely busy since I started making a usable malware framework/wrapper about a month ago. I've started a new job and haven't had as much time to develop, and likely may not have as much in the future. As such, I decided to make the 
decision to consolidate my previous modular-malware framework Mallet into a CLI based tool called Shroud, centered around a singular shellcode dropper wrapper.

Frankly, this is probably what I should have done from the start. The modularity of Mallet was already rapidly growing out of control for me to handle (especially in C++). Instead, I have decided to focus on quality of techniques that I can pour into
a single dropper wrapper and general usability.

## Current State

I'm fairly happy with the usability of Shroud right now. It is a Linux based generation and compiler tool to inject either custom shellcode or `msfvenom` templates into an evasive wrapper. It works fairly well and is extremely easy to work with compared
to Mallet. Making changes to either the wrapper or techniques are much faster and realistic for me now.

The tool is admittedly pretty simple in this regard - it is just a python script to automate `msfvenom` shellcode generation, some IO for writing decryption schemes to files, and then the `GNU-C` cross compiler options. It may be simple, but its fast and
works pretty well.

## Move to Indirect Syscalls

I've got a lot more options for the future of the project now that I have downsized the scope significantly. My first major goal is to get rid of the custom `GetProcAddress` and `GetModuleHandle` implementations and use `SysWhispers 3` instead. I haven't used
SysWhispers very much to understand how signatured it is, but I figure indirect syscalls are a lot less likely to be flagged than whatever the hell I am cooking up. The implementation of this is likely going to be an issue due to the close interplay of
SysWhispers' output files and Visual Studio. I'd like to keep the project focused on Linux and very basic cross-compilation calls, so I'll try to find a work-around if necessary.

The swap to SysWhispers will also free up the opportunity to swap back to file mapping injection which is a favority of mine. In the current version, API calls are dynamically linked via custom load functions. The issue with this is that `InitializeProcThreadAttributeList`
returns an error BY DESIGN on its first call. This call is required for setting thread working directories and PPIDs. While this isn't incredibly necessary, I want some level of user-camoflauge for the coolness factor.

## Future Goals

After the SysWhispers integration I will be updating the script to randomly select from a variety of innocuous windows processes (`svchost.exe`,`ctfmon.exe`,etc) for the host process. The issue with this is that strings will be hardcoded. Unfortunately I'll
likely need to create a new custom hashed function for this, as a full image path is required for creating new threads.

After I make more process on the above issues I may work towards a DLL format, which I know is significantly less signatured for detection. After that I may create some optional user-level persistence payloads to bake in.

## UPDATE - GetProcAddress Signatured

Unfortunately, the custom implementation of `GetProcAddress` and `GetModuleHandle` have since been heavily signatured by Defender, and no doubt other EDRs. I've taken this time to step back from the project and focus on other things. Shroud is pretty easy to use and work on, so if I happen to create a more robust payload I can easily update the source files. Maybe I will in the future, maybe not.
