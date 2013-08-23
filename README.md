git-coalesce
============

Scripted `rebase -i` to automatically squash commits based on special symbols
in commit messages.

This command is a wrapper over `git-rebase(1)` that automatically controls the
interactive rebase operation and processes the special symbols it supports,
without human intervention. You can pass it the same options you would pass
to `git-rebase(1)`, except `-i`, which is already used.

Motivation
==========

Why? You ask. I'm a strong proponent of preserving the entire history of my
development, including every mistake I make, and every code block that's
deleted because there is a better way to implement the same functionality.
I'm also a strong proponent of having an elegant project history: every
commit should serve a purpose, should have a meaning, and reading a project's
history should be equally pleasant as reading a top-selling novel.

It's harder than expected to achieve both goals. Yes, it's [a well understood
pattern](https://sandofsky.com/blog/git-workflow.html) to squash commits and
transform a branch with lots of checkpoint commits to one that only contains
stable, safe-stage ones. But the question is how often do you `rebase -i`?

To not ever lose a commit, every time you run `rebase -i`, you have a branch
to backup. So to avoid things getting out of control, you can't rebase too
often. Additionally, rebasing too often is also not an option if you have
several people working on the same feature branch.

But rebasing too sparingly also has its problems. When faced with dozens or
even hundreds of commits accumulated over several days, do you still remember
what each commit does and/or if it passes the tests? If not, how do you know
how to best group or reorder them and form a clean history? And how do you
write detailed commit messages for the combined commits when you no longer
remember all the details?

The problem is the separation of the time when you know the commits, and when
you actually do the rebase to organize them. [Another workflow](http://blog.elliottcable.name/posts/granular_committing.xhtml)
tries to solve this problem by employing two parallel branches: one with
checkpoint commits and one with only stable commits. You normally work on the
first branch, and when the code is in a stable state and ready for a stable
commit, you would checkout the second branch and merge the first one using
`git merge --squash` to create a single stable commit in the branch. So this is
like doing `rebase -i` one squash at a time, on a separate branch. And an added
benefit compared to `rebase -i` is that you have a list of squashed commits
at the end of the commit message of the combined commit, for future reference.
The problem with this approach is that you now have 2x the number of branches,
and each time you rebase against the upstream, you have to rebase two branches.

The idea behind this tool is to solve this problem by documenting how to squash
commits right when you are making these commits, through special symbols in the
commit message. You will do the rebase later, maybe even much later, but you
write down how to do that now, when you are making these commits and know them
well. When your code is in a good state and passes all tests, you document in
the commit message that this commit should be made into a stable one when a
later rebase is done, along with the commit message that should be used for that
stable commit. The actual rebase will then be carried out automatically by this
tool, following your instructions.

An added benefit of this workflow is that you have complete information in the
old branch, including how checkpoint commits are organized into stable commits
and both the old and new commit messages, which you can backup for future reference.
Personally I back up my old branches in a separate namespace `refs/archive/*`.
You can also pass the `--list` option to make `git-coalesce` list at the end of
each stable commit's message which checkpoint commits are combined to form it.
This way you can easily find these commits in the future if needed.


Install
=======

Place the `git-coalesce` file anywhere in PATH, and make sure it has the `execute`
permission.


Usage
=====

```
usage: git coalesce [options] [--exec <cmd>] [--onto <newbase>] [<upstream>] [<branch>]

    This command is a wrapper over git-rebase(1) that automatically
    controls the interactive rebase operation and processes the
    special symbols it supports, without human intervention. You can
    pass it the same options you would pass to git-rebase(1), except
    "-i", which is already used.

    Note that if the rebase process is interrupted, you should use
    "git coalesce --continue" instead of "git rebase --continue"
    after resolving the issue. Just replace "rebase" with "coalesce"
    in the entire process.

    Supported Symbols

    1. One or more opening braces ({) at the beginning of the first
       line (summary) of the commit message, and zero or more closing
       braces (}) at the end of the line. This commit is considered a
       milestone and is combined with all previous commits up until
       the last milestone to form a single commit. The last commit in
       the sequence is always considered a milestone no matter what.

    2. One or more opening braces ({) followed by an exclamation mark
       (!) at the beginning of a line in the commit message. This
       symbol starts a block.

       An exclamation mark (!) followed by one or more closing braces
       (}) at the end of a line in the commit message. This symbol
       ends a block.

       Everything in such a block is used as part of the message of
       the new combined commit to which this commit belongs.

       Say commits c1, c2, ..., cN are combined, and each has a block
       of text m1, m2, ..., mN in their commit messages between these
       symbols, respectively (multiple such blocks in a commit message
       are concatenated to form a single block). The message of the
       combined commit is formed by aggregating m1 ~ mN in the following
       order: the first paragraph of mN, m1, m2, ..., m(N-1), the rest
       of mN.

       Of course, not all commits need to contain these symbols. In
       most cases you only need to include the message for the
       combined commit in the message of cN.

       If, however, none of c1 ~ cN contains these symbols, the default
       commit message which concatenates the messages of all "member"
       commits is used.

optional arguments:
  -h, --help            show this help message and exit
  -l [NUM=15], --list [NUM=15]
                        list all squashed commits at the end of the combined
                        commit's message. For commits with summaries shorter
                        than NUM chars, only include commit ID; otherwise
                        include both ID and summary
  -v, --version         show program's version number and exit
```


License
=======

MIT
