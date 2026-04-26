# IPv6 Network — สรุป

## การเชื่อมต่อ IPv6 Network ด้วย Neighbor Discovery Protocol (NDP) ⭐

- **NDP** = protocol ที่ host ใช้คุยกับ host อื่นใน IPv6 network
- ทำหน้าที่คล้าย **ARP + ICMPv4** ของ IPv4 แต่ใช้ **ICMPv6 messages** + algorithm หลายแบบ
- ไม่ได้อธิบาย packet structure แต่ใช้ ICMPv6 + algorithm

### ขั้นตอน Stateless Autoconfiguration ⭐

| ขั้น | ผู้ส่ง | Message | ICMPv6 Type | จุดประสงค์ |
|------|--------|---------|-------------|-----------|
| 1 | Client | **Router Solicitation (RS)** | `133` | ถามหา Router/Gateway ใน network |
| 2 | Router | **Router Advertisement (RA)** | `134` | ตอบกลับ + ระบุ IP 64-bit แรก |
| 3 | Client | สร้าง 64-bit ที่เหลือเอง | — | ใช้ MAC Address แปลงเป็น IID |

- **ไม่ต้องมี DHCP Server** สำหรับ Stateless mode
- ถ้าใช้ Stateful → ยังคงต้องใช้ **DHCPv6** เหมือน IPv4
- RA สามารถระบุ MTU ของ Router ได้ด้วย

### การติดต่อระหว่าง Host ใน Network เดียวกัน

| ขั้น | ผู้ส่ง | Message | ICMPv6 Type | จุดประสงค์ |
|------|--------|---------|-------------|-----------|
| 1 | Client A | **Neighbor Solicitation (NS)** | `135` | ถาม MAC ของ host เป้าหมาย |
| 2 | Host เป้าหมาย | **Neighbor Advertisement (NA)** | `136` | ตอบกลับพร้อม MAC ของตน |

> 💡 เทียบกับ IPv4: NS/NA ≈ ARP Request/ARP Reply

---

## การใช้งาน IPv4 ร่วมกับ IPv6 ⭐

ปัจจุบัน Internet ส่วนใหญ่ยังเป็น IPv4 → ต้องมีวิธีให้ทั้งสองทำงานร่วมกัน

### 1. Dual Stacks
- รองรับทั้ง IPv4 และ IPv6 พร้อมกัน
- เลือก Stack ตาม Application:
  - App ใช้ IPv4 → ส่งผ่าน IPv4 stack
  - App ใช้ IPv6 → ส่งผ่าน IPv6 stack
- ✅ ง่ายที่สุด — ❌ ต้องใช้ Router ที่รองรับทั้งสอง

### 2. Tunneling
- **Encapsulate** IPv6 packet ไว้ใน IPv4 packet
- ขั้นตอน:
  1. ต้นทาง: รับ IPv6 จาก Client → ใส่ IPv4 Header ครอบ
  2. ส่งผ่าน Router IPv4 (มองเป็น IPv4 ปกติ — IPv6 = payload)
  3. Router ปลายทางถอด IPv4 Header → ส่ง IPv6 ต่อให้ host เป้าหมาย

### 3. Translation
2 รูปแบบ:
- **End-point translation** — ใส่ translation function ที่ host (Network/TCP/Socket Layer)
- **Network Device translation** — Router/Gateway แปลง IPv6 ↔ IPv4 ที่ขอบ network

---

## วิธีการเขียน IPv6 ⭐

### โครงสร้าง
- **128 bit แบ่งเป็น 8 ชุด ชุดละ 16 bit**
- คั่นด้วย `:` (IPv4 ใช้ `.`)

**ตัวอย่างเต็ม:**
```
fe80:0001:0002:0003:020c:29ff:feed:0d41
```

**Binary:**
```
1111111010000000  0000000000000001  0000000000000010  0000000000000011
0000001000001100  0010100111111111  1111111011101101  0000110101000001
```

### ย่อด้วย `::`
- ใช้แทนกลุ่ม `0` ที่ติดกันหลายชุด
- ตัวอย่าง: `fe80:0:0:0:0:29ff:feed:d41d` → `fe80::29ff:feed:d41d`

> 🔧 เครื่องมือแปลง Hex ↔ Binary: http://www.gestioip.net/cgi-bin/subnet_calculator.cgi

### กฎตาม RFC 5952 ⭐ — "Recommendation for IPv6 Address Text Representation"

| กฎ | รายละเอียด | ตัวอย่าง |
|----|------------|----------|
| 1 | ตัด `0` นำหน้าในแต่ละชุด | `0211` → `211` |
| 2 | ทั้งชุดเป็น `0` ให้เขียน `0` | `0000` → `0` |
| 3 | ใช้ `::` ได้ **ครั้งเดียว** | — |
| 4 | `::` ต้องอยู่ตำแหน่งที่มี `0` ติดกัน **มากที่สุด** | `fe8:0:1:2:3:0:0:4` → `fe8:0:1:2:3::4` |
| 5 | `::` แทน `0` **ตัวเดียว** ไม่ได้ | ❌ `fe8::1:2:3:4:5:6` (ผิด) จาก `fe8:0:1:2:3:4:5:6` |
| 6 | ตัวอักษร hex ต้องเป็น **lowercase** | `a, b, c, d, e, f` เท่านั้น |

### การเขียน IPv6 + Port

| รูปแบบ | สถานะ |
|--------|-------|
| `[fe8::2:3:4:5:6]:80` | ✅ **Recommend** (RFC 3986) |
| `fe8::2:3:4:5:6.80` | ใช้ได้ |
| `fe8::2:3:4:5:6 port 80` | ใช้ได้ |
| `fe8::2:3:4:5:6p80` | ใช้ได้ |
| `fe8::2:3:4:5:6#80` | ใช้ได้ |
| `fe8::2:3:4:5:6:80` | ❌ กำกวม — ห้ามใช้ |

### Loopback ของ IPv6
- **`::1/128`** ← เทียบเท่า `127.0.0.1` ของ IPv4

---

## สรุปสำคัญ ⭐

- **NDP (Neighbor Discovery Protocol)** = หัวใจการสื่อสารใน IPv6 network
- ใช้ **ICMPv6 type 133/134** สำหรับหา Router → Stateless Autoconfig (ไม่ต้องพึ่ง DHCP)
- ใช้ **ICMPv6 type 135/136** สำหรับหา Host (แทน ARP)
- มี 3 วิธีรวม IPv4 + IPv6: **Dual Stacks**, **Tunneling**, **Translation**
- เขียน IPv6 ด้วย hex 8 ชุด คั่น `:` — ใช้ `::` ย่อ `0` ติดกันได้ครั้งเดียว
- Loopback คือ `::1/128`

---

## Additions by Claude

ฉันได้เพิ่ม/ปรับปรุงเนื้อหาต่อไปนี้:
- ✅ **เปรียบเทียบ NS/NA ↔ ARP Request/Reply** ของ IPv4 — เพื่อให้เห็นความสัมพันธ์
- ✅ จัดเนื้อหา NDP เป็น **ตาราง ICMPv6 type** เพื่อให้จดจำง่าย
- ✅ จัดกฎ RFC 5952 เป็น **ตาราง 6 กฎ** พร้อมตัวอย่าง

หมายเหตุ:
- ต้นฉบับเขียน IID ตัวอย่างว่า `0d41` แต่ในไฟล์ใช้ `d41` (ตัด `0` นำหน้า) — ฉันคงตามกฎ RFC 5952
- เนื้อหาทั้งหมดที่เหลือยังคงเดิมตามต้นฉบับ
