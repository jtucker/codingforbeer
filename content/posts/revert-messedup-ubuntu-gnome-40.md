---
title: "Revert Messed up Ubuntu install of Gnome 40"
date: 2021-04-01T11:00:39-04:00
tags: ['ubuntu', 'linux', 'gnome']
draft: false
---

I tend to run the latest and greatest dev version of Ubuntu on my laptop (Surface Book 2 currently) and wanted to check out the [new Gnome 40](https://help.gnome.org/misc/release-notes/40.0/) release because it looks great and I really want some of those sweet workspace improvements.

So I checked out the first [PPA](https://launchpad.net/~shemgp/+archive/ubuntu/gnome-40) I could find via [Reddit](https://reddit.com/r/ubuntu) and fired it up. 

And then I couldn't boot into the system. It looked very similar to this: 
![Oh no! You messed something up buddy](https://i.stack.imgur.com/Q6kBe.png)
_credits: Khayordey Phemmy via [StackOverflow](https://askubuntu.com/questions/1187562/cant-use-my-desktop-on-ubuntu-19-10-after-upgrading-from-18-04)_

As mild panic set in, I figured there has to be a way to rollback these changes and sure enough, GooBingDuck led me to pretty simple solution.

1. Boot like you normally would but select your Recovery kernel. Mine was in the Advanced menu item in Grub
1. Enable Networking in the menu. I needed to be hardwired in to get a network connection which I was lucky to have. I'm not sure how you would do it via Wi-Fi
1. Enter a root shell
1. Install [ppa-purge](https://launchpad.net/ppa-purge) 
   ```
   apt-get install ppa-purge
   ```
1. Remove the PPA 
   ```
   ppa-purge ppa:shemgp/gnome-40
   ```
1. Reboot

Once the reboot finished, I was back to where I was pre-Gnome 40 and back to waiting patiently. 