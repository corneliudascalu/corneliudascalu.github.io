---
title: "Git is fun"
date: 2015-11-26T15:31:49+02:00
description: 'Many people are put off by Git because it seems difficult. For people transitioning from SVN, for example, the idea of local commits is often difficult to grasp. Using the command line tools is even more off-putting for people used to rich GUI tools.
However, after getting past the initial learning curve, you start feeling the power of Git on the command line.'


---

Many people are put off by Git because it seems difficult. For people transitioning from SVN, for example, the idea of local commits is often difficult to grasp. Using the command line tools is even more off-putting for people used to rich GUI tools.

However, after getting past the initial learning curve, you start feeling the power of Git on the command line.

But before saying anything else, I want to argue for using Git on the command line as the only way to really learn Git. GUI tools are limited by the number of items they can cram into a menu, so they have to compromise, and dumb it down. On the contrary, the command line gives you nearly unlimited freedom. Once you know enough to handle day-to-day tasks, you are free to learn and explore at your own pace (or not). And once you know how Git works behind the scenes, you know how to use any GUI client.

----

## Aliases

Git aliases are used to create shortcuts for often used commands. They save you time by letting you write just a few characters instead of a long, complicated command. They also help you avoid mistakes.

```git
[alias]  
 st = status  
 ci = commit  
 co = checkout  
 rev = merge --no-ff --no-commit  
 br = branch  
 ls = ls-files
```

## Customized log

The log is one of the most often used commands in Git. As such, it is a good thing that it can be customized for different use cases. Sometimes you only need a quick glance at the recent changes, and sometimes you need the full picture, so you can display only what you need.

```git
git log --graph  
git log --oneline  
git log --pretty=format:git log <revision>  
git log -L <start>,<end>:<file>![image](/images/git-is-fun/1.jpg)
```

{{< figure src="/images/git-is-fun/1.jpg" caption="This is what the default log looks like" >}}
{{< figure src="/images/git-is-fun/2.jpg" caption="And this is how it can be customized" >}}

## Diff tools

Diffing is where the command line is not suited for human abilities. We are used to nice shapes, arrows, and colors, while the terminal displays pluses and minuses. There are very powerful GUI diff tools that help you. On Linux, one of the best is Meld.

{{< figure src="/images/git-is-fun/3.png" caption="Meld screenshot" >}}

Where *diff* shines in the command line though, is comparing different versions of a file, or show the difference between commits easily.

Show the differences between the current *index.html* file and the one on the *develop* branch:

```git
git diff develop -- index.html
```

Show all the differences between two branches (or commits):

```git
git diff develop..master
git diff HEAD..HEAD~5
```

If you want to use the terminal, though, [cdiff](https://github.com/ymattw/cdiff) is an awesome tool that improves the diff experience a lot:

{{< figure src="/images/git-is-fun/4.png" caption="`git diff develop | cdiff -s`" >}}

## tig

If you are not confident enough to use git commands but still want to use the terminal, you can use a tool like [tig](http://jonas.nitro.dk/tig/). It simplifies browsing the repository and helps with some simple commands.

{{< figure src="/images/git-is-fun/5.png" >}}

## Advanced usage

Where Git really shines is the advanced commands that are probably not available in your favorite git GUI client.

To list the branches that are merged (or not) in the current branch:

```git
git branch --merged  
git branch --no-merged
```

For recovering lost branches or commits

```git
git reflog
```

To find the common ancestor of two branches

```git
git merge-base <branch1> <branch2>
```

To rewrite history

```git
git rebase -i <revision>
```

To commit partially

```git
git commit -p  
git commit *.java  
git commit *Test.java  
git commit **controller/**
```

Combine all this with some Bash magic and you can get amazing results.

---

Finally the most fun part of using the Git command line tools is the satisfaction of achieving what you wanted and knowing exactly what you did. It’s a huge difference between clicking a single button in a GUI client like [SourceTree](https://www.sourcetreeapp.com/) to “release” a feature (while the client runs some obscure magic behind the scenes) and doing each step by hand, and crafting each commit message the way you want it.

It’s like eating a burger from the street corner versus cooking an entire meal yourself.