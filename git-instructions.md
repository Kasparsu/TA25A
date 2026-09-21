# Git Starter: Build the `ta25alearngit` Repo From Scratch

Goal: learn the core Git workflow by reproducing a small repo locally, then publish it on GitHub over HTTPS.

End state matches: https://github.com/Kasparsu/ta25alearngit
- one file: `readme.md`
- branches: `main`, `feature`, `testing`
- tag: `v0.1.0`
- one merge commit
- `feature` left unmerged at the end (that is what makes the history fork visible)

The lessons this covers, if you want to rewatch or reread:
- [24.03 — Git basics: init, add, commit, remote](26-03-24.md)
- [31.03 — clone, log, fetch vs pull, branches](26-03-31.md)
- [07.04 — merge conflicts, rebase, interactive rebase, amend](26-04-07.md)
- [14.04 — tags, semver, commit messages, checkout, reset](26-04-14.md)

---

## Part 0 — Install + One-Time Setup

### Install Git
- Windows: `winget install -i Git.Git`
  - `-i` opens the interactive installer so you can pick options instead of silent defaults.
  - During install, choose:
    - **"Add a Git Bash Profile to Windows Terminal"** (checkbox in the components step)
    - **Default editor: Visual Studio Code** (dropdown)
    - **Default branch name: `main`** (radio button)
  - Already installed with defaults? No need to reinstall, fix it with the config commands below.
- Arch / CachyOS: `sudo pacman -S git`
- Debian / Ubuntu: `sudo apt install git`
- macOS: `brew install git`

Verify:
```bash
git --version
```

### Configure identity (once per machine)
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
Use the same email as your GitHub account, otherwise your commits will not link to your profile.

### Configure defaults
```bash
git config --global init.defaultBranch main          # new repos start on 'main'
git config --global core.editor "code --wait"        # VSCode as default editor
```
`code --wait` makes Git wait until you close the VSCode tab before continuing. It is needed for commit, merge and interactive rebase messages.

Check config:
```bash
git config --list
```

---

## Part 1 — Build the Repo Locally

Open a terminal in the directory where you keep your projects.

### Lesson 1 — `init`, `status`, `add`, `commit`, `log`

```bash
mkdir ta25alearngit
cd ta25alearngit
git init
```
`git init` creates a hidden `.git/` directory. The folder is now a Git repo on branch `main`.

```bash
printf "# Learngit\n" > readme.md
git status                       # untracked file
git add readme.md                # stage it
git status                       # now staged
git commit -m "Init commit"
git log                          # full log
git log --oneline                # short
```

Two more commits so there is some history:
```bash
printf -- "- main line 1\n" >> readme.md
git add readme.md
git commit -m "Add main line 1"

printf -- "- main line 2\n" >> readme.md
git add readme.md
git commit -m "Add main line 2"
```

The command you will use constantly to see the shape of history:
```bash
git log --oneline --decorate --graph --all
```

### Lesson 2 — Branches

```bash
git branch feature        # create branch at current commit
git branch                # list branches, * marks current
git switch feature        # switch to it
```
Create and switch in one go: `git switch -c feature`.

`git checkout` does the same thing and you will see it in older tutorials. `switch` and `restore` were added later to split `checkout`'s two unrelated jobs apart. Use `switch` for branches.

Commit on `feature`:
```bash
printf -- "- feature line 1\n" >> readme.md
git add readme.md
git commit -m "Add feature line 1"
```

`main` and `feature` now point at different commits:
```bash
git log --oneline --decorate --graph --all
```

### Lesson 3 — Clean merge (no conflict)

`main` has not moved since the branch point, so merging is a **fast-forward**: Git just slides the pointer forward and makes no merge commit.

```bash
git switch main
git merge feature
git log --oneline --decorate --graph --all
```

To force a real merge commit even when a fast-forward is possible: `git merge --no-ff feature`.

### Lesson 4 — Merge with a conflict, resolved in VSCode

Make both branches change the **same area** so Git cannot decide for you.

```bash
# on main
printf -- "- main line 3\n" >> readme.md
git add readme.md
git commit -m "Add main line 3"

git switch feature
printf -- "- feature line 2\n" >> readme.md
git add readme.md
git commit -m "Add feature line 2"
```

Trigger it:
```bash
git switch main
git merge feature
```
Git reports `CONFLICT (content): Merge conflict in readme.md` and stops. `git status` shows the file as "both modified".

