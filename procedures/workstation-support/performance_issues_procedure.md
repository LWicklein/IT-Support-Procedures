# Workstation Performance Issues Diagnosis and Resolution

## Overview
**Category:** Workstation Support / Performance Optimization  
**Severity:** Medium (impacts productivity but not blocking work)  
**Estimated Resolution Time:** 30-60 minutes  
**Required Access Level:** Help Desk with local admin rights

### Description
User reports workstation running slowly, experiencing lag, freezing, or general performance degradation. This is one of the most common support requests and can have multiple root causes ranging from simple resource constraints to malware infections. Systematic diagnosis is essential to identify and resolve the underlying issue efficiently.

### Common Symptoms
- Applications take a long time to open or respond
- Windows startup/shutdown is extremely slow
- Frequent "Not Responding" messages
- Mouse cursor lags or stutters
- System freezes or hangs periodically
- High fan noise or excessive heat
- Blue screens or random crashes
- Windows updates or Office applications unusually slow

---

## Prerequisites
- [ ] Remote access to user's workstation or on-site physical access
- [ ] Local administrator credentials
- [ ] Performance monitoring tools (Task Manager, Resource Monitor, Event Viewer)
- [ ] Malware scanning tools if available
- [ ] User's patience (diagnosis and resolution can take time)

---

## Resolution Steps

### Step 1: Gather Information and Assess Symptoms
**Action:**
Interview user to understand the scope and timeline of performance issues.

**Key Questions to Ask:**
- When did the slowness start? (Sudden or gradual?)
- Has anything changed recently? (New software, Windows updates, hardware changes?)
- Which applications are affected? (All programs or specific ones?)
- Does it happen all the time or at specific times of day?
- Have you noticed any error messages or unusual pop-ups?
- Is the computer making unusual noises (fan running constantly)?
- How long ago was the computer restarted?

**Quick First Check:**
```powershell
# Check system uptime
Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object LastBootUpTime

# Get basic system info
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Total Physical Memory"
```

**Expected Result:**
Understanding of symptom pattern and timeline helps narrow potential causes.

**Common Patterns:**
- **Sudden onset + recent update** → Windows update issue
- **Gradual over weeks/months** → Disk fragmentation, accumulation of junk files
- **Specific application slow** → Application-specific issue
- **Morning slowness only** → Background maintenance tasks, antivirus scans
- **After installing software** → New software consuming resources or malware

---

### Step 2: Check Resource Utilization
**Action:**
Use Task Manager and Resource Monitor to identify what's consuming system resources.

**Open Task Manager (Ctrl+Shift+Esc):**

**Check Performance Tab:**
1. CPU Usage - Should be under 70% at idle, spikes to 100% are normal during tasks
2. Memory - If consistently over 80-85%, memory constraint exists
3. Disk - Sustained 100% disk usage indicates bottleneck
4. Network - High usage when user isn't transferring files suggests issue

**Check Processes Tab:**
```
Sort by CPU → Identify processes consuming high CPU
Sort by Memory → Identify memory-hungry applications
Sort by Disk → Identify what's hammering the disk
```

**Using PowerShell for detailed analysis:**
```powershell
# Top CPU consumers
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 ProcessName, CPU, PM

# Top Memory consumers
Get-Process | Sort-Object WS -Descending | Select-Object -First 10 ProcessName, @{Name="Memory(MB)";Expression={[math]::Round($_.WS/1MB,2)}}

# Check for processes with unusual memory usage
Get-Process | Where-Object {$_.WorkingSet -gt 500MB} | Select-Object Name, @{Name="Memory(MB)";Expression={[math]::Round($_.WS/1MB,2)}}
```

**Expected Result:**
Identification of resource bottleneck (CPU, memory, disk, or network).

**Common Findings:**
- **100% disk usage** → Most common cause of slowness on mechanical drives or failing SSDs
- **High memory usage** → Too many programs open, memory leak, or insufficient RAM
- **Antivirus scanning** → Legitimate but can cause temporary slowness
- **Windows Update service** → Background updates consuming resources
- **Unknown processes** → Potential malware

---

### Step 3: Check Disk Health and Space
**Action:**
Verify disk has sufficient free space and is healthy.

