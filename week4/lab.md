# 🐧 **TASK1: LAB Assignment**

## **สัปดาห์ที่ 4: การรักษาความปลอดภัยของระบบปฏิบัติการ (2วัน)**

**ข้อกำหนด:**
- ทำงานเป็นกลุ่ม 2-3 คน
- แต่ละคนต้องมี environment ส่วนตัวในการทำ LAB
- **ทำ Lab 1 หรือ Lab 2** บน Virtual Machine (เช่น Ubuntu Server และ Windows Server/10) และจัดทำรายงานสรุปผลการตั้งค่า
- รายงานต้องระบุส่วนงานของแต่ละคน
- Presentation 15 นาที (optional bonus 5 คะแนน)
- กำหนดส่ง สัปดาห์ถัดไป
---

## 🐧 **LAB 1: Linux Security Configuration (Day 1)**

### **Prerequisites:**
- Ubuntu 20.04/22.04 LTS หรือ CentOS 8/9
- Root access หรือ sudo privileges
- Network connectivity

---

### **Task 1: สร้าง User Accounts สำหรับ Team (30 นาที)**

**1.1 สร้าง Users และ Groups:**
```bash
# สร้าง groups
sudo groupadd developers
sudo groupadd testers
sudo groupadd dbadmin

# สร้าง users
sudo useradd -m -s /bin/bash -G developers alice
sudo useradd -m -s /bin/bash -G developers bob
sudo useradd -m -s /bin/bash -G testers charlie
sudo useradd -m -s /bin/bash -G dbadmin david

# ตั้งรหัสผ่าน (ต้องตาม policy)
sudo passwd alice
sudo passwd bob
sudo passwd charlie
sudo passwd david
```

**1.2 ตั้งค่า Password Policy:**
```bash
# แก้ไขไฟล์ /etc/login.defs
sudo nano /etc/login.defs

# เปลี่ยนค่าเหล่านี้:
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
PASS_MIN_LEN    12

# ติดตั้ง libpam-pwquality
sudo apt install libpam-pwquality

# แก้ไข /etc/pam.d/common-password
sudo nano /etc/pam.d/common-password
# เพิ่มบรรทัด:
password requisite pam_pwquality.so retry=3 minlen=12 difok=3 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1
```

**ที่ต้องจับภาพ:**
- คำสั่ง `cat /etc/passwd | tail -4`
- คำสั่ง `groups alice bob charlie david`
- การทดสอบ password policy

---

### **Task 2: ตั้งค่า Sudo Permissions (45 นาที)**

**2.1 สร้าง Sudo Groups:**
```bash
# สร้าง custom sudo groups
sudo groupadd sudo-developers
sudo groupadd sudo-limited

# เพิ่ม users เข้า groups
sudo usermod -aG sudo-developers alice
sudo usermod -aG sudo-developers bob
sudo usermod -aG sudo-limited charlie
```

**2.2 Configure Sudoers:**
```bash
# แก้ไขไฟล์ sudoers
sudo visudo

# เพิ่มกฎเหล่านี้:
# Developers - full sudo access
%sudo-developers ALL=(ALL:ALL) ALL

# Limited sudo - specific commands only
%sudo-limited ALL=(ALL) /usr/bin/systemctl status *, /usr/bin/tail /var/log/*, /bin/ps

# Database admin - database commands only
david ALL=(ALL) /usr/bin/mysql, /usr/bin/mysqldump, /bin/systemctl restart mysql

# Sudo session timeout (15 minutes)
Defaults timestamp_timeout=15

# Log sudo commands
Defaults logfile="/var/log/sudo.log"
Defaults log_input, log_output
```

**2.3 ทดสอบ Sudo Permissions:**
```bash
# ทดสอบด้วย alice
sudo -u alice sudo ls /root

# ทดสอบด้วย charlie (ควรใช้ได้เฉพาะคำสั่งที่อนุญาต)
sudo -u charlie sudo systemctl status ssh
sudo -u charlie sudo apt update  # ควร fail
```

**ที่ต้องจับภาพ:**
- ไฟล์ `/etc/sudoers` (เฉพาะส่วนที่เพิ่ม)
- ผลการทดสอบ sudo permissions
- Log file `/var/log/sudo.log`

---

### **Task 3: Configure SSH Security (45 นาที)**

**3.1 Backup และแก้ไข SSH Config:**
```bash
# Backup original config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# แก้ไข SSH configuration
sudo nano /etc/ssh/sshd_config

# เปลี่ยนค่าเหล่านี้:
Port 2222                          # เปลี่ยนจาก default port
PermitRootLogin no                 # ห้าม root login
PasswordAuthentication yes          # อนุญาต password (ชั่วคราว)
PubkeyAuthentication yes           # เปิดใช้ key-based auth
MaxAuthTries 3                     # จำกัดความพยายาม
ClientAliveInterval 300            # Timeout session
ClientAliveCountMax 2              # Max idle sessions
AllowUsers alice bob charlie david  # อนุญาตเฉพาะ users เหล่านี้
Protocol 2                         # ใช้ SSH Protocol 2
```

**3.2 สร้าง SSH Keys:**
```bash
# สร้าง SSH key pair สำหรับ alice
sudo -u alice ssh-keygen -t rsa -b 4096 -C "alice@company.com"

# Copy public key (สำหรับทดสอบ)
sudo -u alice cp /home/alice/.ssh/id_rsa.pub /home/alice/.ssh/authorized_keys
sudo -u alice chmod 600 /home/alice/.ssh/authorized_keys
```

**3.3 Configure SSH Banner:**
```bash
# สร้าง warning banner
sudo nano /etc/ssh/ssh_banner.txt

# เนื้อหา banner:
*******************************************
WARNING: Authorized access only!
All connections are monitored and recorded.
Disconnect immediately if you are not an
authorized user.
*******************************************

# เพิ่มใน sshd_config
Banner /etc/ssh/ssh_banner.txt
```

**3.4 Restart SSH และทดสอบ:**
```bash
# ทดสอบ config ก่อน restart
sudo sshd -t

# Restart SSH service
sudo systemctl restart sshd

# ทดสอบการเชื่อมต่อ
ssh -p 2222 alice@localhost
```

**ที่ต้องจับภาพ:**
- ไฟล์ `/etc/ssh/sshd_config` (ส่วนที่แก้ไข)
- การทดสอบ SSH connection
- SSH banner message

---

### **Task 4: Set up Firewall Rules (30 นาที)**

**4.1 Configure UFW:**
```bash
# Reset UFW to default
sudo ufw --force reset

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (new port)
sudo ufw allow 2222/tcp

# Allow web services
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow specific IPs only for SSH (optional)
# sudo ufw allow from 192.168.1.0/24 to any port 2222

# Enable UFW
sudo ufw enable

# Show status
sudo ufw status verbose
```

**4.2 Advanced UFW Rules:**
```bash
# Rate limiting for SSH
sudo ufw limit 2222/tcp

# Allow MySQL only from specific network
sudo ufw allow from 192.168.1.0/24 to any port 3306

# Log all denied connections
sudo ufw logging on

# Show numbered rules
sudo ufw status numbered
```

**ที่ต้องจับภาพ:**
- `sudo ufw status verbose`
- `sudo ufw status numbered`
- ไฟล์ log ใน `/var/log/ufw.log`

---

### **Task 5: Enable System Monitoring (60 นาที)**

**5.1 Install Monitoring Tools:**
```bash
# Install required packages
sudo apt update
sudo apt install fail2ban logwatch sysstat htop iotop

# Install ELK stack components (optional)
sudo apt install elasticsearch logstash kibana
```

**5.2 Configure Fail2Ban:**
```bash
# Backup original config
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.conf.backup

# สร้าง local config
sudo nano /etc/fail2ban/jail.local

# เนื้อหาไฟล์:
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3
backend = systemd

[sshd]
enabled = true
port = 2222
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

[apache-auth]
enabled = true
port = http,https
logpath = /var/log/apache2/error.log

[apache-badbots]
enabled = true
port = http,https
logpath = /var/log/apache2/access.log
bantime = 86400
maxretry = 1
```

**5.3 Configure System Monitoring:**
```bash
# Enable sysstat
sudo systemctl enable sysstat
sudo systemctl start sysstat

# Create monitoring script
sudo nano /usr/local/bin/system_monitor.sh

#!/bin/bash
# System monitoring script
DATE=$(date)
echo "=== System Monitor Report - $DATE ===" >> /var/log/system_monitor.log

# CPU Usage
echo "CPU Usage:" >> /var/log/system_monitor.log
top -bn1 | grep "Cpu(s)" >> /var/log/system_monitor.log

# Memory Usage
echo "Memory Usage:" >> /var/log/system_monitor.log
free -h >> /var/log/system_monitor.log

# Disk Usage
echo "Disk Usage:" >> /var/log/system_monitor.log
df -h >> /var/log/system_monitor.log

# Active Users
echo "Active Users:" >> /var/log/system_monitor.log
who >> /var/log/system_monitor.log

# Failed Login Attempts
echo "Recent Failed Logins:" >> /var/log/system_monitor.log
tail -10 /var/log/auth.log | grep "Failed password" >> /var/log/system_monitor.log

echo "================================" >> /var/log/system_monitor.log

# Make executable
sudo chmod +x /usr/local/bin/system_monitor.sh

# Add to crontab (run every hour)
sudo crontab -e
# เพิ่มบรรทัด:
0 * * * * /usr/local/bin/system_monitor.sh
```

**5.4 Configure Log Rotation:**
```bash
# Create logrotate config
sudo nano /etc/logrotate.d/system_monitor

/var/log/system_monitor.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    copytruncate
}
```

**ที่ต้องจับภาพ:**
- `sudo fail2ban-client status`
- `sudo fail2ban-client status sshd`
- ไฟล์ `/var/log/system_monitor.log`
- `sudo systemctl status fail2ban`

---

### **End of Day 1 Checklist:**
- [ ] Users และ groups ถูกสร้างแล้ว
- [ ] Password policy ทำงานได้
- [ ] Sudo permissions ถูกต้อง
- [ ] SSH security configured
- [ ] Firewall rules active
- [ ] Monitoring tools installed และ configured
- [ ] All screenshots captured
- [ ] Services ทั้งหมดทำงานได้

**รายงานที่ต้องส่งสำหรับ Day 1:**
- Screenshots ทุกขั้นตอน
- Configuration files ที่แก้ไข
- ผลการทดสอบแต่ละ task
- ปัญหาที่พบและวิธีแก้ไข

---

## 🪟 **LAB 2: Windows Security Hardening (Day 2)**

### **Prerequisites:**
- Windows 10/11 Pro หรือ Windows Server 2019/2022
- Administrator privileges
- PowerShell 5.1 หรือสูงกว่า

---

### **Task 1: Configure User Account Control (UAC) (45 นาที)**

**1.1 ตรวจสอบสถานะ UAC ปัจจุบัน:**
```powershell
# เปิด PowerShell as Administrator
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" | Select-Object EnableLUA, ConsentPromptBehaviorAdmin, ConsentPromptBehaviorUser

# ตรวจสอบ UAC Level
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name ConsentPromptBehaviorAdmin
```

**1.2 สร้าง User Accounts:**
```powershell
# สร้าง local users
$SecurePassword = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

New-LocalUser -Name "DevUser1" -Password $SecurePassword -Description "Developer User 1" -PasswordNeverExpires:$false
New-LocalUser -Name "TestUser1" -Password $SecurePassword -Description "Test User 1" -PasswordNeverExpires:$false
New-LocalUser -Name "AdminUser1" -Password $SecurePassword -Description "Admin User 1" -PasswordNeverExpires:$false

# เพิ่ม users เข้า groups
Add-LocalGroupMember -Group "Users" -Member "DevUser1", "TestUser1"
Add-LocalGroupMember -Group "Administrators" -Member "AdminUser1"

# ตรวจสอบ users
Get-LocalUser | Where-Object {$_.Name -like "*User*"}
```

