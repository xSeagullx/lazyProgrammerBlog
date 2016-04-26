---
layout:     post
title:      "Git tips: To The Hotfix and Back"
date:       2016-02-28 18:46:00
summary:    Git trick, that allows switching tasks without loosing context and messing with stash.
jsUrl:      "http://cdn.rawgit.com/nicoespeon/gitgraph.js/v1.1.3/build/gitgraph.js"
js: |
  jQuery(document).ready(function() {
    var config = { 
      colors: [ '#5cb85c', '#5bc0de', '#f0ad4e', '#d9534f',], // branches colors, 1 per column
        branch: {
          lineWidth: 8,
          spacingX: 50,
        },
        commit: {
          spacingY: -80,
          dot: {
            size: 10,
            strokeWidth: 5,
            strokeColor: '#000000',
          },
          message: {
            display: true,
            displayAuthor: false,
            displayBranch: !true,
            displayHash: false,
            font: "normal 12pt Arial"
          },
        },
    };
  
    var myTemplate = new GitGraph.Template( config );
    function commonSetup() {
      gitgraph = new GitGraph({
        template: myTemplate,
        orientation: "vertical"
      });
      master = gitgraph.branch("master");
      gitgraph.commit("Master commit");
      feature = master.branch("feature");
    }
  
    var curr = 0
    function msg(str) {
      jQuery("#gitCommand").text(curr + ". " + str)
    }
  
    var steps = [
      function() {
        commonSetup()
        msg("Work on feature branch")
  
        feature.commit({message: "Ongoing work", tag: "HEAD*"});
      },
      function() {
        commonSetup()
        feature.commit({message: "Ongoing work"});
        feature.commit({message: "tmp", tag: "HEAD"});
        msg("Apply git tmp command (working directory clean)")
      },
      function() {
        commonSetup()
        feature.commit({message: "Ongoing work"});
        feature.commit({message: "tmp"});
        master.commit({message: "doHotfix", tag: "HEAD"})
        msg("Checkout hotfix branch. Work on it")
      },
      function() {
        commonSetup()
        feature.commit({message: "Ongoing work"});
        feature.commit({message: "tmp"});
        master.commit({message: "doHotfix"})
        master.commit({message: "finishHotfix", tag: "HEAD"})
        msg("Finalize hotfix")
      },
      function() {
        commonSetup()
        feature.commit({message: "Ongoing work"});
        feature.commit({message: "tmp", tag: "HEAD"});
        master.commit({message: "doHotfix"})
        master.commit({message: "finishHotfix"})
        msg("Checkout feature branch")
      },
      function() {
        commonSetup()
        feature.commit({message: "Ongoing work", tag: "HEAD*"});
        master.commit({message: "doHotfix"})
        master.commit({message: "finishHotfix"})
        msg("Breakdown tmp commit")
      }
    ]
  
    gitNext = function() {
      curr++
      curr = curr % steps.length
      steps[curr]()
    }
  
    gitBack = function() {
      curr--
      curr = (curr + steps.length) % steps.length
      steps[curr]()
    }
    steps[curr]()
  })

categories: git tips
---

### What:
Save the state of unfinished feature-branch, to checkout something else. Then back to it.  

### How:
Make an temporary commit before checking out other branch. Break it down to resume work.

### Demo:
{% include gitGraph.html %}

### Why:
Sometimes you need to pause your current work, and do independent changes in other branch: e.g. hotfix. In many cases you can't checkout from branch with uncommited changes. 
There is a nice `git stash` command, that allows you to save your work as a detached diff, and then apply it when necessary. It's really good, if you need to checkout and keep your changes with you.
But when you change context, it's much better to left a real commit behind, in the branch you are working with. In that case you will never forget where is your work. So, I use two aliases, allowing me to left temporary commit and break it with ease.

### Setup:
Add it to your `~/.gitconfig` in `[alias]` section.

```
[alias]
    tmp = commit -a -m"tmp"
    utmp = reset --soft HEAD~1
```

### Real Example:
```
unfinished_feature ➜ ✗ git checkout master 
error: Your local changes to the following files would be overwritten by checkout:
        change.txt
Please, commit your changes or stash them before you can switch branches.
Aborting

unfinished_feature ➜ ✗ git tmp
[unfinished_feature b408f65] tmp
 1 file changed, 1 insertion(+)

unfinished_feature ➜ git checkout master
 Switched to branch 'master'
 # do your job

master ➜ git checkout -
Switched to branch 'unfinished_feature'
unfinished_feature ➜ git graph

 * b408f65 - (HEAD -> unfinished_feature) tmp (2 minutes ago) <Pavel Dionisev>
 * b79bd19 - Commit 1 (5 minutes ago) <Pavel Dionisev>

unfinished_feature ➜ git utmp

unfinished_feature ➜ ✗ git status
On branch unfinished_feature
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   change.txt

unfinished_feature ➜ ✗ git graph
* b79bd19 - (HEAD -> unfinished_feature) Commit 1 (6 minutes ago) <Pavel Dionisev>
```
