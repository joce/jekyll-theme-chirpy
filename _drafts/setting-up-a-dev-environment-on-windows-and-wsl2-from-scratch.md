---
layout: post
title: Setting up a dev environment on Windows and WSL2 from scratch
description: Notes on how to get a development environment up and running
  on Windows with WSL2, assuming pristine machine and no prior knowledge.
category: How-to Guide
tags:
- how-to
- dev
- windows
- linux
- wsl
image:
  src: "/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/header/fotis-fotopoulos-DuHKoV44prg-unsplash.jpg"
---

Over the years, I've setup, improved and tweaked my dev environment. If I were asked me what to install, how to configure it and so on and so forth, to get the same setup, I'd have to think about it for a while, and the answer would maybe overwhelm the questioner.

Interestingly this is exactly what a buddy of mine, let's call him Jim to protect his identity, asked me recently. Jim is a long time Windows C++ developer who spent the bulk of his career working for large (and not so large) firms. He's never really used Linux, is not really comfortable with the command line and has no idea how git works. So you can imagine that when Jim asked me to help him get up and running, I was happy to have the chance to share my knowledge with him, and I by the same token decided to take notes for myself and ultimately share this information with the few readers (if any) who might stumble on this page. There is nothing new, original or transformational in here. For many of the steps, it's going to refer simply and directly to the sources I pointed Jim to.

We'll start from scratch, installing all the basics on Windows and go up from there.

## The Windows side of things

First off we'll install all that we need on the Windows side of things. We'll eventually get to installing Windows Terminal (which I like dearly!) in a bit, but before we do, I'd like to install all the components I used that can be launched from it.

### Powershell

