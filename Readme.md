
# Week 6 Assignment: Advanced Data Orchestration with Azure Data Factory 🚀

## 📋 Project Overview

This project demonstrates advanced data engineering patterns using **Azure Data Factory (ADF)**. The focus is on building **resilient, automated, and efficient pipelines** for hybrid data movement, external integration, and incremental data warehousing.

> 🔗 **Thanks to [CSI (Celebal Summer Internship)](https://www.celebaltech.com/)**  
> This task deepened my understanding of cloud-scale data pipelines, showcasing real-world techniques and design considerations.

---

## 🛠️ Technologies Used

- **Orchestration:** Azure Data Factory (ADF)
- **Cloud Services:** Azure SQL Database, Azure Blob Storage
- **On-Premises:** SQL Server via SHIR
- **External Source:** SFTP Server (test.rebex.net)
- **Runtime:** Self-Hosted Integration Runtime (SHIR)
- **Automation:** Schedule Triggers, Retry Logic

---

## 📂 Project Tasks & Implementation

### 1️⃣ Hybrid Data Movement: On-Premise SQL Server to Azure SQL Database 🏢☁️

**Objective:** Securely extract **all tables** from a local SQL Server and load them into Azure SQL DB using **Self-hosted Integration Runtime (SHIR)**.

#### Steps:
- Used my **friend’s Windows system** to install **SQL Server** and set up the **Northwind DB** (since I use a **MacBook**).
- Installed **Self-Hosted Integration Runtime (SHIR)** and registered it in my Azure Data Factory.
- Created a **dynamic pipeline (`PL_Full_OnPrem_Replication`)** that:
  - Uses a `Lookup` activity to list all table names
  - Applies `ForEach` + `Copy data` to replicate **all tables**
- Also replicated the same setup using a **Virtual Machine** for validation.

📸 **Evidence:**

- ✅ Self-hosted node is connected to the cloud service

  ![SHIR](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/d21d972a22cf92c2181cb61d90d6924cca3ef01b/Screenshots/Self-hosted%20node%20is%20connected%20to%20the%20cloud%20service.jpeg)
  
- ✅ `SHIR` visible and running in Integration Runtimes
  
  ![SHIR](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/Integration%20runtimes.png)
  
- ✅ `PL_Full_OnPrem_Replication` pipeline copied all tables

  ![`PL_Full_OnPrem_Replication` pipeline copied all tables](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/PL_Full_OnPrem_Replication%20pipeline%20copied%20all%20tables.png)
  
- ✅ Azure SQL DB showing all replicated tables

  ![Azure SQL DB showing all replicated tables](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/Cloud_Destination_DB%20.png)

- ✅ Protocols for SQL Server

   ![Protocols](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/d21d972a22cf92c2181cb61d90d6924cca3ef01b/Screenshots/Protocols%20for%20SQL%20Server.jpeg)

---

### 2️⃣ External System Integration: SFTP File Extraction 📥

**Objective:** Connect to an external SFTP server and copy files to Azure Blob Storage.

#### Steps:
- Configured linked service to **`test.rebex.net`** (public SFTP)
- Built pipeline `PL_SFTP_File_Copy` to download `readme.txt` using Binary format
- Stored in Azure Blob Storage container `data-output`

📸 **Evidence:**
- ✅ SFTP connection succeeded via SHIR

  ![SFTP](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/SFTP%20Linked%20Service%20in%20ADF.png)
  
- ✅ File `readme.txt` copied to Azure Blob Storage

  ![File](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/Azure%20Blob%20Storage%20account%20(container)%20.png)
  
- ✅ Verified blob presence in `data-output` container

  ![blob](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/Created%20a%20Container.png)

  

---

### 3️⃣ Incremental Loading Using High-Watermark Logic 📈

**Objective:** Move only new or updated data from the Orders table using watermark pattern.

#### Activities Used:
- `Lookup: GetOldWatermark`
- `Lookup: GetNewWatermark`
- `Copy data: CopyDelta`
- `Stored Procedure: UpdateWatermark`

#### SQL Scripts Used:

```sql
-- Create watermark table
CREATE TABLE dbo.WatermarkTable (
    TableName NVARCHAR(255) PRIMARY KEY,
    WatermarkValue DATETIME
);
INSERT INTO dbo.WatermarkTable VALUES ('Orders', '1990-01-01');
```

```sql
-- Stored procedure to update watermark
CREATE PROCEDURE dbo.sp_UpdateWatermark
    @NewWatermarkValue DATETIME,
    @TableName NVARCHAR(255)
AS
BEGIN
    UPDATE dbo.WatermarkTable
    SET WatermarkValue = @NewWatermarkValue
    WHERE TableName = @TableName;
END
```

```sql
-- Dynamic query for Copy Activity
@concat(
  'SELECT * FROM dbo.Orders WHERE OrderDate > ''',
  activity('GetOldWatermark').output.firstRow.WatermarkValue,
  ''' AND OrderDate <= ''',
  activity('GetNewWatermark').output.firstRow.NewWatermark,
  ''''
)
```

📸 **Evidence:**
- ✅ Watermark table created successfully

  ![blob](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/24cf491d5d98dd42f8c534c99cfcab55f62f9a6d/Screenshots/Watermark%20table%20created%20successfully.png)
  
- ✅ PL_Incremental_Load_Orders

  ![blob](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/24cf491d5d98dd42f8c534c99cfcab55f62f9a6d/Screenshots/PL_Incremental_Load_Orders.png)
  

---

### 4️⃣ Pipeline Resilience & Automation ⚙️

**Objective:** Add reliability with retry logic and automate runs with monthly trigger.

#### Implementation:
- Configured **Retry Count: 3**, **Interval: 30 seconds** on critical activities
- Created **custom trigger** to run on the **last Saturday of every month at 7 AM**

📸 **Evidence:**
- ✅ Monthly Trigger scheduled correctly

  ![blob](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/TR_Monthly_Last_Saturday.png)
  
- ✅ Trigger fired successfully and executed master pipeline

  ![blob](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/Master%20PL.png)
  
- ✅ Retry logic handled transient failures in debug mode

- ![blob](https://github.com/Ayush03A/Celebal-DE-Internship-Week-6/blob/32e4cbefae8455cba8693b8e3df800e405e38562/Screenshots/Retrieving%20data.%20Wait%20a%20few%20seconds%20and%20try%20to%20cut%20or%20copy%20again.png)


---

## ✅ Summary Table

| Task                              | Status   |
| --------------------------------- | -------- |
| Hybrid On-Prem → Azure SQL        | ✅ Done  |
| SFTP File Transfer → Blob Storage | ✅ Done  |
| Incremental Copy via Watermark    | ✅ Done  |
| Retry Logic + Monthly Trigger     | ✅ Done  |

---

## ❗ Challenges Faced

- 💻 I use a **MacBook**, so I couldn’t run SQL Server locally.
  - 👉 Used my **friend’s Windows machine** to install and run SQL Server
- 🧪 Also tried a **Windows Virtual Machine** on Azure for better testing flexibility.
- 🔐 Faced issues with **SHIR registration and connection**, had to regenerate keys multiple times.
- 🔁 Copying **all tables dynamically** needed handling table names with spaces/brackets.
- 🐞 Debugged dynamic SQL expressions in Copy Activities using JSON input tracing.

---

## 🧠 Key Learnings

- Built **hybrid pipelines** securely using Self-hosted IR
- Created **metadata-driven pipelines** with Lookup + ForEach
- Applied **watermark logic** for incremental loads
- Successfully handled **external integration via SFTP**
- Automated data pipelines with **custom triggers and retry handling**
- Gained **hands-on project experience** with ADF pipeline design

---

## 🙏 Acknowledgements

Huge thanks to my amazing mentors and HR team for constant guidance and feedback throughout this journey:

👨‍🏫 **Jash Tewani & Ajit Kumar Singh** – Technical Mentor  
🙌 **Prerna Kamat** – HR, Celebal CSI  
🙌 **Priyanshi Jain** – HR, Celebal CSI  
🏢 **Celebal Technologies** – For this amazing real-world data engineering internship
