---
title: "Never take candy from strangers"
date: 2016-02-14
author: Luigi Vezzoso
layout: post
tags: 
    - CVE
    - vulnerability
category: posts
---

In these days I’m testing some tools used to evade antivirus (both client AV and gateway AV). The result is very impressive: the basic tools can evade most of antivirus.

One of the tool is [shellter](https://www.shellterproject.com/).

Shellter is a dynamic code injector inside [PE windows file](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680547%28v=vs.85%29.aspx)

From the shellter project website:

*Shellter takes advantage of the original structure of the PE file and doesn’t apply any modification such as changing memory access permissions in sections (unless the user wants), adding an extra section with RWE access, and whatever would look dodgy under an AV scan.*

Shellter uses a unique dynamic approach which is based on the execution flow of the target application, and this is just the tip of the iceberg.
Shellter is not just an EPO infector that tries to find a location to insert an instruction to redirect execution to the payload. Unlike any other infector, Shellter’s advanced infection engine never transfers the execution flow to a code cave or to an added section in the infected PE file.

It has a lot of features:
- Compatible with  Windows x86/x64 (XP SP3 and above)  & Wine/CrossOver for Linux/Mac.
- Portable – No setup is required.
- Doesn’t require extra dependencies (python, .net, etc…).
- No static PE templates, framework wrappers etc…
- Supports any 32-bit payload (generated either by metasploit or custom ones by the user).
- Compatible with all types of encoding by metasploit.
- Compatible with custom encoding created by the user.
- Stealth Mode – Preserves Original Functionality.
- Multi-Payload PE infection.
- Proprietary Encoding + User Defined Encoding Sequence.
- Dynamic Thread Context Keys.
- Supports Reflective DLL loaders.
- Embedded Metasploit Payloads.
- Junk code Polymorphic engine.
- Thread context aware Polymorphic engine.
- User can use custom Polymorphic code of his own.
- Takes advantage of Dynamic Thread Context information for anti-static analysis.
- Detects self-modifying code.
- Traces single and multi-thread applications.
- Fully dynamic injection locations based on the execution flow.
- Disassembles and shows to the user available injection points.
- User chooses what to inject, when, and where.
- Command Line support.
- Free

## The Test and related reports

After injecting a meterpreter payload inside a standard putty.exe software to have a reverse TCP connection to the attacker workstation, I have tried to send It trough firewall with AV and advanced sandboxing technics

I submitted also the sample to different well know antivirus/sandbox available on the Internet.

The hash of the bad file is **ce607a29e8079592c9d412cb6308cd9cdb93af74**

![windows defender](assets/postimages/windows-defender.png)

An other submission to a major security vendor did not detect the malicious network activity neither the evil payload…

![checkpoint](assets/postimages/checkpoint.png)

The most accurate test was performed by Virustotal where two of the many engine have detected correctly the evil behavior of the executable:

![virustotal](assets/postimages/virustotal.png)

In particular it has detected also the remote connection to the attcker machine.

![remote-shell](assets/postimages/remote-shell.png)

## Some considerations

Finally some obvious raccomandation

- don’t install software if you have a slightest doubt about the provenience (be aware of cracked software!!!!)always verify the software producer hash of the software (if available)
- don’t trust only in the technology…. spent money to the user awareness
- build a stratified security
- perform frequent VA/PT
