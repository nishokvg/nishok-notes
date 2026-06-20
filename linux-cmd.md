# Linux / Bash Command Reference — Interview Prep

A systematic refresher organized by category, with the "extra knowledge" angle interviewers look for built in. Each section moves from common → less common, mirroring how live coding rounds escalate.

---

## 1. File Search & Filtering

| Task | Command | Notes / Gotchas |
|---|---|---|
| Find files by size | `find /path -type f -size +100M` | `+` = greater than, `-` = less than, no sign = exact |
| Find files by age (modified) | `find /path -mtime -1` | `-1` = modified within last 24h. `-mmin -60` for minute precision |
| Find files by age (accessed) | `find /path -atime -1` | Some filesystems mount with `noatime`, so this can be unreliable |
| Find by name (case-insensitive) | `find /path -iname "*.log"` | `-iname` vs `-name` |
| Find and execute on each | `find /path -name "*.tmp" -exec rm {} \;` | `\;` runs once per file; `+` batches into fewer calls |
| Find empty files/dirs | `find /path -empty` | |
| Find by permission | `find /path -perm 0777` | Security audit favorite |
| Find by owner | `find /path -user nishok` | |
| Fast filename search (indexed) | `locate filename` | Needs `updatedb` to be current — can be stale |
| Find file type | `file /path/to/file` | Identifies binary/text/script regardless of extension |

---

## 2. Process Management

| Task | Command | Notes / Gotchas |
|---|---|---|
| List all processes | `ps aux` | `a` = all users, `u` = user-oriented, `x` = no tty |
| Sort by memory | `ps aux --sort=-%mem \| head` | `-` prefix = descending |
| Sort by CPU | `ps aux --sort=-%cpu \| head` | |
| Interactive process viewer | `top` / `htop` | `htop` is friendlier if installed |
| Find PID by port | `sudo lsof -i :8080` or `sudo ss -tulpn \| grep 8080` | `ss` is the modern, faster replacement for `netstat` |
| Find PID by name | `pgrep nginx` | |
| Kill by PID | `kill -9 1234` | `-9` = SIGKILL (force). `-15` = SIGTERM (graceful, default) |
| Kill by name pattern | `pkill -f worker` | `-f` matches full command line, not just process name |
| Kill all matching | `killall nginx` | Matches exact process name only |
| Run in background | `command &` | |
| Detach from terminal | `nohup command &` | Survives terminal/SSH session close |
| List background jobs | `jobs` | |
| Bring job to foreground | `fg %1` | |
| Adjust process priority | `nice -n 10 command` / `renice -n 5 -p 1234` | Lower = higher priority (-20 to 19) |

---

## 3. Networking

| Task | Command | Notes / Gotchas |
|---|---|---|
| Show listening ports | `ss -tulpn` | `t`=tcp `u`=udp `l`=listening `p`=process `n`=numeric |
| Legacy equivalent | `netstat -tulpn` | Deprecated on many distros, `ss` preferred now |
| Test connectivity | `ping -c 4 host` | `-c` limits count so it doesn't run forever |
| DNS lookup | `dig domain.com` or `nslookup domain.com` | `dig` gives more detail (TTL, all record types) |
| Trace network path | `traceroute host` or `mtr host` | `mtr` combines ping + traceroute, live updating |
| HTTP request | `curl -I https://site.com` | `-I` = headers only, fast health check |
| Download file | `wget https://url/file` | |
| Show routing table | `ip route` | Modern replacement for `route -n` |
| Show interfaces | `ip a` | Modern replacement for `ifconfig` |
| Check open connections | `ss -tan` | Shows established/listening TCP state |
| Firewall rules (basic) | `sudo iptables -L -n` | Or `ufw status` on Ubuntu's simplified frontend |

---

## 4. Disk & Filesystem

| Task | Command | Notes / Gotchas |
|---|---|---|
| Disk space (filesystem level) | `df -h` | `-h` = human readable |
| Directory size | `du -sh /path` | `-s` = summary, no per-file breakdown |
| Subdirectory sizes, sorted | `du -sh /path/*/ \| sort -rh` | Sorted largest first |
| Top 10 largest files | `find / -type f -exec du -h {} + \| sort -rh \| head -10` | Common "what's eating disk" question |
| List block devices | `lsblk` | Shows partitions, mount points |
| Mount a filesystem | `mount /dev/sdb1 /mnt/data` | |
| Check filesystem type | `lsblk -f` or `blkid` | |
| Inode usage | `df -i` | Different from disk space — can run out of inodes with full disk space remaining |

---

## 5. System Info

| Task | Command | Notes / Gotchas |
|---|---|---|
| Uptime + load average | `uptime` | Load avg (1/5/15 min) — compare against `nproc` (core count) to judge overload |
| CPU core count | `nproc` | |
| Kernel/OS version | `uname -a` | |
| Distro version | `cat /etc/os-release` | |
| Memory usage | `free -h` | Watch "available" not just "free" — Linux caches aggressively |
| Live resource stats | `vmstat 1` | Refreshes every 1 sec, good for spotting trends |
| Kernel ring buffer (boot/hw logs) | `dmesg \| tail` | First place to check for OOM kills, disk errors |
| Hardware info | `lscpu`, `lsusb`, `lspci` | |

---

## 6. Permissions & Users

