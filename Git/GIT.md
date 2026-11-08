


CONFIGURATION

system				-	applies to all users, e.g. /etc/gitconfig
global				-	applies to all repos for this user, e.g. ~/.gitconfig
local					-	applies to this repo for this user, e.g. /path/to/repo/.git/config
attributes				-	.gitattributes in the repo folder


Credentials

.git/config
~/.git-credentials



REPOSITORY

  Working Tree

	In sync with the local file system


  Staging Tree / Index															[‘git ls-files -s’ will show the staging tree]
	
	tracks the working tree. keeps a list of files and their hashes							
	if working tree changes then staging tree will know by comparing hashes 



   Local Commit Tree:																		[references are at ./.git/refs/heads]
															tag = v1.1.3	
															(merge commit)		refs/heads/master
 	MASTER				-		56b39...a1cf 	---	2bfaf...cbc0	---	3e72b...e23d	---	938e9...3703
															|	|			  |
												refs/heads/dev		|	|			  |
	DEV					-		56b39...a1cf	---	8189c....f655	--------	|		    	  |
																|		    	  |
																|		    	  |	HEAD (.git/HEAD)	[HEAD - locally the hotfix branch is checked out and is at 912ab...67yc5, the latest local commit in this case]
																|		    	  |	refs/heads/hotfix
	HOTFIX				-									3e72b...e23d	    	  |_______	 912ab...67yc5

																			refs/heads/feature
	FEATURE				-						2bfaf...cbc0	---	e2ae2...da1aa	---	911ab....1c1c



  Remote Commit Tree: (lets say its called ‘origin’ in this case)											[references are at ./.git/refs/remotes/origin]
														refs/remotes/origin/HEAD
														refs/remotes/origin/master
	ORIGIN/MASTER		-		COMMIT1 		COMMIT2		COMMIT3	
	
								refs/remotes/origin/dev
	ORIGIN/DEV			-		56b39...a1cf

																	refs/remotes/origin/hotfix
	ORIGIN/HOTFIX		-		COMMIT1			COMMIT2		COMMIT3		COMMIT4

												refs/remotes/origin/feature
	ORIGIN/FEATURE		-		COMMIT1			COMMIT2





 
THE .git DIRECTORY

.git/
	branches/
	hooks/
	info/
	logs/
	objects/
	refs/
		heads/							-		the references for the local commit tree
			branch1
			branch2
			...
		remotes/
			origin/						-		the name of the remote, can be robert, richard, etc.
				HEAD
				master
		tags/
	config
	FETCH_HEAD
	HEAD
	index
	ORIG_HEAD
	packed-refs



CONCEPTS

Branch
a lightweight movable pointer to a commit, the pointer has a name


Objects in git
  commit						-	a pointer to the snapshot of data, authors name, authors email, message, pointer to commirt OR pointers to commits that came before this commit 
  blobs (files)
  trees (directories of files)
  tags



Commit
saved changes at a particular point of time

What does a commit have:
 - unique hash
 - metadata 
	- author
	- date
	- reference to the parent which is the previous commit



References (refs)
a pointer to a commit or to another reference 

type of references
	- local HEAD											-	.git/HEAD
	- remote HEAD											-	.git/refs/remote/origin/HEAD			(where your local instance believe that the remote HEAD is pointing to, gets updated with git fetch/pull)
	- heads (branch references)
			local repo refs are at 								-	./.git/refs/heads
			remote repo refs for a remote repo called origin are at 		-	./.git/refs/remotes/origin
	- tags




The HEAD reference
 - a label noting where you are in a commit tree
 - a reference to the current commit (latest) on the current branch
 - there can only be a single HEAD at any given time (excluding git worktree)

Location			-	.git/HEAD
reflog			-	shows how the HEAD has been moving in the repository
detached HEAD	-	HEAD is not pointing to the latest commit in the commit history, it is pointing to a prior commit
					to get out of a detached HEAD state, 
						run ‘git reflog’ make a note of the commit you want to go to 
						‘git stash’ to stash any work 
						and then run ‘git reset --hard <commit-id>’
						‘git stash pop’ to recover stashed work


Reference log		-	a listing of commits and information about how the commit came to be



revision
"revision" refers to an id you can use as a parameter to reference an object in git
an object could be a commit, blob, tree or tag.
there are various ways to spell object names
 - <rev>:<path>, e.g. HEAD:README, :README, master:./README


File
is a file


Fork
Make a copy of an upstream repo on your GitHub account.
Merging your changes with upstream are via a pull request


Clone
Make a copy of an upstream repo on your local computer
updates to upstream are via push if you have write permissions


Pull Request
a way to check in code to a repo in which you don’t have write request. 
The owner of the repo will review your request and make a decision
OR
a process to review code in a branch before merging it into another branch






ACTIONS


Checkout
moves HEAD to a commit in a currently active or different branch

-b/--track,  will create a new branch from the currently active commit and make that branch active



Merge
merging is Git’s way of putting a forked history back together again.
‘git merge’ command lets you take the independent lines of development created by git branch and integrate them into a single branch


Reset
a way of synchronizing with a commit history
reset moves the head reference of your local branch to the commit or reference you tell it to



Rebase



Fast forward
the branch that is the target (could be your local branch) has its head moved to the latest commit in the source branch (could be the remote branch)
the history from a commit in the source branch is linear to a commit in the target branch.


When is a fast forward not possible?
Fast-forwards don’t happen in situations where changes have been made in the source branch and the target branch such that their commits are no longer linear.



CONFLICTS

A conflict means that the branch you are merging into has a newer version of a file than the one your changes are based on

When ‘git merge’
ours 		-	refers to the branch you are merging into 
thiers	-	refers to the branch you are merging


when ‘git rebase’





COMMANDS

View tracking branches
git branch -vv


Get full info on remote
git remote show <remote-name>
e.g. git remote show origin


View local commit logs
git log


Show files in staging/index
git ls-files -s


Diff commands
git diff [ -- filename filename .... ]			-	between a/index and b/working tree 
git diff HEAD [ -- filename filename .... ]		-	between a/last commit and b/working tree 
git diff --cached	[ -- filename filename .... ]		-	between a/last commit and b/index


Show tags in sorted order
git tag --sort=v:refname


Show information about a tag
git show <tag>


Show a commit
git show <commit_hash>


Show the reference log
git reflog


Revision listing commands
git rev-list --all							-	get all the revisions
git rev-list --children $(git rev-parse HEAD)	-	get all the children commits of the commit that HEAD is pointing to


Revision parsiong commands
git rev-parse HEAD				-	get the commit that HEAD is pointing to
git rev-pars origin/master			-	get the commit that origin/master is pointing to in our local repo
git rev-parse HEAD^				-	get the commit one before the commit that HEAD is pointing to
git rev-parse HEAD@{1}			-	get the commit one before the commit that HEAD is pointing to
git rev-parse HEAD@{2}			-	get the commit two before the commit that HEAD is pointing to
git rev-parse master:./pom.xml		-	this revision is a file, when i ran it i did not get a commit, i think i got a hash of the file, i am not sure 
git rev-parse --show-toplevel		-	get the absolute path for the top level directory of this git repo


Grep for text in all commits
git rev-list --all | xargs git grep “regexp”


Find out who changed a line in a file and when
git blame -L <lineno>[, <lineno>] <filename>
  

Show a file from the previous commit
git show HEAD~1:<file> 




SOLUTIONS

Get out of a detached HEAD state:
run ‘git reflog’ make a note of the commit you want to go to 
‘git stash’ to stash any work 
and then run ‘git reset --hard <commit-id>’
‘git stash pop’ to recover stashed work
