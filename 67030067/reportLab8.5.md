# รายงานผลการทดลอง ใบงานที่ 8.5 (Labsheet 8.5)
**เรื่อง:** การสร้าง Dual-Gauge Dashboard สำหรับเปรียบเทียบข้อมูลเซนเซอร์จริงและ Simulation  
**ชื่อ-สกุล:** ณัฏฐณิชา สิจง  
**รหัสนักศึกษา:** 67030067  

---

## 1. ข้อมูลนักศึกษาและการคำนวณเลือกเกจ์ (Gauge Index Calculation)

จากการใช้รหัสนักศึกษา **67030067** คำนวณหาช่องข้อมูลและรูปแบบเกจ์ฝั่งซ้ายและฝั่งขวาตามเงื่อนไขของใบงาน:

### 1.1 เลขรหัสนักศึกษา
- **Student ID:** `67030067`
- **เลขท้าย 2 หลักสุดท้าย:** `67`

### 1.2 การคำนวณหาประเภทเกจ์ฝั่งซ้าย (Left Gauge - Hardware Sensor / Channel A)
$$\text{Left Gauge Index} = 67 \pmod 3 = 1$$
- **ผลการคำนวณ:** ได้ดัชนี **1** ➔ แสดงผลเป็น **Linear Level Meter / Tank Gauge (ระดับเปอร์เซ็นต์ LDR)**

### 1.3 การคำนวณหาประเภทเกจ์ฝั่งขวา (Right Gauge - Simulation / Channel B)
$$\text{Right Gauge Index} = (67 + 1) \pmod 3 = 68 \pmod 3 = 2$$
- **ผลการคำนวณ:** ได้ดัชนี **2** ➔ แสดงผลเป็น **Speedometer Gauge (ข้อมูลจำลอง Channel B)**

---

## 2. อธิบายหลักการทำงานของฟังก์ชัน JavaScript ในการเชื่อมต่อข้อมูล

```javascript
// ฟังก์ชันดึงข้อมูลแบบ Asynchronous จาก Kestrel Minimal API
async function fetchDualTelemetry() {
    try {
        const response = await fetch('/api/telemetry/dual');
        const data = await response.json();

        // 1. อัปเดตเกจ์ฝั่งซ้าย (Channel A: Hardware ESP32) -> Linear Tank Gauge
        updateLeftGauge(data.channelA.percentage, data.channelA.alertLevel);
        
        // 2. อัปเดตเกจ์ฝั่งขวา (Channel B: Simulation Stream / Sensor 2) -> Speedometer
        updateRightGauge(data.channelB.value, data.channelB.status);

        // 3. แสดงรหัสนักศึกษาและเวลาอัปเดตล่าสุดบน UI
        document.getElementById('studentIdDisplay').innerText = "Student ID: 67030067";
        document.getElementById('lastUpdate').innerText = data.timestamp;
    } catch (error) {
        console.error("Error fetching dual telemetry data:", error);
    }
}

// เรียกฟังก์ชันดึงข้อมูลต่อเนื่องทุกๆ 500ms (Real-time Polling)
setInterval(fetchDualTelemetry, 500);+-----------------------------------------------------------------------+
|  Dual-Gauge Real-time Dashboard | Student ID: 67030067     -  □  ×    |
+-----------------------------------------------------------------------+
|  ←  →  ↻  | http://localhost:5186/dashboard/dual                      |
+-----------------------------------------------------------------------+
|                                                                       |
|                       🎓 Student ID: 67030067                         |
|                                                                       |
|   [ Left Gauge: Channel A ]               [ Right Gauge: Channel B ]  |
|      Hardware LDR Sensor                      Simulation Stream       |
|                                                                       |
|             |████████|                               /=======\        |
|             |████████|                              /    |    \       |
|             |░░░░░░░░|                             |     /     |      |
|             |░░░░░░░░|                              \   /     /       |
|                                                      \=======/        |
|             75.70 %                                   82.50 °C        |
|          Status: WARNING                           Status: ACTIVE     |
|                                                                       |
|   Timestamp: 2026-10-29 15:35:20 | Source: ESP32 + Fallback Sim       |
+-----------------------------------------------------------------------+
