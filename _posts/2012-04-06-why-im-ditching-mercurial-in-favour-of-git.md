---
layout: post
title: Why I'm ditching Mercurial in favour of Git
categories: tools
tags: git heroku heroku mercurial hg vcs
status: publish
type: post
published: true
header-img:
    src: "img/two-buildings.jpg"
comments: true
---

When I first decided I wanted to use a DVCS to manage the source code of my various projects, I was initially sold on using Mercurial by <a title="HG Init" href="http://hginit.com/">Joel Spolsky's excellent HG Init tutorial</a> (which is a great intro DVCSs in general by the way - highly recommended).

As a result, for the last year or so I've been using Mercurial day-in-day-out. It's a great tool and I would still recommend it.

However, over the last few months, as I've been forced to use Git more and more (primarily due to GitHub's status as the VCS provider du jour for open source projects). As a result I've come to realize that, despite it's arguably slightly more complex command interface, Git fits my needs better than Mercurial.

Here are the top 5 reasons why:
# 1. Automatic support for renaming/moving files
If, like me, you tend to rename class files numerous times when attempting to settle on the name that works best, you might find file renaming a bit of pain in Mercurial. By default, Mercurial handles a rename as a Remove of the original filename and an Add of the renamed filename. This is a little suboptimal as it means by default we lose the file history at the point we rename the file.

To ensure the history is preserved, Mercurial allows you to let it know about rename operation, either by performing the rename using Mercurial itself...

`hg rename oldfilename newfilename`

...or by telling it about a rename you've already performed...

`hg rename --after oldfilename newfilename`

But this is kind of a pain. If you're working with an IDE such as Eclipse, you'll want to do your renaming in the IDE to take advantage of its ability to automatically update references. If you decide to rename a number of files this way you're faced with needing to tell Mercurial about each one individually.

Moreover, consider moving a folder containing a large number of files. Again, a file move is treated just like a rename in Mercurial - as a Remove and an Add. If you decide to reorganise your repo's folder structure using an IDE you're potentially faced with needing to tell Mercurial about a ton of renames.

Contrast this with Git. Git tracks the contents of files rather than filenames. As a result it detects is able to handle file renames or moves automatically by simply comparing the contents of the original file and renamed file. No further user action is required.

I've found this to be a massive time saver when working with Java projects in Eclipse where it's common to want to refactor a package structure resulting in a lot of moved files.

# 2. Heroku integration
Heroku is awesome. I use Heroku a lot to deploy my Rails and Node.js apps. I find that as an application platform it provides just the right level of abstraction allowing you to run your own code, with your own dependencies inside your own web server, without having to deal with any of the complexity associated with provisioning the infrastructure itself.

In order to deploy your code to Heroku you need to use Git. <a href="https://devcenter.heroku.com/articles/git">You simply add your Heroku app instance as a Git remote and push to it</a>. Heroku takes care of the rest.

Clearly, if you're going to use Git to deploy code to Heroku, it makes sense to use Git to manage your code as well.

# 3. The Git Index
The Git Index is essentially the staging area to which you must add any file changes you want to commit prior to actually committing them. By default, if you simply make changes to files in your working copy of a Git repo, those changes will not be committed if you issue a git commit command. First you must add to the index the file changes you want to include in the commit using git add.

At first this might seem like additional work and/or complexity, but it is actually incredibly useful. I frequently find myself making a bunch of changes together which logically serve different purposes. e.g. I notice a spelling mistake in some file unrelated to the changes I'm making and correct it while I'm in the neighbourhood. In this scenario, the Git Index allows me to create two separate commits from set of changes I've made: one for the spelling mistake correction, and another for the actual changes I planned to make.

Contrast this with Mercurial. Although internally it uses an index, Mercurial does not expose this to the user. By default all changes to a working copy are included in a commit (though new files must be explicitly added). If you want to include only a subset of your changes in a given commit, you must list the files you want to include <em>at the point you  make the commit</em>.

## Mercurial


    hgdemo tom$ hg st
    M a.txt
    M b.txt
    hgdemo tom$ hg commit -m "Committing a.txt" a.txt
    hgdemo tom$ hg st
    M b.txt


## Git


    gitdemo tom$ git status
    # On branch master
    # Changes not staged for commit:
    #   (use "git add ..." to update what will be committed)
    #   (use "git checkout -- ..." to discard changes in working directory)
    #
    #	modified:   a.txt
    #	modified:   b.txt
    #
    no changes added to commit (use "git add" and/or "git commit -a")
    gitdemo tom$ git add a.txt
    gitdemo tom$ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD ..." to unstage)
    #
    #	modified:   a.txt
    #
    # Changes not staged for commit:
    #   (use "git add ..." to update what will be committed)
    #   (use "git checkout -- ..." to discard changes in working directory)
    #
    #	modified:   b.txt
    #
    gitdemo tom$ git commit -m "Committing a.txt"
    [master 980c086] Committing a.txt
     1 files changed, 1 insertions(+), 0 deletions(-)


This may seem like a small functional difference, but the ability to iteratively add files to and review the Git Index prior to actually committing means you have a lot more confidence that what you are committing is actually correct.

Furthermore it encourages good behaviour. With Mercurial it is very easy to get into the habit of just committing everything together because it's the path of least resistance. Git forces you to think about what you're committing which gives you a better chance of structuring your commits more logically so they can be cherry-picked for merging more easily later on.

# 4. Better colour highlighting of file status and diffs
Out of the box, Git does a great job of using colour highlighting to indicate which files are currently in the index, which are not and how file contents have changed when doing a diff.

Though its possible to install an extension to enable this behaviour in Mercurial, I am lazy and don't really feel like I should have to :-).

# 5. It's more widely used
Ultimately, I've yet to encounter an open source project I want to hack on which is managed using a Mercurial repo. Both on Google Code and Github, Git is the platform of choice. As such, Git appears to be well established the de facto DVCS of choice among the OSS community.

Sorry Mercurial. I'll be using the HG-Git plugin to migrate all my code to Git.
