# Roadmap: TCS Power BI Developer Interview Prep (Superstore Project)

แผนงานนี้ถูกออกแบบมาเพื่อใช้ข้อมูล `Superstore` ในการฝึกฝนและสร้างผลงาน (Portfolio) ให้ครอบคลุมทุกหัวข้อทางเทคนิค (Technical Core) ที่ TCS คาดหวังจากผู้สมัครตำแหน่ง Power BI Developer ตามเอกสาร `TCS_Interview_Prep.md`

---

## 🚀 Step 1: Data Modeling & Power Query (แปลง Flat File เป็น Star Schema)
**เป้าหมาย:** ตอบโจทย์ข้อ 3 (Data Modeling) และข้อ 4 (Power Query)
**อธิบาย:** 
ผู้สัมภาษณ์มักจะถามว่า "คุณออกแบบ Data Model อย่างไร?" การนำเข้าไฟล์ CSV มาทำตารางเดียว (Flat table) ไม่ใช่วิธีที่ถูกต้องในระดับ Enterprise เราจึงต้องใช้ Power Query แปลงให้อยู่ในรูปของ Star Schema

**สิ่งที่จะทำ:**
- [x] นำเข้าไฟล์ `Superstore.csv` 
- [x] ใช้ Power Query (Reference/Duplicate) เพื่อแยกข้อมูลออกเป็น:
  - **1 Fact Table:** `Fact_Sales` (เก็บเฉพาะ Transaction และ Foreign Keys)
  - **3 Dimension Tables:** `Dim_Customer`, `Dim_Product`, `Dim_Location`
- [x] สร้าง Relationship (1-to-Many) ระหว่างตาราง Fact และ Dimension
- [x] ลิงก์เข้ากับตารางวันที่ที่มีอยู่แล้ว (`Dim_Date_PQ`)

---

## 🚀 Step 2: Advanced DAX & Time Intelligence
**เป้าหมาย:** ตอบโจทย์ข้อ 2 (DAX & Complex Calculations)
**อธิบาย:** 
ยกระดับจากการใช้แค่ `SUM`, `DIVIDE` พื้นฐาน ไปสู่การเขียน DAX ระดับกลาง-สูง เพื่อเอาไปใช้ตอบคำถามเรื่อง Time Intelligence และ Filter Context

**สิ่งที่จะทำ:**
- [x] สร้าง Measure สำหรับ Time Intelligence พื้นฐาน เช่น `YTD Sales`, `MTD Sales`
- [x] สร้าง Measure เปรียบเทียบข้อมูลข้ามเวลา เช่น `YoY Growth %` (Year-over-Year)
- [x] สร้าง Measure หาระยะยาวระดับ Business Finance เช่น `CAGR %`
- [x] ใช้ฟังก์ชัน `CALCULATE` ซ้อนเงื่อนไขที่ซับซ้อน (Filter Context)
- [x] 🌟 **Extra Credit:** เทคนิค UX/UI ขั้นสูง (Conditional Visibility ด้วยฟังก์ชัน `HASONEVALUE`)

---

## 🚀 Step 3: Row-Level Security (RLS) Implementation
**เป้าหมาย:** ตอบโจทย์ข้อ 6 (Security & Data Visibility)
**อธิบาย:**
คำถามบังคับ "คุณจำกัดสิทธิ์ให้แต่ละคนเห็นข้อมูลไม่เหมือนกันอย่างไร?" การทำ RLS ไว้ในโปรเจกต์นี้จะทำให้คุณเห็นภาพและอธิบายได้ไหลลื่นขึ้น

**สิ่งที่จะทำ:**
- [x] สร้าง Roles ในเมนู Manage Roles ของ Power BI Desktop
- [x] ตัวอย่างที่ 1: สร้าง Role **"West Manager"** และตั้งค่า DAX Filter ให้เห็นเฉพาะ `Region = "West"`
- [x] ทดสอบการมองเห็นข้อมูลผ่านฟีเจอร์ "View as roles"

---

## 🚀 Step 4: Power Automate Integration
**เป้าหมาย:** ตอบโจทย์ข้อ 10 (Integration with Power Platform)
**อธิบาย:**
เพื่อให้สอดคล้องกับประสบการณ์ใน Resume ของคุณที่มีทักษะ Power Automate การมีปุ่มกดใน Dashboard จะเพิ่มความโดดเด่นเหนือแคนดิเดตคนอื่นมาก

**สิ่งที่จะทำ:**
- [x] ทำความเข้าใจ Concept การทำงานแบบ Cross-platform ระหว่าง Power BI และ Power Automate
- [x] เตรียมสคริปต์คำตอบเกี่ยวกับการทำ "Actionable Dashboard" (ส่งอีเมลแจ้งเตือนอัตโนมัติจากหน้าจอ Power BI)

---
*หากพร้อมที่จะเริ่ม Step ไหน สามารถแจ้งให้ AI ช่วยพาทำแบบ Step-by-Step ได้เลยครับ!*
