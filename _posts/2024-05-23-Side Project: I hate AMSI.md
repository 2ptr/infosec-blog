---
title: Side Project - I hate AMSI
date: 2024-05-23 02:08:01 +0800
categories: [Malware Development]
tags: [maldev,windows,shellcode]     # TAG names should always be lowercase
pin: true
---

![I hate AMSI](/assets/img/ihateamsi.jpg)

## Inspiration

Recently I decided I wanted to dive into another certification, this time focused on Active Directory abuses. I have done quite a lot of AD attacks and feel 
pretty comfortable in red team scenarios but I had yet to put my skills to the test. I decided on the CRTP - Certified Red Team Professional, offered by
AlteredSecurity. The course so far is quite good, but interestingly focuses a lot on using PowerShell primitives and not touching disk.

## G.O.M.D (Get Off My Disk)

Up until the CRTP course, I had an admittedly pretty nooby approach to tooling and 'evasion'. I would get a foothold on a machine, escalate to SYSTEM if possible,
and then just flood a random user's desktop with various tools of destruction. Of course, some of this could be avoided if you use a C2 (I use metasploit for easy stuff
and Havoc for anything more involved, although I can't stand its beacon syntax), but only if you use an in-memory (scripted) method for a dropper.

If I did this in a real enterprise network then I would be fired by lunch. FileCreate (Sysmon 11) events are great at exposing pentesters who think they're slick by
using `certutil` to download a file called `Not-Mimikatz.exe`. As such, we should probably stick to memory if we want to survive and get DA one day. If this is your
first experience trying to not touch disk, this is much easier said than done.

## AMSI

Everyone knows that if you drop a virus executable on your desktop, Windows Defender will pop up and say "hey that's a bad file" (maybe). But what about bad commands? 
Or PowerShell scripts that never actually touch your desktop? 

Windows uses AMSI, the Anti Malware Scanning Interface, to handle in-memory or scripted executions. Each time you submit a PowerShell command or run a `.ps1` script,
AMSI will take your command and hand it over to the registered anti-virus solution on your machine. In most cases, this is Microsoft Defender. Therefore it should be clear
that "bypassing AMSI" typically means "bypassing whatever AV is installed on your system" (usually Defender).

## I hate AMSI

There are a LOT of ways to try and bypass AMSI. AMSI is loaded via a DLL into the PowerShell process, and as such its exports are availble to be manipulated in the processes'
memory. Unfortunately, manipulation of internal AMSI logic and provider links has been so exploited that the DLL's memory is heavily checked and monitored now.

Instead of the more involved approach of memory modification or forcing errors, we can also just attempt to modify our malicious scripts to go undetected by Microsoft
Defender. This is admittedly much less cool and interesting, but I think it is a great example of how important the idea of 'custom' is when it comes to malware. The
obvious caveat is that this doesnt actually 'bypass AMSI' per-se, but only passes the interfaced checks provided by Defender through AMSI - this won't work for other
AV solutions.

To do this, I used [DefenderCheck](https://github.com/matterpreter/DefenderCheck). The tool will break a binary or script down into byte chunks and submit them to
Defender for detection. If it flags, then clearly something in that byte chunk is triggering the AV. Most of the time this is usually just a bad string literal or
signatured variable name.

For example - `winPEAS.bat` is correctly recognized as malicious when you download it straight out the repo. However, Defender is so incredibly reliant on strings
and byte chunks as signatures that changing the _variable name used for terminal color output_ from `ColorLine` to `ihateamsi` is enough to bypass checks entirely.

```PowerShell
:ColorLine
SET "ihateamsi=%~1"
FoR /F "delIms=" %%A In ('FORFILES.EXE /P %~dp0 /M %~nx0 /C "cMd /C eChO.!ihateamsi!"') DO ECHO.%%A
EXIT /B
```

Some bypasses are a little more involved, admittedly. `Invoke-Sharphound` is a classic .NET assembly reflective loader with a giant base-64 blob in it. There are 
some hard-coded strings in the script that are necessary for reflective loading which get picked up by Defender (you can verify this with `DefenderCheck.exe`). After
messing around for a while, turns out you can just base-64 decode the encoded versions of these strings and the script will appear non malicious!

I plan to make some bypassed versions of popular tools like `PowerView` as well as a generally obfuscated .NET assembly loader for stuff like `Rubeus`.
