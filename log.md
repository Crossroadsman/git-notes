git log
=======

`git log` is used to browse the commit history.

Examples:
---------

- `git log` : list the commits made in the repo in reverse chronological order
- `git log -p` aka `git log --patch` : show the diff ('the patch output') introduced in each commit.
- `git log <directory_name>` : only show commits that had changes in the specified directory
- `git log <branch_name>` : only show commits from a particular branch
- `git log <branch_name> --no-merges` : same as above but exclude merge commits

See Also:
---------
- [Git: 2.3 Git Basics - View Commit History](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History)
- [Mijingo: Understanding Git Log](https://mijingo.com/blog/understanding-git-log)
