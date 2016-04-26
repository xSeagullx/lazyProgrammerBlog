---
layout:     post
title:      "Git revisions explained"
date:       2016-03-15 18:46:00
summary:    "Git has a somewhat complex set of concepts to refer a particular point in history. We have commits, branches, tags, remotes. Moe than enough to get confused!
             I'll try clarify this by sorting out all this concepts in holistic system, connecting them together from simple to complex."
categories: git

jsUrl:      "http://cdn.rawgit.com/nicoespeon/gitgraph.js/v1.1.3/build/gitgraph.js"
js: |
  var config = { 
    colors: [ '#5cb85c', '#5bc0de', '#f0ad4e', '#d9534f',], // branches colors, 1 per column
      branch: {
        lineWidth: 8,
        spacingX: 35,
      },
      commit: {
        spacingY: -25,
        dot: {
          size: 8,
          strokeWidth: 3,
          strokeColor: '#000000',
        },
        message: {
          display: true,
          displayAuthor: false,
          displayBranch: true,
          displayHash: false,
          font: "normal 10pt Arial"
        },
      },
  };

  var myTemplate = new GitGraph.Template( config );
  function createGraph(id) {
    return new GitGraph({ elementId: id, template: myTemplate, orientation: "vertical" })
  }
---

As in any version control system, in git we have a concept of __revisions__. To put it simple, revision is a tracked state of project.
Every revision in git have it's own `sha-1` [^1] hash, and can be referenced by it. From this very simple concept we will start.

### Commits
So, what is `sha-1` hash I've mentioned? It's unique 40 character string, containing digits and english letters. In this post we don't care how we get it, let's just stick with that fact, that it's unique and used to identify particular revision.
My last commit here is `482707027ec693dd65a9b5b5b87d0356b48b77b9`. As you can see it is very long, and cumbersome to type. But don't worry, it can be shortened to _minimal non ambiguous sequence_.
In my case it's 4 characters, i.e. command `git show 4827` will work. As repository becomes bigger and bigger,
you'll have to use longer sequences, but it's rarely more than 8-10 characters.

> from __Git Pro Book__
> Generally, eight to ten characters are more than enough to be unique within a project. One of the largest Git projects, the Linux kernel, is beginning to need 12 characters out of the possible 40 to stay unique.

But even short sha is not descriptive enough to work with it. When we work, we wan't to name our revisions in a bit more human-centric way. And next concept I'll show is __tags__.

### Tags
Actually, git has 2 types of tags.
__Lightweight tag__ is a named pointer to existing commit. Just like an alias to existing commit.
Creating this kind of tag it simple: `git tag blog_post_about_revisions`. After that you can use it everywhere, where git expects revision.
Now and I can check it out, without remembering any sha-hashes. `git checkout blog_post_about_revisions`.

There is also an annotated tag. It's a bit different, it has metadata and own sha-code. In that sense it is more like commit of it's own, than just alias.
So, it's used for a long-term history, mostly releases. Don't use it for simple aliasing purpose.

Tag's are fixed in time, and if we wan't our textual alias to move with us, as we progress in our work, we should use __branches__.

### Branches
Branches lay in the core of git, and they are much more wide-known and used, than tags.
At the same time, branch is also just a named pointer to particular commit, exactly as lightweight tag! It does't have it's own sha, it's just an alias.
Main difference from tag, is that it will move as you add commits, allowing you to use same alias for _latest state_ of your work.
With this in mind, it's not surprising, that if you provide branch name to command expecting single revision, last commit from branch will be taken.
Example of these commands are `show` or `cherry-pick`.

### HEAD
HEAD is synthetic branch, that always points to the currently checked out revision.
If there is no underlying branch, you are checking out, i.e. if you checkout hash, tag, or even stash entry, like `stash@{1}`, you'll end up in so-called detached HEAD state.

{% include gitGraph.html %}
<style>
		#gitCommand > code, #gitCommand > small { vertical-align: middle; }
		#gitCommand > code { font-size: medium; }
</style>
<script>
	var commonSetup = function() {
		var gitgraph = createGraph("gitGraph");
		gitgraph.branch("master");
		return gitgraph;
	}
	var curr = 0;
  var msg = function(str) {
    jQuery("#gitCommand").html(curr + ". " + str)
  }

  var steps = [
		function() {
			commonSetup().commit("[HEAD] C1");
			msg("<code>commit C1</code> <small># On master</small>")
		},
		function() {
			commonSetup().commit("C1").commit("[HEAD] C2");
			msg("<code>commit C2</code> <small># On master: Both HEAD and master moved.</small>")
		},
		function() {
			commonSetup().commit("C1").commit("[HEAD] C2")
			msg("<code>checkout C2</code> <small># Detached HEAD: No active branch for this checkout</small>")
		},
		function() {
			commonSetup().commit("C1").commit("C2").branch("HEAD").commit("C3");
			msg("<code>commit C3</code> <small># Detached HEAD: HEAD updated, master unaffected</small>")
		},
  ];
  gitNext = function() {
      curr++;
      curr = curr % steps.length;
      steps[curr]();
    };
  
    gitBack = function() {
      curr--;
      curr = (curr + steps.length) % steps.length;
      steps[curr]();
    };
  steps[curr]();
