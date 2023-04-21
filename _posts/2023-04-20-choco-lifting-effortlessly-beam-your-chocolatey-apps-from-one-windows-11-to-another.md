---
layout: post
title: 'Choco-Lifting: Effortlessly Beam Your Chocolatey Apps from One Windows installation to Another'
date: 2023-04-20 21:33 -0400
description: How to quickly and easily replicate (part of) your Windows setup on a new machine.
image:
  path: /assets/img/posts/choco-lifting-effortlessly-beam-your-chocolatey-apps-from-one-windows-11-to-another/header/a_little_box_of_chocolateson_an_antique_table.png
  title: "MidJourney:\nA little box of chocolateson an antique table in the window of a Belgian chololatier's shop. Red tablecloth. Fujicolor superia x-tra. ISO 400. 20mm lens. f/5.6. 1/50. Amazing light. --v 5 --ar 5:2"
  alt: "A box of chocolate in the window of a chocolatier"
category: How-to Guide
tags:
- chatGPT
- how-to
- command line
- script
authors:
- joce
- chatgpt
---

I'm currently in the process of migrating to a new laptop. My setup has its own quirks that I like to port over and it's definitely a multi-day process. One thing that can be migrated quickly and effortlessly is all the programs that I've installed with Chocolatey.

I've asked ChatGPT to help me compose the body and the title of this text (though I've written this intro myself). I've edited what was generated slightly, but it's mostly AI written from hereon out.

Here you go... Just follow the guide!

### Prerequisites

1. Both Windows systems must be accessible.
2. Chocolatey is installed on both Windows setups.

### Step 1: Export the list of installed Chocolatey packages

1. On the existing Windows setup, open a cmd shell or PowerShell as administrator.
2. Execute the following command to export the list of installed Chocolatey packages:

    ```batch
    choco list --local-only --id-only -r | sort > choco-packages.txt
    ```

3. Locate the `choco-packages.txt` file in your user's home directory.

### Step 3: Transfer the 'choco-packages.txt' file to the new Windows setup

1. Use a USB drive, network transfer, or cloud storage to copy the `choco-packages.txt` file to the new Windows 11 setup.
2. Place the file in your desired directory on the new system.

### Step 3: Install the exported Chocolatey packages on the new Windows setup

1. On the new Windows setup, open PowerShell as administrator.
2. Navigate to the directory where you placed the `choco-packages.txt` file using the `cd` command, e.g.,

    ```batch
    cd C:\Users\YourUsername\Downloads\
    ```

3. Execute the following command to install the packages listed in the `choco-packages.txt` file:

    ```powershell
    Get-Content choco-packages.txt | ForEach-Object { choco install $_ -y }
    ```

### Step 4 (Facultative): Automated verification of migrated apps

1. On the new Windows setup, export the list of new installed packages, sorted alphabetically, to a file, just as you did in [Step 1](#step-1-export-the-list-of-installed-chocolatey-packages):

    ```batch
    choco list --local-only --id-only -r | sort > choco-packages-new.txt
    ```

2. Compare the two sorted package lists by executing the following command in PowerShell:

    ```powershell
    Compare-Object (Get-Content choco-packages.txt) (Get-Content choco-packages-new.txt)
    ```

3. If the output is empty, the package lists match, and the migration was successful. If there are any differences, the output will display packages with a `=>` symbol, indicating they are present only on the new system, or a `<=` symbol, indicating they are present only on the existing system. In my case, I noticed that a couple packages had not installed for some reason and I then proceeded to install them by hand.

And there you have it! Migrating Chocolatey apps between Windows systems is now seamless and efficient for me and it can also be for you. This method simplifies setup and ensures consistency across devices, allowing you to focus on making the most of your Windows 11 experience, confident that your essential tools are in place. The only thing remaining is to properly port all the configuration of all these tools (and no, they're not all nice dot files living in my home directory. Sadly.)
