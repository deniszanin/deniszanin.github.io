Using GIT in Qubes 4 (with Split SSH and Split GPG)
===================================================

[A few posts ago](https://github.com/deniszanin/deniszanin.github.io/blob/master/_posts/06-30-2018-using-split-ssh-qubes-4-en.md), I wrote how to configure **Split SSH** on **Qubes 4** and how to set it properly to use *SSH* with your keys in a different environment, to connect to your *SSH* servers, e.g. **Split SSH** on *Qubes 4.0* has a really nice *catch*: it allows you to store your keys in a *vault AppVM*, with **no** Internet connection, while another *AppVM* is used to connect to your servers, holding *zero* keys, meaning that it'll only connect, locally, to the *vault* and read keys stored there. *Almost* same concept applied to **Split GPG**.

On this post, I want to share an accomplishment: how about having an *AppVM* dedicated to *development* with **git** tool, and being able to increase its security *committing*, *pushing repos* to [Github](https://github.com/), via *SSH*, and signing with your **GPG** subkey? 

On this case, we'll have three different **AppVMs**: two *vault domains* which stores a **Github** *SSH* key pair (1) and other *vault* holding your GPG key (2); the third is a *dev* workstation, which has *git tool* installed and it will be used for development and for *pushing* your commits to the **Github** servers.

#### *Domains* structure: ####

* **ssh-vault**: contains your private and public *SSH* keys (*RSA, DSA, ed25519, etc*).
* **gpg-vault**: contains your private and public *GnuPG* keys.
* **work-dev**: your *work domain* which has Internet connection, and will **NOT** hold any of your private and public keys; it will connect to *gpg-vault* and *ssh-vault* for get information about your corresponding keys, and it will only conclude its operation after **user** authorization.

That's a *scenario* that looks hard to set up, but having all your *vaults* independently configured, *work-dev* will be straight-forward.

## Configure ##

### Configure Split GPG ###

I highly recommend to read [Qubes Split GPG on Qubes Documentation](https://www.qubes-os.org/doc/split-gpg/), in case you haven't yet. It covers all basic and advanced topics over **GnuPG** with **Qubes**. On this post you're reading, I'll not cover it extensively. A useful step-by-step guide is available there. To match our case here, replace *work-gpg* referenced in the documentation as the *vault domain* to **gpg-vault**, name used here in this post.

dom0, *"the special domain"*
----------------------------

1. Install package **qubes-gpg-split-dom0**, with the command `sudo qubes-dom0-update qubes-gpg-split-dom0`.
2. Add required policies to **dom0** to allow *interdomain* connections, editing `/etc/qubes-rpc/policy/qubes.Gpg` and adding the following line **to the top of file qubes.Gpg**.

`work-dev gpg-vault allow`

gpg-vault
---------

The idea is to store your GPG keys here, as described in documentation,

> Start with creating a dedicated AppVM for storing your keys (the GPG backend domain). It is recommended that this domain be network disconnected (set its netvm to none) and only used for this one purpose. In later examples this AppVM is named work-gpg, but of course it might have any other name.

After creating it, let's set this up:

1. Install package **qubes-gpg-split**, typing on its *TemplateVM*:

```
# for Debian Template:
sudo apt install qubes-gpg-split # for Debian Template

# **OR**, for Fedora Template, running
sudo dnf install qubes-gpg-split # for Fedora Template
```

2. Add *"autoaccept"* variable in `~/.bash_profile` (or `~/.profile`, for Debian Template based) file.

`echo "export QUBES_GPG_AUTOACCEPT=86400" >> ~/.bash_profile`

work-dev
--------

We certainly have to make some changes over this *domain* to finish **Split GPG** configuration, but I'll leave for later on this text. It'll be easier to change everything once on *work-dev*, besides going back and forth over this *domain*.

### Configure Split SSH ###

Configuring **Split SSH** requires a bunch of patience and concentration, but you can do it, and you will, *right*? Like I said before, I already wrote about it in an extensively post with most of the major steps explained. Follow [this guide about how to configure *Split SSH* on Qubes 4](https://github.com/deniszanin/deniszanin.github.io/blob/master/_posts/06-30-2018-using-split-ssh-qubes-4-en.md).

**An important note:** the concept is **ssh-vault** *domain* to hold all of your *SSH* keys. *All* your *SSH* keys are secure by this *domain*, which means - if you're familiar with *SSH* keys -, that at some point you'll have a conflict between its names and will have to create new names for each file/key. That's where `ssh-agent` and `ssh-add` comes to deal with those conflicts.

So, after setting **ssh-vault** up, like I explained [here](https://github.com/deniszanin/deniszanin.github.io/blob/master/_posts/06-30-2018-using-split-ssh-qubes-4-en.md), you must generate a new **key pair** to be used with **Github** (as authorization access) and add its *fingerprint* to *Github user interface*. And then, switching remote URLs from *HTTPS* to *SSH* to **git** use a safer connection between servers and your computer.

**On Github documentation about these topics**:
* [Generating a new SSH key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
* [Adding a new SSH key to your Github account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)
* [Switching remote URLs from HTTPS to SSH](https://help.github.com/articles/changing-a-remote-s-url/#switching-remote-urls-from-https-to-ssh)

**An import note, parte two:** generating a new *SSH key pair* on **ssh-vault**, you'll be prompted for file location to save the key. At this moment, don't hit *Enter*; instead, complete with a different location (name) for the key.

```
#  Generating a new key pair in ssh-vault, 
# change its default location when prompted.
Enter a file in which to save the key (/home/you/.ssh/id_rsa):

# Enter a file with different location and name, like
                /home/user/.ssh/my_github_rsakey
        or      /home/user/.ssh/github_secret_sshkey
        etc. etc. etc.
```

*Don't worry! Take your time, I'm still here.**

Doing these things, you should have now in **ssh-vault** a new *SSH key pair* that must be working with **Github**. Last thing to do is to add the new key to `ssh-agent` in **ssh-vault**. That must be done because its custom name we choose a few minutes ago, sometimes `ssh-agent` doesn't check all files/keys inside *SSH* home folder. Only after typing the following command `ssh-add YOUR_GITHUB_KEY_LOCATION`, *ssh-agent* will recognize the key and add it to its *database*.

```
# For example:
ssh-add ~/.ssh/my_github_rsakey
# ...etc. etc. etc.
```

If it worked, `ssh-agent` asks key's passphrase and then a confirmation message is printed, like `Identity added: /home/user/.ssh/my_github_rsakey (/home/user/.ssh/my_github_rsakey)`. To double check, run `ssh-add -L` command to list all your keys installed on **ssh-vault** *domain*.

And finally, add the following lines to your local *SSH* config file, `~/.ssh/config` to specify for *SSH* which information is going to be used for `github.com` host.

```
# Github account for SSH access                                                Host github.com
    Hostname github.com
    User git
    IdentityFile ~/.ssh/my_github_rsakey
```

Done! Are you still here? :)

### Configure *work-dev* domain ###

Our last move in this tutorial is to set everything up on your *dev domain*, which is named as **work-dev** here. Remember what is its purpose? Working on *git* tool for signed commits and pushing/pulling over *SSH*, **with** Internet connection, and with *zero* private/public keys held inside it.

One topic by time, let's move to configure **work-dev**.

git tool
---------

*git* tool is able to handle with GPG natively, but in order to work with **Split GPG**, *git* configuration file (`~/.gitconfig`, by default) must be edited. The reason is that *Split GPG* uses a different *client* named `qubes-gpg-client-wrapper`, instead of `gpg` command, for managing connections between *client AppVM* and *vault*.

Editing `~/.gitconfig` is our next goal.

```
[user]
    name = YOUR NAME
    email = YOUR EMAIL ADDRESS
    signingkey = 0xYOUR_GPG_KEY_ID!
[commit]
    gpgsign = true
[gpg]
    program = qubes-gpg-client-wrapper
```

**Note**: variable `signingkey`, in `~/.gitconfig`, must be declared so that *git* use a specific **GPG subkey**. To work, must be appended with `!`, as explained [here](https://stackoverflow.com/questions/48230336/git-uses-wrong-subkey-for-signing-commits-with-gpg-key) (found it after several issues signing my commits).

Split SSH
----------

Again, if you haven't configured **Split SSH** on **work-dev** *AppVM*, follow [steps detailed here](https://github.com/deniszanin/deniszanin.github.io/blob/master/_posts/06-30-2018-using-split-ssh-qubes-4-en.md), over the item **4. Configure ssh-client**. **work-dev** has exactly same function as *ssh-client*, introduced on this page I linked. Tasks to do are:

1. Append content to `/rw/config/rc.local` file, and making it executable.
2. Append content to `~user/.bashrc`.

The "contents" to be appended on files are available on same [page I linked](https://github.com/deniszanin/deniszanin.github.io/blob/master/_posts/06-30-2018-using-split-ssh-qubes-4-en.md) *two lines ago*.

Split GPG
----------

Backing on track of **Split GPG** configuration, we continue to finish our setup on *GPG*. This time working only with **work-dev** *domain*.

Add `export QUBES_GPG_DOMAIN` to define *Qubes Split GPG* variable to your `~/.bash_profile` (or `~/.profile`). Running the following command will have same effect.

```
echo "export QUBES_GPG_DOMAIN=gpg-vault" >> ~/.bash_profile
```

Restart your **work-dev** *AppVM*, and run `gpg -K` on a Terminal. If you've previously added any key to your **local GPG folder**, then it will print a list with your keys stashed on **work-dev** (remember that for security ground, it shouldn't have keys on it), or giving you a empty return, meaning that there's no private/public keys stored. Running the following command, it will make **work-dev** to find keys on the **gpg-vault** and list it for you.

```
qubes-gpg-client -K
```

And... it's done!

## Conclusion ##

Are you still here? So, congrats! It was **NOT** an easy task... but you made it! And thank you for your patience. To finish, test your workstation.

Restart each *AppVM* and test it out.

First, start **ssh-vault** and it will ask you passphrases for you *SSH* keys. Then, start **gpg-vault** and **work-dev**.

Bugs?
------

Did you find any bugs? Corrections? Feel free to comment. :-)
