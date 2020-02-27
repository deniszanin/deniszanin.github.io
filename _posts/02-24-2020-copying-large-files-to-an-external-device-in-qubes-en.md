Copying large files to an external device in Qubes 4
----------------------------------------------------

If you are used to work with several different OSes *(Operational Systems)*, it's pretty straightforward to copy a large file to an *external device*. You plug it to the computer and copy/move a file to the device, right? **Qubes OS** has, as we know, a structure, with *"many security implications"*, that does not allow devices to be attach to the **dom0**, *the special domain*. Quoting *"external device", I mean*, it is, *or* a different partition (not native to *Qubes data structure*), *or* an external HDD, *or* an USB drive. In my case, it's a different partition, encrypted with **LUKS**, in a different internal HDD. In the Qubes' terminology we'd call a ***block device***, a *"fancy way to say 'something that stores data'"* [^1].

---

### Attention!

==Important notice: be advised not to plug or mount any external device on Qubes; it offers an opportunity, an attack surface, to any untrusted device over your system==

***Note about file systems:*** unfortunately, size limits saving exists; not many partitions accepts files over 4GB (i.e. FAT32, *"The file is too large for the destination file system"* error). Check your external device file system and use `split` command line tool to *split* your large file into several.

*Cool?* Ok...

---

On my work environment I have *several* files (including Qubes backup files) and I decided to transfer it to a safe external drive, which I encrypted a couple weeks ago. Unfortunately, working with large files and transferring it to an *AppVM* is not an easy task in Qubes; increasing constantly the size of an *AppVM* (around 100GB) can be a pain - sometimes with data corruption -, so I started to save weekly backups in ***dom0***.

A week ago I managed to create a *"proxy-ish" AppVM*, making it possible to copy large files (around 50GB) directly to an external device, without increasing *AppVM'* size and without saving the file on the *AppVM* temporary.

### 1. Setting it up

First, create an *AppVM*, that it will work like a bridge connecting ***dom0*** to your external device. On this example, this VM will be based on a ***Fedora 30*** template and with **no** Internet connection.

Run the following commands in ***dom0***:

```
qvm-create --template fedora-30 --label black proxy-transfer-vm

qvm-prefs proxy-transfer-vm netvm ""
```

