# การสร้าง Dashboard "Store Analytics" ใน Power BI

คำแนะนำแบบ Step-by-Step พร้อมสูตร DAX เพื่อให้คุณสร้าง Dashboard ใน Power BI ให้ออกมาเหมือนภาพต้นฉบับ (Qlik) มากที่สุด

### 1. การเตรียมสูตร DAX (Measures)
ก่อนเริ่มสร้างกราฟ เราต้องสร้าง Measures พื้นฐานเตรียมไว้ก่อน (สมมติว่าตารางข้อมูลชื่อ `Superstore`):

```dax
Total Sales = SUM(Superstore[Sales])
Total Quantity = SUM(Superstore[Quantity])
Total Profit = SUM(Superstore[Profit])
Profit Margin % = DIVIDE([Total Profit], [Total Sales], 0)
```

*(หมายเหตุ: สำหรับสูตรหาจำนวนวันจัดส่งด้านล่างนี้ ให้สร้างเป็น **New column** ที่ตาราง Superstore ไม่ใช่ Measure)*
```dax
Shipping Days = DATEDIFF(Superstore[Order Date], Superstore[Ship Date], DAY)
```

---

### 2. การตั้งค่าหน้ากระดาษและธีม (Page & Theme)
- **Background Color:** เปลี่ยนสีพื้นหลังของหน้า (Canvas Background) เป็นสีเทาอ่อนๆ (เช่น `#F0F4F7` หรือใกล้เคียง)
- **Visual Background:** กราฟทุกตัวให้ตั้งค่าพื้นหลัง (Background) เป็นสีขาว (`#FFFFFF`) และอาจจะเปิด Shadow หรือ Border อ่อนๆ เพื่อให้กราฟดูลอยขึ้นมา
- **Color Palette (สีหลัก):** สีหลักที่ใช้ในกราฟแท่งคือสีน้ำเงินอมเขียว (Teal/Dark Cyan) โค้ดสีประมาณ `#005C6E` หรือ `#066B79`

---

### 3. วิธีสร้าง Visuals แต่ละส่วน

#### 📌 แถวบน: KPI Cards (Card Visual)
ใช้ Visual ประเภท **Card** (หรือ New Card Visual ถ้าใช้ Power BI เวอร์ชันใหม่) สร้าง 4 ตัวเรียงกัน:
1. **Sales:** นำ `Total Sales` ใส่ ฟอร์แมตเป็น `0.0M`
2. **Quantity:** นำ `Total Quantity` ใส่ ฟอร์แมตเป็น `0.00k`
3. **Profit:** นำ `Total Profit` ใส่ ฟอร์แมตเป็น `0.0k`
4. **Profit Margin:** นำ `Profit Margin` ใส่ ฟอร์แมตเป็น Percentage `0.00%`
*(อย่าลืมใส่ Category Label หรือ Title ไว้ด้านบนของแต่ละ Card)*

#### 📊 1. Sales & Profit Margin by Year
- **Visual:** `Line and stacked column chart` (หรือ Clustered column)
- **X-axis:** `Year` (จากคอลัมน์ Order Date)
- **Column y-axis:** `Total Sales`
- **Line y-axis:** `Profit Margin`
- **Formatting:** เปลี่ยนสีแท่ง Column เป็นสี Teal และสีเส้น Line เป็นสีแดงเลือดหมู/ส้ม เปิดใช้งาน `Markers` บนเส้นด้วย
*(หมายเหตุ: ต้องแน่ใจว่านำ `Profit Margin` ไปใส่ในช่อง **Line values** เพื่อให้แสดงผลเป็น 2 แกน ไม่ใช่ Column values)*

#### 📊 2. Quantity by Region
- **Visual:** `Clustered column chart`
- **X-axis:** `Region`
- **Y-axis:** `Total Quantity`
- **Formatting:** จัดเรียง (Sort) ตาม Quantity แบบ Descending เพื่อให้แท่งไล่ระดับความสูงเหมือนในรูป

#### 📊 3. Shipping days by Ship Mode (ใช้เครื่องมือพื้นฐาน)
*เนื่องจาก Power BI ไม่มี Box Plot มาให้ และการโหลด Custom Visual ต้องใช้เมลองค์กร แนะนำให้ใช้ตาราง Matrix เพื่อดูการกระจายตัวของข้อมูลแทนครับ:*

- **Visual:** `Matrix`
- **Rows:** `Ship Mode`
- **Values:** ลาก `Shipping Days` ลงมา 3 ครั้ง แล้วคลิกขวาที่ฟิลด์เพื่อเปลี่ยนการคำนวณเป็น `Minimum`, `Average` และ `Maximum` ตามลำดับ
*(วิธีนี้จะทำให้เราเห็นระยะเวลาที่ใช้จัดส่งน้อยที่สุด, เฉลี่ย, และมากที่สุด ของการจัดส่งแต่ละประเภทได้อย่างชัดเจนครับ)*

