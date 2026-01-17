# Week 4: Operating System Security & Hardening

## 🐧 TASK1: LAB Assignment
**ข้อกำหนด:**
- ทำงานเป็นกลุ่ม 2-3 คน
- แต่ละคนต้องมี environment ส่วนตัวในการทำ LAB
- **ทำ Lab 1 หรือ Lab 2** บน Virtual Machine (เช่น Ubuntu Server และ Windows Server/10) และจัดทำรายงานสรุปผลการตั้งค่า
- รายงานต้องระบุส่วนงานของแต่ละคน
- Presentation 15 นาที (optional bonus 5 คะแนน)

---
## 🐧 LAB 1: Linux Security Configuration
### Prerequisites:
- Ubuntu 20.04/22.04 LTS หรือ CentOS 8/9
- Root access หรือ sudo privileges
- Network connectivity

### Task 1: สร้าง User Accounts สำหรับ Team
#### 1.1 สร้าง Users และ Groups
![](./img/1-1.png)

#### 1.2 ตั้งค่า Password Policy:
#### แก้ไขไฟล์ /etc/login.defs
![](./img/1-2-1.png)
![](./img/1-2-2.png)

#### ติดตั้ง libpam-pwquality
![](./img/1-2-3.png)

#### แก้ไข /etc/pam.d/common-password
![](./img/1-2-4.png)

#### ที่ต้องจับภาพ:
![](./img/1.png)


### Task 2: ตั้งค่า Sudo Permissions

#### 2.1 สร้าง Sudo Groups:
![](./img/2-1.png)

#### 2.2 Configure Sudoers:
![](./img/2-2.png)

#### 2.3 ทดสอบ Sudo Permissions:
![](./img/2-3.png)

#### ที่ต้องจับภาพ:
![](./img/2.png)


### Task 3: Configure SSH Security

#### 3.1 Backup และแก้ไข SSH Config:
![](./img/3-1.png)

#### 3.2 สร้าง SSH Keys:
![](./img/3-2.png)

#### 3.3 Configure SSH Banner:
![](./img/3-3.png)

#### 3.4 Restart SSH และทดสอบ:
![](./img/3-4.png)

### Task 4: Set up Firewall Rules

#### 4.1 Configure UFW:
![](./img/4-1.png)

#### 4.2 Advanced UFW Rules:
![](./img/4-2.png)

#### ที่ต้องจับภาพ:
![](./img/4.png)

### Task 5: Enable System Monitoring

#### 5.1 Install Monitoring Tools:
![](./img/5-1.png)

#### 5.2 Configure Fail2Ban:
![](./img/5-2.png)

#### 5.3 Configure System Monitoring:
![](./img/5-3.png)

#### 5.4 Configure Log Rotation:
![](./img/5.png)

#### ที่ต้องจับภาพ: