# `pstree` Command in Linux

## 🔹 What is `pstree`?
- `pstree` = Process Tree
- It shows all running processes on a Linux system in a tree-like structure, starting from the root (`init` or `systemd`).
Easier to visualize parent-child relationships between processes compared to `ps` or `top`.
---
### 🔹 Basic Usage

```bash
pstree
```
Example output:
```
systemd─┬─NetworkManager───2*[{NetworkManager}]
        ├─sshd───sshd───bash───pstree
        └─nginx───nginx
```
👉 Meaning:
- `systemd` is the root parent process.
- `sshd` (SSH daemon) started by systemd.
- A user logged in with SSH → created `bash` → which is running `pstree`.
- `nginx` master process → created worker processes.

### 🔹 Useful Options
Show process IDs (PIDs):
```
pstree -p
```
Output:
```
systemd(1)─┬─sshd(1234)─┬─bash(5678)─┬─pstree(9876)
```
Show usernames:
```
pstree -u
```
Output:
```
systemd─┬─sshd(sahil)───bash(sahil)───pstree(sahil)
```
Compact view (group same processes):
```
pstree -c
```
Example:
```
nginx─┬─nginx
      └─nginx
```
Limit depth of tree (e.g., only 2 levels):
```
pstree -l 2
```
## 🔹 Why is `pstree` Important for DevOps?

- Helps you debug which process spawned another.
- Useful in Kubernetes/Docker troubleshooting:
 - You might see how `docker` daemon runs containers.
 - Inside a pod, you can see init process → main app → child workers.
- In CI/CD pipelines, useful to check if background jobs are running correctly.

### 🔹 Example in Real Life
🔹 SSH into a Kubernetes node:
```
ssh user@node-ip
pstree -p
```
You might see something like:
```
systemd(1)─┬─containerd(500)─┬─docker(1200)─┬─nginx(2400)─┬─nginx(2401)
```
👉 Meaning:

- `systemd` started `containerd` (container runtime).
- `containerd` is managing `docker`.
- Docker started an NGINX container, which spawned worker processes.

### 🔹 Summary
- `pstree` = visualize running processes in a tree.
- Options: `-p` (PID), `-u` (username), `-c` (compact), `-l N` (depth).
- Super useful for debugging parent-child process relationships in Linux, Docker, Kubernetes nodes.