#### 📈 4. Sales Trend by Month by Year
- **Visual:** `Line chart`
- **X-axis:** `Month` (ชื่อเดือน Jan, Feb, ...)
- **Y-axis:** `Total Sales`
- **Legend:** `Year`
- **Formatting:** ปรับสีเส้นแต่ละปีให้แตกต่างกันตามต้นฉบับ (ฟ้า, ส้ม/น้ำตาล, ม่วง, ชมพู)

#### 🔲 5. Sales by Segment & Category
- **Visual:** `Matrix` (เนื่องจาก Power BI ไม่มี Heatmap พื้นฐานที่เหมือนใน Qlik เป๊ะๆ)
- **Rows:** `Segment`
- **Columns:** `Category`
- **Values:** `Total Sales`
- **Formatting:** ไปที่ Cell elements เปิดใช้งาน **Background color** ให้กับ Total Sales โดยตั้งค่าสี (Conditional Formatting) ให้ไล่เฉดสีน้ำเงินตามค่ามาก-น้อย (Lowest value = สีฟ้าอ่อน, Highest value = สีน้ำเงินเข้ม) และเปลี่ยนสี Font ให้อ่านง่าย (เช่น สีขาว)

#### 🔴 6. Profit Cross Profit Margin for Rank
- **Visual:** `Scatter chart`
- **Values / Details:** `Sub-Category` (เช่น Paper, Copiers, Phones)
- **X-axis:** `Total Profit`
- **Y-axis:** `Profit Margin`
- **Formatting (พื้นฐาน):** เปิดใช้ Category labels เพื่อให้โชว์ชื่อ Sub-Category ขึ้นมาบนจุดแต่ละจุด

**🎯 เทคนิคระดับ Advance: ทำให้เส้นแบ่งและสีของจุด ขยับตามค่าเฉลี่ยและ Filter (Dynamic Quadrants):**
1. **ตีเส้นค่าเฉลี่ยแบบไดนามิก (Dynamic Average Lines):** ไปที่แถบ **Analytics** (รูปแว่นขยาย)
   - ไม่ใช้ Constant line แล้ว ให้ค้นหาคำว่า **`X-axis average line`** แทน แล้วกด Add (มันจะสร้างเส้นค่าเฉลี่ยของจุดทั้งหมดให้ และขยับตาม Filter)
   - ค้นหาคำว่า **`Y-axis average line`** แล้วกด Add (ตอนนี้จะได้กราฟกากบาทที่เส้นสามารถขยับได้เองแล้ว)
2. **เปลี่ยนสีจุดให้เปลี่ยนตามเส้นค่าเฉลี่ย (Dynamic DAX):** สร้าง Measure ใหม่ที่คำนวณค่าเฉลี่ยแบบ Real-time
   ```dax
   Quadrant Color Dynamic = 
   // 1. หาค่าเฉลี่ยของแกน X และ Y จากข้อมูลที่ถูก Filter อยู่
   VAR AvgProfit = AVERAGEX(ALLSELECTED(Dim_Product[Sub-Category]), [Total Profit])
   VAR AvgMargin = AVERAGEX(ALLSELECTED(Dim_Product[Sub-Category]), [Profit Margin %])
   
   // 2. เปลี่ยนสีจุดโดยเทียบกับค่าเฉลี่ยด้านบน
   RETURN 
   IF([Total Profit] >= AvgProfit && [Profit Margin %] >= AvgMargin, "#28A745", // โซนบนขวา (เขียว): Stars
   IF([Total Profit] < AvgProfit && [Profit Margin %] >= AvgMargin, "#FFC107",  // โซนบนซ้าย (เหลือง): Growth Opportunities
   IF([Total Profit] >= AvgProfit && [Profit Margin %] < AvgMargin, "#17A2B8",  // โซนล่างขวา (ฟ้า): Careful Monitoring
   "#DC3545"))) // โซนล่างซ้าย (แดง): Review or Discontinuation
   ```
   - จากนั้นไปที่รูปแปรงทาสี (Format) > **Data colors** (หรือ Markers > Color)
   - คลิกปุ่ม **`fx`** > เลือก Format by เป็น **`Field value`** > เลือกฟิลด์ `Quadrant Color Dynamic`
   *(เพียงเท่านี้ เมื่อคุณกดคลิก Filter เช่น เลือกดูเฉพาะบาง Region เส้นค่าเฉลี่ยจะขยับ และสีของจุดก็จะคำนวณใหม่และเปลี่ยนสีตามโซนใหม่แบบ Real-time เลยครับ!)*

#### 📝 Text Box: Analytics Descriptions
- ใช้แถบเมนู Insert > **Text box**
- คัดลอกข้อความคำอธิบายจากรูปภาพมาวาง แล้วจัดฟอร์แมตตัวหนา (Bold) และขีดเส้นใต้ (Underline) ตรงหัวข้อให้เหมือนต้นฉบับเป๊ะๆ 

---

**💡 ทริคเพิ่มเติมสำหรับการจัดวาง (Layout):**
ให้ไปที่เมนู `View` > `Gridlines` และเปิด `Snap to grid` จะช่วยให้คุณสามารถลากเรียงกล่อง Visual ทุกกล่องให้ขอบตรงกันและเว้นช่องไฟได้สวยงามเหมือนใน Qlik ต้นฉบับเป๊ะๆ ครับ
