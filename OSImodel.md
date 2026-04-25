OSI Model
OSI Model (Open Systems Interconnection Model) คือรูปแบบการพูดคุยเพื่อแลกเปลี่ยนข้อมูลระหว่างอุปกรณ์ต่างๆ (ในที่นี้รวมถึง Computer, Mobile, Internet of Thing และอื่นๆ) ซึ่งรูปแบบดังกล่าวนั้นจะทำให้ผู้รับ (Client) และผู้ส่ง(Server) นั้นพูดคุยกันได้อย่างรู้เรื่อง โดยจะแบ่งการทำงานออกเป็น 7 Layers โดย

Layer 1-3 จะเป็นการติดต่อผ่าน Hardware
Layer 4-7 จะเป็นการติดต่อผ่าน Software
ทั้งนี้แต่ละ Layer จะมีบทบาท, หน้าที่และหลักการทำงานที่แตกต่างกันออกไป รวมถึงข้อมูลที่ส่งต่อกันในแต่ละขั้นจะถูกเพิ่มหรือถอด header หรือเรียกว่า Encapsulation/Decapsulation ก่อนส่งไปยัง layer ที่อยู่ใกล้เคียง โดย Layer ทั้ง 7 มีดังต่อไปนี้
 

image
รูปภาพสรุป OSI Model
Layer 7: Application Layer
Application Layer เป็น Layer ที่อยู่ใกล้กับ Users มากที่สุด โดยจะใช้ Software ในการ Interact กับ Users เช่น แอพพลิเคชั่นจำพวก Browser, Line, และอื่นๆ

Layer6: Presentation Layer
Presentation Layer เป็น Layer ที่ใช้ในการ Translate ข้อมูลจาก/ไปยัง Application layer เช่น หาก client ส่งข้อความเป็น “Hello, how are you?” layer นี้จะทำการแปลงข้อความเหล่านั้นเป็นรหัส (JPEG,ASCII,PNG,TIFF,etc.) และส่งให้ Server เมื่อ Server ได้รับแล้วมาถึงขั้น Presentation layer ทาง Server จัแปลงรหัสให้กลายเป็น  “Hello, how are you?” อีกที

Layer5: Session Layer
Session Layer เป็น Layer ที่ sync การใช้งานของ session แต่ละ connection ว่าจะใช้เวลาในการใช้บริการนานเท่าไหร่ ทั้งนี้หาก session นั้นหมดเวลาการทำงาน ก็จะตัดการเชื่อมต่อนั้นๆไป

Layer4: Transport Layer
Transport Layer เป็น Layer ที่ควบคุมการรับส่งข้อมูลระหว่าง Client และ Server ทั้งนี้ใน Layer นี้จะมีการแบ่งข้อมูลให้อยู่ในลักษณะของ “Segment” จากนั้นจะทำการเพิ่ม L4Header (Protocol, Source Port, Destination Port) เข้าไปแต่ละ Segment ซึ่งเรียกกระบวนการนี้ว่า Segmentation
 

image
รูปภาพจาก https://apipong.weebly.com/3623363635943634360736373656362636293609.html
Layer3: Network Layer
Network Layer เป็น Layer ที่สร้างการเชื่อมต่อระหว่าง Client และ Server โดยใช้ IP Address ทั้งนี้หลังจากได้รับ Segment ที่มี L4 Header มาแล้ว ก็จะเพิ่ม L3 Header เข้าไปเพิ่มเติม (Source IP, Destination IP) ซึ่งหลังจากกระบวนการเพิ่ม L3 Header เข้าไปแล้วจะเรียกว่า “Packet”
 

image
รูปภาพจาก https://www.ablenet.co.th/2020/08/28/what-is-osi-model/
Layer2: Data Link Layer
Data Link Layer เป็น Layer ที่สร้างการเชื่อมต่อระหว่างเครื่องกับ Network device ต่างๆ เช่น Switch, Router, Hub เป็นต้น โดยจะนำ Packet มาเพิ่ม L2 Header และ L2 Trailer เข้าไปเพิ่มเติม (Source MAC Address, Destination MAC Address, Tag VLAN, etc.) เมื่อทำการเพิ่ม Header เข้าไปแล้วจะเรียกว่า “Frame”
 

image
รูปภาพจาก https://www.ablenet.co.th/2020/08/28/what-is-osi-model/
Layer1: Physical Layer
Physical Layer เป็น Layer ที่แปลง Frame ให้อยู่ในรูปแบบต่างๆ ตามสื่อกลาง เช่น สาย UTP, สาย Fiber Optic โดยใน Layer นี้จะเรียกว่า “Bit” (8 Bits มีค่าเท่ากับ 1 Byte)
 

image
รูปภาพจาก https://www.ablenet.co.th/2020/08/28/what-is-osi-model/
Conclusion
จากหัวข้อนี้จะเห็นว่าการใช้งานของ Network นั้นทำงานในลักษณะที่เป็นลำดับขั้น ทั้งนี้แบ่งออกเป็น
1. Physical Layer สำหรับการรับส่งข้อมูลผ่านสื่อกลาง เรียกข้อมูลใน layer นี้ว่า Bits
2. Data Link Layer สำหรับการรับส่งข้อมูลระหว่าง network device เรียกข้อมูลใน layer นี้ว่า Frame
3. Network Layer สำหรับการรับส่งข้อมูลระหว่าง computer กับ network device เรียกข้อมูลใน layer นี้ว่า Packet
4. Transport Layer สำหรับการควบคุมการรับส่งข้อมูลระหว่าง Client และ Server เรียกข้อมูลใน layer นี้ว่า Segment
5. Session Layer สำหรับการ sync การใช้งานของ session แต่ละ connection
6. Presentation Layer สำหรับการแปลงค่าข้อมูลให้เป็นรหัสต่างๆ
7. Application Layer สำหรับการแสดงค่าข้อมูลให้กับ user ได้เห็น

