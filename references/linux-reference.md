# Linux for AI Quick Reference

My cheat-sheet from Phase 00, Lesson 11 (Linux for AI), practiced on my Azure Ubuntu 24.04 VM,
plus extras. Linux servers use **bash**; the config file is `~/.bashrc` (not `~/.zshrc`).

## Connecting

```bash
chmod 400 ~/.ssh/your-key.pem              # SSH refuses keys others can read
ssh -i ~/.ssh/your-key.pem psw@VM-IP       # connect with a key file
ssh azvm                                   # with a nickname in ~/.ssh/config
exit                                       # log off (or Ctrl+D); VM keeps running
```

`~/.ssh/config` on my Mac:
```
Host azvm
    HostName VM-IP
    User psw
    IdentityFile ~/.ssh/your-key.pem   # only if using a key file
```

**Azure:** Stop the VM from the **portal** ("Stopped (deallocated)") to stop compute billing.
`sudo shutdown now` from inside still bills. If SSH hangs after my home IP changes,
update the port 22 rule in the VM's Networking settings.

## First look at a new machine

```bash
whoami; hostname          # who and where am I
cat /etc/os-release       # which Linux / Ubuntu version
echo $SHELL               # which shell
nproc; free -h; df -h /   # CPU cores, memory, disk space
cat /proc/cpuinfo | grep "model name" | head -1
```

## File system layout

| Folder | What's there |
|---|---|
| `/home/psw` (`~`) | my files; almost all work happens here |
| `/tmp` | temporary, cleared on reboot |
| `/usr` | installed programs and libraries |
| `/etc` | system config files |
| `/var/log` | logs (syslog, auth.log, kern.log, dpkg.log) |
| `/mnt`, `/media` | extra drives |
| `/proc`, `/sys` | virtual files: live kernel and hardware info |

## Essential commands

```bash
pwd; ls -la; cd dir; cd ~; cd ..
mkdir -p a/b/c
cp file copy; cp -r dir/ dir-copy/
mv old new; rm file; rm -rf dir/     # rm is permanent
touch a.txt                          # create empty file
cat, head -20, tail -20, tail -f, less (q to quit)
grep -r "text" .; grep -i "text" file
find . -name "*.py"; find . -name "*.ckpt" -size +1G
```

## Reading `ls -l`

```
-rwxr-xr--  1 psw psw 2048 Sep 24 10:00 train.sh
│└┬┘└┬┘└┬┘     └┬┘ └┬┘
│ │  │  │       │   group
│ │  │  │       owner
│ │  │  others
│ │  group
│ owner
type: - file, d directory
```
r = read, w = write, x = execute (for a folder: allowed to enter).

## Permissions

| Number | Result | Use |
|---|---|---|
| 755 | rwxr-xr-x | scripts, programs |
| 644 | rw-r--r-- | normal files |
| 600 | rw------- | private files |
| 400 | r-------- | SSH keys |

r=4, w=2, x=1, add them up per group (owner, group, others).

```bash
chmod +x script.sh        # make runnable, then ./script.sh
chmod 644 file
sudo chown psw:psw file   # change owner:group
```
"Permission denied" is almost always a missing `x` (chmod +x) or a file owned by
someone else (sudo / chown). `./` is needed because the current folder isn't in PATH.

## sudo

```bash
sudo command              # run one command as root
sudo !!                   # rerun the last command with sudo
echo "x" | sudo tee -a /etc/file   # sudo echo >> fails: the redirect runs as me
```
Use sudo only when needed.

## Packages (apt)

```bash
sudo apt update                  # refresh the catalog (does NOT upgrade anything)
apt list --upgradable
sudo apt upgrade                 # actually upgrade installed packages
apt search htop; apt show htop
sudo apt install -y htop         # -y = don't ask
sudo apt remove / purge htop     # purge also deletes config
sudo apt autoremove              # remove unneeded dependencies
```

Fresh GPU box:
```bash
sudo apt update && sudo apt install -y build-essential git curl wget tmux htop unzip python3-venv
```

Gotchas:
- "Could not get lock": background auto-updates are running. Wait and retry. Don't delete the lock.
- `pip install` outside a venv is blocked on Ubuntu 24.04 (externally-managed-environment).
  Use `python3 -m venv .venv && source .venv/bin/activate`, or uv.
- `brew` = Mac, no sudo. `apt` = Ubuntu, needs sudo (installs into root-owned /usr).

## Processes

