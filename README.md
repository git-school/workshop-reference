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
