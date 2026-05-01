# Covert Channel — สรุป

## Covert Channel คืออะไร ⭐

- **Covert Channel** = การใช้ช่องทางสื่อสาร**ปกติ** เพื่อแฝงการติดต่อแบบลับ
- ใช้สำหรับ:
  - **หลบเลี่ยงการตรวจจับ** (network filter, firewall)
  - ติดต่อระหว่าง attacker ↔ เครื่องเหยื่อโดยไม่ให้ถูกจับได้
- **ไม่ใช่** การเข้ารหัส (cryptography) — ข้อมูลถูก **แฝง** ใน payload ของ protocol ปกติ
- เทคนิคใกล้เคียง: **Steganography** (การซ่อนข้อมูลในสื่ออื่น)
- ต้องเข้าใจ Protocol ที่ใช้อย่างลึกซึ้ง เพื่อหาช่องว่างใส่ข้อมูล

---

## ทำไม ICMP จึงเป็นที่นิยม ⭐

| เหตุผล | รายละเอียด |
|--------|------------|
| **ไม่มี Port** | ทำงาน Layer 3 (Network Layer) — ใช้แค่ IP |
| **Firewall มักอนุญาต** | เพราะใช้สำหรับเช็คเครื่อง online (ping) |
| **มีช่องใน packet** | Field `Message` (data) ใส่อะไรก็ได้ |

---

## โครงสร้าง ICMP Packet

**IP Datagram** ประกอบด้วย:
- **IP Header** — Source IP, Destination IP
- **ICMP Message** — ส่วนหลักที่ใช้แฝงข้อมูล

| Field | ขนาด | หน้าที่ |
|-------|------|---------|
| **Type** | 8 bits | ประเภท ICMP message (มี 15 แบบ) — ใช้บ่อย: `8` = Echo Request, `0` = Echo Reply |
| **Code** | 8 bits | รายละเอียดย่อยของ Type |
| **Checksum** | — | ตรวจ packet สมบูรณ์หรือไม่ |
| **Identifier** | — | หมายเลข datagram |
| **Sequence Number** | — | ลำดับ datagram (เริ่ม 0, +1 ทุกครั้ง) |
| **Message** | variable | ⭐ **ส่วนที่ใช้แฝงข้อมูล** |

---

## ตรวจสอบ ICMP ด้วย Wireshark

```bash
ping -c 1 192.168.77.157   # -c = จำนวน packet
```

**Default behavior:** ใส่ Timestamp + ข้อมูลขยะ ~48 bytes ใน Message

ถ้าอยากดู **packet เปล่า** (ไม่มี filler):
```bash
ping -s 0 192.168.77.157   # -s 0 = buffer = 0
```

---

## สร้าง Custom ICMP Content ด้วย Hping2

```bash
hping2 -1 -c 1 -e "Hello World" 192.168.77.157
```

| Option | ความหมาย |
|--------|----------|
| `-1` | ใช้ ICMP mode |
| `-c` | จำนวน packet |
| `-e` | **เพิ่มข้อความใน Message** ⭐ |

> ข้อความ `"Hello World"` จะถูกแนบต่อท้าย ICMP Message — ตรวจดูได้ใน Wireshark

---

## ตัวอย่างโปรแกรม ICMP Covert Channel ⭐

> **[เพิ่มโดย Claude]** Code ต้นฉบับเป็น **Python 2** (syntax `print "..."`, `except X, (a, b)`) — ปัจจุบัน Python 2 หมด support แล้ว ต้องแปลงเป็น Python 3 ก่อนใช้จริง (เปลี่ยน `print` → `print(...)`, `except X as e:`, `raw_input` → `input`)

### หลักการทำงาน

```
┌─────────┐                              ┌─────────┐
│ Client  │ ──── ICMP Echo Request ────► │ Server  │
│         │      data="Covert:<cmd>"     │         │
│         │                              │         │
│         │ ◄──── ICMP Echo Reply ────── │         │
│         │       data="Covert:<output>" │         │
│         │                              │         │
│         │ ◄──── "Covert:end of shell"──│         │
└─────────┘                              └─────────┘
```

### ส่วน Server

**โครงสร้าง code 3 ส่วน:**
1. `checksum()` — คำนวณ ICMP checksum
2. `response_icmp()` — ส่ง ICMP packet กลับ
3. `main()` — รอรับ packet, ดึง command, รันด้วย `os.popen()`, ส่งผลลัพธ์กลับ

**Logic:**
- Bind raw socket: `socket.SOCK_RAW, socket.IPPROTO_ICMP`
- รับ packet → ดึงข้อความที่ขึ้นต้นด้วย `Covert:` ด้วย regex
- รัน command, ส่งผลลัพธ์ทีละบรรทัดกลับ
- จบด้วย `"end of shell"` เป็น marker

### ส่วน Client