</script>

As a proper branch, HEAD will always progress forward as you do a commit, always pointing last one.
To publish changes made in that state you have to create branch or tag out of it.

### Referring parents
Remember, that in git all revisions are interconnected. It'a graph, and every node have one or more parents. You may refer these parents using `~N` and `^N` syntax. [^2]
Examples: `HEAD~6`, `master^2` or even `v.1.2~1^2~4`. As you see in the last example they can be combined.
Here we have a small illustration:
<div class="row" style="margin-top: 1em"><canvas id="gitGraphParent"></canvas></div>
<script>
  var gitgraph = createGraph("gitGraphParent");
  var master = gitgraph.branch("master");
  master.commit("C1 (HEAD~3, HEAD~1^2~2)");
  var feature = master.branch("feature");
  feature.commit("F1 (HEAD~1^2~1)");
  master.commit("C2 (HEAD~2 or HEAD^^ or HEAD^~)");
  feature.commit("F2 (HEAD~1^2 or HEAD~^2)");
  feature.merge(master, "Merge branch 'feature' (HEAD~1 or HEAD^1)");
  master.commit("C3 (HEAD)");
</script>
`~2` is going 2 commits back. In case of a merge commit on the way, we follow it's first parent. Merge commit has multiple parents. `^2` is used to select one of those, going back only one step, but taking second parent.
Again, this accessors might be used for any revision. `HEAD~1^2~2` is correct, thought somehow cryptic revision reference, going 2 commits back from current state, then one commit following second parent, and then again 3 commits back.
If you ommit number after `~` or `^`, it will meant `~1` or `^1` respectively.

### Referring checkouts history
Last, but not least. Git tracks your checkout history. It's kind of alias again, accessible via `@{-N}` syntax, where `N` is the number of steps you have taken.
So, if you were checking out from `master` to `hotfix`, `git show @{-1}` will show you topmost master commit. For checkout command, there is a shortcut: `git co -` is equivalent to `git co @{-1}`.
It's not widely known, but useful when you often move back and forth between revisions or if you write scripts/aliases, that do checkout.
Consider following alias as an example:  

```
done = "!f() { git co master && git merge @{-1}; }; f"
```

Although it's a bit artificial, it illustrates how branch ref be used to checkout master, and merge your work in it.
To use this alias you have to be on the feature branch you want to merge.

### Two words about remotes
There is nothing special about remotes. Same revisions, with their own unique hashes are stored in your local git repository when you fetch changes.
References to remote branches are automatically created, based on name of remote branch, and prepended with the name of remote.

### Esoteric revisions
It's also worth mentioning, that both stash and reflog are valid revisions. Stash command creates actual commit (with proper parent),
and provides symbolic name to it: `stash@{1}`. As far as I know, reflog's symbolic entry, i.e. `HEAD@{4}` is just a pointer, referring particular repository state.

### Holmes toolkit: `rev-parse` and `name-rev`
And before you start experimenting remember 2 commands that will not let you get lost and always give you information about revision you need.

First one is `rev-parse [--short]`, giving you sha-1 hash (or shortened version) of any symbolic revision.

```
~ git rev-parse HEAD
482707027ec693dd65a9b5b5b87d0356b48b77b9
~ git rev-parse --short HEAD  
4827070
~ git rev-parse --short HEAD^1
4b58d25
~ git rev-parse --short master
4827070
~ git rev-parse --short lightWeightTag
4827070
~ git rev-parse --short annotatedTag
c473254
~ git rev-parse --short annotatedTag~
4b58d25
```

I'm on `master` branch and it is `4827070`. Previous commit is `4b58d25`. Both tags are looking at same revision as `HEAD`.  

> Note, that although my annotated tag is pointing same revision as my lightweight tag, rev-parse give different revision.
It's curious, because if I checkout `annotatedTag`, I'll end up on same commit I'm now, and there is no way to actually checkout `c473254`.

Another direction is also possible, Use `name-rev` to convert revision to unique symbolic name.
There could be multiple symbolic representations of single revision, and this command gave you only one of these variants.

```
> git name-rev 4827070        
4827070 master
> git name-rev 4b58d25
4b58d25 master~1
> git name-rev 4b58d25 --tags
4b58d25 tags/lightWeightTag~1
```

### Summary

Almost everything in git is a revision. Every revision has unique `sha-1` id, which can be shortened for convenience.
For every revision you can create a textual alias: lightweight tag. Branch is an alias,
but it can move as you add commits. You can see it as an updatable lightweight commit. HEAD is a synthetic branch, which is always with you.
Stash reference and reflog reference are also valid revision reference and you can see them as lightweight tag too.
You can refer your previously checked-out state with `@{-N}` syntax and move back in history following commit's parents with `~` and `^`.
To deep-dive in any reference you may use `rev-parse` and `name-rev` commands.

Hope this post helped you to build better understanding of git revision system.

### References

[^1]: [Wikipedia SHA-1](https://en.wikipedia.org/wiki/SHA-1)
[^2]: [Pro Git: 7.1 Git Tools - Revision Selection](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection)
