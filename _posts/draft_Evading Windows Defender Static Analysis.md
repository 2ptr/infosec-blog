---
title: Evading Windows Defender Static Anaylsis
date: 2024-04-04 09:08:01 +0800
categories: [Malware Development]
tags: [maldev,windows,shellcode]     # TAG names should always be lowercase
pin: true
---

## Static Analysis

Windows Defender, like most EDRs and AV solutions, has a static analysis component. This component has multiple detection techniques for analyzing suspicious files. The important techniques to keep in mind for evasion are file hashes and signatured code.

File hashes are what they sound like - Defender will compare the file hash of a file to a database of known-bad hashes. If it's in the database, the file is quarantined. Signatured code involves an actual review of the instructions within the file for known-bad blocks of recognizable shellcode. While relatively simple, this technique can prove to be the first major obstacle for individuals when it comes to evasion.

Fortunately, Windows Defender (and many other AV engines) static analysis techniques can be defeated with some basic malware obfuscation techniques.


## Why Your Payload Got Caught

Before we get started, lets talk about why your `msfvenom` PE payload didn't work. Everything online and open-source is signatured and hashed to hell by most AV solutions these days. There are certainly tools on GitHub that will help you get the job done, but nothing is going to beat making your own shellcode runner. Basic, custom methods of obfuscating your shellcode (even using `msfvenom`) has a signficantly better chance of working than something off-the-shelf online.

That being said, let's talk about how to actually start evading Defender.

## Running Shellcode

The first thing we need to do is just get our program to run custom shellcode. There are plenty of ways to do this but I'll just be sticking to creating a thread within our malicious process for simplicity.

We can use this cookie-cutter C program with Win32 API calls to accomplish the following:
- Allocate some memory in our running process
- Copy our shellcode (held in `.data`, currently) to the new memory buffer
- Open a new thread which executes the memory buffer

Note that I chose to create a `RunThread()` function to handle part of this process. This is completely optional.

```c
#include <Windows.h>

int main()
{

  return EXIT_SUCCESS;
}
```

Great - we can now run shellcode in a thread under a process. However, storing payloads in `.text` is not ideal for many reasons, especially considering the fact that we'll be encrypting it within this section later on.
Let's move our payload to `.rsrc`. There are several ways to do this, but a popular method is to import a `.ico` file into Resources if you are using Visual Studio. Regardless, just make sure you have the raw shellcode imported as a resource and identify the RSRC ID under `resource.h`.
It's important to realize that `.rsrc` is read only. We'll need to copy it to a 'working buffer' within our process if we want to make any modifications before execution.  For now, let's just copy it and execute it:

```c

```

## Basic AES Encryption

While our setup is functional, using most malicious shellcode will instantly get picked up by Defender and the file will be quarantined. Let's fix that.

