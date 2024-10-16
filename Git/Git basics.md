Git commands are divided into porcelain and plumbing. Porcelain is what you as a user use for absolute majority of the time.

Examples of porcelain commands
`git status`
`git add`
`git commit`
`git pull`
`git push`
`git log`

Examples of pluming commands
`git apply`
`git commit-tree`
`git hash-object`

You set your identity with the following commands
```terminal
git config --add --global user.name "Name you want displayed"
git config --add --global user.email "email@example.com"
```

To make a git repository you navigate to the root of your project and type `git init`.

A file in a git repo can be in several states. The important states to know are:
- Untracked - not being tracked by Git
- Staged - marked for inclusion in next commit.
- Committed - saved to the repository's history.

You make an untracked file staged by using `git add {files}`. 

A commit is a state/snapshot of the repo at any given time. It's how Git keeps track of changes. It comes with a message that can be used to describe what has changed. `git commit -m {message}`. Every commit got a unique hash, which is a long string of characters i.e. `41af1a86054e27957364d8c0be6d524a40f1f5f8 `. For convenience you can refer to any commit in Git by using the 7 first characters, so in the example above it would be `41af1a8`. Git uses SHA-1 for the hashes and they're generated based on the content changes but also the commit message, the authors name and email, the date and time and previous commit hashes so you won't risk having two of the same hash.

`git log` can be used to check past commits and their hashes.

#### Plumbing
All data in a Git repo is stored in the .git directory. Git is made up of objects and they're stored in `.git/objects`. A commit is just a type of object.

If you go to the objects directory you will see directories which are named with two characters and if you have a commit you'll see it will have a directory here. Go into one of those and you'll see that a file will have the rest of the hash from the commit as a name. It just contains raw bytes but you can check the contents with the cat-file command in git. `git cat-file -p {hash}

If you run the cat-file for a commit you'll see that it will give you a **tree** value. Trees are git's way of storing a directory. **blob** is git's way of storing a file. So when checking the object for the commit you won't see the actual changes but the tree's hash and that is where the changes are stored.

If you run a cat-file command on the tree hash it will give you the blob hash. Run cat-file on that and you'll finally see the contents you added.

The commits aren't stored in a vacuum, git stores an entire snapshot of files on a per-commit level. Git uses compression and packs files to store and deduplicates files that are the same across different commits to make it so it doesn't take up too much space.

When it comes to branches, they are just named pointers to commits. A new branch is just a new pointer to a specific commit. The commit that it points to is called the tip of the branch. Pointers are very lightweight so git is again very efficient. They can be found in `.git/refs/heads`.

#### Merge and rebase

When merging git will 
1. Find the merge base commit or best common ancestor of the two branches. 
2. Replaying the changes from the branch getting merged into starting from the best common ancestor into a new commit.
3. Replaying the changes from the branch merging onto starting from best common ancestor.
4. Record the result as a new commit.
5. The new commit will now have two parents.

The simplest merge is the fast-forward merge. If the branch that's merging onto the branch got the same commits git only needs to point to the branch that merging onto. It doesn't even get a merge commit as it's basically just a normal commit.

When rebasing you don't get a merge commit either. Git does that by replaying commits from the new branch on top of the branch you rebasing.

Example

```
Pre rebase
A - B - C    main
   \
    D - E    feature_branch

Post rebase
A - B - C         main
         \
          D - E   feature_branch
```

When rebasing a branch the following happens:
1. Checkout the latest commit of the branch used to rebase
2. Replay one commit at a time from the branch you're on onto the branch used to rebase.
3. Update the branch you're on to point to the last replayed commit.
4. The rebase doesn't affect the other branch while the current branch has all of the changes from the other branch.

Advantage of merge is that it preserve the true history of a project but the history is also harder to read due to all merge commits.

Avoid rebasing a public branch onto anything else. Other devs can have it checked out and changed history may cause problems.

#### Reset

The git reset command can be used to undo changes.
A soft reset is useful if you want to go back to a previous commit but keep all of your changes. Committed changes will be made uncommitted and staged instead and those that were staged will stay staged.

`git reset --soft {hash}`

If you don't want to keep the changes you can do a hard reset. If you use hard all the changes will be gone and you'll be back at the commit you referenced in the hash.

`git reset --hard {hash}`

When you use this command you need to be aware you can't recover anything that's gone it's undoing the commits.

#### Remote

When using git another repo is called a remote. Standard convention is that if the remote is the main place for the code you should name it **origin**,

`git remote add {name} {uri}`

To get the contents of the remote you can use the fetch command. If you don't have the contents and you run the command you will download the objects in the .git directory.It's metadata so if you run something like git log on a repo that you only fetched it will give you an error message.

`git fetch`

To get the commits, just like the local branch you need to merge. If you don't have anything on the branch it will just be a fast-forward merge.

`git merge {remote}/{branch}`

To get your code to the remote you use the push command.

`git push origin main`

You can also push a local branch to a remote branch with another name.

`git push origin {local branch}:{remote branch}`

To delete a remote branch you can push an empty branch to it.

`git push origin :{remote branch}`

To get the actual file changes from a remote you use the command pull. If you run a pull without specify anything it will pull the current branch from your current branch from the remote.

`git pull {remote} {branch}`

#### Misc

Renaming a branch
`git branch -m {old name} {new name}`

Create a new branch and move to it
`git switch -c {branch name}`

Show log entries on one line
`git log --oneline`

Representation of commit history as ASCII
`git log --oneline --graph --all`

Change most recent commit message
`git commit --amend -m {message}`

Delete branch
`git branch -d {branch name}`

Base new branch on old commit
`git switch -c {branch name} {hash for commit}`

Rebase on pull instead of merge
`git config --global pull.rebase true`