**Check Disk Space:**
```powershell
# Check disk space on all drives
Get-PSDrive -PSProvider FileSystem | Select-Object Name, @{Name="Used(GB)";Expression={[math]::Round($_.Used/1GB,2)}}, @{Name="Free(GB)";Expression={[math]::Round($_.Free/1GB,2)}}, @{Name="Free(%)";Expression={[math]::Round(($_.Free/$_.Used)*100,2)}}

# Or using WMI
Get-WmiObject Win32_LogicalDisk -Filter "DriveType=3" | Select-Object DeviceID, @{Name="Size(GB)";Expression={[math]::Round($_.Size/1GB,2)}}, @{Name="FreeSpace(GB)";Expression={[math]::Round($_.FreeSpace/1GB,2)}}
```

**Check Disk Health:**
```cmd
# Check disk for errors (requires admin)
chkdsk C: /scan

# Check SMART status
wmic diskdrive get status
# Should return "OK" for healthy drives

# For SSD health, check manufacturer tools or:
Get-PhysicalDisk | Get-StorageReliabilityCounter | Select-Object DeviceId, Temperature, Wear
```

**Expected Result:**
- At least 15-20% free space on system drive (minimum 10GB)
- Disk status shows "OK" or "Healthy"
- No bad sectors or hardware errors

**Troubleshooting:**
- **Less than 10% free space:** Critical - proceed to disk cleanup
- **Disk status "Pred Fail" or errors:** Failing drive, backup data immediately
- **Mechanical drive constantly at 100%:** Consider SSD upgrade recommendation

---

### Step 4: Identify and Disable Startup Programs
**Action:**
Review and disable unnecessary programs that launch at startup.

**Using Task Manager:**
1. Open Task Manager → Startup tab
2. Sort by "Startup impact"
3. Disable non-essential "High impact" programs
4. Right-click → Disable for programs user doesn't need at startup

**Using PowerShell:**
```powershell
# List all startup programs
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location, User

# Check startup programs for current user
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"

# Check startup programs for all users
Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

**Common Culprits to Disable:**
- Adobe Creative Cloud (unless user actively uses it)
- Skype (if not needed immediately at startup)
- Third-party updaters (Java, Adobe, etc.)
- Manufacturer bloatware (HP, Dell utilities)
- Old backup software
- Unnecessary cloud sync tools

**Expected Result:**
Reduced startup time and fewer programs competing for resources at boot.

**Warning:** Never disable:
- Antivirus software
- Critical Windows services
- VPN clients (if required for work)
- Essential business applications

---

### Step 5: Run Disk Cleanup and Optimization
**Action:**
Free up disk space and optimize drive performance.

**Run Disk Cleanup:**
```cmd
# Open Disk Cleanup utility
cleanmgr /d C:

# Or automated cleanup (requires admin)
cleanmgr /sagerun:1
```

**Manual Steps:**
1. Open Disk Cleanup (cleanmgr)
2. Click "Clean up system files" (requires admin)
3. Select all checkboxes except "Downloads"
4. Check "Windows Update Cleanup" and "Previous Windows installations" if available
5. Click OK to clean

**Empty Temp Folders:**
```cmd
# Delete temp files (safe to do)
del /q /f /s %TEMP%\*
del /q /f /s C:\Windows\Temp\*

# Clear Windows Update cache if needed
net stop wuauserv
del /q /f /s C:\Windows\SoftwareDistribution\Download\*
net start wuauserv
```

**Defragment or Optimize Drive:**
```cmd
# For mechanical drives - defragment
defrag C: /U /V

# For SSDs - optimize (TRIM)
defrag C: /O /V

# Check optimization status
defrag C: /A
```

**Using PowerShell:**
```powershell
# Optimize all drives
Get-Volume | Optimize-Volume -Verbose

# Check defrag status
Get-Volume | Select-Object DriveLetter, @{Name="Fragmented(%)";Expression={(Get-Volume -DriveLetter $_.DriveLetter | Optimize-Volume -Analyze).FragmentedPercentage}}
```

**Expected Result:**
- Several GB of disk space freed
- Improved disk read/write performance
- System drive optimized

---

### Step 6: Check for Malware and Unnecessary Software
**Action:**
Scan for malware and identify unwanted programs consuming resources.

**Run Antivirus Scan:**
```powershell
# If Windows Defender is used
Start-MpScan -ScanType FullScan

# Check Windows Defender status
Get-MpComputerStatus | Select-Object AntivirusEnabled, AMServiceEnabled, RealTimeProtectionEnabled, IoavProtectionEnabled

# Update definitions first
Update-MpSignature
```

**Check Installed Programs:**
```powershell
# List all installed applications
Get-WmiObject -Class Win32_Product | Select-Object Name, Version, InstallDate | Sort-Object Name