```bash
code readme.md
```
VSCode shows the markers and inline buttons:
```
<<<<<<< HEAD
- main line 3
=======
- feature line 2
>>>>>>> feature
```
Click **Accept Current Change**, **Accept Incoming Change**, **Accept Both**, or edit by hand. Delete the `<<<<<<<`, `=======` and `>>>>>>>` lines. Save.

Finish it:
```bash
git add readme.md
git status                # "All conflicts fixed but you are still merging"
git commit                # editor opens with the default merge message, save and close
git log --oneline --decorate --graph --all
```
This is the merge commit in the end state. Stuck halfway? `git merge --abort` puts everything back.

### Lesson 5 — Rebase

Rebase replays your commits **on top of** another branch instead of merging them. The result is linear, with no merge commit.

Diverge again:
```bash
git switch main
printf -- "- main line 4\n" >> readme.md
git add readme.md
git commit -m "Add main line 4"

git switch feature
printf -- "- feature line 3\n" >> readme.md
git add readme.md
git commit -m "Add feature line 3"
```

Rebase:
```bash
git switch feature
git rebase main
git log --oneline --decorate --graph --all
```
`feature`'s commits now sit after `main`'s tip.

If a rebase hits a conflict it pauses at that commit. Fix the file the same way as a merge, then:
```bash
git add readme.md
git rebase --continue
```
Also available: `git rebase --skip` to drop that commit, `git rebase --abort` to roll the whole thing back.

> **Rule of thumb:** never rebase commits you have already pushed to a shared branch. Rewriting public history forces everyone else to recover. Rebase your local work *before* pushing.

### Lesson 6 — Interactive rebase: `reword`, `fixup`, `squash`

Make some deliberately messy commits:
```bash
git switch feature
printf -- "- wip 1\n" >> readme.md
git add readme.md
git commit -m "Add big feture"          # typo on purpose

printf -- "- wip 2\n" >> readme.md
git add readme.md
git commit -m "fix typo"                # tiny fix that belongs to the previous commit

printf -- "- wip 3\n" >> readme.md
git add readme.md
git commit -m "more wip"                # also part of the same work
```

Clean them up:
```bash
git rebase -i HEAD~3
```
VSCode opens a todo list, **oldest commit at the top**:
```
pick a1b2c3d Add big feture
pick d4e5f6a fix typo
pick 7g8h9i0 more wip
```
Change the commands to:
```
reword a1b2c3d Add big feture
fixup  d4e5f6a fix typo
squash 7g8h9i0 more wip
```
- `reword` — keep the commit, edit its message
- `fixup` — merge into the commit above, **throw away** this message
- `squash` — merge into the commit above, **keep both** messages so you can combine them

Save and close. Git then stops at `reword` (fix the typo to "Add big feature"), applies `fixup` silently, and stops at `squash` with both messages so you can write one clean message.

```bash
git log --oneline --decorate --graph --all
```
Three messy commits are now one clean commit.

### Lesson 7 — `commit --amend`

For fixing the commit you just made, when you have not pushed it yet:
```bash
git commit --amend -m "A better message"      # change only the message
```
Forgot a file?
```bash
git add forgotten.txt
git commit --amend --no-edit                  # add it to the last commit, keep the message
```
Amend creates a *new* commit and throws the old one away, so the same warning as rebase applies: not on anything already pushed.

### Lesson 8 — Writing commit messages

Git's own template says it best: **summarize the change in around 50 characters or less**. Write it in the imperative, as if completing the sentence "this commit will ...":

```
Add feature line 3          good
added feature line 3        wrong tense
stuff                       says nothing
Add some cool line to readme for testing and to show class that this is very cool. bla bla bla    far too long
```

Keep commits **atomic**: one commit does one thing. If the message needs the word "and", it is probably two commits.

### Lesson 9 — Tags and semantic versioning

```bash
git switch main
git tag v0.1.0                            # lightweight tag
git tag -a v0.1.0 -m "First release"      # annotated tag, preferred for releases
git tag                                   # list
git show v0.1.0                           # what it points at
```

Semantic versioning is `MAJOR.MINOR.PATCH`:
- **PATCH** (0.1.**0** to 0.1.1) — bug fix, nothing else changes
- **MINOR** (0.**1**.0 to 0.2.0) — new feature, old code still works
- **MAJOR** (**0**.1.0 to 1.0.0) — breaking change

Full spec: https://semver.org/

Delete a tag locally: `git tag -d v0.1.0`. On the remote: `git push origin :refs/tags/v0.1.0`.

### Lesson 10 — Looking at and undoing history

