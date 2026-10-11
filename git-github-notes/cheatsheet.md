# Git & GitHub Cheatsheet

Source: Codédex Git & GitHub cheatsheet. Notes for Git and GitHub (mid-May 2026).

## Git Version

```sh
git --version
```

## Initializing a Repository

```sh
git init
git remote add origin <repository_url>
git branch -M main
```

## Add One File / All Files to Next Commit

```sh
git add index.html
git add .
```

## Git Commit

```sh
git commit -m 'Fix mobile bug'
```

## Check Repo Status

```sh
git status
```

## First Push

```sh
git push -u origin main
```

## All Other Pushes

```sh
git push
```

## Clone Repository

```sh
git clone https://github.com/ellie/hi.git
```

## Create a New Branch

```sh
git branch <new-branch-name>
```

## Switch Branches

```sh
git switch <new-branch-name>
```

## Create and Switch Branches

```sh
git checkout -b <new-branch-name>
```

## Updating Repo to Recent Changes

```sh
git checkout main
git pull
```

## Merge Branch to Current Branch

```sh
git merge <branch-name>
```

## My additions

- **The loop is always status → diff → add → commit → push.** `git status` tells you what changed, `git diff` shows exactly how, then stage and commit. Never `git add .` blind: review the diff first so junk (passwords, huge binaries, OS files) never enters history.
- **Write commits for future-you.** Format `Area: what changed`, e.g. `Complete: Sequential Circuits + update Log`. Then `git log --grep="Complete:"` lists only completions, while `wip:` commits hold the messy middle. This repo runs on exactly that split.
- **One commit, one idea.** A commit that updates notes *and* renames a folder *and* fixes the README is three commits smashed together and can't be reverted cleanly. Small commits revert cleanly.
- **Branch per task, main stays clean.** New notes or risky edits go on a branch, merge when done. `git pull` on main before branching so you never build on stale history.
- **Never commit secrets.** API keys, tokens, and passwords don't belong in git: once pushed, they're in history forever even if you delete them later. Use `.gitignore` for env files and check `git status` for accidents before every commit.
- **Undo ladder (least to most destructive):** unstage with `git restore --staged <file>`, drop file edits with `git restore <file>`, undo last commit but keep work with `git reset --soft HEAD~1`. Anything with `--hard` destroys work: only after a clean `git status` review.
- **Push is publishing.** Local commits are private drafts you can rewrite. Pushed commits are public: prefer new commits over rewriting history on shared branches.

---
[Index](README.md)
