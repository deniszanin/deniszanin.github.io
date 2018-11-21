Bad, bad GRUB: fixing Qubes boot option missing
===============================================

One of the major problems that I've been facing when working with different Linux distributions on the same machine has been and still is, **GRUB**! Sometimes it help us a *ton* handling with *boot* managing, but sometimes... times like these...!! Specially working over Ubuntu on the same machine and its infinite system updates.

The problem that I've been struggling with is on each system update on Ubuntu, **Grub** needs to update its kernel variables on its *config* file, so, it must rewrite everything back to place. What really happens is that *Grub* also rewrites a particular **EFI** partition which holds Qubes *boot* files. After that, things get messy and Qubes panics starting to do a dance called *loop-and-reboot*.

So, how to solve these issues? And how do we fix Qubes boot without reinstalling it? Over this post, I'll show you how to recover Qubes system due to Ubuntu-Grub issues and any emergency caused by boot failure.

Creating an USB Rescue drive
----------------------------

First step is essential, we want to create a *bootable* USB drive to install (or to recover in our case) a Qubes system. I always carry with me an emergency USB Recovery drive when I travel, or leave it at work. Having a USB drive with a recovery option is a huge time saver and we never know when will need it. Trust me, we don't know... 

To create your USB, follow instructions detailed on [Qubes documentation page](https://www.qubes-os.org/doc/installation-guide/). Briefly, steps are: to download *Qubes ISO*, to verify its authenticity and signature, and burn it on USB drive or a DVD disc. For me, USB are quicker and laptops nowadays doesn't have DVD drives anymore, like mine. So, grab an USB and save it for future.

BIOS options
------------

In some cases, Qubes won't boot on USB, because may be inconsistencies between system and **BIOS**. So, change any *BIOS* options that are required to boot your system (see [Hardware Compatibility List](https://www.qubes-os.org/hcl/)). Being owner of a *DELL* laptop doesn't help me a lot, so I had to tweak a few options to install for the first time, and tweak different options to make *Rescue* possible (detailed tweaks done by me are available in [Qubes Documentation on Hardware Compatibility List](https://groups.google.com/forum/#!msg/qubes-users/YdN8Ks2K3Qg/BRBXUdrbCAAJ)). Due to not being able to boot recovery system on **UEFI**, for example. For me, it's only possible using *Legacy Mode*.

Fixing "Qubes *boot* option missing"
----------------------------------

Choose to boot Qubes in **Legacy Mode**, and select to boot it from your USB drive that contains Qubes system. It will show a screen for you to choose between installing it or troubleshooting it. For our purposes, choose **Troubleshooting** and **Rescue a Qubes system option**.

Qubes will boot in **Rescue Mode** now. And if everything load perfectly, it will show you this rescue first message.

```
==============================================
==============================================
Rescue

The rescue environment will now attempt to find your Linux 
installation and mount it under the diretory: /mnt/sysimage. 
You can then make any changes required to your system. 
Choose '1' to proceed with this step. You can choose to mount 
your file system read-only instead of read-write by 
choosing '2'. If for some reason this process does not work 
choose '3' to skip directly to a shell.

1) Continue
2) Read-only mount
3) Skip to shell
4) Quit (Reboot)

Please make a selection from the above:
```

Choose **Continue**, option **1**, and hit *Enter*.

So, next prompt: **Qubes Recovery system** will try to access each partition available on your hard drive. If there are encrypted volumes, besides Qubes itself, then it will ask you corresponding passphrases.

```
==============================================
==============================================
Password

You must enter your LUKS passphrases to decrypt device X

Passphrase:
```

For each encrypted *LUKS* volumes, type its passphrase.

And wait while *Rescue* system recovers and decrypts information about your devices, one by one. Advice: be patient! It'll take some minutes.

```
==============================================
==============================================
Root Selection

The following installations were discovered on your system.

[x] 1) Ubuntu Linux on /dev/...
[ ] 2) Qubes Linux on /dev/...

Please make your selection from the above list.
Press 'c' to continue after you have made your selection.
```

So, after it finishes and with every system discovered, select your Qubes system, typing the number which corresponds to the list. In my case, option **2**. Hit *Enter*. 

After typing it, it will print same list of systems, but with appropriated partition selected. Check it, and when you're ready, press **c** to continue.

```
==============================================
==============================================
Rescue mount

Your system has been mounted under /mnt/sysimage.

If you would like to make your system the root environment, 
run the command:

     chroot /mnt/sysimage

Your system is mounted under /mnt/sysimage directory.
Please press [Enter] to get a shell.
```

When you get a shell, enter to the **EFI** folder, typing:

```
$ cd /mnt/sysimage/boot/efi/EFI
```

Now, continue with these following commands:

```
$ cp -R qubes/. BOOT/
$ cp BOOT/xen.cfg BOOT/BOOTX64.cfg
$ cp qubes/xen-4.8.4.efi qubes/xen.efi
$ cp qubes/xen-4.8.4.efi BOOT/BOOTX64.efi
```

Everything is done now. 

You managed to replace missing, or *bad structured*, boot file into its proper location. Reboot your system (typing `reboot`) and back to your BIOS, to revert any changes, if you had. 

Rebooting, everything must be in place, and Qubes will boot normally now.

To solve those issues for good, a good idea is to set manually each boot option on BIOS and *purge* GRUB from others operational systems, like Ubuntu here.

Bugs?
-----

Did you find any bugs? Corrections? Feel free to comment. :-)