**1.3 Configure UAC Settings:**
```powershell
# Set UAC to highest level
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name ConsentPromptBehaviorAdmin -Value 2
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name ConsentPromptBehaviorUser -Value 3
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name EnableLUA -Value 1

# Enable Admin Approval Mode
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name FilterAdministratorToken -Value 1

# Restart required for changes to take effect
# Restart-Computer -Force
```

**ที่ต้องจับภาพ:**
- PowerShell output ของ UAC settings
- User accounts ที่สร้าง
- UAC Control Panel settings

---

### **Task 2: Set up Group Policy (60 นาที)**

**2.1 Configure Local Security Policy:**
```powershell
# เปิด Local Security Policy
secpol.msc

# หรือใช้ PowerShell commands:
# Password Policy
secedit /export /cfg C:\temp\current_policy.inf

# Account Lockout Policy via Registry
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\RemoteAccess\Parameters\AccountLockout" -Name MaxDenials -Value 3
```

**2.2 Password Policy Configuration:**
```batch
REM Create batch file for password policy
echo [System Access] > C:\temp\password_policy.inf
echo MinimumPasswordAge = 7 >> C:\temp\password_policy.inf
echo MaximumPasswordAge = 90 >> C:\temp\password_policy.inf
echo MinimumPasswordLength = 12 >> C:\temp\password_policy.inf
echo PasswordComplexity = 1 >> C:\temp\password_policy.inf
echo PasswordHistorySize = 5 >> C:\temp\password_policy.inf
echo LockoutBadCount = 3 >> C:\temp\password_policy.inf
echo LockoutDuration = 30 >> C:\temp\password_policy.inf
echo ResetLockoutCount = 30 >> C:\temp\password_policy.inf

REM Apply policy
secedit /configure /db C:\temp\secedit.sdb /cfg C:\temp\password_policy.inf
```

**2.3 User Rights Assignment:**
```powershell
# Grant specific rights (requires Carbon PowerShell module or manual GPO)
# Download and install Carbon module first:
# Install-Module -Name Carbon -Force

# Example user rights (manual configuration via secpol.msc):
# - Log on as a service: Assign to service accounts only
# - Log on locally: Assign to specific users
# - Backup files and directories: Assign to backup operators
```

**2.4 Security Options:**
```powershell
# Disable Guest Account
Disable-LocalUser -Name "Guest"

# Set interactive logon message
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name LegalNoticeCaption -Value "WARNING"
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name LegalNoticeText -Value "This system is for authorized users only. All activities are monitored."

# Rename Administrator account
Rename-LocalUser -Name "Administrator" -NewName "SystemAdmin"

# Disable unnecessary services
Set-Service -Name "Telnet" -StartupType Disabled -ErrorAction SilentlyContinue
Set-Service -Name "SimpleGCP" -StartupType Disabled -ErrorAction SilentlyContinue
```

**ที่ต้องจับภาพ:**
- Local Security Policy screenshots
- Password policy settings
- User rights assignments
- Security options

---

### **Task 3: Enable Windows Defender (45 นาที)**

**3.1 Check Windows Defender Status:**
```powershell
# Check Defender status
Get-MpComputerStatus | Select-Object AntivirusEnabled, RealTimeProtectionEnabled, OnAccessProtectionEnabled

# Check Defender preferences
Get-MpPreference | Select-Object DisableRealtimeMonitoring, DisableBehaviorMonitoring, DisableBlockAtFirstSeen
```

**3.2 Configure Windows Defender:**
```powershell
# Enable all protection features
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -DisableBehaviorMonitoring $false
Set-MpPreference -DisableBlockAtFirstSeen $false
Set-MpPreference -DisableIOAVProtection $false
Set-MpPreference -DisablePrivacyMode $false
Set-MpPreference -DisableIntrusionPreventionSystem $false
Set-MpPreference -DisableScriptScanning $false

# Set scan schedule
Set-MpPreference -ScanScheduleDay Everyday
Set-MpPreference -ScanScheduleTime 02:00:00

# Configure exclusions (if needed)
# Add-MpPreference -ExclusionPath "C:\TrustedFolder"

# Update signatures
Update-MpSignature

# Run quick scan
Start-MpScan -ScanType QuickScan
```

