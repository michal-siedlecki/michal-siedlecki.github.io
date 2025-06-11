## Git commands
Init repo:
`git init` – Start new repo

Track files:
`git add <file>` – Add to stage

Squash commits:
`git rebase -i HEAD~<n>` – Merge last n commits

Untrack file:
`git rm --cached <file>` – Remove from tracking

Add remote:
`git remote add origin <url>` – Link remote

Edit last commit:
`git commit --amend` – Change message

Delete local branch:
`git branch -d <branch>` – Remove local

Delete remote branch:
`git push origin --delete <branch>` – Remove remote

Fetch remote branches:
`git fetch --all` – Get all refs

Show remote branches
`git remote show origin`

Undo local commit:
- `git reset --soft HEAD~1` – Keep changes
- `git reset --hard HEAD~1` – Discard changes