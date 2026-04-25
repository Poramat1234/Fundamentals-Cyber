# OSI Model — สรุป

## OSI Model คืออะไร ⭐

- **OSI = Open Systems Interconnection Model**
- รูปแบบการพูดคุยเพื่อแลกเปลี่ยนข้อมูลระหว่างอุปกรณ์ (Computer, Mobile, IoT ฯลฯ)
- ทำให้ **Client (ผู้รับ)** และ **Server (ผู้ส่ง)** สื่อสารกันได้รู้เรื่อง
- แบ่งการทำงานเป็น **7 Layers**:
  - **Layer 1–3** → ติดต่อผ่าน **Hardware**
  - **Layer 4–7** → ติดต่อผ่าน **Software**
- ข้อมูลผ่านแต่ละ layer จะถูก **เพิ่ม/ถอด header** เรียกว่า **Encapsulation / Decapsulation**

---

## ตารางสรุป 7 Layer ⭐

| # | Layer | ชื่อข้อมูล (PDU) | บทบาทหลัก |
|---|-------|------------------|----------|
| 7 | **Application** | Data | ติดต่อกับ user ผ่าน software |
| 6 | **Presentation** | Data | แปลง/เข้ารหัสข้อมูล |
| 5 | **Session** | Data | จัดการ session |
| 4 | **Transport** | **Segment** | ควบคุมการรับส่งระหว่าง client/server |
| 3 | **Network** | **Packet** | กำหนดเส้นทางด้วย IP |
| 2 | **Data Link** | **Frame** | เชื่อมต่อกับ network device (MAC) |
| 1 | **Physical** | **Bit** | ส่งข้อมูลผ่านสื่อกลาง |

---

## รายละเอียดแต่ละ Layer

### Layer 7 — Application
- ใกล้กับ user มากที่สุด
- ใช้ software interact กับผู้ใช้
- ตัวอย่าง: Browser, Line

### Layer 6 — Presentation
- **แปลงข้อมูล** ระหว่าง Application layer
- เปลี่ยนข้อความเป็น **รหัส** (JPEG, ASCII, PNG, TIFF ฯลฯ)
- ฝั่งรับจะ **decode กลับ** เป็นข้อความเดิม
- ตัวอย่าง: ส่ง "Hello, how are you?" → encode → ส่งผ่าน network → decode กลับเป็น "Hello, how are you?"

### Layer 5 — Session
- **Sync session** แต่ละ connection
- กำหนดเวลาการใช้บริการ
- เมื่อหมดเวลา → ตัด connection

### Layer 4 — Transport
- ควบคุมการรับส่งข้อมูลระหว่าง **Client ↔ Server**
- แบ่งข้อมูลเป็น **Segment**
- เพิ่ม **L4 Header**: Protocol, Source Port, Destination Port
- กระบวนการนี้เรียกว่า **Segmentation**

### Layer 3 — Network
- สร้างการเชื่อมต่อโดยใช้ **IP Address**
- เพิ่ม **L3 Header**: Source IP, Destination IP
- Segment + L3 Header → เรียกว่า **Packet**

### Layer 2 — Data Link
- เชื่อมต่อระหว่างเครื่องกับ **Network Device** (Switch, Router, Hub)
- เพิ่ม **L2 Header + L2 Trailer**: Source MAC, Destination MAC, Tag VLAN ฯลฯ
- Packet + L2 Header/Trailer → เรียกว่า **Frame**

### Layer 1 — Physical
- แปลง **Frame → Bit** ในรูปแบบของสื่อกลาง
- ตัวอย่างสื่อกลาง: สาย UTP, Fiber Optic
- **8 Bits = 1 Byte**

---

## สรุป Encapsulation Flow ⭐

```
Application Data
    ↓ (+L4 Header)  → Segment
    ↓ (+L3 Header)  → Packet
    ↓ (+L2 Header/Trailer) → Frame
    ↓                → Bit (ส่งผ่านสื่อกลาง)
```

ฝั่งรับจะทำ **decapsulation** ย้อนกลับขึ้นไปแต่ละ layer

---

## Conclusion

การทำงานของ Network เป็น **ลำดับขั้น (layered)**:

1. **Physical** — ส่งผ่านสื่อกลาง → **Bits**
2. **Data Link** — รับส่งระหว่าง network device → **Frame**
3. **Network** — รับส่งระหว่าง computer ↔ network device → **Packet**
4. **Transport** — ควบคุมการรับส่งระหว่าง Client/Server → **Segment**
5. **Session** — sync session แต่ละ connection
6. **Presentation** — แปลงค่าข้อมูลเป็นรหัส
7. **Application** — แสดงค่าให้ user เห็น

---

> **[เพิ่มโดย Claude]** เปรียบเทียบ OSI กับ TCP/IP Model
>
> | OSI (7 layers) | TCP/IP (4 layers) |
> |----------------|-------------------|
> | Application + Presentation + Session | **Application** |
> | Transport | **Transport** |
> | Network | **Internet** |
> | Data Link + Physical | **Network Access** |
>
> OSI เป็น **conceptual reference model** ส่วน TCP/IP เป็นโมเดลที่ Internet ใช้งานจริง

> **[เพิ่มโดย Claude]** เคล็ดลับช่วยจำ
> - Top → Bottom: **"All People Seem To Need Data Processing"**
> - Bottom → Top: **"Please Do Not Throw Sausage Pizza Away"**

---

## Additions by Claude

ฉันได้เพิ่มเนื้อหาต่อไปนี้ที่ไม่ได้อยู่ในต้นฉบับ:
- ✅ **ตารางสรุป 7 Layer** (PDU + บทบาท) — เพื่อให้สแกนอ่านได้ไว
- ✅ **Encapsulation Flow diagram** — สรุปกระบวนการเป็นภาพรวม
- ✅ **OSI vs TCP/IP comparison table** — ช่วยเชื่อมโยงกับโมเดลที่ใช้จริง
- ✅ **Mnemonic ช่วยจำ** ทั้งภาษาอังกฤษ — เพื่อช่วยจำลำดับ layer

เนื้อหาคำอธิบายแต่ละ layer มาจากต้นฉบับ `OSImodel.md` โดยตรง
