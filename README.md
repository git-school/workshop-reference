# Git Workshop Reference

We covered a lot of material! Here's a reference so you can look back later.

## Under the Hood

Git is made up of four major object types:

* **Blobs** store the contents of files
* **Trees** store the contents of directories
* **Commits** store a pointer to a tree along with metadata about the commit
* **Refs** point, or refer to, other objects in the graph using a friendly name

### Plumbing Commands

* `git cat-file <rev>` shows information about an object in the repository; `-t` shows the type, `-p` shows the contents.
* `git ls-files -s` shows the contents of the index.
* `git rev-parse <rev>` can turn any revision into a SHA that the revision refers to.
* `git rev-list <rev>` can turn revision lists like `A..B` and `A...B` into a list of commits.
* `git merge-base <rev> <rev2>` finds the closest common ancestor (aka, the merge base) of `<rev>` and `<rev2>`.

### Porcelain Commands

* `git add` places files from your working directory onto the index.
* `git commit` commits the current state of the index into the repository.
* `git checkout <rev>` moves `HEAD` to `<rev>`.
* `git checkout -b <branch> <rev>` creates a new branch called `<branch>` at `<rev>` (or at `HEAD` if not specified) and switches to that branch.
* `git reset --hard <rev>` moves `HEAD` to `<rev>` and resets the index and your working directory to match the contents of the repository at `<rev>`.
* `git reflog <ref>` shows the reflog for `<ref>` (or for `HEAD` if none specified).
* `git merge <rev>` merges `<rev>` into your current branch.
  * Passing the `--no-ff` keeps Git from performing fast-forward merges.
  * Passing `-X` allows you to specify an option for the current merge strategy, for example, `-X ignore-space-change`.
  * Passing `-s` allows you to specify a completely different merge strategy, for example, `-s ours`.
* `git rebase <rev>` replays the commits from `HEAD` to the merge base of `HEAD` and `<rev>` on top of `<rev>`, and moves your current branch to the newest created commit.
  * Passing the `-i` flag allows you to modify the commit history between `HEAD` and `<rev>`.
* `git tag <tag> <rev>` creates an unannotated tag called `<tag>` at `<rev>` (or at `HEAD` if none specified).
  * Passing the `-a` flag creates an annotated tag.
* `git cherry-pick <rev>` creates a new commit at `HEAD` containing the changes at `<rev>`.
  * If `<rev>` a merge commit, you must specify the `-m <n>` option, where `<n>` is the parent to treat as the mainline for the merge.
* `git bisect` allows you to find the commit that introduced a change via a binary search algorithm.

### Referring to Revisions

* The first few characters of a SHA
* The name of any ref, like `master` - the SHA referred to by the ref
* `<ref>@{n}` - entry `n` in the reflog for `<ref>`
* `@` - HEAD
* `<rev>@{date}` - `<rev>` at the given date; found using the reflog
* `@{n}` - nth prior version of the current branch
* `<rev>^` and `<rev>^n` - nth parent of `<rev>`; `^` and `^1` is the first parent, `^0` is the commit itself
* `<rev>~` and `<rev>~n` - nth generation ancestor of `<rev>` following first parents only
* `:/<text>` - youngest commit with message text matching regex `<text>`
* `<rev>^{/<text>}` - youngest commit with message text matching regex `<text>` accessible from `<rev>`

Note that suffixes can be changed together, e.g. `HEAD^2~3^` is the first parent of the third ancestor of the second parent of `HEAD`.

Some Git commands accept ranges of commits; the double and triple dot syntax is useful for specifying these:
* `<rev1>..<rev2>` - all commits reachable from `rev2` but not reachable from `rev1`
  * Same as `^rev1 rev2` (aka `rev2 --not rev1`)
* `<rev1>...<rev2>` - all commits reachable from either `rev1` or `rev2`, but not both
  * Same as `r1 r2 --not $(git merge-base --all r1 r2)`

## The Three Trees

The official Git website has [a great page documenting `git reset` and `git checkout`](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified).

[Scripts](https://github.com/git-school/workshop-reference/blob/master/three-trees-watch-scripts.md) for watching the three trees.

## Reflog

https://www.youtube.com/watch?v=Vxc9m_OVyo0

Throughout the course we identified ways to alter the commit history, and we know that since commits are immutable in Git, when we change our Git history with commands like `reset`, `rebase`, and `amend`, what we’re really doing is changing what our branch reference is pointing to, sometimes creating new commits in the process.

But what happens to the old commits that used to be in our branch history? Well if there is no longer a branch pointing to them, they become "dangling commits" and Git will eventually garbage collect them since it assumes they’re no longer needed. Thankfully this garbage collection doesn’t happen for a month or so, because sometimes we’ll find ourselves in situations where we want to recover those “dangling commits” and restore our history to what it was before we changed it.

This can be the case when we do a `rebase`, `reset`, or `amend` and realize that we made a mistake and want to undo the operation. Fortunately, Git keeps a handy chronological history of every commit that `HEAD` has ever been on as well as a description of why `HEAD` moved to that particular commit. This history is knowns as the Reflog and can be viewed via the `reflog` command:

```
>> git reflog
70ad6cd HEAD@{0}: reset: moving to head~
08c773c HEAD@{1}: rebase finished: returning to refs/heads/branch
08c773c HEAD@{2}: rebase: Fixed the thing
da272e2 HEAD@{3}: rebase: checkout master
821ef69 HEAD@{4}: commit: Fixed the thing
dce83e5 HEAD@{5}: pull origin branch: Fast-forward
5a24d37 HEAD@{6}: checkout: moving from master to branch
5feca61 HEAD@{7}: commit (amend): Initial commit
079d913 HEAD@{8}: commit: Initial commit
29345e6 HEAD@{9}: clone: from git@github.com:github/repo.git
```

Using the reflog, we can identify what commit we want to go back to, and use `git reset --hard <sha>` to effectively undo and go back to the state we were in before re-writing our history. To go back `n` commits I would type `git reset --hard HEAD@{n}`

Using the example reflog above, to “undo” my last `reset`, I can type `git reset --hard 08c773c
 ` or simply `git reset --hard HEAD@{1}` to go to the commit that head was on 1 commit ago.

You can also view the reflog for your branches by typing `git reflog branch-name`. Fun fact - when you’re using Git’s stash feature you’re creating commits that get added to a special stash reflog. You can access these commits via `git reflog stash`.

The reflog allows us to re-write history and delete branches without the fear of forever losing the commits. We can take comfort in the fact that we have a whole month to recover dangling commits and restore our branch to whatever previous state we had before. Commits in the reflog that are not danging and are still on a branch will be kept in the reflog for 3 months.

In terms of enhancing your day-to-day workflow, you might consider taking advantage of the reflog by making frequent work-in-progress commits. Once your work is committed, even if you re-write history or delete branches, you can always recover it via the reflog.
