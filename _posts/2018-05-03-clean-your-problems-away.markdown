---
layout: post
title: Clean your problems away!
date: '2018-05-03 09:47:10'
---

Don't you hate when you switch branches and your Xamarin project stop working?
Or even when it starts to give you some weird errors, weird behavior "for no reason"?

Well, fear no more! There's a simple solution for that (actually 2):

A while back, before I started using git on a terminal, I had a simple shell script to clean my `bin/obj` folders: 
```bash
find . -iname "bin" -o -iname "obj" | xargs rm -rf
```

What I liked to do was add it inside the project with the name `clean.s` and when a problem arose, I would just close my Solution, run it and reopen the solution and usually, the weird errors would be gone. 

Since I used to live in an area with a very poor connection, cleaning the packages was kinda problematic, but since I moved to NL and I have a true broadband connection at my disposal, I changed it a bit to something like this: 
```bash
find . -iname "bin" -o -iname "obj" -iname "packages" | xargs rm -rf
```

So even the packages are now gone!

One extra functionality of this is: If you, like me, have a Git folder with all your projects and not that much space on your computer, you can add the script on Git folder and clean your projects regularly, that way you will free a lot of space.

Remember that I said 2 solutions? Well, if you are a git terminal user, just run this on the project root:
```bash
git clean -xdf
```

That's even more effective because it will remove anything that's not tracked by git. 
Just be careful to not lose work, add anything you want to the stage area before running it. 