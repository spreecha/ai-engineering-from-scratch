# Git Quick Reference

My personal cheat-sheet for the AI Engineering from Scratch course.
Run all commands from inside the repo folder (`~/projects/ai-engineering-from-scratch`).

## The mental model — four places your work lives

```
edit files  →  git add  →  git commit  →  git push
(working)      (staging)    (local repo)   (GitHub)
```

- **Working directory** – the actual files I edit.
- **Staging area** – the pile of changes I want in the next snapshot (`git add`).
- **Local repo** – saved history on my Mac (`git commit`).
- **Remote (GitHub)** – cloud copy (`git push` up, `git pull` down).

## One-time setup

```bash
git config --global user.name "Preecha S."
git config --global user.email "psinsawas@gmail.com"
git config --global --list          # check it
```

## Daily workflow

```bash
git status                          # where am I / what changed (run this constantly)
git add <file>                      # stage one file   (git add .  = stage everything)
git commit -m "Add perceptron"      # save a snapshot
git push                            # back up to my fork on GitHub
```

Commit messages: imperative present, ~50 chars. "Add…", "Fix…", "Refactor…".

## Look before you commit

```bash
git diff                            # unstaged changes, line by line
git diff --staged                   # what's about to be committed
git log --oneline                   # history, one line each  (press q to exit the pager)
```

## Undo / fix things

```bash
git restore <file>                  # discard un-staged edits to a file
git restore --staged <file>         # unstage a file (keeps the edit)
git commit --amend                  # fix the LAST commit's message or contents
git remote set-url <name> <url>     # fix a mistyped remote URL
```

Golden rule: never rewrite (`--amend`, `reset`, `push --force`) commits that are
already pushed and others might have. On my own un-pushed work it's fine.

## Branches — try things safely

```bash
git checkout -b experiment/idea     # create a branch AND switch to it
# ...edit, commit freely; main stays untouched...
git checkout main                   # switch back (no -b to switch to an existing branch)
git merge experiment/idea           # fold the branch's changes into main
```

## .gitignore — keep junk out of git

A file listing patterns git should never track. Useful for AI work:

```
*.pt
*.pth
*.safetensors
__pycache__/
.venv/
```

Note: `>` OVERWRITES a file completely. Use `>>` to append. If I clobber a
tracked file, `git restore <file>` brings the committed version back.

## Remotes — my fork vs the author's repo

```bash
git remote -v                       # list remotes
```

- **origin**  → github.com/spreecha/ai-engineering-from-scratch   (my fork — I push here)
- **upstream** → github.com/rohitg00/ai-engineering-from-scratch   (author — I pull updates here)

## Stay in sync with the course

```bash
git fetch upstream                  # download author's latest (doesn't touch my files)
git log --oneline HEAD..upstream/main   # list commits I don't have yet (nothing = up to date)
git merge upstream/main             # pull those updates into my main
```

Also: click **Watch** on the author's GitHub repo to get notified of updates,
or check my fork's page for a "N commits behind" banner.

## Commands I actually need for this course

| Command | When |
|---|---|
| `git clone` | Get a repo |
| `git add` + `git commit` | Save my work |
| `git push` | Back up to GitHub |
| `git checkout -b` | Try something without breaking main |
| `git log --oneline` | See what I've done |
| `git fetch upstream` + `git merge upstream/main` | Get course updates |

Don't need rebase, cherry-pick, or submodules for this course.
