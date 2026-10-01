
# Linux Performance & Splunk-on-EC2 Cheat Sheets

## 1. The 60-second checklist

Run these ten in order when a box feels slow; each takes a few seconds and together they show which resource is stressed. Ctrl+C stops anything that repeats. If a command is missing, install the toolset: `sudo apt install sysstat` covers `mpstat`, `pidstat`, `iostat` and `sar`.
### 1. uptime: is load rising or falling?
Shows load averages for the last 1, 5 and 15 minutes. Compare them to your core count (`nproc`). 1-min above 15-min means load is rising.

```
uptime
```
### 2. dmesg: any kernel errors?
The kernel's own log. Look for OOM kills ("Out of memory: Killed process"), disk I/O errors and filesystems remounted read-only. `-T` gives readable timestamps.

```
dmesg -T | tail -n 50
```
### 3. vmstat: system-wide view
One line per second, 20 samples. Ignore the first line (averages since boot). `r` above core count means CPU queueing, `b` above 0 means tasks blocked on I/O, non-zero `si`/`so` means swapping, high `wa` means disk wait, high `st` means the hypervisor is taking your CPU.

```
vmstat -SM 1 20
```
### 4. mpstat: per-core balance
Shows each core separately. One core near 100% with the rest idle points to a single-threaded bottleneck. Also check `%iowait`, `%steal` and `%sys`.

```
mpstat -P ALL 1
```

### 5. pidstat: which process uses the CPU?
Per-process CPU, split into `%usr` (own code) and `%system` (kernel work on its behalf). Rolling output, so you see who is busy right now. Unlike `top`, it scrolls and can be pasted into a ticket.
```
pidstat 1
```

### 6. iostat: disk load and latency
`-s` short format, `-x` extended stats, `-z` hides idle devices. Watch `await` (ms per request: sustained high values mean a slow or capped disk) and `%util`. On NVMe and EBS, `%util` near 100 can overstate saturation because they serve requests in parallel, so trust `await` more.
```
iostat -sxz 1
```

### 7. free: memory and cache
Read the `available` column, not `free`. Linux fills spare RAM with file cache and hands it back on demand. Low `available` plus swap use means real memory pressure.

```
free -m
```

### 8. sar -n DEV: network throughput
Packets and kB/s per interface. Ignore `lo`. Compare `rxkB/s` and `txkB/s` against what the instance type allows. On EC2, limits are enforced outside the OS, so a capped link can still look quiet here.

```
sar -n DEV 1
```

### 9. sar -n TCP,ETCP: connections and retransmits
`active/s` and `passive/s` are outgoing and incoming connection rates. `retrans/s` is the key one: sustained retransmits relative to `oseg/s` mean packet loss or a congested path.

```
sar -n TCP,ETCP 1
```

### 10. top: confirm and name the culprit

The overview. The header repeats the earlier checks, and the list shows which process is responsible. Press `1` for per-core view and `M` to sort by memory.

```
top
```

---
# `top` cheat sheet

`top` is on every Linux box and answers one question: which process is using the machine? Read the header first, then sort the list. Keys are case-sensitive.

### Reading the header

| Line           | What it tells you                                                                                                |
| -------------- | ---------------------------------------------------------------------------------------------------------------- |
| `load average` | Same as `uptime`: 1, 5, 15 minutes. Compare to core count                                                        |
| `Tasks`        | Totals. A growing `zombie` count means a parent isn't reaping children                                           |
| `%Cpu(s)`      | `us` user, `sy` kernel, `ni` nice, `id` idle, `wa` I/O wait, `hi`/`si` interrupts, `st` stolen by the hypervisor |
| `MiB Mem`      | Watch `avail Mem` (on the Swap line), not `free`. Cache is reclaimable                                           |
| `MiB Swap`     | `used` rising over time means memory pressure                                                                    |

### Reading the columns

| Column      | Meaning                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------------ |
| `PR` / `NI` | Scheduling priority and nice value. Higher nice = lower priority                                 |
| `VIRT`      | Address space mapped. Often huge; rarely the problem                                             |
| `RES`       | Physical RAM in use. The memory figure that matters                                              |
| `SHR`       | Part of `RES` shared with other processes                                                        |
| `S`         | State: `R` running, `S` sleeping, `D` waiting on disk (uninterruptible), `Z` zombie, `T` stopped |
| `%CPU`      | Per-core share, so a multi-threaded process can exceed 100                                       |
| `TIME+`     | Total CPU time used since it started                                                             |

Many processes in state `D` alongside high `wa` points to a disk problem.

### Keys inside top

