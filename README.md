# 🚕 NYC Taxi Data Analysis - January 2024

> **Project:** Big Data Engineering Mini-Challenge  
> **Goal:** ค้นหาประเภท Taxi ที่มียอดการใช้งาน (Rides) สูงที่สุดในเดือนมกราคม 2024

---

## 📋 Overview

โปรเจกต์นี้วิเคราะห์ข้อมูล NYC Taxi Trip Records เดือนมกราคม 2024 โดยใช้ **AWS Serverless Stack** ตามหลักการ **Modern Data Lakehouse** เพื่อเปรียบเทียบจำนวน rides ของ Taxi แต่ละประเภท

---

## 🏗️ Data Pipeline Architecture

### 📐 Architecture Diagram

![AWS Data Pipeline Architecture](diagram.png)

### ⚙️ How It Works: The Serverless Data Pipeline

การทำงานแบ่งเป็น 5 ขั้นตอนตาม Data Flow:

| Stage | Description | AWS Service |
|-------|-------------|-------------|
| 1️⃣ **Ingest** | Download Parquet files & upload to S3 | Amazon S3 |
| 2️⃣ **Store** | Partitioned folder structure | Amazon S3 |
| 3️⃣ **Define Schema** | Create External Tables (Schema-on-Read) | AWS Glue |
| 4️⃣ **Compute** | Serverless SQL Query | AWS Athena |
| 5️⃣ **Report** | Display results | Pandas/Jupyter |

#### 1️⃣ Ingest (นำเข้าข้อมูล)
- ใช้ Python Script (`boto3`) ทำหน้าที่เป็น Ingestion Layer
- ดาวน์โหลดข้อมูลดิบ (Parquet) จาก NYC Taxi Source
- **Stream Upload** ขึ้นสู่ AWS S3 ทันที

#### 2️⃣ Store (จัดเก็บข้อมูล)
- ข้อมูลถูกจัดเก็บใน **AWS S3** (Data Lake)
- จัดเก็บแบบ **Partitioned Folder** (`/type=yellow`, `/type=green`, `/type=fhv`, `/type=hvfhv`)
- แยก Storage ออกจาก Compute (Decoupled Storage)

#### 3️⃣ Define Schema (กำหนดโครงสร้าง)
- ใช้หลักการ **Schema-on-Read** ผ่าน **AWS Glue Data Catalog**
- สร้าง **External Table** เพื่อกำหนดโครงสร้างครอบไฟล์ดิบ
- ไม่ต้องแปลงไฟล์จริง เพียงสร้าง Metadata

#### 4️⃣ Compute (ประมวลผล)
- ใช้ **AWS Athena** (Serverless SQL Engine)
- ส่งคำสั่ง SQL (`SELECT count(*) ... UNION ALL ...`) ไปประมวลผลบน Cloud
- Athena สแกนไฟล์บน S3 และรวบรวมผลลัพธ์ (Aggregation)

#### 5️⃣ Report (รายงานผล)
- ผลลัพธ์ (Aggregated Results) ส่งกลับมายัง **Jupyter Notebook**
- ใช้ **Pandas** จัดเรียงลำดับ (Ranking) และแสดงผลสรุป

---

## 🛠️ Tech Stack

| Service | Role | Key Benefit |
|---------|------|-------------|
| **AWS S3** | Storage | Decoupled storage, 11 nines durability, ~$0.023/GB/month |
| **AWS Glue** | Metadata | Data Catalog, Schema-on-Read |
| **AWS Athena** | Compute | Serverless SQL, ~$5/TB scanned |
| **Python/Boto3** | Orchestration | AWS SDK for automation |
| **Pandas** | Analysis | Data manipulation & reporting |

---

## 📊 Final Results

### ตารางสรุปจำนวน Rides (เรียงจากมากไปน้อย)

| อันดับ | ประเภท Taxi | จำนวน Rides | เปอร์เซ็นต์ |
|--------|-------------|-------------|-------------|
| 🥇 1 | **HVFHV** | **19,663,930** | **82.0%** |
| 🥈 2 | Yellow Taxi | 2,964,624 | 12.4% |
| 🥉 3 | FHV | 1,290,116 | 5.4% |
| 4 | Green Taxi | 56,551 | 0.2% |

### 🏆 Answer
**Top taxi type (Jan 2024): HVFHV — 19,663,930 rides**

---

## 🚀 Getting Started