**3.3 Configure Windows Defender Firewall:**
```powershell
# Get firewall profile status
Get-NetFirewallProfile | Select-Object Name, Enabled

# Enable all firewall profiles
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# Configure firewall logging
Set-NetFirewallProfile -Profile Domain,Public,Private -LogAllowed True -LogBlocked True -LogMaxSizeKilobytes 4096

# View current firewall rules
Get-NetFirewallRule | Where-Object {$_.Enabled -eq 'True'} | Select-Object DisplayName, Direction, Action | Sort-Object DisplayName
```

**3.4 Advanced Threat Protection:**
```powershell
# Enable Windows Defender Application Guard (if available)
Enable-WindowsOptionalFeature -Online -FeatureName Windows-Defender-ApplicationGuard

# Configure exploit protection
Set-ProcessMitigation -System -Enable DEP,SEHOP,ForceRelocateImages,RequireInfo

# Check exploit protection status
Get-ProcessMitigation -System
```

**ที่ต้องจับภาพ:**
- Windows Defender status
- Firewall profile settings
- Scan results
- Exploit protection settings

---

### **Task 4: Configure Firewall Rules (45 นาที)**

**4.1 Basic Firewall Configuration:**
```powershell
# Create inbound rules
New-NetFirewallRule -DisplayName "Allow HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
New-NetFirewallRule -DisplayName "Allow HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
New-NetFirewallRule -DisplayName "Allow SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow

# Create outbound rules
New-NetFirewallRule -DisplayName "Block Telnet Outbound" -Direction Outbound -Protocol TCP -RemotePort 23 -Action Block
New-NetFirewallRule -DisplayName "Block FTP Outbound" -Direction Outbound -Protocol TCP -RemotePort 21 -Action Block

# Allow specific applications
New-NetFirewallRule -DisplayName "Allow Notepad" -Direction Inbound -Program "C:\Windows\System32\notepad.exe" -Action Allow
```

**4.2 Advanced Firewall Rules:**
```powershell
# Create rules with specific source/destination
New-NetFirewallRule -DisplayName "Allow RDP from LAN" -Direction Inbound -Protocol TCP -LocalPort 3389 -RemoteAddress 192.168.1.0/24 -Action Allow

# Block specific IP ranges
New-NetFirewallRule -DisplayName "Block Suspicious Range" -Direction Inbound -RemoteAddress 10.0.0.0/8 -Action Block

# Create rules with time restrictions (requires Group Policy)
# This needs to be done via Advanced Windows Firewall with Security

# View specific rules
Get-NetFirewallRule -DisplayName "*HTTP*" | Select-Object DisplayName, Enabled, Direction, Action
```

**4.3 Firewall Monitoring:**
```powershell
# Enable firewall logging
Set-NetFirewallProfile -All -LogFileName "C:\Windows\System32\LogFiles\Firewall\pfirewall.log"
Set-NetFirewallProfile -All -LogMaxSizeKilobytes 4096
Set-NetFirewallProfile -All -LogAllowed True
Set-NetFirewallProfile -All -LogBlocked True

# View recent firewall events
Get-WinEvent -LogName "Microsoft-Windows-Windows Firewall With Advanced Security/Firewall" -MaxEvents 10
```

**ที่ต้องจับภาพ:**
- Firewall rules list
- Firewall profile settings
- Firewall log file content
- Windows Firewall with Advanced Security console

---

### **Task 5: Set up Event Monitoring (60 นาที)**

**5.1 Configure Event Logging:**
```powershell
# Enable audit policies
auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable
auditpol /set /category:"Account Logon" /success:enable /failure:enable
auditpol /set /category:"Account Management" /success:enable /failure:enable
auditpol /set /category:"Policy Change" /success:enable /failure:enable
auditpol /set /category:"Privilege Use" /success:enable /failure:enable

# Check current audit policy
auditpol /get /category:*
```

