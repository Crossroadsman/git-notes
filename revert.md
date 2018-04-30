git revert
==========

Creates a new commit to record the changes necessary to move from the current HEAD to the state before the specified commit. The surface
result is similar to performing a [`git reset`](https://github.com/Crossroadsman/git-notes/blob/master/reset.md) but `revert` preserves an
audit trail by adding new changes to get to the specified point while `reset` moves HEAD back to the prior commit as though the subsequent
changes had never happened.

Syntax: `git revert <commit>` where '<commit>' is the commit we want to reverse (i.e., we want to restore the state immediately before that
commit).
We can use `HEAD` as a shorthand so `git revert HEAD` will create a restore the state before the commit that HEAD currently points to.
