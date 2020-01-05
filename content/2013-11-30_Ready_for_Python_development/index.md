+++
title = "Ready for Python development with Emacs in just 60 seconds"
date = 2013-11-30T12:40:00Z
[taxonomies]
categories = ["articles", "recipe"]
tags = ["python", "emacs", "ide", "explained"]
+++
_This post demonstrates how you can configure a very decent environment for Python development with Emacs, in just 60 seconds._

A dream is now true: The first time you start Emacs, it automagically downloads and configures all plugins you need.  Emacs is then just ready to work and you can start typing code immediately. 

## For the impatient

1. Save configuration files you eventually have!

```bash
#!/bin/bash

cd $HOME 
tar cpf dot-emacs.ORIGINAL.tar.gz .emacs .emacs.d
mv .emacs   dot-emacs.ORIGINAL
mv .emacs.d dot-emacs.d.ORIGINAL
```

2. Remove any Emacs configuration files you eventually have.
```bash
#!/bin/bash

rm -r -f .emacs .emacs.d
```

3. Install Python libraries

This should be done preferably inside a virtual environment.
```bash
#!/bin/bash

workon py276 #-- py276 is a virtualenv I'm using
pip install epc
pip install jedi
pip install elpy
```

4. Download my .emacs file onto your home folder.
```bash
#!/bin/bash

cd $HOME 
wget https://raw.github.com/frgomes/dot-emacs/master/dot-emacs.el
ln -s dot-emacs.el .emacs
```

4. Start emacs. It will configure itself when it first run!
```bash
#!/bin/bash

emacs test.py
```

Features in a nutshell

* python-mode and cython-mode
* jedi: provides auto completion
* flymake: highlight syntax errors as you type

That's it: ready for coding in 60 seconds :)

## Contribute

Please [let me know](https://github.com/frgomes/.emacs.d/issues) if you find issues.

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
