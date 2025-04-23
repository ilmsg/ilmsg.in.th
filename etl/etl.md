## สอน ETL (Extract, Transform, Load) ด้วย Python สำหรับ Data Analytics

ETL เป็นกระบวนการสำคัญในงาน Data Engineering และ Data Analytics เพื่อรวบรวมข้อมูลจากแหล่งต่างๆ มาทำความสะอาด ปรับปรุง และจัดเก็บในรูปแบบที่พร้อมสำหรับการวิเคราะห์

**ETL ประกอบด้วย 3 ขั้นตอนหลัก:**

1.  **Extract (สกัด):** ดึงข้อมูลจากแหล่งข้อมูลต้นทาง (Source) แหล่งข้อมูลอาจเป็น:
    *   ฐานข้อมูล (SQL, NoSQL)
    *   ไฟล์ (CSV, Excel, JSON, XML, Text)
    *   API (Application Programming Interface) จากเว็บเซอร์วิสต่างๆ
    *   Web Scraping (ดึงข้อมูลจากหน้าเว็บ)
    *   ระบบอื่นๆ (ERP, CRM)

2.  **Transform (แปลง):** ปรับเปลี่ยนรูปแบบโครงสร้าง และคุณภาพของข้อมูลที่ดึงมาให้เหมาะสมกับการใช้งาน การแปลงข้อมูลอาจรวมถึง:
    *   **Cleaning:** ลบข้อมูลซ้ำซ้อน, จัดการค่าว่าง (Missing Values), แก้ไขข้อมูลที่ผิดพลาด
    *   **Structuring/Reshaping:** เปลี่ยนรูปแบบตาราง (เช่น Pivot, Unpivot), รวม/แยกคอลัมน์
    *   **Enriching:** เพิ่มข้อมูลใหม่เข้าไป (เช่น คำนวณคอลัมน์ใหม่, Join ข้อมูลกับตารางอื่น)
    *   **Formatting:** เปลี่ยนชนิดข้อมูล (Data Type), ปรับรูปแบบวันที่/เวลา, ทำให้ข้อมูลเป็นมาตรฐานเดียวกัน (Standardization)
    *   **Filtering:** กรองข้อมูลที่ไม่ต้องการออก
    *   **Aggregation:** สรุปข้อมูล (เช่น หาผลรวม, ค่าเฉลี่ย)

3.  **Load (โหลด):** นำข้อมูลที่ผ่านการแปลงแล้วไปจัดเก็บยังปลายทาง (Destination) ซึ่งอาจเป็น:
    *   Data Warehouse (คลังข้อมูล)
    *   Data Mart (คลังข้อมูลเฉพาะเรื่อง)
    *   Database
    *   Data Lake
    *   ไฟล์รูปแบบต่างๆ
    *   ระบบ Business Intelligence (BI)

**ทำไมต้องใช้ Python สำหรับ ETL?**

*   **Ecosystem ที่แข็งแกร่ง:** มีไลบรารี (Libraries) ที่ทรงพลังมากมายสำหรับจัดการข้อมูล เช่น Pandas, NumPy, Dask.
*   **ความยืดหยุ่น:** รองรับการเชื่อมต่อกับแหล่งข้อมูลและปลายทางที่หลากหลาย (ไฟล์, ฐานข้อมูล, API).
*   **ง่ายต่อการเรียนรู้และใช้งาน:** Syntax อ่านง่าย เขียนโค้ดได้รวดเร็ว
*   **Integration:** ทำงานร่วมกับเครื่องมือ Data Science และ Machine Learning อื่นๆ ได้ดี
*   **Automation:** สร้าง Script อัตโนมัติสำหรับกระบวนการ ETL ได้ง่าย

---

### ตัวอย่าง: ETL ข้อมูลยอดขายอย่างง่ายด้วย Python (ใช้ Pandas)

**สถานการณ์:** เรามีไฟล์ CSV (`sales_data.csv`) ที่เก็บข้อมูลยอดขายดิบๆ เราต้องการทำ ETL เพื่อ:
1.  **Extract:** อ่านข้อมูลจากไฟล์ CSV
2.  **Transform:**
    *   คำนวณคอลัมน์ `TotalSales` (Price * Quantity)
    *   แปลงคอลัมน์ `OrderDate` เป็นรูปแบบ Datetime
    *   เติมค่าว่างในคอลัมน์ `Category` ด้วย 'Unknown'
    *   ทำให้ชื่อ `Product` เป็นตัวพิมพ์เล็กทั้งหมด (Lowercase)
    *   เลือกเฉพาะคอลัมน์ที่จำเป็นสำหรับการวิเคราะห์
