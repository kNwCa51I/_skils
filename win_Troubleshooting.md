## The 60-second checklist on Windows 10

Run these in PowerShell as Administrator. `-SampleInterval 1 -MaxSamples N` takes one reading a second, N times. Ctrl+C stops anything running.

**1. Uptime and CPU queue (Linux: uptime)**
There is no load average. The processor queue length is the closest equivalent: tasks waiting for a CPU. Sustained above 2 per core means CPU contention.
```
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
Get-Counter '\System\Processor Queue Length' -SampleInterval 1 -MaxSamples 5
```
Core count: `$env:NUMBER_OF_PROCESSORS`

**2. Event Log errors (Linux: dmesg)**
Critical, error and warning events from the last hour. Look for disk and NTFS errors, Kernel-Power 41 (unexpected shutdown) and driver failures.
```
Get-WinEvent -FilterHashtable @{LogName='System'; Level=1,2,3; StartTime=(Get-Date).AddHours(-1)} -MaxEvents 50 | Format-Table TimeCreated, ProviderName, Id, Message -Wrap
```
The out-of-memory equivalent:
```
Get-WinEvent -FilterHashtable @{LogName='System'; ProviderName='Microsoft-Windows-Resource-Exhaustion-Detector'} -MaxEvents 10
```

**3. System-wide statistics (Linux: vmstat)**
CPU queue, CPU use, free memory, paging and disk queue in one go. High `Pages/sec` with low `Available MBytes` means heavy paging. A disk queue persistently above 2 means the disk is the bottleneck.
```
Get-Counter '\System\Processor Queue Length','\Processor(_Total)\% Processor Time','\Memory\Available MBytes','\Memory\Pages/sec','\PhysicalDisk(_Total)\Current Disk Queue Length' -SampleInterval 1 -MaxSamples 20
```

**4. Per-CPU balance (Linux: mpstat)**
One core pinned near 100% with the rest idle points to a single-threaded bottleneck.
```
Get-Counter '\Processor(*)\% Processor Time' -SampleInterval 1 -MaxSamples 5 | ForEach-Object { $_.CounterSamples | Format-Table InstanceName, @{n='CPU%';e={[math]::Round($_.CookedValue,1)}} -AutoSize }
```

**5. Per-process CPU (Linux: pidstat)**
Top ten by current CPU, as a share of the whole machine. `Get-Process ... CPU` is misleading because it is total seconds since start, not current load. For the user versus kernel split, use `\Process(*)\% User Time` and `\Process(*)\% Privileged Time`.
```
(Get-Counter '\Process(*)\% Processor Time').CounterSamples | Where-Object { $_.InstanceName -notin '_total','idle' } | Sort-Object CookedValue -Descending | Select-Object -First 10 InstanceName, @{n='CPU%';e={[math]::Round($_.CookedValue / $env:NUMBER_OF_PROCESSORS,1)}}
```

**6. Disk I/O (Linux: iostat)**
IOPS, queue length and latency. Latency is in seconds, so 0.010 is 10 ms. Sustained above roughly 20 to 25 ms is poor, and SSDs should be far lower.
```
Get-Counter '\PhysicalDisk(*)\Disk Transfers/sec','\PhysicalDisk(*)\Avg. Disk sec/Read','\PhysicalDisk(*)\Avg. Disk sec/Write','\PhysicalDisk(*)\Current Disk Queue Length','\PhysicalDisk(*)\% Idle Time' -SampleInterval 1 -MaxSamples 10
```

**7. Memory (Linux: free -m)**
`Available MBytes` matches Linux's `available` column, because Windows also keeps reclaimable cache. Low available memory plus heavy paging means real pressure.
```
Get-CimInstance Win32_OperatingSystem | Select-Object @{n='TotalMB';e={[int]($_.TotalVisibleMemorySize/1KB)}}, @{n='FreeMB';e={[int]($_.FreePhysicalMemory/1KB)}}
Get-Counter '\Memory\Available MBytes','\Memory\% Committed Bytes In Use' -SampleInterval 1 -MaxSamples 5
```

**8. Network throughput (Linux: sar -n DEV)**
Bytes and packets per second per adapter. `Output Queue Length` above 2 suggests the adapter can't keep up.
```
Get-Counter '\Network Interface(*)\Bytes Total/sec','\Network Interface(*)\Packets/sec','\Network Interface(*)\Output Queue Length' -SampleInterval 1 -MaxSamples 10
```

**9. TCP statistics (Linux: sar -n TCP,ETCP)**
Retransmits relative to segments sent. Sustained retransmits mean packet loss or congestion. Connection failures and resets show unhealthy connections.
```
Get-Counter '\TCPv4\Segments Sent/sec','\TCPv4\Segments Retransmitted/sec','\TCPv4\Connections Established','\TCPv4\Connection Failures','\TCPv4\Connections Reset' -SampleInterval 1 -MaxSamples 10
```
Cumulative totals since boot:
```
netstat -s -p tcp
```

**10. Overview (Linux: top)**
Task Manager (Ctrl+Shift+Esc), or Resource Monitor for per-process disk files and network connections:
```
resmon
```
Quick top ten by memory in the terminal:
```
Get-Process | Sort-Object WS -Descending | Select-Object -First 10 Name, Id, @{n='WS_MB';e={[int]($_.WS/1MB)}}
```
For an `htop`-style view, Sysinternals Process Explorer is the usual choice (separate download from Microsoft).
