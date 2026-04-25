# Lab 1 — Basic Cyber Security Knowledge

แหล่งที่มา: https://play.mooc.ncsa.or.th/lab/1

---

## แนวคิดหลัก
ผู้ใช้ทั่วไปมักมองข้อมูลส่วนตัวเล็ก ๆ น้อย ๆ (โพสต์, ข้อมูลชิงรางวัล ฯลฯ) ว่าไม่สำคัญ แต่ hacker สามารถนำมา **ประติดประต่อ** เพื่อเจาะระบบได้ → ต้องคิดก่อนเปิดเผยข้อมูล

---

## 1. CIA Triad — องค์ประกอบพื้นฐานของ Information Security

| องค์ประกอบ | ความหมาย | ตัวอย่างการป้องกัน |
|---|---|---|
| **C**onfidentiality | ข้อมูลเป็นความลับ เฉพาะคนที่ได้รับอนุญาตเท่านั้นที่เข้าถึงได้ | Access Control, username/password, การแยกสิทธิ์ตาม service |
| **I**ntegrity | ข้อมูลถูกต้องสมบูรณ์ ไม่ถูกแก้ไขระหว่างส่ง/เก็บ | การเข้ารหัส, ตรวจสอบ MD5 (`md5sum` ใน Linux) |
| **A**vailability | ข้อมูล/ระบบพร้อมใช้เมื่อต้องการ | Load Balancing, Hardening, Backup Plan |

### + Accountability (เพิ่มเติม)
ยืนยันว่า "ใครทำอะไร" ในระบบได้ ประกอบด้วย:
- **Authentication** — พิสูจน์ว่าใคร
- **Authorization** — ทำอะไรได้บ้าง
- **Non-repudiation** — ปฏิเสธการกระทำของตัวเองไม่ได้ (ใช้เป็นหลักฐานในชั้นศาลได้)

---

## 2. เป้าหมายของการป้องกัน CIA

1. **Prevention** — ป้องกันการโจมตีที่รู้จักอยู่แล้ว (well-known threats) ด้วยการปรับ policy / hardening
2. **Detection** — ตรวจจับเหตุการณ์ผิดปกติ รวมถึงการโจมตีแบบใหม่ (เช่น 0day) ผ่าน log หรือ monitoring software → **ยากและสำคัญที่สุด**
3. **Response** — แผนรับมือและกู้คืนระบบหลังถูกโจมตี ควรมีหลายแผนสำรอง

---

## 3. ประเภทของ Exploit

> **Vulnerability** = ช่องโหว่ / ข้อผิดพลาดใน software ที่อาจนำไปสู่ความเสียหาย

| ประเภท | ลักษณะ |
|---|---|
| **Remote Exploit** | โจมตีผ่านเครือข่าย ไม่ต้องเข้าถึงหน้าเครื่อง |
| **Local Exploit** | ต้องเข้าถึงหน้าเครื่องก่อน เพื่อยกระดับสิทธิ์ (privilege escalation) |

---

## 4. เป้าหมายการโจมตีของ Hacker (3 แบบ ตรงกับ CIA)

### 4.1 Access Attack → ทำลาย **Confidentiality**
- **Dumpster Diving** — คุ้ยเอกสารจากถังขยะ
- **Eavesdropping** — ดักฟังโดยบังเอิญ
- **Snooping** — ดักฟังอย่างจงใจ (ติดเครื่องดักฟัง)
- **Interception** — ดูข้อมูลใน traffic โดยไม่ได้รับอนุญาต

### 4.2 Modification & Repudiation → ทำลาย **Integrity**
- **DNS Spoofing** — แก้ข้อมูลปลายทางใน DNS packet
- **Man-in-the-Middle (MITM)** — ดักจับแล้วแก้ข้อมูลก่อนส่งต่อ

### 4.3 DoS / DDoS → ทำลาย **Availability**
- **TCP SYN Flood** — ส่ง SYN packet ท่วมจน server หมด memory
- **Ping of Death** — ส่ง ICMP ขนาดใหญ่เกินมาตรฐาน → เครื่อง Hang / BSOD
- **Buffer Overflow** — ส่ง input เกิน buffer limit → crash service หรือ bypass auth
- **DDoS** — ใช้ Zombie Machines / Botnet หลายเครื่องโจมตีพร้อมกัน

---

## สรุปความสัมพันธ์ CIA ↔ การโจมตี

```
Confidentiality  ←  Access Attack
Integrity        ←  Modification & Repudiation
Availability     ←  DoS / DDoS
```

---

## คำศัพท์ที่ต้องจำ
- CIA Triad, Accountability, Authentication, Authorization, Non-repudiation
- Vulnerability, Exploit, 0day
- Remote vs Local Exploit
- DoS, DDoS, Botnet, Zombie Machine
- MITM, DNS Spoofing, SYN Flood, Buffer Overflow