```bash
git checkout <hash>        # jump to an old commit to look around ("detached HEAD")
git switch -                # back to where you were
git switch -c testing <hash>  # or turn that point into a new branch
```
Detached HEAD is not an error. It means you are on a commit rather than a branch. Commits made there belong to no branch and are easy to lose, so make a branch if you want to keep them.

```bash
git restore <file>          # discard unstaged changes in one file
git restore --staged <file> # unstage but keep the changes
git reset --soft HEAD~1     # undo last commit, keep changes staged
git reset --hard HEAD~1     # undo last commit AND throw the changes away
```
`--hard` is the one that actually deletes work. Lost something? `git reflog` lists every position HEAD has been in, and `git reset --hard <sha>` jumps back to one.

### Lesson 11 — The `testing` branch

The end state has a third branch. Make it from `main` and put a couple of commits on it:
```bash
git switch main
git switch -c testing
printf -- "- some code\n" >> readme.md
git add readme.md
git commit -m "Add code"

printf -- "- some cool code\n" >> readme.md
git add readme.md
git commit -m "Add cool code"
```

Leave `feature` unmerged so the fork stays visible in the history graph.

### Final inspection
```bash
git log --all --oneline --decorate --graph
git branch
git tag
```

---

## Part 2 — Publish to GitHub over HTTPS

### Step 1. Create the repo on GitHub
1. Go to https://github.com/new
2. Repository name: `ta25alearngit`
3. Public or private, your choice.
4. **Do not** tick README, .gitignore or license. You are pushing an existing repo, and those would collide.
5. Create, then copy the HTTPS URL: `https://github.com/<your-user>/ta25alearngit.git`

### Step 2. Personal Access Token
GitHub does not accept your account password over HTTPS. You need a token.

1. GitHub, top-right avatar, **Settings**
2. **Developer settings** (bottom of the left sidebar)
3. **Personal access tokens** > **Tokens (classic)** > **Generate new token (classic)**
4. Note: `git cli`. Expiration: your choice. Scope: tick **`repo`**.
5. Generate and **copy it now**, it is not shown again.

Optional, so you do not paste it on every push:
```bash
git config --global credential.helper store   # saved in plaintext on disk
# or
git config --global credential.helper cache   # memory only, 15 minutes
```

### Step 3. Add the remote and push
```bash
git remote add origin https://github.com/<your-user>/ta25alearngit.git
git remote -v                     # verify

git push -u origin main
git push -u origin feature
git push -u origin testing
git push origin v0.1.0
```
Or everything at once:
```bash
git push -u origin --all
git push origin --tags
```
When it asks for credentials: username is your GitHub username, password is **the token**.

### Step 4. Check it on GitHub
- `readme.md` has all the lines
- the branch dropdown lists `main`, `feature` and `testing`
- the tags page shows `v0.1.0`
- **Insights > Network** draws the branch/merge graph

Then hand the repository link in through the Teams assignment.

---

## Cheat Sheet

```bash
git status                  # what changed
git diff                    # unstaged changes
git diff --staged           # staged changes
git add <file>              # stage one file
git add .                   # stage everything here
git commit -m "message"     # commit what is staged
git log --oneline           # short history
git log --all --graph --oneline --decorate   # the full picture
git push                    # upload
git pull                    # download + merge
```

### Getting an existing repo
```bash
git clone <url>             # copy a repo from GitHub, remote is set up for you
git fetch                   # download new commits, change nothing locally
git pull                    # fetch AND merge into your branch
```
`fetch` is the safe one: it lets you look before anything touches your working files. `pull` is `fetch` plus `merge` in one step.

### Branching
```bash
git branch                       # list
git switch -c feature/login      # create + switch
git switch main                  # switch back
git merge feature/login          # merge into current branch
git branch -d feature/login      # delete a merged branch
git push -u origin feature/login # publish it
```

### `.gitignore`
Create `.gitignore` in the repo root:
```
node_modules/
*.log
.env
.DS_Store
build/
```
```bash
git add .gitignore
git commit -m "Add gitignore"
```

---

## References

- Pro Git book, free: https://git-scm.com/book
- GitHub docs: https://docs.github.com/get-started
- Command reference: https://git-scm.com/docs
- Cheat sheet PDF: https://education.github.com/git-cheat-sheet-education.pdf
- Practise branching visually: https://learngitbranching.js.org/
- Good commit messages: https://cbea.ms/git-commit/
- Semantic versioning: https://semver.org/
- HTTPS auth with a token: https://docs.github.com/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