3.  **Load:** บันทึกข้อมูลที่แปลงแล้วลงในไฟล์ CSV ใหม่ (`cleaned_sales.csv`)

**ขั้นตอนที่ 1: เตรียมข้อมูลตัวอย่าง**

สร้างไฟล์ชื่อ `sales_data.csv` โดยมีเนื้อหาดังนี้:

```csv
OrderID,Product,Category,Price,Quantity,OrderDate
1,Laptop,Electronics,1200,2,2023-01-15
2,Keyboard,Electronics,75,5,2023-01-16
3,Mouse,,25,10,2023-01-16
4,Monitor,Electronics,300,3,2023-01-17
5,Webcam,Electronics,50,,2023-01-18
6,Desk Chair,Furniture,150,2,2023-01-19
7,Laptop,Electronics,1250,1,2023-01-20
8,KEYBOARD,Electronics,80,3,2023-01-21
9,Notebook,Stationery,5,50,2023-01-22
10,Pen,Stationery,2,100,invalid_date
```

**ข้อสังเกต:**
*   แถวที่ 3: `Category` ว่าง
*   แถวที่ 5: `Quantity` ว่าง
*   แถวที่ 8: `KEYBOARD` เป็นตัวพิมพ์ใหญ่
*   แถวที่ 10: `OrderDate` มีค่าไม่ถูกต้อง

**ขั้นตอนที่ 2: เขียนโค้ด Python สำหรับ ETL**

