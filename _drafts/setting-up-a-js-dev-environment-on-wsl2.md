---
layout: post
title: Setting up a web dev environment on WSL2
description: Notes on how to get a Linux web development environment up and running
  on Windows with WSL2
category:
- Howtos
tags:
- howto
- dev
- js
- windows
- linux
- wsl
image:
  src: "/assets/img/posts/headers/fotis-fotopoulos-DuHKoV44prg-unsplash.jpg"
---
Over the years, I've setup, improved and tweaked my web development[^web-dev-fn] environment. If I were asked me what to install, how to configure it and so on and so forth, to get the same setup, I'd have to think about it for a while, and the answer would maybe overwhelm the questioner.

Interestingly this is exactly what a buddy of mine, let's call him Jim to protect his identity, asked me recently. Jim is a long time Windows C++ developer who spent the bulk of his career working for large (and not so large) firms. He's never really used Linux, is not really comfortable with the command line and has no idea how git works. So you can imagine that when Jim asked me to help him get up and running, I was happy to have the chance to share my knowledge with him, and I by the same token decided to take notes for myself and ultimately share this information with the few readers (if any) who might stumble on this page. There is nothing new, original or transformational in here. For many of the steps, it's going to refer simply and directly to the sources I pointed Jim to.

We'll start with the very basics of installing Linux on Windows and go up from there.

## The Windows side of things

### Linux (Ubuntu)

![Ubuntu](/assets/img/posts/2022/02/02/ubuntu-4-logo-png-transparent-192.png){: .right} Most steps in this "how to" are important, but clearly, if you want to develop on Linux, installing Linux is unavoidable! I've opted to use Ubuntu. It's one of the most used distros. I know there are many others available on Windows, but it's the one I use myself and I'm plenty satisfied with it. I also think that the popularity of Ubuntu makes it a particularly interesting choice as if I'm to hit a speed bump, I'm quite certain that I will find help somewhere on the web.

The [steps given on the Ubuntu site](https://ubuntu.com/tutorials/ubuntu-on-windows#4-install-ubuntu-for-windows-10) are what we followed.

We enabled WSL2 and installed Ubuntu 20.04 LTS from the Microsoft store.

### Chocolatey & Windows command line utilities

<!-- Elevated prompt -->

### Git for Windows

### Windows Terminal

### VS Code

### Fork

## The Linux side of things

### Get up-to-date

<!-- apt -->

### SSH

[^web-dev-fn]: I'm not really a web developer. I don't even play one on TV and I barely dabble in such development my spare time. But I still have a functional, working environment. Go figure. :sweat_smile:
