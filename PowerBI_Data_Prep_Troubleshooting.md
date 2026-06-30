# 🛠️ Power BI Data Preparation & Troubleshooting Notes
*(บันทึกความรู้และปัญหาที่พบบ่อยในการเตรียมข้อมูลและสร้าง Date Table)*

## 1. ข้อควรระวังเมื่อสร้าง Date Table ด้วย DAX
*   **สร้างผิดประเภท:** เวลาสร้าง Date Table ด้วย DAX (เช่น การใช้ `CALENDARAUTO()`) **ต้องใช้ปุ่ม "New table" เสมอ** (ห้ามใช้ New measure) เพราะ Measure จะคืนค่าได้แค่ค่าเดียว ไม่สามารถสร้างเป็นตารางข้อมูลได้
*   **Error หาคอลัมน์วันที่ไม่เจอ:** หากพิมพ์ `CALENDARAUTO()` แล้วขึ้น Error: 
    > `CALENDARAUTO function can not find a base column of DateTime type in the model.`
    **สาเหตุ:** Power BI มองไม่เห็นข้อมูลประเภทวันที่ใน Model ของเราเลย (มักเกิดจากคอลัมน์วันที่จะถูกมองเป็น Text)
    **วิธีแก้:** ไปที่คอลัมน์วันที่ (เช่น Order Date) แล้วเปลี่ยน **Data type** จาก Text เป็น **Date** ก่อนสร้าง Date Table

## 2. ปัญหา Format วันที่สลับกัน (US vs Global) ใน Power Query
*   **สาเหตุ:** ข้อมูลดิบ (เช่น CSV) บันทึกวันที่มาเป็นแบบ **วัน-เดือน-ปี** (DD-MM-YYYY เช่น 17-06-2013) แต่คอมพิวเตอร์หรือ Power BI อ่านค่าเริ่มต้นเป็น **เดือน-วัน-ปี** (US Format) ทำให้พอเจอเดือนที่ 13, 14, 15 โปรแกรมจะแปลงไม่ผ่านและขึ้นค่า `Error`
*   **วิธีแก้ (Best Practice):** ต้องใช้ฟีเจอร์ **Using Locale**
    1. ยกเลิก Step Changed Type เดิมที่พังออกไปก่อน
    2. คลิกคลิกขวาที่หัวคอลัมน์ (หรือไอคอน ABC) เลือก **Change Type -> Using Locale...**
    3. เลือก Data Type เป็น `Date` และตั้งค่า Locale เป็นประเทศที่ใช้วัน-เดือน-ปี เช่น `English (United Kingdom)`

## 3. การรวบโค้ด (Consolidate) การเปลี่ยน Type และ Locale ใน Step เดียว
แทนที่จะทำ Changed Type ปกติ 1 ครั้ง และทำ Using Locale อีก 1 ครั้ง เราสามารถรวบโค้ด (M Code) ให้จบในบรรทัดเดียวได้ ซึ่งจะช่วยให้โค้ดสะอาดและ Performance ดีขึ้น

**ความลับของสูตร:** ฟังก์ชัน `Table.TransformColumnTypes` สามารถรับพารามิเตอร์ตัวที่ 3 ได้ คือการระบุ Locale (เช่น `"en-GB"`) เข้าไปต่อท้ายวงเล็บปีกกา

**ตัวอย่างโค้ด M Code:**
```powerquery
= Table.TransformColumnTypes(#"Promoted Headers", {
    {"Row ID", Int64.Type}, 
    {"Order ID", type text}, 
    {"Order Date", type date}, // เปลี่ยนเป็น date
    {"Ship Date", type date},  // เปลี่ยนเป็น date
    {"Sales", type number}
    // ... คอลัมน์อื่นๆ
}, "en-GB") // <-- ใส่ "en-GB" เพื่อบังคับให้อ่านวันที่แบบ UK (วัน-เดือน-ปี)
```
การเติม `"en-GB"` เข้าไป จะเป็นการการันตีว่าการเปลี่ยนชนิดข้อมูลเป็นวันที่ (type date) ใน Step นี้ทั้งหมด จะถูกอ่านแบบ **วัน-เดือน-ปี** แน่นอน โดยไม่ต้องกลัว Error เลย
