### Q1. What’s the difference between ps, top, and pstree?
- ps → one-time snapshot of processes
- top → dynamic real-time monitoring
- pstree → shows hierarchical process tree

### Q2. How do you find the parent process of a given PID?
- Use ps -o ppid= -p <pid>
- Or pstree -p to see visually

### Q3. Difference between kill, pkill, and killall?
- kill <pid> → kills specific process ID
- pkill <name> → kills process by name (pattern match)
- killall <name> → kills all processes with that exact name

### Q4. Foreground vs Background process in Linux?
- Foreground = running directly in terminal (blocks shell).
- Background = runs behind the scenes (& operator or bg).
- Commands: jobs, fg, bg.

### Q5. Real DevOps Example?
- Pod stuck in Kubernetes.
- Check node processes:
  - ssh node
  - top
  - pstree -p
  - ps aux | grep containerd
  - kill -9 <pid>
- Restart stuck container.

### Technical Summary
- Linux process management is core skill for DevOps engineers.
- Tools:
  -   ○ ps (snapshot)
  -   ○ top/htop (real-time)
  -   ○ pstree (hierarchical view)
  -   ○ jobs/bg/fg (job control)
  -   ○ kill/pkill/killall (termination)
- Strong knowledge helps in debugging servers, Docker containers, Kubernetes pods.