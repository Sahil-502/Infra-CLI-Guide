# `ps aux` Command in Linux

## 🔹 What is `ps aux`?
- `ps` = process status.
- It shows information about running processes in Linux.
---
### 🔹 What does `ps aux` mean?
- `ps aux` is a classic BSD-style command.
- `a` → show processes for all users (not just yours).
- `u` → show the process owner/username.
- `x` → show processes that are not attached to a terminal (like daemons, background services).

👉 So ps aux = Show all processes, with owner, including system/background ones.

```
ps aux
```
Example output:
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 168512  9100 ?        Ss   Aug16   0:05 /sbin/init
root       723  0.1  1.2 372564 50200 ?        Ssl  Aug16   3:15 /usr/bin/dockerd
sahil     1502  2.0  5.3 982456 210000 pts/0   Sl+  10:00   1:05 /usr/bin/python3 script.py
```
👉 Understanding the Columns
| Column  | Meaning                                                                   |
| ------- | ------------------------------------------------------------------------- |
| USER    | Process owner                                                             |
| PID     | Process ID                                                                |
| %CPU    | CPU usage                                                                 |
| %MEM    | Memory usage                                                              |
| VSZ     | Virtual memory size                                                       |
| RSS     | Resident Set Size (actual RAM used)                                       |
| TTY     | Terminal associated (or `?` if none)                                      |
| STAT    | Process status (e.g., `S`=sleeping, `R`=running, `Z`=zombie, `T`=stopped) |
| START   | When process started                                                      |
| TIME    | Total CPU time used                                                       |
| COMMAND | The command/program running                                               |

### 🔹 Useful Variations
Show only processes of a specific user:
```
ps aux | grep sahil
```
Sort by CPU usage:
```
ps aux --sort=-%cpu | head -n 10
```
Sort by memory usage:
```
ps aux --sort=-%mem | head -n 10
```
Find process by name:
```
ps aux | grep nginx

```


## 🔹 WDevOps Use Cases
### 🔹 Debugging high CPU/memory:
🔹 If a container or app is consuming too much, check with:
```
ps aux --sort=-%mem | head -n 5
```
Check background daemons:
Services like `docker`, `kubelet`, `sshd` often don’t have a terminal (shown with `?`).

### 🔹 Combine with kill:
```
ps aux | grep python
kill -9 <PID>
```
---
### 🔹 Summary
- `ps aux` = most common way to view all running processes in Linux.
- Columns show user, PID, CPU, memory, and command.
- Essential for debugging performance issues, stuck apps, container processes.
