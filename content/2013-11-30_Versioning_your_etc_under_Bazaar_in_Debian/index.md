+++
title = "Versioning your /etc under Bazaar in Debian"
date = 2013-11-30T12:37:00Z
[taxonomies]
categories = ["articles", "recipe"]
tags = ["linux", "debian", "configuration", "versioning", "etckeeper", "bazaar", "explained"]
+++
_In this post we will see how we can manage configuration of our Linux boxes so that we keep a better control of changes, so that we can revert unwanted changes, if that is the case._

Suppose you've installed some packages on your Linux box, which made some configurations which messed your system. In general, you just need to uninstall the packages you've just installed and everything goes back to normal. In general. Unfortunately, this is not always the case. When it is not the case, you face an uncomfortable situation: you don't really know what changed and obviously you cannot revert the changes that you are not aware of them in the first place.

## Managing configuration

One way to tackle the problem presented above is installing a tool called ``etckeeper``, which provides two things:

* a versioned ``/etc``, so that any changes to it can be tracked;
* integration with your package manager, so that you keep track of packages being installed or uninstalled.

This way, you know which files changes, you know what changed and these changes are properly documented so that you can track which packages were installed or uninstalled and you know which files were affected.

### Installation

It is recommended that you install ``etckeeper`` as soon as you first install the operating system into your computer. The earliest you do that, the better.

```bash
#!/bin/bash

apt-get etckeeper -y
ls -ald /etc/.git
```

> As ``etckeeper`` is installed, it automatically starts versioning ``/etc``, employing Git as default version control system.

## Choosing Bazaar as an alternate VCS instead of Git

In this post we show what needs to be done if, for whatever reason, you prefer another version control system, instead of Git. In this case, we are going to use Bazaar as the sake of demonstration. The first thing you need to do is "uninitialize" ``etckeeper``, which removes the ``.git`` folder from ``/etc``:

```bash
#!/bin/bash

etckeeper uninit
ls -ald /etc/.git
```

Now you need to tell ``etckeeper`` which VCS you would like to use. Edit ``/etc/etckeeper/etckeeper.conf`` and make sure you have only the alternate VCS enabled. Remember to disable Git, as we can see below:

```bash
#!/bin/bash

head -5 /etc/etckeeper/etckeeper.conf
# The VCS to use.
#VCS="hg"
#VCS="git"
VCS="bzr"
#VCS="darcs"
```

Let's now install Bazaar and create the repository again, this time using Bazaar.


```bash
#!/bin/bash

apt-get install bzr -y
etckeepet init
ls -ald /etc/.bzr
```

## Versioning /etc

Review what was done and perform your first commit.


```bash
#!/bin/bash

cd /etc
bzr status | less
bzr commit -m 'first commit'
bzr status
```

We will also create two clones of our repository, one in the same computer and another one remotely. This is how we can create a clone in the same computer, under folder ``/srv/etckeeper``

```bash
#!/bin/bash

cd /srv
bzr init --no-tree etckeeper
cd /etc
bzr config push_location=/srv/etckeeper
bzr push
```

Now we are going to create another repository on a remote computer. In order to to that, we first need to initialize a bare repository on the remote computer and then we push our repository from our local computer onto the remote computer. Below you can see an example on how we can initialize a bare repository on a remote computer using Bazaar, which is intended to keep the configuration of our local computer, named ``pinguim`` in this example:

```bash
#!/bin/bash

ssh myself@server
COMPUTER=penguim
mkdir -p /srv/bzr/etckeeper
bzr init --no-tree /srv/bzr/etckeeper/${COMPUTER}
exit
```

Now we get back to our local computer and we push a copy from your second repository (the one on your second hard drive) onto the remote server:


```bash
#!/bin/bash

cd /srv/etckeeper
COMPUTER=$( hostname )
bzr config push_location=bzr+ssh://myself@server/srv/bzr/etckeeper/${COMPUTER}
bzr push
```

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
