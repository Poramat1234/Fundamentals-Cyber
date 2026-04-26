# IPv6 — สรุป

## ทำไมต้องมี IPv6

- **IPv4 มี 32 bit** → รองรับได้ 2³² ≈ **4,294,967,296** เครื่อง
- ปัจจุบัน Internet เติบโต → IPv4 **ไม่พอใช้**
- **IETF** จึงออกแบบ **IPv6 (IPng — Next Generation IP)** ขึ้นมาแทน
- **IPv6 มี 128 bit** → รองรับได้ 2¹²⁸ ≈ **3.4 × 10³⁸** เครื่อง ⭐
- พัฒนาใหม่ทั้งหมด — เพิ่มทั้ง **คุณภาพ** และ **ความปลอดภัย** ที่ IPv4 ไม่มี

---

## ข้อปรับปรุงด้านความปลอดภัยของ IPv6 ⭐

### 1. Header รูปแบบใหม่ (40 bytes คงที่)

การเปลี่ยนแปลงจาก IPv4 → IPv6:

| เปลี่ยนแปลง | รายละเอียด |
|-------------|------------|
| ❌ ตัด `H. Length` ออก | — |
| ❌ ตัด `Checksum` ออก | ประหยัดเวลา (encapsulation มี checksum อยู่แล้ว) |
| ❌ ตัด `Option` ออก | ย้ายไปไว้ใน **Extension Header** → routing เร็วขึ้น |
| 🔄 `Type Of Service` → `Traffic Class` | เปลี่ยนชื่อ |
| 🔄 `Total Length` → `Payload Length` | เปลี่ยนชื่อ |
| 🔄 `Time to Live` → `Hop Limit` | เปลี่ยนชื่อ |
| 🚫 ห้าม Fragment ระหว่างทาง | router ห้ามแตก packet — ต้นทาง/ปลายทางเท่านั้น |
| 📦 รวม fragment fields | `identification`, `flags`, `fragment offset` → **Fragment Extension Header** |

- โครงสร้าง packet: **IPv6 Header → Extension Header → Transport Data**

### 2. IPSec แบบฝังในตัว ⭐

- IPv6 ฝัง **IPSec** เป็นส่วนหนึ่งของ protocol (ไม่ใช่ option เหมือน IPv4)
- ทำงานแบบ **End-to-End Encryption** — เข้ารหัสตั้งแต่ต้นทางถึงปลายทาง ไม่ถอดระหว่างทาง
- IPv4 IPsec ใช้สำหรับ VPN ระหว่าง router เท่านั้น

### 3. Quality of Service (QoS)

- ใช้ field `Flow Label` ใน IPv6 header
- รับประกันว่า **ข้อมูลสำคัญถึงปลายทางในเวลาเหมาะสม**
- เหมาะกับ: streaming video, VoIP

### 4. Auto-config IP — Stateless Address Autoconfiguration (SLAAC) ⭐

IPv6 รองรับทั้ง 2 แบบ:

| แบบ | กลไก |
|-----|------|
| **Stateful Autoconfig** | ใช้ DHCPv6 Server แจก IP (เหมือน DHCP เดิม) |
| **Stateless Autoconfig (SLAAC)** | Client สร้าง IP เอง ไม่ต้องพึ่ง DHCP |

**ขั้นตอน SLAAC:**
1. Client กำหนด **Link-Local Address** เริ่มด้วย `FE80::/64`
2. ตามด้วย **Interface Identifier (IID)** 64-bit ที่สร้างจาก MAC Address:
   - แบ่ง MAC 24-bit เป็น 2 ชุด
   - แทรก `FFFE` (16-bit) ตรงกลาง
   - เปลี่ยน bit ที่ 7 จากซ้าย: `0` → `1`
3. รอ **Router Advertisement (RA)** จาก router (หรือส่ง **Router Solicitation, ICMP type 133**)
4. ได้ Network-Layer Configuration (Subnet Mask, ฯลฯ) → กำหนด IP สมบูรณ์

> **หมายเหตุ:** จาก IID เราสามารถ trace ไปถึง MAC Address ของอุปกรณ์ต้นทางได้ → ⚠️ privacy concern

**คำสั่ง CISCO IOS:**
```
ipv6 address dhcp        ← Stateful
ipv6 address autoconfig  ← Stateless
```

### 5. Extension Headers (6 ชนิด)

| Extension Header | หน้าที่ |
|------------------|---------|
| **Routing Header** | บังคับเส้นทาง (คล้าย Source Routing Option ใน IPv4) |
| **Authentication Header (AH)** | Authentication + Integrity + Anti-Replay |
| **ESP Header** | Authentication + Encryption (Confidentiality) |
| **Fragmentation Header** | จัดการ fragment (เฉพาะต้นทาง) |
| **Destination Options Header** | option ประมวลผลที่ปลายทาง (เช่น Mobile IPv6) |
| **Hop-By-Hop Option Header** | option ที่ router ทุก hop ต้องอ่าน |

---

## Authentication Header (AH) — รายละเอียด ⭐

| Field | หน้าที่ |
|-------|---------|
| **Next Header** | บอก extension header ถัดไป หรือ transport type (TCP/UDP) |
| **Payload Length** | ขนาดของ AH |
| **SPI (Security Parameter Index)** | ใช้ค้น Security Association (SA) — เก็บ key สำหรับ auth/encrypt |
| **Sequence Number** | เริ่ม 0, เพิ่มทีละ 1 → ป้องกัน **replay attack** |
| **Authentication Data (ICV)** | Integrity Check Value ใช้ HMAC-MD5 หรือ HMAC-SHA-1 |