### Prerequisites
- Python 3.x
- AWS Account with Learner Lab access
- Required packages: `boto3`, `pandas`, `botocore`

### Installation
```bash
pip install boto3 pandas botocore
```

### Configuration
Update the following credentials in the notebook:
```python
AWS_ACCESS_KEY = "your-access-key"
AWS_SECRET_KEY = "your-secret-key"
AWS_SESSION_TOKEN = "your-session-token"
REGION_NAME = "us-east-1"
BUCKET_NAME = "cs341-taxi-{student_id}-bucket"
```

---

## 📁 Project Structure

```
taxi/
├── After/
│   ├── MiniChallenge_TaxiType_Jan2024_6609612178_After.ipynb  # AWS Cloud version
│   ├── README.md
│   └── diagram.png
└── diagram.py
```

---

## 💡 Key Concepts

### ELT > ETL
- **ELT (Extract, Load, Transform):** โหลดข้อมูลขึ้น S3 ก่อน แล้วค่อย Transform ด้วย Athena
- **Serverless Querying:** Query ข้อมูลโดยตรงจาก Parquet files โดยไม่ต้อง provision server

### Schema-on-Read
- ไม่ต้องแปลงไฟล์ข้อมูล
- สร้าง Metadata เพื่อบอกให้ Athena รู้วิธีอ่านข้อมูล
- ยืดหยุ่น สามารถเปลี่ยน schema ได้โดยไม่ต้อง reprocess data

---

## 🔄 Comparison: Before vs After

| Aspect | Before (Local/DuckDB) | After (AWS Cloud/S3 + Athena) |
|--------|----------------------|-------------------|
| **Storage** | เก็บใน Hard Disk เครื่องตัวเอง (กินพื้นที่) | เก็บใน **S3 Data Lake** (รองรับระดับ Petabyte) |
| **Compute** | ใช้ RAM/CPU เครื่องตัวเอง (ถ้าข้อมูลใหญ่ เครื่องค้าง) | ใช้ **Athena Serverless** (ประมวลผลบน Cloud เร็วและแรง) |
| **Scalability** | ทำงานได้จำกัดแค่สเปคเครื่องที่มี | **Unlimited Scalability** รองรับข้อมูลมหาศาลได้ทันที |
| **Cost** | ฟรี (แต่เสียค่าไฟ/ค่าเครื่อง) | **Pay-as-you-go** จ่ายตามพื้นที่เก็บและปริมาณข้อมูลที่ Scan |

---

## 📝 Reflection

### 1. What was the most difficult part?
* **AWS Configuration & IAM:** ความยากที่สุดคือการจัดการเรื่อง **Credentials (Access Key/Secret Key)** และการทำความเข้าใจเรื่อง Permission ว่าต้องอนุญาตให้ Athena เข้าถึง S3 ได้ อย่างไรก็ตาม การใช้ **Learner Lab** ช่วยลดความซับซ้อนเรื่อง Permission ลงไปได้บ้าง แต่ต้องคอยระวังเรื่อง Session Token หมดอายุ
* **Schema-on-Read Concept:** การปรับความเข้าใจว่าเราไม่ต้อง "Create Table" แบบ Database ทั่วไปที่มีข้อมูลอยู่ข้างใน แต่เป็นการชี้ (Point) ไปที่ S3 แทน ทำให้ต้องระวังเรื่อง Path ของข้อมูลให้ถูกต้องแม่นยำ

### 2. What did I learn?
* **Modern Data Architecture:** ได้เรียนรู้โครงสร้าง **Data Lakehouse** ที่แยก Storage (S3) ออกจาก Compute (Athena) อย่างชัดเจน
* **ELT Process:** เข้าใจกระบวนการ **Extract-Load-Transform** ที่เน้นเอาข้อมูลขึ้น Cloud ให้เร็วก่อน (Load to S3) แล้วค่อยใช้พลังของ Cloud มาจัดการแปลงข้อมูลทีหลัง (Query via Athena)
* **Serverless Power:** เห็นภาพชัดเจนว่า Serverless ช่วยลดงาน Maintenance (No-Ops) และประหยัดค่าใช้จ่ายสำหรับงาน Big Data ได้จริง

---

## 👨‍💻 Author
**NAME:** Ratthatummanoon Kosasang  
**Student ID:** 6609612178

---

## 📄 License

This project is for educational purposes as part of CS341 Big Data Engineering course.