Before we do anything else, just make sure you have the latest version of [Powershell](https://docs.microsoft.com/en-us/powershell/). You can get it on the [Microsoft store](https://www.microsoft.com/en-us/p/powershell/9mz1snwt0n5d). As of this February 2022, the most recent version is 7.2.1

### OpenSSH

Another easy thing to turn on is OpenSHH. It used to be that on needed to install a 3rd party software to have access to SSH on Windows, but in recent versions, it's built right in. It just needs to be turned on. We'll go about configuring SSH and the creation of key pairs a bit later.

Go to the **Optional Features** in the Windows settings:

- Open **Settings**
- Click on **Apps**
- Click on **Optional features**

![Optional Features](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/app-features-optional.png)

- Click on **Add a feature**
- Search for `OpenSSH`, select it and press **Install**

![Install OpenSSH](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/add-openssh.png)

That's it! You're done.

### Chocolatey & Windows command line utilities [![Chocolatey](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/choco-logo-square.svg){: .right w="128"}](https://chocolatey.org)

Next, we're going to install a package manager for Windows. I have some colleagues who swear by [Scoop](https://scoop.sh/) (I have to admit it appears nice), while others can't stop extolling the virtues of [winget](https://docs.microsoft.com/en-us/windows/package-manager/winget/). I must admit that I'm intrigued by Scoop, but the reality is that I've been using [Chocolatey](https://chocolatey.org/) for the longest time, and I've not yet encountered any issue sufficiently hindering (or actually, any issue at all) that made me reconsider and change.

You will find the steps to [install Chocolatey on their site](https://chocolatey.org/install).

The tl;dr is to run the following command from an elevated PowerShell prompt:

```console
$ Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Once you've installed Chocolatey, we'll install just install a single program for now, [`gsudo`](https://github.com/gerardog/gsudo), which will be required for the [setup of the Windows Terminal](#windows-terminal) we're about to do.

Again, from an elevated console (doesn't have to be Powershell this time.)

```console
$ choco install -y gsudo
```

If you'd like to not have to pass in the `-y` param every time you install something (or actively accepting when prompted), you can turn on the auto confirm flag:

```console
$ choco feature enable -n allowGlobalConfirmation
```

### Git for Windows [![Git for Windows](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-bash.svg){: .right w="128"}](https://gitforwindows.org/)

Head over to to the [git for Windows site](https://gitforwindows.org/) and press the download button. This will download the installer. Once downloaded, I would suggest running it as administrator so it's installed globally. You can do this by right clicking on the executable, and select "Run as administrator".

![Run as administrator](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/run-git-install-as-admin.png)

The installer will then make you go through the Wizard.

First, it will ask you to agree on the license and to select the install path (just choose the default one).

Next you'll have to decide what components to install. I personally opted out of everything but the large file support; I don't like to have my desktop polluted with icons, I don't want the various filetype associations, I don't wish to use the bash that comes with git (we're going to be using the WSL2 one) and we're going to install a better GUI tool than git GUI.

![git install wizard - Components](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/01-components.png)

For the default text editor associated with git, let's just stick with the good ol' Windows notepad for now.

![git install wizard - Editor](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/02-editor.png)

Let's stick with default initial branch name for new repos

![git install wizard - Initial branch](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/03-initial-branch.png)

Same thing for the PATH environment variable; stick to default

![git install wizard - PATH environment](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/04-PATH-environment.png)

As we're [already installed OpenSSH on Windows](#ssh), we won't be installing any new SSH for git.

![git install wizard - SSH](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/05-SSH.png)

For the next options, HTTPS transport backend, I have no idea what it entails. Let's just keep the default.

![git install wizard - HTTPS](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/06-HTTPS.png)

I'm choosing not to let git mess around with line endings. I think this is a setting for another era. Nowadays, pretty much all programming text editors handle EOLs just fine.

![git install wizard - Line endings](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/07-line-endings.png)

We're not going to be using git bash anyway. Let's not install anything we'll not be using.

![git install wizard - Git bash](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/08-git-bash.png)

We're going to be using the default behavior for `git pull`.

![git install wizard - Git pull](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/09-git-pull.png)

I have no idea what the credential manager is. I don't remember using this. Let's keep the default here as well.

![git install wizard - Credential manager](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/10-credential-manager.png)

For the extra options, let's turn on the file system caching on. For the symbolic links, I've had to think about it a bit, and I decided against it on Windows, since last I checked, sym links are not well supported. I could be wrong on that, however.

![git install wizard - Extras](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/11-extras.png)

Finally, I have no use for any experimental features at the moment. Let's not enable any.

![git install wizard - Experimental](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/git-install-screenshots/12-experimental.png)

That's it! Git is now installed.

### VS Code

[![VS Code](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/Visual_Studio_Code_1.35_icon.svg){: .right w="128"}](https://code.visualstudio.com/)

Next up in the install list is Visual Studio Code. This is a source code editor, verging to an IDE. It doesn't come right out of the box with all the builds and debugging abilities one can expect from an actual IDE, but with its extensive set of extensions, it sure can feel like one sometimes. It also starts blazingly fast! There was a rather interesting talk on that topic a few years back: [VS Code: The First Second](https://www.youtube.com/watch?v=XgDus4lPmR8). It's worth checking if you have a interest in performance.

Installing VS Code is as easy as navigating to its [web page](https://code.visualstudio.com/docs/?dv=win), downloading the installer and running it. That's it.

One excessively nice bonus is its capabilities to open any folder or file in WSL from your Windows box, using the [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) extension. We'll get back this this later, when we're all setup on the Linux side.

### Fork

Another thing I like to have installed on the Windows side of things is [Fork](https://git-fork.com/), an excellent GUI for git. I think it's important to be able to know how to do most of the work that one needs to do with git on the command line, but there are definitely some tasks that can benefit from a good user interface (e.g. some more complex rebases are much more pleasant to do using Fork).

The tool can be used for free, but the developers ask that you buy a license, which I did. Their tool is very well done, very useful, and the $50 I spent for it is worth every penny.

Also non negligible is the fact that you can invoke and use fork for repos that live on the WSL side of things (more info on that once we get to the Linux part of the setup).

To install Fork, simply head over to their web site, download the installer and run it.

### Linux (Ubuntu) [![Ubuntu](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/UbuntuCoF.svg){: .right w="128"}](https://ubuntu.com)

Most steps in this "how to" are important, but clearly, if you want to develop on Linux, installing Linux is unavoidable! I've opted to use [Ubuntu](https://ubuntu.com). It's one of the most used distros. I know there are many others available on Windows, but it's the one I use myself and I'm plenty satisfied with it. I also think that the popularity of Ubuntu makes it a particularly interesting choice as if I'm to hit a speed bump, I'm quite certain that I will find help somewhere on the web.

> As I went through this with Jim, In the course of the installation, we'll need to run commands from an elevated Powershell prompt. If you don't know how to open one, there are thousands of site explaining how out there. The one on [Bleeping Computer](https://www.bleepingcomputer.com/tutorials/how-to-open-an-elevated-powershell-admin-prompt-in-windows-10/) explains it well enough.
{: .prompt-warning }

The [steps given on the Ubuntu site](https://ubuntu.com/tutorials/ubuntu-on-windows) are what we followed.

Basically:

1. enable WSL2 using this command line from an elevated powershell:

    ```console
    $ dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    ```

2. install [Ubuntu 20.04 LTS from the Microsoft store](https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6).

### Windows Terminal [![Windows Terminal](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/Windows_Terminal_logo.svg){: .right w="128"}](https://github.com/Microsoft/Terminal)

For quite a number of years, the official offering from Microsoft in terms of terminal was simply abysmal. As a matter of fact, the default terminal that is still the default one to this day is just pathetic. To that effect, I used [ConEmu](https://conemu.github.io/en/TableOfContents.html) as a replacement. However Microsoft came released in 2019 the new, open source, quite versatile and very customizable [Windows Terminal](https://docs.microsoft.com/en-us/windows/terminal/). It's still quite actively being developed and new features are added continuously.

You can install it from the [Microsoft Store](https://www.microsoft.com/en-ca/p/windows-terminal/9n0dx20hk701)

Once installed, we can configure it so we can access the regular Windows prompt, Powershell (both of them as regular user AND admin), as well as a Linux shell.

To do so, open Windows Terminal, from the little downward arrow, select `Settings`, and from the settings page, click on the cogwheel on the lower left of the window. This should open the `settings.json` file.

To have access to the prompts, replace the `profiles.list` section with the following:

```jsonc
{
  // [...]

  "profiles": {

    // [...]
    "list": [
      {
        "commandline": "cmd.exe",
        "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
        "icon": "ms-appx:///Images/LockScreenLogo.png",
        "name": "Command Prompt",
        "tabTitle": "cmd"
      },
      {
        "commandline": "gsudo.exe -d cmd.exe",
        "guid": "{97364cae-241d-4ead-b255-f092da336b8f}",
        "icon": "ms-appx:///Images/LockScreenLogo.png",
        "name": "Command Prompt (Admin)",
        "tabTitle": "cmd"
      },
      {
        "commandline": "pwsh.exe",
        "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
        "icon": "ms-appx:///ProfileIcons/{574e775e-4f2a-5b96-ac1e-a2962a402336}.png",
        "name": "PowerShell",
        "tabTitle": "pwsh"
      },
      {
        "commandline": "gsudo.exe -d pwsh.exe",
        "guid": "{f3ccca13-d497-4a22-bac7-7fe44afada86}",
        "icon": "ms-appx:///ProfileIcons/{574e775e-4f2a-5b96-ac1e-a2962a402336}.png",
        "name": "PowerShell (Admin)",
        "tabTitle": "pwsh"
      },
      {
        "commandline": "wsl ~",
        "guid": "{2c4de342-38b7-51cf-b940-2309a097f518}",
        "icon": "<YOUR_PATH_TO>/ubuntu_icon.png",
        // ^^=== <YOUR_PATH_TO> with the path where you save your icon
        "name": "Ubuntu",
        "source": "Windows.Terminal.Wsl"
      }
    ]
  }

  // [...]
}
```
{: file='settings.json'}

![Ubuntu Icon](/assets/img/posts/setting-up-a-dev-environment-on-windows-and-wsl2-from-scratch/ubuntu_icon.png){: .left}
Using `gsudo` we installed earlier, we can start elevated prompt from within Windows Terminal. Pretty much everything should work out of the box, with the sole exception of the Ubuntu icon. I've found one that I like that I'm sharing beside here, but any 32x32 image would to.

You will notice that I'm referencing `pwsh.exe` in the config. That's the executable for PowerShell. It appears that Jim, even though he has _a_ version of PowerShell installed, it was an older one, and not one names `pwsh.exe`. If you're having problems opening a PowerShell from Windows Terminal, consider installing a recent version of [PowerShell](#powershell) or if you don't really have any use for it, you can also remove the sections from the config file.

## The Linux side of things

### Get up-to-date

<!-- apt -->

### SSH

[If you get stuck, forgot your pw...](https://medium.com/@TheLittleNaruto/reset-wsl-password-609037c2d6c6)

<!-- Get the SSH config back to Windows -->

<!-- Setting up multiple accounts -->

### Niceties

#### .bashrc && .bash_profile

#### Aliases

#### git aliases

#### Git Posh
