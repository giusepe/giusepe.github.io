---
layout: post
title: How-to Easly Open Multiple Instance of the Visual Studio for Mac
date: '2018-04-05 15:00:58'
---

Actually, this approach will actually solve 2 issues :
How to open Visual Studio for Mac from the terminal in an easy way?
and 
How to open several instances when working with multiple solutions at the same time?

Open your terminal and type `open -n /Applications/Visual\ Studio.app` and hit enter. This will open a new instance of VSfM every time, it solves the problem but it is an ugly solution and a hard command to remember.
So let's make it a little better, let's create an alias to the command making it more rememberable: 

On Terminal, go to your home directory (Type `cd` [enter])
Edit your .bashrc (`vim .bashrc` [enter] or`code .bashrc` [enter]) 
Go to the bottom of the file and paste this lines: 
```bash
vs () {
        open -n /Applications/Visual\ Studio.app
}
```
Save.
Close the Terminal and reopen it.

Now if you type vs the Visual Studio will open a new instance, much better hun?! Easy to remember and elegant.
But what if I'm a Terminal kinda of person and I want to open my solution direct from the Terminal instead of selecting it in the VS interface every time?
That's easy, let's add a condition to our alias:
```bash
vs () {
    if [[ $# = 0 ]]
    then
        open -n /Applications/Visual\ Studio.app
    else
        [[ $1 = /* ]] && F="$1" || F="$PWD/${1#./}"
        open -n /Applications/Visual\ Studio.app "$F"
    fi
}
```
Save.
Close the Terminal and reopen it.

Now navigate to your solution and type something like `vs MySolution.sln`
Vua-lá! You have a new instance of the Solution you just typed opened on VSfM.

That's especially handy when using git on the Terminal.



