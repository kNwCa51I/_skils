# Sysinternals Cheat Sheet for Troubleshooting

Oct 3, 2026 · @Human

## How to use this sheet

Each tool below has at least two real situations, the command to run, and a plain-English note on what you are looking at. I wrote these from memory and have not run them on your machine, so run `toolname /?` to confirm the switches on your version, and test on a non-critical PC first.

**Getting the tools**

1. Download the Sysinternals Suite zip from Microsoft Learn (search "Sysinternals Suite") and unzip it to `C:\Tools\Sysinternals`. Alternatively, run tools live from `\\live.sysinternals.com\tools` if the machine has internet access.
2. Right-click Start and open **Windows PowerShell (Admin)**. Most tools need the Admin window.
3. Go to the folder and run a tool with `.\`:

```powershell
cd C:\Tools\Sysinternals
.\handle.exe -accepteula
```

`-accepteula` skips the licence pop-up the first time you run a tool. Some antivirus products flag PsExec and similar tools as suspicious because attackers also use them, so tell the security team before you use them on a managed network.

**Words used in this sheet**

| Word | Plain meaning |
| --- | --- |
| Process | A running program. One program can have several. |
| PID | The number Windows gives a running process. Many tools ask for it. |
| Handle | A "hold" a program has on a file, folder or setting while using it. |
| DLL | A shared chunk of code that programs borrow. |
| Service | A program that runs in the background without a window. |
| Driver | Software that lets Windows talk to hardware. |
| Registry | Windows' big settings database. Editing it wrongly can break things. |
| Dump | A snapshot of a program's memory, saved so an expert can study a crash or hang. |
| Latency | How long something takes to answer. Higher is worse. |

**Tools marked Caution** can change or delete things, or affect other computers. Read the command twice before pressing Enter, and never run them on a machine you don't have permission to change.

## Process and system monitoring tools

These answer "what is the machine doing right now?" Start here for slowness, freezes and odd behaviour.

### Process Explorer (procexp.exe): a much better Task Manager

**1. Find the program using the most CPU or memory**

```
procexp.exe
```

Click the **CPU** or **Private Bytes** column heading to sort. The busiest program rises to the top. Hover over a name to see its full file location.

**2. Check whether a strange program is safe**

```
procexp.exe
```

Menu: **Options > VirusTotal.com > Check VirusTotal.com**. Each program gets a score such as 0/72, where 0 means no scanner objected. Anything above 0 deserves a closer look. This sends the file's fingerprint (not the file) to VirusTotal over the internet.

### Process Monitor (procmon.exe): a live log of everything programs do to files and settings

**1. Record 60 seconds of activity without the window**

```
procmon.exe /AcceptEula /Quiet /Minimized /BackingFile C:\temp\trace.pml /Runtime 60
```

Reproduce the problem during those 60 seconds. It saves a recording you can open later. Unfiltered recordings get big fast, so keep them short.

**2. Turn a recording into a spreadsheet-style file you can search**

```
procmon.exe /OpenLog C:\temp\trace.pml /SaveAs C:\temp\trace.csv
```

Open the CSV in Excel. Look for the program's name and results such as `NAME NOT FOUND` or `ACCESS DENIED`, which often explain why something fails to start.

### ProcDump (procdump.exe): saves a snapshot of a program for experts to study

**1. A program is frozen right now**

```
procdump.exe -accepteula -ma 1234 C:\temp
```

Replace 1234 with the program's PID (shown in Task Manager > Details). `-ma` takes a full snapshot. Send the file to the vendor or developer. The program is not closed.

**2. A program only spikes sometimes**

```
procdump.exe -accepteula -ma -c 90 -s 10 -n 3 chrome.exe C:\temp
```

Waits until the program stays above 90% CPU for 10 seconds, then saves a snapshot, up to 3 times. Leave it running in a window until the problem happens.

### Sysmon (sysmon64.exe): a background recorder of program starts and connections

**1. Install it with a settings file (Caution: installs a driver)**

```
sysmon64.exe -accepteula -i sysmonconfig.xml
```

You need a configuration file (security teams usually have a standard one). Without one it logs very little. Ask before installing on a managed machine.

**2. See which programs started recently**

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; Id=1} -MaxEvents 20 | Format-Table TimeCreated, Message -Wrap
```

Event 1 means "a program started". Each entry shows what started it, which is useful when something unexpected keeps launching.

### TCPView (tcpview.exe / tcpvcon.exe): which programs are using the network

**1. See who is talking to the internet right now**

```
tcpvcon.exe -accepteula -a -n
```

Lists each connection with the program that owns it. `-n` skips name lookups so it runs faster. The window version, `tcpview.exe`, shows the same live and highlights new connections in green.

**2. Save the list to send to someone**

```
tcpvcon.exe -accepteula -a -c -n > C:\temp\connections.csv
```

Creates a spreadsheet-style file of every connection. Handy for a ticket or a security review.

### VMMap (vmmap.exe): where one program's memory is going

**1. Find out why one program uses so much memory**

```
vmmap.exe -p chrome.exe
```