**5.2 Monitor Security Events:**
```powershell
# Create monitoring script
$MonitorScript = @'
# Security Event Monitoring Script
$Events = @()

# Failed logon attempts (Event ID 4625)
$FailedLogons = Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625; StartTime=(Get-Date).AddHours(-1)} -ErrorAction SilentlyContinue
if ($FailedLogons) {
    $Events += "Failed Logon Attempts: " + $FailedLogons.Count
}

# Successful logon attempts (Event ID 4624)
$SuccessLogons = Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624; StartTime=(Get-Date).AddHours(-1)} -ErrorAction SilentlyContinue
if ($SuccessLogons) {
    $Events += "Successful Logon Attempts: " + $SuccessLogons.Count
}

# Account lockouts (Event ID 4740)
$Lockouts = Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4740; StartTime=(Get-Date).AddHours(-1)} -ErrorAction SilentlyContinue
if ($Lockouts) {
    $Events += "Account Lockouts: " + $Lockouts.Count
}

# Policy changes (Event ID 4719)
$PolicyChanges = Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4719; StartTime=(Get-Date).AddHours(-1)} -ErrorAction SilentlyContinue
if ($PolicyChanges) {
    $Events += "Policy Changes: " + $PolicyChanges.Count
}

# Write to log file
$LogEntry = "$(Get-Date): " + ($Events -join ", ")
$LogEntry | Out-File -FilePath "C:\SecurityLogs\MonitoringLog.txt" -Append

# Display on console
Write-Host "Security Monitoring Report - $(Get-Date)"
$Events | ForEach-Object { Write-Host $_ }
'@

# Create directory and save script
New-Item -ItemType Directory -Path "C:\SecurityLogs" -Force
$MonitorScript | Out-File -FilePath "C:\SecurityLogs\Monitor.ps1"

# Create scheduled task for monitoring
$Action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File C:\SecurityLogs\Monitor.ps1"
$Trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Hours 1)
$Principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount
$Settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable

Register-ScheduledTask -TaskName "SecurityMonitoring" -Action $Action -Trigger $Trigger -Principal $Principal -Settings $Settings
```

**5.3 Performance Monitoring:**
```powershell
# Enable Performance Counters
$Counters = @(
    "\Processor(_Total)\% Processor Time",
    "\Memory\Available MBytes",
    "\LogicalDisk(_Total)\% Free Space",
    "\Network Interface(*)\Bytes Total/sec"
)

# Create performance monitoring script
$PerfScript = @'
$Date = Get-Date
$CPUUsage = (Get-Counter "\Processor(_Total)\% Processor Time").CounterSamples.CookedValue
$MemoryAvailable = (Get-Counter "\Memory\Available MBytes").CounterSamples.CookedValue
$DiskFree = (Get-Counter "\LogicalDisk(_Total)\% Free Space").CounterSamples.CookedValue

$LogEntry = "$Date,CPU:$([math]::Round($CPUUsage,2))%,Memory:$([math]::Round($MemoryAvailable,2))MB,DiskFree:$([math]::Round($DiskFree,2))%"
$LogEntry | Out-File -FilePath "C:\SecurityLogs\PerformanceLog.csv" -Append
'@

$PerfScript | Out-File -FilePath "C:\SecurityLogs\PerfMonitor.ps1"

# Schedule performance monitoring
$PerfAction = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File C:\SecurityLogs\PerfMonitor.ps1"
$PerfTrigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 15)

Register-ScheduledTask -TaskName "PerformanceMonitoring" -Action $PerfAction -Trigger $PerfTrigger -Principal $Principal -Settings $Settings
```

**5.4 Event Log Configuration:**
```powershell
# Increase Security log size
wevtutil sl Security /ms:104857600  # 100MB

# Configure log retention
wevtutil sl Security /rt:false      # Do not overwrite

# Configure Application log
wevtutil sl Application /ms:52428800 # 50MB

# Configure System log
wevtutil sl System /ms:52428800     # 50MB

# Export current security events for baseline
wevtutil epl Security C:\SecurityLogs\SecurityBaseline.evtx
```

