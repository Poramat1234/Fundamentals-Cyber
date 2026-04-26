# เครื่องมือสำหรับการโจมตี IPv6 — สรุป

> ⚠️ **คำเตือน:** เนื้อหานี้เพื่อการศึกษาและการทดสอบความปลอดภัย (Penetration Testing / Authorized Security Testing) เท่านั้น

## ภาพรวม

- IPv6 ต่างจาก IPv4 มาก → เครื่องมือโจมตี/รวบรวมข้อมูลต้องเปลี่ยนตาม
- เครื่องมือหลักที่ใช้กับ IPv6:
  1. **NMAP**
  2. **Metasploit**
  3. **THC-IPv6**

---

## 1. NMAP ⭐

- รองรับ IPv6 ตั้งแต่ **version 6 ขึ้นไป**
- เพิ่ม option **`-6`** เท่านั้น

### ตัวอย่างคำสั่ง

| จุดประสงค์ | คำสั่ง |
|-----------|--------|
| Scan OS เป้าหมาย | `nmap -6 -O fe80::20c:29ff:feed:d41d` |
| Scan port 0–10000 + OS | `nmap -6 -n -p0-10000 fe80::20c:29ff:feed:d41d` |

---

## 2. Metasploit

มี module ใช้สำหรับ IPv6 โดยเฉพาะ — ตัวอย่างที่ใช้บ่อย:

| Module | หน้าที่ |
|--------|---------|
| `auxiliary/scanner/discovery/ipv6_multicast_pin` | ส่ง ICMPv6 ไป multicast → ตรวจสอบ host ใน network |
| `auxiliary/scanner/discovery/ipv6_neighbor` | หา host ที่ตอบ Neighbor Solicitation (คล้าย ARP scan) |
| `auxiliary/scanner/discovery/ipv6_neighbor_router_advertisement` | ปลอม RA แบบ high priority → บีบให้ host ทำ autoconfig + ดู IP |

---

## 3. THC-IPv6 ⭐

ชุดเครื่องมือเฉพาะสำหรับ IPv6 — มีเครื่องมือหลายตัว

### การติดตั้ง

```bash
# 1. download
wget http://www.thc.org/releases/thc-ipv6-2.1.tar.gz

# 2. แตกไฟล์
tar xvf thc-ipv6-2.1.tar.gz

# 3. เข้า folder
cd thc-ipv6-2.1

# 4. compile + install
make && make install
```

### เครื่องมือเด่น

#### `alive6` — ตรวจสอบเครื่องที่เปิดอยู่
```bash
alive6 -M -D eth0
```
| Option | ความหมาย |
|--------|----------|
| `-M` | ดู MAC Address |
| `-D` | ดู DHCP Address space |

#### `dnsdict6` — หา IPv6 ของ domain
```bash
dnsdict6 -t 16 twitter.com
```
- `-t 16` = ใช้ 16 thread

> 🔗 อ่านเพิ่ม: http://www.aldeid.com/wiki/THC-IPv6-Attack-Toolkit
> รองรับการโจมตีหลายแบบ ทั้ง **DoS** และ **Man-In-The-Middle**

---

## ปัญหาและช่องโหว่ของ IPv6 ⭐

> **หมายเหตุ:** ปัญหาหลายอย่างไม่ได้เกิดจาก IPv6 โดยตรง แต่เกิดจาก **OS จัดการ IPv6 ไม่ดี**

ช่องโหว่ที่พบบ่อย:
- RA Spoofing
- DHCPv6 Spoofing
- NS Floods (DoS)
- ICMPv6 Redirect Spoofing

---

## การโจมตีที่ 1: DoS ด้วย RA Packet ⭐

### หลักการ

- เครื่องที่กระทบ: **Windows 7, XP, Vista, Server 2008, X-Box, PS3**
- ผลกระทบ: เครื่อง **แฮงค์** → ต้อง reboot
- กลไก: spam fake RA packet → client คำนวณ IP 128-bit ใหม่ทุกครั้ง → **CPU peak → hang**

### วิธีทดสอบ

ใช้ **`flood_router6`** จาก THC-IPv6:

```bash
# 1. เข้า path ที่ติดตั้ง THC-IPv6
# 2. ปล่อย flood RA ที่ interface
./flood_router6 eth0
```

ตรวจ traffic ใน Wireshark → จะเห็น RA จำนวนมาก (broadcast, source เปลี่ยนทุก packet)

ตรวจ IP ใน Windows 7:
- **ก่อน flood** → IP ปกติ
- **หลัง flood** → IP เพิ่มเข้ามาเรื่อยๆ จนเครื่อง hang

### วิธีป้องกัน RA Spoofing ⭐

| ข้อ | วิธี | ข้อดี/ข้อเสีย |
|-----|------|--------------|
| 2.1 | **เลิกใช้ IPv6** | ✅ ง่ายและเร็ว — ❌ ไม่ได้ใช้ IPv6 |
| 2.2 | **ปิด Router Discovery** (ใช้แต่ Stateful) | ✅ เหมาะกับ Server network — ❌ ไม่เหมาะกับ Client เยอะ |
| 2.3 | **Firewall กรอง RA** อนุญาตเฉพาะ Gateway ที่ trust | ✅ **สมเหตุสมผลที่สุด** ⭐ |

**คำสั่งปิด Router Discovery (Windows):**
```
netsh interface ipv6 set interface "Local Area Connection" routerdiscovery=disabled
```

---

## การโจมตีที่ 2: Man-In-The-Middle (MITM) ⭐

### หลักการ

- ใช้ **Neighbor Discovery (ND) Message** (เทียบ ARP Request/Reply ของ IPv4)
- ปกติ:
  1. Client A ส่ง **Neighbor Solicitation (NS)** ถาม MAC ของ B
  2. B ตอบ **Neighbor Advertisement (NA)** + MAC ของตน
  3. A เก็บ MAC เข้า cache
- โจมตี:
  1. ผู้โจมตีสร้าง **NA ปลอม** อ้างเป็น B
  2. ส่งซ้ำๆ ให้ A
  3. A เก็บ MAC ผู้โจมตี = MAC ของ B
  4. ทุก packet ที่ A ส่งให้ B → ไปหา**ผู้โจมตี**แทน

### ตัวอย่าง MITM

| เครื่อง | OS | IPv6 | MAC |
|---------|-----|------|-----|
| Client | Windows 7 | `2001:db8:1:2::1001/64` | `00:21:CC:60:C3:87` |
| Web Server | Ubuntu 12.04 | `2001:db8:1:2::1002/64` | `00:0C:29:D3:89:43` |
| Hacker | Backtrack 5 R3 | `2001:db8:1:2::2000/64` | `00:0C:29:79:54:2B` |

**ขั้นตอน:**
1. Client ตรวจ IP ของตน:
   ```
   ipconfig
   ```
2. Client ดู neighbor ใน network:
   ```
   netsh interface ipv6 show neighbors 14
   ```
   *(14 = หมายเลข interface ของ Local Area Connection)*
3. Client เข้าเว็บ Server ปกติ → ใช้งานได้
4. Hacker รัน **`parasite6`** จาก THC-IPv6:
   - รอ NS จาก Client
   - ส่ง NA ปลอมกลับ
5. Wireshark → เห็น NA จำนวนมากจาก Hacker
6. ตรวจ neighbor ใหม่ → MAC ของ Web Server **เหมือน** MAC ของ Hacker
7. Client เข้าเว็บ → ได้หน้าเว็บของ Hacker แทน

---

## สรุปสำคัญ ⭐

- เครื่องมือหลักโจมตี IPv6: **NMAP** (`-6`), **Metasploit** (modules), **THC-IPv6** (ชุดเฉพาะทาง)
- THC-IPv6 มีเครื่องมือเด่น:
  - `alive6` — host discovery
  - `dnsdict6` — DNS enumeration
  - `flood_router6` — RA flooding (DoS)
  - `parasite6` — NA spoofing (MITM)
- ช่องโหว่ส่วนใหญ่ไม่ได้มาจาก IPv6 แต่มาจาก **OS ที่จัดการไม่ดี**
- การป้องกัน RA Spoofing ที่ดีที่สุด: **Firewall อนุญาต RA จาก Gateway ที่ trust**
- IPv6 ไม่ได้ปลอดภัย 100% — ยังพบ RA spoofing, DHCPv6 spoofing, NS flood, ICMPv6 redirect spoofing

---

## Additions by Claude

ฉันได้เพิ่ม/ปรับปรุงเนื้อหาต่อไปนี้:
- ✅ **คำเตือนด้านจริยธรรม** — เนื้อหานี้สำหรับการศึกษา/Penetration Testing เท่านั้น
- ✅ จัดเครื่องมือ THC-IPv6 เป็น **ตารางสรุปเครื่องมือเด่น** ในส่วนสรุป
- ✅ จัดวิธีป้องกัน RA Spoofing เป็นตารางพร้อมข้อดี-ข้อเสีย
- ✅ จัดขั้นตอน MITM เป็น **list เรียงลำดับ** เพื่อให้ตามทันได้ง่าย

หมายเหตุ:
- ต้นฉบับมีรูปภาพ (image) หลายรูปที่ฉันไม่สามารถเก็บมาได้ — แทนด้วยคำอธิบายในข้อความแทน
- THC-IPv6 version 2.1 ในต้นฉบับเป็นเวอร์ชันเก่า — ปัจจุบันมี version ใหม่กว่า แต่คงตามต้นฉบับเพื่อความถูกต้องทาง historical
