# Deep Dive into `kill` Command

## 📌 What is the `kill` Command?
The `kill` command in Linux/Unix is used to send signals to processes.
Despite its name, it doesn’t always mean "terminate" — it can send different types of signals to a process (stop, continue, reload, kill, etc.).

👉 By default, `kill` sends the SIGTERM (signal 15), which politely asks a process to terminate.
If the process ignores it, you can use SIGKILL (signal 9) to forcefully kill it.

---

### ⚡ Basic Usage

```bash
kill [options] <PID>
```
`PID` = Process ID (unique number assigned to each process).

Example:
```bash
kill 1234
```
This sends `SIGTERM` to process with PID `1234`.

### ⚡ Commonly Used Signals
| Signal Name | Number | Meaning                               | Example           |
| ----------- | ------ | ------------------------------------- | ----------------- |
| SIGTERM     | 15     | Graceful termination (default)        | `kill 1234`       |
| SIGKILL     | 9      | Force kill (cannot be caught/ignored) | `kill -9 1234`    |
| SIGSTOP     | 19     | Stop/suspend process (like pause)     | `kill -STOP 1234` |
| SIGCONT     | 18     | Resume a stopped process              | `kill -CONT 1234` |
| SIGHUP      | 1      | Reload config / restart process       | `kill -HUP 1234`  |

### 🎯 Why Use kill?

- Stop a program that’s misbehaving.
- Restart daemons by sending SIGHUP.
- Pause/resume long-running tasks.  
- Clean up stuck processes.