**ที่ต้องจับภาพ:**
- Event Viewer showing security events
- Scheduled tasks for monitoring
- Log files created
- Performance counter data

---

### **End of Day 2 Checklist:**
- [ ] UAC configured to highest security level
- [ ] Local security policies applied
- [ ] Windows Defender fully enabled and configured
- [ ] Firewall rules created and tested
- [ ] Event monitoring and logging enabled
- [ ] Scheduled tasks for monitoring created
- [ ] All screenshots captured
- [ ] Performance baseline established

---

### **Final Deliverables (Due: 2 วัน)**

**📋 รายงานรวม (10-15 หน้า):**

### **Final Deliverables (Due: 2 วัน)**

**📋 รายงานรวม (10-15 หน้า):**

**1. Executive Summary (1 หน้า)**
- สรุปการทำ LAB ทั้ง 2 วัน
- ปัญหาหลักที่พบและการแก้ไข
- ระดับความปลอดภัยที่ได้หลังการตั้งค่า
- คำแนะนำสำหรับการใช้งานจริง

**2. Day 1: Linux Security Implementation (4-5 หน้า)**
- **User Management Results**
  - ตารางแสดง users และ groups ที่สร้าง
  - Password policy ที่ใช้และผลการทดสอบ
  - Screenshots ของการจัดการ user accounts
  
- **Sudo Configuration**
  - กฎ sudo ที่ตั้งค่า
  - ผลการทดสอบสิทธิ์แต่ละ user
  - Log entries ที่เกี่ยวข้อง
  
- **SSH Security**
  - การตั้งค่า SSH configuration
  - SSH key management
  - ผลการทดสอบการเชื่อมต่อ
  
- **Firewall Setup**
  - UFW rules ที่สร้าง
  - ผลการทดสอบ firewall
  - Log analysis
  
- **System Monitoring**
  - Tools ที่ติดตั้งและการตั้งค่า
  - Fail2ban configuration และผล
  - Monitoring scripts และ cron jobs

**3. Day 2: Windows Security Implementation (4-5 หน้า)**
- **UAC Configuration**
  - UAC settings และ justification
  - User account creation และ group assignments
  - Screenshots ของ UAC prompts
  
- **Group Policy Implementation**
  - Local security policies ที่ตั้งค่า
  - Password และ account lockout policies
  - Security options และผลกระทบ
  
- **Windows Defender Setup**
  - Defender configuration และ status
  - Firewall rules และ testing results
  - Scan results และ threat detection
  
- **Event Monitoring System**
  - Audit policies ที่เปิดใช้งาน
  - Monitoring scripts และ scheduled tasks
  - Event log analysis และ baseline

**4. Security Checklist (1-2 หน้า)**

**Linux Security Checklist:**
- [ ] Strong password policy enforced
- [ ] User accounts properly segregated
- [ ] Sudo access limited and logged
- [ ] SSH hardened (key-based auth, non-standard port)
- [ ] Firewall rules implemented and tested
- [ ] System monitoring active
- [ ] Fail2ban protecting against brute force
- [ ] Log rotation configured
- [ ] Regular security updates scheduled
- [ ] Unnecessary services disabled

**Windows Security Checklist:**
- [ ] UAC set to highest security level
- [ ] Strong password policy enforced
- [ ] Account lockout policy configured
- [ ] Guest account disabled
- [ ] Administrator account renamed
- [ ] Windows Defender fully enabled
- [ ] Firewall profiles active with custom rules
- [ ] Event auditing configured
- [ ] Security monitoring automated
- [ ] Unnecessary services disabled

**5. ข้อเสนอแนะสำหรับการปรับปรุง (1-2 หน้า)**

**Short-term Improvements (1-2 สัปดาห์):**
- **Linux:**
  - ติดตั้ง centralized logging (ELK Stack)
  - เพิ่ม intrusion detection system (OSSEC)
  - ตั้งค่า automated backup scripts
  - Configure SELinux/AppArmor
  