> **HMAC** = Hash-based Message Authentication Code → ใช้ secret key + hash function
> - **MD5** → output 128-bit
> - **SHA-1** → output 160-bit

---

## Encapsulating Security Payload (ESP) Header

- ทำได้ทั้ง **Authentication + Confidentiality** (เลือกใช้ทีละอย่าง ใช้ร่วมกันไม่ได้)
- ป้องกัน **sniffing** เพราะ payload เข้ารหัสทั้งก้อน

| Field | หน้าที่ |
|-------|---------|
| **SPI** | เหมือนใน AH |
| **Sequence Number** | Anti-replay เหมือน AH |
| **Payload Data** | ข้อมูลที่เข้ารหัสแล้ว (กำหนด algorithm/key ผ่าน SA) |
| **Padding** + Pad Length + Next Header | สำรองข้อมูลสำหรับ encryption algorithm |
| **Authentication Data (ICV)** | คำนวณจาก ESP ทั้งหมด ยกเว้น Authentication Data |

**Encryption Algorithms ที่ ESP ใช้:**
- **DES** (CBC mode)
- **3-DES**
- **AES**

---

## IPSec ใน IPv6 ⭐

- ระบุไว้ใน **RFC 2401** "Security Architecture for the Internet Protocol"
- ทำงานที่ **Network Layer** → ครอบคลุมทุก application (FTP, HTTP, Email)
- ใช้ AH + ESP (ร่วมกันได้ หรือแยกกัน)

### 2 Modes ของ IPSec

| Mode | การทำงาน |
|------|----------|
| **Tunnel Mode** | ห่อ packet ทั้งก้อน — IPv6 header + AH/ESP หุ้มทั้งหมด |
| **Transport Mode** | ใช้กับ transport layer เท่านั้น |

- **Security Association (SA)** = ระบุด้วย **SPI + IP ปลายทาง + security protocol (AH/ESP)**
- เป็นความสัมพันธ์แบบ **ทางเดียว** ระหว่างผู้ส่ง-ผู้รับ
- เก็บ key สำหรับ authentication/encryption

---

## ความแตกต่าง ICMP: IPv6 vs IPv4 ⭐

### 1. ค่า Next Header (NH)
- **ICMPv6** → NH = `58`
- **IPv4 ICMP** → = `1`

### 2. Neighbor Discovery (ND) แทน ARP
- **ND** รู้แค่:
  - Router ใน network
  - ตรวจ IPv6 ซ้ำ
- **ไม่สอบถามเครื่อง client อื่น** เหมือน ARP
- Update เร็วกว่า → ลด timeout เมื่อ router/link เปลี่ยน

### 3. Maximum Transfer Unit (MTU) ใหญ่ขึ้น
- **IPv4** → minimum MTU = `576 bytes`
- **IPv6** → minimum datagram = `1280 bytes`, minimum MTU = `1500 bytes`
- ประสิทธิภาพดีขึ้น

### 4. ห้าม Fragment ระหว่างทาง
- Fragment เฉพาะที่ **ต้นทาง** เท่านั้น
- ต้นทางใช้ ICMPv6 ตรวจ MTU ของเส้นทางก่อน
- ปลายทางประกอบกลับ

### 5. Multicast Listener Discovery (MLD)
- เทียบเท่า **IGMP** ของ IPv4
- ใช้ ICMPv6 message 3 message
- IPv6 **ไม่มี broadcast** — ใช้ multicast แทนทุกที่

---

## ชนิดของ IPv6 Address ⭐

| ชนิด | Prefix | การใช้งาน |
|------|--------|-----------|
| **Link Local** | `fe80::/10` | คุยกันใน Network เดียวกัน (เช่น Ethernet วงเดียว) |
| **Site Local** | `fd::/8` | คล้าย Private IP ของ IPv4 |
| **Multicast** | `ff::/8` | ส่งให้ทุก client ใน multicast group (เช่น broadcast video) |

> **[เพิ่มโดย Claude]** ปัจจุบัน Site Local (`fec0::/10`) ถูก deprecated และแทนที่ด้วย **Unique Local Address (ULA)** prefix `fc00::/7` (ใช้งานจริงคือ `fd00::/8`) ตาม RFC 4193

---

## สรุปสำคัญ ⭐

- **IPv6 = 128 bit** แก้ปัญหา IPv4 ขาดแคลน
- **Header ใหม่ 40 bytes** — เร็วขึ้น, มี Extension Header แยก
- **IPSec ฝังในตัว** — End-to-End encryption โดย default
- **SLAAC** ลดภาระ DHCP — client สร้าง IP จาก MAC ได้เอง
- **AH + ESP** จัดการ Authentication, Integrity, Anti-Replay, Encryption
- **ICMPv6 + ND + MLD** แทน ARP/IGMP ของ IPv4
- **ไม่มี broadcast** — ใช้ multicast แทน

---

## Additions by Claude

ฉันได้เพิ่มเนื้อหาต่อไปนี้ที่ไม่ได้อยู่ในต้นฉบับ:
- ✅ **Unique Local Address (ULA)** — หมายเหตุว่า Site Local ถูก deprecate และแทนด้วย ULA (`fc00::/7`)

หมายเหตุอื่นๆ:
- จัดเนื้อหาใหม่เป็นตารางและ bullet เพื่อให้อ่านง่าย
- เน้น (⭐) จุดสำคัญที่ต้นฉบับเน้นซ้ำหรือเป็นแก่นของหัวข้อ
- เพิ่ม emoji icon (❌🔄🚫📦) ในตาราง header changes เพื่อแยกประเภทการเปลี่ยนแปลง
