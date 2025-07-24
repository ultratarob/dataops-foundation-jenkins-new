# Simple ETL CI/CD Pipeline

🚀 **ETL Pipeline with Jenkins CI/CD Integration**

## 📋 Overview

โปรเจคนี้เป็น Simple ETL Pipeline ที่ใช้ Jenkins สำหรับ CI/CD โดยมีฟังก์ชันหลัก 3 ตัวในการประมวลผลข้อมูล Loan Data และสร้าง Star Schema

## 🏗️ Project Structure

```
dataops-foundation-jenkins-new/
├── functions/                          # ETL Functions Package
│   ├── __init__.py                     
│   ├── guess_column_types.py           # ฟังก์ชันเดาประเภทข้อมูล
│   ├── filter_issue_date_range.py      # ฟังก์ชันกรองช่วงวันที่
│   └── clean_missing_values.py         # ฟังก์ชันทำความสะอาด missing values
├── tests/                              # Unit Tests
│   ├── guess_column_types_test.py      
│   ├── filter_issue_date_range_test.py 
│   └── clean_missing_values_test.py    
├── etl_pipeline.py                     # ETL Pipeline หลัก
├── Jenkinsfile                         # Jenkins Pipeline Definition
├── requirements.txt                    # Python Dependencies
└── README.md                           # เอกสารนี้
```

## 🎯 Pipeline Flow

### 1. 🧪 Unit Tests (Parallel)
- ทดสอบ `guess_column_types()` - เดาประเภทข้อมูลจากไฟล์ CSV
- ทดสอบ `filter_issue_date_range()` - กรองข้อมูลตามช่วงวันที่ 2016-2019
- ทดสอบ `clean_missing_values()` - ลบคอลัมน์ที่มี missing values มากกว่า 30%

### 2. 🔄 ETL Processing
- โหลดข้อมูลจาก `LoanStats_web_small.csv`
- ใช้ฟังก์ชันทั้ง 3 ในการประมวลผลข้อมูล
- สร้าง Star Schema (Fact + Dimension Tables)
- แสดงผลลัพธ์และสถิติ

### 3. 📤 Continuous Deployment
- ส่ง Fact Table และ Dimension Tables ไปยัง MSSQL Database
- รันทุกครั้งเมื่อ tests ผ่านทั้งหมด
- Database: `mssql.minddatatech.com/TestDB`

## 🚀 Quick Start

### Prerequisites
- Python 3.9+
- Jenkins with Pipeline plugin
- Access to MSSQL Server (`mssql.minddatatech.com`)
- Data file: `../dataops-foundation-jenkins/data/LoanStats_web_small.csv`

### Local Testing

```bash
# 1. Setup environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# หรือ venv\Scripts\activate  # Windows

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run unit tests
cd tests
python guess_column_types_test.py
python filter_issue_date_range_test.py
python clean_missing_values_test.py

# 4. Run ETL pipeline
cd ..
python etl_pipeline.py

# 5. Run with database deployment
python etl_pipeline.py --deploy
```

## 🔧 Jenkins Setup

### 1. Create Jenkins Job
```
1. New Item → Pipeline
2. Pipeline script from SCM
3. Git Repository: [your-repo-url]
4. Script Path: Jenkinsfile
```

### 2. Configure Credentials
```
Manage Jenkins → Manage Credentials → Add Credentials
- Kind: Secret text
- ID: mssql-password
- Secret: Passw0rd123456
```

### 3. Database Configuration
```
Server: mssql.minddatatech.com
Database: TestDB  
Username: SA
Password: Passw0rd123456 (from Jenkins credentials)
```

### 4. Pipeline Parameters (Optional)
```
DEPLOY_TO_DB: Boolean (default: false) - ไม่จำเป็นแล้ว เพราะ deploy ทุกครั้ง
```

## 📊 ETL Functions

