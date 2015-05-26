---
layout: post
comments: true
categories: [unix, macox]
tags: [unix, macos, bash, named folders]
title: Named Folders for MacOS/Unix 
excerpt: FAR Manager that I was using on Windows for a long time had a very neat extention called "Named Folders". It allowed you to make a shortcut for any directory you wanted and type it quickly in the consolde or in FAR. After switching to MacOS I really wanted to replicated the same feature in the console environement.
---

So, the idea is pretty easy. Having a folder 'path/to/my/local/folder' we usually have to type ```cd /path/to/my/local/folder``` to get there in the console. That's not really fast.

What *Named Folders* offers is that you can say that **/path/to/my/local/folder** is mapped to a shortcut **mylocalfolder**, and then type ```cd :mylocalfolder``` to get there.

Well, actually, FAR extention was even better - you could go to any folder, type ```cd::shortcut ```to make a shortcut, and then in any place type ```cd:shortcut``` to get there.

I'm not that good in Bash scripting to make the exact copy, but here what I could come up in a couple of minutes, mostly copy-pasting code from Stackoverflow answers.

## 1. Make a folder to store shortcuts
- first of all make a *bin* folder by calling 
```
mkdir ~/bin
chmod a+rwx ~/bin
```
- then make a *shortcuts* file
```
touch ~/bin/shortcuts
```

You will use this file the following way to make a new shortcut:
```
"echo mylocalshortcut:/path/to/my/local/shortut" >> ~/bin/shortcuts
```

## 2. Make a 'cd' command wrapper to use you shortcuts
- create a wrapper file
```
touch ~/bin/cd_wrapper
```
- use your favourite editor to add these lines inside:
```bash
# format: [semicolon] shortcut colon destination [semicolon] ...

IFS=$'\n' read -d '' -r -a lines < $HOME/bin/shortcuts
cdataexports=$(printf '%s;' "${lines[@]}")

echo "Applying shortcuts: " ${lines[@]}

export CDDATA=$cdataexports

cd () {
    local dest=$1
    if [[ $dest == :* ]]
    then
        [[ $CDDATA =~ (^|;)${dest:1}:([^;]*)(;|$) ]]
        dest=${BASH_REMATCH[2]}
    fi
    if [[ -z $dest ]]
    then
        echo "cd_wrapper >> " $dest
        cd
    else
        builtin cd "$dest"
    fi
}

```
- Add created wrapper as a source, you can add these lines in your **~/.bash_profile** file
```
source $HOME/bin/cd_wrapper
```


## Usage

To make a shortcut type:
```"echo mylocalshortcut:/path/to/my/local/shortut" >> ~/bin/shortcuts```

To use a shortuct type:
```cd :mylocalshortcut ```