The coloured bar splits memory into types. A huge **Heap** or **Private Data** slice that keeps growing suggests the program is leaking memory.

**2. Compare before and after**

```
vmmap.exe
```

Pick the program, then **File > Save** a snapshot. Use the program for a while, take another, and use **View > Compare Snapshots**. Whatever grew is the suspect.

### RAMMap (rammap.exe): how the whole machine is using its memory

**1. Is memory really full, or just used as a speed-up cache?**

```
rammap.exe
```

Open the **Use Counts** tab. Large **Standby** memory is cache Windows hands back when needed, so it is not a problem. Large **Active** or **Modified** memory means programs really are using it.

**2. Test whether the cache is hiding the real problem**

```
rammap.exe
```

Menu: **Empty > Empty Standby List**. Caution: the machine may feel slower for a moment while the cache refills. Use it for testing only, not as a regular fix.

### DebugView (dbgview.exe): reads the hidden messages programs print while running

**1. A program fails and shows no error**

```
dbgview.exe /accepteula
```

In the menu tick **Capture > Capture Win32** and **Capture Global Win32**, then reproduce the problem. Developers often leave helpful messages here.

**2. Save the messages to a file**

```
dbgview.exe /accepteula /l C:\temp\debug.log
```

Writes everything it captures to a file you can attach to a support ticket.

### Coreinfo (coreinfo.exe): what the processor is made of

**1. Is virtualisation switched on?**

```
coreinfo.exe -accepteula -f
```

Look at the `VMX` (Intel) or `SVM` (AMD) line. A `*` means supported and available, a `-` means not. If it shows `-` on a modern PC, it may be turned off in the BIOS.

**2. How many cores and how much cache**

```
coreinfo.exe -accepteula -c
coreinfo.exe -accepteula -l
```

The first lists the physical and logical cores, the second lists cache sizes. Useful for checking a machine matches its specification.

### ClockRes (clockres.exe): how often Windows checks its internal timer

**1. Find out whether a program is forcing fast timers**

```
clockres.exe -accepteula
```

The usual "current timer interval" is around 15.6 ms. A much lower number, such as 1 ms, means some program asked for faster timing, which can shorten laptop battery life.

**2. Find the culprit**

```
clockres.exe
```

Run it, close a suspect program (browser, video player, chat app), run it again. When the number goes back up, you found it.

### CPUSTRES (cpustres.exe): a CPU load generator for testing

**1. Check cooling and stability (Caution: test machines only)**

```
cpustres.exe
```

Set one or more threads to **Maximum** activity and watch temperatures and fan noise. If the machine crashes or overheats, you have found a hardware or cooling problem.

**2. See how a program behaves on a busy machine**

```
cpustres.exe
```

Set threads to **Medium** or **High**, then use the program you are testing. This copies the "it is only slow when the PC is busy" situation on demand.

### Handle (handle.exe): who has a file or folder open

**1. "The file is in use by another program"**

```
handle.exe -accepteula report.docx
```

Lists any program holding a file with that name (part of a name is fine). Close that program normally, then retry. Do not use the force-close options unless you know what you are doing.

**2. "Cannot eject the USB drive"**

```
handle.exe -accepteula E:\
```

Replace E: with the drive letter. It names every program still using the drive. Close those and eject again.

### ListDLLs (listdlls.exe): the shared code each program has loaded

**1. Look for unsigned code**

```
listdlls.exe -accepteula -u
```

Shows loaded DLLs without a valid digital signature. Many are harmless (in-house software), but unknown ones are worth a question.

**2. Which programs are using one particular DLL**

```
listdlls.exe -accepteula -d mscoree.dll
```

Useful when a vendor says "update this file" or a fix needs every program using it closed first.

### PipeList (pipelist.exe): private channels programs use to talk to each other

**1. See what channels exist**

```
pipelist.exe -accepteula
```

Each line is a channel. Security teams use it because some attack tools use unusual channel names.

**2. Check whether a program set up its channel**

```
pipelist.exe -accepteula | findstr /i sql
```

Change `sql` to part of a program's name. Nothing listed while the program is running can mean it failed to start properly.

### LogonSessions (logonsessions.exe): who or what is signed in

**1. List all active sign-ins**

```
logonsessions.exe -accepteula
```

Includes people and background services. Look for accounts you don't expect, or old disconnected sessions.

**2. See which programs run under each sign-in**

```
logonsessions.exe -accepteula -p
```

Helps when a program runs as the wrong account, or when a disconnected remote user left programs running.

### WinObj (winobj.exe): a map of Windows' internal names

View-only. Mostly useful to support engineers and developers.

**1. See which real device sits behind a drive letter**

```
winobj.exe
```

Open `GLOBAL??` in the left list. Each drive letter shows the device it points at.

**2. Check a driver has created its device**

```
winobj.exe
```

Open `\Device` and look for the driver's name. If a vendor says "the device should appear here" and it does not, the driver did not load.

## Startup and persistence tools

These answer "what launches when Windows starts, and what is waiting for a restart?" Use them for slow boots and for unwanted programs that keep coming back.

