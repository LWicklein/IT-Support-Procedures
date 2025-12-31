# Active Directory Account Lockout Response

## Overview
**Category:** Active Directory / User Management  
**Severity:** Medium to High (user cannot work)  
**Estimated Resolution Time:** 10-20 minutes  
**Required Access Level:** Help Desk with AD unlock permissions or Domain Admin

### Description
User account has been locked out of Active Directory domain, preventing login to workstation, email, network resources, and enterprise applications. This is one of the most common support requests and requires quick resolution to minimize productivity impact.

### Common Symptoms
- User cannot log into Windows workstation (error: "Your account has been locked")
- "Account is locked out" message when attempting domain login
- Authentication failures across all domain-connected services
- User reports password suddenly "not working" despite entering correctly
- Multiple failed login attempts may have triggered automatic lockout

---

## Prerequisites
- [ ] Active Directory Users and Computers (ADUC) access or PowerShell with AD module
- [ ] User's full name or username (SAM account name)
- [ ] Verification of user identity (employee ID, manager name, or security questions)
- [ ] Access to domain controller or admin workstation with AD tools

---

## Resolution Steps

### Step 1: Verify User Identity and Gather Information
**Action:**
Before unlocking any account, verify you are speaking with the legitimate user or their authorized manager. Gather the following information:

- Username (domain\username format)
- Last successful login time (if user knows)
- Recent password changes
- Whether user has multiple devices or remote access enabled
- Any recent system updates or changes

**Expected Result:**
Confirmed identity and baseline information for troubleshooting.

**If this fails:**
Follow your organization's identity verification procedures. Do not unlock accounts without proper verification.

---

### Step 2: Locate and Check Account Status
**Action:**
Open Active Directory Users and Computers or use PowerShell to check account status.

**Using ADUC:**
1. Open Active Directory Users and Computers
2. Navigate to the OU containing the user account
3. Right-click the user → Properties
4. Go to the "Account" tab
5. Check for "Unlock account" checkbox

**Using PowerShell:**
```powershell
# Search for the user
Get-ADUser -Filter "SamAccountName -like '*username*'" -Properties LockedOut, LastBadPasswordAttempt, BadPwdCount, PasswordLastSet

# Check specific user lockout status
Get-ADUser username -Properties LockedOut, LastBadPasswordAttempt, BadPwdCount | Select Name, LockedOut, LastBadPasswordAttempt, BadPwdCount

# Find where lockout occurred (requires admin permissions)
Get-ADUser username -Properties * | Select Name, LockedOut, LastBadPasswordAttempt, BadLogonCount
```

**Expected Result:**
Account shows as locked out, with timestamp of last bad password attempt and count of failed attempts.

**Troubleshooting:**
- If account doesn't appear locked in ADUC, check Group Policy lockout threshold settings
- If PowerShell command fails, verify AD module is installed: `Import-Module ActiveDirectory`
- Check multiple domain controllers for replication status if results seem inconsistent

---

### Step 3: Identify Lockout Source
**Action:**
Before unlocking, determine what is causing repeated lockouts to prevent immediate re-lockout.

**Common Lockout Sources:**
- Saved credentials in Windows Credential Manager (old password cached)
- Mobile device with old Exchange password (iPhone, Android)
- Mapped network drives with saved credentials
- Scheduled tasks running with old credentials
- Service accounts on servers using the user's credentials
- Browser saved passwords for internal sites
- VPN client with saved credentials
- Remote Desktop saved credentials

**Commands to Check:**
```powershell
# Find lockout source from domain controller security logs
# (Requires access to DC Event Viewer or centralized logging)
# Event ID 4740 shows which computer caused the lockout
```

**Expected Result:**
Identification of the device or service causing authentication failures.

**If you cannot identify source:**
Proceed with unlock but warn user that lockout may recur until source is found.

---

### Step 4: Unlock the Account
**Action:**
Unlock the account using ADUC or PowerShell.

**Using ADUC:**
1. In the user's Properties → Account tab
2. Check the box "Unlock account"
3. Click Apply, then OK
4. Wait 1-2 minutes for replication across domain controllers