| Task | Command | Notes / Gotchas |
|---|---|---|
| Change permissions | `chmod 755 file` | Read=4, Write=2, Execute=1 — owner/group/other |
| Change permissions (symbolic) | `chmod u+x file` | Add execute for owner only |
| Change owner | `chown user:group file` | |
| Recursive chmod/chown | `chmod -R 644 /path` | Careful — easy to lock yourself out of dirs (need 755 for dirs, not 644) |
| Add a user | `sudo useradd -m username` | `-m` creates home directory |
| Add user to group | `sudo usermod -aG groupname username` | `-aG` appends, plain `-G` overwrites all groups |
| Check current user's groups | `groups` or `id` | |
| Switch user | `su - username` | `-` loads their environment too |
| Run as root | `sudo command` | |
| Edit sudoers safely | `sudo visudo` | Never edit `/etc/sudoers` directly — syntax errors can lock out all sudo access |

---

## 7. Text Processing & Logs

| Task | Command | Notes / Gotchas |
|---|---|---|
| Live tail with filter | `tail -f file.log \| grep "ERROR"` | Add `--line-buffered` to grep — without it, output can lag due to pipe buffering |
| Show first/last N lines | `head -n 20 file` / `tail -n 20 file` | |
| Search recursively | `grep -r "pattern" /path` | `-i` for case-insensitive, `-n` to show line numbers |
| Extract column | `cut -d',' -f2 file.csv` | `-d` = delimiter, `-f` = field number |
| Sort lines | `sort file` | `-n` numeric, `-r` reverse, `-u` unique |
| Remove duplicate lines | `sort file \| uniq` | `uniq` only removes *adjacent* duplicates — must sort first |
| Count occurrences | `grep -c "pattern" file` | Or `sort \| uniq -c` for frequency counts |
| In-place text replace | `sed -i 's/old/new/g' file` | `-i` edits in place; omit for dry-run to stdout first |
| Column-based processing | `awk '{print $1, $3}' file` | Powerful for structured log parsing |
| systemd service logs | `journalctl -u servicename -f` | `-f` follows live, like `tail -f` for systemd services |
| Logs since a time | `journalctl --since "1 hour ago"` | |

---

## 8. Package & Service Management

| Task | Command | Notes / Gotchas |
|---|---|---|
| Install package (Debian/Ubuntu) | `sudo apt install pkg` | `apt update` first to refresh package index |
| Install package (RHEL/CentOS) | `sudo dnf install pkg` | `yum` is the legacy name, `dnf` is current |
| Check installed version | `dpkg -l \| grep pkg` (Debian) | Or `rpm -qa \| grep pkg` (RHEL) |
| Start/stop/restart a service | `sudo systemctl restart nginx` | |
| Enable on boot | `sudo systemctl enable nginx` | |
| Check service status | `systemctl status nginx` | |
| List all services | `systemctl list-units --type=service` | |

---

## 9. Compression & Archives

| Task | Command | Notes / Gotchas |
|---|---|---|
| Create tar archive | `tar -cvf archive.tar /path` | `c`=create `v`=verbose `f`=filename |
| Create compressed tar | `tar -czvf archive.tar.gz /path` | `z` = gzip |
| Extract tar | `tar -xvf archive.tar` | `x` = extract |
| Extract compressed tar | `tar -xzvf archive.tar.gz` | |
| Zip a directory | `zip -r archive.zip /path` | |
| Unzip | `unzip archive.zip` | |

---

## 10. SSH & Remote Access

| Task | Command | Notes / Gotchas |
|---|---|---|
| SSH config for specific key | See block below | `IdentitiesOnly yes` prevents SSH from trying default keys first |
| Generate SSH keypair | `ssh-keygen -t ed25519 -C "comment"` | `ed25519` is modern/preferred over `rsa` now |
| Copy file to remote | `scp file user@host:/path` | |
| Sync directory to remote | `rsync -avz /local/ user@host:/remote/` | `-a`=archive mode `v`=verbose `z`=compress. Resumable, unlike scp |
| Run command on remote | `ssh user@host "command"` | |
| SSH config example: | ```Host test-app1\n    HostName test-app1\n    User deploy\n    IdentityFile ~/.ssh/id_rsa_test_app1\n    IdentitiesOnly yes``` | |

---

## Interview Tactics for This Format

Based on the prep doc Apple sent you, here's the pattern to practice:

1. **Answer fast, then add one "extra knowledge" sentence.** They explicitly said they want this — alternatives, edge cases, gotchas. Every table row above has one built in for exactly this reason.
2. **If you don't know the exact flag, say the approach out loud.** "I'd use `find` with a size filter, let me check the exact flag syntax" is a perfectly good answer — they said this explicitly in the doc.
3. **Narrate while you type.** Live coding interviews reward thinking-out-loud. Silence while typing reads as uncertain even when you're not.
4. **Use of Google/AI tools is allowed** — but say it out loud: "Let me quickly check the exact syntax for this" rather than silently switching tabs.

---

## Quick Self-Test (cover the right column, answer, then check)

- Find all `.log` files under `/var` modified in the last 7 days
- Show top 5 CPU-consuming processes
- Find the PID listening on port 443 and kill it gracefully
- Show disk usage for `/var/log`, human readable
- Tail the last 50 lines of a file and follow live updates
- Recursively change all files in a folder to be readable/writable by owner only
- Check which systemd service is using the most resources right now