```python
import pandas as pd
import os # เพื่อใช้จัดการ path ของไฟล์

# --- Configuration ---
INPUT_CSV_PATH = 'sales_data.csv'
OUTPUT_CSV_PATH = 'cleaned_sales.csv'

# --- 1. Extract ---
def extract_data(file_path):
    """
    อ่านข้อมูลจากไฟล์ CSV
    Args:
        file_path (str): Path ไปยังไฟล์ CSV ต้นทาง
    Returns:
        pandas.DataFrame or None: DataFrame ที่มีข้อมูล หรือ None ถ้าไฟล์ไม่พบ
    """
    print(f"เริ่มต้น: กำลัง Extract ข้อมูลจาก {file_path}...")
    try:
        # ตรวจสอบว่าไฟล์มีอยู่จริงหรือไม่
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"Error: ไม่พบไฟล์ {file_path}")

        df = pd.read_csv(file_path)
        print(f"สำเร็จ: Extract ข้อมูล {len(df)} แถวเรียบร้อย")
        return df
    except FileNotFoundError as e:
        print(e)
        return None
    except Exception as e:
        print(f"เกิดข้อผิดพลาดระหว่าง Extract: {e}")
        return None

# --- 2. Transform ---
def transform_data(df):
    """
    แปลงข้อมูล DataFrame
    Args:
        df (pandas.DataFrame): DataFrame ดิบที่ได้จากขั้นตอน Extract
    Returns:
        pandas.DataFrame: DataFrame ที่ผ่านการแปลงแล้ว
    """
    if df is None:
        print("ข้ามขั้นตอน Transform เนื่องจากไม่มีข้อมูล")
        return None

    print("เริ่มต้น: กำลัง Transform ข้อมูล...")
    transformed_df = df.copy() # ทำสำเนาเพื่อไม่ให้กระทบข้อมูลเดิม

    # --- Cleaning & Formatting ---
    # เติมค่าว่างใน Quantity ด้วย 0 (สมมติว่าถ้าว่างคือไม่ได้ซื้อ)
    transformed_df['Quantity'] = transformed_df['Quantity'].fillna(0)
    
    # เติมค่าว่างใน Category ด้วย 'Unknown'
    transformed_df['Category'] = transformed_df['Category'].fillna('Unknown')

    # แปลง Price และ Quantity เป็น numeric (เผื่อมี text ปน) ทำให้ error เป็น NaN
    transformed_df['Price'] = pd.to_numeric(transformed_df['Price'], errors='coerce')
    transformed_df['Quantity'] = pd.to_numeric(transformed_df['Quantity'], errors='coerce')

    # ลบแถวที่ Price หรือ Quantity เป็น NaN หลังการแปลง (ข้อมูลจำเป็นสำหรับการคำนวณ)
    initial_rows = len(transformed_df)
    transformed_df.dropna(subset=['Price', 'Quantity'], inplace=True)
    if len(transformed_df) < initial_rows:
         print(f"  - ลบ {initial_rows - len(transformed_df)} แถวเนื่องจาก Price หรือ Quantity ไม่ถูกต้อง")

    # แปลง Quantity เป็น integer
    transformed_df['Quantity'] = transformed_df['Quantity'].astype(int)

    # --- Enriching ---
    # คำนวณ TotalSales
    transformed_df['TotalSales'] = transformed_df['Price'] * transformed_df['Quantity']
    print("  - คำนวณคอลัมน์ TotalSales เรียบร้อย")

    # --- Formatting ---
    # แปลง OrderDate เป็น datetime, ถ้าแปลงไม่ได้ให้เป็น NaT (Not a Time)
    transformed_df['OrderDate'] = pd.to_datetime(transformed_df['OrderDate'], errors='coerce')
    print("  - แปลงคอลัมน์ OrderDate เป็น Datetime เรียบร้อย")

    # ลบแถวที่ OrderDate เป็น NaT (ข้อมูลวันที่ไม่ถูกต้อง)
    initial_rows = len(transformed_df)
    transformed_df.dropna(subset=['OrderDate'], inplace=True)
    if len(transformed_df) < initial_rows:
         print(f"  - ลบ {initial_rows - len(transformed_df)} แถวเนื่องจาก OrderDate ไม่ถูกต้อง")

    # แปลง Product เป็น lowercase
    transformed_df['Product'] = transformed_df['Product'].str.lower()
    print("  - แปลงคอลัมน์ Product เป็นตัวพิมพ์เล็กเรียบร้อย")

    # --- Structuring ---
    # เลือกเฉพาะคอลัมน์ที่ต้องการ
    final_columns = ['OrderID', 'Product', 'Category', 'Price', 'Quantity', 'TotalSales', 'OrderDate']
    transformed_df = transformed_df[final_columns]
    print(f"  - เลือกเฉพาะคอลัมน์: {', '.join(final_columns)}")

    print(f"สำเร็จ: Transform ข้อมูลเสร็จสิ้น เหลือ {len(transformed_df)} แถว")
    return transformed_df

# --- 3. Load ---
def load_data(df, output_path):
    """
    บันทึก DataFrame ลงในไฟล์ CSV
    Args:
        df (pandas.DataFrame): DataFrame ที่ผ่านการแปลงแล้ว
        output_path (str): Path สำหรับบันทึกไฟล์ CSV ปลายทาง
    """
    if df is None:
        print("ข้ามขั้นตอน Load เนื่องจากไม่มีข้อมูลให้บันทึก")
        return

    print(f"เริ่มต้น: กำลัง Load ข้อมูลไปยัง {output_path}...")
    try:
        df.to_csv(output_path, index=False, encoding='utf-8') # index=False คือไม่บันทึก index ของ DataFrame ลงไฟล์
        print(f"สำเร็จ: บันทึกข้อมูล {len(df)} แถว ลงใน {output_path} เรียบร้อย")
    except Exception as e:
        print(f"เกิดข้อผิดพลาดระหว่าง Load: {e}")

# --- Main ETL Pipeline ---
if __name__ == "__main__":
    print("--- เริ่มกระบวนการ ETL ---")

    # 1. Extract
    raw_data = extract_data(INPUT_CSV_PATH)

    # 2. Transform
    transformed_data = transform_data(raw_data)

    # 3. Load
    load_data(transformed_data, OUTPUT_CSV_PATH)

    print("--- กระบวนการ ETL เสร็จสิ้น ---")

    # (Optional) แสดงผลลัพธ์ 5 แถวแรก
    if transformed_data is not None and os.path.exists(OUTPUT_CSV_PATH):
        print("\nตัวอย่างข้อมูล 5 แถวแรกในไฟล์ cleaned_sales.csv:")
        print(pd.read_csv(OUTPUT_CSV_PATH).head())
```

**ขั้นตอนที่ 3: รันโค้ด**

1.  บันทึกโค้ด Python ด้านบนเป็นไฟล์ (เช่น `simple_etl.py`) ในโฟลเดอร์เดียวกับ `sales_data.csv`
2.  เปิด Terminal หรือ Command Prompt ไปยังโฟลเดอร์นั้น
3.  รันคำสั่ง: `python simple_etl.py`

