# Terminal & Shell Quick Reference

My cheat-sheet from Phase 00, Lesson 10 (Terminal & Shell), plus Mac-specific extras.
My shell is **zsh**; its config file is `~/.zshrc`.

## Shortcuts

| Keys | Does |
|---|---|
| Ctrl+R | Search command history (press again to go further back) |
| Ctrl+C | Stop the running command |
| Ctrl+Z / `fg` | Pause a command / bring it back |
| Ctrl+L | Clear the screen |
| Tab | Autocomplete file and folder names |
| Ctrl+A / Ctrl+E | Jump to start / end of line |
| Ctrl+U / Ctrl+W | Delete to start of line / delete previous word |
| `!!` | Repeat last command |
| `!$` | Last argument of previous command (`mkdir x` then `cd !$`) |
| `q` | Quit a pager (`git log`, `man`, `less`) |

## Redirects and pipes

| Symbol | Meaning |
|---|---|
| `>` | Output to file (**overwrite**) |
| `>>` | Output to file (append) |
| `>\|` | Force overwrite when `noclobber` is on |
| `2>` | Errors (stderr) to file |
| `2>&1` | Errors to same place as normal output |
| `\|` | Pipe: output of one command into the next |
| `tee file` | Show on screen AND save to file |

Two output streams: **stdout** (normal output) and **stderr** (errors).

```bash
grep "loss" train.log | wc -l                  # count matching lines
grep "loss" train.log | awk '{print $NF}'      # print last field of each line
tail -f train.log | grep --line-buffered loss  # follow a log live, filtered
python train.py > train.log 2>&1               # save everything to a log
python train.py 2>&1 | tee train.log           # watch it AND save it
```

## Background processes

| Method | Survives closing terminal? | Can reattach? |
|---|---|---|
| `cmd &` | No | No |
| `nohup cmd > log 2>&1 &` | Yes | No (read the log) |
| tmux | Yes | Yes |

```bash
jobs                  # list background jobs
fg %1 / kill %1       # foreground / kill job 1
ps aux | grep python  # find processes and their PIDs
kill <PID>            # stop a process by PID
```

## tmux

Prefix is **Ctrl+B**, then release, then the key.

| Action | Keys / command |
|---|---|
| New named session | `tmux new -s name` |
| Split top/bottom | Ctrl+B `"` |
| Split left/right | Ctrl+B `%` |
| Move between panes | Ctrl+B arrow |
| Detach (keeps running) | Ctrl+B `d` |
| Reattach | `tmux attach -t name` |
| List / kill | `tmux ls` / `tmux kill-session -t name` |

## Monitoring (on my Mac)

- `htop`: CPU/memory. F6 sort (by MEM% to find hogs), F5 tree, `/` search, F9 kill, `q` quit.
- GPU: Activity Monitor → Window → GPU History, or `sudo powermetrics --samplers gpu_power -i 1000`.
- `nvidia-smi` / `nvtop` are **NVIDIA-only**. They won't work on my Mac (it uses Apple MPS).

## SSH (for cloud GPU machines later)

```bash
ssh user@host                          # connect
scp file user@host:~/path/             # copy to server
scp user@host:~/file ./                # copy from server
rsync -avz ./data/ user@host:~/data/   # sync a folder (only changes)
ssh -L 8888:localhost:8888 user@host   # remote Jupyter at localhost:8888
```
`~/.ssh/config` nickname: `Host gpu` / `HostName ...` / `User ...` / `IdentityFile ...`, then `ssh gpu`.
Pattern: SSH in → run job inside tmux → detach → log off.

## Aliases

Lesson aliases are loaded from `~/.zshrc` with the **full path**:
```bash
source ~/projects/ai-engineering-from-scratch/phases/00-setup-and-tooling/10-terminal-and-shell/code/shell_aliases.sh
```
Useful on Mac: `ae` / `de` (venv on/off), `uvvenv`, `watchloss`, `diskuse`, `bigmodels`,
`tn` / `ta` / `tls` / `tk` (tmux), `psg <name>`, `newexp <name>`.
NVIDIA-only (don't work here): `gpu`, `gpuwatch`, `gpumem`, `gpuprocs`, `checkgpu`.

## Handy one-liners

```bash
du -sh phases/* | sort -rh | head -5                               # biggest folders
find . -name "*.py" -not -path "./.venv/*" -exec cat {} + | wc -l  # lines of code, excluding venv
df -h .                                                            # free disk space
diff <(grep acc a.log) <(grep acc b.log)                           # compare filtered logs
```
Lesson learned: `find .` includes `.venv` (millions of library lines), and
`xargs ... | tail -1` only shows the last batch's total. Question surprising numbers.

## Safety

- `setopt noclobber` in `~/.zshrc`: `>` refuses to overwrite existing files (use `>|` to force).
- Appending to config: always `>>`, never `>` (a single `>` wipes `~/.zshrc`).
- `rm` is permanent: no Trash, no undo. Use `rm -i` when unsure; beware `rm -rf`.
- `mv -n` / `cp -n`: never overwrite an existing destination.

## Mac extras

```bash
caffeinate -i python train.py   # keep Mac awake until the command finishes
open .                          # open current folder in Finder
open progress-tracker.html      # open a file in its default app
cat file | pbcopy               # copy to clipboard
pbpaste > file                  # paste from clipboard
```

## Figuring things out

```bash
type ae                   # alias, function, or program?
which python              # which python actually runs (venv or system?)
echo $PATH | tr ':' '\n'  # folders the shell searches for commands
man ls                    # full manual (q to quit)
ls --help                 # quick help
tldr tar                  # practical examples (brew install tldr)
```
"Command not found" right after installing? The installer added a folder to `PATH` in a
config file. Open a new terminal or `source ~/.zshrc`.
