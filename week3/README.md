# Vulnerabilities & Basic Risk Assessment

### Case Study: วิเคราะห์ช่องโหว่ในระบบ Login
#### สถานการณ์:
คุณเป็น Software Engineer ที่ได้รับมอบหมายให้ตรวจสอบความปลอดภัยของระบบ Login ของบริษัท
```py
// โค้ดระบบ Login (ตัวอย่างที่มีช่องโหว่)
<?php
$username = $_POST['username'];
$password = $_POST['password'];

// ช่องโหว่ที่ 1: SQL Injection
$sql = "SELECT * FROM users WHERE username='$username' AND password='$password'";
$result = mysqli_query($conn, $sql);

if (mysqli_num_rows($result) > 0) {
    // ช่องโหว่ที่ 2: Session Management
    $_SESSION['user'] = $username;
    
    // ช่องโหว่ที่ 3: No Password Hashing
    // รหัสผ่านเก็บแบบ plaintext ในฐานข้อมูล
    
    echo "Login successful!";
} else {
    echo "Invalid credentials!";
}
?>
```

### แบบฝึกหัด: Risk Assessment

```
┌──────────────────────────────────────────────────────┐
│     Risk Assessment Worksheet                        │
└──────────────────────────────────────────────────────┘

1. IDENTIFY VULNERABILITIES
   ┌─────────────────────────────────────────────┐
   │ ช่องโหว่ที่พบ:                                 │
   │ V1: SQL Injection (Improper Neutralization) │
   │ V2: Cleartext Storage (No Password Hashing) │
   │ V3: Insecure Session Management             │
   └─────────────────────────────────────────────┘

2. ANALYZE THREATS
   ┌──────────────────────────────────────────┐
   │ Threat 1: SQL Injection Attack           │
   │ Threat Actor: External Hacker            │
   │ Attack Method: ใส่ชุดคำสั่ง SQL (Payload)    │
   │   เช่น ' OR '1'='1 ลงในช่อง Input เพื่อ      │
   │   หลอกให้ Database ทำงานตามคำสั่ง          │
   │                                          │
   │ Threat 2: Credential Theft / Leaks       │
   │ Threat Actor: Hacker / Malicious Insider │
   │ Attack Method: การเข้าถึงฐานข้อมูลโดยตรง      │
   │   (ผ่าน SQLi หรือ Local Access) แล้วอ่าน     │
   │   รหัสผ่านได้ทันทีโดยไม่ต้องเสียเวลา Crack       │
   └──────────────────────────────────────────┘

3. EVALUATE IMPACT & LIKELIHOOD
   ┌─────────────────────────────────────────────┐
   │ Vulnerability | Impact | Likelihood | Risk  │
   ├───────────────┼────────┼────────────┼───────┤
   │ SQL Injection │ High   │ High       │ HIGH  │
   │ No Hash       │ High   │ Medium     │ HIGH  │
   │ Session Mgmt  │ Medium │ High       │ HIGH  │
   └─────────────────────────────────────────────┘

4. RISK TREATMENT RECOMMENDATIONS
   ┌────────────────────────────────────────────┐
   │ V1 - SQL Injection:                        │
   │ • ใช้ Prepared Statements (PDO/MySQLi)      │
   │ • ทำ Input Validation (Filter data)        │
   │ • ใช้ ORM Framework (Eloquent/Doctrine)     │
   │                                            │
   │ V2 - No Password Hashing:                  │
   │ • ใช้ฟังก์ชัน password_hash() (Bcrypt/Argon2)  │
   │ • ห้ามใช้ MD5 หรือ SHA1 เก่าๆ                   │
   │                                             │
   │ V3 - Session Management:                    │
   │ • ใช้ session_regenerate_id(true) หลัง Login  │
   │ • ตั้งค่า Cookie params (HttpOnly, Secure)     │
   └─────────────────────────────────────────────┘

   ```

### Mini Assignment: Risk Assessment Exercise

#### โจทย์:
วิเคราะห์ระบบ "Online Booking System" ของโรงพยาบาลขนาดเล็ก ที่มีฟีเจอร์ดังนี้:

- ผู้ป่วยสามารถจองคิวออนไลน์
- เก็บข้อมูลส่วนตัว (ชื่อ, เบอร์โทร, อาการ)
- มี Admin panel สำหรับเจ้าหน้าที่

#### ให้นักศึกษา:
1. ระบุ Assets (อย่างน้อย 3 รายการ)
    ```
    Asset 1: ฐานข้อมูลผู้ป่วย (Patient Database - ชื่อ, เบอร์โทร, อาการ)
    Asset 2: ระบบ Web Application สำหรับการจอง (Availability ของระบบ)
    Asset 3: บัญชีผู้ใช้ Admin (Admin Credentials)
    ```

2. ระบุ Threats (อย่างน้อย 3 รายการ)
    ```
    Threat 1: Data Breach / Data Theft (แฮกเกอร์ขโมยข้อมูลส่วนตัวผู้ป่วยไปขาย)
    Threat 2: Brute Force Attack (การสุ่มเดารหัสผ่านเพื่อเข้าสู่ Admin Panel)
    Threat 3: Denial of Service (DDoS) (ทำให้ระบบล่ม ผู้ป่วยจองคิวไม่ได้)
    ```

3. ระบุ Vulnerabilities (อย่างน้อย 3 รายการ)
    ```
    Vulnerability 1: Weak Password Policy (อนุญาตให้ตั้งรหัสผ่านง่าย เช่น 123456)
    Vulnerability 2: Lack of Encryption (ส่งข้อมูลผ่าน HTTP ธรรมดา ไม่ใช้ HTTPS)
    Vulnerability 3: Outdated Software (Server หรือ CMS ไม่ได้รับ Security Patch)
    ```

4. ทำ Risk Assessment Matrix
   ```
    ┌──────────────┬────────┬───────────┬───────┬──────────┐
    │ Risk Item    │ Impact │ Likelihood│ Risk  │ Priority │
    │              │ (L/M/H)│  (L/M/H)  │ Level │  (1-9)   │
    ├──────────────┼────────┼───────────┼───────┼──────────┤
    │ SQL Injection│   H    │    H      │ HIGH  │    1     │
    ├──────────────┼────────┼───────────┼───────┼──────────┤
    │ Weak Admin PW│   H    │    M      │ HIGH  │    2     │
    ├──────────────┼────────┼───────────┼───────┼──────────┤
    │ No HTTPS (SSL│   M    │    M      │ MEDIUM│    3     │
    └──────────────┴────────┴───────────┴───────┴──────────┘
   ```

5. เสนอมาตรการลดความเสี่ยง

    ```
    Risk 1: Weak Admin Password (รหัสผ่านเดาง่าย)
    Mitigation:
    • บังคับใช้นโยบายรหัสผ่านที่รัดกุม (Strong Password Policy) ขั้นต่ำ 8-12 ตัวอักษร
    • เปิดใช้งาน Multi-Factor Authentication (MFA) สำหรับ Admin Panel
    • ใช้กลไก Account Lockout (ล็อกบัญชีหากใส่รหัสผิดเกิน 5 ครั้ง)

    Risk 2: Unencrypted Data Transmission (การดักจับข้อมูลผ่าน HTTP)
    Mitigation:
    • ติดตั้ง SSL/TLS Certificate และบังคับใช้ HTTPS ตลอดทั้งเว็บ
    • ตั้งค่า HTTP Strict Transport Security (HSTS) เพื่อบังคับ Browser ให้ใช้ Secure Connection เสมอ

    ```


