# 🚀 Power BI Advanced Tips: Dynamic Date Table & Sorting

## 1. การสร้าง Date Table แบบ Dynamic (ยืดหดตามข้อมูลจริง)
การสร้างตารางวันที่ใน Power Query โดยใช้วิธีหาค่า Min/Max จากข้อมูล Fact Table ถือเป็น **Ultimate Best Practice** เพราะตารางปฏิทินจะปรับเปลี่ยนตามข้อมูลที่มีอยู่จริงโดยอัตโนมัติ (คล้ายกับการทำงานของ `CALENDARAUTO()` ใน DAX)

**M Code สำหรับ Dynamic Date Table:**
```powerquery
let
    // 1. ไปดึงค่า วันที่น้อยที่สุด (Min) และ มากที่สุด (Max) จากตาราง Superstore (Fact Table)
    MinDate = List.Min(Superstore[Order Date]),
    MaxDate = List.Max(Superstore[Order Date]),
    
    // 2. ดึงเฉพาะค่า "ปี" ออกมา แล้วปัดให้เริ่ม 1 มกราคม และ จบที่ 31 ธันวาคม (เพื่อให้ได้ปีปฏิทินที่สมบูรณ์)
    StartDate = #date(Date.Year(MinDate), 1, 1),
    EndDate = #date(Date.Year(MaxDate), 12, 31),
    
    // 3. คำนวณจำนวนวันทั้งหมด
    DayCount = Duration.Days(Duration.From(EndDate - StartDate)) + 1,
    
    // 4. สร้าง List ของวันที่ และแปลงเป็น Table
    Source = List.Dates(StartDate, DayCount, #duration(1, 0, 0, 0)),
    TableFromList = Table.FromList(Source, Splitter.SplitByNothing()),
    ChangedType = Table.TransformColumnTypes(TableFromList,{{"Column1", type date}}),
    RenamedColumns = Table.RenameColumns(ChangedType,{{"Column1", "Date"}}),
    
    // 5. เพิ่มคอลัมน์ ปี, ไตรมาส, เดือน 
    InsertYear = Table.AddColumn(RenamedColumns, "Year", each Date.Year([Date]), Int64.Type),
    InsertQuarter = Table.AddColumn(InsertYear, "Quarter", each "Q" & Text.From(Date.QuarterOfYear([Date]))),
    InsertMonth = Table.AddColumn(InsertQuarter, "Month Number", each Date.Month([Date]), Int64.Type),
    InsertMonthName = Table.AddColumn(InsertMonth, "Month Name", each Date.MonthName([Date]), type text)
in
    InsertMonthName
```
*💡 ข้อควรระวัง: ต้องแน่ใจว่าคอลัมน์อ้างอิง (เช่น `Superstore[Order Date]`) ถูกแปลง Data Type เป็น Date เรียบร้อยแล้วใน Query ของมันเอง*

---

## 2. ปัญหาคลาสสิก: กราฟเดือนไม่เรียงตามปฏิทิน (Sort by Column)
**ปัญหา:** เมื่อนำคอลัมน์ที่เป็นข้อความ เช่น `Month Name` (January, February...) ไปสร้างกราฟ กราฟมักจะเรียงตามตัวอักษร A-Z หรือเรียงตามยอดขาย (Value) ซึ่งผิดหลักการดูข้อมูลแบบ Time-series

**วิธีแก้ (Sort by Column):**
เราต้องผูกคอลัมน์ข้อความเข้ากับคอลัมน์ตัวเลข เพื่อให้โปรแกรมรู้ลำดับที่ถูกต้อง

1. ไปที่แถบ **Fields** ด้านขวา 
2. คลิกเลือกคอลัมน์ที่เป็นข้อความ (เช่น `Month Name`)
3. ที่เมนูด้านบน (Ribbon) ไปที่หมวด **Column tools**
4. คลิกที่ปุ่ม **Sort by column** แล้วเลือกคอลัมน์ตัวเลขที่ต้องการใช้เป็นเกณฑ์ (เช่น `Month Number`)

**การตั้งค่าที่กราฟเพิ่มเติม:**
หลังจากผูก Column แล้ว ต้องไปสั่งให้กราฟเรียงตามแกน (Axis) ที่เราต้องการด้วย:
1. คลิกจุด 3 จุด (`...`) ที่มุมขวาบนของกราฟ
2. เลือก **Sort axis** -> เลือกชื่อคอลัมน์แกน X (เช่น `Month Name`)
3. คลิก `...` อีกครั้ง แล้วเลือก **Sort ascending** (เพื่อเรียงจากเดือน 1 ไป 12)
