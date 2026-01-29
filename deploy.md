<h1 class="card-title">How to deploy changes</h1>

Update and make commits often. (At least hourly if you are editing code.)
Append "nopr" or "No PR" if you are not yet ready to send a Pull Request.

## Using git.sh (Recommended)

Your Code CLI can write your PR comments if you run the "push" command below.

Or run your git.sh commands as follows in a separate terminal from your CLI.

Start a secure virtual session in your local webroot and give the git.sh files permission.  


	python3 -m venv env
	source env/bin/activate
	chmod +x git.sh
	chmod +x team/git.sh

Run ./git.sh in your webroot.  In the root git.sh is a pass-through to the team/git.sh file.

You can watch your webroot's file status change in Github Desktop to confirm updates are deployed.

	./git.sh push           # Push all repositories with changes (auto-pulls first)
	./git.sh pull           # Pull all repositories (webroot + submodules + extra repos)
	./git.sh push [name]    # Push specific repository (webroot, submodule, or extra repo)
	./git.sh pull [name]    # Pull specific repository

You probably won't need these since cmds above resolve detached heads for submodules that differ from their parent repos.

	./git.sh fix                    # Fix detached HEAD states
	./git.sh remotes                # Update remotes for current GitHub user

"push" also sends a Pull Request (PR) unless you include "nopr" 

### Wait to submit Pull Request:
- Add `nopr` to skip PR creation: `./git.sh push nopr`

<!-- 
Advanced option (not recommended for typical use):
- Add `overwrite-local` to let parent repository override your local commits: `./git.sh pull overwrite-local`

WARNING: `overwrite-local` will delete local work in submodules:
- Uncommitted changes: Permanently lost, no recovery possible
- Committed but unpushed changes: Can be recovered using git's reflog

To recover previously committed work that was overwritten locally:
	cd [submodule_name]
	git reflog                    # Find your lost commit hash
	git checkout [commit_hash]    # Restore your work
	git checkout -b recovery      # Create new branch to save it
-->

### Supported repositories:
- **Webroot**: webroot
- **Submodules**: Automatically detected from .gitmodules file
- **Extra Repos**: Automatically detected from .siterepos file

## Using Github Desktop

You can also use Github Desktop to choose a repo in the webroot using "File > Add Local Repository". 
Then submit a PR through the Github.com website.  
Or "./git.sh push" to send a PR automatically, but there won't be detailed comments from your CLI coding.
Or prompt "push" with your CLI to have a description of your changes included.  
Note: Sometimes CLIs gets confused and treat the team folder as the webroot.


IMPORTANT: If you're using Github Desktop to push, you'll still need to send the PR from within Github.com.


## Using your CLI with ./git.sh push

For the first usage, include extra guidance. Your push will also pull recent updates from others on Github.

	push using webroot/AGENTS.md with git.sh  


If you find "push" is asking for multiple approvals, your CLI isn't following its AGENTS.md instructions.
When AGENTS.md is followed, "push" uses the git.sh file to first pull, then update the webroot, submodules and forks.

	push

Additional deployment commands:

	push [folder name]  # Deploy a specific submodule or fork
	push submodules  # Deploy changes in all submodules
	push forks  # Deploy the extra forks added

"push" also sends a Pull Request (PR) unless you include "nopr" 


## Manual submodule refresh

You can refresh all your local submodules by running:

	git submodule foreach 'git pull origin main || git pull origin master'