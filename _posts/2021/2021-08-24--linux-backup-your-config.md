---
layout: post
published: true
title: '[LINUX] Backup your config'
subtitle: Automate synchronizing your config between your devices!
date: '2021-08-24 15:00'
---
Hey, 

If you use linux you probably know what are dot files. If not it's small files what are included configuration of applications, there are informations like what theme do you use or what terminal profile is default one. 

Those files are called **dot files** because most of them are started with `.` char what in linux will make then hidden files so regular user will no even know about they existences.

But when you changing system you may want to get them or maybe you want synchronize then between devices that what we gonna to do in this post.

# Credits
I will show how to do this on bare repository, this is [link](https://www.atlassian.com/git/tutorials/dotfiles) to post where I found bare repository as backup system for first time.

## Slave and master
In this idea we will use _slave_ and **master** devices, of course you can also use it if you have only one device, in that case you will have only **master**. 

So **master** gonna be device what give the order so this one where you will would like make a changes and _slave_ gonna be device what will no make new changes but will copy **master** device(s) changes.

## Master
At the first thing what we need to do is make a remote repository, it doesn't matter if it will be in GitHub, GitLab or somewhere else but you need one.

When you have it just copy and paste below script into your terminal:
```
echo "Paste your repository SSH url: " && read &&       # Getting repo url
git init --bare $HOME/.cfg &&                           # Initialize repository
# Add alias to .bashrc
echo "alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'" >> $HOME/.bashrc &&
source $HOME/.bashrc &&                                 # Reload .bashrc 
config config --local status.showUntrackedFiles no &&   # Disable showing untracked files for this repo 
config remote add origin ${REPLY}                       # Connect this repo with remote one

config add ~/.bashrc &&                                 # Add .bashrc to the repo
config commit -m "Add .bashrc"                          # Confirm changes

config branch -m main &&                                # Set branch
config push -u origin main                              # Push changes to remote repo
```

If you only restoring your config into your new master device you should use this one:

_If you already have files what would be overwritten script will move them to `.config-backup`_
```
echo "Paste your repository SSH url: " && read &&       # Getting repo url
echo ".cfg" >> .gitignore &&                            # Prevent recursion
git clone --bare ${REPLY} $HOME/.cfg &&                 # Clone repository
# Add alias to .bashrc
echo "alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'" >> $HOME/.bashrc &&
source $HOME/.bashrc &&                                 # Reload .bashrc
config config --local status.showUntrackedFiles no      # Disable showing untracked files for this 

# Move files with conflicts to ./.config-backup
mkdir -p .config-backup && \
config checkout 2>&1 | egrep "\s+\." | awk {'print $1'} | \
xargs -I{} mv {} .config-backup/{}

config checkout                                         # Apply remote config files
```

## Slave

On the slave device use below script:

**You local configuration gonna be overwritten!**

```
echo "Paste your repository HTTPS url: " && read &&       # Getting repo url
echo ".cfg" >> .gitignore &&                              # Prevent recursion
git clone --bare ${REPLY} $HOME/.cfg &&                   # Clone repository
# Add alias to .bashrc
echo "alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'" >> $HOME/.bashrc &&
source $HOME/.bashrc &&                                   # Reload .bashrc
config checkout --force                                   # Force apply remote config files
```

# Usage

Now on master device if you want add new file just type
```
config add file_name
config commit -m "Adding file"
config push
```

For sending changing about already added files type this
```
config commit -a -m "Updating files"
config push
```

Or this for getting changes from remote to local:
```
config pull            # On master device
config pull --force    # On slave  device
```

But I will suggest you to only type adding command on master device and sending & receiving changes leave for automatic run commands (see below how to)

# Automate <3

So to automate that you need to just add to crontab some commands

## Master
This script will add to crontab hourly command for save changes, receiving changes from remote repo and sending local changes to remote repo

```
crontab -l > ~/.cronTemp &&
echo "0 * * * * /usr/bin/git --git-dir=/home/${USER}/.cfg/ --work-tree=/home/${USER} commit -a -m "Auto backup!"; /usr/bin/git --git-dir=/home/${USER}/.cfg/ --work-tree=/home/${USER} pull; /usr/bin/git --git-dir=/home/${USER}/.cfg/ --work-tree=/home/${USER} push" >> ~/Downloads/cronTemp &&
crontab ~/.cronTemp && 
rm ~/.cronTemp
```

## Slave,

This script will add to crontab hourly command for pulling changes from remote repo and overwrite potentially changes

```
crontab -l > ~/.cronTemp &&
echo "5 * * * * /usr/bin/git --git-dir=/root/.cfg/ --work-tree=/root pull --force" >> ~/Downloads/cronTemp &&
crontab ~/.cronTemp && 
rm ~/.cronTemp
```

# Conclusion 

The bare repository will let us to make pretty nice backup master/slave system and crontab will let us to automate sending and receiving changes. In this example crontab will do it every hour but you can change to apply changes more/less often "_"

For any comments leave it below (._,)