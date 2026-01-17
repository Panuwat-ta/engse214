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

