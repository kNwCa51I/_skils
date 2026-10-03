
# Windows "Playing Up" Cheat Sheet: The First 5 Minutes

None of these commands change anything on the machine except Step 8, which creates a report folder. 
**Setup (30 seconds):** Right-click **Start** and choose **Windows PowerShell (Admin)**. Paste each command into the blue window and press Enter. If you see "access denied", you're not in the Admin window.

**Ask the user three things first.** The answers often save you the whole investigation.
1. What is slow: everything, one program, or only the internet?
2. When did it start: today, since an update, or has it always been like this?
3. Is it a real PC or a virtual machine? A virtual machine can be slow because of the computer it runs on, not itself.
## Step 1: How long since it last restarted? (20 seconds)

**What this checks:** Machines left on for weeks get clogged up. A restart clears a lot.  
**How to read it:** More than 7 days is worth noting.  
**Catch:** Windows 10 "Fast Startup" means _Shut down_ doesn't fully restart it. Only **Restart** does.

```powershell
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
```
## Step 2: Is the processor (CPU) overworked? (60 seconds)

**What this checks:** The CPU is the machine's "brain". If it's at 100%, everything feels slow.  
**Where it looks:** It samples the CPU load 5 times, then lists which programs are using the most.  
**How to read it:**

- Under 50% is calm.
- 50-80% is busy.
- Above 90% for a long time is a problem.

Look at the program names at the top of the second list. Common culprits are `MsMpEng` (Windows Defender scanning), `TiWorker` (Windows Update), `SearchIndexer` (file searching), or the program the user complains about.

**Overall CPU load (5 readings, 1 second apart)**
```powershell
Get-Counter '\Processor(_Total)\% Processor Time' -SampleInterval 1 -MaxSamples 5
```

**Top 10 programs by CPU (100 = the whole machine fully used)**
```powershell
(Get-Counter '\Process(*)\% Processor Time').CounterSamples |
  Where-Object { $_.InstanceName -notin '_total','idle' } |
  Sort-Object CookedValue -Descending | Select-Object -First 10 InstanceName,
  @{n='CPU%';e={[math]::Round($_.CookedValue/$env:NUMBER_OF_PROCESSORS,1)}}
```
## Step 3: Is the memory (RAM) full? (45 seconds)
**What this checks:** RAM is the machine's "desk space". When the desk is full, Windows shuffles things to the slow hard drive, which makes everything crawl.  
**How to read it:**

- **FreeGB:** If free memory is under about 10% of the total, it's tight.
- **Pages Input/sec:** This counts how often Windows fetches things back from the disk. Sustained values above about 100 suggest memory trouble. This is a rule of thumb, not a hard limit.
**Total vs free memory in GB**
```powershell
Get-CimInstance Win32_OperatingSystem | Select-Object `
  @{n='TotalGB';e={[math]::Round($_.TotalVisibleMemorySize/1MB,1)}},
  @{n='FreeGB';e={[math]::Round($_.FreePhysicalMemory/1MB,1)}}
```

**Is Windows constantly fetching from the disk? (5 readings)**
```powershell
Get-Counter '\Memory\Pages Input/sec' -SampleInterval 1 -MaxSamples 5
```
**Top 10 programs by memory used**
```powershell
Get-Process | Sort-Object WS -Descending | Select-Object -First 10 Name, Id,
  @{n='MemoryMB';e={[int]($_.WS/1MB)}}
```
## Step 4: Is the disk struggling? (60 seconds)
**What this checks:** The disk stores everything. If it's slow to respond, every program waits.  
**Where it looks:** It measures how many seconds the disk takes to answer a request. **The numbers are in seconds**, so 0.005 means 5 milliseconds.  
**How to read it:**
- Under 0.010 (10 ms) is good for modern drives (SSD).
- Above 0.020 (20 ms) is slow, even for an old spinning drive.
- Anything above 0.100 (100 ms) is very bad. Suspect a failing disk or a scan hogging it.

The second command checks the disk isn't nearly full. **Under 10% free is a warning sign.**
**How fast the disk answers (5 readings)**
```powershell
Get-Counter '\PhysicalDisk(_Total)\Avg. Disk sec/Read','\PhysicalDisk(_Total)\Avg. Disk sec/Write' -SampleInterval 1 -MaxSamples 5
```

**Free space on each drive**
```powershell
Get-CimInstance Win32_LogicalDisk -Filter "DriveType=3" | Select-Object DeviceID,
  @{n='SizeGB';e={[math]::Round($_.Size/1GB,0)}},
  @{n='FreeGB';e={[math]::Round($_.FreeSpace/1GB,0)}},
  @{n='Free%';e={[math]::Round(100*$_.FreeSpace/$_.Size,0)}}
