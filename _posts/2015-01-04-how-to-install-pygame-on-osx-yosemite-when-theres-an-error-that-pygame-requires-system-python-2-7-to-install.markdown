---
author: nhuhu
comments: true
date: 2015-01-04 05:51:12+00:00
layout: post
link: https://artificialcognition.wordpress.com/2015/01/04/how-to-install-pygame-on-osx-yosemite-when-theres-an-error-that-pygame-requires-system-python-2-7-to-install/
slug: how-to-install-pygame-on-osx-yosemite-when-theres-an-error-that-pygame-requires-system-python-2-7-to-install
title: How to Install Pygame on OSX Yosemite (when there's an error that pygame requires
  System Python 2.7 to install)
wordpress_id: 49
---

When I was trying to install Pygame today, I tried to use the installer but out popped this error:

![The ERROR...](https://artificialcognition.files.wordpress.com/2015/01/screen-region-2015-01-04-at-13-57-41.png?w=300)

<blockquote>pygame 1.9.1 release can't be installed on this disk.
pygame requires System Python 2.7 to install.</blockquote>


This was frustrating so I used my google foo and found[ a stackoverflow page](http://stackoverflow.com/questions/21806723/pygame-installation-fails-due-to-requirement-of-system-python-2-7-even-though-i), exactly my problem. Now, I have to admit, it took me pathetically about half an hour to figure it out, even though the solution was ON that page, albeit somewhat obscured (at least to me), I just didn't realise it.

I tried using homebrew, which for some reason needed to be reinstalled and then tried to do various things based on other links I found.

When that all failed, I looked back and found that once I installed XQuartz and used the MacOS 10.7 build, it worked. Here's what you only have to do:

**Step 1: Make sure you have Homebrew and install XQuartz in Terminal**

```
brew install Caskroom/cask/xquartz
```

**Step 2: Install Pygame with [this](http://pygame.org/ftp/pygame-1.9.1release.tar.gz).**