| Key        | Does                                                             |
| ---------- | ---------------------------------------------------------------- |
| `P`        | Sort by CPU                                                      |
| `M`        | Sort by memory                                                   |
| `T`        | Sort by cumulative CPU time                                      |
| `R`        | Reverse the sort order                                           |
| `1`        | Show each CPU core separately                                    |
| `c`        | Toggle full command line                                         |
| `H`        | Show threads instead of processes                                |
| `V`        | Tree (forest) view                                               |
| `u`        | Filter by user                                                   |
| `o`        | Add a filter, e.g. `COMMAND=splunkd`. Press `=` to clear         |
| `i`        | Hide idle processes                                              |
| `d` or `s` | Change refresh interval (seconds)                                |
| `Space`    | Refresh now                                                      |
| `k`        | Kill a process: enter PID, then signal (15 first, 9 last resort) |
| `r`        | Renice a process (raising nice is safe; lowering needs root)     |
| `f`        | Choose and order columns                                         |
| `E` / `e`  | Cycle memory units in the header / in the list                   |
| `W`        | Save your layout for next time                                   |
| `q`        | Quit                                                             |
### Command-line options
```
top -b -n 1 | head -20
```

One snapshot to paste into a ticket (`-b` batch mode, `-n 1` one iteration).
```
top -u splunk
```

Only the `splunk` user's processes.
```
top -H -p $(pgrep -o splunkd)
```
Threads of the oldest `splunkd` process.

```
top -b -n 5 -d 2 > /tmp/top.txt
```
Five samples, two seconds apart, saved to a file for later.

---
# `htop` cheat sheet

`htop` shows the same data as `top` but is easier to drive: per-core bars on top, a scrolling process list, a function-key menu along the bottom, and mouse support. Install with `sudo apt install htop` if it is missing. Key names below match recent versions; press `F1` to check yours.
### Function keys