**Using PowerShell:**
```powershell
# Unlock the account
Unlock-ADAccount -Identity username

# Verify unlock was successful
Get-ADUser username -Properties LockedOut | Select Name, LockedOut
```

**Expected Result:**
Account shows LockedOut = False, user can attempt login.

**Troubleshooting:**
- If account immediately locks again, the lockout source is still active (return to Step 3)
- If unlock doesn't take effect, check domain controller replication
- Verify you have sufficient permissions to unlock accounts

---

### Step 5: Address Root Cause
**Action:**
Guide user to resolve the lockout source based on your findings in Step 3.

**For Credential Manager Issues:**
```powershell
# Have user run on their workstation to clear cached credentials
rundll32.exe keymgr.dll,KRShowKeyMgr
# Then manually remove entries with old password
```

**For Mobile Device:**
- Have user remove and re-add Exchange account with current password
- On iPhone: Settings → Passwords & Accounts → [Work Account] → Delete
- On Android: Settings → Accounts → [Work Account] → Remove

**For Mapped Drives:**
```cmd
# Remove all mapped drives
net use * /delete /yes

# Or remove specific drive
net use X: /delete
```

**Expected Result:**
Lockout source is eliminated, preventing future automatic lockouts.

---

### Step 6: Password Reset (If Necessary)
**Action:**
If user genuinely forgot password or if required by policy after lockout:

```powershell
# Set new password (user must change at next login)
Set-ADAccountPassword -Identity username -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "TemporaryP@ssw0rd!" -Force)
Set-ADUser -Identity username -ChangePasswordAtLogon $true
```

**Expected Result:**
User has temporary password and must change it at next login.

---

## Verification
After completing resolution steps, verify:
- [ ] User can successfully log into workstation
- [ ] Email access is restored (Outlook connects)
- [ ] Network resources are accessible
- [ ] Account does not immediately lock out again
- [ ] User has updated all devices with current password

**Test with user on phone:**
"Please log out and log back in while I'm on the phone with you to confirm everything is working."

---

## Documentation Template
Use this when logging the ticket:

**Issue Summary:**
User [username] was locked out of Active Directory domain account and unable to access workstation and network resources.

**Root Cause:**
[Mobile device with stale credentials / Cached credentials in Credential Manager / Multiple failed password attempts / Unknown source - monitoring for recurrence]

**Resolution Applied:**
Account unlocked via [ADUC / PowerShell]. [Cleared cached credentials / Updated mobile device password / Reset password / Identified lockout source as X]. User verified successful login to all required systems.

**User Communication:**
Explained cause of lockout and provided guidance on updating credentials across all devices to prevent recurrence. Advised user to contact Help Desk immediately if lockout occurs again.

**Follow-up Required:**
[None / Monitor account for 24 hours for repeat lockouts / Follow up with user tomorrow to confirm no further issues]

---

## Related Resources
- [Link to: Password Reset Playbook]
- [Link to: Mobile Device Configuration Guide]
- [Link to: Group Policy Lockout Settings Documentation]
- Microsoft Docs: [Account Lockout Troubleshooting](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/usernames-passwords-account-lockout)

---

## Escalation Criteria
Escalate to Tier 2 / Systems Administrator if:
- Account locks out repeatedly within minutes after unlock (potential security incident)
- Lockout source cannot be identified after 30 minutes of investigation
- User's account shows suspicious activity (logins from unusual locations)
- Account is a service account or privileged admin account
- Multiple users in same department experiencing simultaneous lockouts (possible broader issue)

---

## Notes and Tips
- **Pro Tip:** Most lockouts are caused by mobile devices or cached credentials. Always ask about phones/tablets first.
- **Time-Saver:** Use PowerShell for bulk checks if multiple users are locked out
- **Common Pitfall:** Unlocking without finding the source leads to immediate re-lockout and frustrated users
- **Security Note:** Multiple lockouts can indicate brute force attack. Document suspicious patterns.
- **Best Practice:** After hours of normal business, check for scheduled tasks or services that might use the user's credentials

---

**Last Updated:** January 2025  
**Author:** Lisa Wicklein  
**Version:** 1.0