# Or faster method using registry
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, Publisher, InstallDate
```

**Common Unnecessary Software (Ask Before Removing):**
- Browser toolbars (Ask, Babylon, etc.)
- PC optimization tools (CCleaner, registry cleaners - often cause more harm)
- Trials of software user doesn't use
- Duplicate programs (multiple PDF readers, media players)
- Old software no longer needed

**Expected Result:**
- No malware detected
- Unnecessary programs identified for removal
- System cleaner without resource-hogging software

---

### Step 7: Check Windows Updates and Drivers
**Action:**
Verify Windows and drivers are up to date, or if recent update caused issue.

**Check Windows Update Status:**
```powershell
# Check for pending updates
Get-WindowsUpdate

# Check update history
Get-WmiObject -Class Win32_QuickFixEngineering | Select-Object Description, HotFixID, InstalledOn | Sort-Object InstalledOn -Descending

# If updates are pending/stuck
Get-Service wuauserv | Restart-Service
```

**Check Driver Issues:**
```powershell
# Check for devices with driver problems
Get-WmiObject Win32_PnPEntity | Where-Object {$_.ConfigManagerErrorCode -ne 0} | Select-Object Name, DeviceID, ConfigManagerErrorCode

# List all drivers
Get-WindowsDriver -Online -All | Select-Object Driver, ProviderName, Date, Version
```

**Using Device Manager:**
1. Open Device Manager (devmgmt.msc)
2. Look for yellow exclamation marks (driver issues)
3. Check display adapter, network adapter, disk controllers
4. Right-click → Update driver for any problem devices

**Expected Result:**
- Windows is up to date or problematic update identified
- All device drivers functioning properly
- No hardware conflicts

**Troubleshooting:**
- **Recent update caused slowness:** May need to uninstall specific update
- **Driver issues:** Download latest from manufacturer website
- **Graphics driver problems:** Common cause of UI lag

---

### Step 8: Check System Temperature and Hardware Health
**Action:**
Verify computer isn't overheating or experiencing hardware failure.

**Check Event Viewer for Hardware Errors:**
```powershell
# Check for critical errors in last 7 days
Get-EventLog -LogName System -EntryType Error -After (Get-Date).AddDays(-7) | Select-Object TimeGenerated, Source, EventID, Message | Format-Table -AutoSize

# Check for specific hardware warnings
Get-WinEvent -FilterHashtable @{LogName='System'; Level=2,3; StartTime=(Get-Date).AddDays(-7)} | Select-Object TimeCreated, ProviderName, Id, Message
```

**Physical Inspection (if on-site):**
- Check if computer feels hot to touch
- Listen for unusual fan noise (grinding, loud constant spinning)
- Look for dust buildup in vents
- Verify fans are spinning when computer is on

**Temperature Check (if monitoring software available):**
- CPU should be under 80°C under load
- GPU should be under 85°C under load
- Sustained high temps indicate cooling problem

**Expected Result:**
- Normal operating temperatures
- No hardware errors in Event Viewer
- Fans operating normally

**Troubleshooting:**
- **Overheating:** Clean dust from vents, consider thermal paste replacement
- **Fan failure:** Replace fan immediately
- **Frequent hardware errors:** Failing component, escalate for replacement

---

### Step 9: Disable Visual Effects and Adjust Power Settings
**Action:**
Reduce visual effects and optimize power settings for performance.

**Adjust Performance Options:**
```cmd
# Open Performance Options
SystemPropertiesPerformance.exe
```

**Manual Steps:**
1. Click "Adjust for best performance" (disables all effects)
2. Or select "Custom" and disable:
   - Animate windows when minimizing/maximizing
   - Animations in taskbar
   - Fade or slide menus into view
   - Show shadows under mouse pointer
3. Keep enabled:
   - Show thumbnails instead of icons
   - Smooth edges of screen fonts

**Adjust Power Settings:**
```powershell
# Set to High Performance power plan
powercfg -setactive 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c

# Check current power plan
powercfg -getactivescheme