**โครงสร้าง:**
1. `checksum()` — เหมือน Server
2. `response_icmp()` — ส่ง packet ที่มี command
3. `receive_icmp()` — รอรับ output จนเจอ `"end of shell"`
4. `main()` — รับ Server IP + รับ command จาก user แบบ loop

### กลไกที่สำคัญ ⭐

| กลไก | จุดประสงค์ |
|------|-----------|
| **Prefix `"Covert:"`** ทุก message | กรองให้รู้ว่าเป็นข้อความ covert (ไม่ใช่ ICMP ปกติ) ป้องกัน parsing ผิดพลาด |
| **Marker `"end of shell"`** | ระบุจุดจบของ output จาก command |

---

## การตรวจจับ Covert Channel

### IDS — Snort
- มี **rule สำเร็จรูป** สำหรับดักจับ ICMP Covert Channel
- แต่ถ้า attacker **ปลอมข้อความให้ใกล้เคียงปกติ** อาจหลบ rule ได้
- ต้อง **เขียน custom rule** เพิ่มเอง

> **[เพิ่มโดย Claude]** เทคนิคตรวจจับสมัยใหม่:
> - **Statistical anomaly detection** — ICMP ปกติมี payload ขนาด/รูปแบบคงที่ การมี payload ขนาดผิดปกติ = สัญญาณ
> - **Baseline analysis** — เปรียบเทียบความถี่/ปริมาณ ICMP กับ baseline ของ network
> - **Entropy check** — payload ที่มี entropy สูงผิดปกติ = อาจเป็นข้อมูลที่เข้ารหัส/บีบอัด
> - **DPI (Deep Packet Inspection)** — Next-gen firewall ตรวจ ICMP payload pattern

---

## Covert Channel ใน Protocol อื่นๆ ⭐

| Protocol | เครื่องมือยอดนิยม |
|----------|-------------------|
| **ICMP** | Loki, ICMPTX |
| **DNS** ⭐ | **iodine**, **dnscat2** |
| **HTTP** ⭐ | Web Shell (เช่น China Chopper, b374k) |

### ทำไม DNS / HTTP ฮิตที่สุด
- DNS: ทุก network ต้องเปิด port 53 → หลบ firewall ได้แทบเสมอ
- HTTP: traffic ปกติของ web → glance ดูเหมือน traffic ทั่วไป

> **[เพิ่มโดย Claude]** Covert channel อื่นๆ ที่ควรรู้:
> - **TCP Timing Channel** — เข้ารหัสข้อมูลในช่วงเวลาที่ส่ง packet (timing-based)
> - **TCP Header Field abuse** — ใช้ Sequence Number, ISN, Urgent Pointer แฝงข้อมูล
> - **TLS SNI / DNS-over-HTTPS (DoH)** — ใช้ encrypted channel ปกติเป็นท่อ exfil
> - **Cloud service C2** — ใช้ Slack, Telegram, GitHub, Twitter API เป็นช่องทางสั่งงาน (เพราะ block ไม่ได้)

---

## สรุปแก่นสำคัญ ⭐

- **Covert Channel** ≠ Cryptography → เป็นการ **ซ่อน** การติดต่อใน traffic ปกติ
- **ICMP** เป็นเป้าหมายยอดนิยมเพราะ firewall มักอนุญาต และไม่มี port
- **Field `Message`** ใน ICMP packet = ที่แฝงข้อมูลหลัก
- **ใช้ Hping2** สร้าง custom payload, **ใช้ Wireshark** ตรวจสอบ
- **Detection:** Snort rule + statistical/anomaly analysis
- **Protocol นิยมรองลงมา:** DNS (iodine, dnscat2), HTTP (Web Shell)

---

## Additions by Claude

เพิ่มเนื้อหาที่ไม่ได้อยู่ในต้นฉบับ:
- ✅ หมายเหตุว่า code เป็น **Python 2** ที่หมด support แล้ว + คำแนะนำการ port ไป Python 3
- ✅ เทคนิคการตรวจจับสมัยใหม่: **statistical anomaly, baseline, entropy, DPI**
- ✅ Covert Channel เพิ่มเติม: **TCP timing/header abuse, DoH, cloud-service C2**
- ✅ ตัวอย่าง Web Shell ที่นิยม (China Chopper, b374k)
- ✅ เหตุผลที่ DNS/HTTP เป็น protocol ที่นิยมใช้ทำ covert channel

ปรับปรุงโครงสร้าง:
- เพิ่ม **ASCII diagram** แทนภาพต้นฉบับเพื่อแสดง flow Client ↔ Server
- จัด ICMP fields เป็นตารางเปรียบเทียบ
- แยก server/client code ออกเป็น **โครงสร้าง 3-4 ส่วน** เพื่อเข้าใจง่าย แทนที่จะวาง code ทั้งก้อน
- เน้น (⭐) แก่นที่ต้นฉบับเน้นซ้ำหรือเป็นจุดสำคัญ
