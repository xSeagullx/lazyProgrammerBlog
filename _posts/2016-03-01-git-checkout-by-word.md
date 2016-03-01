---
layout:     post
title:      "Git tips: Pattern-based Checkout and Search for Branch"
date:       2016-03-01 10:39:00
summary:    Checkout a branch using just a part of it's name. Ideal if you have branch naming convention with unique part in it (e.g. JIRA number).
            Or select a branch same way for other operation.


categories: git checkout bash get-branch-name tips
---

### What:
Checkout a branch using just a part of it's name. Ideal if you have branch naming convention with unique part in it (e.g. JIRA number).
Or select a branch same way for other operation.

### Examples:
```
21:00:16 ➜  lazyProgrammerBlog (master) ✗ git br
  example/one-name
  example/other-name
* master

21:00:32 ➜  lazyProgrammerBlog (master) ✗ git col one
Switched to branch 'example/one-name'

21:00:42 ➜  lazyProgrammerBlog (example/one-name) ✗ git col other
Switched to branch 'example/other-name'

21:01:00 ➜  lazyProgrammerBlog (example/other-name) ✗ git col name
result is ambiguous
  example/one-name
* example/other-name

21:01:05 ➜  lazyProgrammerBlog (example/other-name) ✗ git col name 1
Switched to branch 'example/one-name' 
```

### Just find branch name, no checkout:
```
21:03:45 ➜  lazyProgrammerBlog (example/one-name) ✗ git fbl name
example/one-name
example/other-name

21:03:56 ➜  lazyProgrammerBlog (example/one-name) ✗ git fbl name 2
example/other-name
```

### Other usage (merge):
```
21:06:25 ➜  lazyProgrammerBlog (example/one-name) git merge `git fbl name 2` -m"Merge message"
Updating d3768ab..e54ec45
Fast-forward (no commit created; -m option ignored)
 _posts/2016-03-01-git-checkout-by-word.md | 59 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 1 file changed, 59 insertions(+)
 create mode 100644 _posts/2016-03-01-git-checkout-by-word.md
```

### Other usage (delete ):
```
21:19:13 ➜  lazyProgrammerBlog (example/one-name) ✗ g col mas
M       _posts/2016-02-29-git-tips-intro.md
A       _posts/2016-03-01-git-checkout-by-word.md
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
21:19:32 ➜  lazyProgrammerBlog (master) ✗ g br -D `git fbl name 1`
Deleted branch example/one-name (was d3768ab).
21:19:38 ➜  lazyProgrammerBlog (master) ✗ g br -D `git fbl name 1`
Deleted branch example/other-name (was e54ec45).
```

### Why:
Checking out branches is one of the most common git routine. Normally you have to type it's name, which is inconvenient if you have a lot of local branches, or if you follow some verbose convention.
In our team we have git-flow inspired convention, where branch name uses following schema: `(prefix)/(JIRA_ticket)-human-readable-branch-name`. e.g. `feature/JIRA-123-my-super-cool-feature`.
Typing it, even with autocompletion is a bit annoying.

### Setup:
Add it to your `~/.gitconfig` in `[alias]` section. _Disclaimer_ this is an awfull code, I know.

```
[alias]
    co = checkout
    br = branch
    col = "!f() { \
             (test $# == 2) && git co $(git br | grep $1 | sed -n \"$2 p\") || ( (test $# == 1) && ( (test `git br | grep -c $1` == '1') && (git co `git br | grep $1`) || echo \"result is ambiguous\n$(git br | grep $1)\" ) || echo 'git col pattern [num]');\
           }; f"
    fbl = "!f() { (test $# == 2) && (git br | grep $1 | sed -n \"$2 p\" | cut -c 3-) || git br | grep $1 | cut -c 3-; }; f"
```

### Disclaimer:
I strongly believe that you can write code for these aliases much better, I'm not a bash expert. Through I hope that this post may be a good inspiration for you! =)
