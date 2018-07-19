---
layout: post
title: Setting up a new Python virtual environment for Jupyter notebooks
commentIssueId: 7
tags: python, virtualenv, jupyter
---

A lot of my lab work and course work involved the use of Jupyter notebooks, though the Python dependencies needed conflict with other areas. I've been using [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) to isolate these, and other project, environments from each other. This post goes through the process of installing everything needed to get up and running with a clean Python environment for Jupyter notebooks with separate kernels for each environment, including the installation of `jupyter_contrib_nbextensions` which adds community developed features.

## Initial setup

This only needs to be done once on your machine/user account, in order to get the building blocks in place for creating an indefinite amount of virtual environments for Python. First, you should install a suitable copy of Python on your machine. For macOS, I recommend using the [Homebrew](brew.sh) package manager (installation instructions at the link), then install Python. Note that I'm using Python 3 since Python 2 will be end-of-life'd come the year 2020, but if you're on macOS consider installing Python 2 via Homebrew as well, since the system copy seems to be antiquated. Anyways, to install on mac via Homebrew:

```terminal
$ brew install python3  # Follow any instructions given here from the output
```

In Ubuntu/Debian based systems:

```terminal
$ sudo apt-get install python3 python3-pip
```

On Arch Linux based systems:

```terminal
$ pacman -S python-virtualenvwrapper 
```

Now, assuming Python 3 and `pip` are both installed, install `virtualenvwrapper` and modify your shell start up file according to these instructions: [Install `virtualenvwrapper`](https://virtualenvwrapper.readthedocs.io/en/latest/install.html). I do the following for my system:

```terminal
$ sudo pip install virtualenvwrapper
$ echo "export WORKON_HOME=$HOME/.virtualenvs" >> $HOME/.profile
$ echo "source /usr/local/bin/virtualenvwrapper.sh" >> " >> $HOME/.profile
$ source ~/.profile
```

## Creating new virtual environments

Now every time you need to create a new environment, use the following as an example. My example virtualenv will be named `example`, we'll install Jupyter and any other dependencies, and we'll add a line to `$VIRTUAL_ENV/bin/postactivate` so that when activating the environment, our current working directory will be switched to our project directory `~/path/to/example/code`.

```terminal
$ mkvirtualenv example -p python3 # Note we specify which interpreter to use
(example) $ echo "cd $HOME/path/to/example/code" >> $VIRTUAL_ENV/bin/postactivate
(example) $ pip install ipykernel
(example) $ pip install jupyter_contrib_nbextensions
(example) $ jupyter contrib nbextension install --sys-prefix  # Kinda important
(example) $ pip install jupyter_nbextensions_configurator
(example) $ jupyter nbextensions_configurator enable --sys-prefix
(example) $ python -m ipykernel install --user --name=example
(example) $ pip install <anything-else-you-want>
```

Note that after creating the virtualenv `example`, the environment is automatically activated (which you can tell by the `(example)` prefix in your terminal as well as by running `which python`, which should output a path to the Python interpreter belonging to the environment). When activated, any calls to `python` use the environment's Python interpreter as well as `pip`, which is why we didn't have to call `pip3` instead of `pip`. Note that for installing `jupyter contrib nbextension` and `jupyter nbextensions_configurator`, we used the option `--sys-prefix` which configures these extensions for use in the virtual environment and not the global system enviroment, which is what we're trying to isolate ourselves from. 