```
## Step 5: Is the network OK? (45 seconds)
**What this checks:** Whether the machine can reach the router, the internet, and look up website names. "The internet is slow" is often a network problem, not a machine problem.  
**How to read it:**
- **Gateway test** (the router): replies should be quick (under about 5 ms) with no lost packets. If this fails, the problem is the cable, Wi-Fi, or the router.
- **Internet test:** if the router works but this fails, suspect the internet connection.
- **Name lookup (DNS):** if the internet test works but this fails, the DNS settings are the problem.

1. **Can we reach the router?**
```powershell
$gw = (Get-NetRoute -DestinationPrefix '0.0.0.0/0' | Select-Object -First 1).NextHop
$gw
Test-Connection $gw -Count 4
```

2. **Can we reach the internet?**
```powershell
Test-Connection 8.8.8.8 -Count 4
```

3. **Can we look up website names?**
```powershell
Resolve-DnsName example.com
```

## Step 6: Has Windows been writing down errors? (45 seconds)

**What this checks:** Windows keeps a diary (the Event Log). This pulls the serious entries from the last 24 hours.  
**How to read it:** Look for repeated entries, or anything at the time the problem began. Key names to know:
- **Kernel-Power 41:** the machine shut down unexpectedly (power cut, crash, or overheating).
- **disk, Ntfs, or storahci:** disk problems.
- **WHEA-Logger:** hardware faults (memory, CPU, motherboard). Take these seriously.
- **Resource-Exhaustion-Detector:** Windows ran out of memory.

**Serious errors in the last 24 hours, grouped so repeats stand out**
```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; Level=1,2; StartTime=(Get-Date).AddDays(-1)} -ErrorAction SilentlyContinue |
  Group-Object ProviderName, Id | Sort-Object Count -Descending |
  Select-Object -First 15 Count, Name
```
(If it returns nothing, that's good news: there were no serious errors.)

**Step 7: Is the power plan throttling it? (15 seconds)**
**What this checks:** "Power saver" mode deliberately slows the processor.  
**How to read it:** If it says _Power saver_, switch to _Balanced_ or _High performance_ and retest.

```powershell
powercfg /getactivescheme
```

## Step 8: Start the full health report, then keep going

**What this checks:** Windows runs its own checkup, covering CPU, memory, disk, network, drivers and settings, then writes a report.  
**How to use it:** It takes **a few minutes**, longer on a struggling machine, and the "60 seconds" on screen is only the first stage. Start it, carry on with your notes, then open the result.  
**Where the result goes:** `C:\PerfLogs\System\Diagnostics\`. Open `report.html` and read the **Warnings** section first.

```powershell
perfmon /report
```

**When it finishes, open the newest report**
```powershell
$f = Get-ChildItem C:\PerfLogs\System\Diagnostics -Recurse -Filter report.html |
  Sort-Object LastWriteTime -Descending | Select-Object -First 1
Invoke-Item $f.FullName
```
## Quick Decision Table

| What you found                                     | What it probably means                         | What to try next                                                                         |
| -------------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------------------- |
| CPU high, one program at the top                   | That program is the cause                      | Check whether it's supposed to be busy. Restart it, or look for updates.                 |
| `MsMpEng` at the top                               | Defender is scanning                           | Wait for the scan, or check scan schedules.                                              |
| `TiWorker` at the top                              | Windows Update is working                      | Let it finish, then restart.                                                             |
| Free memory very low, high Pages Input             | Not enough RAM, or a program is leaking memory | Close or restart the biggest memory user. Consider adding RAM.                           |
| Disk latency high                                  | Failing disk, a scan, or a busy disk           | Check for errors in Step 6. Back up important data if you suspect failure.               |
| Disk nearly full                                   | Windows has no room to work                    | Free up space.                                                                           |
| Router test fails                                  | Cable, Wi-Fi, or router problem                | Check the cable, move closer, restart the router.                                        |
| Router OK but internet test fails                  | Internet connection problem                    | Contact the provider.                                                                    |
| Internet OK but DNS fails                          | DNS settings problem                           | Check the machine's DNS settings.                                                        |
| Kernel-Power 41 or WHEA errors                     | Crashes, overheating or hardware faults        | Check temperatures and dust. Escalate.                                                   |
| Everything looks fine, but the user says it's slow | The problem is intermittent                    | Set up background logging (Step 6 of the earlier guide) and wait for it to happen again. |

## Write Down for the Ticket

- Time and date, and how long since the last restart
- CPU %, free memory, disk delay, and free disk space
- Top 3 programs by CPU and by memory
- Any repeated errors from Step 6
- The `report.html` location

**Golden rule:** change one thing at a time and recheck, so you know what fixed it.

---


