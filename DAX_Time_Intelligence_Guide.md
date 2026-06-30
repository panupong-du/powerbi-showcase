# คู่มือเขียน Advanced DAX และ Time Intelligence (เตรียมสัมภาษณ์ TCS)

หัวข้อนี้ตอบโจทย์คำถามสัมภาษณ์เรื่อง **"ประสบการณ์เขียน DAX ที่ซับซ้อน"** การใช้แค่ฟังก์ชัน `SUM` หรือ `DIVIDE` ธรรมดาจะไม่พอครับ เราต้องโชว์ของด้วยการดึงข้อมูลข้ามเวลา (Time Intelligence) และการเปลี่ยนบริบทฟิลเตอร์ (Filter Context) ด้วยคำสั่ง `CALCULATE` สุดฮิตครับ

---

## 📅 1. Time Intelligence (วิเคราะห์ข้ามเวลา)
ฟังก์ชันตระกูล Time Intelligence จะ **บังคับว่าต้องมีตารางวันที่ (Dim_Date)** และต้องลากความสัมพันธ์เรียบร้อยแล้ว ซึ่งคุณทำไว้สมบูรณ์แบบแล้วครับ! ลุยเขียน Measure เหล่านี้ลงในตาราง `_Measures` ได้เลยครับ:

### 1.1 ยอดขายสะสมตั้งแต่ต้นปีจนถึงปัจจุบัน (YTD - Year-To-Date)
นี่คือค่ายอดฮิตที่ทุก Dashboard ต้องมี
```dax
Total Sales YTD = 
TOTALYTD(
    [Total Sales], 
    'Dim_Date'[Date]
)
```

### 1.2 ยอดขายของช่วงเวลาเดียวกันในปีก่อนหน้า (Last Year)
เพื่อเอาไว้เทียบว่าปีนี้เราทำได้ดีกว่าปีที่แล้วไหม
```dax
Total Sales Last Year = 
IF(
    HASONEVALUE('Dim_Date'[Year]),
    CALCULATE(
        [Total Sales],
        SAMEPERIODLASTYEAR('Dim_Date'[Date])
    ),
    0
)
```

### 1.3 เปอร์เซ็นต์การเติบโตเทียบกับปีก่อน (YoY Growth %)
เอาค่าปัจจุบัน ลบ ค่าอดีต หารด้วย ค่าอดีต (ถ้าไม่มีข้อมูลให้คืนค่า 0)
*(อย่าลืมปรับ Format ของ Measure นี้ให้เป็น % ที่แถบด้านบนด้วยนะครับ)*
```dax
YoY Growth % = 
DIVIDE(
    [Total Sales] - [Total Sales Last Year],
    [Total Sales Last Year],
    BLANK() // เปลี่ยนจาก 0 เป็น BLANK() เพื่อไม่ให้โชว์ 0% ตอนไม่ได้เลือกปี
)
```

### 1.4 อัตราการเติบโตเฉลี่ยสะสมหลายปี (CAGR - Compound Annual Growth Rate)
ถ้ากรรมการถามว่า "อยากเห็นภาพรวมหลายปีที่ผ่านมาว่าธุรกิจโตขึ้นเฉลี่ยปีละเท่าไหร่?" ให้งัดสูตรนี้มาโชว์เลยครับ (สูตรนี้คือระดับ Advanced DAX & Business Finance ของแท้!):
```dax
CAGR % = 
// 1. หาปีแรกสุดและปีล่าสุด จากข้อมูลที่มีอยู่
VAR MinYear = MIN('Dim_Date'[Year])
VAR MaxYear = MAX('Dim_Date'[Year])
VAR NumYears = MaxYear - MinYear

// 2. หายอดขายของปีแรกสุด และปีล่าสุด
VAR BeginningValue = CALCULATE([Total Sales], 'Dim_Date'[Year] = MinYear)
VAR EndingValue = CALCULATE([Total Sales], 'Dim_Date'[Year] = MaxYear)

// 3. เข้าสมการการเงิน (Ending/Beginning)^(1/n) - 1
RETURN
IF(
    NumYears > 0 && BeginningValue > 0,
    (EndingValue / BeginningValue) ^ (1 / NumYears) - 1,
    BLANK()
)
```
*(ลองเอา `CAGR %` ไปใส่ใน Card โดยไม่กด Filter ใดๆ ดูครับ มันจะโชว์เปอร์เซ็นต์การเติบโตเฉลี่ยตั้งแต่ปี 2011 ถึง 2014 ทันที และถ้าคุณเปลี่ยน Slicer เป็น 2012-2014 ตัวเลขก็จะเปลี่ยนตามอย่างชาญฉลาดครับ)*

