# Advanced Data Orchestration with Azure Data Factory 🚀

## 📋 Project Overview

This project demonstrates advanced data engineering patterns using **Azure Data Factory (ADF)**. The focus is on building resilient, automated, and efficient pipelines for hybrid data movement, integration with external systems, and modern data warehousing techniques.

---

## 🛠️ Technologies & Concepts Demonstrated

*   **Hybrid Data Movement:** Self-hosted Integration Runtime (SHIR)
*   **External Integration:** SFTP Connector
*   **Data Warehousing:** Incremental Loading (High-Watermark Method)
*   **Advanced Orchestration:** Complex Triggers, Activity Chaining
*   **Pipeline Resilience:** Retry Logic for Transient Failures
*   **Core Services:** Azure Data Factory, Azure SQL Database, Azure Blob Storage, On-Premises SQL Server

---

## 📂 Project Tasks & Implementation

### 1. Hybrid Data Movement: On-Premises to Cloud 🏢☁️

**Objective:** Securely extract data from a local SQL Server and load it into an Azure SQL Database using a Self-hosted Integration Runtime (SHIR).

**Architecture:**
*   A **local SQL Server** (simulated on a Windows machine) was set up with the Northwind database.
*   The **Self-hosted Integration Runtime (SHIR)** software was installed on the local machine.
*   The SHIR was securely registered with my Azure Data Factory instance using an authentication key. This created a secure bridge between the on-premises network and the Azure cloud.
*   An ADF pipeline was built to copy the `dbo.Orders` table from the local server to an Azure SQL Database.

**Evidence of Success:** 📸
*A screenshot showing the SHIR connected and running in the ADF Manage tab.*
`![SHIR Connected](path/to/shir-running.png)`

*A screenshot of the successful pipeline run moving data from on-prem to the cloud.*
`![On-Prem Pipeline Run](path/to/onprem-run.png)`

---

### 2. External System Integration: SFTP Data Extraction 🤝

**Objective:** Connect to an external partner's SFTP server to extract files.

**Implementation:**
*   A `Linked Service` was configured to connect to a public test SFTP server (`test.rebex.net`).
*   A `Copy data` pipeline (`PL_SFTP_File_Copy`) was built to transfer a file (`readme.txt`) from the SFTP server.
*   The file was copied in **Binary format** to preserve its integrity and loaded into Azure Blob Storage.

**Evidence of Success:** 📸
*A screenshot showing the `readme.txt` file in the Azure Blob Storage container.*
`![SFTP Output File](path/to/sftp-output.png)`

---

### 3. Incremental Data Loading (High-Watermark Method) 📈

**Objective:** Design an efficient daily pipeline that only processes new or updated records from a source table.

**Implementation:**
This pattern uses four sequential activities to ensure data integrity and efficiency.

1.  **`Lookup 1: GetOldWatermark`**: Retrieves the latest `OrderDate` from the last successful run, which is stored in a dedicated watermark table in the destination database.
2.  **`Lookup 2: GetNewWatermark`**: Finds the maximum `OrderDate` from the on-premises source table.
3.  **`Copy data: CopyDelta`**: Copies only the rows where the `OrderDate` is greater than the old watermark and less than or equal to the new one.
4.  **`Stored Procedure: UpdateWatermark`**: If the copy succeeds, this activity updates the watermark table with the new watermark value, preparing for the next run.

**SQL Scripts Used:**

*   **Watermark Table Creation:**
    ```sql
    CREATE TABLE dbo.WatermarkTable (
        TableName NVARCHAR(255) PRIMARY KEY,
        WatermarkValue DATETIME
    );
    INSERT INTO dbo.WatermarkTable (TableName, WatermarkValue) VALUES ('Orders', '1990-01-01');
    ```

*   **Stored Procedure for Updates:**
    ```sql
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

*   **Dynamic Query in Copy Activity Source:**
    ```javascript
    @concat('SELECT * FROM dbo.Orders WHERE OrderDate > ''', activity('GetOldWatermark').output.firstRow.WatermarkValue, ''' AND OrderDate <= ''', activity('GetNewWatermark').output.firstRow.NewWatermark, '''')
    ```

**Evidence of Success:** 📸
*A screenshot of the incremental pipeline canvas.*
`![Incremental Pipeline Canvas](path/to/incremental-pipeline.png)`

*A screenshot showing the second debug run where `Rows written: 0`, proving the incremental logic works.*
`![Incremental Proof Run](path/to/incremental-proof.png)`

---

### 4. Advanced Automation & Resilience ⚙️

**Objective:** Implement custom scheduling and improve pipeline resilience.

**Implementation:**

*   **Custom Monthly Trigger:** An ADF trigger was configured to run a pipeline on a specific schedule: the **Last Saturday of every month**. This is ideal for month-end financial or summary reports.

    `![Custom Monthly Trigger](path/to/monthly-trigger.png)`

*   **Activity Retry Logic:** The `Copy data` activities were configured with a **Retry** count of `3` and a **Retry Interval** of `30` seconds. This makes the pipeline more resilient by automatically retrying on transient network or service issues without failing the entire run.

    `![Retry Logic Configuration](path/to/retry-config.png)`

---

## 💡 Key Learnings

This assignment provided deep, practical experience in core data engineering concepts beyond simple data movement. I successfully implemented a hybrid data solution, integrated with external systems, built an efficient data warehousing pattern, and configured advanced automation and resilience features, all within Azure Data Factory.
