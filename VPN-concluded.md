# VPN — สรุป

## VPN คืออะไร ⭐

- **VPN (Virtual Private Network)** = การขยาย network ภายในผ่านเครือข่ายภายนอก (public)
- สร้าง **ท่อเข้ารหัส (encrypted tunnel)** ระหว่าง client ↔ VPN Server
- VPN Server เป็นผู้ตัดสินใจว่า traffic จะส่งต่อไปทางใด

---

## คุณสมบัติหลัก ⭐

### 1. เข้ารหัสข้อมูลทั้งหมด
- ทุก traffic ที่วิ่งผ่าน VPN ถูกเข้ารหัส
- **ISP, hacker ใน network เดียวกัน → อ่านข้อมูลไม่ได้**
- แก้ปัญหา HTTP ธรรมดาที่ไม่ได้เข้ารหัส และมีความเสี่ยงถูก sniff

### 2. เปลี่ยน IP / ข้ามข้อจำกัดภูมิภาค
- เชื่อม VPN Server ในประเทศอื่น → ใช้ IP ของประเทศนั้น
- เข้าถึง content ที่ถูก geo-block (เช่น Netflix, HBO ในบางประเทศ)

### 3. เพิ่มความปลอดภัย / ความเป็นส่วนตัว
- VPN บางเจ้ามีระบบ **กรองเว็บอันตราย** ในตัว
- ป้องกัน tracking จาก ISP

---

## VPN ทำงานอย่างไร

1. Client เชื่อมต่อ VPN Server ผ่าน internet
2. ใช้ VPN เป็น **gateway** ไปยังปลายทาง
3. Client ได้ **interface + IP ใหม่** ที่อยู่ network เดียวกับ VPN Server
4. Admin สามารถกำหนด routing ให้:
   - ทุก traffic ออกผ่าน VPN (full tunnel)
   - หรือเฉพาะ traffic ภายในองค์กรผ่าน VPN, ที่เหลือออก internet ตรง (**split tunnel** — *[เพิ่มโดย Claude]* คำเทคนิค)

---

## ข้อเสีย ⭐

| ข้อเสีย | รายละเอียด |
|---------|------------|
| **ความเร็วลดลง** | Traffic ต้องวิ่งผ่าน VPN Server ก่อน → เพิ่ม hop, latency สูงขึ้น (โดยเฉพาะถ้า server อยู่ไกล) |
| **เสี่ยงถูกเฝ้ามอง** | ถ้าใช้ VPN Server ที่ไม่น่าเชื่อถือ → server เห็น traffic ทั้งหมด |
| **บางเว็บบล็อก VPN** | Netflix และอื่นๆ ตรวจจับ IP ของ VPN และปฏิเสธการเข้าใช้งาน |

> **[เพิ่มโดย Claude]** ควรเลือก VPN ที่มีนโยบาย **No-Log Policy** และตรวจสอบ jurisdiction ของผู้ให้บริการ — เพื่อจำกัดความเสี่ยงข้อ 2

---

## วิธี Setup OpenVPN บน Ubuntu 16.04

> **[เพิ่มโดย Claude] หมายเหตุ:** Ubuntu 16.04 หมดระยะ support (EOL) ตั้งแต่ปี 2021 — ในการใช้งานจริงควรใช้ Ubuntu LTS เวอร์ชันใหม่ หรือพิจารณา **WireGuard** ซึ่งเป็น VPN protocol สมัยใหม่ที่เร็วและ config ง่ายกว่า

### Phase 1 — เตรียม CA (Certificate Authority)

```bash
# 1. ติดตั้ง package
apt-get install openvpn easy-rsa

# 2. สร้าง CA directory
make-cadir ~/openvpn-ca
cd openvpn-ca
```

**3. แก้ไฟล์ `vars`** กำหนดค่า:
```bash
export KEY_COUNTRY="..."
export KEY_PROVINCE="..."
export KEY_CITY="..."
export KEY_ORG="..."
export KEY_EMAIL="..."
export KEY_OU="..."
export KEY_NAME="server"
```

```bash
# 4-5. สร้าง CA
source vars
./clean-all
./build-ca
```

### Phase 2 — สร้าง Certificates / Keys

```bash
# 6. Server cert
./build-key-server server

# 7. Diffie-Hellman key
./build-dh

# 8. HMAC signature (TLS hardening)
openvpn --genkey --secret keys/ta.key

# 9. Client cert + key
./build-key client1
# หากต้องการ password เพิ่ม:
./build-key-pass client1
```

### Phase 3 — Config Server

```bash
# 10. คัดลอก keys ไป /etc/openvpn
cp ca.crt ca.key server.crt server.key ta.key dh2048.pem /etc/openvpn

# 11. แตก sample config
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz \
  | sudo tee /etc/openvpn/server.conf
```