| Key          | Does                                                           |
| ------------ | -------------------------------------------------------------- |
| `F1`         | Help and colour legend                                         |
| `F2`         | Setup: meters, columns, display options                        |
| `F3` or `/`  | Search by name (jumps to next match)                           |
| `F4` or `\`  | Filter: show only matching processes                           |
| `F5` or `t`  | Tree view (parent and child)                                   |
| `F6`         | Choose the sort column                                         |
| `F7` / `F8`  | Lower / raise nice value (`F7` raises priority and needs root) |
| `F9`         | Kill: pick a signal, 15 first, 9 last resort                   |
| `F10` or `q` | Quit                                                           |
### Other keys worth knowing

| Key             | Does                                                           |
| --------------- | -------------------------------------------------------------- |
| `P` / `M` / `T` | Sort by CPU / memory / time                                    |
| `I`             | Invert the sort order                                          |
| `u`             | Show only one user's processes                                 |
| `Space`         | Tag a process; act on all tagged ones (e.g. kill)              |
| `c`             | Tag a process and its children                                 |
| `U`             | Untag everything                                               |
| `+` / `-`       | Expand / collapse a branch in tree view                        |
| `H` / `K`       | Hide / show user threads and kernel threads                    |
| `s`             | Trace system calls with `strace` (needs `strace`, run as root) |
| `l`             | List open files with `lsof` (needs `lsof`)                     |
| `e`             | Show a process's environment variables                         |
| `a`             | Pin a process to chosen CPU cores                              |
| `F`             | Follow the selected process as the list re-sorts               |

### Reading the display
- **CPU bars (top left):** one per core. Colours by default: blue low-priority, green normal user, red kernel. `F1` lists the rest, including I/O wait and steal in detailed mode.
- **Mem and Swp bars:** green is used memory, blue buffers, yellow or orange cache. A full bar is normal if most of it is cache; look at the figures, not the bar length.
- **Tasks line:** processes, threads, and how many are running.
- **Load average and Uptime:** same as the `uptime` command.
- **State column (`S`):** `R` running, `S` sleeping, `D` waiting on disk, `Z` zombie. Lots of `D` suggests an I/O problem.

### Setup worth doing once (F2)

1. **Display options:** turn on detailed CPU time so `iowait` and `steal` show in the bars.
2. **Meters:** add Disk IO and Network IO to the header if your version offers them.
3. **Columns:** add the I/O rate columns to see which process is hammering the disk.
4. Settings save automatically to `~/.config/htop/htoprc`.

### Command-line options

```
htop -u splunk
```

Start showing only the `splunk` user's processes.

```
htop -p 1234,5678
```

Watch specific PIDs only.

```
htop -t
```

Start in tree view.

```
htop -d 10
```

Refresh every 1 second (the value is in tenths of a second).

```
htop -s PERCENT_MEM
```

Start sorted by memory (`htop --sort-key help` lists the column names).

---
# Splunk on EC2 Ubuntu: on-the-box troubleshooting

Start with the 60-second checklist to find the stressed resource, then use the symptom sections below to find the Splunk-side cause. Paths assume the defaults: `SPLUNK_HOME=/opt/splunk`, service user `splunk`, systemd unit `Splunkd`. Adjust if yours differ. Splunk CLI commands may prompt for credentials or need `-auth user:pass`.

### Step 0: is Splunk up, and what is it saying?

```
sudo systemctl status Splunkd
sudo -u splunk /opt/splunk/bin/splunk status
sudo tail -n 100 /opt/splunk/var/log/splunk/splunkd.log
```

Service state, Splunk's own view of its processes, then the last 100 log lines. The next two commands pull out only the problems:

```
sudo grep -E "ERROR|WARN" /opt/splunk/var/log/splunk/splunkd.log | tail -n 50
sudo journalctl -u Splunkd -n 50 --no-pager
```

### Splunk won't start
Check ports, disk, ownership and config validity, in that order.
```
sudo ss -tlnp | grep -E "8089|8000|9997|8088"
df -h /opt/splunk && df -i /opt/splunk
sudo find /opt/splunk ! -user splunk | head
sudo -u splunk /opt/splunk/bin/splunk btool check
```

- A listener already on 8089, 8000, 9997 or 8088 means a port clash or a stuck old process.
- Full disk or no free inodes stops startup.
- Files owned by root appear when Splunk was once started as root. Fix with `sudo chown -R splunk:splunk /opt/splunk`.
- `btool check` reports syntax errors in `.conf` files.

### Disk filling, indexing paused
Splunk pauses indexing when free space falls below `minFreeSpace` (default 5000 MB, in `server.conf`).
```
df -h
sudo du -sh /opt/splunk/var/lib/splunk/* | sort -h | tail
sudo du -sh /opt/splunk/var/run/splunk/dispatch
```

Big index directories point to retention or volume sizing. A large `dispatch` directory means search artefacts piling up. Also check the instance's EBS volume size against growth.

### Indexing lag or blocked queues
On the box, look at queue state and disk latency together.
```
sudo grep "blocked=true" /opt/splunk/var/log/splunk/metrics.log | tail -n 20
iostat -sxz 1
```
Blocked parsing, aggregation or indexing queues plus high `await` in `iostat` means the disk is the limit. On EC2 that is usually the EBS volume's IOPS or throughput cap, which the OS cannot see; confirm in CloudWatch.

### Slow searches
```
mpstat -P ALL 1
ps -eo pid,user,rss,pcpu,etime,args --sort=-rss | head -n 15
```

A few cores pegged while others idle is normal for heavy concurrent searches, since each search mostly runs on one core. Long-running, high-RSS `splunkd` search processes show the offenders. Scheduled searches all firing on the hour is the classic cause.

### Data not arriving from forwarders
On the receiver:
```
sudo ss -tlnp | grep 9997
sudo grep -i "blocked" /opt/splunk/var/log/splunk/splunkd.log | tail
```

On the forwarder:
```
/opt/splunkforwarder/bin/splunk list forward-server
nc -vz <indexer-ip> 9997
```

No listener means receiving isn't enabled. A failed `nc` means a security group, NACL or routing problem. Add `sar -n TCP,ETCP 1` on either side to check retransmits.

### HEC problems
```
curl -k https://localhost:8088/services/collector/health
sudo ss -tlnp | grep 8088
```
A healthy endpoint returns `HEC is healthy`. Use `http://` if SSL is off for HEC. No listener means HEC is disabled or Splunk isn't running. Rejected events usually mean a bad token or disabled index; the reason is in `splunkd.log`.

### Splunk killed or restarting unexpectedly
```
sudo dmesg -T | grep -i -E "out of memory|oom|killed process"
free -m
```

A kill line naming `splunkd` means the OOM killer. Cause is usually memory-hungry searches or an undersized instance type. `Killed process` with no Splunk line points elsewhere.
### Limits and kernel settings Splunk complains about

Splunk logs warnings at startup about these; check them directly:

```
cat /proc/$(pgrep -o splunkd)/limits | grep -i -E "open files|processes"
cat /sys/kernel/mm/transparent_hugepage/enabled
```

Splunk recommends an open-files limit of 64000 or more, and transparent huge pages set to `never`. Limits for systemd services are set in the unit file (`LimitNOFILE`), not in `ulimit` shell settings.

### Clock skew

Bad timestamps and cluster flapping often trace back to time drift.
```
timedatectl
chronyc tracking
```
The clock should show as synchronised, with a small offset.

### Clustering checks

```
/opt/splunk/bin/splunk show cluster-status
/opt/splunk/bin/splunk show shcluster-status
```

The first runs on the cluster manager, the second on a search head cluster member. After a peer restarts, expect a burst of disk and network activity while buckets replicate and fix up; that is normal, so compare with your baseline before chasing it.

### Limits that live outside the OS (EC2)
If the box looks healthy but Splunk is slow, AWS may be throttling it.

| Symptom on the box                   | Check in AWS                                                                                                           |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| High `await`, disk cannot keep up    | CloudWatch EBS volume queue length, throughput and IOPS against the volume's provisioned limits; `BurstBalance` on gp2 |
| High `%steal`, nothing busy in `top` | `CPUCreditBalance` on burstable instances (t3, t3a)                                                                    |
| Network retransmits or drops         | ENA allowance counters on the instance (command below)                                                                 |
```
ethtool -S ens5 | grep -i allowance
```

Non-zero and rising `*_allowance_exceeded` counters mean AWS is delaying or dropping packets. Your interface name may not be `ens5`; check with `ip a`.