**ผลลัพธ์ที่คาดหวัง:**

*   โปรแกรมจะแสดงขั้นตอนการทำงาน (Extract, Transform, Load) บนหน้าจอ
*   จะมีการสร้างไฟล์ใหม่ชื่อ `cleaned_sales.csv` ในโฟลเดอร์เดียวกัน
*   ไฟล์ `cleaned_sales.csv` จะมีเนื้อหาประมาณนี้ (ข้อมูลแถวที่มีปัญหา Price/Quantity/OrderDate จะถูกลบออกไป):

```csv
OrderID,Product,Category,Price,Quantity,TotalSales,OrderDate
1,laptop,Electronics,1200.0,2,2400.0,2023-01-15 00:00:00
2,keyboard,Electronics,75.0,5,375.0,2023-01-16 00:00:00
3,mouse,Unknown,25.0,10,250.0,2023-01-16 00:00:00
4,monitor,Electronics,300.0,3,900.0,2023-01-17 00:00:00
6,desk chair,Furniture,150.0,2,300.0,2023-01-19 00:00:00
7,laptop,Electronics,1250.0,1,1250.0,2023-01-20 00:00:00
8,keyboard,Electronics,80.0,3,240.0,2023-01-21 00:00:00
9,notebook,Stationery,5.0,50,250.0,2023-01-22 00:00:00
```

**ข้อสังเกตในข้อมูลที่แปลงแล้ว:**
*   ไม่มีค่าว่างใน `Category` (ถูกแทนที่ด้วย 'Unknown')
*   คอลัมน์ `TotalSales` ถูกคำนวณแล้ว
*   `OrderDate` เป็นรูปแบบ datetime ที่ถูกต้อง (แถวที่มี 'invalid_date' ถูกลบ)
*   `Product` เป็นตัวพิมพ์เล็กทั้งหมด
*   แถวที่ 5 เดิม (Webcam ที่มี Quantity ว่าง) และแถวที่ 10 เดิม (Pen ที่มี OrderDate ไม่ถูกต้อง) ถูกลบออกไปในขั้นตอน Transform

---

**ไลบรารี Python อื่นๆ ที่นิยมใช้ในงาน ETL:**

*   **Requests:** สำหรับดึงข้อมูลจาก API
*   **Beautiful Soup / Scrapy:** สำหรับ Web Scraping
*   **SQLAlchemy / psycopg2 / mysql-connector-python:** สำหรับเชื่อมต่อและจัดการฐานข้อมูล SQL
*   **pymongo:** สำหรับเชื่อมต่อฐานข้อมูล NoSQL อย่าง MongoDB
*   **Dask:** สำหรับประมวลผลข้อมูลขนาดใหญ่ที่ไม่สามารถโหลดเข้า Memory ได้ทั้งหมด (Parallel Computing)
*   **Apache Airflow / Prefect / Dagster:** สำหรับสร้าง Workflow, จัดการตารางเวลา (Scheduling), และติดตามการทำงานของ ETL Pipeline ที่ซับซ้อน

**ข้อควรพิจารณาเพิ่มเติมสำหรับ ETL ในโลกจริง:**

*   **Error Handling:** จัดการข้อผิดพลาดที่อาจเกิดขึ้นในแต่ละขั้นตอน (เช่น ไฟล์หาย, API ล่ม, ข้อมูลผิดรูปแบบ)
*   **Logging:** บันทึกการทำงานและข้อผิดพลาด เพื่อให้สามารถตรวจสอบย้อนหลังได้
*   **Scalability:** ออกแบบให้รองรับปริมาณข้อมูลที่เพิ่มขึ้นในอนาคต
*   **Scheduling:** ทำให้ ETL ทำงานอัตโนมัติตามเวลาที่กำหนด (เช่น ทุกวัน, ทุกชั่วโมง)
*   **Monitoring:** ติดตามสถานะการทำงานของ ETL pipeline
*   **Testing:** เขียน Test Case เพื่อตรวจสอบความถูกต้องของข้อมูลหลังการ Transform

ตัวอย่างนี้เป็นเพียงจุดเริ่มต้นง่ายๆ แต่แสดงให้เห็นหลักการพื้นฐานของ ETL ด้วย Python และ Pandas ซึ่งเป็นเครื่องมือที่ทรงพลังมากสำหรับงาน Data Analytics ครับ