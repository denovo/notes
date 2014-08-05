**Github workflow**

Commands

- git add --all - Add all files to commit
- git config --global alias.s "status -s" - Shorthand for Git Status
- git config --global alias.lg "log --oneline --decorate --graph --all" - Shorthand for Git Log
- git branch - List branches
- git branch -a  - List all branches even remote
- git branch -m "feature_branch" Create new feature branch and move to branch
- git checkout feature_branch - Checkout remote branch
- git config --global color.ui true - Add colour to git status

Do not commit directly to Master
Use pull requests on the repo
Use a feature branch for any changes - repo/feature_branch

When changes are ready to be merged in Master create a pull request to start a conversation with Team. Use @mentions to message team.

The pull request should be from the feature_branch into Master.
Use a single repo for collaboration
When you create a pull request you should be the person to Merge the request into Master - See video http://newrelic.com/resources/past-webinars

Merge should only be completed once the other devs have given a +1

Feature_branches should be between 1-5 days no longer and should be cxreated for single changes. Pull requests should be started to early in the process to start the communication

 