# Disable USB selective suspend (can cause peripheral lag)
powercfg -setdcvalueindex SCHEME_CURRENT 2a737441-1930-4402-8d77-b2bebba308a3 48e6b7a6-50f5-4782-a5d4-53bb8f07e226 0
powercfg -setacvalueindex SCHEME_CURRENT 2a737441-1930-4402-8d77-b2bebba308a3 48e6b7a6-50f5-4782-a5d4-53bb8f07e226 0
```

**Expected Result:**
- Reduced graphical overhead
- More consistent performance
- Faster UI response

---

### Step 10: Restart and Verify Improvement
**Action:**
Restart computer and verify performance improvements with user.

**Before Restart:**
- Save all user work
- Note current resource usage for comparison
- Ensure Windows updates won't install during restart (if time-sensitive)

**After Restart:**
```powershell
# Check boot time
Get-EventLog -LogName System -Newest 1 -Source "EventLog" | Where-Object {$_.EventID -eq 6005}

# Verify resource usage improved
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
```

**Test with User:**
- Open commonly-used applications
- Perform typical tasks
- Verify responsiveness improved
- Check if specific slow operations are now faster

**Expected Result:**
Noticeable performance improvement, faster application response, reduced lag.

---

## Verification
After completing resolution steps, verify:
- [ ] Task Manager shows normal resource utilization (not constantly maxed)
- [ ] Applications open in reasonable time
- [ ] System feels responsive to user interactions
- [ ] No freezing or "Not Responding" messages during testing
- [ ] Startup time improved (if that was an issue)
- [ ] User confirms improvement meets expectations

**Document Baseline:**
Take note of current performance metrics for future reference if issue recurs.

---

## Documentation Template
Use this when logging the ticket:

**Issue Summary:**
User [username] reported workstation [computer name] experiencing slow performance, lag, and application delays.

**Root Cause:**
[Primary cause identified: Low disk space / 100% disk utilization / Insufficient RAM / Excessive startup programs / Malware detected / Failing hard drive / Outdated drivers / Recent Windows update conflict / Overheating due to dust buildup]

**Resolution Applied:**
[Actions taken: Freed X GB disk space via cleanup / Disabled non-essential startup programs / Removed malware/unwanted software / Updated [specific driver] / Defragmented drive / Adjusted visual effects for performance / Cleaned dust from vents / Uninstalled problematic update KB######]. Restarted workstation and verified improved performance with user.

**User Communication:**
Explained cause of performance degradation. Advised user on [maintaining free disk space / closing unused applications / avoiding installation of unnecessary software / regular restarts]. User confirmed significant performance improvement.

**Follow-up Required:**
[None / Monitor for performance regression / Schedule hard drive replacement / Recommend RAM upgrade / Check for recurring issue in one week]

---

## Related Resources
- [Link to: Malware Removal Procedure]
- [Link to: Windows Update Troubleshooting]
- [Link to: Hardware Replacement Process]
- Microsoft Docs: [Windows Performance Troubleshooting](https://learn.microsoft.com/en-us/windows/client-management/troubleshoot-windows-performance)
- [Internal KB: Standard Hardware Specifications]
- [Internal KB: Approved Software List]

---

## Escalation Criteria
Escalate to Desktop Engineering / Hardware Team if:
- Hardware failure suspected (failing drive, memory errors, overheating)
- Performance issues persist after all software troubleshooting completed
- Upgrade required (RAM, SSD) that exceeds help desk scope
- Multiple workstations in same area experiencing similar issues (possible network/server problem)
- Advanced malware requiring specialized removal tools
- Issue appears to be application-specific and requires developer input

---

## Notes and Tips
- **Pro Tip:** Start with quick wins (restart, disk cleanup, disable startup programs) before deep troubleshooting - often solves 60% of cases
- **Time-Saver:** 100% disk usage on mechanical drives is the most common cause - check this first
- **Common Pitfall:** Don't recommend registry cleaners or PC optimizer software - they often cause more problems
- **Prevention:** Educate users to restart at least weekly and keep 15-20% free disk space
- **Best Practice:** Document baseline performance metrics when resolving issues for future comparison
- **Quick Win:** Many "slow computer" complaints are actually "specific application slow" - narrow the scope early
- **Hardware Note:** If workstation is 5+ years old with mechanical drive and 4GB RAM, performance issues may be hardware limitations rather than software problems - set appropriate expectations

**Common Performance Benchmarks:**
- **Good boot time:** Under 2 minutes from power on to usable desktop
- **Application launch:** Under 10 seconds for Office applications
- **Idle CPU:** Under 10% with no user activity
- **Idle RAM:** 40-60% usage on 8GB system
- **Disk:** Should not sustain 100% usage during normal operations

---

**Last Updated:** January 2025  
**Author:** Lisa Wicklein  
**Version:** 1.0