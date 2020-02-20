---
title: "Solve the conflicts of different branches by Git"
layout: post
date: 2020-01-16 12:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- method
- translation
star: false
category: tech
author: Lynx
description: I saw a article about git and I just get sort of bored so I translated it.什么叫闲的蛋疼，这就是。。。闲得去把一篇文章翻译成英文。。。
---



> Well, this is a translation for the article  [解决冲突](https://www.liaoxuefeng.com/wiki/896043488029600/900004111093344)
>
> I just get sort of bored so I make a translation for fun.



Build and switch to a new branch dev from current branch

```ruby
git checkout -b dev
```



when we say `git checkout <branch>`, we mean to switch the branch from the default one to that branch, but still `git checkout -- <file>` means to cancel a change. We get two usage when we use `checkout`. so Git make a new order `git switch` to switch branch.Both work now.

```ruby
git switch -c dev
```

- create and switch to the new branch dev
  git switch master
- switch to the master branch



merge branch dev to master

```ruby
* master
git merge dev
```



delete branch dev

```ruby
git branch -d dev
```



When merged two parallel branch, there would be conflicts.

Prepare new brach feature1

~~~ruby
git switch -c feature1
~~~

- switched to a new branch feature1

Edit `readme.txt` with any words (let's say `ffff`) and commit

~~~ruby
git add readme.txt
git commit -m "feature1"

[feature1 14096d0] AND simple
 1 file changed, 1 insertion(+), 1 deletion(-)
~~~

Switch to master

~~~ruby
git switch master

Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
~~~

Edit `readme.txt` with any words(let's say `mmmm`) and commit in master branch

~~~ruby
git add readme.txt
git commit -m "master~"

[master 5dc6824] & simple
 1 file changed, 1 insertion(+), 1 deletion(-)
~~~

Now branch master and branch feature1 have their own commit. In this situation, Git cannot fast merge.

When we try to merge, it comes:

~~~ruby
git merge feature1

Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
~~~

There is a conflict which must be salved before commited.`git status` can show the conflict file:

```ruby
git status

On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

We can check `readme.txt`,	it's like:

~~~ruby
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of files.
<<<<<<< HEAD
mmmm
=======
ffff
>>>>>>> feature1
~~~

Git will use `<<<<<<<`, `=======`, `>>>>>>>` to mark the content from different branch. We can modify the content, let's say:

~~~ruby
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of files.
another word.
~~~

Then save and commit:

~~~ruby
git add readme.txt
git commit -m "conflict fixed"

[master cf810e4] conflict fixed
~~~

- we can use `git log` with parameters to see the merge info like `git log --graph --pretty=oneline --abbrev-commit`.

We can delete the branch feature1 after what we have done.

~~~ruby
git branch -d feature1

Deleted branch feature1 (was 14096d0)
~~~

So the merge is finished.

