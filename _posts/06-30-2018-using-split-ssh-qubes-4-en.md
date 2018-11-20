Using Split-SSH in Qubes 4
==========================

***Note:*** *This is an English version, translated from my last post, in Portuguese (available [here](https://deniszanin.com/configurando-split-ssh-gpg-qubes/)), with a few corrections to make it "readable". Sorry for any grammar mistakes.*

Introduction
------------

**Split SSH** is a concept applied to the operation system [Qubes](https://www.qubes-os.org/), similar to ***Split GPG***, introduced and described in the system documentation[^1]. This security model consists in safeguarding public and private keys from the environment where it gonna be used and, in case its compromissed, *GPG* (or *SSH*) keys will ***not*** be affected.

In **Qubes' documentation**, the following text explains the concept of *sharing across* keys:

> **Split GPG** implements a concept similar to having a smart card with your private GPG keys, except that the role of the “smart card” plays another Qubes ***AppVM*** . This way one, not-so-trusted domain, e.g. the one where Thunderbird is running, can delegate all crypto operations, such as encryption/decryption and signing to another, more trusted, network-isolated, domain. This way the compromise of your domain where Thunderbird or another client app is running  arguably a not-so-unthinkable scenario  does not allow the attacker to automatically also steal all your keys. (We should make a rather obvious comment here that the so-often-used passphrases on private keys are pretty meaningless because the attacker can easily set up a simple backdoor which would wait until the user enters the passphrase and steal the key then.) [^longnote]

Same idea applies to **Split SSH**: private and public keys are held and configured on a SSH *domain*, isolated and with **no** network access; a different *domain*, with Internet, will access *secret VM* (isolated with the keys). This technique was proposed by Jason Hennessey, [@henn](https://github.com/henn), on [qubes-app-split-ssh](https://github.com/henn/qubes-app-split-ssh) [^2]. This connection will be established by a *local socket* (created by ***ssh-agent*** on each *domain*) with ***Qubes framework*** handling it.

> This is done by using Qubes's qrexec framework to connect a local SSH Agent socket from an AppVM to the SSH Agent socket within the ssh-vault VM. [^longnote2]

Main differences between **Split GPG** and **Split SSH** starts on its implementation and how to configure it. *Split GPG* is a *native* feature - so to speak! -, easily configurable on Qubes, while *Split GPG* requires a few tweaks (and headache!) to work with.

Configuration
-------------

Setting system, domains and files up is a easy task, but it can get confused if not followed as described, so I recommend to follow steps below to avoid any problems during process. It will be set on four *domains* (VMs), on the respective order as follows:

* ***fedora-27***: *TemplateVM* that it will be used by *domains* as base system. ***Note:*** *it can also be a debian template*.
* ***dom0***: *special* domain, where *inter-VMs* rules and policies must be defined.
* ***ssh-vault***: isolated, no network access. It will safeguard public and private keys.
* ***ssh-client***: network access *domain*, used for connecting to external servers. On this *VM*, keys will **NOT** be saved.

### 1. Install packages and *scripts* on *TemplateVM(s)* ###

First and most importante step, I think, that I took *ages* to find out: required *packages* were not installed on *TemplateVM*, **Fedora 27** on Qubes 4. Days ago, I stepped with [@homotopycolimit](https://github.com/henn/qubes-app-split-ssh/issues/9) comment on Github, describing which *packages* must be installed to work.

So, now, let's focus only on ==TemplateVM==, to be used by any other *VM*. In my case, it's ==fedora-27==, but it also works with different images. The most important is to install packages and all files needed by **Split SSH** .

Open Terminal and install packages `nmap-ncat` and `openssh-askpass` on *template* ==fedora-27==, with:

```
$ sudo dnf install nmap-ncat openssh-askpass
```

If, and only if, the choosen *template* is ==debian==, then *packages* names differs: `nmap`, `netcat` and `ssh-askpass`. Install it with the command:

```
# For debian, only:
$ sudo apt-get install nmap netcat ssh-askpass
```

Leave Terminal open in ***fedora-27*** when installer finishes, and create `/etc/qubes-rpc/qubes.SshAgent` file having:

```
#!/bin/sh
# Qubes App Split SSH Script
#    for TemplateVM fedora-27
#
notify-send "[`qubesdb-read /name`] SSH agent access from: $QREXEC_REMOTE_DOMAIN"
ncat -U $SSH_AUTH_SOCK
```

First part is concluded. All tweaks on **fedora-27** *template* have been made and now, following to the second part of this tutorial.

### 2. Configure **dom0**, *special domain* ###

Now, it's ==dom0== turn, the *special domain*. Create this one-line content to `/etc/qubes-rpc/policy/qubes.SshAgent` file.

```
$anyvm $anyvm ask
```

Simple, isn't it?

After creating it, let's move on. We have to create two *VMs* to be used by **Split-SSH** as said before: ***ssh-vault*** and ***ssh-client***. This task can be done with *Qubes Manager* wizard or executing command lines on **dom0** .

For command line option, leave **dom0** Terminal opened, and run each command separately.

```
$ qvm-create -t fedora-27 -l green ssh-client
```

This new *VM* will be created having **fedora-27** as *TemplateVM*, *green* as its color, and **sys-firewall** as default proxy to Internet connection; ***ssh-client*** named and defined. 

Now, to create the second *domain*, ==ssh-vault==, execute:

```
$ qvm-create -t fedora-27 -l black ssh-vault
```

***ssh-vault*** will hold secrets and keys, right? So, it must have no network access. To disable network, and isolate it, run:

```
$ qvm-prefs -s ssh-vault netvm none
```

And it's done! *Domain* ==dom0== is set and our two *frontend domains* were created.

### 3. Configure ***ssh-vault*** ###

As noted before, ==ssh-vault== will hold public and private keys with **no** network access. That said, create or copy your SSH keys to this *VM*, keys to be used by **ssh** command (*ssh keys* are saved in domain `~user/.ssh` directory).

To enable **Split-SSH** on ==ssh-vault==, `~user/.config/autostart/ssh-add.desktop` file is necessary. If `~user/.config/autostart` doesn't exist, create it with `mkdir -pv ~user/.config/autostart`.

Add the following content to the new file, `~user/.config/autostart/ssh-add.desktop`:

```
[Desktop Entry]
Name=ssh-add
Exec=ssh-add
Type=Application
```

Ok?

So, for everything to work properly on ***ssh-vault***, this *domain* must be stopped (and leave it on this status, for now); it can be done by *Qubes Manager* window. 

### 4. Configure ***ssh-client*** ###

To finish this "saga" and moving on to set ==ssh-client== *VM* up, with Internet connection, enabling to connect it to SSH servers out there. 

On files below, watch for `SSH_VAULT_VM` variable, its value define which *domain/VM* name Qubes will try to connect to, creating an *inter-VM* connection on the *vault VM*.

Append content to the file `/rw/config/rc.local` on ==ssh-client==. This *script* will be executed when *VM* starts.

```
####################
# SPLIT SSH CONFIG
#   for ssh-client VM
#   file /rw/config/rc.local
#
# Uncomment next line to enable ssh agent forwarding to the named VM
SSH_VAULT_VM="ssh-vault"

if [ "$SSH_VAULT_VM" != "" ]; then
	export SSH_SOCK=~user/.SSH_AGENT_$SSH_VAULT_VM
	rm -f "$SSH_SOCK"
	sudo -u user /bin/sh -c "umask 177 && ncat -k -l -U '$SSH_SOCK' -c 'qrexec-client-vm $SSH_VAULT_VM qubes.SshAgent' &"
fi
```

To make this *script* executable, set its permission running:

```
$ sudo chmod +x /rw/config/rc.local
```

Last file to be set on ==ssh-client== is `~user/.bashrc`. Append this content to the file:

```
#####################
# SPLIT SSH CONFIG
#   for ssh-client VM
#
# Append this to ~/.bashrc for ssh-vault functionality
# Set next line to the ssh key vault you want to use
SSH_VAULT_VM="ssh-vault"

if [ "$SSH_VAULT_VM" != "" ]; then
	export SSH_AUTH_SOCK=~user/.SSH_AGENT_$SSH_VAULT_VM
fi
```

And done! 

For everything to work on ***ssh-client*** and to apply those changes, **stop/shutdown** its execution on *Qubes Manager* (same thing done on last *domain*), and leave it stopped. Or run the following commands in ==dom0== Terminal: 

```
$ qvm-shutdown ssh-client
$ qvm-shutdown ssh-vault
```

Conclusion and tests
---------------------

Operacional system and *domains* configuration is finished.

***Split-SSH*** is ready to be used on **Qubes**, allowing you to acess SSH servers with different keys (safe and isolated from a *not-so-trusted environment*).

Test it and make sure that everything is working.

### 1. First, start ***ssh-vault*** *domain* ###

Start ==ssh-vault== *VM*, open Terminal. When it launchs, `openssh-askpass` will ask the keys password installed in `~user/.ssh` folder. Type your passphrases when asked.

To verify if ***ssh-agent*** is running on ==ssh-vault==, run:

```
$ eval `ssh-agent -s`
# Retorn: Agent pid 000000
```

### 2. Start ***ssh-client*** *domain* ###

Launch Terminal on ==ssh-client== to also check if ***ssh-agent*** is running.

```
$ eval `ssh-agent -s`
# Retorn: Agent pid 00000
```

### 3. Testing connections between *domains* ###

These two *VMs* (***ssh-vault*** and ***ssh-client***) must be running to test *interVMs* connection. Type the following command on ==ssh-client==, if **dom0** policies were inserted properly, then a window will popup asking for permissions:

```
$ ssh-add -L
```

This command should list all keys installed on ***ssh-vault***. And also try to connect to a SSH server and check if everything is working. 

### 4. Bugs? Errors? ###

In case is not working as expected, try to solve those *bugs* with the  following solutions.

* Shutdown (stop execution) these two *VMs*, ==ssh-vault== and ==ssh-cliente==; start it again following the sequence: first, ==ssh-vault==, then after, start ==ssh-client==.
* In ==ssh-client== files and *scripts*, check if `SSH_VAULT_VM` variable is set with your *ssh-vault VM* name.
* On ==ssh-client==, check if command `sudo chmod +x /rw/config/rc.local` was executed.
* On ==ssh-client==, run `/rw/config/rc.local` *script* and look for any errors returned.
* On ==ssh-client==, delete `~user/.SSH_AGENT_******` file and restart this *domain*.
* Add keys manually in ==ssh-vault==, with `ssh-add`.
* *Debug* connection between ==ssh-client== and a SSH server, adding `-v` parameter to `ssh` command.

And now, FINALLY, it's done!

Did you find any *bugs*? Corrections? Feel free to comment. :-)

##### Footnotes #####

[^1]: Split GPG documentation available at [https://www.qubes-os.org/doc/split-gpg/](https://www.qubes-os.org/doc/split-gpg/)

[^2]: Github link to the project: [https://github.com/henn/qubes-app-split-ssh](https://github.com/henn/qubes-app-split-ssh)

[^longnote]: Extracted from [https://www.qubes-os.org/doc/split-gpg/](https://www.qubes-os.org/doc/split-gpg/)

[^longnote2]: Extracted from [qubes-app-split-ssh](https://github.com/henn/qubes-app-split-ssh)
