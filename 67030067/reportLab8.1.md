
# Report Lab 8-1: IoT Edge Gateway ด้วย .NET Minimal API

## กิจกรรมการทดลอง

### กิจกรรมที่ 2: เปลี่ยนข้อความทักทายเป็นชื่อนักศึกษา
กำหนดให้หน้าแรกของ Web Server แสดงข้อความต้อนรับ

```csharp
app.MapGet("/", () => "Welcome to IoT Edge Gateway by Kittikarn!");
```

### กิจกรรมที่ 3: การแปลง C# Object เป็น JSON อัตโนมัติ
สร้าง API `/api/status` เพื่อแสดงสถานะของ IoT Gateway ในรูปแบบ JSON โดยมีข้อมูลดังนี้

- gateway: ESP32-EdgeGateway
- status: Online
- uptimeSeconds: เวลาที่ระบบทำงาน
- isHealthy: true

### กิจกรรมที่ 4: การรับค่าผ่าน URL Path (Route Parameters)
สร้าง API `/api/led/{state}` เพื่อรับค่าการควบคุม LED ผ่าน URL โดยมีการตรวจสอบสถานะ ON/OFF และบันทึก Log ลงบน Terminal พร้อมส่งผลลัพธ์กลับในรูปแบบ JSON

### ภารกิจท้าทาย (Micro-Challenge): /api/student
สร้าง API `/api/student` เพื่อแสดงข้อมูลนักศึกษา ได้แก่

- Student ID: 67030023
- Student Name: Kittikarn
- Faculty: Faculty of Science and Technology, Computer Science and Digital Systems Engineering
- Target Sensor: DHT22
- Timestamp: วันและเวลาปัจจุบัน

---

## คำถามท้ายการทดลอง (Review Questions)

### 1. ในสถาปัตยกรรมของ Kestrel ตัวแปร builder ทำหน้าที่อะไร และตัวแปร app ทำหน้าที่อะไร

**ตัวแปร builder (WebApplicationBuilder)**

ทำหน้าที่จัดการเตรียมสภาพแวดล้อมเริ่มต้นของระบบ (Initialization & Configuration) ก่อนจะสร้างแอปพลิเคชัน เช่น การลงทะเบียนบริการต่าง ๆ (Dependency Injection) การโหลดค่าคอนฟิกจากไฟล์ appsettings.json และการปรับแต่งค่าคอนฟิกของ Kestrel Web Server

**ตัวแปร app (WebApplication)**

ทำหน้าที่เป็นตัวแอปพลิเคชันหลักที่สร้างขึ้นมาจาก builder มีหน้าที่จัดการเส้นทางคำขอ (Request Routing / Endpoints) กำหนดลำดับกระบวนการทำงานกลาง (Middleware Pipeline) และสั่งเริ่มต้นการทำงานของ Kestrel Web Server ผ่านคำสั่ง app.Run()

### 2. เปรียบเทียบความสะดวกระหว่างการสร้าง Web Server บน .NET Minimal API กับการรันผ่าน LAMP Stack (Apache + PHP) ว่ามีข้อดีข้อเสียต่างกันอย่างไรในมุมมองของงาน IoT Gateway

**.NET Minimal API (Kestrel)**

ข้อดี:
- เป็น Self-contained Web Server ในตัว
- ประมวลผลแบบ Asynchronous High-Performance
- กินทรัพยากร CPU และ RAM ต่ำ เหมาะกับอุปกรณ์ IoT Edge Gateway
- มีระบบแปลง C# Object เป็น JSON ให้อัตโนมัติ

ข้อเสีย:
- จำเป็นต้องติดตั้ง .NET Runtime หรือคอมไพล์โปรเจกต์แยกตามสถาปัตยกรรมของฮาร์ดแวร์ก่อนนำไปใช้งาน

**LAMP Stack (Apache + PHP)**

ข้อดี:
- เป็นระบบดั้งเดิมที่หาตัวอย่างการใช้งานได้ง่าย
- สามารถปรับแก้ไฟล์โค้ด PHP บนเซิร์ฟเวอร์ได้ทันทีโดยไม่ต้องคอมไพล์ใหม่

ข้อเสีย:
- โครงสร้างซับซ้อนเพราะต้องติดตั้งบริการหลายตัว (Apache + PHP Engine)
- กินทรัพยากรระบบสูงกว่าจากการสร้าง Thread ประมวลผล
- ต้องใช้ฟังก์ชัน json_encode() / json_decode() จัดการแปลงข้อมูล JSON ด้วยตนเอง

### 3. นักศึกษาคิดว่าการเพิ่ม /api/ เข้าไปใน route นั้นมีประโยชน์อย่างไรบ้าง ถ้าไม่ใส่จะเกิดปัญหาอะไรบ้าง

**ประโยชน์ของการใส่ /api/**

1. **การจัดหมวดหมู่เส้นทาง (API Namespacing):** ช่วยแยกเส้นทางที่ส่งคืนข้อมูลแบบ JSON/Data ออกจากเส้นทางที่แสดงผลหน้าเว็บเพจทั่วไป (HTML/CSS) ได้อย่างชัดเจน
2. **ความสะดวกในการจัดการระบบ:** ช่วยให้ตั้งค่า Reverse Proxy เช่น Nginx, ระบบรักษาความปลอดภัย (Authentication), การจำกัดอัตราส่งข้อมูล (Rate Limiting) และการกำหนดสิทธิ์การเข้าถึงข้อมูล (CORS) ได้ง่ายขึ้น
3. **รองรับ API Versioning:** สามารถต่อยอดเป็น `/api/v1/...` หรือ `/api/v2/...` ในอนาคตได้ง่าย

**ปัญหาที่อาจเกิดขึ้นถ้าไม่ใส่ /api/**

1. **เกิดการชนกันของ Route (Route Collision):** ชื่อ Endpoint อาจซ้ำกับชื่อโฟลเดอร์หรือไฟล์ Static Web เช่น HTML/CSS/JS
2. **เกิดความสับสนในการพัฒนา:** ผู้พัฒนาร่วมหรือระบบภายนอกอาจไม่ทราบแน่ชัดว่า Endpoint นั้นส่งคืนข้อมูล (API Data) หรือเป็นหน้าจอสำหรับแสดงผล (UI)

---

## สรุปผลการทดลอง

จากการทดลองได้เรียนรู้การสร้าง Web Server ด้วย .NET Minimal API โดยใช้ Kestrel สามารถกำหนด Route เพื่อแสดงข้อความต้อนรับ สร้าง API สำหรับตรวจสอบสถานะของ IoT Gateway รับค่าควบคุม LED ผ่าน URL Path และแสดงข้อมูลนักศึกษาผ่าน API ในรูปแบบ JSON ได้ นอกจากนี้ยังได้เข้าใจหน้าที่ของ builder และ app รวมถึงประโยชน์ของการจัดกลุ่ม API ด้วย `/api/` ในการพัฒนาระบบ IoT Gateway