### Autoruns (autoruns.exe / autorunsc.exe): everything that starts by itself

**1. Find what slows down sign-in**

```
autoruns.exe
```

Open the **Logon** tab. Menu: **Options > Hide Microsoft Entries** to leave only third-party items. Untick an entry to stop it starting (Caution: untick, don't delete, so you can undo it). Restart and test.

**2. Make a checked list of startup items for a report**

```
autorunsc.exe -accepteula -a * -s -m -c > C:\temp\autoruns.csv
```

This is the command-line version. `-s` checks digital signatures, `-m` hides Microsoft items, `-c` makes a spreadsheet file. Anything shown as unsigned, or with a missing file, deserves a question.

### Autologon (autologon.exe): sign in automatically at start-up

**1. Set up a kiosk or display PC (Caution: weakens security)**

```
autologon.exe -accepteula kioskuser MYPC Pa55w0rd
```

Format is username, then computer or domain name, then password. Windows stores the password so anyone with admin rights on that machine can recover it. Do not use it on a normal user PC or a laptop.

**2. Turn it off again**

```
autologon.exe
```

Run it without details and click **Disable**. Do this when a machine is repurposed or leaves the building.

### LoadOrder (loadorder.exe): the order drivers load at boot

**1. See what loads and in what order**

```
loadorder.exe
```

Lists drivers and services in the order Windows starts them. View-only.

**2. Investigate a new boot problem**

```
loadorder.exe
```

If a machine began failing or slowing at start-up after a hardware or software install, look at what loads near the end, because new drivers are usually last. Tell the vendor which one you suspect.

### PendMoves (pendmoves.exe): file changes waiting for the next restart

**1. Why does Windows keep asking for a restart?**

```
pendmoves.exe -accepteula
```

Lists files that are queued to be replaced or deleted when the machine restarts. If items are listed, a restart will finish an update or install.

**2. Check a restart really finished the job**

```
pendmoves.exe
```

Run it after restarting. If the same files are still listed, something is blocking them (often security software or a service). Run it again after investigating.

### MoveFile (movefile.exe): move or delete a file at the next restart

**1. Delete a file that is locked (Caution)**

```
movefile.exe C:\temp\stuck.dll ""
```

The empty quotes mean "delete". The file goes when the machine restarts, before programs can lock it. Be sure of the file name first.

**2. Swap in a new version of a file that is in use (Caution)**

```
movefile.exe C:\temp\new.dll C:\App\old.dll
```

At the next restart, `old.dll` is replaced by `new.dll`. Take a copy of the old file first. Use `pendmoves.exe` to confirm the move is queued.

## Disk and file system tools

These answer "where is my space going, what is on this disk, and can I trust this file?" Several can permanently destroy data, so look for the Caution labels.

### Disk2vhd (disk2vhd.exe): a copy of a whole disk as one file

**1. Back up a PC before a risky change**

```
disk2vhd.exe C: D:\Backup\pc.vhdx
```

Saves drive C: into one file while the PC is running. The destination must be on a different drive from the one being copied, and have enough room.

**2. Turn a physical PC into a virtual machine for testing**

```
disk2vhd.exe * D:\Backup\pc-all.vhdx
```

The `*` copies every drive. You can then attach the file to a virtual machine (for example Hyper-V) and try fixes without touching the real PC.

### DiskExt (diskext.exe): which physical disk holds a drive letter

**1. Which physical disk is C: really on?**

```
diskext.exe -accepteula C:
```

Useful on machines with several disks, so you know which one to replace or test.

**2. See all volumes at once**

```
diskext.exe -accepteula
```

If a drive letter spans more than one disk, losing either disk loses that drive.

### Diskmon (diskmon.exe): a live feed of disk reads and writes

This is an old tool and may not work on current Windows. If it opens empty, use Process Monitor or Resource Monitor (`resmon.exe`) instead.

**1. See whether the disk is busy at all**

```
diskmon.exe
```

Each line is one read or write, with the time it took. A constant stream when the PC is meant to be idle suggests a background program is working the disk.

**2. Keep a record**

```
diskmon.exe
```

Use **File > Save** to keep the log, then show it to whoever is helping. Look for very long durations, which mean the disk answered slowly.

### DiskView (diskview.exe): a picture of what is where on the disk

**1. See where one file sits on the disk**

```
diskview.exe
```

Pick the drive, click **Refresh**, then click a file in the list. It lights up on the map. Many scattered blocks mean a fragmented file.

**2. Get a feel for how full or messy the drive is**

```
diskview.exe
```

Colours show used and free areas. It is only a picture: use it to explain a problem, not to fix one.

### Disk Usage (du.exe): which folders are using the space

**1. Find the big folders**

```
du.exe -accepteula -l 1 C:\Users
```

`-l 1` shows the size of each folder one level down. The biggest numbers are your clean-up targets.

**2. Export folder sizes to a spreadsheet**

```
du.exe -accepteula -c -l 2 C:\Data > C:\temp\sizes.csv
```

Creates a spreadsheet-style file you can sort in Excel.

### Contig (contig.exe): tidy up one file or check how scattered it is

**1. Check one file**

```
contig.exe -accepteula -a C:\Data\big.mdb
```

`-a` only analyses. The report says how many pieces the file is in. Large databases can be in thousands of pieces.

**2. Defragment just that file**

```
contig.exe -accepteula C:\Data\big.mdb
```

Joins the pieces together. It helps on spinning hard drives. Skip it on SSDs, where it gives little benefit.

### Junction (junction.exe): a folder shortcut that programs treat as the real thing

**1. Find existing junctions**

```
junction.exe -accepteula -s C:\Users
```

Lists junctions under a folder. Useful when a folder seems to be the wrong size or when a clean-up reaches somewhere unexpected.

**2. Move a big folder to a bigger drive without breaking programs (Caution)**

```
robocopy C:\Games D:\Games /E /MOVE
junction.exe C:\Games D:\Games
```

The first line moves the files, the second leaves a link at the old location. Programs still find `C:\Games`. To remove the link later, run `junction.exe -d C:\Games`.

### FindLinks (findlinks.exe): all the names one file has

**1. Does this file appear in more than one place?**

```
findlinks.exe -accepteula C:\Data\report.xlsx
```

A file can have several names that all point to the same data (hard links). This lists them.

**2. Why deleting did not free any space**

```
findlinks.exe C:\Data\report.xlsx
```

Space is only freed when every name is deleted. If other names exist, deleting one frees nothing.

### Streams (streams.exe): hidden extra data attached to files

**1. Find files carrying hidden data**

```
streams.exe -accepteula -s C:\Downloads
```

Windows can attach extra data to a file. One common example is the "downloaded from the internet" mark. Anything else unexpected may be worth investigating.

**2. Remove the internet mark on a file you trust (Caution)**

```
streams.exe -d C:\Downloads\tool.exe
```

This stops the "this file came from another computer" warning. Only do it for files you are certain are safe, because the warning exists for a reason.

### NTFSInfo (ntfsinfo.exe): details of a drive's file system

**1. Check the drive's basic settings**

```
ntfsinfo.exe -accepteula C:
```

Shows the size of each storage block (cluster) and the total space. Needed when a vendor asks for the cluster size.

**2. Check the drive's master file table**

```
ntfsinfo.exe C:
```

The MFT is the drive's index of every file. If it is huge, or the drive is nearly full of tiny files, the numbers here explain why.

### SDelete (sdelete.exe): deletes files so they can never be recovered

**1. Securely delete one file (Caution: cannot be undone)**

```
sdelete.exe -accepteula -p 3 C:\temp\confidential.xlsx
```

`-p 3` overwrites the file three times first. A normal delete just removes the entry, so the contents can often be recovered.

**2. Wipe the "empty" space before handing a PC on (Caution)**

```
sdelete.exe -accepteula -c C:
```

Overwrites free space so old deleted files cannot be recovered. It can take hours and makes the drive busy, so run it out of hours. Use `-z` instead to zero the space, which helps shrink virtual disks.

### Sync (sync.exe): writes anything waiting to be saved

**1. Before removing a USB stick**

```
sync.exe -accepteula -r
```

Forces any waiting data out to removable drives. Useful when you can't use the normal eject.

**2. Before pulling the plug on a stuck machine**

```
sync.exe -accepteula C:
```

Saves what Windows is holding in memory for drive C:. It reduces, but does not remove, the risk of losing data.

### VolumeID (volumeid.exe): change a drive's ID number

**1. Give a cloned drive its own ID (Caution)**

```
volumeid.exe -accepteula D: 1A2B-3C4D
```

Copies of a disk share the same ID. Giving each its own avoids clashes with software that tracks the ID. A restart may be needed.

**2. Restore the old ID after a rebuild (Caution)**

```
volumeid.exe C: 5E6F-7A8B
```

Some older software licences are tied to the drive ID. Note the ID before rebuilding (`vol C:` shows it).

### EFSDump (efsdump.exe): who can open an encrypted file

**1. A user cannot open an encrypted file**

```
efsdump.exe -accepteula C:\Secure\plan.docx
```

Lists the accounts that can decrypt it. If the user is not listed, they cannot open it, and you need someone on the list to share it again.

**2. Check a whole folder**

```
efsdump.exe -s C:\Secure
```

The `-s` includes sub-folders. Look for files that only one person, or a leaver's account, can open.

### Strings (strings.exe): reads the readable words inside any file

**1. See what a program says about itself**

```
strings.exe -accepteula -n 8 C:\App\app.exe
```

Pulls out text of 8 or more characters. Look for file paths, error messages, or web addresses.

**2. Search for web addresses inside a suspicious file**

```
strings.exe -accepteula C:\Downloads\invoice.exe | findstr /i http
```

If a harmless-looking file contains unexpected web addresses, report it to your security team. Don't run it.

### Sigcheck (sigcheck.exe): is this file genuine and unaltered?

**1. Check a single file**

```
sigcheck.exe -accepteula C:\Tools\tool.exe
```

Shows who signed it, whether the signature is valid, and its version. "Unsigned" is not proof of danger but needs a reason.

**2. Hunt for unsigned programs in a folder**

```
sigcheck.exe -accepteula -u -e -s C:\ProgramData
```

`-u` shows unsigned items, `-e` limits it to programs, `-s` includes sub-folders. Review the list, as legitimate in-house software often appears.

### LDMDump (ldmdump.exe): dynamic-disk layout information

Rarely needed. I'm not sure of the exact switches, so run `ldmdump.exe /?` first.

**1. A mirrored or spanned volume reports a problem**

```
ldmdump.exe -accepteula
```

Shows how the disks are arranged. Compare it with Disk Management to see which disk is missing or out of sync.

**2. Gather evidence for a support case**

```
ldmdump.exe /?
```

Check the help for the output option on your version, save the output, and send it with your ticket.

## Security and permissions tools

These answer "who is allowed to do what?" Use them when someone gets "access denied" or when you are checking that sharing is not too open. Only scan networks and machines you have permission to check.

### AccessChk (accesschk.exe): who can access a file, folder, setting or service

**1. Who has access to this folder?**

```
accesschk.exe -accepteula -d C:\Data
```

`-d` looks at the folder itself rather than every file in it. Each account is listed with R (read) or W (write). Look for "Everyone" or "Users" with W, which is often too generous.

**2. Can this person write anywhere under this folder?**

```
accesschk.exe -accepteula -w -s bob C:\Data
```

`-w` shows only places with write access, `-s` includes sub-folders. If nothing is listed, Bob cannot change anything there. This explains many "I can open it but can't save it" complaints.

### AccessEnum (accessenum.exe): a permissions report for a whole folder tree

**1. Review who can reach what**

```
accessenum.exe
```

Type or browse to a folder and click **Scan**. It lists every file and folder with its readers, writers and who can change permissions.

**2. Find folders with unusual permissions**

```
accessenum.exe
```

Tick **Show only differences from parent**. The list shrinks to places where someone changed the permissions, which is where mistakes and special cases hide.

### ShareEnum (shareenum.exe): lists shared folders across the network

Caution: scanning a network can look like an attack. Get permission and tell the security team before you run it.

**1. Find every shared folder in an address range**

```
shareenum.exe
```

Enter a start and end address and click **Refresh**. Each shared folder is listed with who can reach it.

**2. Find shares open to everyone**

```
shareenum.exe
```

Click the **Local Path** column to sort, then look at the permissions column for `Everyone`. Tighten any that are not meant to be public.

### ShellRunas (shellrunas.exe): "run as a different user" on the right-click menu

**1. Add the right-click option**

```
shellrunas.exe /reg
```

Then hold Shift, right-click a program and choose **Run as different user**. Handy for testing as a normal user or running one program as an admin without logging off.

**2. Launch one program as someone else, or remove the option**

```
shellrunas.exe notepad.exe
shellrunas.exe /unreg
```

The first line asks for the other user's sign-in and opens that program under that account. The second removes the right-click option again.

### NotMyFault (notmyfault64.exe): deliberately crashes the machine to test crash handling

Caution: this will crash or freeze Windows. Use it only on a test machine with no unsaved work.

**1. Check crash dumps are being saved**

```
notmyfault64.exe /crash
```

The machine shows a blue screen and restarts. Afterwards, check `C:\Windows\MEMORY.DMP` exists. If not, crash recording is not set up properly. I believe the `/crash` switch exists, but check `notmyfault64.exe /?`.

**2. Test how the system copes with a memory leak**

```
notmyfault64.exe
```

In the window, choose a leak type and click **Leak**. Watch Task Manager. This helps you practise spotting a leak and test your alerts.

### BlueScreen (bluescreen.scr): a fake blue screen screensaver

Caution: a fake crash can alarm colleagues and trigger monitoring alerts, so tell people first.

**1. Training or demonstrations**

```
bluescreen.scr /s
```

Shows the fake crash screen immediately. Press any key to leave it. It helps staff recognise what a real blue screen looks like.

**2. Install it as a screensaver**

Right-click `bluescreen.scr` and choose **Install**. It then appears in the screensaver list. Remove it by choosing a different screensaver.

## Active Directory tools

Active Directory (AD) is the company's central list of users, computers and groups. These tools only apply on a company network with a domain. Ask the domain admin before changing anything.

### AD Explorer (adexplorer.exe): browse the company directory

**1. Look up exactly what the directory holds about a user or computer**

```
adexplorer.exe
```

Connect (leave the server blank to use your own domain), then open the folders on the left and click an account. The right side lists every detail, such as last sign-in, group memberships and whether it is locked.

**2. Take a "before" picture of the directory**

```
adexplorer.exe -snapshot "" C:\temp\ad-before.dat
```

Saves a copy of the directory you can open offline. Take another after a change and use **File > Compare** to see exactly what changed. I believe the `-snapshot` switch is correct but check on your version.

### AD Insight (adinsight.exe): watch the conversation between a program and the directory

**1. Sign-in or lookups are slow**

```
adinsight.exe
```

Click the capture button, reproduce the slowness, and stop. Sort by **Duration**. Requests taking seconds instead of milliseconds are the cause, and the directory server named in the line is the one responding slowly.

**2. Find out which directory server a program is using**

```
adinsight.exe
```

The **Server** column shows who the program contacted. If it is a server that should have been retired, or one in a faraway office, that explains many odd delays.

### AD Restore (adrestore.exe): bring back a deleted directory account

Caution: this changes the live directory. Only do it with the domain admin's approval.

**1. See what was deleted recently**

```
adrestore.exe -accepteula
```

Lists deleted accounts and other objects still recoverable. Add part of a name at the end to narrow the list.

**2. Restore one account**

```
adrestore.exe -accepteula -r bob
```

It asks "Do you want to restore?" for each match. Say yes only to the right one. Some details, such as group memberships, may not come back, so check the account afterwards. If your domain has the AD Recycle Bin turned on, the admin may prefer the PowerShell route instead.

## PsTools: command-line tools that work on this PC or another one

These run from PowerShell. Put a computer name after two backslashes (for example `\\PC01`) to run them on another machine. That needs admin rights on that machine, and Windows firewall and file-sharing settings must allow it. Antivirus often flags this family, so tell the security team before use.

### PsExec (psexec.exe): run a program on another computer

**1. Run one command on a remote PC**

```
psexec.exe \\PC01 ipconfig /all
```

Runs the command there and shows the result here. Good for a quick check without walking over or opening a remote session.

**2. Open a command window with the highest privilege (Caution)**

```
psexec.exe -s -i cmd.exe
```

`-s` runs as the built-in SYSTEM account, which can reach things even admins can't. Use it to investigate stuck settings, then close the window. Do not leave it open.

### PsFile (psfile.exe): which files other people have open on this PC

**1. Who is using a shared file right now?**

```
psfile.exe
```

Lists files on this machine that are open from other computers (for example shared folders). Each entry has an ID number.

**2. Close one so you can update or move it (Caution)**

```
psfile.exe 5 -c
```

Replace 5 with the ID from the list. The other person may lose unsaved changes, so warn them first.

### PsGetSid (psgetsid.exe): turn account numbers into names

**1. A permissions list shows a long number instead of a name**

```
psgetsid.exe S-1-5-21-1111111111-2222222222-3333333333-1001
```

Paste the number (called a SID). It tells you which account owns it. If it says the account no longer exists, the permission is left over from a deleted account.

**2. Find a user's or computer's ID**

```
psgetsid.exe bob
psgetsid.exe \\PC01
```

The first shows Bob's SID, the second shows the machine's. Cloned machines sharing one ID can cause odd trouble.

### PsInfo (psinfo.exe): a quick fact sheet about a computer

**1. What is this machine?**

```
psinfo.exe -accepteula
```

Shows uptime, Windows version, processor and memory. Paste it into a ticket so the next person doesn't have to ask.

**2. Installed software and disks on a remote PC**

```
psinfo.exe \\PC01 -s -d
```

`-s` lists installed software, `-d` lists disks and free space. Useful for "is the right version installed?" and "is the disk full?" without visiting.

### PsKill (pskill.exe): force a program to stop

Caution: stopped programs lose unsaved work.

**1. Stop a frozen program**

```
pskill.exe notepad
```

Ends the program by name (or give its PID number). Use this when it does not respond to the normal close button.

**2. Stop a program on another PC**

```
pskill.exe \\PC01 -t chrome
```

`-t` also ends everything that program started. Check you have the right PC name, as the effect is immediate.

### PsList (pslist.exe): running programs in a text list

**1. A live, refreshing view**

```
pslist.exe -s 2
```

Refreshes every 2 seconds. It works like Task Manager but in a window you can copy from. Press Ctrl+C to stop it.

**2. Memory detail for one program**

```
pslist.exe -m chrome
```

Shows how much memory each copy of Chrome uses. Check for one that is much larger than the others, which may be a leaking tab or add-on.

### PsLoggedOn (psloggedon.exe): who is signed in

**1. Who is using this PC?**

```
psloggedon.exe -accepteula
```

Lists people signed in at the keyboard and people connected over the network. A good first check before restarting a machine.

**2. Who is on a remote PC?**

```
psloggedon.exe \\PC01
```

Tells you whether a colleague is working on it before you interrupt them. Very useful before a restart or update.

### PsLogList (psloglist.exe): the Windows event log in a text window

**1. Problems from the last day**

```
psloglist.exe -accepteula -d 1 -f ew system
```

`-d 1` is the last day, `-f ew` keeps only errors and warnings, and `system` picks the System log. Look for entries repeating or matching the time the problem began.

**2. Latest application errors on a remote PC**

```
psloglist.exe \\PC01 -n 20 -f e application
```

Shows the last 20 errors in the Application log. Save the output to a file by adding `> C:\temp\errors.txt`.

### PsPasswd (pspasswd.exe): change an account password

Caution: the new password is typed into the command and may be stored in the window's history. Most companies now prefer a managed system (such as Microsoft LAPS) for local admin passwords. Check your policy first.

**1. Change a local account password on one PC**

```
pspasswd.exe \\PC01 Administrator NewP@ssw0rd!
```

Sets a new password for that account on that PC. Use a long, unique password and record it in the approved password vault.

**2. Change it on a list of PCs**

```
pspasswd.exe @C:\temp\pclist.txt Administrator NewP@ssw0rd!
```

The text file has one computer name per line. Note which machines failed, because the change may only succeed on some.

### PsPing (psping.exe): test whether a machine or service answers, and how fast

**1. Basic reachability and speed**

```
psping.exe -n 10 8.8.8.8
```

Sends 10 test messages and reports the reply time. Consistently slow replies or lost messages point at the network.

**2. Is a particular service port open?**

```
psping.exe -n 10 server01:443
```

Tests the connection to that service (443 is secure web traffic) and times it. If this fails but ordinary ping works, a firewall or the service itself is the issue.

### PsService (psservice.exe): check and control Windows services

**1. Is a service running?**

```
psservice.exe query spooler
```

Shows the state of the print service. Replace `spooler` with the service name. Add `\\PC01` first to check another PC.

**2. Restart a stuck service (Caution)**

```
psservice.exe restart spooler
```

Stops and starts it again. This is the classic fix for a print queue that will not clear. Anything using the service is interrupted for a moment.

### PsShutdown (psshutdown.exe): restart or shut down a PC

Caution: unsaved work is lost.

**1. Restart with a warning countdown**

```
psshutdown.exe -r -t 120 -m "Restarting for maintenance in 2 minutes. Save your work."
```

`-r` restarts, `-t 120` waits 2 minutes, `-m` shows a message to the person at the screen.

**2. Change your mind**

```
psshutdown.exe -a
```

Cancels a countdown that is in progress. Remember this one before you run the first command on a remote PC.

### PsSuspend (pssuspend.exe): pause a program without closing it

**1. Pause a program that is eating the CPU while you investigate**

```
pssuspend.exe chrome.exe
```

Freezes the program in place, using almost no CPU, with its work intact. Do not pause system programs or anti-virus.

**2. Resume it**

```
pssuspend.exe -r chrome.exe
```

The program carries on where it stopped. It might appear stuck for a moment while it catches up.

## Registry, networking and everyday utilities

A mixed group. The registry tools can break Windows if misused, so back up first.

### Regjump (regjump.exe): open the registry editor at an exact spot

**1. Follow a vendor's instruction that gives a registry path**

```
regjump.exe HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

Opens the registry editor already at that location, so you don't have to click through folders. This particular spot lists programs that start at sign-in.

**2. Look before you change anything**

```
reg export HKLM\SOFTWARE\MyVendor C:\temp\myvendor-backup.reg
regjump.exe HKLM\SOFTWARE\MyVendor
```

The first line saves a backup of that branch. The second opens it. If a change goes wrong, double-click the backup file to put things back.

### RegDelNull (regdelnull.exe): remove registry entries Windows cannot normally delete

Caution: back up first. Some security software uses hidden entries like these, so only run this on advice or after checking with the security team.

**1. Scan for broken or hidden keys**

```
regdelnull.exe -accepteula -s HKLM\SOFTWARE
```

Searches under that branch for keys with a hidden null character in the name. Most machines show none.

**2. Delete a key it found**

```
reg export HKLM\SOFTWARE C:\temp\software-backup.reg
regdelnull.exe -s HKLM\SOFTWARE
```

Export first, then run it again and answer its prompt for the specific key. Don't delete anything you don't recognise without checking.

### PortMon (portmon.exe): watch what happens on serial and parallel ports

A legacy tool for old-style connections (for example a barcode scanner, a till, or a serial label printer). It may not work on current 64-bit Windows.

**1. A serial device is not responding**

```
portmon.exe
```

Start capture, use the device, and look at the data going back and forth. If nothing is sent when you trigger the device, the fault is likely the cable or the device.

**2. Give the supplier proof of the settings**

```
portmon.exe
```

Use **File > Save** after reproducing the problem. The log shows the speed and format settings the program tried to use, and the supplier can compare them with their device.

### Whois (whois.exe): who owns a web address

**1. Look up a company domain**

```
whois.exe -accepteula example.com
```

Shows the registrar and the registration and expiry dates. An expired or soon-to-expire domain can cause email and website outages.

**2. Check a suspicious sender's domain**

```
whois.exe suspicious-site.com
```

A domain registered only a few days ago is a warning sign for phishing. Results vary by type of domain, so use it as one clue, not proof.

### BgInfo (bginfo.exe): put key facts on the desktop background

**1. Show the computer name and address on every desktop**

```
bginfo.exe C:\Tools\standard.bgi /timer:0 /silent /nolicprompt
```

First open `bginfo.exe`, choose which details to show (name, IP address, free disk space, uptime), and save as `standard.bgi`. The command then applies it with no windows. Great for telling similar servers apart in a remote session.

**2. Show disk space on servers**

```
bginfo.exe
```

In the settings window, add the **Disk Space** or **Free Space** fields. You can then see at a glance when a server is running out of room.

### Desktops (desktops.exe): extra virtual screens

**1. Keep admin tools out of the way**

```
desktops.exe
```

Runs up to four separate desktops, each with its own open windows. Switch with the shortcuts you set in the settings. Windows 10 and 11 also have this built in, using Win+Tab.

**2. Isolate a risky test**

```
desktops.exe
```

Open the risky program on a separate desktop so it doesn't clutter your main work. It is only a tidy-up and gives no security protection.

### Ctrl2Cap (ctrl2cap.exe): swap Caps Lock for Ctrl

Caution: installs a keyboard driver that affects every user of the PC.

**1. Install it**

```
ctrl2cap.exe /install
```

Restart afterwards. Caps Lock then acts as Ctrl, which is handy for people who use shortcuts a lot.

**2. Remove it**

```
ctrl2cap.exe /uninstall
```

Restart again. If a keyboard behaves strangely after a cleanup, check whether this was left installed.

### ZoomIt (zoomit.exe): zoom and draw on screen for demos

**1. Zoom in while explaining**

```
zoomit.exe
```

It sits in the system tray. Press `Ctrl+1` to zoom the screen, use the mouse wheel to adjust, and press Esc to leave.

**2. Draw on the screen, or run a break timer**

```
zoomit.exe
```

Press `Ctrl+2` to draw (type a letter for a colour, such as R for red), and `Ctrl+3` for a countdown timer during a training session. Check the shortcuts in the settings window.

### Hex2dec (hex2dec.exe): convert between hexadecimal and ordinary numbers

I'm not certain of every switch, so run `hex2dec.exe /?` first.

**1. Decode an error number**

```
hex2dec.exe 0x80070005
```

Error messages often show a code starting `0x`. Converting it to an ordinary number can help when a search or a vendor guide uses the decimal form. This particular code means "access denied".

**2. Convert the other way**

```
hex2dec.exe 2147942405
```

Turns an ordinary number into hexadecimal. PowerShell can also do it: `'{0:X}' -f 2147942405`.

### CacheSet (cacheset.exe): view and change the file cache limit

Rarely needed on modern Windows, which manages the cache well. Use only on a specialist's advice. Caution: a wrong setting can make things slower.

**1. Look at the current cache size**

```
cacheset.exe
```

Shows how much memory Windows will use to remember recently used files. View-only unless you click **Apply**.

**2. Cap the cache on an older server where it starves programs**

```
cacheset.exe
```

Set a maximum, click **Apply**, and watch whether the programs recover. Note the original values first so you can put them back.

### LiveKd (livekd.exe): examine the running system with a debugger

For specialists. It needs Microsoft's debugging tools (WinDbg) installed. Use it only on advice, or when a vendor asks for a dump from a machine you can't restart.

**1. Open a debugger on the live machine**

```
livekd.exe -w
```

Starts WinDbg looking at the running system, with no restart or crash needed. Experts can look at why something is hung.

**2. Save a snapshot for a vendor**

```
livekd.exe -o C:\temp\live.dmp
```

Writes a snapshot of the system's memory to a file. It can be large (several gigabytes), so check the free space on that drive first.

## Quick lookup: what's wrong, which tool

Find the symptom, start with the first tool listed. This sheet covers the tools I could describe confidently; Microsoft adds and retires tools, so check the official Sysinternals list for anything missing.

| Symptom | Start with | Then try |
| --- | --- | --- |
| PC slow, not sure why | Process Explorer | RAMMap, Process Monitor |
| One program uses huge memory | VMMap | Process Explorer |
| Program frozen, vendor wants evidence | ProcDump | LiveKd (specialists) |
| Program fails with no error | Process Monitor | DebugView |
| Slow sign-in or start-up | Autoruns | LoadOrder |
| "File is in use by another program" | Handle | Process Explorer (Ctrl+F) |
| Cannot eject a USB drive | Handle | Sync |
| Windows keeps asking for a restart | PendMoves | MoveFile |
| Disk filling up | Disk Usage (du) | FindLinks, Junction |
| Large file very slow to open | Contig | DiskView |
| Is this file genuine? | Sigcheck | Strings, Streams |
| Something suspicious is talking to the internet | TCPView | Sysmon, Process Monitor |
| "Access denied" | AccessChk | AccessEnum |
| Looking for over-shared folders | ShareEnum | AccessEnum |
| Need facts from a remote PC | PsInfo | PsExec, PsList |
| Who is on that PC before I restart it | PsLoggedOn | PsShutdown |
| Print queue stuck | PsService | PsKill |
| Slow directory lookups or sign-in on a domain | AD Insight | AD Explorer |
| Deleted directory account | AD Restore | AD Explorer |
| Handing a PC to someone else | SDelete | Disk2vhd (backup first) |
| Backing up a PC before a risky change | Disk2vhd | Sync |

**Safe order of use on a problem machine:** look first (Process Explorer, TCPView, Autoruns, Sigcheck, PsInfo), record what you see, then record behaviour (Process Monitor, ProcDump), and only then change things (anything marked Caution). Change one thing at a time and recheck.

**Before sharing a screenshot or log with a vendor,** check it for computer names, usernames and file paths you don't want to send.
