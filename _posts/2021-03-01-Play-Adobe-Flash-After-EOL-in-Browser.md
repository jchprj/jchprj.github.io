---
layout: post
title: Play Adobe Flash After EOL in Browser
date: 2021-03-07 12:27
category: old
author: Jue Chen
tags: ["Docker", "Flash"]
summary: Dockerised old Chrome, old Firefox and latest standalone Flash player accessible in browser.
comment_issue_id: 3
---

Finally Adobe Flash came to the end of life. I used it before work, and used as major work in early years, although pivoted later. I hope some old Flash contents are still playable not impacted by evolving software of OS. Docker is an ideal choice.

The base image is the powerful [linuxserver/docker-baseimage-guacgui](https://github.com/linuxserver/docker-baseimage-guacgui) which targets a Ubuntu desktop in Docker accessible in browser and interested me to create a container for old Flash. Without the ability to access an inner browser or other GUI application directly in browser, it is not so attractive to do these work.

This is the result:  
![Screenshot](/assets/images/play-adobe-flash-after-eolscreenshot.png)  
Firefox running in Docker accessible in Chrome

![Screenshot](/assets/images/play-adobe-flash-after-eolscreenshot2.png)  
Standalone Flash player running in Docker accessible in Chrome

To get this running, just run(Docker should be installed, [Get Docker | Docker Documentation](https://docs.docker.com/get-docker/)):
```
docker run -p 8080:8080 --name play-adobe-flash-after-eol jchprj/play-adobe-flash-after-eol
```

Then open [http://localhost:8080](http://localhost:8080) in your browser.

Check Docker Hub [jchprj/play-adobe-flash-after-eol Tags](https://hub.docker.com/r/jchprj/play-adobe-flash-after-eol) or GitHub [jchprj/Play-Adobe-Flash-After-EOL](https://github.com/jchprj/Play-Adobe-Flash-After-EOL) for more details include customizable settings.

Below is a longer story on more developing details.

## Desktop Linux in Docker

I didn't find any ready to use ways, but did investigations and tests and got it works successfully. I happened to know a way that could run a desktop Linux inside a Docker container and visit the desktop not only through RDP or VNC, but directly from browser without any client software other than browser. 

The Linux OS could be Alpine or Ubuntu or others. There is one such Docker image based on Alpine ready to use, but Alpine does not support Flash, so I have to build one from Ubuntu myself.

Here is how I find the final working container:

1. I found [danielguerra69/firefox-rdp: docker firefox + flash with rdp + audio](https://github.com/danielguerra69/firefox-rdp) first, but too old, cannot enter the desktop remotely.
2. But from the Dockerfile of `firefox-rdp` repo, got `FROM danielguerra/dockergui` which is [danielguerra69/dockergui](https://github.com/danielguerra69/dockergui). This seems a base image.
3. Although Dockerfile in the `dockergui` repo is `FROM phusion/baseimage:0.9.16`, but I noticed the whole repo is forked from [linuxserver/docker-baseimage-guacgui](https://github.com/linuxserver/docker-baseimage-guacgui). This is the very powerful container.
4. There is no same name Docker image in Docker Hub, I got answer from this thread: [can't download the linuxserver/docker-baseimage-guacgui image · Issue #28 · linuxserver/docker-baseimage-guacgui](https://github.com/linuxserver/docker-baseimage-guacgui/issues/28)  
   > The image you should be pulling is: lsiobase/guacgui


Next I tried some wrong ways to run desktop and went into some problems, after checking this issue [Errors when building a docker container on this base - Unable to find an X Display · Issue #15 · linuxserver/docker-baseimage-guacgui](https://github.com/linuxserver/docker-baseimage-guacgui/issues/15), the easiest part came: just follow the [example](https://github.com/linuxserver/docker-baseimage-guacgui/tree/master/example) in the repo.

The example is very straight forward, very simple Dockerfile, just need to notice how it runs the GUI app: it is started in a service configuration file. From there, no matter Chrome, Firefox or standalone Flash player all could be started and accessed remotely from just a browser.

OK, desktop Linux in Docker is resolved. 



## Play Flash

There are several options to run old Flash files, like run in an old version Chrome or Firefox, or run in the standalone Flash player. All work in the Docker container. Latest Chrome or Firefox could not run Flash, means when the system date is in 2021 and later, Flash player will stop work, this is coded internally. I don't want to find the code and recompile the browser, because I want a container so it could be packaged independently. The solution is find the latest version that works after the EOL date.


### Old Chrome

If use Chrome in container, need to add option `--no-sandbox` in CLI according to [linux - google-chrome Failed to move to new namespace - Stack Overflow](https://stackoverflow.com/questions/59087200/google-chrome-failed-to-move-to-new-namespace), or will get below error:
```
Failed to move to new namespace: PID namespaces supported, Network namespace supported, but failed: errno = Operation not permitted Failed to generate minidump.Illegal instruction
```

Chrome does not offer older version officially according to [browser - How can I download an old version of Google Chrome - Super User](https://superuser.com/questions/1381356/how-can-i-download-an-old-version-of-google-chrome). From the answer we can use this place for downloading old Chrome: [Download older versions of Google Chrome for Windows, Linux and Mac](https://www.slimjet.com/chrome/google-chrome-old-version.php). The latest available version did not set out of date flag for Flash Player is 53.0.2785.116(with Flash Player version 23.0.0.162) which is used in this Dockerfile. Direct download link: [link](https://www.slimjetbrowser.com/chrome/lnx/chrome64_53.0.2785.116.deb).  

One disadvantage compared to the Firefox is running some 3D Flash contents will crash in this Chrome Flash player. No such issue in Firefox or standalone player.



### Old Firefox

Old Firefox is available officially, got it here: https://ftp.mozilla.org/pub/firefox/releases/. Firefox 64-bit does not have such early time limit as Chrome, even version 84.0 allow play Flash, but 53.0.3 is the latest version I tested workable for some 3D contents(later version will show black). Direct download link for the version 53.0.3 used in this Dockerfile: [link](https://ftp.mozilla.org/pub/firefox/releases/53.0.3/linux-x86_64/en-US/firefox-53.0.3.tar.bz2).

If use Firefox, need some Flash player libraries, as it's old 64-bits Firefox, should put the `.so` file in the folder `/usr/lib/mozilla/plugins` according to - [Where is Firefox's plugins directory? - Ask Ubuntu](https://askubuntu.com/questions/383960/where-is-firefoxs-plugins-directory)

There is no offical support for the Flash player library for Firefox, even no official download available. I found the archive from here: [flashplayerarchive directory listing](https://archive.org/download/flashplayerarchive/pub/flashplayer/installers/archive/). Have to use the second latest version(32.0.0.371) to be run in Firefox, latest will be out of date. Direct link to the Linux used Flash Player inside the big zip file: [link](https://archive.org/download/flashplayerarchive/pub/flashplayer/installers/archive/fp_32.0.0.371_archive.zip/32_0_r0_371_debug%2Fflashplayer32_0r0_371_linux_debug.x86_64.tar.gz)

There is a additional article that is not used here, just mentioned how to make the library working for Firefox portable in case instersted: [Portable Adobe Flash Player \| Angry Sheep Blog](https://rejzor.wordpress.com/portable-adobe-flash/)


### Standalone Flash player

Unlike old browsers, it's still workable for the latest standalone Flash player - 32.0.0.465. Just download directly from Adobe([Official download page](https://www.adobe.com/support/flashplayer/debug_downloads.html)). But if play local content and needs to load external resources in the local drive, the security policy will block it, and seems no configuration files available in Linux to update the policy. 

So my solution is, run a local web server inside the container and load the contents in `http://localhost` as standalone player also support load a URL, but should be `.swf` file only. If you need to load HTML, use the browser ways.

When running the container, any folder mounted to `/flash` will be the root of the localhost web server.

## Other information
### Some other workarounds

Also there are some other ways, but again, I want most is a independent container, so I did not try them, just to mention here.

- [Enable Adobe Flash on Chrome after End of Life - The Tech Journal](https://www.stephenwagner.com/2021/01/13/enable-adobe-flash-chrome-after-end-of-life/)  
This way is using Chome version in 2020 and change some configuration files locally.

- [Flash Player in Chrome is Dead in 2020: How to Play Flash Files](https://www.online-tech-tips.com/computer-tips/flash-player-in-chrome-is-dead-in-2020-how-to-play-flash-files/)  
An online emulator Ruffle, but it's hard to play 3D contents and other complex projects.

- [How to play Flash content in your browser in 2021 - gHacks Tech News](https://www.ghacks.net/2020/12/17/how-to-play-flash-content-in-your-browser-in-2021/)  
Ruffle again, an extension for Firefox.

- [Running Adobe Flash in old browser after 2020 - Google Chrome Community](https://support.google.com/chrome/thread/11921467?hl=en)  
Some discussions, could see there are still people need to keep Flash running, but the thread was locked.


### Some contents about EOL of Flash

- [Adobe Flash Player End of Life](https://www.adobe.com/products/flashplayer/end-of-life.html)  
Adobe's official EOL annoucement

- [Google Chrome 88 released with no Flash support, bringing an end to an era | ZDNet](https://www.zdnet.com/article/google-chrome-88-released-with-no-flash-bringing-an-end-to-an-era/)  
Google has released Chrome 88 today, permanently removing support for Adobe Flash Player and bringing an end to an internet era.

- [Get ready to finally say goodbye to Flash — in 2020 – TechCrunch](https://techcrunch.com/2017/07/25/get-ready-to-say-goodbye-to-flash-in-2020/)  
During the 20+ years it has been around, it has played a key role in advancing interactivity and creative content on the web. Few technologies have had such a profound and positive impact in the internet era.

- [End of support for Adobe Flash | Firefox Help](https://support.mozilla.org/en-US/kb/end-support-adobe-flash#:~:text=Firefox%20will%20end%20support%20for,final%20version%20to%20support%20Flash)  
Even if you install an older version of Firefox, the Flash plugin itself stopped loading Flash content after January 12, 2021.

- [Solved: Does build in Adobe Flash Player in old chrome ver... - Adobe Support Community - 11617931](https://community.adobe.com/t5/flash-player/does-build-in-adobe-flash-player-in-old-chrome-version-still-working-after-2020/td-p/11617931)  (in fact old version in 2016 works)  
all versions of Flash Player will be blocked by Chrome. See the Enterprise Release notes:
''Important: Adobe will no longer update and distribute Flash Player after December 31, 2020. Therefore, after this date, all versions of Chrome will stop supporting Flash content. Pinning to or keeping an earlier version of Chrome through any other mechanism, will not prevent this change.''


### Some online pages still containing `.swf` file for test
- http://ultrasounds.com/
- http://web.csulb.edu/~ttravis/SURF/flashTest.htm

But they might be unavailable at any time. 

If you want to compile `.swf` from source, another Docker image([jchprj/docker-flex-4.6-sdk-ant](https://hub.docker.com/r/jchprj/docker-flex-4.6-sdk-ant)) I created years ago could be used, for example:
```
docker run -v ${PWD}:/flash -it --rm jchprj/docker-flex-4.6-sdk-ant mxmlc /flash/HelloWorld.as
```

## Lastly

Further question is: if no old Chrome or old Firefox or any other software could be found or you don't trust the non official package, is it possible to compile an old version from source in Git history as they are open source? Maybe we need to prepare a environment to compile various open source software as common sense first. That could also benifit compiling the latest version.