**12. แก้ `/etc/openvpn/server.conf`** ⭐
```conf
dev tap                                      # interface mode
;dev tun

tls-auth ta.key 0                            # HMAC auth
key-direction 0

cipher AES-128-CBC                           # encryption
auth SHA256                                  # HMAC digest

push "redirect-gateway def1 bypass-dhcp"     # full tunnel
push "dhcp-option DNS 208.67.222.222"        # OpenDNS
push "dhcp-option DNS 208.67.220.220"
```

**13. เปิด IP Forwarding** ใน `/etc/sysctl.conf`
```conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.disable_ipv6 = 1
```
```bash
sudo sysctl -p
```

### Phase 4 — Start VPN

```bash
systemctl restart openvpn@server      # 14. start
systemctl status openvpn@server       # 15. check
systemctl enable openvpn@server       # 16. autostart
```

### Phase 5 — Config Client

```bash
# 17. โครงสร้างโฟลเดอร์
mkdir -p ~/client-configs/files
chmod 700 ~/client-configs/files
ln -s ~/client-configs/ /etc/openvpn/

# 18. คัดลอก base config
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf \
   ~/client-configs/base.conf
```

**19. แก้ `base.conf`** ให้ตรงกับ server:
```conf
dev tap
remote <Destination_Server> 1194
proto udp
user nobody
group nogroup
cipher AES-128-CBC
auth SHA256
key-direction 1
```

**20. Script รวม config ของ client** (`make_config.sh`):
```bash
#!/bin/bash
KEY_DIR=~/openvpn-ca/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>')        ${KEY_DIR}/ca.crt        <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt      <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key      <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key        <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```

```bash
# 21. รัน
chmod +x make_config.sh
./make_config.sh client1
# → ~/client-configs/files/client1.ovpn
```

### Phase 6 — ติดตั้งฝั่ง Client

| OS | วิธี |
|----|------|
| **Windows** | ดาวน์โหลด OpenVPN client → วาง `client1.ovpn` ไว้ที่ `C:\Program Files\OpenVPN\config` |
| **Debian/Ubuntu** | `sudo apt-get install openvpn` |
| **CentOS/RHEL** | `sudo yum install epel-release && sudo yum install openvpn` |

---

## Optional Configurations

### 1. ป้องกันการแชร์ certificate ระหว่าง user
แทน `keepalive 10 120` ด้วย:
```conf
ping 10
ping-restart 240
push "ping 10"
push "ping-exit 60"
```

### 2. กำหนด routing เพิ่มเติมให้ client
```conf
push "route 10.10.10.0 255.255.255.0"
```

### 3. ให้ VPN client เข้าถึง internal network ผ่าน NAT
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth0 -o tap0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i tap0 -o eth0 -j ACCEPT
```

### 4. ใช้ใน Docker
ปัญหา: `Cannot open TUN/TAP dev /dev/net/tun`

แก้:
```bash
docker run -d -i -t -p 1194:1194/udp --privileged="true" <Name>:<Tag> /bin/bash
```

---

## สรุปแก่นสำคัญ ⭐

- **VPN = ท่อเข้ารหัส** ระหว่าง client ↔ server
- **3 ประโยชน์หลัก:** encryption, change IP, privacy filtering
- **3 ข้อเสีย:** ช้าลง, server อาจ snoop, บางเว็บบล็อก
- **OpenVPN setup** = 4 phase: CA → Certs → Server config → Client config
- **Config สำคัญ:** `cipher AES-128-CBC`, `auth SHA256`, `tls-auth ta.key`
- ต้องเปิด **IP forwarding** ใน sysctl และ `iptables MASQUERADE` สำหรับ NAT

---

## Additions by Claude

เพิ่มเนื้อหาที่ไม่ได้อยู่ในต้นฉบับ:
- ✅ คำเทคนิค **"split tunnel"** สำหรับการกำหนด routing แบบเลือก
- ✅ คำแนะนำเรื่อง **No-Log Policy** + jurisdiction ของผู้ให้บริการ VPN
- ✅ หมายเหตุว่า **Ubuntu 16.04 EOL แล้ว** และแนะนำให้พิจารณา **WireGuard** เป็นทางเลือกสมัยใหม่

ปรับปรุงโครงสร้าง:
- จัด setup OpenVPN เป็น **6 phases** เพื่อให้เห็นภาพรวม (ต้นฉบับเรียงเป็น 23 ข้อต่อกัน)
- ใช้ตารางสรุปข้อเสียและขั้นตอนติดตั้งฝั่ง client
- เน้น (⭐) จุดที่เป็นแก่นของหัวข้อ
- จัด config snippet เป็น code block แยกตามไฟล์