### 1. `guess_column_types(file_path)`
```python
success, column_types = guess_column_types('data.csv')
# Returns: (True, {'col1': 'integer', 'col2': 'string', ...})
```

### 2. `filter_issue_date_range(df)`
```python
filtered_df = filter_issue_date_range(df)
# Keeps only records from 2016-2019
```

### 3. `clean_missing_values(df, max_null_percentage=30)`
```python
clean_df = clean_missing_values(df, max_null_percentage=30)
# Removes columns with >30% missing values
```

## 📈 Star Schema Output

### Dimension Tables
- **home_ownership_dim**: `home_ownership_id`, `home_ownership`
- **loan_status_dim**: `loan_status_id`, `loan_status`  
- **issue_d_dim**: `issue_d_id`, `issue_d`, `month`, `year`, `quarter`

### Fact Table
- **loans_fact**: `fact_id`, `loan_amnt`, `funded_amnt`, `term`, `int_rate`, `installment`, `home_ownership_id`, `loan_status_id`, `issue_d_id`

## 📝 Jenkins Pipeline Stages

```groovy
1. 🔄 Checkout & Setup     - โครงสร้างโปรเจค
2. 🐍 Python Environment   - ติดตั้ง dependencies  
3. 🧪 Unit Tests          - ทดสอบฟังก์ชันทั้ง 3 (parallel)
4. 🔍 ETL Validation      - ตรวจสอบ components
5. 🔄 ETL Processing      - รัน ETL pipeline
6. 📤 Deploy to Database  - ส่งข้อมูลไป MSSQL (conditional)
```

## 🎯 Expected Results

### Success Case
```
🎉 ETL Pipeline succeeded!
✅ All tests passed (12/12)
✅ ETL processing completed
✅ Star schema created:
   - 3 dimension tables
   - 1 fact table with X,XXX records
✅ Deployed to database
```

### Test Results
```
📊 SUMMARY RESULTS
==================
1. Test Case 1: การตรวจจับประเภทข้อมูลพื้นฐาน: ✅ PASS
2. Test Case 2: ทดสอบการตรวจจับรูปแบบวันที่และเวลา: ✅ PASS
3. Test Case 3: ทดสอบตัวแบ่งที่แตกต่างกัน: ✅ PASS
4. Test Case 4: Edge Cases: ✅ PASS

🎯 Overall Result: 4/4 tests passed
🎉 ALL TESTS PASSED!
```

## 🛠️ Troubleshooting

### Common Issues

1. **Database Connection Failed**
   ```
   ❌ Database connection failed: connection timeout
   ```
   **Solution**: ตรวจสอบ network และ credentials

2. **Data File Not Found**
   ```
   ⚠️ Data file not found: ../dataops-foundation-jenkins/data/LoanStats_web_small.csv
   ```
   **Solution**: ตรวจสอบ path ของไฟล์ข้อมูล

3. **Import Error**
   ```
   ModuleNotFoundError: No module named 'functions'
   ```
   **Solution**: รัน `pip install -r requirements.txt`

### Debug Mode
```bash
# เปิด debug mode
export JENKINS_DEBUG=true
python etl_pipeline.py
```

## 📞 Support

หากมีปัญหาหรือข้อสงสัย:
1. ตรวจสอบ Console Output ใน Jenkins
2. ดู logs ในส่วน troubleshooting
3. ตรวจสอบ requirements และ dependencies

## 🎖️ Features

- ✅ **Simple & Clean**: โครงสร้างง่าย เข้าใจได้
- ✅ **Parallel Testing**: รัน tests พร้อมกันเพื่อความเร็ว
- ✅ **Error Handling**: จัดการ error แบบครบถ้วน
- ✅ **Star Schema**: สร้าง dimension และ fact tables
- ✅ **Database Integration**: ส่งข้อมูลไป MSSQL อัตโนมัติ
- ✅ **CI/CD Ready**: พร้อมใช้กับ Jenkins pipeline

---

🎉 **Happy ETL Processing!** 🚀
