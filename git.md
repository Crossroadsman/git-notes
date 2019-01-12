Git
===

Index
-----

1. [Install and Setup](#s1)
2. [HowTos](#s2)
   1. [Combine Multiple Git Repos](#s2.1)
   2. [Use logs](#s2.2)
   3. [Work With Relative Refs](#s2.3)
   4. [Undo](#s2.4)
   5. [Browse the Trees](#s2.5)
   6. [Merging and Rebasing](#s2.6)
   7. [Conflict Resolution](#s2.7)
   8. [Cloning](#s2.8)
   9. [Fetching/Pulling](#s2.9)
   10. [Diff](#s2.10)
   11. [Tagging](#s2.11)
   12. [Remove Large Files](#s2.12)
3. [Understanding Git](#s3)
   1. [Trees](#s3.1)
   2. [Commits](#s3.2)
4. [Other Useful Resources](#s4)
5. [Glossary](#s5)
6. [Footnotes](#s6)


<a name="s1">1: Install and Setup</a>
-------------------------------------

### 1: Install git ###
Update the apt package cache

```console
sudo apt-get update
```

Install git

```console
sudo apt-get install git
```

### <a name="s1.2">2: Configure global settings</a> ###
username:

```console
git config --global user.name "<name>"
```

user.email:

```console
git config --global user.email <email_address>
```

Note that you can view git's settings using the `-l` (list) argument, and additionally view the source of the settings with 
the `--show-origin` flag:
```console
$ git config -l --show-origin
file:/home/userName/.gitconfig      user.name=UserName
file:/home/userName/.gitconfig      user.email=user@email.com
file:.git/config        remote.origin.url=git@github.com:UserName/some_repo.git
```

In the above example, the first two settings are globals (saved in `$HOME/.gitconfig`), while the third is a repo-level setting.

You can also edit the ~/.gitconfig file directly. It has the following format:
```
[user]
	name = UserName
	email = user@email.com
[push]
	default = simple
        # push.followTags will push all the refs that would be pushed without this option, and also
        # push annotated tags in refs/tags that are missing from the remote
        # but are pointing at commit-ish that are reachable from the refs
        # being pushed.
	followTags = true
[alias]
    hist = log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
```


### 3: Configure system settings ###
Append the following to `~/.bashrc`:

```bash
# set $VISUAL and $EDITOR (including for child shells)
export EDITOR='vim'
export VISUAL='vim'
```

Reload bashrc:

```console
. ~/.bashrc
```

Confirm that bashrc gets loaded for [login][link01] sessions by checking .profile and .bash_profile:

```console
less ~/.bash_profile
less ~/.profile
```

One of them (at least) should load .bashrc.


### 4: Link SSH key with Github ###
a) Create a key (if there isn't one already at `~/.ssh/id_rsa.pub`)

```console
ssh-keygen -t rsa -b 4096 -C "<email_address>"
```

b) copy the contents of the created public key to the clipboard

c) go to github and select add a new SSH key, pasting the contents of the ssh key into the field


### 5: Clone a repo ###
```console
git clone git@github.com:<user_id>/<repo>.git
```


<a name="s2">2: HowTos</a>
--------------------------
Note that a lot of the content in this section comes from the excellent [Git Immersion](http://gitimmersion.com) tutorial by Jim Weirich.
If you are not me, you should stop reading this and just go there.

###  <a name="s2.1">1: Combining Two Git Repos</a> ###

1. [Create a new empty repository New.](#s2.1.1)
2. [Make an initial commit because we need one before we do a merge.](#s2.1.2)
3. [Create a new branch for old repo A's files](#s2.1.3)
4. [Create a remote association to old repo A](#s2.1.4)
5. [Pull the remote's files](#s2.1.5)
6. Move all the remote's files into a subdirectory
7. Checkout master
8. Merge the branch back into master
9. Repeat steps 3–8 for all remaining old repos

#### <a name="s2.1.1">Create a new empty repository New</a> ####

```console
mkdir new
cd new
git init
```

#### <a name="s2.1.2">Make an Initial Commit</a> ####
This creates a new master branch.
```console
echo New > README.md
git add -A
git commit -m "Initial commit"
```

#### <a name="s2.1.3">Create a new branch for old repo A's files</a> ####
For clarity, branches are in snake case, remote names in Pascal case

```console
git checkout -b repo_a
```

#### <a name="s2.1.4">Create a remote association to old repo A</a> ####
`RepoA` is the name we are using to associate the remote repo's url inside this repo.

```console
git remote add -f RepoA https://www.git.files/path/to/repo
```

Note: [`-f`](https://git-scm.com/docs/git-remote) means perform a fetch immediately after the remote is set up

#### <a name="s2.1.5">Pull the remote's files</a> ####
Will probably need to explicitly allow unrelated histories

```console
git pull RepoA master --allow-unrelated-histories
```

Then will probably need to do a merge. 

The master repo's README might have been overwritten, get it back with `git checkout master -- README.md`. (the `--` 
separator comes from shell where it is [used in bash built-in commands and many other commands to signify the end of 
command options, after which only positional parameters are 
accepted](https://unix.stackexchange.com/questions/11376/what-does-double-dash-mean-also-known-as-bare-double-dash))


### <a name="s2.2">Use Logs</a> ###

- `git log`  
  Lists all commits in reverse chronological order.  
  Example:  
  ```console
  $ git log
  commit 8da33089a28c38b2106caa6f89c761861e8dec97 (HEAD -> master)
  Author: UserName <user@email.com>
  Date:   Thu Oct 25 15:06:33 2018 +0100

      Added a comment
  ```

- `git log --pretty=oneline`  
  List all commits in reverse chronological order, showing only hash and commit message.  
  Example:
  ```console
  $ git log --pretty=oneline
  d91d1fd7a5a1ec809d7994a6d5c11409163429d3 (HEAD -> master, origin/master) Convert initial test to unittest test case
  ```
  
- You can add additional flags to further customise the output (these are mostly (all?) stackable), such as:
  ```console
  $ git log --pretty=oneline --max-count=2  # show just the last two commits
  $ git log --since='7 days ago' --until='5 minutes ago'  # show commits from the specified date range
  $ git log --author=<name>  # show commits only for the specified user, quotes optional, username or email accepted
  $ git log --all  # "Pretend all refs in refs/, along with HEAD, are listed on the command line as <commit>." In practice, this means show other branches
  ```

- Here is an example of a [custom format](http://gitimmersion.com/lab_10.html) 
  (see the link for detailed explanation of each of the flags):  
  ```console
  $ git log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
  * 7bf0bf1 2018-09-28 | Added a comment (HEAD -> master) [Jim Weirich]
  * 9cf3f21 2018-09-28 | Added a default value [Jim Weirich]
  * 94e1b8b 2018-09-28 | Using ARGV [Jim Weirich]
  * f656098 2018-09-28 | First Commit [Jim Weirich]
  ```

### <a name="s2.3">Work with Relative References</a> ###

Instead of always using the commit hash, we can use a couple of symbols to reference commits relatively:
- `^`  
  Refers to the previous commit.  
  Example:  
  `HEAD^` - the previous commit before the current HEAD
- `~`  
  Refers to the commit *n* commits before the specified commit.  
  Example:  
  `v1~2` - the commit two commits prior to the one tagged `v1`.

We can also use tags (get a list of tags using `git tag`) to name a particular commit. Let's suppose we wanted to tag the current commit 
as `v1` and the immediately previous commit as `v1-beta`:
```console
$ git tag v1
$ git checkout v1^  # alternatively we could have used v1~1
Note: checking out 'v1^'.

You are in 'detached HEAD' state. You can look around, make experimental
<...>

HEAD is now at e283143 Handle case where no arg
$ git tag v1-beta
$ git checkout master
Previous HEAD position was e283143 Handle case where no arg
Switched to branch 'master'
$ git hist  # this is an alias, see s1.2 for how to create
* 951db27 2018-10-28 | Add comment' (HEAD -> master, tag: v1) [UserName]
* e283143 2018-10-28 | Handle case where no arg (tag: v1-beta) [UserName]
* d1ed2ea 2018-10-28 | Add command line arg [UserName]
* 25dff66 2018-10-28 | create ruby file [UserName]
* 48c6a21 2018-10-27 | initial commit [UserName]
```

### <a name="s2.4">Undo</a> ###

#### <a name="s2.4.1">Unstaged Files</a> ####
Suppose we have been editing `myfile.py` and want to revert it. We can do this by simply checking out `HEAD`'s copy of 
the file. It will silently overwrite the changed file in the working directory.

```console
$ git checkout myfile.py
```

#### <a name="s2.4.2">Staged but not Committed</a> ####
Suppose we've added a bad version of `myfile.py` to the staging area. We can reset the staging area to how it is under 
the `HEAD` commit:

```console
$ git reset HEAD myfile.py
```

Note we could omit the filename in which case all the staged files would be reverted to the version from `HEAD`.

Note also that this use of reset only affects the staging area. The working directory is unchanged, so the bad version 
of `myfile.py` is still the version in the working directory.

#### <a name="s2.4.3">Committed Files</a> ####

##### revert #####
The first way to reverse an unwanted commit is to make a `revert` commit. This creates a new commit that exactly reverses the commit 
that we want to change. This is good because it preserves all history and thus a) if best for keeping an audit trail and b) makes it 
safe to use on any branch, even publicly shared ones.

Example:

```console
$ git revert HEAD
```

Note we don't have to use HEAD, we could use an absolute or relative reference to any commit.

##### reset --hard #####
The second way to reverse an unwanted commit is to make it look like the bad commit never happened. This makes for a cleaner-looking
history. However, it can cause confusion on shared branches so shouldn't be used in such cases.

Example:

```console
$ git hist
* 01925f8 2018-10-28 | Revert "Oops, we didn't want this commit" (HEAD -> master) [UserName]
* e174f47 2018-10-28 | Oops, we didn't want this commit [UserName]
* 951db27 2018-10-28 | Add comment (tag: v1) [UserName]
* e283143 2018-10-28 | Handle case where no arg (tag: v1-beta) [UserName]
<...>
$ git tag oops # this is just to show later that this commit still exists
$ git reset --hard v1 # v1 is equivalent to oops~2 or 951db27
HEAD is now at 951db27 Add comment
```

Note that the unwanted commits still exist, we can see them by browsing `all` commits:

```console
$ git hist --all
* 01925f8 2018-10-28 | Revert "Oops, we didn't want this commit" (tag: oops) [UserName]
* e174f47 2018-10-28 | Oops, we didn't want this commit [UserName]
* 951db27 2018-10-28 | Add comment (HEAD -> master, tag: v1) [UserName]
```

##### Amend a commit #####

An `amend` is equivalent to performing a soft `reset` followed by a commit.

Example:

We've edited `myfile.py` to include a comment specifying the author.

```console
$ git add myfile.py  # stage the revision containing the author comment
$ git commit -m "Add an author comment"
[master 8ef923b] Add an author comment
 1 file changed, 1 insertion(+)
```

Now we realise that we should have included both author name and email in the comment. We edit `myfile.py` to include the amended
comment.

```console
$ git add myfile.py # stage the revision containing the author/email comment
$ git commit --amend -m "Add an author/email comment"
[master a3cf871] Add an author/email comment
 Date: Sun Oct 28 13:33:36 2018 -0600
 1 file changed, 1 insertion(+)
```

The new commit takes the place of the previous commit. Note that the amended commit is a different commit, with a different hash. And 
we can still access the original commit if we can remember the hash ID.


### <a name="s2.5">Browse the Trees</a> ###
We can browse the trees using `cat-file` with the `-t` (type) and `-p` (dump) flags.

First, let's get a SHA for a commit:
```console
$ git hist --all --max-count=1
* e1627a6 2018-10-29 | Add shebang (hello) [UserName]
```

The type of hash e1627a6:
```console
$ git cat-file -t e1627a6
commit
```

Dump the contents of that hash:
```console
$ git cat-file -p e1627a6
tree 0b6e846af1c3ff99979436ef9c12ac5c1cacbb81
parent 63d927bc1e2ff8f78a3eb831749949baf6344312
author UserName <user@email.com> 1540846607 -0600
committer UserName <user@email.com> 1540846607 -0600

Add shebang
```

This tells us:
- `tree` : the hash of the corresponding tree
- `parent` : the hash of the parent commit
- `author` : the identity of the author, together with the UNIX timestamp and timezone of the commit
- `committer` : the identity of the committer, together with the UNIX timestamp and timezone of the commit

If we look at the tree, we can see what files were in that commit, and the hash reference to their contents:
```console
$ git cat-file -t 0b6e846
tree
$ git cat-file -p 0b6e846
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	README.md
100644 blob e31070457fb6fe5df98048304b2bb0e6a8bcc514	hello.py
```

Here we see:
- `100644` : the Unix permissions of the file
- `blob` : 'binary large object'. Like a file except without any metadata (the file's metadata is in the tree).
- `e69de29` : the SHA hash for the file itself
- `README.md` : the filename

We can look at the file itself:
```console
$ git cat-file -t e69de29
blob
$ git cat-file -p e69de29
# README.md
This is the readme file
```

We can follow the commit history back to the original commit by repeatedly getting the dump of the commit then dumping the parent commit.

### <a name="s2.6">Merging and Rebasing</a> ###
By merging `master` into your working branch periodically, you can pick up any changes from the mainline and keep your working branch
compatible. But this approach can produce ugly commit graphs. Instead it may be possible to rebase rather than merging.

Use rebase exactly like merge:

```console
$ git hist --all
* 7ea0a76 2018-10-29 | Updated readme (HEAD -> master) [UserName]
| * da4901a 2018-10-29 | Updated Rakefile (greet) [UserName]
| * 2bf6fd5 2018-10-29 | Added greeter class [UserName]
|/
* 48c6a21 2018-10-20 | Initial commit [UserName]
$ git branch
* master
  greet
$ git checkout greet
Switched to branch 'greet'
$ git rebase master
First, rewinding HEAD to replay your work on top of it...
Applying: Added greeter class
Applying: Updated Rakefile
$ git hist --all
* 747cfe2 2018-10-29 | Updated Rakefile (HEAD -> greet) [UserName]
* 4711d2b 2018-10-29 | Added greeter class [UserName]
* 7ea0a76 2018-10-29 | Updated readme (master) [UserName]
* 48c6a21 2018-10-27 | Initial commit [UserName]
```

Notes:
1. The "Updated Rakefile" and "Added greeter class" commits now have different hashes (necessary because history has been rewritten)
2. The apparent order of the commits has changed

#### When to use which? ####
Don't use rebase if:
- the branch is public and shared: rewriting a publicly-shared branch tends to screw up other team members; or
- the exact commit history is important.

Thus, use rebase for short-lived local branches and merge for branches in the public repo.

#### Fast-forward ####

When performing a merge, Git can detect whether `HEAD` is an ancestor of the commit being merged. When it does, it can,
instead of writing a merge commit, simply move the `HEAD` pointer to the incoming commit.

While this is very clean, sometimes you might want to preserve the history of the branch topology (e.g., merging a 
feature branch). Using a FF would hide that there was ever a feature branch. We can force Git to perform a merge commit 
and preserve the topology by using the no fast-forward flag:
```console
$ git merge --no-ff my-feature-branch
```
**TODO: Add example output**


### <a name="s2.7">Conflict Resolution</a> ###

Example merge error message:
```console
Auto-merging lib/hello.rb
CONFLICT (content): Merge conflict in lib/hello.rb
Automatic merge failed; fix conflicts and then commit the result.
```

Example conflicted file:
```ruby
<<<<<<< HEAD
require 'greeter'

# default is 'World'
# author: UserName (user@email.com)
name = ARGV.first || "World"

greeter = Greeter.new(name)
puts greeter.greet
=======
# author: UserName (user@email.com)
puts "What's your name?"
my_name = gets.strip

puts "Hello, #{my_name}!"
>>>>>>> master
```

Notes:
1. Note that the bookends and divider are exactly 7 characters long
2. The top section is HEAD (the current branch); the bottom section is the branch being merged into the current branch

#### Resolution Process ####
1. Open the conflicted file
2. Fix it however you want (you don't have to just pick between the two versions, you can mix and match code from both versions)
3. `git add` the file
4. `git commit` the file

#### Resolution Process (binary files) ####
1. Determine which version you want
2. `git checkout` the appropriate file. If you want HEAD's version (the branch receiving the merge):  
   ```console
   $ git checkout --ours -- path/to/some_binary.file
   ```
   If you want the version from the branch being merged in:
   ```console
   $ git checkout --theirs -- path/to/some_binary.file
   ```
3. `git add` the file
4. `git commit` the file


Note you can use third-party merge tools.


### <a name="s2.8">Cloning</a> ###

Syntax:
```console
$ git clone repo_old repo_new
Cloning into 'repo_new'
done.
```

Note that `repo_old` is a path/url to a directory containing a .git subdirectory. If `repo_new` is omitted, git will attempt to create
a directory with the same name as the directory containing .git in the `repo_old` path.

The clone of the original will look almost identical to the original.

One of the only differences will be visible when viewing the log of the clone:
```console
$ git hist --max-count=1
* 747cfe2 2018-10-29 | Updated Rakefile (HEAD -> master, origin/master, origin/HEAD, origin/greet) [UserName]
```

The clone will get a reference (`remote`) to the original repo (with the default name `origin`):

```console
$ git remote
origin
```

We can get more information:
```console
$ git remote show origin
* remote origin
  Fetch URL: /Users/username/repo_old
  Push URL: /Users/username/repo_old
  HEAD branch: master
  Remote branches:
    master tracked
    greet
  Local branch configured for 'git pull':
    master merges with master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

Note that `git branch` only shows local branches. We can view remote branches by adding the `-a` flag:
```console
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/greet
```

### <a name="s2.9">Fetching/Pulling</a> ###

Example situation: remote has made a change to `readme.md`.
```console
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Nothing to commit, working tree clean
```

At first glance, this makes no sense: we know that there is a change on the master branch of origin. However, git only knows the contents
of the local copy of `origin/master` (we'll call this \<local\> origin/master). This is now out of sync with the real `origin/master` 
(we'll disambiguate by specifying \<remote\> origin/master where applicable).

We can get update our copy of `origin/master` using `fetch`:
```console
$ git fetch
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 1), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
From /Users/username/repo_old
  747cfe2..d6742dc master -> origin/master
```

We can observe that this update is now in our local tree:
```console
$ git hist --all --max-count=2
* d6742dc 2018-10-29 | Changed readme.md in original repo (origin/master, origin/HEAD) [UserName]
* 747cfe2 2108-10-29 | Updated Rakefile (HEAD -> master) [UserName]
```

Note that although our tree now has the up to date commits from \<remote\> origin/master, they are still in the `origin/master` branch
and not in our `master` branch.

If we rerun `status` we will now see the message we might have expected earlier:
```console
$ git status
On branch master
Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)
  
Nothing to commit, working tree clean.
```

Thus fetch will essentially download the commits for the remote branch but will preserve them in the local version of the remote branch.

To merge those changes, we simply do a `merge` as though `origin/master` were any other branch:
```console
$ git merge origin/master
Updating 747cfe2..d6742dc
Fast-forward
  readme.md | 1+
  1 file changed, 1 insertion(+)
```

Note that `git pull` is equivalent to `git fetch` followed by `git merge <branch>`.


#### Tracking additional remote branches ####

By default, cloning a repo will see all the remote branches but will only bring in the contents of the tracked branch. We can track 
additional branches as follows:
```console
$ git branch -a  # view the remote branches
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/hello
  remotes/origin/master
  remotes/origin/pythonise
$ git branch --track hello origin/hello
Branch 'hello' set up to track remote branch 'hello' from 'origin'.
$ git branch -a # view all the branches again
  hello
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/hello
  remotes/origin/master
  remotes/origin/pythonise
```

Note that when we have multiple branches on local we can push all of them to remote by using:
```console
$ git push --all -u
```
(Where `--all` means 'all refs under refs/heads should be pushed', `-u` means 'for every branch that is up-to-date or 
successfully pushed, add upstream (tracking) ref')


### <a name="s2.10">Diff</a> ###

Here are some common ways of performing a diff:

- `git diff`: Compare the working tree to the staging area
- `git diff --cached [<commit>]`: Compare the staging area to the specified commit.  
                                  If no commit specified, defaults to `HEAD`.  
				  If there is no `HEAD`, shows all staged changes.
- `git diff <commit>`: Compare the working tree to the specified commit
- `git diff <commit> <commit>`: Compare the changes between two arbitrary commits
- `git diff <commit>..<commit>`: Mostly equivalent to `git diff <commit> <commit>` except that in this case, one of the commits could
                                 be omitted and `HEAD` would be substituted automatically
- `git diff <commit_a>...<commit_b>`: Compare the changes between commit_b and the common ancestor of commit_a and commit_b


### <a name="s2.11">Tagging</a> ###

#### Tag Commands ####

- list tags:
  ```console
  $ git tag
  ```
- create tag:
  - *lightweight*  
    A lightweight tag is just a pointer to a commit
    ```console
    $ git tag <tagname> [commit]
    ```
    Example tag with a datestamp (format: `DEPLOYED-20181116-162753-0700`):
    ```console
    $ export TAGDATA=$(date +DEPLOYED-%Y%m%d-%H%M%S%z)
    $ git tag $TAGDATA
    ```
  - *annotated*  
    Annotated tags are stored as full objects in the Git database. They're checksummed; contain the tagger's name/email/date; contain
    a tagging message; can be signed and verified with GPG
    ```console
    $ git tag -a <tagname> [-m "<message>"] [commit]
    ```

- replace a tag:
  ```console
  $ git -f <tagname>
  ```

- view tag data:
  ```console
  $ git show <tagname>
  ```

- push tag(s):  
  `push` doesn't send tags by default.
  ```console
  $ git push origin <tagname>
  ```
  Or if you have a lot of tags to push:
  ```console
  $ git push origin --tags
  ```

#### Tag Syntax ####
[Not all characters are valid in a tagname](https://git-scm.com/docs/git-check-ref-format).

### <a name="s2.12">Fix Committed Large File</a> ###
Situation: you've got a repo with no .gitignore, you've added a bunch of node
packages and committed, then you try to push to GitHub and get this message:
```
...
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
remote: error: Trace: bcb1bf43ee0435af6ef20f998f7a02e1
remote: error: See http://git.io/iEPt8g for more information.
remote: error: File task_runners/bad_files/sample_project/node_modules/puppeteer/.local-chromium/linux-579032/chrome-linux/chrome is 201.89 MB; this exceeds GitHub's file size limit of 100.00 MB
To github.com:MyUser/javascript-notes.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'git@github.com:MyUser/javascript-notes.git'
```
You've tried adding the .gitignore, `git rm -r --cached .`, add then recommitting, but
the file keeps reappearing when you push.

Try:
1. create and checkout a new branch:
   ```console
   $ git checkout -b temp-newfiles
   Switched to a new branch 'temp-newfiles'
   ```
2. checkout master
3. reset hard to the most recent commit before adding the big files:
   ```console
   $ git reset --hard origin/master
   HEAD is now at a9664be Update grunt.md
   ```
4. run `git status` to see what files are lingering that aren't part of the commit we resetted to
   ```console
   $ git status
   On branch master
   Your branch is up to date with 'origin/master'.

   Untracked files:
   (use "git add <file>..." to include in what will be committed)

	task_runners/bad_files/

   nothing added to commit but untracked files present (use "git add" to track)
   ```
5. delete the offending files:
   ```console
   $ rm -rf task_runners/bad_files
   ```
6. run `git status` again to confirm clean:
   ```console
   $ git status
   On branch master
   Your branch is up to date with 'origin/master'.

   nothing to commit, working tree clean
   ```
7. checkout the `.gitignore` from the temp branch:
   ```console
   $ git checkout temp-newfiles -- .gitignore
   ```
8. Run `git status`. you should find that `.gitignore` has been copied into the working directory and staging 
   area<sup>[1](#footnote_2.12_01)</sup>:
   ```console
   $ git status
   On branch master
   Your branch is up to date with 'origin/master'.

   Changes to be committed:
   (use "git reset HEAD <file>..." to unstage)

	new file:   .gitignore

   ```
9. commit the `.gitignore`:
   ```console
   $ git commit -m "Add .gitignore"
   ```
10. checkout the offending files from temp branch (in this case, they were all in a subdirectory called 'bad_files'):
    ```console
    git checkout temp-newfiles -- task_runners/bad_files
    ```
11. run `git status`. You should find that the files from bad_files (except those covered by `.gitignore`) have been copied into 
    the working directory and staging area.
11. commit the changes
12. push the commit


<a name="s3">Understanding Git</a>
----------------------------------

### <a name="s3.1">Trees</a> ###
Behind the scenes, Git is a bunch of trees. To get a sense for this, let's examine the structure of a simple `.git` directory 
(i.e., not the working directory):

```
.git
├── COMMIT_EDITMSG                 [NOTE1]
├── HEAD                           [NOTE2]
├── ORIG_HEAD                      [NOTE3]
├── config                         [NOTE4]
├── description			   [NOTE5]
├── hooks                          [NOTE6]
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   └── update.sample
├── index                          [NOTE7]
├── info
│   └── exclude                    [NOTE8]
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           ├── hello
│           └── master
├── objects                        [NOTE9]
│   ├── 0b
│   │   └── 6e846af1c3ff99979436ef9c12ac5c1cacbb81
│   ├── 41
│   │   └── 6f7b0878d85f0f4ef53de23468e829c8e57f2a
│   ├── 63
│   │   └── d927bc1e2ff8f78a3eb831749949baf6344312
│   ├── 7d
│   │   └── c46d6e73e2ba58fe6f12095dd9d86b7b116e81
│   ├── a3
│   │   └── 45c8b2c3102a8043801637b9f2a95d36eb2202
│   ├── c1
│   │   └── 0d7fed83d6dc60440117ecb610654ac8d961a4
│   ├── e1
│   │   └── 627a63969a02945296a3425d702a84bf0a1557
│   ├── e3
│   │   └── 1070457fb6fe5df98048304b2bb0e6a8bcc514
│   ├── e6
│   │   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
│   ├── f0
│   │   └── cb3b45291697326597b008d0cdaa969abe8871
│   ├── f9
│   │   └── 3e3a1a1525fb5b91020da86e44810c87a2d7bc
│   ├── ff
│   │   └── 870e9f220613420e975f24ced7f40afef96de4
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   ├── hello                [NOTE10]
    │   └── master
    └── tags                     [NOTE11]
```

Notes:
1. The text of the most recent commit (not necessarily on the current branch)  
   Example:  
   `Add shebang`
2. A textual reference to the currently checked out HEAD  
   Example:  
   `ref: refs/heads/master`
3. The SHA hash of the original commit  
   Example:  
   `a345c8b2c3102a8043801637b9f2a95d36eb2202`
4. Project-specific configuration (overrides `~/.gitconfig`)  
   Example:
   ```INI
   [core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
   ```
5. Text description of the repo  
   Example:  
   `Unnamed repository; edit this file 'description' to name the repository.`
6. Various hook scripts
7. A compressed binary file representing the contents of the index (i.e., the staging area)
8. Like `.gitignore`. Difference is that `.gitignore` is a file in the repo and as such is available to anyone who clones the repo while
   `exclude` exists only in the local clone. [Accordingly, it makes sense to include here files that are only relevant to the person using 
   the local clone, e.g., files relating to the user's IDE](https://stackoverflow.com/questions/22906851/when-would-you-use-git-info-exclude-instead-of-gitignore-to-exclude-files)  
   Example:  
   ```shell
   # git ls-files --others --exclude-from=.git/info/exclude
   # Lines that start with '#' are comments.
   # For a project mostly in C, the following would be a good set of
   # exclude patterns (uncomment them if you want to use them):
   # *.[oa]
   # *~
   ```
9. Compressed and encoded files containing the actual data objects
10. The SHA hash of the corresponding commit  
   Example:  
   `e1627a63969a02945296a3425d702a84bf0a1557`
11. The SHA hash(?) of each corresponding commit

[We can browse the trees using `cat-file`](#s2.5).


### <a name="s3.2">Commits</a> ###
There are a variety of ways to refer to commits and ranges of commits. Most of the ways work in most of the inputs to git commands.

Type          | Example        | Explanation
--------------|----------------|------------
branchname    | `master`       | The name of any branch is simply an alias for the most recent commit on that branch.
tagname       | `v1`           | A tag alias is identical to a branch alias except in two ways: 1. tag aliases never change, while branch aliases always point to the newest commit on the branch; 2. a tag alias can contain a description of the tag
`HEAD`        | `HEAD`         | The currently checked out branch or commit is always called HEAD
hash          | `c82a22c`      | A commit can be referenced by its full hash id, or just as many characters that are required to make a unique reference in the repo
name^         | `v1^`          | The \[first] parent of a specified commit
name^n        | `v1^2`         | The second parent of a specified commit where the commit has multiple parents (e.g., merge commit)
name^^...     | `v1^^`         | Two ancestors back from the specified commit. `^` can be used in an arbitrarily long sequence
name~n        | `v1~2`         | The *nth* ancestor of the specified commit. In this example, it is equivalent to `v1^^`
name:path     | `git diff HEAD^1:Makefile HEAD^2:Makefile` | reference a particular file in a commit's content tree
name^{tree}   |                | Reference just the tree held by the commit, rather than the commit itself
name1..name2  | `v1~3..v1`     | All the commits reachable from name2 back to, but not including, name1. If either name1 or name2 is omitted, HEAD is used in its place
name1...name2 |                | Depends on the type of command. For commands like `log`, it refers to all the commits referenced by name1 or name2, but not by both. The result is a list of the unique commits in both branches.</br>For commands like `diff`, the range expressed is between name2 and the common ancestor of name1 and name2.
master..      |                | Equivalent to `master..HEAD`.
..master      |                | Equivalent to `HEAD..master`.
since/after   | `--since="2 weeks ago"` |
until/before  | `--until="1 week ago"` |
grep          | `--grep=<pattern>` |
committer/author | `--committer=<pattern>` |

See [Git From the Bottom Up][bottom-up]



<a name="s4">Other Useful Resources</a>
---------------------------------------

- [Git Flight Rules](https://github.com/k88hudson/git-flight-rules)  
  Inspired by NASA's flight rules for the space program, detailed step-by-step instructions for a wide range of git tasks
- [A Successful Git Branching Model](https://nvie.com/posts/a-successful-git-branching-model/)  
  An example of a branching strategy that can be effective for development projects of any size
- [Adding a folder from one repo to another](https://github.community/t5/How-to-use-Git-and-GitHub/Adding-a-folder-from-one-repo-to-another/m-p/5574#M1817)  
  Clear instructions for moving a folder from repoA to repoB while preserving history



<a name="s5">Glossary</a>
-------------------------

- [`blob`][bottom-up]  
  A file-like object stored in git's tree. The difference between a Git blob and a filesystem’s file is that a blob stores no metadata
  about its content. All such information is kept in the tree that holds the blob.
- [`commit`][bottom-up]  
  An snapshot/archive of what the project's *working tree* looked like at a particular point in time.
- `detached`  
  A state where HEAD is not pointing to any branch. This occurs when checking out a specific commit instead of a branch. When detached
  any commits will also not belong to a branch. Note that while in the detached state, you can create a new branch using 
  `git checkout -b <branch-name>` and it will add all the commits in the tree from the checkout that caused the detached state through
  to the current commit into this new branch.
- `HEAD`  
  A pointer to the currently checked-out branch or commit
- `master`  
  An arbitrary name that by convention refers to the main/default branch of the repo
- `origin`  
  An arbitrary name that by convention refers to the canonical version of the repo
- `remote`  
  A reference to a URL for a repo
- [`repository`][bottom-up]  
  A repository is a collection of *commits*. It also defines *HEAD*, and it contains a set of branches and tags, to identify certain 
  commits by name.
- [`working tree`][bottom-up]  
  A working tree is any directory on the filesystem which has a *repository* associated with it (typically indicated by the presence of 
  a subdirectory within it named `.git.`). It includes all the files and subdirectories in that directory.



<a name="s6">Footnotes</a>
--------------------------
1. <a name="footnote_2.12_01"> </a>* It seems that checkout behaves differently if you specify files. Any checkout will copy the files from the branch/commit into the working directory and the staging area. However, if one or more files are specified (i.e., checking out files not a branch/commit), the HEAD pointer isn't moved, which means that the working directory will match the staging area for files, but not match the tree as of
a particular commit (since we didn't check out a whole commit). Thus all the files that changed as a result of the checkout will appear
staged (ready to be committed). On the face of it, it looks like the difference in behaviour is due to what git stages, but the real
difference in behaviour is just that the HEAD pointer isn't being moved. This is obvious when you think about it: checking out just
part of a commit doesn't necessarily leave the working directory/staging area in a condition that matches either the previous HEAD
state or the stage of the commit we checked out from. Thus git can't simply move the HEAD pointer to a different commit (as it could
if we checkout out a branch or commit hash).




[link01]: https://github.com/Crossroadsman/TerminalTips/blob/master/BashEnvironmentVariables.md
[bottom-up]: http://ftp.newartisans.com/pub/git.from.bottom.up.pdf
