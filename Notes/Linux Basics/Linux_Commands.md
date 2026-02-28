# Linux Commands - Comprehensive Notes

## Essentials & patterns

**Command shape**
- General form: `command [options] [args...]`
- Get help:
  - `man <cmd>` (authoritative manual when installed)
  - `<cmd> --help` (quick option summary)
  - `apropos <keyword>` / `man -k <keyword>` (search man-page descriptions)
- Identify what you’re running:
  - `type <name>` (builtin vs alias vs function vs external command)
  - `which <name>` / `command -v <name>` (path resolution; `command -v` is more portable)

**Core “everywhere” utilities**
- Many everyday file + text tools come from GNU coreutils (e.g., `cat`, `cp`, `mv`, `rm`, `ls`, `mkdir`, `sort`, `wc`) and are described as “standard programs for text and file manipulation” in its manual. [geeksforgeeks](https://www.geeksforgeeks.org/linux-unix/basic-linux-commands/)

**Exit codes (critical for scripting)**
- `$?` holds the previous command’s exit status.
- Convention: `0` = success, non‑zero = failure.

**Pipes and redirection**
- Pipe stdout → stdin: `cmd1 | cmd2`
- Redirect stdout:
  - Overwrite: `cmd > out.txt`
  - Append: `cmd >> out.txt`
- Redirect stderr:
  - `cmd 2> err.txt`
  - Combine: `cmd > all.txt 2>&1`
- Read from a file: `cmd < in.txt`
- Send both stdout+stderr through a pipe (common): `cmd 2>&1 | less`

**Conditionals and sequencing**
- Run sequentially: `cmd1 ; cmd2`
- Run only if success: `cmd1 && cmd2`
- Run only if failure: `cmd1 || cmd2`
- Background: `cmd &`, then inspect with `jobs`, bring back with `fg`

**Quoting and expansion**
- Single quotes: `'...'` (no expansion; safest for literals/regex)
- Double quotes: `"..."` (variable expansion happens; spaces preserved)
- No quotes: word-splitting + globbing can bite you (`*`, `?`, `[]`)
- Prefer: `"$var"` almost always.

**Sudo basics**
- `sudo <cmd>` runs a command as root (or another user with `-u user`).
- Safer pattern for “redirect needs root”:
  - Good: `echo 'line' | sudo tee -a /etc/file`
  - Bad: `sudo echo 'line' >> /etc/file` (redirection is done by your shell, not sudo)

***

## Files & directories

**Navigation**
- `pwd` — print working directory
- `ls` — list directory contents
  - Common flags: `-l` (long), `-a` (include dotfiles), `-h` (human-readable sizes, typically with `-l`)
- `cd <dir>` — change directory
  - `cd ..` parent, `cd -` previous directory

**Creating and removing**
- `mkdir dir` / `mkdir -p a/b/c` (create parents as needed)
- `touch file` (create empty file or update timestamp)
- `rm file` (remove file), `rm -r dir` (recursive), `rm -rf dir` (force + recursive; dangerous)
- `rmdir dir` (only removes empty directories)

**Copying and moving**
- `cp src dst`
  - `cp -r dir1 dir2` (recursive)
  - `cp -a src dst` (archive mode; preserves attributes as much as possible)
- `mv old new` (rename or move)

**Viewing file content**
- `cat file` (small files)
- `less file` (large files; `/` search, `n` next, `q` quit)
- `head -n 20 file`, `tail -n 50 file`
- Follow logs: `tail -f /var/log/...`

**Inspecting file metadata**
- `stat file` (permissions, timestamps, inode, etc.)
- `file thing` (guess file type by content)
- `du -sh path` (disk usage for a path)
- `df -h` (free space by filesystem)

**Links**
- Hard link: `ln existing newname`
- Symlink: `ln -s target linkname`
- Resolve symlink: `readlink link` / `readlink -f link` (canonical path; varies by system)

**Find and filter files**
- `find <where> <tests> <actions>`
  - By name: `find . -name "*.c"`
  - By type: `find . -type d`
  - By size: `find . -size +100M`
  - By mtime: `find . -mtime -7` (last 7 days)
  - Execute: `find . -name "*.tmp" -delete`
  - Safer exec: `find . -name "*.py" -exec python -m py_compile {} \;`
- `locate name` (fast DB-based search; may require `updatedb`)

**Permissions and ownership**
- View: `ls -l`
- Change mode:
  - Symbolic: `chmod u+rwx,g+rx,o-r file`
  - Numeric: `chmod 755 script.sh` (rwx r-x r-x)
- Change owner/group:
  - `chown user file`
  - `chgrp group file`
  - Recursive: add `-R` (use carefully)
- Default permissions mask: `umask` (affects new files/dirs)

**Archiving**
- Tar:
  - Create: `tar -czf out.tar.gz dir/`
  - Extract: `tar -xzf out.tar.gz`
  - List: `tar -tzf out.tar.gz`
- Zip:
  - `zip -r out.zip dir/`
  - `unzip out.zip`

**Fast sync / copy**
- `rsync -avh src/ dst/` (archive + verbose + human-readable)
- With deletion (mirror): `rsync -avh --delete src/ dst/` (be sure `src/` is correct)

***

## Text processing & pipelines

**Counting and simple transforms**
- `wc -l file` (lines), `wc -w` (words), `wc -c` (bytes)
- `sort file` / `sort -n` (numeric) / `sort -r` (reverse)
- `uniq` (usually after sorting):
  - `sort file | uniq`
  - Count duplicates: `sort file | uniq -c | sort -nr`
- `cut -d, -f1,3 file.csv` (select fields)
- `tr 'a-z' 'A-Z'` (character translation)
- `paste a.txt b.txt` (merge lines side-by-side)

**Search**
- `grep 'pattern' file`
  - Recursive: `grep -R 'pattern' dir/`
  - Line numbers: `grep -n`
  - Invert match: `grep -v`
  - Extended regex: `grep -E 'foo|bar'`
- Practical examples:
  - Errors in a log: `grep -n 'error' app.log`
  - Filter processes: `ps aux | grep -i python` (better: `pgrep -a python`)

**Stream editing**
- `sed` (good for substitutions):
  - Replace first match per line: `sed 's/old/new/' file`
  - Replace all matches: `sed 's/old/new/g' file`
  - In-place (GNU sed): `sed -i 's/old/new/g' file`
- `awk` (great for columns/records):
  - Print column 1: `awk '{print $1}' file`
  - CSV-ish (simple): `awk -F, '{print $1,$3}' file.csv`

**Comparing**
- `diff -u a.txt b.txt` (unified diff)
- `comm file1 file2` (requires both sorted; shows common/unique lines)

**Pipelines that show output and save**
- `somecmd | tee out.txt` (also prints to terminal)
- Append: `somecmd | tee -a out.txt`

**One useful “log triage” pipeline**
```bash
journalctl -u nginx --since "today" | grep -i error | tail -n 50
```

***

## System, networking & services

**Processes**
- `ps aux` (snapshot)
- `top` / `htop` (live view; `htop` may need install)
- `pgrep name` / `pkill name`
- Send signals:
  - `kill PID` (SIGTERM by default)
  - `kill -9 PID` (SIGKILL; last resort)
- Background/foreground:
  - `cmd &`, `jobs`, `fg %1`, `bg %1`

**System info**
- `uname -a` (kernel + system)
- `uptime` (load averages)
- CPU/mem:
  - `free -h`
  - `lscpu` (CPU details; availability varies)
- Disks:
  - `lsblk` (block devices)
  - `mount`, `umount`
  - `dmesg` (kernel ring buffer)

**Logs (systemd-based systems)**
- `journalctl`:
  - Follow: `journalctl -f`
  - By unit: `journalctl -u ssh -f`

**Services with systemd**
- `systemctl status <unit>` shows runtime status and recent logs for a unit (human-readable output). [freecodecamp](https://www.freecodecamp.org/news/the-linux-commands-handbook/)
- Start/stop/restart:
  - `sudo systemctl start <unit>` starts (activates) units. [freecodecamp](https://www.freecodecamp.org/news/the-linux-commands-handbook/)
  - `sudo systemctl stop <unit>` stops (deactivates) units. [freecodecamp](https://www.freecodecamp.org/news/the-linux-commands-handbook/)
  - `sudo systemctl restart <unit>` stops then starts the unit. [freecodecamp](https://www.freecodecamp.org/news/the-linux-commands-handbook/)
- Enable at boot:
  - `sudo systemctl enable <unit>` enables a unit (creates symlinks) and does **not** automatically start it unless combined with `--now`. [freecodecamp](https://www.freecodecamp.org/news/the-linux-commands-handbook/)
  - `sudo systemctl enable --now <unit>` enables and starts immediately. [freecodecamp](https://www.freecodecamp.org/news/the-linux-commands-handbook/)

**Networking (modern Linux)**
- `ip` is used to “show / manipulate routing, network devices, interfaces and tunnels.” [unidel.edu](https://unidel.edu.ng/focelibrary/books/Operating%20System%20Concepts%20and%20Basic%20Linux%20Commands%20(%20PDFDrive%20).pdf)
- Common `ip` usage:
  - Show addresses: `ip addr` [unidel.edu](https://unidel.edu.ng/focelibrary/books/Operating%20System%20Concepts%20and%20Basic%20Linux%20Commands%20(%20PDFDrive%20).pdf)
  - Bring link up/down: `ip link set <iface> up|down`  [unidel.edu](https://unidel.edu.ng/focelibrary/books/Operating%20System%20Concepts%20and%20Basic%20Linux%20Commands%20(%20PDFDrive%20).pdf)
  - Routes: `ip route` [unidel.edu](https://unidel.edu.ng/focelibrary/books/Operating%20System%20Concepts%20and%20Basic%20Linux%20Commands%20(%20PDFDrive%20).pdf)
- Sockets:
  - `ss -tulpn` (list listening TCP/UDP with processes; may need sudo for names)
- Connectivity:
  - `ping -c 4 8.8.8.8`
  - `traceroute example.com` (may need install)
  - DNS: `dig example.com` / `nslookup example.com`
- HTTP:
  - `curl -I https://example.com` (headers)
  - `curl -L -o file https://...` (follow redirects, save)
  - `wget URL` (alternative)

**Remote access**
- SSH: `ssh user@host`
- Copy:
  - `scp file user@host:/path/`
  - Better for directories/resume: `rsync -avh -e ssh dir/ user@host:/path/`

***

## Packaging, storage & automation

**Packages (depends on distro)**
- Debian/Ubuntu (`apt`):
  - Update index: `sudo apt update`
  - Install: `sudo apt install <pkg>`
  - Remove: `sudo apt remove <pkg>` (or `purge`)
  - Search: `apt search <term>`
  - Show installed files: `dpkg -L <pkg>`
- Fedora/RHEL family:
  - `sudo dnf install <pkg>`, `sudo dnf remove <pkg>`, `dnf search <term>`
- Arch:
  - `sudo pacman -S <pkg>`, `sudo pacman -R <pkg>`, `pacman -Ss <term>`

**Compression utilities**
- `gzip file` / `gunzip file.gz`
- `xz -z file` / `unxz file.xz`

**Cron (scheduled jobs)**
- Edit user crontab: `crontab -e`
- List: `crontab -l`
- Example (run daily at 2:30 AM):
  - `30 2 * * * /path/to/script.sh >> /var/log/myscript.log 2>&1`

**Bash scripting essentials (practical defaults)**
- Start scripts with:
```bash
#!/usr/bin/env bash
set -euo pipefail
```
- Read args safely: `"$1"`, `"$@"`
- If you parse filenames, prefer null-delimited flows:
```bash
find . -type f -name "*.log" -print0 | xargs -0 -n 1 echo
```