- **Windows:**
  - ติดตั้ง Windows Server Update Services (WSUS)
  - Configure BitLocker disk encryption
  - ตั้งค่า PowerShell script execution policy
  - เพิ่ม application whitelisting

**Medium-term Improvements (1-2 เดือน):**
- **Infrastructure:**
  - Implement Active Directory (Windows)
  - ตั้งค่า certificate authority
  - Deploy endpoint detection and response (EDR)
  - Configure network segmentation
  
- **Compliance:**
  - Conduct vulnerability assessments
  - Implement security incident response plan
  - ตั้งค่า compliance reporting
  - Regular security training

**Long-term Strategic Improvements (3-6 เดือน):**
- **Advanced Security:**
  - Zero-trust network architecture
  - Security orchestration and automation
  - Advanced threat hunting capabilities
  - Cloud security integration
  
- **Governance:**
  - Information security management system (ISMS)
  - Regular penetration testing
  - Third-party security audits
  - Continuous security improvement program

**6. Screenshots และหลักฐาน (ภาคผนวก)**
- ทุก screenshots ที่จำเป็นจาก LAB exercises
- Configuration files สำคัญ
- Log files samples
- Test results และ verification

---

### **การส่งงานและเกณฑ์การประเมิน**

**รูปแบบการส่ง:**
- ไฟล์ PDF รายงานหลัก
- โฟลเดอร์ Screenshots แยกตาม Day และ Task
- Configuration files ที่สำคัญ (.conf, .inf, .ps1)
- Security checklist ในรูปแบบ Excel หรือ PDF

**เกณฑ์การประเมิน (100 คะแนน):**
- **Technical Implementation (60 คะแนน)**
  - Day 1 Linux LAB: 30 คะแนน
  - Day 2 Windows LAB: 30 คะแนน
  
- **Documentation Quality (25 คะแนน)**
  - ความชัดเจนของรายงาน: 10 คะแนน
  - Screenshots และหลักฐาน: 10 คะแนน
  - Security checklist: 5 คะแนน
  
- **Analysis และ Recommendations (15 คะแนน)**
  - การวิเคราะห์ปัญหา: 7 คะแนน
  - ข้อเสนอแนะการปรับปรุง: 8 คะแนน

**ข้อกำหนดเพิ่มเติม:**
- **ทำงานเป็นกลุ่ม 2-3 คน**
- แต่ละคนต้องมี environment ส่วนตัวในการทำ LAB
- รายงานต้องระบุส่วนงานของแต่ละคน
- Presentation 15 นาที (optional bonus 5 คะแนน)

**Timeline:**
- **Day 1 (วันแรก):** Linux Security Configuration
- **Day 2 (วันที่สอง):** Windows Security Hardening  
- **Day 3-4:** จัดทำรายงานและ documentation
- **วันส่งงาน:** สัปดาห์หลังสอบกลางภาค

**หมายเหตุสำคัญ:**
- บันทึกทุกขั้นตอนด้วย screenshots
- เก็บ backup ของ configuration files
- ทดสอบการทำงานหลังจากตั้งค่าเสร็จทุกครั้ง
- อย่าลืม restore ระบบกลับสู่สภาพเดิมหลังเสร็จ LAB (ถ้าใช้ระบบจริง)

---

## **ทรัพยากรเพิ่มเติม**

**Documentation และ References:**
- Linux Security Documentation: https://www.kernel.org/doc/html/latest/admin-guide/security.html
- Windows Security Baselines: https://docs.microsoft.com/en-us/windows/security/
- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks/
- NIST Cybersecurity Framework: https://www.nist.gov/cyberframework

**Tools และ Software:**
- VirtualBox/VMware สำหรับสร้าง test environment
- Kali Linux สำหรับ security testing
- Windows 10/11 Evaluation: https://www.microsoft.com/en-us/evalcenter/
- Security scanning tools: Nessus, OpenVAS

**Learning Resources:**
- Linux Academy Security Courses
- Microsoft Learn Security Modules  
- SANS SEC401: Security Essentials
- CompTIA Security+ Study Materials