After creating and setting its preferences, run this *AppVM* with `qvm-start proxy-transfer-vm` command (still typing in the **dom0** terminal), and then plug into your computer the external device (in case it's a external HDD).

To handle all your connected devices, the ***block devices***, **Qubes** offers a command line utility that lists all (those, not Qubes native) external devices. Run the following command to list it all and attach the correct one to your `proxy-transfer-vm` *AppVM*.

```
# note: 'qvm-block' command is a shortcut for
# 'qvm-device block list' command.
#
# for block devices, type:
qvm-block

# It will list all available devices in your system.
# And should return, something like:
#
# BACKEND:DEVID   DESCRIPTION     USED BY
# dom0:sda1       ()
# dom0:sdb2       ()
# sys-usb:sdc1    ()
# sys-usb:dev1    ()
```

`BACKEND` is the *qube hosting* (the ***source VM***) and `DEVID` is the *device ID*. In this example, let's consider using the `dom0:sda1` as our external device. To attach it to the *AppVM*, run the following command with the `BACKEND:DEVID` identification showed on the list. Examples:

```
# Syntax: qvm-block attach <APPVM> <BACKEND>:<DEVICEID>
qvm-block attach proxy-transfer-vm dom0:sda1
```

After running the last command, run again `qvm-block` command. Now, it will list the same available devices, but `USED BY` column will be filled by the *AppVM*, pointing which device is attached and where it is (`frontend-dev=xvdi`), meaning for `/dev/xvdi` (if that name is not already taken, in any case can be `/dev/xvdj`, `/dev/xvdk`, etc.) [^2].

Now, we move to type commands in the `proxy-transfer-vm` *AppVM*, which will be the bridge between ***dom0*** and the external device. Run this command to start a new terminal session in the *AppVM*.

```
qvm-run proxy-transfer-vm gnome-terminal
```

Type the following commands, in your *AppVM* terminal, to create the *default inter-qube* folder (when secure copying files between qubes - *domains, AppVMs* -, with the `qvm-copy` command, all files are copied to the same *destination* folder in a destination qube `/home/user/QubesIncoming` or `~/QubesIncoming` alias); and the mount point for the external devices which is already attached, but not mounted.

```
mkdir -pv ~/QubesIncoming
mkdir -pv ~/ExternalDeviceMount
```

### (Optional) Decrypt your external device

Inside your *AppVM*, as we named `proxy-transfer-vm`, we will decrypt the drive which will be holding your files received from ***dom0***. As detailed before, the location of the external device, which is already attached, is, for this example, `/dev/xvdi`. The `MAPPER_ALIAS` is a nickname to identify that specific device (locally).

```
# cryptsetup is a very common encryption tool.
# This command should be run as 'superuser'.
# 
# sudo cryptsetup luksOpen DEVICE MAPPER_ALIAS
#
# Run:
sudo cryptsetup luksOpen /dev/xvdi myDeviceAlias

# cryptsetup will check the device, and ask your 
# decryption key (passphrase).
#
# (LUKS passphrase will be asked. Type it)
```

### 2. Mouting your device

After attaching your device, and decrypting it (if it's your case), there is one last step before copying files: mount your attached device to the local AppVM data structure.

```
# IF YOU DECRYPTED YOUR DEVICE:
# To mount the attached device, after decrypting it, 
# mount the MAPPER (create with last command), to the
# mount point folder that you created before.
sudo mount /dev/mapper/myDeviceAlias /home/user/ExternalDeviceMount

# OR, IF NOT:
#
# To mount the attached device, indicate the device
# location (e.g., /dev/xvdi) to the mount 
# point folder that you created before.
sudo mount /dev/xvdi /home/user/ExternalDeviceMount
```

After mounting it, run `ls -al /home/user/ExternalDeviceMount` to list all files inside your external device. If it was mounted properly then you should be able to check the data stored in the device and also be able to create new folders and files. In other words, all changes to the `~/ExternalDeviceMount` directory will be reflected directly to your external device. Create a new folder to store files copied from ***dom0***.

```
mkdir -pv ~/ExternalDeviceMount/dom0copy
```

### 3. *Proxy-ish-ing-ish*

If any changes in the mount point folder, `ExternalDeviceMount`, will be reflected to the mounted device, what would happen if a file is copied ***directly*** to this folder (***without*** transferring to the *AppVM* first)? That's exactly the workaround we're making here...

To work properly, create a ***symbolic link*** from the *default inter-qube* folder to the mount point folder (where files will be copied directly):

```
ln -s /home/user/ExternalDeviceMount/dom0copy ~/QubesIncoming/dom0
```

As said before, by default, files copied from a ***sourceVM***, like *dom0*, are copied to the *default inter-qube* folder, `~/QubesIncoming/sourceVM`, inside the destination *AppVM* (`dom0 - file_a ---copy---> appvm - /home/user/QubesIncoming/dom0`). Creating the symbolic link is a workaround to point the *default inter-qube* folder to the folder in the external device. By doing this, bytes transferred securely are stored ***directly*** into this device.

### 4. Copying files

Back to ***dom0*** terminal, run the command to copy the file from ***dom0*** to the *AppVM*.

```
# qvm-copy-to-vm command only works when transferring 
# files from dom0 to any other AppVM. For an AppVM to
# AppVM transfer, use qvm-copy command.
#
qvm-copy-to-vm proxy-transfer-vm LARGE_FILE.EXT
```

And wait data to be transferred.

### 5. Most important: umount and detach your external device

After file transfer is completed, and before stop running your AppVM, or unplugging the device, *always* - and *I mean*, ***ALWAYS!*** -, umount and detach your external device, otherwise you could damage your storage.

In the *AppVM* terminal:

```
# Umount your mount point folder.
sudo umount /home/user/ExternalDeviceMount

# And, IF YOU DECRYPTED YOUR DEVICE:
sudo cryptsetup luksClose myDeviceAlias
```

In ***dom0*** terminal:

```
# Syntax: qvm-block detach <APPVM> <BACKEND>:<DEVICEID>
qvm-block detach proxy-transfer-vm dom0:sda1
```

And... it's done! ;-)

### Bugs?

Did you find any bugs? Corrections? Feel free to comment. :-)

#### Footnotes

[^1]: Qubes user documentation: Block (Storage) Devices, [external link](https://www.qubes-os.org/doc/block-devices/)

[^2]: Qubes user documentation: Block (Storage) Devices, Additional Attach Options, [external link](https://www.qubes-os.org/doc/block-devices/#additional-attach-options)


