git revert
==========

Creates a new commit to record the changes necessary to move from the current HEAD to a prior commit. The surface result is similar to 
performing a [`git reset`](https://github.com/Crossroadsman/git-notes/blob/master/reset.md) but `revert` preserves an audit trail by adding 
new changes to get to the specified point while `reset` moves HEAD back to the prior commit as though the subsequent changes had never 
happened.

Syntax: `git revert <commit>`