```bash
htop                      # F6 sort (MEM%), H hide threads, F5 tree, / search, q quit
pgrep -a python           # find processes by name, with PIDs
kill PID                  # polite stop: lets it save/clean up
kill -9 PID               # force stop: no cleanup (can corrupt checkpoints)
```
htop columns: RES = real memory (the one that matters), VIRT = ignore,
S = state (S sleeping, R running, Z zombie), NI 19 = lowest priority.
Load average vs core count: below the number of cores = not overloaded.

## Services (systemd)

```bash
systemctl list-units --type=service --state=running
systemctl status ssh
sudo systemctl start | stop | restart | enable <name>
journalctl -u ssh -n 20       # last 20 log lines of a service (-f to follow)
```
Never stop `ssh` (or `xrdp`) while connected through it: you lock yourself out.

## Security checks

```bash
journalctl -u ssh --since "7 days ago" | grep "Accepted"   # successful logins (all should be me)
journalctl -u ssh --since today | grep -c "Failed password" # count failed attempts
journalctl -u ssh --since today | grep "Failed password" | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head
```
Bots scan every public IP. Best fix: limit port 22 (and 3389) to my IP in the Azure network rules.

## Disk space

```bash
df -h /                                   # real disk usage
du -sh ~/.cache                           # Hugging Face models land in ~/.cache/huggingface
du -sh ~/.cache/* 2>/dev/null | sort -rh | head
sudo du -xh --max-depth=1 / 2>/dev/null | sort -hr | head   # -x: stay on one filesystem
pip cache purge; sudo apt clean           # common cleanups
```
- `2>/dev/null` hides errors, including the one telling me I made a typo.
- Without `-x`, `du` counts `/snap` mounts (and `/boot`) and overstates usage vs `df`.
- Mac equivalent of `--max-depth=1` is `-d 1`.

## tmux on a remote box

```bash
tmux new -s train     # start, run the job inside
# Ctrl+B, d           # detach (never `exit` inside tmux: that stops the job)
tmux ls; tmux attach -t train
```
Pattern: SSH in → start job in tmux → detach → log off → come back later.

## File transfer (run on my Mac)

```bash
scp file azvm:~/dir/                   # Mac → VM
scp azvm:~/dir/file ~/Downloads/       # VM → Mac
rsync -avz --progress src/ azvm:~/dest/   # sync a folder; re-run only sends changes
```
- rsync resumes big transfers; scp starts over. Prefer rsync for large data.
- Trailing slash: `src/` copies the contents; `src` copies the folder itself.

## macOS vs Linux gotchas

| macOS | Linux |
|---|---|
| zsh, `~/.zshrc` | bash, `~/.bashrc` |
| `brew install` | `sudo apt install` |
| `open`, `pbcopy`, `pbpaste` | not available on a remote box |
| `du -d 1` | `du --max-depth=1` |
| `sed -i '' 's/a/b/' f` | `sed -i 's/a/b/' f` |
| case-insensitive: `Model.py` = `model.py` | case-sensitive: two different files |

Classic bug: `import model` works on my Mac when the file is `Model.py`, then fails on Linux.

## Extras

**"Killed" with no error** = out of memory (OOM killer):
```bash
sudo dmesg -T | grep -i -E "out of memory|killed process"
```

**Add swap** (safety cushion on a small VM with no swap; lasts until reboot):
```bash
sudo fallocate -l 4G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile && free -h
```
Permanent = a line in `/etc/fstab`. Back it up first (`sudo cp /etc/fstab /etc/fstab.bak`);
a typo there can stop the VM from booting.

**Archives:**
```bash
tar -xzf data.tar.gz [-C dir/]   # extract (x) gzip (z) file (f)
tar -tzf data.tar.gz | head      # list contents first (t)
tar -czf backup.tar.gz folder/   # create (c)
unzip file.zip
```

**Environment variables** (models location, GPU choice, API keys):
```bash
export HF_HOME=~/models-cache                          # this session only
echo 'export HF_HOME=~/models-cache' >> ~/.bashrc      # permanent
source ~/.bashrc
```
Never put API keys in code or commit them to git.

**SSH keys** (no more typing passwords), on my Mac:
```bash
ssh-keygen -t ed25519     # creates ~/.ssh/id_ed25519 (private, never share) + .pub
ssh-copy-id azvm          # installs the public key on the VM
```

**Editing on the server:** nano (Ctrl+O save, Ctrl+X exit, Ctrl+W search).
For real work: VS Code's Remote-SSH extension (Editor Setup lesson).