---

## 🎯 2. การใช้ CALCULATE ขั้นแอดวานซ์ (Filter Context)
`CALCULATE` คือฟังก์ชันที่ทรงพลังที่สุดใน DAX เพราะมันสามารถเสกให้การคำนวณ "เพิกเฉย" ต่อ Filter ปกติ หรือ "ยัดเยียด" เงื่อนไขใหม่เข้าไปได้

### 2.1 ดึงยอดขายเฉพาะลูกค้ากลุ่ม Consumer (โดยไม่สนว่ากราฟจะโดนกด Filter อะไรอยู่)
สมมติคุณอยากได้ตัวเลขยอดขายลูกค้า Consumer แบบเป๊ะๆ มาแปะใน Card
```dax
Sales (Consumer Only) = 
CALCULATE(
    [Total Sales],
    'Dim_Customer'[Segment] = "Consumer"
)
```

### 2.2 สัดส่วนยอดขายต่อยอดขายทั้งหมด (เปอร์เซ็นต์ % of Total)
ถ้าสัมภาษณ์ถามว่า: *"อยากรู้ว่าสินค้านี้ขายได้คิดเป็นกี่เปอร์เซ็นต์ของยอดขายบริษัททั้งหมด จะเขียน DAX ยังไง?"* ให้ตอบด้วยสูตรนี้ครับ:
```dax
% of Total Sales = 
VAR AllCompanySales = CALCULATE([Total Sales], ALL('Fact_Sales'))
RETURN
DIVIDE([Total Sales], AllCompanySales, 0)
```
*(ฟังก์ชัน `ALL` จะทำการล้าง Filter ทั้งหมดทิ้งไป ทำให้เราได้ตัวเลขยอดขายรวมทั้งบริษัทมาหารเป็นฐานครับ)*

---

**🔥 บททดสอบ:**
หลังจากสร้าง Measures เสร็จแล้ว ลองสร้าง **Matrix Visual** ใหม่ดูครับ:
- **Rows:** ลาก `Year` กับ `Month Name` มาวาง
- **Values:** ลาก `[Total Sales]`, `[Total Sales Last Year]`, `[YoY Growth %]` และ `[% of Total Sales]` ลงมา
ลองดูตัวเลขว่ามันทำงานเปรียบเทียบปีต่อปีได้ถูกต้องไหมครับ!
�� เลือกปี 2014 ก็เอาตลอดยอดปี 2014 มาหาร) คุณสามารถเปลี่ยนฟังก์ชันได้ 2 แบบครับ:

**แบบที่ 1: ใช้ `ALLSELECTED()` (นิยมที่สุด)**
ล้าง Filter เฉพาะสิ่งที่อยู่ในตาราง/กราฟ (เช่น ชื่อสินค้า) แต่ยังคงเคารพ Filter ที่กดมาจาก Slicer ด้านนอก (เช่น เลือกปี 2014 หรือเลือก Region West ไว้)
```dax
VAR AllCompanySales = CALCULATE([Total Sales], ALLSELECTED())
```

**แบบที่ 2: ใช้ `REMOVEFILTERS` คู่กับ `VALUES` (ควบคุมได้ 100%)**
สั่งล้างให้หมดทุกตาราง แต่จงใจดึง Filter ของปีกลับมาคืนชีวิตให้มัน
```dax
VAR AllCompanySales = 
CALCULATE(
    [Total Sales], 
    REMOVEFILTERS(), // ล้าง Filter ทั้งโมเดล
    VALUES('Dim_Date'[Year]) // ดึง Filter ของปีกลับมา
)
```

---

**🔥 บททดสอบ:**
หลังจากสร้าง Measures เสร็จแล้ว ลองสร้าง **Matrix Visual** ใหม่ดูครับ:
- **Rows:** ลาก `Year` กับ `Month Name` มาวาง
- **Values:** ลาก `[Total Sales]`, `[Total Sales Last Year]`, `[YoY Growth %]` และ `[% of Total Sales]` ลงมา
ลองดูตัวเลขว่ามันทำงานเปรียบเทียบปีต่อปีได้ถูกต้องไหมครับ!
