---
layout: post
title:  "FCEUX - The NES Emulator Compliant With Happy Chick"
date:   2018-10-11 01:06:30 +0800
categories: play
tag: nes emulator
---
I played some NES games in the past on my phone using Happy Chick app. I have some save records of several games. Now I don't want to Happy Chick app any more, I know there are plenty of NES emulators, but I want one can load my save record.

I can find the save record files on my phone under Xiaoji folder. At first, I tried several emulators on pc, they cannot load the record files. Then I tried emulators on phone, they cannot select my own record file. But after some time, I find what I want today.

I opened one sav record file just using any text editor. The first four characters were "FCSX". This was enough. Then I searched Google: nes emulator sav "fcsx". Some results contained following c++ code: 

```uint8 header[16]="FCSX";```

https://github.com/quarnster/emu-ex-plus-alpha/blob/master/NES.emu/src/fceu/state.cpp

https://gitlab.com/meishijiemeimei/Provenance/blob/80e69affc76647eef1fb2d3afef6091f34109917/PVNES/NES/FCEU/state.cpp

These links are open source projects without update for years. I'm not sure whether they can be compiled and run without problems.
Luckily, the first line of the page shows the name of the emulator: FCE Ultra.

After searched for "FCE Ultra", I downloaded 2 emulators - "FCE Ultra" and "FCEUX", the latter was introduced as "an evolution of the original FCE Ultra emulator". I tried the two, "FCE Ultra" could not load the save file, only "FCEUX" could.

So "FCEUX" is my answer. I don't know any other emulators at the moment. 